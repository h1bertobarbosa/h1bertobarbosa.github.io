---
layout: post
author: humberto_barbosa
title:  "O que é Arquitetura de Software?"
date:   2024-08-01 11:28:14 -0300
categories: [Design de Software, General]
tags: [software, clean-arch]
---

## O que é arquitetura de software?

Que raio é isso? O que isso faz? Vamos começar com algumas definições do Robert Martin (Uncle Bob) onde ele diz que:

> The goal of software architecture is to minimize the human resources required to build and maintain the required system
> Ele disse aqui que:
> O objetivo da arquitetura de software é minimizar os recursos humanos necessários para construir e manter o sistema

Tem outra do Martin Fowler:

> Software architecture is those decisions which are both important and hard to change
> A arquitetura do software são as decisões importantes e difíceis de mudar

A arquitetura de software tem a ver com organização, estrutura, design, relacionamento e comportamento.
Você já se deparou com a situação de escrever um software do zero, estava na empresa com um sistema legado e o chefe autorizou fazer outro, deu carta branca. Tudo que foi decidido ali, as tecnologias, servidores, frameworks etc., são as decisões importantes e difíceis de mudar e vão acompanhar o time ou quem decidiu pelo cotidiano enquanto o software estiver no ar.
Essas decisões são afetadas pelo escopo do produto, valor que a empresa tem para gastar, seja com equipe, servidor etc., volume de requisições, se a equipe é em sua maioria sênior, pleno, júnior, estagiários... qual o prazo de entrega, é preciso levar em consideração todos os requerimentos.
Isso significa que, por exemplo, se a empresa tem uma dúzia de clientes e fizerem um sistema distribuído, com Kafka, Kubernetes, banco de dados replicados etc., será que foi a melhor decisão? Olhando para a quantidade de clientes e para o gasto com servidores.
Cada tipo de software deve ter uma arquitetura diferente, um ERP é diferente de um jogo de RPG online, uma plataforma de streaming é diferente de um programa linha de comando e por aí vai.
Existem muitos tipos de arquiteturas: cliente-servidor, monolítica, distribuída, orientada a serviço, orientada a eventos, serverless... cada uma atende a um determinado tipo de sistema de acordo com seus requerimentos.
Não é porque você usou clean arch em um sistema que você precisa replicar isso para todos, e se você tiver um sistema que só faz cadastro do jeito que vem do browser, tipo uma newsletter, você vai implementar todo conceito de clean arch, ou arquitetura hexagonal ou MVC só para dar um save no nome e email?

## O que é Clean Architecture

![Clean Architecture](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h01jph0qnxgth5sdlnfj.png)

É um modelo arquitetural orientado ao desacoplamento entre as regras de negócio da aplicação e os recursos externos como banco de dados, frameworks etc.
Então, por exemplo, o seu ORM vai ficar isolado em uma camada fazendo o papel dele que são as operações de banco de dados, salvar, atualizar, trazer dados etc., a entidade do ORM que geralmente representa uma tabela no banco de dados é diferente da entidade que fica na camada entity (domínio) que é onde estão as regras de negócios.

## Como surgiu a Clean Architecture?

