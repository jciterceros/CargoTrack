---
title: Stack Tecnológica
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# Stack Tecnológica — CargoTrack

**Backend:** Java/Spring Boot · **Simulador:** Node.js · **Padrão:** CQRS leve + `domain-events`

---

## Resumo

| Componente | Tecnologia | Papel |
|------------|------------|-------|
| Simulador | **Node.js** | POST JSON → ingestion |
| Ingestion | **Java / Spring Boot** | REST → `telemetry-events` |
| Fleet | **Java / Spring Boot** | Write side — event log + projeções + outbox |
| Alert | **Java / Spring Boot** | Consumer `domain-events` |
| Query API | **Java / Spring Boot** | Read side — REST + WebSocket |
| Kafka | `telemetry-events` + `domain-events` | Barramento |
| Event log | PostgreSQL (`domain_events`) | Auditoria append-only |
| Read models | Redis + TimescaleDB | Consultas |
| Projection service | Java | **Pós-MVP** — rebuild assíncrono |

---

## Kafka — dois tópicos

| Tópico | Conteúdo | Produtor | Consumidor |
|--------|----------|----------|------------|
| `telemetry-events` | Telemetria bruta | ingestion-service | fleet-service |
| `domain-events` | Fatos de negócio | fleet-service (outbox) | alert-service, (analytics) |

Ambos com **partition key = `vehicleId`**.

Detalhes: [decisions/002-kafka-dois-topicos.md](./decisions/002-kafka-dois-topicos.md)

---

## CQRS leve + domain-events

```
                    telemetry-events
                           │
                           ▼
                    fleet-service (WRITE)
                     ├── event log (PG)
                     ├── Redis (sync)
                     ├── TimescaleDB (sync)
                     └── outbox → domain-events
                                      │
                                      ▼
                               alert-service
                                      │
                    query-api (READ) ◄┴── Redis + TimescaleDB
```

| Lado | Serviço | Escreve | Lê |
|------|---------|---------|-----|
| Write | fleet-service | PG, Redis, TimescaleDB, outbox | — |
| Read | query-api | — | Redis, TimescaleDB |
| Integração | alert-service | Redis (alertas) | domain-events |

**Não é Event Sourcing completo:** Redis é atualizado pelo fleet de forma síncrona; `domain-events` desacopla alertas e integrações futuras.

Ver: [decisions/001-cqrs-leve-domain-events.md](./decisions/001-cqrs-leve-domain-events.md)

---

## Simulador (Node.js)

| Modo | Endpoint | Status |
|------|----------|--------|
| **REST** | `POST /api/v1/telemetry` | MVP |
| gRPC | `SendTelemetry` | Pós-MVP |

---

## Backend (Java / Spring Boot)

| Item | Escolha |
|------|---------|
| Java | 21 (LTS) |
| Framework | Spring Boot 3.x |
| Kafka | `spring-kafka` |
| Outbox | Tabela PG + `@Scheduled` worker |
| Métricas | Micrometer + Prometheus |

Ver: [decisions/003-java-backend-node-simulator.md](./decisions/003-java-backend-node-simulator.md)

### Serviços MVP

```
services/
├── ingestion-service/    # REST → telemetry-events
├── fleet-service/        # write: PG + Redis + TimescaleDB + outbox → domain-events
├── alert-service/        # domain-events → alertas + push
└── query-api/            # read: Redis + TimescaleDB + WebSocket
```

### Pós-MVP

```
├── projection-service/   # rebuild read models via domain-events
├── analytics/            # KPIs via domain-events
└── gRPC na ingestion
```

---

## O que ficou de fora (deliberadamente)

| Item | Motivo |
|------|--------|
| Event Sourcing completo | CQRS leve + domain-events cobre o MVP |
| Actor Model / Elixir | `ConcurrentHashMap` + Kafka keyed consumer |
| RabbitMQ | Alertas via Kafka + WebSocket no MVP |
| Kubernetes | Docker Compose no MVP |
| Rust | Java no backend |

---

## Evolução futura

1. `projection-service` — read models materializados só via `domain-events`
2. Event Sourcing completo — remover write direto no Redis pelo fleet
3. Schema Registry (Avro/Protobuf)
4. gRPC, RabbitMQ, K8s
