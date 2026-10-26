---
status: aceita
---

# Cada aprofundamento gera seu próprio documento técnico

O mapa tem um documento comportamental e um documento de aprofundamento para cada bloco comportamental que o usuário aprofundou. Cada documento de aprofundamento aponta para o bloco de origem. Quando o mesmo código aparece em dois aprofundamentos, os dois usam a mesma âncora, e o aplicativo reconhece que se trata do mesmo trecho. Com documentos pequenos, o agente lê só o que precisa e uma mudança de código revalida só os documentos afetados. Detalha o ADR-0026.
