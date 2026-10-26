---
status: aceita
---

# O servidor MCP do Visconde roda em HTTP local, com token por sessão

O processo principal do Electron é dono do mapa e serve o MCP em `127.0.0.1`, numa porta aleatória e com um token por sessão. O agente fala direto com o processo que guarda o estado, sem um processo `stdio` intermediário. O token impede que outros programas da máquina escrevam no mapa.
