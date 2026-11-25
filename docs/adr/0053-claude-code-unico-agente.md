---
status: aceita
---

# Claude Code é o único agente

O Visconde integra apenas o Claude Code. O Codex sai do plano. Substitui o ADR-0016 (um agente por sessão, entre Codex e Claude Code), o ADR-0024 (Codex primeiro) e o ADR-0035 (Codex CLI instalado). A seção sobre Codex do ADR-0044 fica obsoleta. A integração segue o ADR-0025.

O motivo é prático: o projeto perdeu o acesso ao Codex. Aceitamos o risco de licença descrito no ADR-0025 e em `docs/research/claude-subscription-licensing.md`, mesmo sem um segundo agente como alternativa. Se esse risco pesar, a decisão volta a ser discutida.

O contrato de adapter do ADR-0018 continua, mas com uma única implementação.
