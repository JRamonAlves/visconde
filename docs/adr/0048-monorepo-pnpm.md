---
status: aceita
---

# Monorepo pnpm com o schema do mapa em pacote próprio

O repositório é um monorepo pnpm com `apps/desktop` (Electron), `packages/map-schema` (schema e validação, compartilhados pelo aplicativo e pelo MCP) e `packages/agent-adapters` (Codex agora, Claude Code depois). Com o schema isolado, a validação é testada sem subir o Electron.
