---
status: aceita em parte
---

# Estratégias para reduzir custo e tornar o mapa verificável

Esta ADR lista as estratégias e registra quais entram na 0.0.1. A ADR-0029 já cobre a validação de âncoras.

1. Índice local de símbolos TypeScript, disponível ao agente como ferramenta MCP (`find_symbol`, `references_of`, `exports_of`, `entrypoints`). Não aparece na interface.
2. Validação local de toda âncora, com rejeição das que não resolvem. Aceita na ADR-0029.
3. Relações estruturais (imports, chamadas diretas, testes que importam símbolos) calculadas pelo compilador e marcadas como verificadas.
4. Orçamento de tokens ou de turnos por análise. Quando acaba, o restante fica marcado como não investigado.
5. Leitura guiada por contratos: manifesto e scripts, pontos de entrada, rotas e handlers, tipos e schemas exportados, e só depois as implementações.
6. Cache de explicações por âncora e hash, reaproveitado na revalidação.
7. Modelo ou esforço de raciocínio menor nas etapas mecânicas.

## Decisão

- Entram na 0.0.1: 1, 2, 4, 5 e 7. O índice de símbolos sobre o TypeScript compiler API (1) é a estratégia principal. O orçamento (4) é um teto de turnos, e não de tokens. O usuário escolhe o teto antes de iniciar a análise, partindo de um valor padrão calibrado nos projetos de avaliação (ADR-0022). O teto vale para toda análise: overview, aprofundamento, contestação e revalidação, cada uma com o seu valor padrão. Os turnos dos subagentes também contam. Quando o teto acaba, o que nunca foi lido fica marcado como não investigado. Numa revalidação, os blocos que faltou conferir continuam desatualizados, com a explicação antiga visível.
- Adiada: 6. Falta entender como reaproveitar explicações em texto livre.
- Rejeitada na forma proposta: 3. As relações que o compiler encontra são indicativos oferecidos ao agente pelo índice de símbolos, não relações inseridas no mapa como `verificada`. O agente decide se a relação entra e com qual confiança (ADR-0028).
- O modelo menor (7) roda as etapas mecânicas, como listar pontos de entrada e catalogar testes, em subagentes do Claude Code definidos pelo Visconde (`--agents`), cada um com o seu modelo. A interpretação de comportamento fica com o modelo principal da sessão. O usuário escolhe o modelo principal, e o padrão é o menor modelo disponível na assinatura (hoje o Haiku), para economizar o limite do usuário. Com o padrão, os subagentes usam o mesmo modelo, e o item 7 só faz diferença quando o usuário escolhe um modelo principal maior.
