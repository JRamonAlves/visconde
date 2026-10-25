# Como o T3 Code integra agentes e o que aproveitar no Visconde

Pesquisa em 25/10/2026. Escopo: Electron local, macOS/Linux, visualização de código TSX, alpha0.0.1 somente leitura, visão semântica inicial e expansão técnica sob demanda. Este documento contém pesquisa e recomendações, sem implementação.

## Conclusão para a decisão

O T3 controla runtimes existentes por adapters. No código público consultado, Codex roda como subprocesso `codex app-server`, com mensagens estruturadas por stdin/stdout. Claude roda por `@anthropic-ai/claude-agent-sdk`, que inicia e controla o executável Claude Code. A interface comum do T3 cuida de sessões, turnos, eventos e respostas a pedidos de aprovação. O Visconde pode adotar essa separação sem construir o ciclo de execução de um agente. [Adapter comum][adapter], [processo Codex][codex-process], [execução Claude][claude-query].

Para a alpha, recomendo começar com Codex app-server e manter um segundo adapter previsto para Claude Code. Se a seleção entre ambos for requisito de lançamento, o caminho Claude precisa de validação específica de autenticação e suporte. O SDK usado pelo T3 prova viabilidade técnica; não transfere ao Visconde eventual autorização comercial do T3.

## Fonte e limites da inspeção

