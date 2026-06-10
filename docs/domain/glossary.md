---
title: Glossário de Domínio
status: stable
last_updated: 2026-06-08
owners: [architecture, product]
---

# Glossário de Domínio — CargoTrack

Este glossário padroniza os principais termos de negócio, entidades, eventos e regras de telemetria/logística do CargoTrack com base na arquitetura EDA documentada.

> Observação de alinhamento: no estado atual do repositório, os serviços (`ingestion-service`, `fleet-service`, `alert-service`, `query-api`) ainda estão descritos na documentação e roadmap, sem implementação de código-fonte publicada.

---

## 1) Termos principais

### Telemetria (`Telemetry`)

- **Correspondência no projeto:** `POST /api/v1/telemetry`, tópico Kafka `telemetry-events`.
- **Definição:** dado bruto enviado por veículo/dispositivo, antes da validação de regra de negócio.
- **Exemplo/gatilho:** cada envio com `eventId`, `vehicleId`, `timestampMs` e `payload` entra no pipeline de ingestão.

### Veículo (`Vehicle`)

- **Correspondência no projeto:** `vehicleId`, `aggregateId`, chave de partição Kafka.
- **Definição:** agregado principal de negócio monitorado pelo CargoTrack; toda ordenação de eventos é por veículo.
- **Exemplo/gatilho:** qualquer atualização de localização, sensor ou evento operacional de `TRK-001` é processada no contexto desse agregado.

### Alerta (`Alert`)

- **Correspondência no projeto:** eventos como `SpeedLimitExceeded`, `CargoTemperatureAlert`, `RouteDeviationDetected`; leitura em `/api/v1/alerts`.
- **Definição:** condição operacional relevante detectada por regra de domínio e publicada para consumo em tempo real.
- **Exemplo/gatilho:** velocidade acima do limite gera alerta de excesso de velocidade.

### Dispositivo/IoT (`Device` / `IoT`)

- **Correspondência no projeto:** campo `deviceId` (opcional), `payload.type = sensor|operational`.
- **Definição:** origem física da telemetria (hardware embarcado e sensores) associada ao veículo.
- **Exemplo/gatilho:** leitura de `fuel_level` ou `cargo_temperature` vinda do dispositivo alimenta o processamento de domínio.

### Cerca eletrônica (`Geofence`)

- **Correspondência no projeto:** atualmente representada por `RouteDeviationDetected` (não existe termo/evento explícito `GeofenceBreach` nos docs atuais).
- **Definição:** regra de desvio de área/rota planejada usada para detectar comportamento fora do esperado.
- **Exemplo/gatilho:** distância da posição atual para a rota planejada acima do `threshold` gera `RouteDeviationDetected`.

### Event log (`domain_events`)

- **Correspondência no projeto:** tabela PostgreSQL `domain_events`.
- **Definição:** trilha append-only de eventos de domínio para auditoria e rastreabilidade.
- **Exemplo/gatilho:** após processar telemetria válida, o `fleet-service` persiste o evento no event log.

### Outbox (`outbox`)

- **Correspondência no projeto:** tabela `outbox`, `OutboxPublisher`/`OutboxWorker`.
- **Definição:** mecanismo de consistência entre persistência no banco e publicação em `domain-events`.
- **Exemplo/gatilho:** evento gravado em `domain_events` é enfileirado em `outbox` e publicado de forma assíncrona no Kafka.

### Read model

- **Correspondência no projeto:** Redis (`vehicle:{vehicleId}:state`) e TimescaleDB (`vehicle_locations`).
- **Definição:** projeções otimizadas para consulta rápida (estado atual e histórico).
- **Exemplo/gatilho:** `LocationUpdated` atualiza estado atual em Redis e histórico em TimescaleDB.

---

## 2) Eventos de telemetria (entrada)

Ver contratos em [telemetry-events.md](../reference/messaging/telemetry-events.md).

### Envelope de telemetria

- **Campos principais:** `eventId`, `vehicleId`, `deviceId` (opcional), `timestampMs`, `payload`.
- **Definição:** contrato padrão para eventos brutos recebidos pela ingestão.
- **Exemplo/gatilho:** request válido em `POST /api/v1/telemetry` publicado em `telemetry-events`.

### `payload.type = location`

- **Campos típicos:** `lat`, `lng`, `accuracyM`, `headingDeg`, `speedKmh`.
- **Definição:** atualização de posição e movimento do veículo.

### `payload.type = sensor`

- **Campos típicos:** `sensor`, `value`, `unit`.
- **Sensores documentados:** `fuel_level`, `cargo_temperature`, `engine_rpm`, `odometer_km`.

### `payload.type = operational`

- **Campos típicos:** `code`, `doorId`, `value`.
- **Códigos documentados:** `engine_on`, `door_open`, `maintenance_dtc`, `panic_button`.

---

## 3) Eventos de domínio (`domain-events`)

Ver catálogo completo em [event-catalog.md](./event-catalog.md).

---

## 4) Estados do veículo

### `vehicle:{vehicleId}:state` (Redis)

- **Definição:** visão quente do estado atual consumida pelo read side.
- **Campos citados na documentação:** `lat`, `lng`, `speed`, `fuel`, `temp`, `engine`, `doors`, `alerts`, `updatedAt`.

### Estado de conectividade

- **Estados relevantes:** online/offline.
- **Gatilho documentado:** sem eventos por mais de 60s → `VehicleOffline`; evento subsequente → `VehicleOnline`.

---

## 5) Regras de alerta e gatilhos de negócio

| Regra | Evento | Gatilho |
|-------|--------|---------|
| Velocidade | `SpeedLimitExceeded` | `speedKmh > 90` |
| Temperatura de carga | `CargoTemperatureAlert` | `cargoTempC` fora da faixa operacional |
| Inatividade | `VehicleOffline` | sem evento por `> 60s` |
| Desvio de rota | `RouteDeviationDetected` | distância para rota planejada `> threshold` |
| Manutenção | `MaintenanceAlertRaised` | ocorrência de `maintenance_dtc` ou regra preventiva |

---

## 6) Canais e fluxo EDA

| Canal | Papel |
|-------|-------|
| `telemetry-events` | Entrada de dados brutos de alta volumetria |
| `domain-events` | Integração de fatos de negócio validados |
| `notification-events` | Notificações derivadas (pós-MVP) |

---

## 7) Padronização de linguagem

- **Telemetria:** dados brutos de entrada (`telemetry-events`).
- **Evento de domínio:** fatos validados (`domain-events` + `domain_events`).
- **Alerta:** condição acionável; referenciar `eventType` específico.
- **Cerca eletrônica (Geofence):** padronizar como regra de `RouteDeviationDetected`.

---

## 8) Observações de consistência

- Há variação de naming entre camelCase e snake_case nos exemplos (`speedKmh` vs `speed_kmh`).
- Recomenda-se contrato canônico por contexto:
  - Telemetria de entrada: camelCase.
  - Eventos de domínio: padrão único versionado (ver [event-catalog.md](./event-catalog.md)).
