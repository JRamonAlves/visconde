---
status: aceita
---

# "Analisar código" inicia o agente; navegar é local

Clicar em "Analisar código" inicia o agente, que lê o repositório e compõe a visualização. Selecionar, navegar e reabrir detalhes já analisados são operações locais. Aprofundar, contestar e revalidar são ações explícitas que chamam o agente, mostram o andamento e podem ser canceladas.

Se o repositório já tem snapshot, "Analisar código" pergunta se o usuário quer retomar a análise, quando o snapshot está incompleto (ADR-0042), ou recomeçar do zero, o que descarta o mapa atual.

Na 0.0.1, essas quatro são as únicas ações que chamam o agente. Não há pergunta livre sobre um bloco; isso fica com o chat, adiado pelo ADR-0012.
