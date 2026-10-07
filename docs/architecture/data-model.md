# Data Model — SQLite

**Localização:** `~/.local/share/rotina/rotina.db`  
**Engine:** SQLite via `rusqlite` (Rust)  
**Migrations:** sequencial, versionada via tabela `schema_version`

---

## Entidades e esquema

### `settings`
Configurações do usuário. Uma row por chave.

```sql
CREATE TABLE settings (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL
);
-- Chaves relevantes:
-- pomodoro_focus_min       (padrão: 25)
-- pomodoro_break_min       (padrão: 5)
-- pomodoro_long_break_min  (padrão: 15)
-- pomodoro_cycles          (padrão: 4)
-- water_goal_cups          (padrão: 8)
-- water_reminder_min       (padrão: 30)
-- posture_sit_min          (padrão: 60)
-- posture_stand_min        (padrão: 15)
-- hevy_api_key             (padrão: null — opt-in)
-- window_x, window_y       (posição da janela)
-- font_size                (padrão: 14)
```

### `sessions`
Sessões de pomodoro e horas trabalhadas.

```sql
CREATE TABLE sessions (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    type       TEXT NOT NULL CHECK(type IN ('focus', 'break', 'long_break')),
    started_at TEXT NOT NULL,  -- ISO 8601
    ended_at   TEXT,           -- NULL se em andamento
    completed  INTEGER NOT NULL DEFAULT 0  -- 0=interrompido, 1=completo
);
```

### `water_logs`
Registro de copos bebidos.

```sql
CREATE TABLE water_logs (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    logged_at  TEXT NOT NULL,  -- ISO 8601
    cups       INTEGER NOT NULL DEFAULT 1
);
```

### `posture_logs`
Registro de ciclos posturais.

```sql
CREATE TABLE posture_logs (
    id           INTEGER PRIMARY KEY AUTOINCREMENT,
    event        TEXT NOT NULL CHECK(event IN ('sit_alert', 'stand_start', 'stand_alert', 'sit_start')),
    occurred_at  TEXT NOT NULL   -- ISO 8601
);
```

### `kanban_cards`
Cards do Kanban.

```sql
CREATE TABLE kanban_cards (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    title       TEXT NOT NULL,
    description TEXT,
    column      TEXT NOT NULL CHECK(column IN ('todo', 'in_progress', 'done')),
    position    INTEGER NOT NULL DEFAULT 0,  -- ordem dentro da coluna
    created_at  TEXT NOT NULL,
    updated_at  TEXT NOT NULL
);
```

### `hevy_cache`
Cache do treino do dia vindo da Hevy API.

```sql
CREATE TABLE hevy_cache (
    id          INTEGER PRIMARY KEY CHECK(id = 1),  -- singleton
    payload     TEXT NOT NULL,   -- JSON bruto da resposta Hevy
    fetched_at  TEXT NOT NULL    -- ISO 8601; TTL: 30 min
);
```

### `schema_version`
Controle de migration.

```sql
CREATE TABLE schema_version (
    version    INTEGER PRIMARY KEY,
    applied_at TEXT NOT NULL
);
```

---

## Ownership

| Tabela | Módulo dono | Leitores |
|---|---|---|
| `settings` | `settings` | todos os módulos (read-only) |
| `sessions` | `pomodoro`, `work_hours` | `work_hours` (agrega) |
| `water_logs` | `water` | — |
| `posture_logs` | `posture` | — |
| `kanban_cards` | `kanban` | — |
| `hevy_cache` | `hevy` | — |

---

## Decisões irreversíveis no modelo

1. **Timestamps em ISO 8601 (TEXT):** SQLite não tem tipo nativo `DATETIME`. ISO 8601 é ordenável lexicograficamente e portável.
2. **`hevy_cache` singleton (id=1):** simplifica UPSERT; o cache é sempre o último fetch bem-sucedido.
3. **`kanban` colunas fixas via CHECK:** evita coluna extra de configuração; extração de colunas dinâmicas é o trigger (ADR 0004).
