---
layout: post
author: humberto_barbosa
title:  "O que é runtime javascript ?"
date:   2024-08-21 08:59:14 -0300
categories: [Javascript, nodejs,General]
tags: [javascript]
---

# O que é runtime javascript ?


## O que é o ambiente de execução(runtime environment) ?

No contexto do browser esse ambiente de execução é composto por:

-   Javascript Engine
-   Web API
-   Fila de callbacks(The callback queue)
-   Loop de eventos(Event Loop)
![enter image description here](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tm6rtjjni7ma2p5u6i8a.png)
## Javascript Engine

Javascript Engine é um programa que executa código JavaScript. É o que torna possível a execução de código JavaScript em um navegador web ou em um ambiente de servidor, como o Node.js e agora mais recentemente o Deno.

Cada navegador web tem sua própria implementação de uma engine JavaScript, como o JavaScript V8 da Google, que é usado no Chrome e no Chromium, ou o Spider Monkey da Mozilla, que é usado no Firefox.

As engines JavaScript são otimizadas para interpretar e executar o código JavaScript de maneira rápida e eficiente. Elas fazem isso através de diversas técnicas, como pré-compilação de código e otimização de memória. As engines também implementam APIs JavaScript, como a API do Document Object Model (DOM), que permite que o código JavaScript acesse e manipule elementos HTML na página da web.

**Call Stack**: Dentro dessa engine nos temos a call stack e memory heap, toda vez que agente executa um código javascript as funções vão para a call stack que é o local onde as funções são empilhadas para serem executadas, essa estrutura é formada ultimo a entrar é primeiro a sair e só passa para a próxima execução quando concluir a atual.
Caso existe uma função da web api na call stack essa função é enviada para lá, apos o termino da execução o callback é enviado para a fila de callback(callback queue) para retornar pra call stack pra concluir a execução do código

**Memory Heap**: é um "pedaço" da memória do computador que é usada para alocar novos objetos. Quando um novo objeto é criado no código JavaScript, ele é alocado na memory heap.

A memory heap é dividida em duas partes: new space e old space. O new space é usada para alocar novos objetos e é otimizada para alocação rápida. O old space é usado para objetos que são mantidos por mais tempo e é otimizada para acesso rápido.

Quando new space está cheia, o V8 realiza uma operação chamada garbage collection, durante a qual ele examina os objetos no new space e remove aqueles que não são mais usados ​​pelo código. Os objetos que ainda são usados ​​são movidos para o old space. Isso libera espaço no new space para novos objetos.

O V8 também faz o garbage collection no old space, mas elas são menos frequentes e mais lentas do que as coletas no new space, pois envolvem a verificação de todos os objetos no old space.

Espero que isso tenha esclarecido um pouco sobre como a memory heap funciona no JavaScript V8.

## Web API

JavaScript Web APIs são conjuntos de interfaces de programação de aplicativos (APIs) que fornecem acesso a recursos do navegador e do sistema operacional. Eles permitem a criação de aplicativos web que possam fazer coisas como acessar a câmera e o microfone do usuário, enviar solicitações HTTP, manipular o DOM e muito mais. Alguns exemplos de JavaScript Web APIs incluem:

**Geolocation API**: permite que os aplicativos web acessem as informações de localização do usuário.
**Web Audio API**: permite que os aplicativos web reproduzam áudio e criem efeitos de áudio complexos.
**Web Storage API**: permite que os aplicativos web armazenem dados no navegador do usuário, como informações de login ou preferências de usuário.
Essas são apenas algumas das muitas APIs disponíveis para os desenvolvedores de aplicativos web. Elas fornecem uma ampla gama de recursos que podem ser usados ​​para criar aplicativos web poderosos e interativos.
O famoso setTimeout, ajax(xmlhttprequest) fazem parte da web api

## Callback queue e Event Loop

A fila de callbacks, também conhecida como a "fila de eventos", é uma fila de funções que são chamadas quando um evento específico ocorre. No JavaScript, esses eventos podem incluir coisas como o carregamento de uma página web, o clique em um botão ou o recebimento de uma resposta de uma solicitação HTTP.

Quando um evento ocorre, a função de callback associada a ele é adicionada à fila de callbacks. O JavaScript possui um loop de eventos que executa continuamente e verifica a fila de callbacks para ver se há funções para serem chamadas. Quando uma função está disponível na fila, ela é removida da fila e chamada. Isso permite que o código JavaScript responda a eventos de maneira assíncrona.

Por exemplo, quando um usuário clica em um botão em uma página web, é adicionada uma função de callback à fila de callbacks. Quando o loop de eventos verifica a fila, essa função é chamada e o código associado ao clique no botão é executado. Isso permite que o código continue a ser executado enquanto o navegador está ocupado processando o clique do botão, sem bloquear a interface do usuário.

## Node.js

O node.js é uma runtime para executar o javascript no backend a diferença do browser é que em vez de ter a Web Api ele tem a libuv que é onde ocorre a execução de funções assíncronas.

Se você quer visualizar essas coisas acontecendo tem esse site  [https://latentflip.com/loupe](https://latentflip.com/loupe)  onde é possível ver o código passando pelos mecanismos da runtime, ao entrar no site já existe um código de exemplo mas você pode colocar esse abaixo ou inventar algum:

```
    console.log("Hi!");
    setTimeout(function timeout() {
    console.log("5 segundos depois");
    }, 5000);
    console.log("executei antes do setTimeout");
```


**Links pesquisados:**
[https://medium.com/@gemma.croad/understanding-the-javascript-runtime-environment-4dd8f52f6fca](https://medium.com/@gemma.croad/understanding-the-javascript-runtime-environment-4dd8f52f6fca)
[https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)
[https://nodejs.org/en/docs/guides/diagnostics/memory/using-gc-traces/](https://nodejs.org/en/docs/guides/diagnostics/memory/using-gc-traces/)
[Artigo original](https://dev.to/h1bertobarbosa/o-que-e-runtime-javascript--55bd)
