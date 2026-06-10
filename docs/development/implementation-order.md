---
title: Ordem de Implementação
status: stable
last_updated: 2026-06-08
owners: [engineering]
---

# Ordem de Implementação

Guia prático para implementar o CargoTrack fase a fase.

**Roadmap completo:** [roadmap.md](../product/roadmap.md)

---

## Sequência recomendada

| # | Fase | Entregável principal | Validação |
|---|------|---------------------|-----------|
| 1 | Infra | Docker Compose + Kafka + PG + Redis | `docker compose up` saudável |
| 2 | Ingestão | `ingestion-service` REST | `curl POST` → mensagem em `telemetry-events` |
| 3 | Simulador | `simulator/` Node.js | 1 veículo → Kafka sem erros |
| 4 | Fleet (write) | `fleet-service` + projeções síncronas | Redis e TimescaleDB atualizados |
| 4b | Outbox | Publicação em `domain-events` | Evento no PG → Kafka via outbox |
| 5 | Read side | `alert-service` + `query-api` | Alerta → WebSocket; GET frota < 50ms |
| 6 | Ops | Dockerfiles + CI/CD + Grafana | Teste de carga documentado |

---

## Ordem de pacotes no fleet-service

1. `consumer/` — `@KafkaListener` para `telemetry-events`
2. `domain/` — `VehicleState`, `VehicleStateManager`
3. `rules/` — regras de negócio (speed, temp, offline)
4. `writemodel/` — `EventLogRepository` (append-only)
5. `projection/` — Redis + TimescaleDB (síncrono)
6. `outbox/` — enqueue + worker para `domain-events`

---

## Scripts e infra necessários antes do código

```bash
# 1. Subir infra
docker compose -f infra/docker/docker-compose.dev.yml up -d

# 2. Criar tópicos Kafka
./scripts/create-kafka-topics.sh

# 3. Aplicar migrações
# migrations/postgres/001_domain_events.sql
# migrations/postgres/002_outbox.sql
# migrations/postgres/003_vehicle_locations.sql
```

---

## Critérios de integração entre serviços

| Checkpoint | Serviços envolvidos | Teste |
|------------|---------------------|-------|
| Pipeline básico | simulator → ingestion → Kafka | Mensagem no tópico |
| Write side | fleet → PG + Redis + Timescale | Estado persistido |
| Domain bus | fleet → outbox → domain-events → alert | Alerta gerado |
| Read side | query-api → Redis/Timescale | GET retorna dados corretos |
