---
status: proposta
---

# Estratégias para reduzir custo e tornar o mapa verificável

Ainda não escolhemos quais destas estratégias entram. A ADR-0029 já cobre a validação de âncoras.

1. Índice local de símbolos TypeScript, disponível ao agente como ferramenta MCP (`find_symbol`, `references_of`, `exports_of`, `entrypoints`). Não aparece na interface.
2. Validação local de toda âncora, com rejeição das que não resolvem. Aceita na ADR-0029.
3. Relações estruturais (imports, chamadas diretas, testes que importam símbolos) calculadas pelo compilador e marcadas como verificadas.
4. Orçamento de tokens ou de turnos por análise. Quando acaba, o restante fica marcado como não investigado.
5. Leitura guiada por contratos: manifesto e scripts, pontos de entrada, rotas e handlers, tipos e schemas exportados, e só depois as implementações.
6. Cache de explicações por âncora e hash, reaproveitado na revalidação.
7. Modelo ou esforço de raciocínio menor nas etapas mecânicas.

Recomendação inicial para a 0.0.1: 1, 2, 5 e 6.
