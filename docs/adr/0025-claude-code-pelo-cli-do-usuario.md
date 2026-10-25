---
status: presumida
---

# Claude Code entra pelo CLI do próprio usuário, sem tocar em credenciais

A página de conformidade da Anthropic proíbe que terceiros ofereçam login claude.ai ou coletem, guardem ou intermediem credenciais de assinatura. A mesma página permite que o usuário entre no binário oficial do Claude Code com a própria assinatura. Por isso, quando o Claude Code entrar, o Visconde vai executar o `claude` que o usuário instalou e autenticou, via `claude -p --output-format stream-json`. O login é detectado com `claude auth status`. O Visconde não lê nem guarda tokens e não empacota o binário. A chave de API do Console é a segunda rota.

Ainda está em aberto se executar o Claude Code dentro de um produto exige os Commercial Terms. Antes de anunciar compatibilidade com a assinatura Claude, vamos pedir confirmação por escrito à Anthropic. Ver `docs/research/claude-subscription-licensing.md`.
