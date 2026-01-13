---
layout: post
author: humberto_barbosa
title: "Como um cache simples reduziu nosso tempo de cotacao em 80%"
date: 2026-01-13 08:50:00 -0300
categories: [ Desenvolvimento Backend,
  Arquitetura de Software,
  Node.js,
  Performance de Código,
  System Design]
tags: [Node.js,
  TypeScript,
  Cache,
  Cache-Aside,
  Performance,
  Latência,
  Otimização de Código,
  Integrações Externas,
  APIs de Terceiros,
  MongoDB,
  TTL,
  Hashing,
  SHA-512,
  Backend,
  Boas Práticas,
  Sistemas em Produção]
---

Esses dias eu estava lembrando de um problema interessante que enfrentei em um serviço de cotação de frete.

O sintoma era simples, mas caro: o sistema fazia a mesma requisição várias vezes para os provedores de frete externos. Isso deixava a cotação lenta para o cliente e, pior, gerava um monte de chamadas desnecessárias para APIs de terceiros.

Na prática, cada cotação levava algo em torno de **1 a 1,5 segundo**, mesmo quando alguém pedia exatamente o mesmo CEP, peso e dimensões minutos depois.

Meu objetivo era bem claro: **deixar a cotação rápida para o usuário** e, ao mesmo tempo, **reduzir o volume de chamadas externas**.

A solução acabou sendo mais simples do que parecia: **cache**.

---

## A ideia: Cache-Aside

Implementei o padrão **Cache-Aside**. A premissa é direta:

1. O sistema tenta buscar o resultado no cache 
2. Se existir → retorna imediatamente
3. Se não existir → chama o provedor externo, salva no cache e segue o fluxo normal
    
O ponto crítico era garantir que **apenas cotações realmente idênticas** compartilhassem o mesmo resultado.

Para isso, criei uma **chave única** baseada nos dados relevantes da cotação.

---

## Criando a chave de cache

A chave era derivada de informações como CEP, peso e dimensões da embalagem. Para evitar colisões e garantir consistência, usei um hash **SHA-512**.

```typescript
import crypto from 'crypto';

type FreightParams = {
  originZip: string;
  destinationZip: string;
  weight: number;
  width: number;
  height: number;
  length: number;
};

function buildCacheKey(params: FreightParams): string {
  const rawKey = JSON.stringify(params);

  return crypto
    .createHash('sha512')
    .update(rawKey)
    .digest('hex');
}
```

Com isso, qualquer variação mínima nos dados gerava uma chave diferente.  
Cotações realmente iguais → mesma chave → mesmo cache.

---

## Buscando primeiro no cache

Antes de chamar qualquer API externa, o fluxo sempre passava pelo cache.

```typescript
async function getCachedQuotation(
  cacheKey: string,
): Promise<FreightQuotation | null> {
  return mongoCollection.findOne({
    cacheKey,
    expiresAt: { $gt: new Date() },
  });
}
```

Se existisse um resultado válido, a resposta era praticamente imediata.

---

## Cache miss: chamar o provedor externo

Quando o cache não tinha o dado, aí sim o sistema fazia a chamada externa.

```typescript
async function fetchQuotationFromProvider(
  params: FreightParams,
): Promise<FreightQuotation> {
  const response = await freightProviderApi.post('/quote', params);

  return response.data;
}
```

Depois disso, o resultado era salvo no cache.

---

## Salvando no cache com TTL

Definimos um **TTL de 24 horas**, porque frete não muda a cada minuto.  
Esse valor trouxe um bom equilíbrio entre **frescor dos dados** e **eficiência**.

```typescript
async function saveQuotationInCache(
  cacheKey: string,
  quotation: FreightQuotation,
) {
  await mongoCollection.insertOne({
    cacheKey,
    quotation,
    createdAt: new Date(),
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24h
  });
}
```

Também garanti que o acesso fosse rápido, criando **índice no campo `expiresAt`**, o que facilitava tanto leitura quanto limpeza automática de dados antigos.

---

## O fluxo completo

Juntando tudo, o fluxo principal ficava assim:

```typescript
async function getFreightQuotation(
  params: FreightParams,
): Promise<FreightQuotation> {
  const cacheKey = buildCacheKey(params);

  const cached = await getCachedQuotation(cacheKey);
  if (cached) {
    return cached.quotation;
  }

  const quotation = await fetchQuotationFromProvider(params);

  await saveQuotationInCache(cacheKey, quotation);

  return quotation;
}
```

Simples, previsível e fácil de manter.

---

## O impacto

O impacto foi imediato.

- ⏱️ **Tempo médio de resposta:** de ~1s para **100–200ms**
- 🔻 **Redução de chamadas externas:** cerca de **60%**
- 💰 Menos custo com APIs de terceiros
- 📈 Sistema muito mais previsível e estável
    

---

## Conclusão

No fim das contas, não foi sobre usar uma tecnologia “mágica”.

Foi sobre **entender o fluxo**, respeitar o domínio do negócio e aplicar um padrão simples de forma bem pensada.

É esse tipo de ajuste invisível que, quando bem feito, muda completamente a experiência de quem está do outro lado da tela.
