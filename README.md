# CargoTrack

Plataforma de rastreamento logístico e telemetria de frotas em tempo real, projetada para simular e processar grandes volumes de dados provenientes de veículos conectados.

O sistema permite a simulação de milhares de caminhões transmitindo simultaneamente coordenadas GPS, velocidade, status operacional e eventos de sensores, reproduzindo cenários reais encontrados em operações logísticas de larga escala.

A solução foi desenvolvida para demonstrar conceitos de arquitetura distribuída, processamento de eventos em tempo real, ingestão massiva de dados IoT, monitoramento de frotas e análise operacional.

---

## Principais Funcionalidades

- **Simulação em escala** — Milhares de veículos conectados transmitindo dados continuamente
- **Geolocalização** — Transmissão contínua de latitude e longitude
- **Telemetria em tempo real**
  - Velocidade
  - Nível de combustível
  - Temperatura da carga
  - Estado do motor
  - Abertura de portas
  - Alertas de manutenção
- **Processamento de eventos** em alta escala
- **Monitoramento de localização** em tempo real
- **Histórico de rotas** e trajetos
- **Dashboard operacional** para acompanhamento da frota
- **Sistema de alertas** e notificações
- **Métricas e observabilidade** da plataforma

---

## Objetivos Técnicos

Este projeto demonstra conhecimentos em:

| Área | Conceitos |
|------|-----------|
| Arquitetura | Microsserviços, Event-Driven Architecture (EDA) |
| Dados & IoT | Internet das Coisas, processamento de streams |
| Infraestrutura | Sistemas distribuídos, escalabilidade horizontal, alta disponibilidade |
| Operações | Observabilidade, monitoramento, CI/CD |

---

## Fluxo da Solução

```
Caminhões Simulados
        │
        ▼
Gateway de Ingestão
        │
        ▼
Broker de Mensagens
        │
        ├── Serviço de Rastreamento
        ├── Serviço de Telemetria
        ├── Serviço de Alertas
        └── Serviço de Analytics
        │
        ▼
Banco de Dados
        │
        ▼
Dashboard em Tempo Real
```
---

## Cenário Simulado

A plataforma representa operações logísticas de grande porte com:

- **1.000+** caminhões conectados
- Atualizações GPS a cada poucos segundos
- **Milhões** de eventos processados diariamente
- Rotas distribuídas por diferentes regiões
- Geração contínua de eventos críticos e operacionais

---

## Stack Tecnológica

Documentação detalhada: [docs/](docs/README.md)

### Backend

- Java 21 · Spring Boot 3
- Spring Security · WebSocket
- Simulador de frota: **Node.js**

### Mensageria

- **Apache Kafka** — `telemetry-events` + `domain-events` (MVP)
- RabbitMQ — pós-MVP (notificações externas)

### Banco de Dados

- PostgreSQL (event log)
- TimescaleDB (histórico de rotas)
- Redis (estado atual da frota)

### Observabilidade

- Prometheus · Grafana (MVP)
- Loki · OpenTelemetry — pós-MVP

### Infraestrutura

| Fase | Tecnologias |
|------|-------------|
| **MVP (desenvolvimento local)** | Docker · Docker Compose |
| **Produção (planejado)** | Kubernetes · Nginx · GitHub Actions |

No MVP, Kafka, PostgreSQL, Redis e os serviços Java rodam via **Docker Compose**. **Kubernetes** entra na evolução para deploy em produção (HPA, Ingress, ambientes staging/prod). Detalhes: [system-design.md](docs/system-design.md) · [roadmap.md](docs/roadmap.md)

---

## Caso de Uso

Uma transportadora deseja monitorar sua frota em tempo real. Cada caminhão envia periodicamente informações de localização e telemetria para a plataforma **CargoTrack**. Os dados são processados instantaneamente, permitindo:

- Visualizar rotas em tempo real
- Identificar desvios de trajeto
- Detectar eventos críticos
- Gerar indicadores operacionais para otimizar a logística

---

## Visão Geral

**CargoTrack** demonstra a construção de uma plataforma moderna de telemetria e rastreamento logístico baseada em eventos (**CQRS leve + Kafka**), preparada para lidar com grandes volumes de dados e requisitos de processamento em tempo real.

---

## Licença

Este projeto é disponibilizado para fins educacionais e de demonstração de arquitetura.
