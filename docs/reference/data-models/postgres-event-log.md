---
title: Modelo de Dados — PostgreSQL Event Log
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# PostgreSQL — Event Log

Tabela append-only para auditoria de eventos de domínio.

> Event log para **auditoria**. Estado atual vem do Redis, não do replay.

---

## Tabela `domain_events`

```sql
CREATE TABLE domain_events (
  id             BIGSERIAL PRIMARY KEY,
  event_id       UUID NOT NULL UNIQUE,
  aggregate_id   TEXT NOT NULL,
  aggregate_type VARCHAR(64) NOT NULL DEFAULT 'vehicle',
  event_type     VARCHAR(128) NOT NULL,
  event_version  INT NOT NULL DEFAULT 1,
  payload        JSONB NOT NULL,
  occurred_at    TIMESTAMPTZ NOT NULL,
  recorded_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## Tabela `outbox`

Garante consistência entre event log e Kafka:

```sql
CREATE TABLE outbox (
  id         UUID PRIMARY KEY,
  event_id   UUID NOT NULL UNIQUE,
  payload    JSONB NOT NULL,
  published  BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Worker no `fleet-service` lê `outbox WHERE published = false` → publica em `domain-events` → marca `published = true`.

---

## Migrações

Arquivos em `migrations/postgres/`:

- `001_domain_events.sql`
- `002_outbox.sql`
