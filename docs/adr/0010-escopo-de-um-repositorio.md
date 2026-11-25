---
status: aceita
---

# Escopo de investigação limitado a um repositório no alpha

Cada projeto corresponde a um repositório, incluindo seus pacotes internos. Contratos com sistemas externos aparecem no mapa como blocos do tipo `externo` (ADR-0027). O tipo já indica que o outro lado não foi investigado, e a confiança (ADR-0028) vale só para o código do repositório que fala com o sistema externo. Vários repositórios podem vir depois.
