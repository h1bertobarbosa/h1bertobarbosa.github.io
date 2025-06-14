---
layout: post
author: humberto_barbosa
title: "Quando a telemetria virou o bug: um caso real com Mongoose e OpenTelemetry"
date: 2025-06-14 20:14:00 -0300
categories: [Engenharia de Software, Microsserviços, Monitoramento e Telemetria, Resolução de Problemas, Boas Práticas em Desenvolvimento, Integrações de Sistemas, Lições Aprendidas, Cultura DevOps, Casos Reais de Debug]
tags: [NodeJS, Mongoose, OpenTelemetry, MongoDB, Microservices, Observability, Debugging, Software Engineering, DevOps, Backend, Cloud, Instrumentation, API Integration, Developer Experience, Production Issues]
---

**O erro inesperado**

Num ecossistema moderno de microsserviços, estabilidade é lei. Nossa stack — fundamentada em Node.js, NestJS e Mongoose — nos passava segurança. Para manter tudo sob vigilância, contávamos com o OpenTelemetry, ferramenta poderosa para rastrear o comportamento da aplicação. Tudo parecia sob controle até que um erro traiçoeiro começou a pipocar justamente num de nossos conectores mais críticos: `MongoRuntimeError: Unexpected null cursor id`.

Neste relato, dividimos os bastidores da investigação que nos levou de suspeitas sobre o MongoDB até uma descoberta surpreendente na camada de instrumentação da nossa própria aplicação.

**O sintoma**

O sinal de alerta soava sempre no mesmo serviço: o app responsável por integrar pedidos e acompanhar entregas com um parceiro logístico. Bastava uma consulta mais pesada — envolvendo leitura de grandes volumes de dados — e pronto: falha abrupta, com a mensagem:

```json
{
  "level": "error",
  "message": "Unexpected null cursor id. A cursor creating command should have set this",
  "stack": [
    "MongoRuntimeError: Unexpected null cursor id...",
    "at FindCursor.getMore (/app/node_modules/.../mongodb/lib/cursor/abstract_cursor.js:605:19)",
    "at FindCursor.toArray (/app/node_modules/.../mongodb/lib/cursor/abstract_cursor.js:427:26)"
  ]
}
```

O estrago era imediato: o serviço não parava em pé.

**A descoberta**

Quando esgotamos todas as hipóteses óbvias, sobrou olhar para o que ninguém costuma ver: a **instrumentação de telemetria**. Foi então que resolvemos um teste quase desesperado — desligar o OpenTelemetry temporariamente. O resultado? Silêncio total. O erro sumiu como se nunca tivesse existido. Identificamos o culpado, mas ficou a pulga atrás da orelha: _como exatamente isso acontecia?_

**Entendendo a causa**

A explicação estava no coração do OpenTelemetry: a prática do _monkey-patching_. Em outras palavras, a biblioteca se infiltra nas funções internas do Mongoose e do driver do MongoDB — envelopando métodos como `.find()`, `.exec()` e `.cursor()` para medir tempos de execução e coletar metadados.

Acontece que havia um desalinhamento crítico. Nosso projeto rodava o **Mongoose v8**, que usa internamente o **MongoDB Node.js Driver v6**. Só que nossas bibliotecas de instrumentação ainda estavam defasadas, escritas para versões mais antigas.

O resultado foi um curto-circuito: mudanças internas do Mongoose v8 alteraram o jeito de gerenciar cursores e seus IDs. A instrumentação velha, ao tentar monitorar isso, acabava bagunçando o fluxo — e o `cursor.id` simplesmente se perdia antes da requisição do próximo lote de dados.

**A resolução**

Uma vez descoberto o ponto fraco, a correção foi quase trivial: bastava alinhar as versões do OpenTelemetry com o novo Mongoose. Bastou uma alteração no `package.json`:

```diff
- "@opentelemetry/instrumentation-mongoose": "0.38.0",
+ "@opentelemetry/instrumentation-mongoose": "0.47.0",

- "@opentelemetry/instrumentation-mongodb": "0.43.0",
+ "@opentelemetry/instrumentation-mongodb": "0.53.0",
```

Com as dependências sincronizadas, reimplantamos o serviço — e não só o erro evaporou, como a telemetria voltou a funcionar impecavelmente.

O Que Fica de Lição

Essa saga de debugging trouxe lições preciosas:

Observabilidade tem custo — Instrumentação não é só passiva: é código rodando junto com seu sistema, sujeito a bugs e incompatibilidades.

Grandes atualizações pedem atenção ao redor — Migrar para uma nova versão principal de uma dependência exige verificar se todo o ecossistema (plugins, wrappers, monitoramentos) acompanha o ritmo.

Desligar para isolar — Quando tudo parece sem explicação, desligar componentes um a um ainda é uma das táticas mais eficientes para achar o verdadeiro vilão.

No fim, um erro incômodo no banco de dados acabou ensinando mais do que mil tutoriais sobre a delicada dança entre dependências e monitoramento. Se esta experiência economizar horas de investigação para outro time, já valeu cada linha de log analisada.

Com as dependências sincronizadas, reimplantamos o serviço — e não só o erro evaporou, como a telemetria voltou a funcionar impecavelmente.

**O Que Fica de Lição**

Essa saga de debugging trouxe lições preciosas:

1. **Observabilidade tem custo** — Instrumentação não é só passiva: é código rodando junto com seu sistema, sujeito a bugs e incompatibilidades.

2. **Grandes atualizações pedem atenção ao redor** — Migrar para uma nova versão principal de uma dependência exige verificar se todo o ecossistema (plugins, wrappers, monitoramentos) acompanha o ritmo.

3. **Desligar para isolar** — Quando tudo parece sem explicação, desligar componentes um a um ainda é uma das táticas mais eficientes para achar o verdadeiro vilão.


No fim, um erro incômodo no banco de dados acabou ensinando mais do que mil tutoriais sobre a delicada dança entre dependências e monitoramento. Se esta experiência economizar horas de investigação para outro time, já valeu cada linha de log analisada.
