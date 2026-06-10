---
title: "ADR-002: Dois tópicos Kafka"
status: accepted
date: 2026-06-08
---

# ADR-002: Dois tópicos Kafka (telemetry-events + domain-events)

## Status

Aceito

## Contexto

O pipeline recebe telemetria bruta em alto volume (~200 eventos/s no MVP, escalável para milhares). Consumidores downstream (alertas, analytics) precisam de fatos de negócio já validados, não de dados brutos. Um único tópico misturaria volume com semântica e acoplaria consumidores ao formato de telemetria.

## Decisão

Separar em dois tópicos Kafka:

| Tópico | Conteúdo | Partition key |
|--------|----------|---------------|
| `telemetry-events` | Telemetria bruta | `vehicleId` |
| `domain-events` | Fatos de negócio validados | `vehicleId` |

Publicação em `domain-events` via **outbox pattern** no `fleet-service`, garantindo consistência com o event log PostgreSQL.

## Consequências

### Positivas

- Separação clara entre ingestão (volume) e integração (valor)
- Consumidores podem evoluir independentemente do formato de telemetria
- Ordenação por veículo mantida em ambos os tópicos
- DLQs independentes por pipeline

### Negativas

- Dois tópicos para operar e monitorar
- Latência adicional entre telemetria e domain-events (processamento + outbox)
- Duplicação de dados entre PostgreSQL (event log) e Kafka (domain-events) — papéis distintos (auditoria vs integração)

## Referências

- [telemetry-events.md](../../reference/messaging/telemetry-events.md)
- [domain-events.md](../../reference/messaging/domain-events.md)
