---
title: Contrato — telemetry-events
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# Contrato — `telemetry-events`

Telemetria bruta recebida via REST e publicada no Kafka.

**Formato adotado:** JSON (REST e Kafka no MVP).  
**API REST:** [rest-ingestion.md](../api/rest-ingestion.md)

---

## Tópico Kafka

| Propriedade | Valor |
|-------------|-------|
| Nome | `telemetry-events` |
| Partition key | `vehicleId` |
| Produtor | `ingestion-service` |
| Consumidor | `fleet-service` |
| DLQ | `telemetry-events.dlq` |

---

## Envelope (REST / Kafka value)

Campos em **camelCase** (convenção Java/Node).

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `eventId` | UUID | sim | Idempotência |
| `vehicleId` | string | sim | Partition key Kafka |
| `deviceId` | string | não | Dispositivo no veículo |
| `timestampMs` | number | sim | Epoch ms (origem) |
| `payload` | object | sim | Ver tipos abaixo |

> O `ingestion-service` adiciona `receivedAtMs` antes de publicar no Kafka.

---

## Tipos de payload (`payload.type`)

Valores: `location` | `sensor` | `operational`

### location

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "vehicleId": "TRK-001",
  "timestampMs": 1717080123123,
  "payload": {
    "type": "location",
    "lat": -23.55052,
    "lng": -46.633308,
    "accuracyM": 5.0,
    "headingDeg": 180.0,
    "speedKmh": 82.5
  }
}
```

### sensor

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440001",
  "vehicleId": "TRK-001",
  "timestampMs": 1717080123123,
  "payload": {
    "type": "sensor",
    "sensor": "fuel_level",
    "value": 67.3,
    "unit": "percent"
  }
}
```

Sensores: `fuel_level`, `cargo_temperature`, `engine_rpm`, `odometer_km`.

### operational

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440002",
  "vehicleId": "TRK-001",
  "timestampMs": 1717080123123,
  "payload": {
    "type": "operational",
    "code": "door_open",
    "doorId": "rear",
    "value": true
  }
}
```

Códigos: `engine_on`, `door_open`, `maintenance_dtc`, `panic_button`.
