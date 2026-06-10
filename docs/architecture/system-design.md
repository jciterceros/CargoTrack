---
title: System Design
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# System Design — CargoTrack

> Plataforma de rastreamento logístico e telemetria de frotas em tempo real.

**Stack adotada:** Java/Spring Boot (backend) · Node.js (simulador) · Kafka · PostgreSQL · Redis · TimescaleDB

**Padrão de dados:** **CQRS leve + `domain-events`** — detalhes em [stack.md](./stack.md)

---

## 1. Contexto e Objetivo

### Problema

Uma transportadora precisa monitorar **1.000+ caminhões** em tempo real. Cada veículo transmite, a cada poucos segundos, GPS, velocidade, combustível, temperatura e eventos operacionais — gerando **milhões de eventos por dia**.

### Objetivo

Demonstrar arquitetura orientada a eventos com **CQRS leve**, Kafka em dois tópicos (`telemetry-events` + `domain-events`), e observabilidade — usando **Java** no backend e **Node.js** no simulador de carga.

### Referências

- [README do projeto](../../README.md) — visão de produto
- [stack.md](./stack.md) — decisão tecnológica adotada

---

## 2. Requisitos

### Funcionais

| ID | Requisito |
|----|-----------|
| RF-01 | Receber telemetria de milhares de veículos simultaneamente |
| RF-02 | Manter estado atual de cada veículo |
| RF-03 | Detectar e emitir alertas (velocidade, temperatura, desvio, offline) |
| RF-04 | Persistir event log append-only (auditoria) |
| RF-05 | Consultas rápidas de frota |
| RF-06 | Histórico de rotas |
| RF-07 | Dashboard com atualizações em tempo real (WebSocket) |
| RF-08 | Simulador de frota (Node.js) |

### Não-funcionais

| ID | Requisito | Meta |
|----|-----------|------|
| RNF-01 | Latência ingestão → Kafka (P99) | < 100 ms (local) |
| RNF-02 | Throughput | ≥ 1.000 eventos/s (MVP) |
| RNF-03 | Disponibilidade | 99,9% (produção) |
| RNF-04 | Isolamento de falha | Erro em um veículo não derruba o consumer |
| RNF-05 | Observabilidade | Métricas, logs, traces |

---

## 3. Arquitetura de Alto Nível

### Princípios

1. **Event-Driven Architecture (EDA)** — Kafka como barramento; dois tópicos com papéis distintos
2. **CQRS leve** — write side (`fleet-service`) grava event log + read models; read side (`query-api`) consulta só Redis e TimescaleDB
3. **`domain-events`** — bus de fatos de negócio para consumidores desacoplados (alertas, analytics); **não** substitui projeções síncronas no MVP
4. **Partition key = `vehicleId`** — ordenação por veículo em ambos os tópicos
5. **REST no MVP** — gRPC como evolução futura — simulador Node suporta ambos protocolos

### CQRS leve vs Event Sourcing completo

| Aspecto | CargoTrack (adotado) | Event Sourcing completo (pós-MVP) |
|---------|----------------------|-----------------------------------|
| Fonte do estado atual | Redis (atualizado pelo fleet) | Replay de eventos |
| Event log (PostgreSQL) | Auditoria | Única fonte de verdade |
| Read models | Projeção **síncrona** no fleet | Projeção **assíncrona** via consumer |
| `domain-events` | Notifica outros serviços | Obrigatório para materializar read models |

### Fluxo principal

```
Simulador (Node)
      │  POST /api/v1/telemetry  (JSON)
      ▼
Ingestion Service
      ▼
Kafka: telemetry-events  (key: vehicleId)
      ▼
Fleet Service (write side)
      │  regras de negócio
      ├──► event log — INSERT domain_events (PostgreSQL)
      ├──► projeção síncrona — UPDATE Redis
      ├──► projeção síncrona — INSERT vehicle_locations (TimescaleDB)
      └──► outbox → publish domain-events (Kafka)
                │
                ├──► alert-service (alertas, push WebSocket)
                └──► analytics (opcional, pós-MVP)
      │
Query API (read side)
      ├── GET ← Redis + TimescaleDB
      └── WebSocket ← alert-service ou consumer de domain-events
```

