# Roadmap de Implementação — CargoTrack

**Padrão:** CQRS leve + Kafka (`telemetry-events` + `domain-events`)  
Stack: **Java/Spring Boot** + **Node.js** (simulador)

Detalhes: [stack.md](./stack.md) · [system-design.md](./system-design.md)

---

## Visão geral

```
Fase 1 ──► Fase 2 ──► Fase 3 ──► Fase 4 ──► Fase 4b ──► Fase 5 ──► Fase 6
 Infra      Ingestão    Simulador   Fleet       domain-    Query +    Deploy
            (Java)      (Node)      (write)     events     Alert      + Obs
```

---

## Fase 1 — Fundações (Infra + Mensageria)

**Objetivo:** Ambiente local reproduzível.

### Entregas

- [ ] `infra/docker/docker-compose.dev.yml` — Kafka (KRaft), PostgreSQL (+ TimescaleDB), Redis
- [ ] Script `scripts/create-kafka-topics.sh` — `telemetry-events`, `domain-events`
- [ ] Migrações: `domain_events`, `outbox`, `vehicle_locations` (hypertable)
- [ ] Validar produce/consume nos dois tópicos

### Critério de done

- `docker compose up` sobe infra saudável
- Tópicos Kafka criados com partições e key strategy documentada

### Estimativa: 3–5 dias

---

## Fase 2 — Ingestão (Java/Spring Boot)

**Objetivo:** REST → `telemetry-events`.

### Entregas

- [ ] Projeto `services/ingestion-service/` (Spring Boot 3, Java 21)
- [ ] `POST /api/v1/telemetry` — JSON conforme [event-model.md](./event-model.md)
- [ ] `TelemetryProducer` — key = `vehicleId`, tópico `telemetry-events`
- [ ] Actuator health + métrica `ingestion_events_total`

### Critério de done

- `curl -X POST` → mensagem no tópico `telemetry-events`

### Estimativa: 3–7 dias

---

## Fase 3 — Simulador (Node.js)

**Objetivo:** Carga de telemetria via REST e gRPC.

### Entregas

- [ ] Projeto `simulator/` — `rest-client.js`, rotas simuladas
- [ ] Config: `VEHICLE_COUNT`, `INTERVAL_MS`, `INGESTION_URL`
- [ ] 100 veículos → ingestion → Kafka sem erros

### Estimativa: 2–4 dias

---

## Fase 4 — Fleet Service (write side — CQRS leve)

**Objetivo:** Consumir telemetria, gravar event log, materializar read models **síncronos**.

### Entregas

- [ ] Projeto `services/fleet-service/`
- [ ] `@KafkaListener` → `telemetry-events`
- [ ] `VehicleStateManager` + regras (speed, temp, offline)
- [ ] `EventLogRepository` — INSERT append-only em `domain_events`
- [ ] `RedisFleetProjection` — UPDATE `vehicle:{id}:state`
- [ ] `TimescaleLocationProjection` — INSERT `vehicle_locations`
- [ ] Pacotes: `writemodel/`, `projection/`

### Critério de done

- Simulador 100+ veículos → Redis e TimescaleDB atualizados
- Eventos gravados em `domain_events`
- Query-api ainda não necessária para validar

### Estimativa: 1–2 semanas

---

## Fase 4b — domain-events (Outbox + publicação)

**Objetivo:** Publicar fatos de negócio no Kafka para consumidores desacoplados.

### Entregas

- [ ] Tabela `outbox` + migração Flyway
- [ ] `OutboxRepository` — enqueue na mesma transação do event log
- [ ] `OutboxPublisher` — worker scheduled que publica em `domain-events`
- [ ] `DomainEventPublisher` — key = `vehicleId`
- [ ] Métrica `domain_events_published_total`, `outbox_pending_count`
- [ ] DLQ `domain-events.dlq` para falhas irrecuperáveis

### Critério de done

- Evento gravado no PG → aparece em `domain-events` (via outbox)
- Restart do fleet não perde eventos não publicados

### Estimativa: 3–5 dias

---

## Fase 5 — Alert Service + Query API (read side)

**Objetivo:** Consultas, histórico e push de alertas.

### Entregas

- [ ] `alert-service`:
  - [ ] Consumer `domain-events` (filtro: `SpeedLimitExceeded`, `CargoTemperatureAlert`, etc.)
  - [ ] Deduplicação por `eventId`
  - [ ] Redis `alerts:active`
- [ ] `query-api`:
  - [ ] `GET /api/v1/fleet`, `/vehicles/{id}`, `/vehicles/{id}/route`, `/alerts`
  - [ ] WebSocket `/ws/fleet` — push (via alert-service ou consumer próprio de `domain-events`)
  - [ ] Spring Security + JWT (básico)
  - [ ] **Somente leitura** — Redis + TimescaleDB

### Critério de done

- Simulador 1.000 veículos sustentado
- Alerta de velocidade → WebSocket no dashboard
- GET frota retorna estado do Redis (< 50 ms local)

### Estimativa: 2–3 semanas

---

## Fase 6 — Deploy, Observabilidade e CI/CD

### Entregas

- [ ] Dockerfile por serviço + simulador
- [ ] GitHub Actions: build, test, Docker
- [ ] Prometheus + Grafana (lag Kafka, outbox pending, throughput)
- [ ] Teste de carga documentado

### Estimativa: 1–2 semanas

---

## Ordem para começar hoje

1. Fase 1 — Docker + tópicos `telemetry-events` e `domain-events`
2. Fase 2 — ingestion REST
3. Fase 3 — simulador Node (1 veículo)
4. Fase 4 — fleet write side (PG + Redis + TimescaleDB)
5. Fase 4b — outbox + `domain-events`
6. Fase 5 — alert-service + query-api

---

## Serviços por fase

| Fase | Serviços ativos |
|------|-----------------|
| 1 | infra (Kafka, PG, Redis) |
| 2 | + ingestion-service |
| 3 | + simulator |
| 4 | + fleet-service |
| 4b | fleet-service (outbox) |
| 5 | + alert-service, query-api |
| 6 | todos containerizados |

---

## Riscos e mitigações

| Risco | Mitigação |
|-------|-----------|
| Outbox acumula | Monitorar `outbox_pending_count`; alerta Grafana |
| domain-events duplica PG | PG = auditoria; Kafka = integração — papéis distintos |
| Muitos serviços Java | 4 serviços no MVP (ingestion, fleet, alert, query) |
| gRPC | Adiar — REST basta |

---

## Backlog pós-MVP

- `projection-service` — rebuild assíncrono via `domain-events`
- Event Sourcing completo — read models só via projeção
- gRPC na ingestão
- RabbitMQ (email/SMS/webhook)
- Schema Registry
- Frontend dashboard (React, Angular ou Flutter)
- Kubernetes
