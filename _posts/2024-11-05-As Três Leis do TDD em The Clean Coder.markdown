---
layout: post
author: humberto_barbosa
title: "As Três Leis do TDD em 'The Clean Coder'"
date: 2024-11-05 07:04:14 -0300
categories: [Desenvolvimento de Software, Engenharia de Software, Metodologias ágeis, Testes e Qualidade]
tags: [Test Driven Development, TDD, Robert C. Martin, desenvolvimento de software, testes unitários, The Clean Coder]
---


Robert C. Martin, em seu livro "The Clean Coder", defende o Test Driven Development (TDD) como uma disciplina crucial para o desenvolvimento profissional de software. Ele apresenta as três leis do TDD como pilares dessa prática:

1. "Você não tem permissão para escrever qualquer código de produção até que tenha primeiro escrito um teste de unidade que falhe."

2. Essa primeira lei garante que o desenvolvimento seja guiado pelos testes. Antes de escrever qualquer código que implemente uma funcionalidade, o desenvolvedor deve primeiro escrever um teste que verifique se essa funcionalidade se comporta como esperado. No entanto, como a funcionalidade ainda não existe, o teste inevitavelmente falhará. Essa falha inicial é crucial, pois demonstra que o teste está realmente verificando a funcionalidade que ainda será implementada.

3. "Você não tem permissão para escrever mais de um teste de unidade do que o suficiente para falhar - e não compilar é falhar."

4. A segunda lei promove a escrita de testes incrementais e focados. O desenvolvedor deve escrever apenas o código de teste necessário para que o teste falhe, sem adicionar código desnecessário. A falha pode ocorrer por diversos motivos, incluindo a falta de compilação do código. Essa lei incentiva a escrita de testes concisos e evita a implementação prematura de código de produção.

5. "Você não tem permissão para escrever mais código de produção do que o suficiente para passar no teste de unidade que está falhando."

6. A terceira lei assegura que o código de produção seja escrito de forma iterativa e orientada para a aprovação nos testes. O desenvolvedor deve implementar apenas o código necessário para fazer o teste, que estava falhando, passar. Essa abordagem evita a escrita de código desnecessário e garante que a implementação esteja alinhada com os requisitos especificados nos testes.

## O Ciclo Virtuoso do TDD

As três leis do TDD, segundo Robert C. Martin, estabelecem um ciclo de desenvolvimento curto e iterativo, com cerca de 30 segundos de duração. O desenvolvedor alterna entre a escrita de testes e a implementação de código de produção, com o objetivo de fazer os testes passarem.

### Benefícios do TDD

Robert C. Martin argumenta que a prática do TDD, seguindo as três leis, proporciona diversos benefícios:

● Certeza: Os testes fornecem um feedback constante sobre o funcionamento do código, garantindo que as funcionalidades estejam sendo implementadas corretamente.

● Coragem: Com uma suíte de testes abrangente, o desenvolvedor se sente mais confiante para refatorar o código e adicionar novas funcionalidades.

● Redução de Defeitos: A detecção precoce de erros por meio de testes minimiza a quantidade de bugs no código de produção.

● Documentação: Os testes servem como uma forma de documentação do código, pois descrevem o comportamento esperado das funcionalidades.

● Melhor Design: O TDD incentiva a modularidade e o baixo acoplamento do código, resultando em um design mais limpo e manutenível.

Vale ressaltar que as informações acima são provenientes exclusivamente das fontes fornecidas.

## O principal benefício do Test Driven Development (TDD)

De acordo com Robert C. Martin em "The Clean Coder", o principal benefício do Test Driven Development (TDD) é a certeza. O autor argumenta que a prática do TDD, seguindo as três leis, permite que o desenvolvedor escreva dezenas de testes por dia, centenas por semana e milhares por ano. A execução constante dessa suíte de testes a cada modificação no código garante ao desenvolvedor a certeza de que suas alterações não introduziram novos erros no sistema.

Robert C. Martin utiliza o exemplo do projeto FitNesse, do qual é o autor principal, para ilustrar esse benefício. O projeto, que contém 64.000 linhas de código, possui mais de 2.200 testes unitários que cobrem pelo menos 90% do código de produção. A execução desses testes leva cerca de 90 segundos, e a aprovação em todos eles garante a Robert C. Martin a confiança necessária para liberar novas versões do software.

Essa certeza, proporcionada pela prática do TDD, se traduz em outros benefícios, como:

● Aumento da coragem para refatorar o código: A confiança de que os testes detectarão qualquer erro introduzido durante a refatoração permite que o desenvolvedor melhore o código com mais frequência e sem medo de quebrar funcionalidades existentes.

● Redução da taxa de injeção de defeitos: A detecção precoce de erros através dos testes minimiza a quantidade de bugs que chegam ao código de produção.

Em suma, a certeza proporcionada pelo TDD, de acordo com Robert C. Martin, é a base para o desenvolvimento de um código mais limpo, confiante e com menos defeitos.