O Bob Martin estava desenvolvendo esse [software](https://docs.fitnesse.org/FrontPage) inclusive com o filho dele, que é um software onde você cria testes baseado numa wiki, onde você coloca os dados de entrada e saída e o programa dele tem uma engine que interpreta os dados e faz o bind com o algoritmo do teste.
No decorrer do projeto ele percebeu que quando olhava para a estrutura de pasta enxergava só o Rails, e ele não queria ver o Rails, ele queria olhar para a estrutura do projeto e enxergar o objetivo da aplicação (a estrutura não é só de pastas), ou seja, a aplicação seguia os conceitos do Rails. Então foi por isso que ele começou a perceber que estava dependente do framework, do ORM, das libs e começou a desenvolver o projeto isolando as regras de negócio, com testes e as camadas de caso de uso e entidades, deixando as outras coisas como ORM, libs externas para depois. Dessa maneira, ele focou no core do sistema apenas com testes.
Ele formulou a teoria de que o centro da sua aplicação não é o banco de dados, e não é um ou mais frameworks que você pode estar usando. O centro da sua aplicação é o caso de uso da sua aplicação.

## Use Cases

> The center of your application is not the database, nor is it one or more of the frameworks you may be using. The center of your application is the use cases of your application.

Caso de uso pode ser comparado ao caso de uso da UML, onde você tem a análise das intenções do usuário, o que ele pretende fazer no sistema e então é mapeado os fluxos do que um determinado ator (usuário ou sistema) pode fazer, ex: fazer check-in, adicionar itens no carrinho etc.
O centro da sua aplicação deve ser esses casos de uso.
Mas até hoje é comum nas empresas, quando vão arquitetar uma funcionalidade, o time já pensa primeiro como será a tabela no banco, é uma quebra de paradigmas pensar no caso de uso.
Os casos de uso contêm a aplicação das regras de negócios, é aqui que você vai chamar as entidades.
Eventualmente você estará codando o seu caso de uso e perceberá que precisa usar uma regra que está contida em outro caso de uso, e vai vir o questionamento: será que eu posso usar um caso de uso dentro do outro? De forma bem direta, não (opinião minha). Quando isso acontecer é porque você encontrou uma regra de negócio independente (genérica) e ela está na camada errada, agora entramos na camada de entities.

```typescript
class Pedido {
  id: number;
  itens: ItemPedido[];
  cliente: Cliente;
}

interface EmailProvider {
  enviarEmail(to: string, emailInfo: T): Promise<void>
}

class EmitirFatura {
  constructor(private emailProvider: EmailProvider) {}

  async execute(pedido: Pedido): Promise<void> {
    const vencimento30dias = new Date(new Date().getTime() + 30 * 24 * 60 * 60 * 1000);
    const itensFatura = pedido.itens.map(item => new ItemFatura(item.produto.nome, item.quantidade, item.produto.preco));
    const fatura = new Fatura(
      1,
      new Date(),
      vencimento30dias,
      pedido.cliente.nome,
      itensFatura
    );

    // Salvar a fatura no banco de dados
    await salvarFatura(fatura);

    // Enviar e-mail
    this.emailProvider.enviarEmail(pedido.cliente.email, {
      vencimento: fatura.dataVencimento,
      valorTotal: fatura.valorTotal()
    });
  }
}
```
## Entities

Uma entidade é uma representação de algo do negócio que possui uma identidade única, distinta, com ciclo de vida definido. Na prática, uma classe Fatura, por exemplo, seria uma entidade. A entidade precisa estar sempre consistente, isso significa que não pode ter um monte de propriedades públicas que podem ser atribuídas, é necessário preservar a invariância do objeto.
A invariância é o estado interno da entidade e só pode ser alterada mediante as regras de negócio que são executadas a partir da ação de um usuário. No teste de unidade, aquilo que agente coloca em uma asserção é a invariância do objeto, a falha de uma asserção significa que o estado atual de uma operação é anormal.
Essa é a camada onde você vai modelar as regras de negócio independentes e que podem ser aplicadas em qualquer contexto, ou seja, são as regras (códigos) que eventualmente você estará usando em mais de um caso de uso. Essas entidades não se confundem com entidades dos ORM, são coisas completamente diferentes, no caso as entidades do ORM são classes que abstraem as tabelas do banco de dados para criar um mapeamento de dados, são um data mappers: coluna X para atributo Y.
O que aconteceria se você adicionasse dentro das entidades de regras de negócio funções do seu ORM, por exemplo, decorators, annotations etc.
Além de quebrar o princípio de single responsibility e open-closed, você ainda estaria gerando um alto acoplamento entre regras de negócios com sua infraestrutura a partir de uma lib externa de terceiro que é seu ORM. Consegue perceber que dessa forma uma mudança lá na sua tabela, uma coluna a mais ou a menos já impactaria nas suas regras de negócio?
Uma entidade deve ser uma classe? Não necessariamente, pode ser um arquivo cheio de funções, isso depende da maneira que você e seu time estão codificando, mas se for uma classe acredito que fica mais fácil até para seguir os princípios e padrões de orientação a objetos.
Exemplo de uma entidade:
```typescript
class ItemFatura {
 private descricao: string;
 private quantidade: number;
 private valorUnitario: number;

constructor(descricao: string, quantidade: number, valorUnitario: number) {
 this.descricao = descricao;
 this.quantidade = quantidade;
 this.valorUnitario = isNaN(valorUnitario) ? 0 : valorUnitario; // Garantindo que valorUnitario seja um número
 this.valorUnitario = valorUnitario;
 }

get valorTotalItem(): number {
 return this.quantidade * this.valorUnitario;
 }
}

class Fatura {
 private id: number;
 private dataEmissao: Date;
 private dataVencimento: Date;
 private cliente: string;
 private itens: ItemFatura[];

constructor(
 id: number,
 dataEmissao: Date,
 dataVencimento: Date,
 cliente: string,
 itens: ItemFatura[]
 ) {
 this.id = id;
 this.dataEmissao = dataEmissao;
 this.dataVencimento = dataVencimento;
 this.cliente = cliente;
 this.itens = itens;
 }

get valorTotal(): number {
 return this.itens.reduce((total, item) => total + item.valorTotalItem, 0);
 }

estaVencida(): boolean {
 const hoje = new Date();
 return this.dataVencimento < hoje;
 }

// Outros métodos...
}
```
Este é apenas um exemplo básico. A complexidade da entidade Fatura pode variar dependendo das necessidades do seu sistema. Você pode adicionar mais atributos, métodos e validações necessárias para garantir a integridade da entidade para que as regras sejam aplicadas.

## Interface adapters

Essa camada faz a conexão entre um dispositivo de I/O e os casos de uso, essa camada faz essa conversão de dados entre a infra (externo) com seus casos de uso.
Aqui está toda a arquitetura MVC, controllers, middlewares, presenters etc.
Aqui que recebemos as requisições do usuário, adaptamos, colocamos dentro do DTO e passamos para o caso de uso. O caso de uso que orquestra as entidades, e depois recebemos um DTO de saída, adaptamos e enviamos para a view.
Essa camada não conhece banco de dados, programa de mensageria etc, abaixo um exemplo de como isso acontece supondo que voce esteja fazendo o encrypt da senha do usuario:
```typescript
export interface HashingProvider {
  hash: (value: string) => Promise<string>;
  isMatch: (value: string, hash: string) => Promise<boolean>;
}

// a implementacao o adaptador

import * as bcrypt from 'bcrypt';
import { HashingProvider } from './hashing-provider.interface';

export class BcryptAdapterHashingProvider implements HashingProvider {
  #saltOrRounds = 10;
  async hash(value: string): Promise<string> {
    return bcrypt.hash(value, this.#saltOrRounds);
  }

  async isMatch(value: string, hash: string): Promise<boolean> {
    return bcrypt.compare(value, hash);
  }
}
```
Dessa forma, você consegue criar vários adaptadores com implementações com libs diferentes sem afetar o funcionamento do sistema, atendendo ao princípio single responsability,Open-closed, Liskov Substituition. As camadas de caso de uso farão uso da interface, não sabendo nada da implementação
## Frameworks and drivers

Essa é a camada mais externa onde está a infra do projeto como banco de dados, Redis, Nginx, protocolos, Kafka etc.
Mantemos essas coisas do lado de fora, onde podem causar poucos danos.

## Conclusão

O que define as camadas? As camadas são definidas logicamente, não confunda com criação de pastas. Você pode jogar todos os arquivos na raiz do projeto e ainda assim terá as camadas. O que vem a ser essa definição lógica? Respeitar a dependência, os arquivos das camadas mais internas não podem depender de quem está nas camadas externas. Dizendo de outra forma, quem está fora pode depender de quem está dentro. A direção de dependência é de fora para dentro.

**Vantagens de usar:**

- Isolam as regras de negócio
- Definem camadas e suas responsabilidades
- Tem fluxo de dependência ordenado e direcional
- Favorecem testabilidade
- São independentes de recursos externos
- Facilitam a evolução tecnológica

**Links:**

- [Clean Coder Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Construindo Sistemas com uma Arquitetura Limpa – Engenharia de Software Moderna](https://engsoftmoderna.info/artigos/arquitetura-limpa.html)
- [10 perguntas sobre Clean Architecture - YouTube](https://www.youtube.com/watch?v=NYoW-ycqbYk)
- [PROGRAMAÇÃO ORIENTADA A OBJETOS - CONCEITOS BÁSICOS](https://goulart.pro.br/ori_obj.html)
- [clean-architecture dev.to](https://dev.to/h1bertobarbosa/clean-architecture-53lj) - este link e o artigo original que eu escrevi no dev.to
