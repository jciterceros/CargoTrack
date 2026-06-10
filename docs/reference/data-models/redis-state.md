---
title: Modelo de Dados — Redis
status: stable
last_updated: 2026-06-08
owners: [architecture]
---

# Redis — Read Model (estado atual)

Projeção quente atualizada **sincronamente** pelo `fleet-service`.

---

## Estado do veículo

```
Key: vehicle:{vehicleId}:state
```

| Campo | Descrição |
|-------|-----------|
| `lat` | Latitude |
| `lng` | Longitude |
| `speed` | Velocidade atual |
| `fuel` | Nível de combustível |
| `temp` | Temperatura da carga |
| `engine` | Estado do motor |
| `doors` | Estado das portas |
| `alerts` | Alertas ativos |
| `updatedAt` | Timestamp da última atualização |

---

## Alertas ativos

```
Key: alerts:active
```

Mantido pelo `alert-service` a partir de `domain-events`.

---

## Papel no CQRS

| Lado | Operação |
|------|----------|
| Write (`fleet-service`) | UPDATE após processar telemetria |
| Read (`query-api`) | GET para consultas de frota |
