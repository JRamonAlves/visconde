---
status: aceita
---

# A 0.0.1 reutiliza o Codex CLI instalado e autenticado pelo usuário

O aplicativo detecta o binário `codex`, verifica o login e, se faltar, orienta o usuário a rodar `codex login`. Não gerencia credenciais. O login gerenciado com compartilhamento do plano ChatGPT exige registrar o cliente na OpenAI e fica para depois. A viabilidade técnica e jurídica de fazer o mesmo com o Claude Code está em pesquisa (ver ADR-0025).
