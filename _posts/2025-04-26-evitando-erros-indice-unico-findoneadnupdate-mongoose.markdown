---
layout: post
author: humberto_barbosa
title: "Evitando Erros de Índice Único (E11000) ao Usar findOneAndUpdate com upsert no Mongoose"
date: 2025-04-26 20:11:00 -0300
categories: [Banco de Dados, Desenvolvimento Backend, Arquitetura de Sistemas, Boas Práticas de Programação, Gestão de Entregas]
tags: [MongoDB, Mongoose, Upsert, Índices Únicos, Multi-tenant, Modelagem de Dados, E11000, findOneAndUpdate, Boas Práticas, Node.js]
---

Como garantir unicidade de remessas em sistemas multi-tenant no MongoDB? Entenda por que erros E11000 acontecem, como alinhar filtros de upsert com índices únicos e evite armadilhas clássicas em modelagem de dados para operações críticas.

## ✅ Cenário

Imagine um sistema de gestão de entregas com múltiplos clientes (empresas) utilizando a mesma plataforma. Cada cliente (representado por um `tenantId`) possui seus próprios **pedidos internos** (`clientOrderId`), que podem coincidir entre diferentes clientes, mas **não dentro do mesmo cliente**.

Cada pedido tem **uma única remessa associada**, representada por um `shipment` com informações como nota fiscal, status e dados logísticos.

Você precisa garantir que **haja apenas uma remessa por pedido e por cliente** — e que atualizações venham a sobrescrever remessas existentes se o pedido já existir.

---

## 💾 Estrutura simplificada dos dados

```typescript
@Schema({
  collection: 'shipments',
  timestamps: true
}) export class ShipmentEntity extends Document {@Prop({
    type: String,
    required: true
  }) tenantId: string;@Prop({
    type: String,
    required: true
  }) clientOrderId: string;@Prop({
    type: String,
    required: true
  }) invoiceNumber: string;@Prop({
    type: String,
    required: true
  }) status: string;@Prop({
    type: Date,
  default:
    Date.now
  }) updatedAt:
  Date;
}
```

---

## 🔐 Definindo o índice único

Para garantir unicidade por cliente + pedido:

```typescript
ShipmentSchema.index(   { tenantId: 1, clientOrderId: 1 },   { unique: true }, );
```

Esse índice impede que existam dois documentos com a mesma combinação `tenantId` e `clientOrderId`.

---
### 📈 O Risco pro negocio

Quando pensamos em unicidade de remessas, não é só uma questão de evitar erros técnicos como o E11000.
**É sobre proteger a integridade operacional do sistema.**

Se um pedido tiver duas remessas associadas por erro, as consequências podem incluir:

- **Faturamento duplicado**: o cliente pode ser cobrado duas vezes pelo mesmo pedido.
- **Erro logístico**: o centro de distribuição pode despachar duas cargas separadas para o mesmo destino.
- **Problemas fiscais**: inconsistência nas notas fiscais emitidas pode gerar multas ou retrabalho junto à Receita.
- **Perda de confiança**: clientes empresariais esperam precisão; falhas assim corroem a relação de confiança.


Ou seja:
**Definir corretamente a unicidade dos dados não é só uma questão técnica — é uma salvaguarda para a operação e para o relacionamento com o cliente.**

## ❌ O erro clássico: E11000

Se você tentar fazer um `findOneAndUpdate` assim:

```typescript
this.shipmentModel.findOneAndUpdate({clientOrderId: shipment.clientOrderId, invoiceNumber: shipment.invoiceNumber}, { $set: shipment }, { new: true, upsert: true });
```

Pode falhar com o erro:

```bash
E11000 duplicate key error collection: logistics.shipments index: tenantId_1_clientOrderId_1 dup key: { tenantId: "tenantA", clientOrderId: "12345" }
```

Isso ocorre porque o filtro ignora `tenantId`, violando a restrição única definida pelo índice.

---

## ✅ Solução: usar o filtro certo

```ts
await this.shipmentModel.findOneAndUpdate({tenantId: shipment.tenantId,clientOrderId: shipment.clientOrderId}, { $set: shipment }, { new: true, upsert: true } );
```

Agora o `upsert` respeita a chave única, atualizando corretamente se já houver uma remessa com o mesmo pedido para o mesmo cliente, ou criando um novo registro caso contrário.

---

## 🧠 Conclusão

Quando trabalhamos com `upsert` no MongoDB, é fundamental que o filtro usado esteja alinhado com o índice único que definimos. No caso que discutimos, isso significa sempre usar `tenantId` e `clientOrderId` juntos. Se você esquecer um deles ou misturar outros campos, como `invoiceNumber`, pode acabar quebrando a unicidade e gerando erros difíceis de diagnosticar.

É importante lembrar que buscar por qualquer campo no MongoDB é possível — mas isso não significa que qualquer campo serve para garantir unicidade em operações críticas. Se sua operação depende da certeza de que "não existe outro igual", então o campo (ou a combinação de campos) precisa estar refletido no índice único.

Quando acontece um erro como o E11000, não é só um problema de código quebrado. É um sinal de que a modelagem dos dados, a estratégia de identificação e a operação do sistema podem estar desalinhadas. Esses erros são uma ótima oportunidade para revisar e fortalecer o desenho do seu banco.

Por fim, sempre vale a pena pensar além da parte técnica: proteger a unicidade dos dados é proteger também a integridade do seu processo logístico, financeiro e de relacionamento com os clientes. No final das contas, um sistema que respeita essas premissas é mais robusto, confiável — e mais fácil de evoluir.
