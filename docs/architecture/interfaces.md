# Interfaces — Contratos

## Tauri Commands (Rust → Frontend)

Todos os commands retornam `Result<T, String>` (erro serializado como string para o JS).

### Pomodoro

```typescript
invoke("pomodoro_start")      → void
invoke("pomodoro_pause")      → void
invoke("pomodoro_skip")       → void
invoke("pomodoro_reset")      → void
invoke("pomodoro_get_state")  → PomodoroState

interface PomodoroState {
  phase: "focus" | "break" | "long_break" | "idle"
  remaining_secs: number
  cycle: number        // 1–4
  total_cycles: number
}
```

### Horas trabalhadas

```typescript
invoke("work_hours_today") → number  // segundos acumulados hoje
```

### Água

```typescript
invoke("water_add_cup")       → void
invoke("water_get_today")     → WaterState

interface WaterState {
  cups_today: number
  goal: number
}
```

### Postura

```typescript
invoke("posture_get_state")   → PostureState
invoke("posture_confirm_standing") → void   // usuário confirmou que levantou
invoke("posture_confirm_sitting")  → void   // usuário confirmou que sentou

interface PostureState {
  mode: "sitting" | "standing"
  elapsed_secs: number
  alert_active: boolean
}
```

### Kanban

```typescript
invoke("kanban_get_cards")                        → KanbanCard[]
invoke("kanban_create_card", { title, description?, column }) → KanbanCard
invoke("kanban_move_card", { id, column, position })          → void
invoke("kanban_update_card", { id, title?, description? })    → void
invoke("kanban_delete_card", { id })                          → void

interface KanbanCard {
  id: number
  title: string
  description: string | null
  column: "todo" | "in_progress" | "done"
  position: number
  created_at: string
  updated_at: string
}
```

### Hevy

```typescript
invoke("hevy_get_today_workout") → HevyWorkout | null

interface HevyWorkout {
  name: string
  exercises: HevyExercise[]
  fetched_at: string
}

interface HevyExercise {
  name: string
  sets: HevySet[]
}

interface HevySet {
  reps: number
  weight_kg: number | null
}
```

### Configurações

```typescript
invoke("settings_get", { key })        → string | null
invoke("settings_set", { key, value }) → void
invoke("settings_get_all")             → Record<string, string>
```

### Sistema

```typescript
invoke("autostart_enable")   → void
invoke("autostart_disable")  → void
invoke("autostart_is_enabled") → boolean
```

---

## Tauri Events (Rust → Frontend, push)

```typescript
listen("timer_tick", (e: { remaining_secs: number, phase: string }) => ...)
listen("alert_pomodoro_done", (e: { phase: string }) => ...)
listen("alert_water", (_) => ...)
listen("alert_posture_sit", (_) => ...)    // hora de levantar
listen("alert_posture_stand", (_) => ...)  // hora de sentar
```

---

## Hevy API (REST, externa)

**Base URL:** `https://api.hevyapp.com/v1`  
**Auth:** `api-key: <hevy_api_key>` (header)  
**Uso:** read-only; chamado pelo módulo `hevy` em Rust

Endpoints utilizados:

| Endpoint | Uso |
|---|---|
| `GET /workouts?page=1&pageSize=10` | Lista workouts recentes para identificar o do dia |

> A Hevy API pública pode mudar. O módulo `hevy` isola toda a lógica de parsing — se o schema mudar, só `hevy.rs` é tocado.

---

## Contrato XDG Autostart

Arquivo criado em `~/.config/autostart/rotina.desktop`:

```ini
[Desktop Entry]
Type=Application
Name=Rotina
Comment=Gerenciador de rotina diária
Exec=/path/to/rotina
Icon=rotina
X-GNOME-Autostart-enabled=true
```

O path do executável é resolvido em runtime pelo módulo `autostart`.
