# PRD — Rotina

**Status:** Draft v1  
**Data:** 2026-10-07  
**Autor:** Thiago (@thiagohf23)  
**Plataforma-alvo:** Pop OS (Linux)

---

## 1. Product Thesis

**Problema:** Desenvolvedores que trabalham longas horas sentados perdem o controle do tempo, esquecem de se hidratar, ignoram a postura e deixam a saúde física de lado — tudo porque não há um ponto central e não-intrusivo de consciência durante o trabalho.

**Solução:** *Rotina* é um widget desktop flutuante para Pop OS que vive por cima de todos os outros apps. Ele reúne num único lugar o controle de tempo (pomodoro + horas trabalhadas), saúde (água, postura, treinos) e organização leve (Kanban) — sem exigir que o usuário troque de contexto.

**Proposta de valor central:**  
> "Tudo que você precisa para trabalhar com saúde, sempre visível, sem atrapalhar."

---

## 2. Personas

**Persona única — Dev solo, trabalho intenso:**
- Trabalha 6–10h/dia sentado no computador
- Usa Pop OS como SO principal
- Já usa Hevy para treinos de academia
- Quer disciplina de rotina sem fricção (não quer abrir apps separados)
- Não quer sincronização em nuvem — privacidade e simplicidade local

---

## 3. Hero Flow

1. **Login no sistema** → Rotina sobe automaticamente no startup
2. Uma **mini barra flutuante** aparece no canto da tela (always-on-top), mostrando:
   - Relógio pomodoro (ex: `🍅 23:14`)
   - Horas trabalhadas no dia (ex: `⏱ 4h32m`)
   - Copos de água (ex: `💧 3/8`)
   - Status postural (ex: `🪑 sentado há 52min`)
3. Usuário **clica na barra** → painel lateral expande com:
   - Controles do pomodoro (iniciar / pausar / pular)
   - Controle de água (+ adicionar copo)
   - Timer postural (configura intervalo sentado/em pé)
   - Kanban leve (To Do / Em progresso / Feito)
   - Seção de treino do dia (importado do Hevy via API)
4. **Alertas não-intrusivos:**
   - Notificação do sistema (`notify-send`) + pisca visual na mini barra
   - Fim de pomodoro, lembrete de água, hora de levantar

---

## 4. Lean v1 Scope

### ✅ Dentro do escopo — v1

| Módulo | Funcionalidade |
|---|---|
| **Widget flutuante** | Mini barra always-on-top; expande ao clicar |
| **Pomodoro** | Timer 25/5 (padrão); configurável pelo usuário |
| **Horas trabalhadas** | Contador diário acumulado (sessões pomodoro + tempo livre) |
| **Água** | Meta diária configurável (padrão 8 copos); botão +1 copo |
| **Postura** | Intervalo configurável sentado → alerta para levantar; confirmação de retorno |
| **Kanban** | 3 colunas (To Do / Em progresso / Feito); cards simples com título |
| **Treinos (Hevy)** | Leitura via Hevy API — exibe treino do dia (nome + exercícios + séries/reps) |
| **Notificações** | `notify-send` para todos os alertas + indicador visual na barra |
| **Startup automático** | Entrada em `~/.config/autostart/` (XDG autostart) |
| **Persistência** | SQLite local (`~/.local/share/rotina/rotina.db`) |

### ❌ Fora do escopo — v1 (futuro)

- Sincronização em nuvem
- Relatórios/histórico gráfico
- Múltiplos usuários
- Escrita de treinos de volta ao Hevy
- App mobile
- Suporte a outras distros além do Pop OS

---

## 5. Stack Recomendada

