---
title: Visão e Storytelling do Produto
status: stable
last_updated: 2026-06-08
owners: [product]
---

# Visão e Storytelling do Produto

O CargoTrack nasceu de uma dor operacional simples de descrever e difícil de resolver: a transportadora não tinha visibilidade em tempo real da frota para agir com velocidade quando algo crítico acontecia.

Com mais de 1.000 caminhões enviando telemetria contínua (GPS, velocidade, combustível, temperatura e eventos operacionais), o desafio não era apenas "construir um sistema", mas garantir decisão rápida com confiabilidade, escala e rastreabilidade.

## O problema que importava

No dia a dia da operação, atrasos de minutos na detecção de um desvio de rota, excesso de velocidade ou falha de temperatura podem gerar prejuízo financeiro, risco de segurança e quebra de SLA com clientes.

O objetivo do produto foi definido com clareza:

- receber telemetria em alto volume;
- manter estado atual por veículo com baixa latência;
- disparar alertas acionáveis em tempo real;
- preservar histórico e auditoria para análise posterior.

## A decisão central de produto

A principal decisão foi separar volume de valor no fluxo de eventos:

- `telemetry-events`: entrada de dados brutos em alta escala;
- `domain-events`: fatos de negócio validados e prontos para consumidores desacoplados.

Essa separação permitiu evolução sem acoplamento excessivo, com pipeline de ingestão robusto e distribuição limpa de responsabilidades.

## Como a arquitetura vira valor para o usuário

Com base no desenho de [system-design.md](../architecture/system-design.md), a jornada ficou assim:

1. O simulador (Node.js) envia telemetria para o `ingestion-service`.
2. O `ingestion-service` publica no Kafka (`telemetry-events`).
3. O `fleet-service` processa regras de negócio, grava event log no PostgreSQL e atualiza read models (Redis e TimescaleDB).
4. O `fleet-service` publica `domain-events` via outbox para consumidores como `alert-service`.
5. O `query-api` entrega consulta rápida e atualizações via WebSocket para operadores.

Traduzindo para resultado: o operador não espera "o sistema consolidar"; ele recebe estado e alertas enquanto a operação acontece.

## O que priorizamos no MVP

A estratégia foi pragmática: entregar valor cedo, sem superengenharia.

- Stack principal: Java/Spring Boot no backend e Node.js no simulador.
- Kafka como barramento de eventos.
- PostgreSQL para auditoria (event log append-only).
- Redis para estado atual da frota.
- TimescaleDB para histórico de localização.

No MVP, adotamos CQRS leve com projeções síncronas no `fleet-service` para reduzir complexidade inicial e acelerar entrega.

## Resultado esperado de negócio

Com essa abordagem, o CargoTrack entrega os pilares que a operação precisava:

- visibilidade em tempo real por veículo;
- alertas relevantes (velocidade, temperatura, desvio, offline);
- consulta consolidada da frota com baixa latência;
- rastreabilidade de eventos para auditoria e melhoria contínua.

## Evolução planejada

A base foi desenhada para crescer sem reescrita:

- projeções assíncronas dedicadas (`projection-service`) no pós-MVP;
- evolução de contratos e integrações (analytics, notificações externas);
- maturidade progressiva de observabilidade, segurança e operação.

Em resumo, a história do CargoTrack não é sobre adotar tecnologia complexa. É sobre usar arquitetura orientada a eventos para transformar dados de telemetria em decisão operacional rápida, confiável e escalável.
