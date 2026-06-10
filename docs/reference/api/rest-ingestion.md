---
title: API REST — Ingestão
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# API REST — Ingestão

Serviço: `ingestion-service`

---

## POST /api/v1/telemetry

Recebe telemetria bruta e publica em `telemetry-events`.

| Propriedade | Valor |
|-------------|-------|
| Método | `POST` |
| Content-Type | `application/json` |
| Resposta sucesso | `202 Accepted` |
| Auth (MVP) | API key header |

### Request body

Contrato completo: [telemetry-events.md](../messaging/telemetry-events.md)

```http
POST /api/v1/telemetry HTTP/1.1
Content-Type: application/json

{
  "eventId": "uuid",
  "vehicleId": "TRK-001",
  "timestampMs": 1717080123123,
  "payload": {
    "type": "location",
    "lat": -23.55052,
    "lng": -46.633308,
    "speedKmh": 82.5
  }
}
```

### Comportamento

- Validar e normalizar payload
- Publicar em Kafka com partition key = `vehicleId`
- Sem lógica de negócio neste serviço
- Adicionar `receivedAtMs` antes de publicar

### Métricas

- `ingestion_events_total`
