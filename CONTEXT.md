# Visconde

Aplicativo local que usa um agente de código, com a assinatura do próprio usuário, para interpretar um repositório e apresentá-lo como um mapa navegável de comportamentos, relações e código.

## Análise

**Análise**:
Investigação do repositório feita pelo agente a pedido do usuário. A primeira análise produz o overview; as seguintes aprofundam ou revalidam regiões.
_Avoid_: Scan, indexação

**Overview**:
Resultado da primeira análise. Apresenta as principais responsabilidades, pontos de entrada e relações do projeto, com referências ao código e com as regiões não investigadas identificadas.
_Avoid_: Resumo, análise completa

**Aprofundamento**:
Análise de uma região já presente no mapa, pedida explicitamente pelo usuário, que acrescenta detalhe técnico sem perder a ligação com o comportamento de origem.
_Avoid_: Drill-down, expansão automática

**Região não investigada**:
Parte do repositório que o mapa reconhece existir, mas que o agente ainda não leu o suficiente para descrever.
_Avoid_: Lacuna, parte desconhecida

**Contestação**:
Texto do usuário associado a um bloco ou relação, apontando uma interpretação que considera errada. O agente confere o código e revisa o mapa.
_Avoid_: Correção manual, feedback

## Mapa

**Mapa**:
Representação do repositório produzida pelo agente e guardada pelo aplicativo. Contém blocos, relações e referências de código, vistos por diferentes visões.
_Avoid_: Diagrama, grafo, gráfico

**Visão**:
Forma de olhar o mesmo mapa num nível de detalhe. As visões compartilham os mesmos blocos e relações.
_Avoid_: Segundo mapa, aba

**Visão comportamental**:
Visão em que os blocos são etapas de comportamento do sistema, como "validar pedido" ou "reservar estoque". É a visão de entrada.
_Avoid_: Visão de negócio, visão semântica

**Visão técnica**:
Visão em que os blocos são elementos do código, como funções, módulos, chamadas e estado, ligados aos comportamentos que implementam.
_Avoid_: Visão de código, visão detalhada

**Documento comportamental**:
Documento único do mapa que contém a visão comportamental do repositório inteiro.
_Avoid_: Mapa principal, overview

**Documento de aprofundamento**:
Documento com a visão técnica de um único bloco comportamental, criado quando o usuário pede o aprofundamento desse bloco.
_Avoid_: Documento técnico, submapa

**Bloco**:
Unidade do mapa que representa um comportamento ou um elemento do código.
_Avoid_: Nó, card, caixa

**Relação**:
Ligação dirigida entre dois blocos com um significado declarado.
_Avoid_: Aresta, conexão, link

**Referência de código**:
Indicação do trecho de código que sustenta um bloco ou relação. É a evidência que o usuário confere no leitor de código.
_Avoid_: Link para o arquivo, citação

**Âncora**:
Identificação de um trecho de código pelo símbolo que ele define, e não pela posição no arquivo, de modo que sobreviva a edições que não alteram esse símbolo.
_Avoid_: Número de linha, posição

## Snapshot

**Snapshot**:
Estado persistido do mapa, associado à versão do código analisada.
_Avoid_: Cache, histórico

**Snapshot desatualizado**:
Snapshot cujo código analisado mudou depois da análise. Continua visível, com as regiões afetadas marcadas, até o usuário pedir a revalidação.
_Avoid_: Snapshot inválido

**Revalidação**:
Análise que parte das diferenças entre a versão analisada e a atual e se estende às regiões que dependem delas.
_Avoid_: Reanálise completa, refresh

## Modos

**Modo exploração**:
Uso do aplicativo para entender um repositório sem pedir mudanças.
_Avoid_: Modo leitura, modo visualização

**Modo edição**:
Uso do aplicativo para pedir alterações ao agente e acompanhar seus impactos no mapa.
_Avoid_: Modo escrita

**Pedido de alteração**:
Instrução do usuário, associada a blocos ou relações selecionados, para que o agente mude o código.
_Avoid_: Prompt, tarefa

**Impacto**:
Consequência de uma alteração sobre o restante do repositório. Classifica-se como alterado, potencialmente afetado ou verificado.
_Avoid_: Efeito colateral, blast radius
