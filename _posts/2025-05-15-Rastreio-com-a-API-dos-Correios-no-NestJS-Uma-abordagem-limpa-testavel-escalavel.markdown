---
layout: post
author: humberto_barbosa
title: "Integração Correios para Rastreamento de Entregas: Arquitetura Escalável com NestJS e Kafka"
date: 2025-05-15 09:00:00 -0300
categories: [Integração com APIs externas, Arquitetura de Software, Engenharia de Software, Desenvolvimento Backend, Boas Práticas de Programação, Testes de Software, Mensageria e Event-Driven Architecture, Observabilidade e Monitoramento, Soluções para E-commerce, Automação de Processos Logísticos, Microsserviços e Escalabilidade, Design Patterns, Clean Code]
tags: [e-commerce, logística, rastreamento de pedidos, Correios, API Gateway, NestJS, Clean Architecture, Kafka, mensageria, status de entrega, integração de sistemas, testes unitários, jest-mock-extended, resiliência, logging, desacoplamento, injeção de dependência, arquitetura hexagonal, microsserviços, eventos, orquestração, transportadoras, fallback, monitoramento, manutenção de software, boas práticas, automatização, reprocessamento, mock, testes automatizados]
---

### Introdução

Nos sistemas de e-commerce e da logística, rastrear o status de entregas é essencial para garantir transparência e agilidade no atendimento ao cliente. Automatizar esse processo com integração direta aos Correios é um desafio que envolve lidar com APIs externas, mapeamento de status inconsistentes e atualização em tempo real.

Neste artigo, vamos apresentar uma abordagem completa para orquestração de rastreamento de pedidos usando NestJS, aplicando os princípios de Clean Architecture, testes unitários robustos e integração com mensageria (Kafka).

### Desenho da Solução

A solução gira em torno do caso de uso `TrackingOrderStatusService`, responsável por:

- Consultar configurações de contas com rastreamento ativo;
- Buscar os pedidos com status finalizados que devem ser atualizados;
- Integrar com a API dos Correios via `ProviderTrackingApiGateway`;
- Filtrar eventos novos não registrados;
- Mapear os eventos para um modelo interno de status;
- Persistir os novos eventos no repositório;
- Emitir mensagens Kafka sobre mudanças de status;
- Registrar novos status desconhecidos para posterior mapeamento.

### Pontos de Complexidade

**1. Identificação de Status**

A API dos Correios utiliza uma combinação de `codigo` e `tipo` para representar eventos. Essa combinação foi padronizada com o helper:

```typescript
export function buildPartnerStatusId(codigo: string, tipo: string): string {
  return `${codigo}${tipo}`;
}
```

Isso garante que qualquer ponto do sistema que precise identificar unicamente um status tenha uma forma unificada, reduzindo erros e facilitando o mapeamento.

**2. Detecção de Eventos Novos**

Para evitar a reprocessamento de eventos já persistidos, usamos o método `getNewHistories`. Ele compara os eventos vindos dos Correios com os eventos já salvos no histórico:

```typescript
private getNewHistories(tracking: TrackingData[], eventos: CorreiosEventoDTO[]): CorreiosEventoDTO[] {
  return eventos.filter(evento => !this.historyExists(evento, tracking));
}
```

E o `historyExists` compara tanto o código concatenado quanto a data:

```typescript
private historyExists(evento: CorreiosEventoDTO, tracking: TrackingData[]): boolean {
  return tracking.some(hist => {
    return (
      hist.partner.statusCode === buildPartnerStatusId(evento.codigo, evento.tipo) &&
      compareDates(hist.eventDate, evento.dtHrCriado)
    );
  });
}
```

**3. Fallback para Status Não Mapeados**

Se um evento dos Correios não tem status mapeado, salvamos esse status para posterior análise e notificamos por Kafka:

```typescript
if (!statusMapped) {
  const newStatusPartner = new StatusCodePartner({ statusId, status });
  await this.statusCodePartnerRepository.save(newStatusPartner);

  await this.messageProvider.sendMessage({
    topic: KAFKA_TOPIC_FREIGHT_STATUS_CODE_PARTNER_NOT_FOUND,
    messages: [{
      key: JSON.stringify({ id: newStatusPartner.id }),
      value: { data: { ... } },
    }],
    headers: { 'X-Tenant-Id': accountId, 'X-Correlation-Id': randomUUID() },
  });
}
```

**4. Resiliência e Logging**

Todas as etapas do processo possuem logging contextualizado:

```typescript
this.logger.log({
  id: 'TrackingOrderStatusService.trackingOrders',
  message: `[${tracking.accountId}|${tracking.order.internalOrderId}] Evento processado com sucesso.`,
});
```

Além disso, exceções são capturadas e logadas para facilitar troubleshooting:

```typescript
catch (error) {
  this.logger.error({
    id: 'TrackingOrderStatusService.execute',
    message: error.message,
    stack: error.stack,
  });
}
```

### Abstração via API Gateway

Outro destaque importante da implementação é o uso do `ProviderTrackingApiGateway`, uma interface que abstrai o consumo da API dos Correios. Ao invés de depender diretamente de chamadas HTTP acopladas no serviço, a integração é feita via uma porta definida por contrato:

```
export interface ProviderTrackingApiGateway {
  getTracking(etiqueta: string, cep: string): Promise<CorreiosApiResponseTrackingStatus>;
}
```

Essa abordagem segue o princípio da Inversão de Dependência (DIP), permitindo:

- **Substituir facilmente a implementação por mocks em testes unitários:**

```
correiosApiGateway = mock<ProviderTrackingApiGateway>();
correiosApiGateway.getTracking.mockResolvedValue(mockCorreiosTrackingResponse);
```

- **Trocar facilmente a origem do rastreamento (ex: transportadoras privadas)** sem alterar o caso de uso

- **Reduzir acoplamento e facilitar manutenção**, pois a lógica de orquestração não precisa conhecer detalhes de implementação HTTP, headers ou autenticação


Essa separação clara entre camada de aplicação e infraestrutura melhora a testabilidade e prepara o código para cenários mais complexos.

### Boas Práticas Adotadas

- Inversão de dependência via tokens de injeção;
- Separacão de responsabilidades (cada provider tem uma única função);
- Logging com identificadores contextuais (`[accountId|orderId]`);
- Utilização de `Promise.all` para operações paralelas (status + configuração);
- Mensagens Kafka emitidas apenas para eventos novos.

### Testes Unitários

#### 🧪 Exemplo de Teste com Gateway Mockado

Abaixo está um exemplo real de como o `ProviderTrackingApiGateway` pode ser simulado em testes unitários usando `jest-mock-extended`:

```typescript
import { mock, MockProxy } from 'jest-mock-extended';
import { ProviderTrackingApiGateway } from '../../correios/interfaces/provider-tracking-api-gateway';
import { CorreiosApiResponseTrackingStatus } from '../../correios/interfaces/provider-tracking-api-gateway';

describe('TrackingOrderStatusService – Integração com Correios API', () => {
  let correiosApiGateway: MockProxy<ProviderTrackingApiGateway>;

  beforeEach(() => {
    correiosApiGateway = mock<ProviderTrackingApiGateway>();
  });

  it('deve retornar eventos simulados dos Correios via gateway mockado', async () => {
    const mockCorreiosResponse: CorreiosApiResponseTrackingStatus = {
      objetos: [
        {
          codObjeto: 'XYZ123456BR',
          dtPrevista: new Date('2025-04-25T18:00:00Z'),
          modalidade: 'F',
          peso: 1.2,
          formato: 'Pacote',
          eventos: [
            {
              codigo: 'BDE',
              tipo: '01',
              dtHrCriado: new Date('2025-04-21T14:00:00Z'),
              descricao: 'Objeto entregue ao destinatário',
            },
          ],
        },
      ],
    };

    correiosApiGateway.getTracking.mockResolvedValue(mockCorreiosResponse);

    const result = await correiosApiGateway.getTracking('XYZ123456BR', '12345');

    expect(result.objetos[0].codObjeto).toBe('XYZ123456BR');
    expect(result.objetos[0].eventos[0].codigo).toBe('BDE');
  });
});
```

Esse mock garante que o `ApiGateway` esteja isolado e testável sem fazer chamadas reais, permitindo simular qualquer tipo de retorno da API externa.

Para garantir a qualidade e a confiabilidade da lógica, foram adicionados testes com `jest-mock-extended` para:

- `getAllStatusMapped`: mapeamento correto e fallback para status desconhecidos;
- `getNewHistories`: detecção de eventos novos;
- `historyExists`: comparação exata por status e data;
- `trackingOrders` e `execute`: cenários completos de sucesso e falha.

### Lições Aprendidas

- Pequenos helpers como `buildPartnerStatusId` e `compareDates` reduzem bugs e aumentam reutilização;
- Separar persistência de envio em `sendMessageStatusNotFound` tornou o código mais testável e legível;
- Centralizar a orquestração no caso de uso facilita extensão para outras transportadoras;
- Cobertura de testes auxilia na manutenção sem medo de regressão.

### Conclusão

A solução apresentada mostra como é possível construir uma integração robusta com os Correios utilizando boas práticas de arquitetura, testes e logging. Além de resolver o problema imediato de rastreamento, essa abordagem serve como base para integração com outros parceiros logísticos.

Essa implementação está pronta para produção, observável e fácil de evoluir.
