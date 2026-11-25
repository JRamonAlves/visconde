---
status: aceita
---

# O agente compõe a visualização com elementos que o aplicativo renderiza

O agente decide o conteúdo e a organização, mas só pode usar elementos que o aplicativo sabe renderizar e tornar exploráveis: blocos, relações e referências de código. Agrupamentos e sequências não são elementos próprios: um grupo é um bloco que `contém` outros, e uma sequência são blocos ligados por `precede` (ADR-0027). Gerar HTML ou componentes livres a cada análise foi rejeitado porque seleção, aprofundamento, referências e contestação deixariam de funcionar da mesma forma entre projetos.
