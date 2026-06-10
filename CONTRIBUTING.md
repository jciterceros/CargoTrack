# Contribuindo com o CargoTrack

Obrigado pelo interesse em contribuir com o CargoTrack.

## Documentação

A documentação vive em `docs/` e segue esta estrutura:

| Pasta | Conteúdo |
|-------|----------|
| `docs/product/` | Visão de produto, roadmap |
| `docs/architecture/` | System design, stack, ADRs |
| `docs/domain/` | Glossário, catálogo de eventos |
| `docs/reference/` | Contratos de API, mensageria, dados |
| `docs/development/` | Guias para desenvolvedores |
| `docs/assets/` | Diagramas e templates |

### Convenções

- Arquivos em **kebab-case** (ex.: `event-catalog.md`)
- Metadados YAML no topo: `title`, `status`, `last_updated`, `owners`
- Uma fonte da verdade por tema — evite duplicar conteúdo; use links
- Decisões arquiteturais novas → ADR em `docs/architecture/decisions/`

### Atualizar o índice

Ao adicionar documentos, atualize `docs/README.md`.

## Código (quando disponível)

### Pré-requisitos

- Java 21, Node.js 20+, Docker

### Fluxo

1. Fork do repositório
2. Branch a partir de `main`: `feat/nome-da-feature` ou `fix/nome-do-bug`
3. Commits descritivos em português ou inglês (consistente por PR)
4. Pull request com descrição do que mudou e por quê

### Padrões

- Seguir a estrutura em [repository-layout.md](docs/development/repository-layout.md)
- Contratos de API/eventos conforme `docs/reference/`
- Testes para lógica de negócio e integrações críticas

## ADRs

Para propor uma decisão arquitetural:

1. Copie [adr-template.md](docs/assets/templates/adr-template.md)
2. Numere sequencialmente (`004-nome-da-decisao.md`)
3. Abra PR com status `proposed`
4. Após revisão, status muda para `accepted`

## Dúvidas

Abra uma issue descrevendo o contexto e o que precisa de esclarecimento.
