---
layout: post
author: humberto_barbosa
title: "Como os pilares da Programação Orientada a Objetos (POO) se conectam ao SOLID"
date: 2025-04-07 11:05:08 -0300
categories: [Fundamentos da POO, Princípios SOLID, Design de Software, Arquitetura e Manutenção]
tags: [Programação Orientada a Objetos, POO, SOLID, Abstração, Encapsulamento, Herança, Polimorfismo, Princípios SOLID, SRP, OCP, LSP, ISP, DIP, Boas práticas de código, Design de software, Clean Code, Engenharia de Software]
---



### **1. Abstração**

Foca no comportamento essencial isolando as características essenciais de um objeto, ignorando os detalhes de implementação.

A abstração é um dos pilares da Programação Orientada a Objetos (POO) que ajuda a reduzir a complexidade no desenvolvimento de software. Ela permite que os programadores representem entidades do mundo real de maneira simplificada, focando apenas nos aspectos essenciais para o sistema.

Isso permite que seu código de negócio **não conheça** como o pagamento será salvo: pode ser banco, arquivo, API.

### **2. Encapsulamento**

Protege o estado interno do objeto ermite ocultando os detalhes internos de implementação e definindo uma interface de uso externo segura.

O encapsulamento é um princípio fundamental da Programação Orientada a Objetos que consiste em **ocultar os detalhes internos de um objeto**, expondo apenas o que é necessário para sua interação.

### **3. Herança**

Permite reuso de comportamento e especialização por meio de classes derivadas.

A herança é um dos pilares fundamentais da Programação Orientada a Objetos que permite **estender funcionalidades** de uma classe já existente. Com ela, é possível criar novas classes (chamadas de **subclasses** ou **classes filhas**) que herdam características (atributos e métodos) de uma **classe base** (superclasse ou classe pai).

### **4. Polimorfismo**

Permite que objetos de diferentes classes possam **responder de maneira diferente a uma mesma chamada de método**, mantendo uma interface comum.

O polimorfismo é um conceito essencial da POO que permite que **um mesmo método se comporte de maneiras diferentes**, dependendo do objeto que o executa. Ele dá flexibilidade ao código, permitindo que classes distintas respondam à mesma ação de formas específicas.

## Os 5 Princípios SOLID

### **1. Single Responsibility Principle (SRP) – Princípio da Responsabilidade Única**

Reúna as coisas que mudam pelos mesmos motivos. Separe as coisas que mudam por motivos diferentes.

Evite misturar regras de negócio com a camada de interface gráfica. Também separamos comandos SQL de protocolos de comunicação. Fazemos isso porque queremos manter isolado o código que muda por motivos diferentes, evitando que alterações em uma parte afetem outras de forma indesejada. Assim, garantimos que módulos que evoluem por razões distintas não fiquem entrelaçados por dependências desnecessárias.

Se uma classe faz muitas coisas diferentes, ela fica difícil de entender e modificar. Separando as responsabilidades, deixamos o código **mais organizado e modular**.


---

### **2. Open/Closed Principle (OCP) – Princípio Aberto/Fechado**

O código deve estar **aberto para extensão, mas fechado para modificação**. Isso significa que devemos ser capazes de **adicionar novas funcionalidades sem alterar o código existente**.

Isso quer dizer que devemos usar interfaces ou classes abstratas para definir o comportamento, e usando polimorfismo esse comportamento pode variar em tempo de execução.


---

### **3. Liskov Substitution Principle (LSP) – Princípio da Substituição de Liskov**

**Subtipos devem ser substituíveis por seus tipos base** sem quebrar a aplicação.

Se uma subclasse altera o comportamento esperado da classe pai, podemos ter problemas imprevisíveis no código.
Existem 3 regras que devem ser seguidas pelas subclasses:

