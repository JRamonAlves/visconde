---
status: aceita
---

# Tipos de bloco e de relação vêm de listas fechadas e versionadas

O agente só usa os tipos de bloco e de relação que o aplicativo conhece, para que ícones, filtros, layout e validação funcionem igual em qualquer projeto. As listas começam pequenas, mas cobrindo bastante, e crescem em versões futuras do schema. Para isso, o snapshot registra a versão do schema que o produziu.

Lista inicial de blocos: `comportamento`, `ponto-de-entrada`, `contrato`, `símbolo`, `estado`, `externo`, `teste`. De relações: `implementa`, `chama`, `depende-de`, `lê`, `escreve`, `emite`, `consome`, `testa`, `precede`, `contém` e `outra` (ADR-0039).