---

## 4. Componentes

### 4.1 Simulador de Frota (`simulator/`) — Node.js

| Aspecto | Detalhe |
|---------|---------|
| Linguagem | **Node.js** 20+ |
| Transporte | **REST** (POST JSON) — MVP |
| Config | `VEHICLE_COUNT`, `INTERVAL_MS`, `INGESTION_URL` |

Ver contrato: [rest-ingestion.md](../reference/api/rest-ingestion.md)

---

### 4.2 Ingestion Service (`services/ingestion-service/`) — Java

| Aspecto | Detalhe |
|---------|---------|
| Entrada | `POST /api/v1/telemetry` — JSON |
| Saída | `spring-kafka` → `telemetry-events` |
| Regra | Validar, normalizar, publicar — sem lógica de negócio |

- Resposta `202 Accepted`
- Partition key = `vehicleId`
- gRPC — pós-MVP

---

### 4.3 Fleet Service (`services/fleet-service/`) — Java (write side)

| Aspecto | Detalhe |
|---------|---------|
| Consumo | `@KafkaListener` → `telemetry-events` |
| Estado | `VehicleStateManager` — `ConcurrentHashMap` (cache de processamento) |
| Event log | `EventLogRepository` — append-only em `domain_events` |
| Projeções | Síncronas: `RedisFleetProjection`, `TimescaleLocationProjection` |
| Publicação | `OutboxPublisher` → `domain-events` (mesma transação do event log) |

**Handler (padrão CQRS leve + domain-events):**

```java
@Transactional
public void handle(TelemetryEvent telemetry) {
    VehicleState next = rules.apply(stateManager.getOrCreate(telemetry.vehicleId()), telemetry);
    List<DomainEvent> events = rules.collectEvents(next, telemetry);

    stateManager.put(telemetry.vehicleId(), next);

    events.forEach(eventLog::append);           // PG — event log
    redisProjection.apply(next);                // read model quente
    timescaleProjection.applyIfLocation(next);  // read model histórico
    events.forEach(outbox::enqueue);            // → domain-events (async worker)
}
```

**Regras de negócio:**

- `speedKmh > 90` → `SpeedLimitExceeded`
- `cargoTempC` fora da faixa → `CargoTemperatureAlert`
- Sem evento > 60s → `VehicleOffline`

---

### 4.4 Alert Service (`services/alert-service/`) — Java

| Aspecto | Detalhe |
|---------|---------|
| Consumo | `@KafkaListener` → `domain-events` (filtro: tipos de alerta) |
| Saída | Redis (alertas ativos), push WebSocket via query-api |
| Idempotência | `eventId` |

Consome fatos de negócio já validados — não processa telemetria bruta.

---

### 4.5 Query API (`services/query-api/`) — Java (read side)

| Aspecto | Detalhe |
|---------|---------|
| Leitura | Redis + TimescaleDB — **nunca escreve** no event log |
| APIs | REST + WebSocket/STOMP |
| Auth | Spring Security + JWT |

Ver endpoints: [rest-query.md](../reference/api/rest-query.md)

---

### 4.6 Projection Service (`services/projection-service/`) — pós-MVP

Serviço **opcional** para rebuild assíncrono de read models a partir de `domain-events`. No MVP, projeções ficam **síncronas no fleet-service**.

---

## 5. Mensageria (Kafka)

| Tópico | Produtor | Consumidor | Key | Papel |
|--------|----------|------------|-----|-------|
| `telemetry-events` | ingestion-service | fleet-service | `vehicleId` | Telemetria bruta (entrada) |
| `domain-events` | fleet-service (outbox) | alert-service, analytics | `vehicleId` | Fatos de negócio (integração) |
| `telemetry-events.dlq` | — | monitoramento | — | Erros de ingestão |
| `domain-events.dlq` | — | monitoramento | — | Erros de domínio |

**Formato:** JSON no MVP.

Detalhes: [messaging/](../reference/messaging/)

### Outbox pattern

Ver: [postgres-event-log.md](../reference/data-models/postgres-event-log.md)

---

## 6. Modelo de Dados