- **Pré-condições não devem ser fortalecidas na subclasse**: Pré-condições representam os requisitos necessários para que um método possa ser executado corretamente, os parâmetros esperados. Por exemplo, se um método aceita um número (positivo ou negativo), uma subclasse **não deve restringir** esse comportamento para aceitar apenas números positivos. Fazer isso **fortalece a pré-condição**, e pode quebrar a lógica do programa, especialmente em casos onde se espera que o método aceite números negativos.
- **Pós-condições não devem ser enfraquecidas na subclasse**: As pós-condições definem o que o método retorna após sua execução. Uma subclasse **não deve retornar menos do que o esperado**. Por exemplo, se o contrato do método exige um valor booleano, a subclasse deve retornar um `true` ou `false`. Embora seja possível "fortalecer" o contrato (por exemplo, retornando sempre `true`), retornar **outros tipos como `string`, `number`, `null`**, ou lançar exceções inesperadas, **quebra a expectativa** da aplicação — especialmente em linguagens dinamicamente tipadas.
- **A invariância deve ser preservada na subclasse**: Invariância se refere ao estado interno do objeto, que deve ser protegido e controlado por meio de encapsulamento. Subclasses **não devem violar esse contrato**, modificando diretamente atributos da superclasse ou alterando sua lógica de maneira inconsistente. É essencial respeitar os **modificadores de visibilidade** (como `private` e `protected`) e manter o **comportamento coerente** em toda a hierarquia de classes.

---

### **4. Interface Segregation Principle (ISP) – Princípio da Segregação de Interface**

Nenhuma classe deve ser forçada a depender de **métodos que não usa**.

É melhor ter **múltiplas interfaces específicas** do que uma interface gigante cheia de métodos inúteis.

#### Por que isso importa? (segundo Uncle bob)

Mesmo hoje, trabalhando com linguagens compiladas, ainda somos afetados por problemas relacionados à **recompilação** e **reimplantação** desnecessária. Imagine o seguinte cenário:

- O **Módulo A** depende de uma interface do **Módulo B** apenas para um subconjunto de seus métodos.
- Se outro método dessa interface for alterado, **mesmo que o Módulo A não o utilize**, ele precisará ser recompilado e reimplantado.


Esse acoplamento em tempo de compilação, mesmo sem dependência em tempo de execução, é um **problema real** — especialmente em linguagens **estaticamente tipadas** como **Java, C#, C++, Go, Swift** etc.


---

### **5. Dependency Inversion Principle (DIP) – Princípio da Inversão de Dependência**

**Dependa de abstrações, não de implementações concretas, ou seja abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.**

Isso significa que implementando esse princípio voce vai **quebrar a dependência direta entre regras de negócio (nível alto)** e **detalhes de implementação (nível baixo)**. Ele propõe que as dependências devem sempre apontar para **abstrações**, nunca diretamente para implementações concretas.
A injeção de dependências resolve o vínculo entre os dois em tempo de execução.


---

## RELAÇÃO: PILARES da POO → PRINCÍPIOS SOLID

| Pilar da POO       | Conexões com Princípios SOLID      | Como se Conectam                                                                                                                                                                                          |
| ------------------ | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Abstração**      | **OCP**, **DIP**, **ISP**, **SRP** | Permite **ocultar complexidade**, definindo **interfaces** que isolam o “o quê” do “como”. Facilita o desacoplamento entre módulos. Base para **Dependency Inversion** e **Interface Segregation**.       |
| **Encapsulamento** | **SRP**, **LSP**, **OCP**          | Protege o estado interno dos objetos, **separando responsabilidades**. Ajuda a manter **coesão** e **evitar que alterações em um detalhe impactem em outros**.                                            |
| **Herança**        | **OCP**, **LSP**                   | Permite a extensão de comportamento sem modificação (essência do **Open/Closed**). Deve ser usada com cautela para não violar substituição (**Liskov**).                                                  |
| **Polimorfismo**   | **OCP**, **DIP**, **LSP**, **ISP** | Permite que objetos de diferentes tipos sejam tratados de forma uniforme via interfaces. Viabiliza **inversão de dependência**, **extensão sem modificação** e **substituição segura** de implementações. |

## Código tem que ser simples

Simplicidade **não é ausência de estrutura** ou codar com falta de boas praticas e principios

> Dizer apenas "Código tem que ser simples" sem oferecer princípios claros é como ser um surdo em bingo.

Com o entendimento básico dos fundamentos de POO fica mais facil entender os principios SOLID

### Links

[https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)
[https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)
[https://www.branas.io/blog/solid-lsp-liskov-substitution-principle.htmlpo](https://www.branas.io/blog/solid-lsp-liskov-substitution-principle.htmlpo)
[https://hub.asimov.academy/blog/programacao-orientada-a-objetos-conceito-e-pilares/](https://hub.asimov.academy/blog/programacao-orientada-a-objetos-conceito-e-pilares/)
