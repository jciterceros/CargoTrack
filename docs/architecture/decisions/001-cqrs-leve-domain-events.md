---
title: "ADR-001: CQRS leve com domain-events"
status: accepted
date: 2026-06-08
---

# ADR-001: CQRS leve com domain-events

## Status

Aceito

## Contexto

O CargoTrack precisa processar telemetria de 1.000+ veículos com baixa latência para consultas de frota, alertas em tempo real e auditoria de eventos. Event Sourcing completo adiciona complexidade significativa no MVP (rebuild de read models, replay, versionamento de eventos).

## Decisão

Adotar **CQRS leve** com:

- **Write side** (`fleet-service`): grava event log append-only no PostgreSQL, atualiza read models (Redis + TimescaleDB) de forma **síncrona**, e publica `domain-events` via **outbox pattern**.
- **Read side** (`query-api`): consulta apenas Redis e TimescaleDB — nunca escreve no event log.
- **`domain-events`**: bus de fatos de negócio para consumidores desacoplados (alertas, analytics futuros).

## Consequências

### Positivas

- MVP mais simples e rápido de entregar
- Consultas de frota com latência baixa (Redis quente)
- Alertas desacoplados via Kafka sem acoplar ao processamento de telemetria
- Event log preservado para auditoria

### Negativas

- Redis não é derivado exclusivamente de replay de eventos
- Inconsistência teórica entre event log e read models em falhas parciais (mitigado por transação no fleet)
- Evolução para Event Sourcing completo exigirá `projection-service` dedicado

## Referências

- [system-design.md](../system-design.md) — seção CQRS leve vs Event Sourcing
- [event-catalog.md](../../domain/event-catalog.md)
