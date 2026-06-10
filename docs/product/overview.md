---
title: Visão Geral do Produto
status: stable
last_updated: 2026-06-08
owners: [product]
---

# Visão Geral do Produto

Plataforma de rastreamento logístico e telemetria de frotas em tempo real, projetada para simular e processar grandes volumes de dados provenientes de veículos conectados.

O sistema permite a simulação de milhares de caminhões transmitindo simultaneamente coordenadas GPS, velocidade, status operacional e eventos de sensores, reproduzindo cenários reais encontrados em operações logísticas de larga escala.

## Principais funcionalidades

- **Simulação em escala** — Milhares de veículos conectados transmitindo dados continuamente
- **Geolocalização** — Transmissão contínua de latitude e longitude
- **Telemetria em tempo real** — velocidade, combustível, temperatura da carga, estado do motor, portas, manutenção
- **Processamento de eventos** em alta escala
- **Histórico de rotas** e trajetos
- **Dashboard operacional** para acompanhamento da frota
- **Sistema de alertas** e notificações
- **Métricas e observabilidade** da plataforma

## Caso de uso

Uma transportadora deseja monitorar sua frota em tempo real. Cada caminhão envia periodicamente informações de localização e telemetria para a plataforma **CargoTrack**. Os dados são processados instantaneamente, permitindo:

- Visualizar rotas em tempo real
- Identificar desvios de trajeto
- Detectar eventos críticos
- Gerar indicadores operacionais para otimizar a logística

## Cenário simulado

A plataforma representa operações logísticas de grande porte com:

- **1.000+** caminhões conectados
- Atualizações GPS a cada poucos segundos
- **Milhões** de eventos processados diariamente
- Rotas distribuídas por diferentes regiões
- Geração contínua de eventos críticos e operacionais

## Objetivos técnicos

| Área | Conceitos |
|------|-----------|
| Arquitetura | Microsserviços, Event-Driven Architecture (EDA) |
| Dados & IoT | Internet das Coisas, processamento de streams |
| Infraestrutura | Sistemas distribuídos, escalabilidade horizontal, alta disponibilidade |
| Operações | Observabilidade, monitoramento, CI/CD |

## Documentação relacionada

- [Visão do PO](./vision-and-story.md)
- [Roadmap de implementação](./roadmap.md)
- [System Design](../architecture/system-design.md)
