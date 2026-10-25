---
status: presumida
---

# Um adapter por agente e um servidor MCP interno por sessão

Seguindo o desenho do T3 Code, cada agente tem um adapter que traduz sessões, turnos e eventos para um contrato comum. O aplicativo oferece ao agente um servidor MCP próprio, registrado apenas nas sessões que ele cria, sem alterar a configuração global do usuário. A interface não conhece o protocolo de nenhum agente. Não trazemos a orquestração completa do T3. Ver `docs/research/t3-agent-integration.md`.
