---
layout: post
author: Humberto Barbosa
title:  "Clean Code: A Programação Eficiente"
date:   2024-08-23 08:59:14 -0300
categories: [Design de Software,General]
tags: [programming, cleancode]
---

# Clean Code: A Programação Eficiente

## Introdução

As técnicas de Clean Code foram popularizadas pelo livro “Clean Code: A Handbook of Agile Software Craftsmanship”, lançado em 2008. O autor, Robert C. Martin, percebeu que o principal gargalo no desenvolvimento de software estava na manutenção.

Um dado alarmante é que **a proporção média de leitura e escrita de códigos-fonte é de 10 para 1**, o que indica que os desenvolvedores passam mais tempo tentando entender códigos existentes do que escrevendo novos. Portanto, Clean Code se trata de aplicar técnicas que facilitam a escrita e a leitura do código.

Muitos desenvolvedores já se depararam com sistemas mal estruturados, onde a complexidade do código se torna um grande obstáculo para a manutenção. Este artigo discutirá os princípios do Clean Code e como eles podem ajudar a evitar esses problemas.

## Os Desafios da Manutenção de Código

Quando um código é desorganizado, pode causar:

-   **Gargalos no desenvolvimento**: Se apenas uma pessoa entende uma classe gigante, isso limita a capacidade do time de avançar.
-   **Custo elevado de manutenção**: O tempo gasto procurando e analisando código ruim resulta em insatisfação na equipe.
-   **Medo do deploy**: Alterações simples podem causar bugs inesperados, gerando hesitação em implementar mudanças.

A qualidade do código é frequentemente medida pela frustração que um desenvolvedor sente ao abrir um arquivo. Se a reação for de raiva, é um sinal de que a qualidade precisa ser melhorada.

### Começando com Clean Code

Uma boa maneira de iniciar a prática de Clean Code é através de pair programming, onde a colaboração entre desenvolvedores reduz interrupções e melhora a qualidade do código. Isso permite um compartilhamento de conhecimento e um nivelamento técnico dentro da equipe.
Outra maneira é o code review, é a oportunidade de melhorar o código, sugerindo melhorias como por exemplo adoção das recomendações do clean code, lembrando que o código não e da pessoa que criou a PR, o código é da empresa e pode ser que você tenha que dar manutenção nele amanha.

## Princípios do Clean Code

### Regra do Escoteiro

Inspirada em um princípio do escotismo, essa regra sugere que você deve deixar o código mais limpo do que estava antes de mexer nele.

### Regras de Design

1.  **Mantenha dados de configuração em alto nível**: Evite sobrescrever configurações em métodos dentro de Controllers.
2.  **Polimorfismo no lugar de IFs**: Utilize polimorfismo para evitar condicionais excessivas que aumentam a complexidade.
3.  **Evite configurações desnecessárias**.
4.  **Utilize injeção de dependência**: Isso torna seu código mais limpo e desacoplado.
5.  **Lei de Demeter**: Cada unidade deve ter conhecimento limitado sobre outras unidades.

### Regras sobre Entendimento do Código

1.  **Seja consistente**: Mantenha um padrão na forma de codificar.
2.  **Utilize variáveis concisas**: Opte por nomes autoexplicativos.
3.  **Obsessão primitiva**: Evite se fixar em tipos primitivos; use objetos de valor.
4.  **Evite dependências lógicas**: Não crie métodos que dependam de estados externos.
5.  **Evite condicionais negativas**: Negações podem ser confusas e difíceis de ler.

### Regras de Nomes

1.  **Escolha nomes descritivos**: Nomes bons são essenciais para um código limpo.
2.  **Faça distinções significativas**: Use nomes que diferenciem claramente os itens.
3.  **Utilize nomes pronunciáveis e buscáveis**: Nomes complicados dificultam a compreensão.
4.  **Evite uso excessivo de strings**: Utilize constantes para evitar erros de comparação.
5.  **Não use prefixo ou caracteres especiais**: Mantenha os nomes claros e simples.

### Regras para Funções ou Métodos

1.  **Pequenas e com apenas um objetivo**: Métodos menores são mais fáceis de reutilizar e entender.
2.  **Opte por poucos parâmetros**.
3.  **Cuidado com efeitos colaterais**: Métodos não devem alterar estados de outras classes.
4.  **Não tome decisões desnecessárias**: Evite "flags" que complicam o fluxo do método.

### Regras de Comentários

1.  **Um código bom é expressivo**: Comentários indicam que o código não é claro.
2.  **Não seja redundante**: Evite comentários desnecessários.
3.  **Não feche os comentários**: Não há necessidade de tags de fechamento.
4.  **Evite códigos comentados**: Utilize controle de versão em vez de deixar código comentado.

### Estrutura do Código

1.  **Separe conceitos verticalmente**: Mantenha uma estrutura de pastas organizada.
2.  **Declare variáveis próximas de seu uso**: Coloque as variáveis perto de onde são utilizadas.
3.  **Agrupe funcionalidades similares**: Mantenha métodos relacionados juntos.
4.  **Declare funções de cima para baixo**: Organize as funções de maneira lógica.
5.  **Mantenha poucas e curtas linhas**: Evite longas funções.
6.  **Não use alinhamento horizontal**: Use espaços em branco para clareza.
7.  **Use os espaços em branco corretamente**: Separe itens relacionados adequadamente.
8.  **Não quebre a indentação**.

### Testes

1.  **Um assert por teste**: Mantenha os testes claros e focados.
2.  **Legível**: Testes devem ser bem organizados e escritos.
3.  **Independentes**: Os testes não devem depender de outros testes ou de entidades externas.
4.  **Repetível**: Testes devem poder ser executados com diferentes parâmetros.

## Conclusão

Adotar os princípios do Clean Code não é apenas uma prática recomendada, mas uma necessidade para qualquer equipe de desenvolvimento que deseja manter a qualidade do código e facilitar a manutenção. Um código limpo não só melhora a produtividade, mas também contribui para um ambiente de trabalho mais satisfatório.

## Referências

-   [Clean Code - O que é?](https://balta.io/blog/clean-code)
-   [Importância do Clean Code](https://www.hostgator.com.br/blog/clean-code-o-que-e/)
-   [Princípios do Clean Code](https://www.sydle.com/br/blog/clean-code%20-602bef23da4d09680935509b/)
-   -   [Artigo original](https://dev.to/h1bertobarbosa/clean-code-para-que-serve-4gk)
