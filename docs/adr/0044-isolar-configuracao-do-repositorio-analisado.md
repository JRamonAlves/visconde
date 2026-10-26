---
status: aceita
---

# O agente ignora hooks, MCPs e configurações do repositório analisado

O Visconde abre repositórios desconhecidos. Se o agente carregasse hooks, servidores MCP ou configurações declaradas no projeto, um repositório malicioso executaria código assim que o usuário clicasse em "Analisar código". Por padrão, a sessão só carrega os arquivos de instrução do projeto. Quando o repositório tiver configuração própria de agente, o aplicativo avisa e o usuário pode ativá-la manualmente.

As flags do Claude Code estão no ADR-0025.

## Codex

O Visconde usa o `CODEX_HOME` padrão do usuário, porque um diretório separado perde o login. No Codex, a configuração de projeto só é carregada quando o projeto está marcado como `trusted`. Por isso, o Visconde não grava nenhuma entrada de confiança para o repositório. Gravar `untrusted` também removeria o `AGENTS.md`. Cada thread abre com `sandbox: read-only`, `approvalPolicy: never`, hooks, plugins e apps desativados, e o MCP do Visconde declarado como `required`. O token do MCP é passado por `bearer_token_env_var`.

Ficam três limites:

- As skills do repositório (`.agents/skills`, `.codex/skills`) não dependem de confiança, e não há como desligá-las.
- Se o usuário já marcou o repositório, ou um diretório acima dele, como `trusted`, o Visconde não consegue impedir a camada de projeto. Nesse caso, ele só avisa.
- Nada disso foi testado na prática. Antes de depender dessa configuração, vale um teste de ponta a ponta. Ver `docs/research/codex-project-config-isolation.md`.
