---
title: "ADR-003: Java no backend, Node.js no simulador"
status: accepted
date: 2026-06-08
---

# ADR-003: Java/Spring Boot no backend, Node.js no simulador

## Status

Aceito

## Contexto

O CargoTrack é um projeto de demonstração de arquitetura distribuída com microsserviços, Kafka, CQRS e observabilidade. É necessário escolher linguagens para backend de produção e para o gerador de carga (simulador de frota).

## Decisão

- **Backend (4 microsserviços MVP):** Java 21 + Spring Boot 3
  - `ingestion-service`, `fleet-service`, `alert-service`, `query-api`
- **Simulador de carga:** Node.js 20+
  - Geração de telemetria em escala via REST (gRPC pós-MVP)

## Alternativas consideradas

| Alternativa | Motivo de rejeição no MVP |
|-------------|---------------------------|
| Rust no backend | Curva de aprendizado; Java cobre requisitos com ecossistema maduro |
| Elixir/Actor Model | Overhead operacional; `ConcurrentHashMap` + Kafka keyed consumer suficiente |
| Python no simulador | Node.js adequado para I/O concorrente e simplicidade de scripts |
| Monolito | Não demonstra microsserviços e EDA |

## Consequências

### Positivas

- Ecossistema Spring maduro (Kafka, Security, Actuator, JDBC)
- Simulador leve e fácil de configurar (`VEHICLE_COUNT`, `INTERVAL_MS`)
- Alinhamento com mercado enterprise Java em logística

### Negativas

- 4 serviços Java no MVP (overhead de build/deploy)
- Dois runtimes para manter (JVM + Node)

## Referências

- [stack.md](../stack.md)
- [repository-layout.md](../../development/repository-layout.md)