| Store | Documento |
|-------|-----------|
| PostgreSQL (event log + outbox) | [postgres-event-log.md](../reference/data-models/postgres-event-log.md) |
| Redis (estado atual) | [redis-state.md](../reference/data-models/redis-state.md) |
| TimescaleDB (histórico) | [timescale-locations.md](../reference/data-models/timescale-locations.md) |
| Catálogo de eventos | [event-catalog.md](../domain/event-catalog.md) |

### Distribuição CQRS leve

```mermaid
flowchart LR
    subgraph kafka
        T[telemetry-events]
        D[domain-events]
    end
    subgraph write["fleet-service (write)"]
        F[Fleet Handler]
    end
    subgraph stores
        PG[(Event log PG)]
        R[(Redis)]
        TS[(TimescaleDB)]
    end
    subgraph read["query-api (read)"]
        Q[Query API]
    end
    subgraph consumers
        A[alert-service]
    end
    T --> F
    F --> PG
    F --> R
    F --> TS
    F --> D
    D --> A
    R --> Q
    TS --> Q
```

---

## 7. Comunicação Entre Serviços

| De → Para | Protocolo | Quando |
|-----------|-----------|--------|
| Simulador → Ingestion | REST POST JSON | Envio de telemetria |
| Ingestion → Fleet | Kafka `telemetry-events` | Sempre |
| Fleet → PG / Redis / TimescaleDB | JDBC / Redis | Projeções síncronas (CQRS leve) |
| Fleet → Alert / Analytics | Kafka `domain-events` | Após persistir event log |
| Query API → Clientes | REST / WebSocket | Consultas e push |
| Alert → Query API | HTTP interno ou WS relay | Push de alertas |

---

## 8. Infraestrutura

### Local — Docker Compose

```
infra/docker/docker-compose.dev.yml
├── kafka (KRaft)
├── postgresql (+ TimescaleDB)
└── redis
```

### Serviços MVP

```
simulator          (Node)
ingestion-service  (Java)
fleet-service      (Java) — write side + outbox
alert-service      (Java) — consome domain-events
query-api          (Java) — read side
```

Kubernetes, RabbitMQ, gRPC — pós-MVP.

---

## 9. Observabilidade

**Métricas-chave:** `ingestion_events_total`, `domain_events_published_total`, `kafka_consumer_lag`, `outbox_pending_count`, `active_vehicles`.

---

## 10. Segurança

| Camada | Medida |
|--------|--------|
| Ingestão | API key header (MVP) |
| Kafka | SASL em produção |
| Query API | JWT + RBAC |

---

## 11. Cenários de Falha

| Cenário | Comportamento |
|---------|---------------|
| Falha ao publicar `domain-events` | Outbox retém; worker retenta |
| Redis indisponível | Fleet falha a transação; Kafka redelivery |
| Alert-service down | Lag em `domain-events`; read side (Redis) continua ok |
| Mensagem inválida | DLQ + offset commit |

---

## 12. Estimativa de Carga

| Parâmetro | Valor |
|-----------|-------|
| Veículos | 1.000 |
| Intervalo | 5 s |
| Throughput | ~200 eventos/s |

---

## 13. Próximos Passos

1. [roadmap.md](../product/roadmap.md)
2. [repository-layout.md](../development/repository-layout.md)
3. Fase 1 — Docker Compose + tópicos Kafka

---

## Apêndice — Diagrama de sequência

```mermaid
sequenceDiagram
    participant S as Simulador
    participant I as Ingestion
    participant T as telemetry-events
    participant F as Fleet Service
    participant PG as Event log
    participant R as Redis
    participant O as Outbox
    participant D as domain-events
    participant A as Alert Service
    participant Q as Query API

    S->>I: POST telemetry
    I->>T: publish
    T->>F: consume
    F->>PG: append domain event
    F->>R: update state
    F->>O: enqueue
    O->>D: publish domain-events
    D->>A: SpeedLimitExceeded
    A->>Q: push alert
    Q->>R: GET fleet
```

## Apêndice B — Evolução pós-MVP

1. `projection-service` — rebuild assíncrono de read models via `domain-events`
2. Event Sourcing completo — Redis materializado só por projeção (remover write direto no fleet)
3. gRPC na ingestão, Schema Registry, RabbitMQ para notificações externas
