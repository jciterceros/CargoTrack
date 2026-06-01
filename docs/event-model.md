# Modelo de Eventos — CargoTrack

Contratos de telemetria e eventos de domínio.

**Formato adotado:** JSON (REST e Kafka no MVP).

---

## Glossário

| Termo | Significado no CargoTrack |
|-------|---------------------------|
| **Telemetria** | Dado bruto do veículo — tópico `telemetry-events` |
| **Evento de domínio** | Fato de negócio validado — tópico `domain-events` + tabela `domain_events` |
| **Event log** | Tabela `domain_events` append-only — auditoria; **não** é Event Sourcing completo |
| **Read model** | Redis (estado atual) e TimescaleDB (histórico) — atualizados pelo fleet de forma síncrona |
| **Outbox** | Tabela que garante publicação de `domain-events` após gravar o event log |

---

## 1. Telemetria — REST `POST /api/v1/telemetry` e Kafka `telemetry-events`

### Request body (REST) / Value (Kafka)

Campos em **camelCase** (convenção Java/Node).

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `eventId` | UUID | sim | Idempotência |
| `vehicleId` | string | sim | Partition key Kafka |
| `deviceId` | string | não | Dispositivo no veículo |
| `timestampMs` | number | sim | Epoch ms (origem) |
| `payload` | object | sim | Ver tipos abaixo |

> O `ingestion-service` adiciona `receivedAtMs` antes de publicar no Kafka.

### Tipos de payload (`payload.type`)

Valores: `location` | `sensor` | `operational`

#### location

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

#### sensor

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

#### operational

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

---

## 2. Eventos de domínio — Kafka `domain-events` + event log (PostgreSQL)

Eventos **imutáveis** emitidos pelo `fleet-service` após aplicar regras de negócio.

Fluxo:

1. Fleet grava no **event log** (`domain_events`) — mesma transação
2. Fleet enfileira na **outbox**
3. Worker publica em **`domain-events`** (Kafka)
4. `alert-service` (e futuros consumers) consomem o Kafka

> Apenas eventos de domínio vão para `domain-events` — telemetria bruta fica em `telemetry-events`.

### Envelope padrão

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

### Catálogo de eventos

| eventType | Versão | Dispara quando | Efeito na projeção |
|-----------|--------|----------------|-------------------|
| `VehicleRegistered` | 1 | Novo veículo na frota | Cria entrada em Redis |
| `LocationUpdated` | 1 | GPS válido recebido | Atualiza lat/lng/speed; append TimescaleDB |
| `FuelLevelUpdated` | 1 | Leitura de combustível | Atualiza `fuel_pct` |
| `CargoTemperatureUpdated` | 1 | Leitura de temperatura | Atualiza `cargo_temp_c`; pode gerar alerta |
| `EngineStateChanged` | 1 | Motor on/off | Atualiza `engine_on` |
| `DoorStateChanged` | 1 | Porta aberta/fechada | Atualiza `doors_open` |
| `SpeedLimitExceeded` | 1 | speed > limite (ex. 90 km/h) | Adiciona alerta ativo |
| `RouteDeviationDetected` | 1 | distância à rota > threshold | Adiciona alerta ativo |
| `CargoTemperatureAlert` | 1 | temp fora da faixa configurada | Adiciona alerta ativo |
| `MaintenanceAlertRaised` | 1 | DTC ou regra preventiva | Adiciona alerta ativo |
| `VehicleOffline` | 1 | Sem evento > N segundos | Marca offline; alerta |
| `VehicleOnline` | 1 | Evento após offline | Remove flag offline |
| `AlertAcknowledged` | 1 | Operador confirma alerta | Remove da lista ativa |
| `AlertResolved` | 1 | Condição normalizada | Remove alerta |

### Exemplos de payload

#### LocationUpdated

```json
{
  "lat": -23.550520,
  "lng": -46.633308,
  "speed_kmh": 82.5,
  "heading_deg": 180.0
}
```

#### SpeedLimitExceeded

```json
{
  "speed_kmh": 105.0,
  "limit_kmh": 90.0,
  "location": { "lat": -23.55, "lng": -46.63 },
  "severity": "warning"
}
```

#### RouteDeviationDetected

```json
{
  "deviation_meters": 850,
  "planned_route_id": "route-sp-rj-001",
  "location": { "lat": -23.55, "lng": -46.63 },
  "severity": "critical"
}
```

---

## 3. Mapeamento telemetria → domínio

| Telemetria (payload) | Evento(s) de domínio |
|----------------------|----------------------|
| `LocationUpdate` | `LocationUpdated`; opcionalmente `SpeedLimitExceeded`, `RouteDeviationDetected` |
| `SensorReading: fuel_level` | `FuelLevelUpdated` |
| `SensorReading: cargo_temperature` | `CargoTemperatureUpdated`; opcionalmente `CargoTemperatureAlert` |
| `OperationalEvent: engine_on` | `EngineStateChanged` |
| `OperationalEvent: door_open` | `DoorStateChanged` |
| `OperationalEvent: maintenance_dtc` | `MaintenanceAlertRaised` |
| (timeout heartbeat) | `VehicleOffline` / `VehicleOnline` |

---

## 4. Schema Registry e evolução

- Registrar schemas Protobuf/Avro no Kafka Schema Registry
- **Backward compatible:** novos campos opcionais; nunca remover/renomear sem nova versão
- `event_version` incrementa em breaking changes; consumidores suportam N e N-1

---

## 5. Idempotência

| Camada | Chave |
|--------|-------|
| Ingestion → Kafka | `event_id` (dedup no producer ou compact topic) |
| Fleet Processor | `event_id` da telemetria — ignorar duplicatas |
| Projections | `event_id` do domínio — upsert idempotente |
| Event Store | UNIQUE constraint em `event_id` |

---

## 6. Notification Events — tópico `notification-events`

Eventos derivados pelo Alert Service para entrega assíncrona (RabbitMQ).

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
