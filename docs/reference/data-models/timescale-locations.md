---
title: Modelo de Dados — TimescaleDB
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# TimescaleDB — Histórico de localização

Read model para consultas de rota e trajetos históricos.

---

## Tabela `vehicle_locations`

```sql
CREATE TABLE vehicle_locations (
  time       TIMESTAMPTZ NOT NULL,
  vehicle_id TEXT NOT NULL,
  lat        DOUBLE PRECISION,
  lng        DOUBLE PRECISION,
  speed_kmh  REAL
);
SELECT create_hypertable('vehicle_locations', 'time');
```

---

## Uso

| Endpoint | Descrição |
|----------|-----------|
| `GET /api/v1/vehicles/{id}/route` | Consulta histórico por veículo |

---

## Migração

Arquivo: `migrations/postgres/003_vehicle_locations.sql`
