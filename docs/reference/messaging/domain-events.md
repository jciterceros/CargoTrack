---
title: Contrato — domain-events
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# Contrato — `domain-events`

Eventos de domínio **imutáveis** emitidos pelo `fleet-service` após aplicar regras de negócio.

**Catálogo:** [event-catalog.md](../../domain/event-catalog.md)

---

## Tópico Kafka

| Propriedade | Valor |
|-------------|-------|
| Nome | `domain-events` |
| Partition key | `vehicleId` (via `aggregateId`) |
| Produtor | `fleet-service` (outbox) |
| Consumidores | `alert-service`, analytics (pós-MVP) |
| DLQ | `domain-events.dlq` |

---

## Fluxo de publicação

1. Fleet grava no **event log** (`domain_events`) — mesma transação
2. Fleet enfileira na **outbox**
3. Worker publica em **`domain-events`** (Kafka)
4. `alert-service` (e futuros consumers) consomem o Kafka

> Apenas eventos de domínio vão para `domain-events` — telemetria bruta fica em `telemetry-events`.

---

## Envelope padrão

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "LocationUpdated",
  "eventVersion": 1,
  "aggregateId": "TRK-001",
  "aggregateType": "vehicle",
  "occurredAt": "2026-05-30T14:22:01.123Z",
  "correlationId": "abc-123",
  "causationId": "telemetry-event-uuid",
  "payload": { }
}
```

---

## Exemplos de payload

### LocationUpdated

```json
{
  "lat": -23.550520,
  "lng": -46.633308,
  "speed_kmh": 82.5,
  "heading_deg": 180.0
}
```

### SpeedLimitExceeded

```json
{
  "speed_kmh": 105.0,
  "limit_kmh": 90.0,
  "location": { "lat": -23.55, "lng": -46.63 },
  "severity": "warning"
}
```

### RouteDeviationDetected

```json
{
  "deviation_meters": 850,
  "planned_route_id": "route-sp-rj-001",
  "location": { "lat": -23.55, "lng": -46.63 },
  "severity": "critical"
}
```

---

## Notification events (pós-MVP)

Tópico `notification-events` — eventos derivados pelo Alert Service para entrega assíncrona (RabbitMQ).

```json
{
  "notification_id": "uuid",
  "channel": "webhook",
  "recipient": "https://hooks.transportadora.com/alerts",
  "alert_type": "RouteDeviationDetected",
  "vehicle_id": "TRK-001",
  "severity": "critical",
  "message": "Veículo TRK-001 desviou 850m da rota planejada",
  "occurred_at": "2026-05-30T14:22:01.123Z"
}
```

Canais: `websocket`, `webhook`, `email`, `sms`.
