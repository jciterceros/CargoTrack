---
title: Getting Started
status: draft
last_updated: 2026-06-08
owners: [engineering]
---

# Getting Started

Guia para configurar o ambiente local do CargoTrack.

> **Status:** O código dos serviços ainda está em implementação. Este guia descreve o setup previsto na Fase 1 do [roadmap](../product/roadmap.md).

---

## Pré-requisitos

| Ferramenta | Versão mínima |
|------------|---------------|
| Docker | 24+ |
| Docker Compose | v2 |
| Java | 21 (LTS) |
| Node.js | 20+ |
| Git | 2.x |

---

## 1. Clonar o repositório

```bash
git clone https://github.com/jciterceros/CargoTrack.git
cd CargoTrack
```

---

## 2. Subir infraestrutura local

```bash
docker compose -f infra/docker/docker-compose.dev.yml up -d
```

Serviços esperados:

- Kafka (KRaft)
- PostgreSQL + TimescaleDB
- Redis

---

## 3. Criar tópicos Kafka

```bash
./scripts/create-kafka-topics.sh
```

Tópicos:

- `telemetry-events`
- `domain-events`

---

## 4. Aplicar migrações

Executar scripts em `migrations/postgres/`:

1. `001_domain_events.sql`
2. `002_outbox.sql`
3. `003_vehicle_locations.sql`

---

## 5. Validar pipeline (quando serviços estiverem disponíveis)

```bash
# Enviar telemetria de teste
curl -X POST http://localhost:8080/api/v1/telemetry \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "550e8400-e29b-41d4-a716-446655440000",
    "vehicleId": "TRK-001",
    "timestampMs": 1717080123123,
    "payload": {
      "type": "location",
      "lat": -23.55052,
      "lng": -46.633308,
      "speedKmh": 82.5
    }
  }'
```

---

## Próximos passos

1. [implementation-order.md](./implementation-order.md) — ordem de desenvolvimento
2. [repository-layout.md](./repository-layout.md) — estrutura do monorepo
3. [system-design.md](../architecture/system-design.md) — arquitetura completa
