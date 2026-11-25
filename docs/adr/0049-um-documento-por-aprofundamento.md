---
status: aceita
---

# Cada aprofundamento gera seu próprio documento

O mapa tem um documento comportamental e um documento de aprofundamento para cada bloco que o usuário aprofundou. Cada documento de aprofundamento aponta para o bloco de origem. Quando o mesmo código aparece em dois aprofundamentos, os dois documentos mostram o mesmo bloco, com o mesmo ID e a mesma âncora. Um bloco não pertence a um único documento. Com documentos pequenos, o agente lê só o que precisa e uma mudança de código revalida só os documentos afetados. Detalha o ADR-0026. Ampliado pelo ADR-0052, que permite aprofundar qualquer bloco.
