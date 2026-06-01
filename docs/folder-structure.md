# Estrutura de Pastas — CargoTrack

Organização do monorepo — **CQRS leve + domain-events** · Java/Spring Boot + Node.js.

```
CargoTrack/
├── docs/
│   ├── README.md
│   ├── system-design.md
│   ├── stack.md
│   ├── event-model.md
│   ├── folder-structure.md
│   └── roadmap.md
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

## Ordem de implementação

1. Infra + tópicos Kafka
2. `ingestion-service`
3. `simulator`
4. `fleet-service` (writemodel + projection síncrona)
5. Outbox + `domain-events` (no fleet-service)
6. `alert-service` + `query-api`
