---
status: aceita
---

# A âncora identifica arquivo, símbolo e hash, e o aplicativo a valida

A âncora contém arquivo, símbolo qualificado (por exemplo `OrderService.reserve`), tipo de símbolo, hash do texto do símbolo e a faixa de linhas, que serve só como dica. O aplicativo resolve cada âncora localmente quando a recebe e rejeita as que não resolvem, devolvendo o erro ao agente. Se o arquivo muda e o hash continua igual, a âncora segue válida. Se o hash muda, a região fica desatualizada. Detalha o ADR-0023.