| Camada | Tecnologia | Motivo |
|---|---|---|
| **UI** | **Tauri 2** (Rust + HTML/CSS/JS) | Leve (~5MB), nativo no Linux, suporte a always-on-top e transparência, sem runtime pesado |
| **Frontend** | HTML + CSS + Alpine.js ou Svelte | Simples, sem build complexo |
| **Backend/lógica** | Rust (Tauri commands) | Performance, acesso ao sistema, gestão de timers confiável |
| **Banco de dados** | SQLite via `rusqlite` | Local, zero configuração, robusto |
| **Notificações** | `notify-rust` ou chamada a `notify-send` | Nativo no sistema |
| **Integração Hevy** | HTTP REST (Hevy API pública) | Leitura de treinos sem dependência de app instalado |
| **Autostart** | Arquivo `.desktop` em `~/.config/autostart/` | Padrão XDG, compatível com Pop OS / GNOME |

---

## 6. Requisitos Não-Funcionais

- **Performance:** Menos de 100MB de RAM em idle; inicializa em < 3 segundos
- **Plataforma:** Pop OS 22.04 LTS+; GNOME 42+
- **Privacidade:** Zero telemetria; nenhum dado sai da máquina (exceto chamadas à Hevy API, opt-in)
- **Resiliência:** Timers persistem entre sessões (estado salvo no SQLite)
- **Acessibilidade:** Barra reposicionável pelo usuário; tamanho de fonte configurável
- **Distribuição:** Instalação via script bash ou AppImage

---

## 7. Critérios de Aceite (por módulo)

### Widget flutuante
- [ ] Aparece sobre todas as janelas em todos os workspaces
- [ ] Mini barra exibe pomodoro, horas, água e status postural em tempo real
- [ ] Clique expande painel; segundo clique (ou clique fora) fecha
- [ ] Posição salva entre reinicializações

### Pomodoro
- [ ] Timer padrão 25min foco / 5min pausa (configurável)
- [ ] Ciclos: 4 pomodoros → pausa longa (15min, configurável)
- [ ] Ao finalizar: notificação do sistema + animação na barra
- [ ] Horas trabalhadas acumula apenas tempo de foco

### Água
- [ ] Meta diária configurável (padrão: 8 copos de 250ml)
- [ ] Botão +1 copo no painel expandido
- [ ] Lembrete de água a cada N minutos (configurável, padrão: 30min)
- [ ] Progresso reseta à meia-noite

### Postura
- [ ] Intervalo configurável (padrão: alerta após 60min sentado)
- [ ] Ao disparar: notificação + destaque visual na barra
- [ ] Usuário confirma "estou em pé" → inicia timer de pé (padrão: 15min)
- [ ] Ao fim do tempo em pé: notificação para sentar e trabalhar

### Kanban
- [ ] Criar / mover / deletar cards via drag-and-drop ou botões
- [ ] 3 colunas fixas: To Do / Em progresso / Feito
- [ ] Cards têm título + descrição opcional
- [ ] Estado persiste no SQLite

### Treinos (Hevy)
- [ ] Usuário insere Hevy API key nas configurações
- [ ] App exibe treino planejado para hoje (ou próximo treino)
- [ ] Exibe: nome do treino, exercícios, séries e repetições
- [ ] Se não há treino hoje: exibe "Dia de descanso 💪"
- [ ] Dados atualizados a cada 30min ou manualmente (botão refresh)

### Startup
- [ ] Arquivo `.desktop` instalado em `~/.config/autostart/`
- [ ] Opção de desabilitar autostart nas configurações do app

---

## 8. Decisões

| Decisão | Escolha | Motivo |
|---|---|---|
| Stack | Tauri 2 + Rust | Leve, nativo Linux, sem Electron overhead |
| Dados | SQLite local | Privacidade, simplicidade, zero config |
| Treinos | Integração Hevy (read-only) | Usuário já usa Hevy; não duplicar workflow |
| Distro | Pop OS only (v1) | Foco; evitar complexidade multi-distro no início |
| Notificações | `notify-send` + visual | Padrão do ecossistema GNOME |
| Autostart | XDG autostart (`.desktop`) | Compatível com Pop OS / GNOME sem sudo |

---

## 9. Próximos Passos

1. `/gdesign` — desenhar a arquitetura de módulos e a estrutura do projeto Tauri
2. `/gplan` — decompor em tasks implementáveis
3. `Sprint 1` — widget flutuante + pomodoro + horas trabalhadas (esqueleto funcional)
