---
status: aceita
---

# O aprofundamento é recursivo e termina no código

Qualquer bloco pode ser aprofundado, não só os comportamentais. Um bloco de um documento de aprofundamento gera o próprio documento de aprofundamento, que aponta para o bloco de origem. Amplia o ADR-0049, que previa aprofundar apenas blocos comportamentais.

A recursão termina no código. Ao criar cada bloco, o agente marca se ainda há o que aprofundar. Aprofundar um bloco folha abre o leitor de código, sem chamar o agente (ADR-0015). Uma regra fixa do aplicativo foi rejeitada porque o tamanho de um símbolo não diz se há detalhe a mostrar.
