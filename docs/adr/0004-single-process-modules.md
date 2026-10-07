# ADR 0004 — Arquitetura: processo único, módulos internos

**Status:** Accepted  
**Data:** 2026-10-07  
**Owner:** Thiago

## Contexto

O Rotina tem 6 módulos funcionais (pomodoro, água, postura, horas, kanban, hevy). Precisamos decidir se eles vivem no mesmo processo ou em processos separados.

## Opções consideradas

| Opção | Prós | Contras |
|---|---|---|
| **Processo único, módulos Rust internos** | Simples, sem IPC, comunicação por chamadas de função, menos overhead | Se um módulo trava, derruba tudo |
| Processos separados (daemon por módulo) | Isolamento de falhas, atualizações independentes | IPC complexo (D-Bus / sockets), over-engineering para este escopo |
| Plugin Tauri por módulo | Modular, recarregável | Tauri plugins são para extensibilidade de terceiros, não para módulos internos |

## Decisão

**Processo único.** Todos os módulos vivem em `src-tauri/src/modules/` como arquivos Rust separados dentro do mesmo crate. Comunicação via chamadas de função Rust diretas — sem traits extras de abstração no v1.

As seams (boundaries) são documentadas em `docs/architecture/boundaries.md` com triggers explícitos para extração futura caso necessário.

## Consequências

- Compilação monolítica — mudança em qualquer módulo recompila o binário inteiro (aceitável: < 30s em Rust incremental)
- Sem isolamento de falhas entre módulos — um panic derruba o app (mitigação: `catch_unwind` nos handlers críticos)
- Mais simples de depurar e testar (sem mocks de IPC)

## Trigger para revisitar

- Se o timer de pomodoro apresentar drift > 5s por hora devido ao processo sendo suspenso pelo sistema
- Se um módulo precisar de permissões de sistema separadas (ex: acesso a hardware para um futuro sensor de postura)
- Se o app crescer para mais de 3 desenvolvedores ativos com conflitos frequentes de merge
