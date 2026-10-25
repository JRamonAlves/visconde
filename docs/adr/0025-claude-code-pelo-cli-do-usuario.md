---
status: presumida
---

# Claude Code entra pelo CLI do próprio usuário, sem tocar em credenciais

A página de conformidade da Anthropic proíbe que terceiros ofereçam login claude.ai ou coletem, guardem ou intermediem credenciais de assinatura. A mesma página permite que o usuário entre no binário oficial do Claude Code com a própria assinatura. Por isso, quando o Claude Code entrar, o Visconde vai executar o `claude` que o usuário instalou e autenticou, via `claude -p --output-format stream-json`. O login é detectado com `claude auth status`. O Visconde não lê nem guarda tokens e não empacota o binário. A chave de API do Console é a segunda rota.

Ainda está em aberto se executar o Claude Code dentro de um produto exige os Commercial Terms. Antes de anunciar compatibilidade com a assinatura Claude, vamos pedir confirmação por escrito à Anthropic. Ver `docs/research/claude-subscription-licensing.md`.

## Condições técnicas

O adapter do Claude Code é experimental e segue o mesmo contrato do Codex. Na sessão:

- abre com `--strict-mcp-config` e `--mcp-config`, e é abortada se o servidor MCP do Visconde não aparecer conectado em `system/init`, porque o CLI segue sem as ferramentas quando o servidor falha;
- limita as ferramentas a `Read`, `Grep` e `Glob` mais as do MCP, porque o sandbox do Claude Code cobre só Bash;
- usa `--setting-sources user` e desativa hooks, para não executar hooks nem servidores `.mcp.json` do repositório analisado;
- não usa `--bare`, que ignora o login por assinatura, nem `--system-prompt`; as instruções entram por `--append-system-prompt`;
- mostra ao usuário o `authMethod`, porque um `ANTHROPIC_API_KEY` no ambiente tem prioridade sobre a assinatura.

Os comandos não foram testados. Fixar uma versão mínima do CLI e detectar recursos em `system/init`.
