# Claude subscription use in a third-party open-source app

Research date: 2026-10-25. All pages below were retrieved on that date.

This is a reading of public documents, not legal advice. Facts (quotes and observed behavior) are separated from interpretation in each section. Anthropic's documents change often, so re-check the quoted pages before launch.

Scope: may Visconde, an open-source Electron app, let users connect a Claude Pro or Max subscription instead of an API key? Visconde does not build its own agent harness. It drives an existing agent.

## Verdicts

| Path | Verdict | One-line reason |
| --- | --- | --- |
| 1. Agent SDK in our app with subscription auth (OAuth login, `claude setup-token`, `CLAUDE_CODE_OAUTH_TOKEN`) | Prohibited as stated. Ambiguous only if the SDK inherits an existing CLI login and we never touch a credential. | Anthropic names "agents built on the Claude Agent SDK" and says third-party developers may not offer claude.ai login or route requests through plan credentials, and may not collect or store tokens. |
| 2. Spawn the user's own installed, logged-in `claude` CLI, our app never handles credentials | Likely allowed, not explicitly granted for third-party apps. | The legal page carves out signing in to the unmodified binary with a subscription, and lists conditions this path can meet. One preamble sentence is unclear for our case. |
| 3. API keys only, the user's own Console key | Allowed. | Anthropic tells developers to use API keys, and the SDK is licensed under the Commercial Terms for products shipped to end users. |
| 4. Open source vs commercial, personal use vs distribution | No document distinguishes open source from commercial. Distribution is where the restrictions bite. | The prohibitions are phrased around "third-party developers" offering something "for their products", not around license or price. |

## Facts

### F1. The legal page defines who may use which credential

Source: https://code.claude.com/docs/en/legal-and-compliance, section "Authentication and credential use".

> OAuth authentication is intended exclusively for purchasers of Claude Free, Pro, Max, Team, and Enterprise subscription plans and is designed to support ordinary use of Claude Code and other native Anthropic applications.

> Developers building products or services that interact with Claude's capabilities, including those using the Agent SDK, should use API key authentication through Claude Console or a supported cloud provider. Anthropic does not permit third-party developers to offer Claude.ai login into their own applications, or to route requests through Free, Pro, or Max plan credentials on behalf of their users. Moreover, developers may not collect, store, or intermediate Claude.ai credentials or session tokens — sign-in to a Claude account must complete through Anthropic's own flow.

> This does not restrict how customers provision and manage their own API keys or third-party inference provider credentials — for example, configuring an API key in a development environment, secrets manager, or machine image for use by the customer's own authorized users — provided the resulting usage is billed to the key owner under their agreement with Anthropic (or the applicable provider) and is not resold or intermediated as described above. Nor does it prevent an end user from signing in to the unmodified Claude Code binary with their own Claude subscription, including where a platform hosts Claude Code as described under *Can customers offer Claude Code in their products?* above.

> Anthropic reserves the right to take measures to enforce these restrictions and may do so without prior notice.

> Advertised usage limits for Pro and Max plans assume ordinary, individual usage of Claude Code and the Agent SDK.

### F2. The legal page also sets conditions for products that run Claude Code

Same page, section "Can customers offer Claude Code in their products?".

> Unless we've mutually agreed otherwise, preinstalling or running Claude Code in your products or services (e.g. in hosted sandboxes or other agent infrastructure) requires agreeing to our Commercial Terms of Service and complying with the conditions below:

> The Claude Code binary must not be modified. Claude Code must be installed and run as published by Anthropic, and customers may not remove, disable, or restrict any authentication method built into it (including methods that permit signing in with a Claude account or the user's own API key).

> Customers may not pay for, resell, or intermediate Claude usage on their end users' behalf. Each end user must authenticate with their own Anthropic API key, Claude subscription plan credentials, or 3P inference provider credential (Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry). That usage is billed directly to the end user under their own agreement with Anthropic or, for third-party inference providers, with the applicable provider.

Naming: "You can accurately say, in plain text, that your product has Claude Code preinstalled or that it runs Claude Code. But you can't use the Claude Code or Anthropic names or logos as part of your own product, feature, or company name, in your own logo, or in a way that suggests Anthropic built, endorses, or is partnered with your product."

### F3. The Agent SDK pages repeat the restriction

Sources: https://code.claude.com/docs/en/agent-sdk/overview and https://code.claude.com/docs/en/agent-sdk/quickstart.

> Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK. Use the API key authentication methods described in the [Quickstart] instead.

The quickstart lists API key, Bedrock, Vertex and Foundry as the SDK's authentication methods. It lists no subscription method.

Branding on the overview page: "Not permitted: 'Claude Code' or 'Claude Code Agent'". Allowed forms include "Claude Agent" and "{YourAgentName} Powered by Claude".

### F4. The SDK is licensed under the Commercial Terms

Sources: overview page and https://github.com/anthropics/claude-agent-sdk-typescript README, retrieved 2026-10-25.

> Use of the Claude Agent SDK is governed by Anthropic's Commercial Terms of Service, including when you use it to power products and services that you make available to your own customers and end users, except to the extent a specific component or dependency is covered by a different license as indicated in that component's LICENSE file.

The README footer reads "© Anthropic PBC. All rights reserved. Use is subject to Anthropic's Commercial Terms of Service." The package is not open source.

The TypeScript SDK reference (https://code.claude.com/docs/en/agent-sdk/typescript) says the SDK "bundles a native Claude Code binary for your platform as an optional dependency". `pathToClaudeCodeExecutable` defaults to "Auto-resolved from bundled native binary", so a default SDK install runs the bundled binary, not the user's installed CLI.

### F5. The authentication page shows the SDK can technically use a subscription

Source: https://code.claude.com/docs/en/authentication.

> `apiKeyHelper`, `ANTHROPIC_API_KEY`, and `ANTHROPIC_AUTH_TOKEN` apply to the CLI and the surfaces that wrap it, including the VS Code extension, the Agent SDK, and GitHub Actions.

> `CLAUDE_CODE_OAUTH_TOKEN` environment variable. A long-lived OAuth token generated by `claude setup-token`. Use this for CI pipelines and scripts where browser login isn't available.

> This token authenticates with your Claude subscription and requires a Pro, Max, Team, or Enterprise plan.

The SDK's `AccountInfo` type exposes `subscriptionType`, `tokenSource` and `apiKeySource` (TypeScript reference). The mechanism exists. The pages above say whether a third party may use it.

The headless page (https://code.claude.com/docs/en/headless) says `--bare` "doesn't use your subscription login" and "never reads OAuth credentials or the system keychain". It also says "In non-interactive mode (`-p`), the key is always used when present", so a stray `ANTHROPIC_API_KEY` in the environment overrides the subscription (authentication page).

### F6. The Help Center says subscription limits still apply to the SDK, `claude -p` and third-party apps

Source: https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan, last updated "over 2 weeks ago" on the retrieval date.

> Update October 7, 2026: Claude Max and Team plans now include monthly API credits, which cover the Claude Agent SDK, claude -p, the Claude API, and Claude Managed Agents. You can still use the Claude Agent SDK, claude -p, and third-party apps with your subscription limits.

> Update June 15, 2026: We've paused the previously-announced changes to Claude Agent SDK usage. For now, nothing has changed: Claude Agent SDK, claude -p, and third-party app usage still draw from your subscription limits.

The page does not define "third-party apps" and does not mention the legal page. A separate platform page (https://platform.claude.com/docs/en/about-claude/api-credits-for-subscribers) says the monthly credits for Max 5x, Max 20x and Team plans cover "Claude API, Claude Managed Agents, the Claude Agent SDK, and playground" and do not cover Claude Code. Pro and Enterprise plans are not eligible.

### F7. Consumer Terms, Commercial Terms and Usage Policy

Consumer Terms of Service, effective Oct 8, 2025 (https://www.anthropic.com/legal/consumer-terms), section 3:

> You may not access or use, or help another person to access or use, our Services in the following ways: ... Except when you are accessing our Services via an Anthropic API Key or where we otherwise explicitly permit it, to access the Services through automated or non-human means, whether through a bot, script, or otherwise.

Section 2: "You may not share your Account login information, Anthropic API key, or Account credentials with anyone else. You also may not make your Account available to anyone else."

Section 3 also bars using the Services to "resell the Services". The page names neither Claude Code nor the Agent SDK. It says "Our Commercial Terms of Service govern your use of any Anthropic API key".

Commercial Terms of Service, dated Jun 17, 2025 (https://www.anthropic.com/legal/commercial-terms):

> Services under these Terms are not for consumer use. Our consumer offerings (e.g., Claude.ai) are governed by our Consumer Terms of Service instead.

> D.4. Use Restrictions. Customer may not and must not attempt to (a) access the Services to build a competing product or service, including to train competing AI models or resell the Services except as expressly approved by Anthropic ...

Usage Policy (https://www.anthropic.com/legal/aup), section "Do Not Abuse Our Platform". The page served on 2026-10-25 shows "Effective Nov 12, 2026", a date after the retrieval date. I could not confirm which version is in force today or what changed from the prior one. The served text includes:

> Resell, proxy, or otherwise provide access to Claude through unauthorized means, including services that route requests through consumer subscriptions or misrepresent the product or client being used

### F8. Older wording and enforcement, from a secondary source

The Register (https://www.theregister.com/software/2026/02/20/anthropic-clarifies-ban-on-third-party-tool-access-to-claude/5014546) quotes the February 2026 version of the legal page as: "Using OAuth tokens obtained through Claude Free, Pro, or Max accounts in any other product, tool, or service ... is not permitted and constitutes a violation of the Consumer Terms of Service." It also quotes an Anthropic engineer, Thariq Shihipar, from a January 2026 thread: "Third-party harnesses using Claude subscriptions create problems for users and are prohibited by our Terms of Service".

This is secondary reporting and I could not retrieve the archived page or the thread. Treat it as history only. The current legal page no longer contains that sentence. It is softer and adds the carve-outs in F1.

### F9. Repository search

I searched issues in anthropics/claude-code, anthropics/claude-agent-sdk-typescript and anthropics/claude-agent-sdk-python for subscription, OAuth and third-party terms. I found no Anthropic staff statement on third-party apps. One open SDK issue, #355, reports the SDK's `subscription_oauth` source silently serving a different model than requested. It shows that SDK users run on subscriptions in practice. It is not a statement of permission. The SDK changelog has no entry about subscription policy.

## Path 1. Agent SDK with subscription authentication

Interpretation, not fact.

- Three designs fall under this path. Each lands differently.
  - Our app offers a "Log in with Claude" screen or runs the OAuth flow itself. This is the "offer Claude.ai login" case in F1 and F3. Prohibited.
  - Our app asks the user to run `claude setup-token` and paste the token, or stores `CLAUDE_CODE_OAUTH_TOKEN` for them. This is "collect, store, or intermediate Claude.ai credentials or session tokens" in F1. Prohibited.
  - Our app calls the SDK with `pathToClaudeCodeExecutable` set to the user's CLI, so the CLI uses its own existing login, and we never see a token. The text does not address this design directly. The F3 phrase "offer ... rate limits for their products, including agents built on the Claude Agent SDK" reads against it, because our product would consume the user's plan limits. The F6 phrase "third-party apps with your subscription limits" reads in favor. Ambiguous, and I lean toward "not authorized unless Anthropic approves".
- The default SDK install runs a bundled Claude Code binary. Shipping that inside an Electron installer is "preinstalling ... Claude Code in your products", which F2 says requires accepting the Commercial Terms.
- Anthropic keeps the right to enforce "without prior notice" (F1). The SDK is not open source (F4), so open-source packaging of our app does not change its terms.

Verdict: prohibited for the first two designs, ambiguous for the third.

## Path 2. Spawn the user's installed, logged-in `claude` CLI

Interpretation, not fact.

- The last sentence of F1 says the restriction does not "prevent an end user from signing in to the unmodified Claude Code binary with their own Claude subscription". The user signs in through Anthropic's own flow, which satisfies "sign-in to a Claude account must complete through Anthropic's own flow".
- F2 conditions map well onto this design. The binary is unmodified. We do not remove or restrict any login method. Each user authenticates with their own credentials and pays Anthropic directly. We never resell.
- Open point 1. The F2 preamble says "running Claude Code in your products or services ... requires agreeing to our Commercial Terms". The examples are hosted sandboxes and agent infrastructure, which differ from a local desktop app that launches the user's own binary. A strict reading could still apply it to us. Nothing in the text excludes it.
- Open point 2. Consumer Terms bar automated access "through a bot, script, or otherwise" unless "we otherwise explicitly permit it". Anthropic documents `claude -p` for scripts, and the Help Center says `claude -p` draws from subscription limits (F6). That reads as explicit permission for the user's own scripting. Whether it extends to a third-party app that scripts on the user's behalf is the same question as open point 1.
- "Advertised usage limits ... assume ordinary, individual usage" (F1). A tool that analyzes a whole repository in one run can use far more than a normal chat turn. This is a product risk for users, not a licensing prohibition.
- The CLI makes the requests, so the app does not misrepresent the client, which the AUP text in F7 warns about.

Verdict: likely allowed, based on the most specific language Anthropic has published. No sentence grants it by name to third-party apps.

## Path 3. API keys

Facts: F1 says developers "should use API key authentication through Claude Console or a supported cloud provider". F1 also says the restriction "does not restrict how customers provision and manage their own API keys", provided "the resulting usage is billed to the key owner". F4 says SDK use is governed by the Commercial Terms "including when you use it to power products and services that you make available to your own customers and end users". F7 says the Consumer Terms exempt access "via an Anthropic API Key" from the automation ban.

Interpretation: allowed. Constraints that still apply:

- The user brings a key from their own Console account. We never create keys, pool keys or pay for them. Commercial Terms D.4 bars reselling.
- Store the key locally, for example in the OS keychain, and send it only to the child process or the SDK. A Bedrock, Vertex or Foundry credential works the same way.
- Do not bundle or redistribute the Claude Code binary unless we accept the Commercial Terms for it. Use the user's installed binary, or have the user install it.
- Follow the naming rules in F2 and F3.

Verdict: allowed.

## Path 4. Open source vs commercial, personal use vs distribution

Interpretation, not fact.

- No Anthropic page above treats open-source and commercial products differently. The restrictions hinge on whether a "third-party developer" offers subscription login or limits "for their products" (F1, F3). A free GPL app is still a product.
- Commercial distribution adds exposure through Commercial Terms D.4 (resale, competing product) and the F2 preamble. Charging for an app that routes users through subscriptions would also look like "intermediating" (F1).
- Personal use is the clearest safe area. A developer who runs the SDK or `claude -p` with their own subscription for their own work matches "ordinary, individual usage of Claude Code and the Agent SDK" (F1) and the F6 statement. The legal page itself does not use the words "personal use" and sets no explicit exception for it.
- Distribution is where Path 1 turns prohibited. The same code is fine in a private script and risky as a published app that other people log in to.
- Contrast with OpenAI, which writes the open-source and local case into its docs (see below). Anthropic has no equivalent page.

## How T3 Code handles it

Evidence of practice, not of permission. Repo state: pingdotgg/t3code at commit e3592ef, dated 2026-10-25T21:32:35Z, read on the retrieval date. Earlier notes in `docs/research/t3-agent-integration.md` pin a different SHA, so line numbers there may differ.

- README: "Works with your subscriptions on Claude Code, Codex, Cursor, Grok Build, OpenCode, and Google Antigravity. If they're set up on your computer, T3 Code can control them." For Claude it tells users to "install Claude Code and run `claude auth login`".
- `docs/user/providers-claude.md`: "T3 Code uses Claude Code's login and configuration." It documents separate `CLAUDE_CONFIG_DIR` directories for multiple accounts and says "If your Claude subscription runs out of usage mid-turn, the thread shows which limit was reached."
- Code: the Claude adapter calls the Agent SDK with `pathToClaudeCodeExecutable` set to the configured `claude` binary path. I found no handling of `CLAUDE_CODE_OAUTH_TOKEN` or `claude setup-token` in `apps`, `packages` or `docs`. This is Path 1, third design, which F3 leaves ambiguous.
- I found no terms-of-service statement about Claude in the README or `docs/user`, and no matching issues in the repository search. That means no evidence either way about a private agreement with Anthropic. Prior notes already said the T3 setup does not transfer any authorization to us.
- T3 treats Codex differently. For Codex it offers a managed ChatGPT sign-in built on OpenAI's documented flow, plus the existing-CLI route.

## OpenAI comparison

Sources retrieved 2026-10-25: https://developers.openai.com/siwc/token-sharing-open-source, https://developers.openai.com/siwc/token-sharing-open-source/codex-app-server, https://developers.openai.com/codex/app-server.

Facts:

> ChatGPT plan usage is an optional capability within Sign in with ChatGPT. In addition to identity scopes, your open-source app can request permission to use the user's ChatGPT plan for eligible Responses API requests. This does not grant access to their ChatGPT conversations or other account context.

> These docs explain ChatGPT plan usage for open-source and locally hosted apps. If you're interested in offering it in a paid or remotely hosted app, complete the interest form.

The Codex app-server page documents how an app passes the plan's OAuth access token to `codex app-server` as `ACCESS_TOKEN` and sets `clientInfo.name` to identify itself. The general app-server page adds:

> If you've built a local or open-source application using Codex app-server authentication, you can continue using it, though we recommend migrating to Sign in with ChatGPT so users have greater control over and visibility into their usage. ... App-server authentication has never been permitted for commercial or hosted services.

Interpretation: OpenAI explicitly permits open-source and local apps to use a user's ChatGPT plan through a registered flow, and points commercial or hosted apps to a waitlist. Anthropic publishes no equivalent. The flow registers an OAuth client with OpenAI at sign-in, so "permitted" means "permitted through that flow". Using the user's existing `codex login` through the CLI is a separate route and the OpenAI auth page lists "Sign in with ChatGPT for subscription access" for the CLI itself.

## Conflicting signals

1. F1 and F3 prohibit third-party subscription login and plan routing. F6 says third-party apps "still draw from your subscription limits". The pages do not cite each other. One reading is that F6 describes what happens when a user runs their own CLI or SDK script, and F1 forbids third parties from offering it. That reading is mine, not Anthropic's.
2. The SDK is documented for building products for "your own customers and end users" (F4) and documented as unavailable to those end users on subscriptions (F3).
3. The Consumer Terms (Oct 2025) and Commercial Terms (Jun 2025) predate the current legal page. The AUP shown today carries a future effective date.

## Open questions

- Does "running Claude Code in your products" (F2) cover a local app that launches the user's own installed binary? If yes, does the user or the app author need Commercial Terms?
- Does "offer ... rate limits for their products" (F3) cover an SDK call whose executable path points at the user's own logged-in CLI?
- What does "third-party apps" mean in the Help Center (F6)?
- Which AUP version is in force on 2026-10-25, and did the "route requests through consumer subscriptions" wording change?
- Does heavy repository analysis fall inside "ordinary, individual usage"? Anthropic gives no number.
- Does T3 Code, or any other app that does this, have written approval from Anthropic? I found none public.

## Recommendation for the alpha

These are decisions I would make. They are not facts.

1. Make the default Claude path "bring your own Claude Code". Spawn the user's installed `claude` with `-p --output-format stream-json --verbose` and stdin or flags for the prompt. Detect login with `claude auth status`, which prints JSON and exits 1 when not logged in. Send the user to a terminal `claude auth login` if needed. Say plainly in the UI that the app runs the user's installed Claude Code.
2. Never handle the credential. Do not read the keychain or `~/.claude/.credentials.json`, do not accept pasted `setup-token` output, do not set `CLAUDE_CODE_OAUTH_TOKEN`, and do not add a login button. Do not use `--bare` for subscription users, since it ignores OAuth.
3. Do not bundle a Claude Code binary and do not depend on the SDK's bundled one. Prefer the CLI subprocess over `@anthropic-ai/claude-agent-sdk`, because the SDK adds the F3 wording and a bundled binary without adding anything Visconde needs.
4. Offer a second, plainly allowed route. Let the user give an Anthropic Console API key, stored in the OS keychain and passed only as `ANTHROPIC_API_KEY` to the child process. Show which method is active. Warn when a stray `ANTHROPIC_API_KEY` in the environment overrides the subscription.
5. Keep Codex as the path with explicit documentation for open-source apps. Ship Codex first if the alpha needs a route with no policy question. Use the registered Sign in with ChatGPT flow or the user's existing `codex login`.
6. Name and brand carefully. Do not put "Claude" or "Claude Code" in the app, feature or company name, or imply endorsement. Plain-text "runs Claude Code" is allowed.
7. Before the README says "works with your Claude subscription", ask Anthropic for written confirmation of Path 2 through the sales contact the legal page gives. Until then, describe subscription use as "uses your own installed and logged-in Claude Code" and list the API key route next to it.
8. Keep usage visible. Show per-run cost or token use from the CLI result message and avoid whole-repo re-analysis on every open, so users stay within what Anthropic calls ordinary use.
9. Stay non-commercial in the alpha. Revisit if Visconde ever charges money, hosts the agent, or pools keys or accounts.

## Sources and retrieval

All retrieved 2026-10-25 unless noted.

- https://code.claude.com/docs/en/legal-and-compliance
- https://code.claude.com/docs/en/agent-sdk/overview
- https://code.claude.com/docs/en/agent-sdk/quickstart
- https://code.claude.com/docs/en/agent-sdk/typescript
- https://code.claude.com/docs/en/authentication
- https://code.claude.com/docs/en/headless
- https://code.claude.com/docs/en/cli-reference
- https://www.anthropic.com/legal/consumer-terms, effective Oct 8, 2025
- https://www.anthropic.com/legal/commercial-terms, dated Jun 17, 2025
- https://www.anthropic.com/legal/aup, shows effective Nov 12, 2026
- https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan
- https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-or-max-plan
- https://platform.claude.com/docs/en/about-claude/api-credits-for-subscribers
- https://github.com/anthropics/claude-agent-sdk-typescript, README and CHANGELOG
- https://github.com/pingdotgg/t3code, README, `docs/user/providers-claude.md`, `docs/user/providers-codex.md`, Claude adapter source
- https://developers.openai.com/siwc/token-sharing-open-source and its codex-app-server and preview-limitations pages
- https://developers.openai.com/codex/app-server and https://developers.openai.com/codex/auth
- Secondary, history only: https://www.theregister.com/software/2026/02/20/anthropic-clarifies-ban-on-third-party-tool-access-to-claude/5014546

## Reutilizar o CLI do Claude Code como no Codex

Pesquisa de 2026-10-25, escrita depois das seções acima. Não é aconselhamento jurídico. Fatos (citações, comportamento documentado) ficam separados de interpretação. Não executei nenhum comando `claude` nesta pesquisa, para não tocar o login do usuário. Toda linha de comando abaixo vem da documentação e está **sem teste**.

Desenho avaliado: o Visconde 0.0.1 localiza o `claude` que o usuário já instalou e autenticou, inicia o processo sem modificá-lo, passa flags e uma configuração MCP própria, nunca lê nem guarda token, não embute o binário, é open source, gratuito e local.

### Resumo

- **Viabilidade técnica.** Viável, com lacunas. Há streaming bidirecional, retomada, interrupção, MCP por processo e restrição de ferramentas. Falta um contrato de protocolo estável como o do `codex app-server`, e a leitura somente-leitura depende de permissões, não de sandbox de sistema operacional.
- **Legal.** Ambíguo, com inclinação para permitido. O trecho "end user signing in to the unmodified Claude Code binary" cobre o núcleo do desenho. O que sobra é a frase sobre "running Claude Code in your products", cujos exemplos são infraestrutura hospedada.
- **Confirmação.** Vale pedir confirmação escrita à Anthropic antes de anunciar suporte a assinatura (pergunta de cinco linhas no fim da seção).

### 1. Viabilidade técnica

#### 1.1 Processo longo ou um processo por turno

Fatos, de https://code.claude.com/docs/en/cli-reference e https://code.claude.com/docs/en/headless.

- `--input-format` aceita `text` ou `stream-json` no modo `-p`. `--output-format stream-json` emite "newline-delimited JSON for real-time streaming". `--include-partial-messages` exige `--print` e `--output-format stream-json` e entrega tokens conforme saem, com `--verbose`.
- `--replay-user-messages` "Re-emit user messages from stdin back on stdout for acknowledgment" e exige os dois formatos `stream-json`.
- A página de streaming do SDK (https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode) descreve o modo de entrada em streaming como "a long lived process that takes in user input, handles interruptions, surfaces permission requests, and handles session management" e diz que ele permite "queued messages ... with ability to interrupt".
- Um processo por turno também é documentado. `--session-id` fixa um UUID para a conversa. `--resume <id>` retoma, `--fork-session` cria um novo ID ao retomar, e o `session_id` vem no resultado JSON. Exemplo oficial: `session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')` seguido de `claude -p "Continue that review" --resume "$session_id"`.
- `--no-session-persistence` desliga a gravação em disco, só no modo `-p`. Sem ele, as sessões ficam em transcrições do usuário e aparecem no `/resume` dele.

Lacuna. O formato das mensagens de entrada e das mensagens de controle (`control_request`, `control_response`, `initialize`, `interrupt`) aparece só na referência do SDK, em tipos como `SDKUserMessage`. A referência admite clientes que falam o protocolo direto: "A client that drives the CLI's control protocol directly, rather than through `interrupt()`...". Não achei uma especificação do protocolo do CLI como contrato separado do SDK. A saída do CLI avisa as capacidades da versão em `system/init.capabilities`, por exemplo `interrupt_receipt_v1`, e a documentação manda usar isso para detectar recursos em vez de comparar versões.

Interpretação. Um processo por turno com `--resume` usa só flags documentadas no CLI e é o caminho mais estável para o 0.0.1. Custa o tempo de partida e a conexão MCP a cada turno, até 30 segundos de espera por padrão (`MCP_TIMEOUT`). O modo de streaming bidirecional evita esse custo, mas amarra o Visconde a tipos do SDK que podem mudar entre versões do CLI.

#### 1.2 Interromper um turno

Fatos, de https://code.claude.com/docs/en/headless.

> If you stop a `claude -p` run with SIGTERM ... Claude Code leaves the turn that was in progress unfinished and records no result for it. To end the turn instead, send SIGINT, or call the Agent SDK's `interrupt()` before you stop the process.

No modo de streaming, o pedido de controle `interrupt` existe, devolve um recibo com as mensagens ainda na fila (`still_queued`) a partir da v2.1.205, e aceita `cancel_queued: true` a partir da v2.1.219 (referência do SDK em TypeScript).

Lacuna. O issue aberto https://github.com/anthropics/claude-code/issues/94741 relata que, depois de um `interrupt` por `control_request` numa sessão `stream-json`, o evento `result` chega sem o campo `result`. O Visconde deve tratar `result` ausente como turno interrompido, não como falha. No Codex, `turn/interrupt` termina o turno com `status: "interrupted"`.

#### 1.3 Registrar o MCP do Visconde sem tocar a configuração global

Fatos.

- `--mcp-config` carrega "MCP servers from JSON files or strings". Com `-p`, o Claude Code espera os servidores pendentes antes do primeiro turno.
- `--strict-mcp-config`: "Only use MCP servers from `--mcp-config`, ignoring all other MCP configurations."
- Um servidor `"type": "sdk"` só funciona para hospedeiros do SDK e o CLI o ignora num arquivo de configuração (https://code.claude.com/docs/en/mcp). O servidor do Visconde precisa ser `stdio` ou `http`, com `headers` para o token de sessão, como o T3 faz com o `/mcp` HTTP dele.
- O CLI relata entradas ignoradas no evento `system/init`, campo `mcp_server_errors`, e o estado de cada servidor em `mcp_servers`.
- Nomes de ferramenta MCP seguem `mcp__<servidor>__<ferramenta>`, e `mcp__<servidor>__*` libera todas as ferramentas do servidor (https://code.claude.com/docs/en/permissions).

Lacuna importante. Se a conexão falha, a corrida segue sem o servidor: "If all five fail, the run continues without the server's tools" (página MCP, linha sobre `claude -p`). No Codex, marcar o servidor como `required` faz `thread/start` falhar. Aqui o Visconde precisa ler `mcp_servers` no `system/init` e abortar o turno se o servidor do mapa não estiver `connected`.

Nada disso grava em `~/.claude.json` nem em `.mcp.json`. O processo recebe a configuração por argumento.

#### 1.4 Somente leitura

Fatos.

- `--tools` restringe as ferramentas embutidas: `"Read,Grep,Glob"` deixa só essas. A flag "doesn't affect MCP tools", então as ferramentas MCP seguem por `--strict-mcp-config` e, se preciso, `--disallowedTools "mcp__*"`.
- Em macOS e Linux, `Glob` e `Grep` ficam fora do conjunto padrão e voltam se você os nomear em `--tools` ou `--allowedTools` (https://code.claude.com/docs/en/tools-reference). Sem isso, o Claude busca por `find` e `grep` via Bash.
- `--disallowedTools`: "A bare tool name removes the matching tools from Claude's context: `"Edit"` removes Edit, `"*"` removes every tool".
- `--permission-mode dontAsk`: "Claude Code auto-denies every tool call that would otherwise prompt you". Leituras dentro dos diretórios de trabalho continuam. Desde a v2.1.302, `claude -p` sem modo definido já começa em Manual, que nega o que exige aprovação.
- `--permission-prompts none` (v2.1.259 ou mais nova) nega tudo que perguntaria e avisa o modelo para não repetir.
- O sandbox só envolve comandos de shell, vem desligado por padrão e não cobre as ferramentas de arquivo: "tools such as Read, Edit, Write, WebFetch, and WebSearch follow permission rules instead. A `denyRead` entry doesn't stop the Read tool" (https://code.claude.com/docs/en/sandboxing).
- `permissions.blockReadsOutsideWorkingDirectories` faz as ferramentas de arquivo recusarem caminhos fora dos diretórios de trabalho (referência de settings). Dá para passá-la em `--settings` como JSON inline.

Configuração herdada, também documentada. Um `claude -p` num repositório nunca confiado ainda usa hooks, o bloco `env`, helpers como `apiKeyHelper` e os servidores do `.mcp.json` do projeto (https://code.claude.com/docs/en/permissions, "What runs before you trust a folder"). A própria página manda, antes de rodar num repositório que você não escreveu:

> Pass `--setting-sources user`, or set the SDK's `settingSources` without project settings, so Claude Code reads neither the project's settings files nor its `.mcp.json`

> Pass `--settings '{"disableAllHooks": true}'` to turn hooks off for that run.

Como o Visconde analisa repositórios arbitrários, isso importa mais que a sandbox. Sem `--setting-sources user`, um repositório hostil executa hooks na máquina do usuário.

Proibido para assinatura. `--bare` "never reads OAuth credentials or the system keychain" (headless), então ele derruba o login do usuário. Não usar.

Incerto. `--safe-mode` desliga "MCP servers" entre outras coisas e mantém "Authentication". Não está documentado se ele também desliga servidores vindos de `--mcp-config`. Precisa de teste antes de depender dele. `--restricted` também existe (v2.1.248 ou mais nova), mas a documentação não diz como ele trata a autenticação por assinatura. Também precisa de teste.

Interpretação. A combinação abaixo é a mais próxima do "sandbox somente leitura" do Codex que a documentação permite. Ela limita o que o modelo pode chamar, não o que o processo pode fazer no sistema.

```bash
claude -p \
  --input-format stream-json --output-format stream-json --verbose --include-partial-messages \
  --session-id "<uuid>" \
  --tools "Read,Grep,Glob" \
  --allowedTools "Read" "Grep" "Glob" "mcp__visconde__*" \
  --permission-mode dontAsk --permission-prompts none \
  --strict-mcp-config --mcp-config '<json do servidor visconde>' \
  --setting-sources user \
  --settings '{"disableAllHooks": true, "permissions": {"blockReadsOutsideWorkingDirectories": true}}' \
  --append-system-prompt-file ./visconde-prompt.md
```

Sem teste. Para o modo um-processo-por-turno, troque a entrada por argumento de texto e use `--resume "<uuid>"` a partir do segundo turno.

#### 1.5 Saída estruturada

`--json-schema` (só com `-p`) devolve o resultado validado no campo `structured_output` da resposta `json`, e a resposta não traz o campo se o modelo recusou ou não produziu saída válida (headless). No desenho do Visconde, o mapa é escrito pelas ferramentas MCP, então a saída estruturada serve só para confirmar o fim de uma etapa. O Codex oferece `outputSchema` por turno em `turn/start`.

#### 1.6 Detectar instalação e login

Fatos, de https://code.claude.com/docs/en/cli-reference.

> `claude auth status` Show authentication status as JSON. Use `--text` for human-readable output. Exits with code 0 if logged in, 1 if not. ... The JSON's `authMethod` field is one of `none`, `claude.ai`, `oauth_token`, `api_key`, `api_key_helper`, or `third_party`

A documentação do comando lista só o método de autenticação e o diretório de configuração, sem token. Para instalação, `claude --version` ou a localização do binário, com caminho configurável como o T3 permite (`binaryPath`). Um login válido segundo o `auth status` não prova que o primeiro turno funciona, como já vale para o Codex.

Cuidado de cobrança. Em modo `-p`, "the key is always used when present" (https://code.claude.com/docs/en/authentication). Um `ANTHROPIC_API_KEY` no ambiente do Visconde faz o processo cobrar API em vez da assinatura. Mostrar `authMethod` e avisar, sem apagar a variável do ambiente do filho. Apagar variáveis de autenticação do filho se aproxima de "restrict any authentication method" (item 2 abaixo).

### 2. Comparação com o Codex app-server

| Capacidade | Codex app-server | Claude Code CLI | Lacuna |
| --- | --- | --- | --- |
| Contrato do protocolo | JSON-RPC por stdio, `generate-ts` e `generate-json-schema` | NDJSON com tipos do SDK e `system/init.capabilities` | Sem especificação do CLI separada do SDK. Fixar versão mínima e testar contrato. |
| Sessão longa | `thread/start`, `thread/resume` | `--input-format stream-json` ou um processo por turno com `--resume` | Equivalente. O processo por turno custa partida e conexão MCP. |
| ID da conversa | O servidor devolve `thread.id` | Escolhemos com `--session-id`, e `session_id` volta no resultado | Claude é mais simples. |
| Streaming | `item/agentMessage/delta` | `stream_event` com `--include-partial-messages` | Equivalente. |
| Interromper | `turn/interrupt`, `status: "interrupted"` | SIGINT ou `control_request` `interrupt`; SIGTERM perde o turno | Issue #94741 aberto. Tratar `result` ausente. |
| MCP por sessão | `-c mcp_servers...` ou config da thread; `required` falha o início | `--mcp-config` com `--strict-mcp-config` | O CLI segue sem o servidor se ele falha. Ler `system/init`. |
| Somente leitura | Política de sandbox read-only do sistema operacional | `--tools`, `--allowedTools`, `--disallowedTools`, `dontAsk` | Sem sandbox de sistema para Read, Grep e Glob. |
| Configuração herdada | `CODEX_HOME` e `-c` isolam | Hooks, plugins, CLAUDE.md e `.mcp.json` carregam por padrão | Exige `--setting-sources user` e `disableAllHooks`. `--bare` está vetado. |
| Saída estruturada | `outputSchema` em `turn/start` | `--json-schema` no resultado final | Equivalente para o que usamos. |
| Aprovações | Pedidos do servidor ao cliente | Negar tudo com `dontAsk`; `--permission-prompt-tool` ou `can_use_tool` se precisarmos | Não precisamos no 0.0.1. |
| Login e conta | `account/read`, `account/rateLimits/read` no protocolo | `claude auth status` fora do protocolo | Sem leitura de limites de uso no CLI que eu tenha achado. |
| Histórico | Threads no `CODEX_HOME` do usuário | Transcrições em `~/.claude`, visíveis no `/resume`; `--no-session-persistence` evita | Decidir se o Visconde grava na pasta do usuário. |
| Fork | `thread/fork` | `--fork-session` com `--resume` | Equivalente. |
| Versões | Esquema gerado por versão | Flags exigem versões (MCP wait v2.1.221, recibo de interrupt v2.1.205, `--permission-prompts` v2.1.259) | Definir versão mínima e recusar abaixo dela. |

### 3. Legal

#### 3.1 Textos aplicáveis

Fatos, de https://code.claude.com/docs/en/legal-and-compliance, recuperada em 2026-10-25. Os dois trechos já aparecem em F1 e F2. Repito o que decide esta questão.

Carve-out de F1:

> Nor does it prevent an end user from signing in to the unmodified Claude Code binary with their own Claude subscription, including where a platform hosts Claude Code as described under *Can customers offer Claude Code in their products?* above.

Preâmbulo de F2:

> Unless we've mutually agreed otherwise, preinstalling or running Claude Code in your products or services (e.g. in hosted sandboxes or other agent infrastructure) requires agreeing to our Commercial Terms of Service and complying with the conditions below:

Condições de F2, em resumo. Binário sem modificação ("installed and run as published by Anthropic"), nenhum método de autenticação removido, desativado ou restringido, e cada usuário autentica com a própria credencial e paga direto à Anthropic.

Restrição proibitiva de F1, que o desenho não toca: oferecer login claude.ai, rotear por credenciais de plano em nome do usuário, e "collect, store, or intermediate Claude.ai credentials or session tokens".

#### 3.2 Aplicação ao desenho, ponto a ponto

Interpretação.

| Elemento do desenho | Texto relevante | Resultado |
| --- | --- | --- |
| Binário que o usuário instalou, sem modificação | "must not be modified ... installed and run as published by Anthropic" | Atende. Não embutimos nem editamos o binário. |
| O usuário entra no fluxo da Anthropic | "sign-in to a Claude account must complete through Anthropic's own flow" | Atende. O Visconde só manda o usuário rodar `claude auth login`. |
| Nunca tocamos tokens | "may not collect, store, or intermediate" credenciais | Atende, se não lermos keychain, `.credentials.json` nem `setup-token`. |
| Cada usuário paga o próprio uso | "billed directly to the end user" | Atende. Não revendemos nem fazemos pool. |
| Nenhum método de login restringido | "may not remove, disable, or restrict any authentication method" | Atende, desde que não usemos `--bare` nem apaguemos variáveis de autenticação do ambiente do filho. |
| Open source, gratuito, local | Nenhum trecho distingue | Neutro. Reduz exposição comercial (D.4 dos Commercial Terms), não cria permissão. |

#### 3.3 Duas leituras do preâmbulo

Interpretação, não fato.

- **Leitura A, favorável.** O título da seção é "Can customers offer Claude Code in their products?" e os exemplos são "hosted sandboxes or other agent infrastructure". O carve-out cita explicitamente "where a platform hosts Claude Code". Se mesmo uma plataforma que hospeda o Claude Code não impede o usuário de entrar com a própria assinatura, um aplicativo local que apenas inicia o binário do próprio usuário é caso mais fraco que o hospedado. Quem roda o Claude Code é o usuário, na máquina dele.
- **Leitura B, restritiva.** A frase diz "running Claude Code in your products or services" sem excluir software local. Um aplicativo que inicia o processo e controla suas entradas pode ser "running Claude Code in your product". Nesta leitura, o autor do Visconde precisa aceitar os Commercial Terms.
- Inclino pela A, com risco residual real. Se a B vale, o custo para o projeto é baixo, porque as condições já estão atendidas e os Commercial Terms são aceitos por clique ao criar uma conta no Console (data de vigência 17 de junho de 2025). Os Commercial Terms dizem que "Services under these Terms are not for consumer use" e proíbem aceitar em nome de uma entidade sem autoridade para obrigá-la, então um mantenedor individual precisa pesar isso antes de aceitar.
- O texto atual não define "your products". Não há como resolver a questão só com a documentação.

Diferença para o caminho 1. Usar `query()` do SDK com `pathToClaudeCodeExecutable` apontando para o mesmo binário produz o mesmo efeito prático. O que muda é o texto. O SDK é nomeado em "including agents built on the Claude Agent SDK" (F3), e o carve-out nomeia o "unmodified Claude Code binary". Eu diria sem rodeio que o caminho CLI se apoia numa distinção textual que a Anthropic pode não pretender. Por isso o pedido de confirmação abaixo.

#### 3.4 `--append-system-prompt` e ferramentas MCP próprias mudam a análise?

Interpretação.

- Nenhum trecho de F1, F2 ou F3 menciona prompts de sistema ou MCP. A condição de F2 trata do binário e dos métodos de login. Flags documentadas do CLI (`--append-system-prompt`, `--mcp-config`) não modificam o binário. `--append-system-prompt` acrescenta ao prompt padrão e não o substitui, então o Claude Code continua como publicado. Evitar `--system-prompt`, que substitui o prompt padrão, reduz a dúvida.
- `--tools`, `--allowedTools` e `--strict-mcp-config` restringem ferramentas, não métodos de autenticação. Não os vejo como "restrict any authentication method".
- Marca. A página do SDK diz: "Your product should maintain its own branding and not appear to be Claude Code or any Anthropic product", e F2 permite dizer em texto simples que o produto "runs Claude Code". O prompt do Visconde não deve pedir ao modelo que se apresente como outro produto nem esconder que roda Claude. O AUP citado em F7 fala em "misrepresent the product or client being used", e o cliente aqui é o próprio Claude Code, que se identifica sozinho.
- Uso. "Advertised usage limits for Pro and Max plans assume ordinary, individual usage of Claude Code and the Agent SDK." Uma análise automática de repositório inteiro pode passar disso. É risco ao usuário, não proibição.

#### 3.5 Declarações oficiais e issues sobre GUIs de terceiros

- Nenhuma página oficial recuperada trata de interfaces gráficas de terceiros que envolvem o CLI. A Central de Ajuda diz que `claude -p` e "third-party apps" ainda usam os limites da assinatura (F6), sem definir o termo.
- Issues em anthropics/claude-code. Os resultados de busca relevantes são https://github.com/anthropics/claude-code/issues/17118 (pedido para usar Max no OpenCode, fechado como concluído em 2026-01-09, 410 comentários, nenhum de membro da Anthropic segundo `author_association`) e https://github.com/anthropics/claude-code/issues/98912 (assinatura para organizações sem fins lucrativos em ferramentas de terceiros, fechado como não planejado por um bot que escreveu "this doesn't appear to be about Claude Code"). O texto do segundo issue afirma que "subscription logins are restricted to Claude Code". Isso é a frase do autor do issue, não uma declaração da Anthropic. Nenhum issue traz resposta oficial sobre envolver o CLI.
- Evidência de prática, não de permissão. Issues do próprio repositório (por exemplo #94430, #89859 e #91400) mostram o aplicativo Claude Desktop falando com o CLI por `initialize`, `control_request` e processos `claude` de longa duração. Isso indica que o protocolo existe e que a Anthropic o usa em produto próprio. Não prova que terceiros podem usá-lo.
- O T3 Code usa o SDK com o binário do usuário (seção anterior). Também é prática, não permissão.

### 4. Veredito

- **Viável?** Sim, com ressalvas. Documentado: streaming, retomada, interrupção, MCP por processo, restrição de ferramentas, detecção de login. Lacunas: contrato de protocolo ligado ao SDK, bug de `result` após interrupt, servidor MCP que falha em silêncio, ausência de sandbox de SO para as ferramentas de arquivo, e configuração herdada que exige `--setting-sources user` e `disableAllHooks`.
- **Permitido?** **Ambíguo, com inclinação para permitido.** O núcleo do desenho cabe no carve-out de F1 e cumpre as condições de F2. A dúvida é o preâmbulo de F2.
- **Para o 0.0.1.** Implementar o adapter Claude atrás da mesma interface do Codex, marcado como experimental. Não anunciar "funciona com sua assinatura Claude" até receber resposta. Descrever como "usa o Claude Code que você já instalou e autenticou". Manter a rota por chave de API do Console como alternativa clara.
- **Testes antes de confiar.** Rodar a linha de comando acima numa máquina de teste e conferir `system/init` (`mcp_servers`, `apiKeySource`, `tools`), confirmar que `Bash`, `Edit` e `Write` não aparecem, que hooks de repositório não rodam e que `--safe-mode` e `--restricted` mantêm o login por assinatura.

### 5. Pergunta para a Anthropic

Enviar pelo contato de vendas que a página legal indica (https://www.anthropic.com/contact-sales). Texto em inglês, pronto para colar.

```text
Visconde is a free, open-source, local-only desktop app. It does not bundle Claude Code. It launches the user's own unmodified `claude` binary, already signed in by the user through Anthropic's flow, and passes documented flags (--mcp-config, --strict-mcp-config, --tools, --append-system-prompt). It never reads, stores or relays tokens.
Does this fall under the "end user signing in to the unmodified Claude Code binary with their own Claude subscription" language on code.claude.com/docs/en/legal-and-compliance?
Or does "running Claude Code in your products or services" require us to accept the Commercial Terms, and if so, is accepting them as an individual maintainer sufficient?
May we say in our README that Visconde works with a user's Claude Pro or Max subscription through their installed Claude Code?
Does any of this change if the app drives the CLI over `--input-format stream-json` and the control protocol instead of one `claude -p` process per turn?
```

### Fontes desta seção

Recuperadas em 2026-10-25.

- https://code.claude.com/docs/en/cli-reference
- https://code.claude.com/docs/en/headless
- https://code.claude.com/docs/en/permissions
- https://code.claude.com/docs/en/permission-modes
- https://code.claude.com/docs/en/sandboxing
- https://code.claude.com/docs/en/mcp
- https://code.claude.com/docs/en/tools-reference
- https://code.claude.com/docs/en/authentication
- https://code.claude.com/docs/en/agent-sdk/typescript
- https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode
- https://code.claude.com/docs/en/legal-and-compliance
- https://developers.openai.com/codex/app-server
- https://github.com/anthropics/claude-code/issues/17118, /98912, /94741, /94430, /89859, /91400
