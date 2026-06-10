---
title: API REST — Query
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# API REST — Query

Serviço: `query-api` (read side)

**Auth:** Spring Security + JWT (MVP)

---

## Endpoints

| Método | Rota | Descrição | Fonte de dados |
|--------|------|-----------|----------------|
| GET | `/api/v1/fleet` | Frota com estado atual | Redis |
| GET | `/api/v1/vehicles/{id}` | Detalhe do veículo | Redis |
| GET | `/api/v1/vehicles/{id}/route` | Histórico de rota | TimescaleDB |
| GET | `/api/v1/alerts` | Alertas ativos | Redis |
| WS | `/ws/fleet` | Push tempo real | alert-service / domain-events |

---

## Regras

- **Somente leitura** — nunca escreve no event log
- Estado atual vem do Redis (`vehicle:{vehicleId}:state`)
- Histórico vem do TimescaleDB (`vehicle_locations`)

---

## WebSocket

| Propriedade | Valor |
|-------------|-------|
| Endpoint | `/ws/fleet` |
| Protocolo | WebSocket/STOMP |
| Uso | Push de alertas e atualizações de frota em tempo real |
