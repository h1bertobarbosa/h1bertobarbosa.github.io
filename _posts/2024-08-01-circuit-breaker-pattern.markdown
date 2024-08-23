---
layout: post
author: Humberto Barbosa
title:  "Padrão - Circuit Breaker"
date:   2024-08-05 11:28:14 -03
categories: [Patterns, Blogging, General]
tags: [patterns]
---

## O que é Circuit Breaker?

### Definição e objetivo

Como tornar o sistema mais resiliente e tolerante a falhas? Resiliência é apenas uma palavra bonita para a capacidade do seu sistema de se recuperar de falhas e continuar operando. Quem popularizou esse padrão foi Miachel Nygard, ele é usado para proteger seu sistema de falhas em serviços externos e evitar cascatas de erros. Ele permite que você construa um serviço tolerante a falhas quando os principais serviços consumidos estiverem indisponíveis ou com alta latência.

### Analogia com disjuntor
💡
Esse padrão funciona exatamente como um disjuntor desses que você tem na sua casa, quando ele recebe uma sobrecarga de energia ele cai (abre) e corta o fluxo de energia. Alguém precisa ir lá e fechar o circuito para o fluxo voltar. No software ele funciona da mesma maneira, quando o sistema tem um fluxo muito alto de requisições e você precisa se comunicar com outra API e este recurso remoto está dando timeout, geralmente 30 segundos para devolver a requisição, se isso acontecer você terá um encadeamento de erros, sobrecarga no servidor e isso pode provocar a queda do seu serviço.

Então o padrão Circuit Breaker (Disjuntor) ajuda a evitar falhas em cascata, introduzindo uma camada de proteção fornecendo um mecanismo para quando uma requisição remota falhar ou demorar muito para responder, o seu sistema identifique isso e devolva uma resposta para evitar que o sistema fique sobrecarregado.

## Como funciona?

### Os três estados: Close, Open e HalfOpen

- **Close**: No estado fechado, o Circuit Breaker permite que as solicitações passem para o sistema subjacente. Esse é o estado normal de operação.
- **Open**: Quando o Circuit Breaker está no estado aberto, ele impede que todas as solicitações sejam enviadas para o sistema subjacente. Isso é feito para evitar novas falhas e para dar ao sistema a chance de se recuperar.
- **HalfOpen**: No estado semiaberto, o Circuit Breaker permite que um número limitado de solicitações passe para o sistema subjacente. Isso é feito para ver se o sistema pode lidar com solicitações novamente, sem sobrecarregá-lo. Se as solicitações forem bem-sucedidas, o Circuit Breaker fará a transição de volta para o estado fechado (normal). Se as solicitações falharem, o disjuntor retornará ao estado aberto.

### Diagrama dos estados

![image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*iR5aU3Qs5TC5zJWz41HkWg.png)

## Implementação

Segue uma implementação que eu fiz em Node.js bem simples, mas que dá pra ver isso acontecer.

Com o Express eu subi dois servidores HTTP em portas diferentes: 3000 e 3001. O 3001 está fazendo o papel de um serviço externo. Quando eu acesso `localhost:3000/` ele chama o 3001, que dependendo de um número randômico de 0 a 10 vai me retornar 200 ou 400 (com delay de 150ms).

Eu usei uma lib conhecida chamada opossum, e nela está configurado timeout de 100ms, resetTimeout de 10 segundos e errorThresholdPercentage. A lib vai acumulando as requisições e quando chegar em uma porcentagem definida de requisições com erro, ele abre o circuito.

Coloquei a chamada HTTP para a porta 3001 dentro da função `asyncFunctionThatCouldFail` (função assíncrona que pode falhar) e jogo essa função pra dentro do meu objeto `breaker.fire`. Ele vai executar a função e observar o tempo e as responses.

```javascript
import express from "express";
import fetch from 'node-fetch';
import CircuitBreaker from 'opossum';

const app = express();
const app2 = express();

async function asyncFunctionThatCouldFail() {
  return new Promise((resolve, reject) => {
    fetch('http://localhost:3001/')
      .then(res => res.json())
      .then(json => {
        resolve(json)
      }).catch(err => reject(err))
  });
}

const options = {
  timeout: 100, // If our function takes longer than 100ms, trigger a failure
  errorThresholdPercentage: 50, // When 50% of requests fail, trip the circuit
  resetTimeout: 10000 // After 10 seconds, try again.
};

const breaker = new CircuitBreaker(asyncFunctionThatCouldFail, options);

app.get("/", async (req, response) => {
  breaker.fire()
    .then(res => response.json(res))
    .catch(err => response.status(500).json(err));
});

app2.get("/", (req, res) => {
  const randomNumber = Math.round(Math.random() * 10)
  console.log(randomNumber)
  if (randomNumber <= 5) {
    res.status(200).json({msg: "Success!"});
  } else {
    setTimeout(() => {
      res.status(400).json({msg: "Failed!"});
    }, 150)
  }
});

app.listen(3000, () => console.log(`Listening at http://localhost:3000`));
app2.listen(3001, () => console.log(`Listening at http://localhost:3001`));

breaker.on('open', () => {
  console.log('circuit has opened')
})

breaker.on('halfOpen', () => {
  console.log('circuit has halfOpened')
})

breaker.on('close', () => {
  console.log('circuit has closed')
})
```

## Considerações

Você deve estar pensando, mas porque existe o estado HalfOpen (semi aberto)? Bom, em um disjuntor aí da sua casa você precisa acionar ele manualmente depois que ele é aberto, mas no software não faz sentido essa ação de fechar o circuito ser manual. Ao implementar o padrão, você vai configurar para quando o circuito foi aberto (erro), quanto tempo vamos esperar para tentar se comunicar novamente com o recurso remoto. Após esse tempo, o sistema envia uma requisição novamente para verificar se está tudo certo. Estando tudo ok, o circuito é fechado novamente e tudo volta ao normal.

Então enquanto o circuito estiver close (fechado) está tudo certo, ele envia e recebe o fluxo de requisições e comunicação com um sistema/api remota. Se a API demorar X tempo, que será definido por você (200ms, 1s etc.), se a API remota demorar o tempo definido, o sistema entende que deu timeout sem precisar esperar os 30 segundos. Quando isso acontece, o circuito é aberto e a nossa aplicação para de se comunicar com o recurso remoto e passa a responder sem encavalar requisições.

Esse padrão não deve ser usado para o tratamento de exceções na lógica de negócios de seus aplicativos.

[Código no GIT](https://github.com/h1bertobarbosa/circuit-breaker-pattern)
[Artigo Original](https://dev.to/h1bertobarbosa/padrao-circuit-breaker-3dm3)

## Links pesquisados:
https://blog.appsignal.com/2020/07/22/nodejs-resiliency-concepts-the-circuit-breaker.html
https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker
https://martinfowler.com/bliki/CircuitBreaker.html
https://medium.com/trainingcenter/design-pattern-para-microservices-circuit-breaker-f4a5b68f73d1
