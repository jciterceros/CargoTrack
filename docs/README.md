# Documentação CargoTrack

**Stack:** Java/Spring Boot · Node.js (simulador) · Kafka · PostgreSQL · Redis · TimescaleDB

**Padrão:** CQRS leve + `domain-events` (Kafka: `telemetry-events` + `domain-events`)

| Documento | Descrição |
|-----------|-----------|
| [System Design](./system-design.md) | Arquitetura, componentes, fluxos, outbox |
| [Stack](./stack.md) | Decisão tecnológica e papéis dos tópicos Kafka |
| [Estrutura de Pastas](./folder-structure.md) | Monorepo — write/read side |
| [Modelo de Eventos](./event-model.md) | Telemetria, domain-events, glossário |
| [Roadmap](./roadmap.md) | Fases 1 → 6 (inclui Fase 4b: domain-events) |

## Referências internas

- [README do projeto](../README.md) — visão geral do produto
