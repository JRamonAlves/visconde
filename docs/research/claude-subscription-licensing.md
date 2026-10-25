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
