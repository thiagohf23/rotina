# ADR 0001 — Stack: Tauri 2 + Rust + Svelte

**Status:** Accepted  
**Data:** 2026-10-07  
**Owner:** Thiago

## Contexto

Precisamos de um app desktop leve para Pop OS que rode como widget flutuante always-on-top. As opções consideradas foram frameworks desktop para Linux com suporte a WebView ou UI nativa.

## Opções consideradas

| Opção | Prós | Contras |
|---|---|---|
| **Tauri 2 + Rust** | ~5MB binário, Rust nativo, always-on-top suportado, zero runtime externo | Curva de aprendizado Rust, WebView depende do sistema |
| Electron | Ecossistema JS amplo, fácil UI | ~150MB+ RAM em idle, bundle pesado (~100MB) |
| GTK4 + Python | 100% nativo GNOME, leve | UI complexa de construir, menos flexível visualmente |
| Flutter Linux | UI rica, hot reload | Suporte Linux ainda beta, bundle médio, sem always-on-top maduro |

## Decisão

**Tauri 2 + Rust** para o backend/lógica; **Svelte** para o frontend (compilado para HTML/JS estático).

Svelte escolhido sobre Alpine.js por: reatividade compilada (zero runtime), componentes bem estruturados para a UI complexa dos módulos, e TypeScript nativo.

## Consequências

- Desenvolvedor precisa aprender Rust básico (Tauri commands, `async`, `rusqlite`)
- WebView usa o WebKitGTK do sistema — garantido no Pop OS, mas não em distros sem GNOME
- UI pode ser estilizada com CSS puro / Tailwind sem restrições de GTK
- Binário final ~5–15MB; RAM em idle esperada < 80MB

## Trigger para revisitar

- Se Pop OS migrar para Wayland puro e o suporte a `always-on-top` do Tauri quebrar em Wayland
- Se o binário exceder 200MB RAM em idle por > 30 dias de uso real
