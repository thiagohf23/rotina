# Boundaries — Módulos e Fronteiras

## Estrutura de módulos (processo único)

Todos os módulos vivem no mesmo processo Tauri. A fronteira entre eles é **Rust trait / interface interna** — não há IPC, não há HTTP entre módulos. Extração para processo separado é possível via seam (ver abaixo).

```
src-tauri/src/
├── main.rs                  # entrypoint Tauri
├── db.rs                    # conexão SQLite, migrations
├── modules/
│   ├── pomodoro.rs          # lógica de timer + ciclos
│   ├── work_hours.rs        # acumulador de horas trabalhadas
│   ├── water.rs             # controle de copos + meta diária
│   ├── posture.rs           # timer postural sentado/em pé
│   ├── kanban.rs            # CRUD de cards e colunas
│   └── hevy.rs              # cliente HTTP Hevy API (opt-in)
├── notifications.rs         # wrapper notify-send / notify-rust
├── settings.rs              # leitura/escrita de configurações
└── autostart.rs             # gestão do arquivo .desktop XDG
```

## Responsabilidades por módulo

| Módulo | Responsabilidade | Persiste em |
|---|---|---|
| `pomodoro` | Estado do timer (foco/pausa/longa), ciclos, dispara notificação | `sessions` |
| `work_hours` | Acumula tempo de foco; reseta à meia-noite | `sessions` |
| `water` | Contagem de copos; meta configurável; timer de lembrete | `water_logs` |
| `posture` | Timer sentado → alerta → timer em pé → alerta; intervalos configuráveis | `posture_logs` |
| `kanban` | CRUD de cards (título + descrição) nas 3 colunas fixas | `kanban_cards` |
| `hevy` | Busca treino do dia via Hevy API; cache de 30min | `hevy_cache` |
| `notifications` | Envia `notify-send`; sem estado próprio | — |
| `settings` | Lê/escreve `settings` table; broadcast de mudanças | `settings` |
| `autostart` | Cria/remove `~/.config/autostart/rotina.desktop` | filesystem |

## Fronteiras e contratos

Cada módulo expõe **somente** Tauri commands e eventos — não há chamada direta entre módulos no frontend. O Svelte fala apenas com commands.

```
Frontend (Svelte)
    │
    │  invoke("pomodoro_start")
    │  invoke("water_add_cup")
    │  invoke("kanban_create_card", {...})
    │  listen("timer_tick")
    │  listen("alert_posture")
    ▼
Rust Core (Tauri commands)
    │
    ├── módulos (chamadas internas diretas, sem trait boundary extra no v1)
    └── SQLite (via db.rs)
```

## Seams — fronteiras para o futuro

| Seam | Forma barata agora (v1) | Extração futura | Trigger para extrair |
|---|---|---|---|
| **Timers** | `tokio::time` no processo Tauri | Processo daemon separado | Se GNOME suspender o processo Tauri ao minimizar, causando drift de timer > 5s medido |
| **Notificações** | Chamada direta a `notify-send` via `std::process::Command` | `notify-rust` async, depois D-Bus direto | Se taxa de notificações > 10/min criar lag perceptível |
| **Hevy cache** | Row `hevy_cache` no SQLite, TTL 30min | Worker separado com sync em background | Se API Hevy exigir WebSocket/streaming no futuro |
| **Kanban** | 3 colunas fixas no SQLite | Serviço de sync / colaborativo | Se usuário pedir multi-dispositivo ou colaboração |
