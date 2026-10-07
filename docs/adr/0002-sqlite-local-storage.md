# ADR 0002 — Persistência: SQLite local

**Status:** Accepted  
**Data:** 2026-10-07  
**Owner:** Thiago

## Contexto

O app precisa persistir configurações, sessões de pomodoro, logs de água, postura, cards Kanban e cache da API Hevy. O usuário explicitamente não quer sincronização em nuvem.

## Opções consideradas

| Opção | Prós | Contras |
|---|---|---|
| **SQLite (`rusqlite`)** | Zero config, único arquivo, queries relacionais, robusto | Sem sync nativo |
| JSON files | Mais simples que SQL | Sem transações, difícil de escalar, sem queries |
| PostgreSQL local | Relacional completo | Overhead de instalar/manter daemon local |
| `sled` (embedded key-value Rust) | Nativo Rust, rápido | Sem SQL, queries manuais, menos maduro |

## Decisão

**SQLite via `rusqlite`** em `~/.local/share/rotina/rotina.db`.

Migrations controladas por uma tabela `schema_version` interna — sem framework de migrations externo no v1 (complexidade não justificada para 6 tabelas simples).

## Consequências

- Schema deve ser versionado com cuidado — migrations irreversíveis (ALTER TABLE em SQLite é limitado)
- Backup é simples: copiar o arquivo `.db`
- Performance mais que suficiente para o volume esperado (< 10k rows em uso normal de 1 ano)

## Trigger para revisitar

- Se o usuário pedir sincronização multi-dispositivo → avaliar SQLite + sync (Turso, Litestream) ou migrar para backend remoto
- Se o volume de `sessions` exceder 100k rows e queries de agregação > 50ms
