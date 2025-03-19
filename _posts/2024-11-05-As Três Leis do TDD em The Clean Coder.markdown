---
layout: post
author: humberto_barbosa
title: "As Três Leis do TDD em 'The Clean Coder'"
date: 2024-11-05 07:04:14 -0300
categories: [Desenvolvimento de Software, Engenharia de Software, Metodologias ágeis, Testes e Qualidade]
tags: [Test Driven Development, TDD, Robert C. Martin, desenvolvimento de software, testes unitários, The Clean Coder]
---


Robert C. Martin, o Uncle Bob, não brinca quando fala de Test Driven Development (TDD). Ele trata como disciplina essencial para quem quer ser um desenvolvedor de verdade. O cara define três leis que tornam o TDD um processo inegociável:

1. **"Não escreva código de produção sem antes escrever um teste que falhe."**

    - Sem exceções. Antes de codar qualquer funcionalidade, escreva um teste que prove que aquilo precisa existir. Como a funcionalidade ainda não foi implementada, o teste falha. Essa falha inicial mostra que o teste realmente verifica o comportamento esperado.

2. **"Não escreva mais de um teste do que o suficiente para falhar - e não compilar já conta como falha."**

    - Sem enrolação. Escreva apenas o necessário para ver o erro acontecer. Nada de sair codando funcionalidade antes da hora.

3. **"Não escreva mais código de produção do que o necessário para passar no teste que falhou."**

    - Código mínimo para o teste passar, nada de firulas. Isso força um ciclo enxuto de implementação e garante que cada linha de código tenha um propósito.


### O Ciclo Virtuoso do TDD

Essas três leis criam um loop de desenvolvimento que dura, em média, 30 segundos. Escreve teste → vê falhar → escreve código → vê passar → refatora. Esse ciclo curto reduz o risco de surpresas desagradáveis no código.

### Benefícios do TDD

Uncle Bob defende que seguir essas regras traz uma série de vantagens:

- **Certeza**: Testes constantes garantem que as funcionalidades funcionam como esperado.

- **Coragem**: Ter uma suíte de testes permite refatorar sem medo de quebrar tudo.

- **Menos bugs**: Erros são encontrados antes de chegarem ao usuário.

- **Documentação viva**: Os testes servem como um manual de como o código deveria se comportar.

- **Design melhor**: TDD força modularidade e baixo acoplamento.


### O Maior Benefício: Certeza

Segundo Uncle Bob, o grande diferencial do TDD é a confiança que ele proporciona. Escrevendo e rodando centenas de testes por semana, um desenvolvedor sabe exatamente o que está funcionando. No projeto FitNesse, com 64 mil linhas de código e mais de 2.200 testes unitários, Uncle Bob garante 90% de cobertura. Tudo roda em 90 segundos, e se passar, ele sabe que pode lançar uma nova versão sem pânico.

Isso se traduz em:

- **Refatoração sem medo**: Melhorar o código sem receio de quebrar algo.

- **Menos defeitos no código final**: Bugs são eliminados antes de virarem problema para o usuário.


Resumindo? Se você quer escrever código que realmente funcione, sem gambiarra e sem susto, TDD não é opcional. É disciplina. Seguir esse processo transforma código ruim em código sólido. Se ainda não usa, tá na hora de repensar sua abordagem.
