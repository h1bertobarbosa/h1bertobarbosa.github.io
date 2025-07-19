---
layout: post
author: humberto_barbosa
title: "Promises em Nodejs Paralelo, Sequencial e Corrida – Qual usar?"
date: 2025-07-18 21:44:00 -0300
categories: [Desenvolvimento Backend, JavaScript Avançado, Node.js, Programação Assíncrona, Performance de Código]
tags: [Node.js, JavaScript, Promises, Async Await, Código Assíncrono, Promise.all, Promise.race, Execução Paralela, Execução Sequencial, Concorrência, Programação Assíncrona, Desenvolvimento Web, Backend, Boas Práticas]
---

# Promises em Node.js: Paralelo, Sequencial e Corrida – Como Escolher?

Em JavaScript (e especialmente no Node.js), **Promises** são uma das ferramentas mais poderosas para trabalhar com **código assíncrono** — aquele que não bloqueia o fluxo principal enquanto espera por uma resposta, como uma chamada de API ou leitura de arquivo.

Mas… quando você tem **várias Promises ao mesmo tempo**, como decidir a melhor forma de executá-las?

Neste artigo, vamos explorar os três padrões mais importantes para isso:

- **Execução Paralela**
- **Execução Sequencial**
- **Corrida (Race Condition)**

E sim, vou te mostrar com **exemplos práticos e simples** como isso funciona.

---

## Antes de tudo: o que é uma Promise?

Uma _Promise_ é um objeto que representa a eventual conclusão (ou falha) de uma operação assíncrona. Em outras palavras, ela **promete** que vai te dar um resultado.

Você pode "esperar" por esse resultado com `await`, ou então usar `.then()` e `.catch()`.

---

## 🔧 Função base para nosso exercício

Vamos criar uma função que simula uma tarefa assíncrona com um `setTimeout`. Ela simplesmente "resolve" com um valor depois de um tempo:

```javascript
function promiseWithDelay(value, delay) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`Valor: ${value}, Delay: ${delay}ms`);
    }, delay);
  });
}
```

Exemplo de uso:
```javascript
promiseWithDelay("A", 3000).then(console.log);
// → Valor: A, Delay: 3000ms (após 3 segundos)
```

---

## Execução Paralela (Parallel)

Quando queremos executar **várias Promises ao mesmo tempo** e esperar todas terminarem juntas, usamos `Promise.all()`.

### Exemplo:

```javascript
async function executarParalelo() {
  const resultados = await Promise.all([
    promiseWithDelay("A", 1000),
    promiseWithDelay("B", 5000),
    promiseWithDelay("C", 3000),
  ]);

  console.log("Execução Paralela Finalizada:", resultados);
}
```
### O que acontece aqui?

- As três Promises são iniciadas **ao mesmo tempo**
- A função espera **todas** terminarem
- O tempo total é do item **mais lento**


**Resultado:**
```bash
(espera 5 segundos)
Execução Paralela Finalizada: [
  'Valor: A, Delay: 1000ms',
  'Valor: B, Delay: 5000ms',
  'Valor: C, Delay: 3000ms'
]
```

## Execução Sequencial (Sequential)

Às vezes, você quer rodar Promises **em ordem**, uma de cada vez. Isso é útil quando o segundo depende do resultado do primeiro.

### Exemplo:

```javascript
async function executarSequencial() {
  const resultado1 = await promiseWithDelay("A", 1000);
  const resultado2 = await promiseWithDelay("B", 5000);
  const resultado3 = await promiseWithDelay("C", 3000);

  console.log("Execução Sequencial Finalizada:", [
    resultado1,
    resultado2,
    resultado3,
  ]);
}
```
### O que acontece?

- Espera o primeiro terminar (`A`)
- Só então executa o segundo (`B`)
- Depois, o terceiro (`C`)
- O tempo total = **soma dos delays** (9 segundos neste caso)

## Corrida (Race)

E se você quiser **só o primeiro resultado que chegar** e ignorar o resto? Use `Promise.race()`.

### Exemplo:
```javascript
async function executarCorrida() {
  const resultado = await Promise.race([
    promiseWithDelay("A", 1000),
    promiseWithDelay("B", 5000),
    promiseWithDelay("C", 3000),
  ]);

  console.log("Corrida Finalizada. Vencedor:", resultado);
}
```
### Resultado:
```bash
(espera 1 segundo)
Corrida Finalizada. Vencedor: Valor: A, Delay: 1000ms
```
Quem chegar primeiro, ganha. Simples assim!

## Testando os três modos

Vamos chamar os três juntos e ver o comportamento:

```javascript
async function testarTodos() {
  executarCorrida();
  executarParalelo();
  executarSequencial();
}

testarTodos();
```
Mesmo sendo chamados na mesma ordem, a **corrida** terminará primeiro, seguida da **execução paralela**, e por último a **sequencial**.


### Quando usar cada um?

| Caso real                                              | Qual usar?     |
| ------------------------------------------------------ | -------------- |
| Carregar 3 dados de API ao mesmo tempo                 | Paralelo       |
| Passo 1: Login → Passo 2: Pegar token → Passo 3: Dados | Sequencial     |
| Testar 3 URLs e usar a primeira que responder          | Corrida (Race) |
## Conclusão

Quando falamos de Promises em Node.js, **a forma como você organiza suas execuções faz toda a diferença** no desempenho e no comportamento do seu código.

Lembre-se:

- `Promise.all()` → tudo junto (paralelo)
- `await` em sequência → um por vez (sequencial)
- `Promise.race()` → quem chegar primeiro vence (corrida)

Use o tipo certo de execução para **otimizar seus fluxos assíncronos**. E se quiser explorar mais, experimente mudar os tempos de delay e veja como isso impacta a execução.
