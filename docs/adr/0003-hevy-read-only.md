# ADR 0003 — Integração Hevy: read-only via REST

**Status:** Accepted  
**Data:** 2026-10-07  
**Owner:** Thiago

## Contexto

O usuário já usa o app Hevy para treinos e quer visualizar o treino do dia dentro do Rotina, sem duplicar o workflow de registro de treinos.

## Opções consideradas

| Opção | Prós | Contras |
|---|---|---|
| **Hevy API REST (read-only)** | Zero duplicação, dados sempre atuais, simples | Depende de API externa; Hevy pode mudar/deprecar |
| Banco de treinos próprio | Independente, offline total | Duplica workflow do usuário; migração de dados |
| Export CSV do Hevy | Sem dependência de API | Manual, não tempo-real |
| Integração bidirecional | Completa | Complexidade alta, risco de conflito de dados |

## Decisão

**Read-only via Hevy REST API.** O módulo `hevy` em Rust faz `GET /workouts` periodicamente (TTL 30min), armazena o resultado em `hevy_cache` (SQLite), e expõe via Tauri command.

A API key é armazenada em `settings` (texto simples no SQLite local) — aceitável para v1 dado o escopo local/pessoal. Para v2, considerar keyring do sistema.

## Consequências

- O app requer conexão à internet para buscar treinos (degradação elegante: mostra cache ou "sem treino disponível")
- Se a Hevy mudar o schema da API, só `hevy.rs` precisa ser atualizado (fronteira isolada)
- Não há escrita de volta ao Hevy — o usuário ainda registra treinos no app Hevy normalmente

## Trigger para revisitar

- Se a Hevy API exigir OAuth2 (hoje usa API key simples)
- Se a Hevy deprecar o endpoint `/workouts` ou exigir plano pago para acesso à API
- Se o usuário pedir registro de treinos dentro do Rotina (→ avaliar banco próprio + sync Hevy)
