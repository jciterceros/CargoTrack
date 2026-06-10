---
title: Layout do Repositório
status: stable
last_updated: 2026-06-08
owners: [engineering]
---

# Layout do Repositório — CargoTrack

Organização do monorepo — **CQRS leve + domain-events** · Java/Spring Boot + Node.js.

```
CargoTrack/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
│
├── docs/
│   ├── README.md
│   ├── product/
│   │   ├── overview.md
│   │   ├── vision-and-story.md
│   │   └── roadmap.md
│   ├── architecture/
│   │   ├── system-design.md
│   │   ├── stack.md
│   │   └── decisions/
│   ├── domain/
│   │   ├── glossary.md
│   │   └── event-catalog.md
│   ├── reference/
│   │   ├── api/
│   │   ├── messaging/
│   │   └── data-models/
│   ├── development/
│   │   ├── getting-started.md
│   │   ├── repository-layout.md
│   │   └── implementation-order.md
│   └── assets/
│       └── diagrams/
│           └── architecture-overview.svg
│
├── simulator/                     # Node.js
│   └── src/
│       ├── clients/rest-client.js
│       └── vehicles/fleet.js
│
├── services/
│   ├── ingestion-service/         # REST → telemetry-events
│   │   └── src/main/java/com/cargotrack/ingestion/
│   │       ├── api/TelemetryController.java
│   │       └── kafka/TelemetryProducer.java
│   │
│   ├── fleet-service/             # WRITE SIDE — CQRS leve + outbox
│   │   └── src/main/java/com/cargotrack/fleet/
│   │       ├── consumer/TelemetryConsumer.java
│   │       ├── domain/
│   │       │   ├── VehicleState.java
│   │       │   └── VehicleStateManager.java
│   │       ├── rules/
│   │       │   ├── SpeedRule.java
│   │       │   └── TemperatureRule.java
│   │       ├── writemodel/
│   │       │   └── EventLogRepository.java      # append-only PG
│   │       ├── projection/
│   │       │   ├── RedisFleetProjection.java      # sync read model
│   │       │   └── TimescaleLocationProjection.java
│   │       └── outbox/
│   │           ├── OutboxRepository.java
│   │           ├── OutboxPublisher.java           # → domain-events
│   │           └── OutboxWorker.java
│   │
│   ├── alert-service/             # consome domain-events
│   │   └── src/main/java/com/cargotrack/alert/
│   │       ├── consumer/DomainEventConsumer.java
│   │       └── service/AlertDeduplicationService.java
│   │
│   ├── query-api/                 # READ SIDE
│   │   └── src/main/java/com/cargotrack/query/
│   │       ├── api/FleetController.java
│   │       └── ws/FleetWebSocketHandler.java
│   │
│   └── projection-service/        # PÓS-MVP — rebuild via domain-events
│
├── migrations/postgres/
│   ├── 001_domain_events.sql
│   ├── 002_outbox.sql
│   └── 003_vehicle_locations.sql
│
├── infra/docker/
│   └── docker-compose.dev.yml
│
└── scripts/
    └── create-kafka-topics.sh     # telemetry-events + domain-events
```

## Dependências entre serviços

```
simulator
    │ REST
    ▼
ingestion-service ──► telemetry-events
                            │
                            ▼
                      fleet-service (WRITE)
                       ├── domain_events (PG)
                       ├── Redis
                       ├── TimescaleDB
                       └── outbox ──► domain-events
                                            │
                            ┌───────────────┼───────────────┐
                            ▼               ▼               ▼
                     alert-service    analytics      projection-service
                     (MVP)            (pós-MVP)       (pós-MVP)
                            │
                            ▼
                      query-api (READ) ◄── Redis + TimescaleDB
```
