# Documentação CargoTrack

> Plataforma de telemetria e rastreamento logístico · CQRS leve + Kafka

**Stack:** Java/Spring Boot · Node.js (simulador) · Kafka · PostgreSQL · Redis · TimescaleDB

---

## Por onde começar

| Perfil | Documento |
|--------|-------------|
| Negócio / PO | [Visão do produto](./product/vision-and-story.md) |
| Arquitetura | [System Design](./architecture/system-design.md) |
| Implementação | [Getting Started](./development/getting-started.md) |
| Contratos | [APIs REST](./reference/api/) · [Mensageria](./reference/messaging/) |

---

## Produto

| Documento | Descrição |
|-----------|-----------|
| [overview.md](./product/overview.md) | Visão geral, funcionalidades e caso de uso |
| [vision-and-story.md](./product/vision-and-story.md) | Storytelling e decisões de produto |
| [roadmap.md](./product/roadmap.md) | Fases 1 → 6 de implementação |

---

## Arquitetura

| Documento | Descrição |
|-----------|-----------|
| [system-design.md](./architecture/system-design.md) | Arquitetura, componentes, fluxos, outbox |
| [stack.md](./architecture/stack.md) | Stack tecnológica e papéis dos serviços |
| [decisions/](./architecture/decisions/) | Architecture Decision Records (ADRs) |

**Diagrama:** [architecture-overview.svg](./assets/diagrams/architecture-overview.svg)

---

## Domínio

| Documento | Descrição |
|-----------|-----------|
| [glossary.md](./domain/glossary.md) | Glossário de termos de negócio |
| [event-catalog.md](./domain/event-catalog.md) | Catálogo de eventos e mapeamentos |

---

## Referência técnica

### APIs

| Documento | Descrição |
|-----------|-----------|
| [rest-ingestion.md](./reference/api/rest-ingestion.md) | `POST /api/v1/telemetry` |
| [rest-query.md](./reference/api/rest-query.md) | Consultas REST + WebSocket |

### Mensageria

| Documento | Descrição |
|-----------|-----------|
| [telemetry-events.md](./reference/messaging/telemetry-events.md) | Contrato de telemetria bruta |
| [domain-events.md](./reference/messaging/domain-events.md) | Contrato de eventos de domínio |

### Modelos de dados

| Documento | Descrição |
|-----------|-----------|
| [postgres-event-log.md](./reference/data-models/postgres-event-log.md) | Event log + outbox |
| [redis-state.md](./reference/data-models/redis-state.md) | Estado atual da frota |
| [timescale-locations.md](./reference/data-models/timescale-locations.md) | Histórico de localização |

---

## Desenvolvimento

| Documento | Descrição |
|-----------|-----------|
| [getting-started.md](./development/getting-started.md) | Setup do ambiente local |
| [repository-layout.md](./development/repository-layout.md) | Estrutura do monorepo |
| [implementation-order.md](./development/implementation-order.md) | Ordem de implementação |

---

## Referências externas

- [README do projeto](../README.md) — porta de entrada do repositório
- [CONTRIBUTING.md](../CONTRIBUTING.md) — como contribuir
