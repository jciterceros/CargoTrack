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

### Backend

- Java
- Spring Boot
- Spring Cloud
- Spring Security
- WebSocket

### Mensageria

- Apache Kafka
- RabbitMQ

### Banco de Dados

- PostgreSQL
- TimescaleDB
- Redis

### Observabilidade

- Prometheus
- Grafana
- Loki
- OpenTelemetry

### Infraestrutura

- Docker
- Kubernetes
- Nginx
- GitHub Actions

---

## Caso de Uso

Uma transportadora deseja monitorar sua frota em tempo real. Cada caminhão envia periodicamente informações de localização e telemetria para a plataforma **CargoTrack**. Os dados são processados instantaneamente, permitindo:

- Visualizar rotas em tempo real
- Identificar desvios de trajeto
- Detectar eventos críticos
- Gerar indicadores operacionais para otimizar a logística

---

## Visão Geral

**CargoTrack** demonstra a construção de uma plataforma moderna de telemetria e rastreamento logístico baseada em eventos, preparada para lidar com grandes volumes de dados e requisitos de processamento em tempo real.

---

## Licença

Este projeto é disponibilizado para fins educacionais e de demonstração de arquitetura.