- O inventário dos nomes em `/Users/vtex-ramon/workspaces` não revelou checkout identificável como T3 Code. Não examinei configurações privadas do aplicativo.
- O [site oficial](https://t3.codes/) identifica `pingdotgg/t3code` como repositório. Clonei a fonte pública em `/tmp/visconde-t3-research.i9jzvn/t3code`, sem instalar dependências ou executar scripts do projeto.
- Revisão examinada: [`38162960213e4952d6d225c5054c3976234f41e6`](https://github.com/pingdotgg/t3code/commit/38162960213e4952d6d225c5054c3976234f41e6). Data do commit: `2026-10-25T13:44:35-07:00`. Todos os links de código abaixo fixam esse SHA.
- O registro público inclui Codex, Claude, Cursor, Grok, OpenCode, Antigravity, Pi, Muse e ACP Registry. Investiguei apenas Codex e Claude em detalhe. A presença de um driver não comprova paridade de recursos entre provedores. [Registro][drivers].
- Não comparei esse checkout com o aplicativo instalado, não executei os adapters e não usei metadados das ferramentas MCP desta conversa como evidência da implementação.
- Não li tokens, não alterei autenticação, threads ou configurações MCP reais. Conferi os caminhos ancestrais e o repositório vazio antes da escrita; não encontrei `AGENTS.md` adicional. As instruções fornecidas na conversa continuam aplicáveis.

## Conexão, transporte e responsabilidade do adapter

### Codex

O T3 escolhe `binaryPath` ou `codex`, monta argumentos começando por `app-server`, fornece ambiente e `CODEX_HOME` quando configurado e cria o processo. Seu cliente conecta a saída do filho à leitura do protocolo e a entrada do filho à escrita. O parser acumula fragmentos até a quebra de linha, inclusive mensagens divididas entre chunks. Portanto, não interpreta a interface visual de terminal. [Inicialização do processo][codex-process], [argumentos][codex-args], [pipes][codex-pipes], [parser][codex-parser].

O adapter envia `initialize` e `initialized` uma vez por conexão compartilhada. Cria conversas com `thread/start` e retoma com `thread/resume`, reenviando a configuração MCP da thread. [Handshake][codex-init], [criação e retomada][codex-resume].

A documentação oficial descreve JSON-RPC bidirecional sem o campo `jsonrpc`, com JSON por linha no transporte stdio padrão. Também documenta `thread/list`, `thread/read` e paginação de histórico. Listar ou ler uma conversa não equivale a retomá-la. `turn/start.outputSchema` permite solicitar resultado estruturado por turno. [OpenAI Docs: app-server](https://learn.chatgpt.com/docs/app-server).

### Claude Code

O caminho principal observado usa `query({ prompt, options })` do Agent SDK. A entrada é um `AsyncIterable` alimentado por fila; a saída também vira um stream assíncrono. As opções incluem modelo, mensagens parciais, diretório, ferramentas, permissões e caminho do executável. Em macOS/Linux, o T3 fornece um spawner com pipes e grupo de processos, para encerrar também subprocessos MCP e evitar filhos órfãos. [Query e registro MCP][claude-query], [entrada e saída][claude-stream], [opções][claude-options], [grupo de processos][claude-process].

Isso difere de afirmar que o T3 apenas executa `claude -p` e lê texto. A API usada pela aplicação é o SDK; o CLI continua sendo o runtime controlado por ele. Não auditei o código interno da dependência SDK nesta pesquisa.

Existe uma alternativa documentada para um adapter menor: `claude -p` com `--output-format stream-json --verbose --include-partial-messages`. Para resultado estruturado, a documentação exemplifica `--output-format json --json-schema`. Capture `session_id` e use `--resume <id>`. A documentação também distingue SIGINT, que interrompe o turno, de SIGTERM, que pode deixá-lo sem resultado registrado. Isso é uma possibilidade para o Visconde, não a descrição do adapter principal do T3. [Claude: execução programática](https://code.claude.com/docs/en/headless).

### Limite da abstração

`ProviderAdapterV2SessionRuntime` oferece `ensureThread`, `resumeThread`, `startTurn`, `interruptTurn`, `respondToRuntimeRequest`, `readThreadSnapshot` e stream de eventos. Recursos opcionais permanecem opcionais. Essa é a parte reutilizável do desenho: a UI não precisa conhecer RPCs do Codex nem mensagens do SDK Claude. Copiar a orquestração completa do T3 traria funções desnecessárias para a alpha, como subagentes, rollback e trabalho em segundo plano. [Contrato][adapter].

## Assinatura e autenticação

### Codex tem dois caminhos no T3

1. **CLI existente.** O usuário instala e autentica Codex na máquina. O T3 documenta `codex login` e reutilização dessa instalação. O driver distingue esse caminho de `setupMode: "managed"`. [Guia do T3][codex-guide], [seleção de modo][codex-driver].
2. **Instalação gerenciada com ChatGPT.** O T3 adquire o executável, prepara um diretório próprio e configura `openai_token_sharing` contra `https://api.openai.com/v1`, com Responses e WebSockets desabilitados. Um comando de autenticação consulta o broker de tokens do T3. O código remove overrides ambientais que poderiam redirecionar a credencial. A autorização usa PKCE, identificador persistente do host e escopos de compartilhamento do plano. [Runtime gerenciado][codex-managed], [autorização][codex-auth].

Logo, seria incorreto resumir a versão pública atual a "T3 nunca gerencia autenticação". Ele oferece as duas estratégias. O Visconde pode começar pela instalação autenticada do usuário e adiar o onboarding gerenciado.

A OpenAI documenta compartilhamento do plano ChatGPT para aplicações abertas e hospedadas localmente. Esse consentimento não concede acesso às conversas do ChatGPT. O fluxo inclui registro do cliente e identidade estável do host. [OpenAI Docs: visão geral](https://developers.openai.com/siwc/token-sharing-open-source).

O exemplo oficial de app-server fornece o access token da própria aplicação ao subprocesso por ambiente, identifica o aplicativo em `clientInfo` e manda renovar o token, reiniciar o processo e retomar a thread. Esse exemplo usa `env_key`; o T3 inspecionado usa um broker por comando. Não são implementações idênticas. Nunca usar a leitura dos tokens de outro aplicativo como atalho de onboarding. [OpenAI Docs: app-server com ChatGPT](https://developers.openai.com/siwc/token-sharing-open-source/codex-app-server).

### Claude exige distinguir execução técnica de autorização

O T3 consulta `initializationResult()` do SDK e obtém metadados como `subscriptionType`, `tokenSource` e `apiProvider`. O probe não envia prompt ao modelo. Isso mostra reconhecimento de uma instalação autenticada, inclusive por assinatura; não prova quais acordos autorizam o T3 a distribuí-la dessa forma. [Probe][claude-auth].

A documentação do Agent SDK afirma que terceiros não podem oferecer login ou limites de claude.ai em seus produtos sem aprovação prévia, e orienta autenticação por API. [Claude: Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview).

A página de conformidade também prevê produtos que executam o binário Claude Code sem modificações, preservando seus métodos de autenticação e deixando cada usuário pagar e autenticar seu próprio uso. Ela proíbe coletar, guardar ou intermediar credenciais claude.ai e esclarece que a restrição não impede o usuário de entrar no binário intacto com sua assinatura. [Claude: conformidade, oferta do CLI e credenciais](https://code.claude.com/docs/en/legal-and-compliance).

Minha leitura técnica é que controlar o CLI intacto, com autenticação concluída no fluxo da Anthropic, merece avaliação separada de oferecer um agente próprio via SDK com login claude.ai. A combinação exata proposta pelo Visconde precisa de confirmação com a Anthropic antes de prometer suporte por assinatura. Esta pesquisa não determina enquadramento jurídico nem presume acordo privado do T3.

## Threads, sessões e persistência

O código separa identidades que não devem virar uma única coluna no Visconde:

- `appThreadId`: conversa pertencente ao T3, com estado de apresentação e vínculo ao projeto.
- `providerInstanceId`: configuração/instância selecionada do provedor.
- `providerSessionId`: sessão de runtime gerenciada pelo T3. Não assumir que corresponde ao ID nativo da conversa.
- `nativeThreadRef.nativeId`: identificador nativo usado para continuar a conversa no provedor.

O mapeamento Codex preserva todos esses campos. O banco tem tabelas separadas para sessões, threads e turnos de provedor, além de eventos e uma relação entre sessões e threads do aplicativo. Portanto, o histórico apresentado não depende apenas de manter o processo vivo. [Mapeamento][native-ids], [persistência][persistence], [eventos persistidos][events-db], [vínculos][bindings].

No Codex, `thread/resume` recebe o ID nativo e usa `excludeTurns: true`. No Claude, o T3 aloca um ID para a conversa, passa `sessionId` na criação e `resume` na continuação. Depois de liberar um processo ocioso, usa turnos persistidos para decidir que precisa retomar a sessão. [Codex][codex-resume], [criação Claude][claude-create], [identidade Claude][claude-options], [recuperação Claude][claude-recovery].

O nome `providerSessionId` também aparece no registro de credenciais MCP, onde identifica um namespace de autorização gerado pelo próprio registro. Não confundir esse UUID com a sessão do CLI. [Registro MCP][mcp-registry].

Não confirmei um importador universal de todas as conversas externas do terminal ou dos aplicativos oficiais. As evidências acima estabelecem criação, vínculo e retomada das conversas controladas pelo T3. Para a alpha, manter apenas sessões criadas pelo Visconde evita depender de formatos privados e descoberta de históricos alheios.

## Streaming, cancelamento e aprovações

O Codex emite `item/agentMessage/delta`; o adapter acumula os deltas por turno/item. Pedidos como `item/commandExecution/requestApproval` viram registros pendentes, com decisão aguardada e evento para a aplicação. A resposta resolve o pedido correspondente. Cancelar chama `turn/interrupt` com os IDs nativos. [Deltas][codex-deltas], [aprovação][codex-approval], [resposta][codex-response], [interrupção][codex-interrupt].

No Claude, o T3 consome mensagens do SDK, chama `queryRuntime.interrupt()` para interromper e `close` para encerrar. `canUseTool` traduz pedidos em eventos próprios e aguarda respostas. O código distingue disponibilidade de ferramentas, pré-aprovação e modo de permissão; em leitura, reduz o conjunto de ferramentas e pode usar `dontAsk`. [Streaming e interrupção][claude-stream], [callback][claude-approval], [política de leitura][claude-readonly].

Para o Visconde, progresso parcial deve alimentar uma visualização provisória. Somente um turno concluído com saída validada deve substituir o snapshot persistente. Cancelamento, falha e conclusão precisam de estados distintos. Recomendo rejeitar qualquer tentativa de escrita na alpha, em vez de abrir uma aprovação que descaracterize a promessa de leitura.

## MCP: registro, duração, confiança e instalação

### MCP interno do T3

O aplicativo oferece um endpoint HTTP `/mcp`. O registro cria credencial aleatória de 32 bytes, mantém seu hash em memória e associa a autorização à thread, instância do provedor e capacidades. A janela padrão de atividade é de 24 horas. Uma chamada válida renova atividade; chamadas com credencial já expirada não a reativam. O runtime confiável pode atualizar sua atividade. Há revogação por thread, namespace e global. O gerenciador reutiliza credenciais compatíveis ou revoga e emite outra. [Registro e duração][mcp-registry], [validação e revogação][mcp-lifetime], [integração ao ciclo de sessão][mcp-lifecycle].

Para Codex, o T3 injeta `mcp_servers.t3-code` na configuração de criação/retomada da thread. Para Claude, prepara uma entrada HTTP e chama `setMcpServers()` após abrir a query. O comentário explica que isso envia credenciais por stdin, evitando argumentos do processo. Esses caminhos registram o servidor interno sem executar `codex mcp add` ou `claude mcp add` e sem reescrever a configuração global do usuário. [Configuração Codex][codex-mcp], [configuração Claude][claude-mcp], [registro dinâmico Claude][claude-query].

O endpoint exige autenticação. A implementação também verifica acesso nos handlers e rejeita operações de escrita para clientes externos autorizados apenas a ler. Anotações como `readOnlyHint` não substituem esses controles. [Middleware][mcp-http], [controle de acesso][mcp-access].

Existe ainda outro fluxo: agentes que o T3 não iniciou podem conectar ao `/mcp` do ambiente por OAuth e receber acesso aprovado pelo usuário. Isso é diferente da credencial interna de uma thread. A alpha do Visconde não precisa expor esse acesso externo. [Guia de agentes externos][outside-agents], [autenticação alternativa no middleware][mcp-http].

### MCPs externos do usuário

O override Codex substitui somente a entrada `t3-code`, preservando os outros servidores configurados. O guia do T3 orienta reiniciar a sessão após alterações em MCPs, plugins ou skills, mantendo a conversa. Não encontrei, nos caminhos examinados, evidência suficiente para descrever um instalador genérico de servidores MCP do T3. [Override][codex-mcp], [reinício de sessão][restart-guide].

No Codex, MCPs ficam em `~/.codex/config.toml` ou `.codex/config.toml` de projetos confiáveis; entradas podem apontar para comando local ou URL HTTP. No Claude, os escopos local e usuário ficam em `~/.claude.json`, enquanto projeto usa `.mcp.json`. A documentação Claude alerta que sessões headless/SDK carregam MCPs de projeto sem a pergunta interativa; `--strict-mcp-config` limita as fontes carregadas, sujeito a políticas gerenciadas. [OpenAI Docs: MCP](https://learn.chatgpt.com/docs/extend/mcp?surface=cli), [Claude: escopos e confiança MCP](https://code.claude.com/docs/en/mcp).

Configurar significa registrar comando, argumentos, URL, ambiente, escopo e autenticação. Instalar significa obter o executável e suas dependências. Um comando com `npx` ou `uvx` pode baixar código ao iniciar; uma URL remota não exige instalar o servidor localmente. Portanto, salvar uma entrada não prova que o servidor existe, iniciou ou é confiável. Essa distinção deve aparecer em qualquer futura configuração MCP do Visconde.

## Recorte recomendado para alpha0.0.1

As propostas abaixo são decisões de produto derivadas da pesquisa, não funções já implementadas no Visconde.

1. Manter a UI Electron separada do processo que controla os agentes. O adapter oferece detectar disponibilidade, iniciar/retomar análise, receber eventos, cancelar e encerrar. Preservar o runtime Codex ou Claude Code e suas ferramentas existentes.
2. Começar com o CLI autenticado pelo próprio usuário. Exibir provedor, versão, estado de conexão e erro acionável. Autenticação detectada e lista de modelos não substituem um turno real bem-sucedido. Adiar gerenciamento de instalações, múltiplas contas e OAuth próprio.
3. Criar um contrato de resultado semântico: áreas funcionais, responsabilidades, relações e evidências em arquivo/símbolo/linhas. Pedir detalhes técnicos apenas para o nó expandido. Exibir trechos TSX como evidência sob demanda, sem árvore de IDE e sem edição.
4. Persistir snapshot próprio fora do projeto analisado: versão do schema, identidade do projeto, revisão Git quando existir, hashes dos arquivos analisados, data, provedor/modelo, IDs nativos e resultado validado. A sessão do agente ajuda a continuar a análise; o snapshot permite reabrir a visualização sem chamar o modelo. Mudanças no código devem marcar evidências como desatualizadas.
5. Aplicar leitura por configuração e por capacidade, restrita ao projeto selecionado. Em Codex, usar política de filesystem somente leitura e negar escaladas. Em Claude, limitar ferramentas e negar escrita; modo de planejamento sozinho não constitui sandbox. Não copiar os modos permissivos do T3 como padrão. Validar a garantia em macOS e Linux antes de anunciá-la.
6. Controlar também hooks, plugins e MCPs herdados. Um subprocesso externo não se torna seguro apenas porque o prompt pede leitura. O próprio código T3 alerta que `mcp_servers={}` faz merge no Codex, não remove servidores existentes. Uma política vazia precisa ser verificada, não presumida. [Exemplo no T3][codex-mcp-off].
7. Começar sem marketplace MCP. O agente pode produzir resultado estruturado sem MCP próprio. Se a expansão semântica precisar consultar um índice local, adicionar depois um servidor Visconde com operações de leitura, escopo por projeto e credencial por sessão. A aplicação grava o snapshot validado; o agente não recebe ferramenta de escrita no projeto.

## O que ainda precisa de confirmação

- Autorização e suporte para a combinação específica de CLI intacto, controle programático e assinatura Claude no Visconde. As páginas oficiais distinguem cenários; não resolvem automaticamente este produto.
- Versões mínimas de CLI/SDK compatíveis com o subconjunto escolhido. Não executei testes de compatibilidade, consumo de assinatura, cancelamento ou retomada.
- Garantia de leitura diante de hooks, comandos, plugins e MCPs, inclusive quando o processo encerra inesperadamente. O código consultado não substitui uma validação do Visconde nos dois sistemas operacionais.
- Importação de históricos externos, paridade de MCP Apps e comportamento dos demais provedores. Estão fora da evidência estabelecida nesta pesquisa.
- A instalação privada atual pode divergir desse SHA público. Nenhuma conclusão aqui identifica seu código ou seus acordos comerciais.

[adapter]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/packages/provider-core/src/server/ProviderAdapter.ts#L543-L625
[drivers]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/builtInDrivers.ts#L18-L38
[codex-process]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L1535-L1577
[codex-args]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/codexLaunchArgs.ts#L5-L15
[codex-pipes]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/packages/effect-codex-app-server/src/_internal/stdio.ts#L13-L22
[codex-parser]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/packages/effect-codex-app-server/src/protocol.ts#L406-L430
[codex-init]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L1777-L1803
[codex-resume]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L6600-L6669
[claude-query]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L920-L999
[claude-stream]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L1021-L1140
[claude-options]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L1377-L1470
[claude-process]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L322-L349
[codex-guide]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/docs/user/providers-codex.md#L1-L26
[codex-driver]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/Drivers/CodexDriver.ts#L112-L139
[codex-managed]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/CodexManagedRuntime.ts#L22-L137
[codex-auth]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/CodexChatGptAuth.ts#L515-L540
[claude-auth]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/provider/ClaudeProvider.ts#L330-L392
[native-ids]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L837-L873
[persistence]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/persistence/Migrations/055_OrchestrationV2.ts#L128-L175
[events-db]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/persistence/Migrations/055_OrchestrationV2.ts#L16-L37
[bindings]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/persistence/Migrations/OrchestrationV2/ProviderSessionBindings.ts#L4-L27
[claude-recovery]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L9616-L9638
[claude-create]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L10479-L10528
[codex-deltas]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L4243-L4255
[codex-approval]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L5114-L5166
[codex-response]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L7227-L7263
[codex-interrupt]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L7072-L7092
[claude-approval]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L9169-L9255
[claude-readonly]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L2295-L2404
[mcp-registry]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/mcp/McpSessionRegistry.ts#L59-L136
[mcp-lifetime]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/mcp/McpSessionRegistry.ts#L140-L189
[mcp-lifecycle]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/ProviderSessionManager.ts#L437-L490
[codex-mcp]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.ts#L1326-L1354
[claude-mcp]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/orchestration-v2/Adapters/ClaudeAdapterV2.ts#L1538-L1568
[mcp-http]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/mcp/McpHttpServer.ts#L149-L181
[mcp-access]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/mcp/McpToolAccess.ts#L68-L115
[outside-agents]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/docs/user/outside-agents.md#L1-L76
[restart-guide]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/docs/user/composer.md#L224-L235
[codex-mcp-off]: https://github.com/pingdotgg/t3code/blob/38162960213e4952d6d225c5054c3976234f41e6/apps/server/src/textGeneration/CodexTextGeneration.ts#L40-L49
