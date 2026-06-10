---
title: Catálogo de Eventos
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# Catálogo de Eventos — CargoTrack

Referência canônica dos eventos de domínio e do mapeamento telemetria → domínio.

**Contratos detalhados:** [telemetry-events.md](../reference/messaging/telemetry-events.md) · [domain-events.md](../reference/messaging/domain-events.md)

---

## Catálogo de eventos de domínio

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

### Por categoria

#### Ciclo de vida do veículo

- `VehicleRegistered`, `VehicleOffline`, `VehicleOnline`

#### Estado operacional e telemetria consolidada

- `LocationUpdated`, `FuelLevelUpdated`, `CargoTemperatureUpdated`, `EngineStateChanged`, `DoorStateChanged`

#### Alertas operacionais

- `SpeedLimitExceeded`, `RouteDeviationDetected`, `CargoTemperatureAlert`, `MaintenanceAlertRaised`

#### Gestão de alerta

- `AlertAcknowledged`, `AlertResolved`

---

## Mapeamento telemetria → domínio

| Telemetria (payload) | Evento(s) de domínio |
|----------------------|----------------------|
| `location` | `LocationUpdated`; opcionalmente `SpeedLimitExceeded`, `RouteDeviationDetected` |
| `sensor: fuel_level` | `FuelLevelUpdated` |
| `sensor: cargo_temperature` | `CargoTemperatureUpdated`; opcionalmente `CargoTemperatureAlert` |
| `operational: engine_on` | `EngineStateChanged` |
| `operational: door_open` | `DoorStateChanged` |
| `operational: maintenance_dtc` | `MaintenanceAlertRaised` |
| (timeout heartbeat) | `VehicleOffline` / `VehicleOnline` |

---

## Idempotência

| Camada | Chave |
|--------|-------|
| Ingestion → Kafka | `eventId` (dedup no producer ou compact topic) |
| Fleet Processor | `eventId` da telemetria — ignorar duplicatas |
| Projections | `eventId` do domínio — upsert idempotente |
| Event Store | UNIQUE constraint em `event_id` |

---

## Evolução de contratos

- Registrar schemas Protobuf/Avro no Kafka Schema Registry (pós-MVP)
- **Backward compatible:** novos campos opcionais; nunca remover/renomear sem nova versão
- `eventVersion` incrementa em breaking changes; consumidores suportam N e N-1
