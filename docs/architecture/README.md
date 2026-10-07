# Architecture — Rotina

**PRD:** [`docs/prd/prd-rotina.md`](../prd/prd-rotina.md)  
**Status:** v1 — greenfield  
**Data:** 2026-10-07

---

## O que é o sistema

*Rotina* é um **app desktop Tauri 2** para Pop OS. É um processo único que roda como widget flutuante always-on-top, composto por um backend Rust e um frontend HTML/CSS/JS servido localmente pelo WebView do Tauri.

Não há servidor remoto. Não há sincronização. Toda a lógica de negócio vive em **Tauri commands** (Rust); o frontend é puramente reativo (Svelte).

---

## Índice de documentos de arquitetura

| Documento | Conteúdo |
|---|---|
| `README.md` (este) | Visão geral e índice |
| [`boundaries.md`](./boundaries.md) | Módulos, fronteiras e seams |
| [`data-model.md`](./data-model.md) | Esquema SQLite e ownership |
| [`interfaces.md`](./interfaces.md) | Contratos Tauri commands + Hevy API |

**ADRs:**

| # | Decisão |
|---|---|
| [0001](../adr/0001-tauri2-rust-stack.md) | Stack: Tauri 2 + Rust + Svelte |
| [0002](../adr/0002-sqlite-local-storage.md) | Persistência: SQLite local |
| [0003](../adr/0003-hevy-read-only.md) | Integração Hevy: read-only via REST |
| [0004](../adr/0004-single-process-modules.md) | Módulos: processo único, sem microserviços |

---

## Diagrama de alto nível

```
┌─────────────────────────────────────────────┐
│               Pop OS / GNOME                │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │         Tauri 2 App Window          │    │
│  │  (always-on-top, transparente)      │    │
│  │                                     │    │
│  │  ┌─────────────┐  ┌──────────────┐  │    │
│  │  │  WebView    │  │  Rust Core   │  │    │
│  │  │  (Svelte)   │◄─►  (commands) │  │    │
│  │  └─────────────┘  └──────┬───────┘  │    │
│  │                          │           │    │
│  │                   ┌──────▼───────┐   │    │
│  │                   │   SQLite DB  │   │    │
│  │                   │ (~/.local/…) │   │    │
│  │                   └──────────────┘   │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  notify-send ◄── Rust (alertas)             │
│  Hevy API    ◄── Rust (HTTP, opt-in)        │
└─────────────────────────────────────────────┘
```
