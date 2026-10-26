---
status: aceita
---

# O agente ignora hooks, MCPs e configurações do repositório analisado

O Visconde abre repositórios desconhecidos. Se o agente carregasse hooks, servidores MCP ou configurações declaradas no projeto, um repositório malicioso executaria código assim que o usuário clicasse em "Analisar código". Por padrão, a sessão só carrega os arquivos de instrução do projeto. Quando o repositório tiver configuração própria de agente, o aplicativo avisa e o usuário pode ativá-la manualmente.

As flags do Claude Code estão no ADR-0025. As do Codex estão sendo pesquisadas.
