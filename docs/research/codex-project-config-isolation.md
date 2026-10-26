# Isolamento da configuração de projeto do Codex

Data: 2026-10-26. Contexto: ADR 0044 (o agente não carrega hooks, servidores MCP nem configuração de agente declarados pelo repositório analisado; só carrega `AGENTS.md`).

## Versão inspecionada

- Código-fonte: `openai/codex`, branch principal, commit `970111f6cb62cc02fe36a82ea7d71b37a2889c97` (2026-10-23), clonado em `/tmp` e lido em `codex-rs/`.
- CLI instalada na máquina: `codex-cli 0.164.0`. Não verifiquei se ela corresponde ao commit acima. Todo comportamento abaixo precisa de um teste de runtime contra o binário que o app vai distribuir ou exigir.
- Documentação: `developers.openai.com/codex/config-advanced` responde `308` para `https://learn.chatgpt.com/docs/config-file/config-advanced`. Usei a URL nova. As páginas `https://learn.chatgpt.com/docs/mcp` e similares que tentei retornaram 404, então os fatos sobre MCP vêm do código-fonte, não da documentação.

## Método e limites

Li o código do loader de configuração (`codex-rs/config/src/loader/mod.rs`), a descoberta de hooks, AGENTS.md, skills, exec policy, o protocolo do app-server e a configuração MCP. Não executei o app-server. Itens marcados como "não verificado" dependem de teste.

## 1. O que o Codex carrega de um repositório

### `.codex/config.toml` e camadas de projeto

Documentação: "Codex walks from the project root to your current working directory and loads every `.codex/config.toml` it finds." e "For security, Codex loads project-scoped config files only when the project is trusted." (https://learn.chatgpt.com/docs/config-file/config-advanced).

Código: `discover_project_layers` (`config/src/loader/mod.rs`, ~linha 1838) percorre de `cwd` até a raiz do projeto (marcador `.git` por padrão) e cria uma camada por diretório `.codex/`. O comentário em `mod.rs:133` diz "loaded but disabled when untrusted". A camada existe, mas fica desabilitada (`disabled_reason`) quando o projeto não é confiável. A mensagem de desabilitação cita o escopo gateado: "project-local config, hooks, and exec policies".

Regra de confiança (`decision_for_dir`, `disabled_reason_for_decision`): só `trust_level = "trusted"` em `projects."<caminho>"` do config efetivo habilita a camada. Ausência de entrada e `untrusted` explícito desabilitam a camada. A diferença entre os dois aparece em AGENTS.md (ver abaixo).

Chaves removidas mesmo em projeto confiável (`PROJECT_LOCAL_CONFIG_DENYLIST`): `openai_base_url`, `chatgpt_base_url`, `apps_mcp_product_sku`, `responses_api_metadata`, `model_provider`, `model_providers`, `notify`, `profile`, `profiles`, `experimental_realtime_*_base_url`, `otel`. A documentação diz: "Codex ignores the following keys in project-local `.codex/config.toml` and prints a startup warning when it sees them". `mcp_servers` e `hooks` não estão na lista, então um projeto confiável pode declarar ambos.

### Servidores MCP do projeto

`mcp_servers` em `.codex/config.toml` de um projeto confiável carrega como qualquer outro servidor (`McpServerConfig` em `config/src/mcp_types.rs`). Não existe arquivo `.mcp.json` lido pelo Codex que eu tenha encontrado. Plugins também podem trazer servidores MCP (`core/src/config/mod.rs`, ~1864).

### Hooks

Documentação: "Codex can also load lifecycle hooks from either `hooks.json` files or inline `[hooks]` tables in `config.toml` files", "Project-local hooks load only when the project `.codex/` layer is trusted." e "User-level hooks remain independent of project trust."

Código: `hooks/src/engine/discovery.rs` itera `config_layer_stack.layers_low_to_high()`, que filtra camadas desabilitadas (`config/src/state.rs:672`). Hooks de projeto não confiável não são descobertos. Existe também um estado de confiança por hook (`hook_states`, `bypass_hook_trust`) que não investiguei.

### Regras de exec policy (`.rules`)

`load_exec_policy` (`core/src/exec_policy.rs:~672`) lê `<config_folder>/rules/` de cada camada habilitada. Camada de projeto desabilitada não contribui. Há também `ignore_user_and_project_exec_policy_rules` em `LoaderOverrides`, exposto como `--ignore-rules` no `codex exec`, mas não vi exposição no app-server.

### AGENTS.md

`core/src/agents_md.rs`: concatena `AGENTS.md` (e `AGENTS.override.md`) da raiz do projeto até o `cwd`, e não passa da raiz. A função `load_project_instructions` começa com:

```rust
if config.active_project.is_untrusted() {
    return Ok((!loaded.is_empty()).then_some(loaded));
}
```

`is_untrusted()` é verdadeiro só para `trust_level = "untrusted"` explícito (`config/src/config_toml.rs:627`). Projeto sem entrada de confiança carrega AGENTS.md. Isso é o que o ADR 0044 precisa: camadas de projeto desabilitadas (ausência de entrada) e AGENTS.md ainda carregado. Marcar o projeto como `untrusted` explícito derruba o AGENTS.md também.

### Skills

`ext/skills/src/host_roots.rs`:

- `roots_from_layer_stack` usa `all_layers_high_to_low()`, que inclui camadas desabilitadas, e adiciona `<.codex>/skills` como `SkillScope::Repo` para camadas de projeto.
- `repo_agents_skill_roots` adiciona `.agents/skills` de cada diretório entre a raiz e o `cwd`, sem consultar confiança.

Pela leitura do código, skills do repositório carregam mesmo com o projeto sem confiança. Não encontrei um portão de confiança em `ext/skills`. Não verificado em runtime. Esta é a maior lacuna (seção 5).

### Plugins

`core-plugins` carrega plugins de marketplaces configurados. Não investiguei se um repositório consegue declarar um marketplace sem camada de projeto habilitada. Como camada desabilitada não entra em `layers_low_to_high`, a resposta provável é não.

## 2. Desabilitar por sessão sem tocar a configuração global

### `thread/start`

`ThreadStartParams` (`app-server-protocol/src/protocol/v2/thread.rs:62`) tem `cwd`, `approvalPolicy`, `approvalsReviewer`, `sandbox`, `permissions` (experimental, não combina com `sandbox`), `config: Option<HashMap<String, JsonValue>>`, `baseInstructions`, `developerInstructions`, `ephemeral`. O campo `config` aceita chaves de configuração arbitrárias por thread. Não verifiquei a precedência exata em relação às camadas de projeto, mas o nome `request_overrides` em `thread_processor.rs` indica que ele funciona como camada de override de maior precedência, equivalente a `-c`.

### `-c key=value`

Documentação: "Use `-c` / `--config` when you need to override an arbitrary key." e "`--config` values are parsed as TOML." Chave desconhecida em `-c` gera erro (`loader/mod.rs:673`: "unknown configuration field ... in -c/--config override"), o que ajuda a detectar nomes errados.

### Mecanismos por componente

| Componente | Desabilitar |
|---|---|
| `.codex/config.toml`, hooks, `.rules` do projeto | Manter o projeto sem `trust_level = "trusted"`. Não adicionar entrada de confiança. |
| AGENTS.md | Manter carregado: não usar `untrusted` explícito. |
| Hooks (global) | `features.hooks = false` (chave `hooks`, padrão `true`, `features/src/lib.rs:1402`). Também desliga os hooks do usuário. |
| Plugins | `features.plugins = false`. |
| Apps/conectores | `features.apps = false`. |
| MCP do usuário | `mcp_servers.<nome>.enabled = false`. Servidores do `~/.codex/config.toml` continuam carregando por padrão. |
| Skills | `skills.include_instructions = false` remove o bloco de catálogo. `skills.config` desabilita skills por caminho. Não há um interruptor global confirmado. |
| Perfis | Desde 0.134.0, `--profile` não lê mais `[profiles.nome]`: "In Codex 0.134.0 and later, `--profile` no longer reads `[profiles.profile-name]` from `config.toml`". Usa `~/.codex/profile-name.config.toml`. Não ajuda a isolar o projeto. |

Existe `ignore_project_config` em `LoaderOverrides` (`config/src/state.rs:56`): "Skip project-root discovery and all project configuration layers." Só o `codex exec` (worktree) e o carregamento de configuração padrão do app-server o ativam (`app-server/src/config_manager.rs:461`). Não achei flag de linha de comando nem parâmetro de protocolo que o exponha. Se o time do Codex expuser, é a opção mais limpa.

### `CODEX_HOME` separado e login

Documentação: "Codex stores its local state under `CODEX_HOME` (defaults to `~/.codex`)." O login vive em `$CODEX_HOME/auth.json` (`login/src/auth/storage.rs:47`), com `cli_auth_credentials_store` aceitando `file` (padrão), `keyring` e `auto`. Um `CODEX_HOME` separado, vazio, perde o login. Para mantê-lo:

- Copiar ou ligar (symlink) `auth.json` para o home isolado. Isso expõe o token a uma segunda cópia e o refresh pode divergir entre as duas. Não verificado.
- Se o usuário usa `keyring`, o login não depende do arquivo, mas a chave do keyring pode ser derivada do caminho do `CODEX_HOME`. Não verificado.

Conclusão: um `CODEX_HOME` próprio quebra o login por padrão e acrescenta risco de credenciais duplicadas. Evitar. Usar o home do usuário e controlar a confiança e as features por `config` no `thread/start`.

## 3. Sandbox somente leitura, aprovações e MCP próprio

- `thread/start` aceita `sandbox` (`SandboxMode`: `read-only`, `workspace-write`, `danger-full-access`) e `approvalPolicy` (inclui `never`). `build_thread_config_overrides` os converte em `ConfigOverrides` tipados, com precedência sobre o arquivo.
- Há restrições gerenciadas (`requirements.toml`) que podem proibir `approval_policy = never`. O app deve tratar o erro `is_unsupported_untrusted_approval_policy_error` e reportá-lo (`app-server/src/config_manager.rs:~443`).
- `McpServerTransportConfig` (streamable HTTP) tem `url`, `bearer_token_env_var`, `http_headers`, `env_http_headers`, `http_headers_helper` (`config/src/mcp_types.rs:~667`). O Codex lê o token de uma variável de ambiente do processo (`resolve_bearer_token`, `codex-mcp/src/rmcp_client.rs:959`) e falha com mensagem clara se a variável estiver ausente ou vazia. Para um token por sessão, o app define a variável no ambiente do processo `codex app-server` e passa `bearer_token_env_var` em `config`. Não coloque o token em `http_headers` literal, porque o valor ficaria em configuração e possivelmente em logs.
- `required`: "When `true`, `codex exec` exits with an error if this MCP server fails to initialize." (comentário em `mcp_types.rs:248`). O comentário só cita `codex exec`. Há `required_servers` em `core/src/session/mcp.rs`, mas não confirmei se o app-server falha o `thread/start` quando um servidor `required` não inicializa. Não verificado. O app deve checar o status do servidor (`mcpServerStatus/list` ou equivalente) antes do primeiro turno.
- Chamada MCP sob `read-only`: o sandbox restringe comandos de shell e escrita de arquivos, não o cliente MCP, que roda no processo do Codex. Não encontrei no código bloqueio de ferramentas MCP por sandbox. Existe `requires_read_only_mcp_tools` (`codex-mcp/src/server.rs:42`), ligado a um modo de acesso confiável, que restringe a ferramentas com `readOnlyHint`. Para o servidor do app, não verificado em runtime. A política de aprovação das ferramentas MCP é separada: `default_tools_approval_mode` por servidor. Com `approvalPolicy = never`, ferramentas que exigem aprovação são rejeitadas, então configure `default_tools_approval_mode` do servidor do app para permitir (valor exato a confirmar no schema).

## 4. Detectar configuração de projeto para avisar

O app detecta por conta própria, no sistema de arquivos, sem iniciar o Codex. A lista abaixo segue o que o Codex realmente carrega, a partir de `cwd` até a raiz do projeto (`.git`):

1. `<dir>/.codex/` existente em qualquer diretório do caminho (`config.toml`, `hooks.json`, `rules/`, `skills/`).
2. `<dir>/.agents/skills/` (skills de repositório).
3. `.codex/config.toml` com chaves `mcp_servers`, `hooks`, `features`, `skills`, `plugins`, `marketplaces`, `projects`, `sandbox_mode`, `approval_policy`.
4. Chaves da denylist presentes (o Codex as ignora, mas elas indicam intenção de redirecionar credenciais): `model_providers`, `openai_base_url`, `notify`, `otel`.
5. `AGENTS.override.md` (variante local do AGENTS.md).

Segunda fonte, em runtime: depois de `thread/start`, o app-server envia avisos de inicialização. `project_ignored_config_keys_warning` emite "Ignored unsupported project-local config keys in ...". A resposta de camadas de configuração (`config/read` com camadas) lista camadas com `disabled_reason`, o que permite mostrar "camada do projeto desabilitada: confiança ausente". Não confirmei o nome exato do método nem os campos expostos. Checar o schema gerado com `codex app-server generate-ts`.

## 5. Lacunas e riscos

1. **Skills do repositório não parecem gateadas por confiança** (`ext/skills/src/host_roots.rs`). Um repositório hostil com `.agents/skills/x/SKILL.md` pode injetar instruções no catálogo de skills. Mitigação parcial: `skills.include_instructions = false`. Mitigação completa precisa de teste e talvez de pedido ao time do Codex. AGENTS.md já é, por definição do ADR, instrução não confiável.
2. **Confiança pré-existente no `~/.codex/config.toml` do usuário.** Se o usuário já marcou o repositório como `trusted`, as camadas de projeto carregam. O app não consegue forçar "sem entrada". Forçar `untrusted` por `config` perde o AGENTS.md. O app deve ler o config global (ou `config/read`) antes, detectar `trusted` e avisar. `ignore_project_config` resolveria, mas não é exposto.
3. **Confiança cobre ancestrais.** `decision_for_dir` consulta o diretório, depois a raiz do projeto e a raiz do repositório git. Um `trusted` em um diretório pai (ou na raiz do repositório) vale para subdiretórios. Worktrees git ligadas usam hooks da checkout raiz (`root_checkout_hooks_folder_for_dir`).
4. **Camadas do usuário continuam ativas.** MCP servers, hooks e plugins de `~/.codex/config.toml` carregam. `features.hooks = false` e `plugins = false` por thread também desligam os do usuário. O ADR precisa decidir se isso é aceitável.
5. **`required` e MCP sob read-only não verificados** (seção 3). Teste obrigatório: iniciar `codex app-server`, `thread/start` com `sandbox: "read-only"`, `approvalPolicy: "never"`, servidor HTTP com `bearer_token_env_var`, e chamar uma ferramenta.
6. **Versão.** O código-fonte é de 2026-10-23 e a CLI local é 0.164.0. Os nomes de campos em `thread/start` marcados `experimental` (`permissions`, `runtimeWorkspaceRoots`) exigem opt-in na inicialização e podem mudar.
7. **Documentação oficial incompleta no que importa.** A página de configuração avançada cobre confiança, hooks e denylist, mas a de MCP não carregou. As afirmações sobre MCP vêm do código, não de documentação.

## Configuração recomendada do app-server

Iniciar `codex app-server` com o `CODEX_HOME` padrão do usuário (mantém o login) e o token da sessão em uma variável de ambiente do processo:

```
VISCONDE_MCP_TOKEN=<token da sessão> codex app-server
```

`thread/start`, com `cwd` na raiz do repositório analisado:

```json
{
  "cwd": "/caminho/do/repo",
  "sandbox": "read-only",
  "approvalPolicy": "never",
  "ephemeral": true,
  "config": {
    "features.hooks": false,
    "features.plugins": false,
    "features.apps": false,
    "skills.include_instructions": false,
    "mcp_servers.visconde": {
      "url": "http://127.0.0.1:<porta>/mcp",
      "bearer_token_env_var": "VISCONDE_MCP_TOKEN",
      "required": true,
      "startup_timeout_sec": 10
    }
  }
}
```

Observações:

- Não escrever `projects."<repo>"` com `trusted` nem `untrusted`. Ausência de entrada mantém as camadas de projeto desabilitadas e o AGENTS.md carregado.
- O formato de `config` (chaves pontilhadas ou objeto aninhado) e a conversão do bloco `mcp_servers.visconde` precisam de teste. Se chaves pontilhadas não valerem, usar objetos aninhados.
- Antes do primeiro turno: ler as camadas de configuração e avisar se o repositório tiver `trusted` no config do usuário ou se o app detectar `.codex/` ou `.agents/skills` (seção 4). Confirmar que o servidor `visconde` inicializou.
- Para que o usuário habilite a configuração do repositório, o caminho é ele mesmo marcar o projeto como `trusted` (o app não escreve no config global sem consentimento explícito).

## Fontes

- https://learn.chatgpt.com/docs/config-file/config-advanced (antes `developers.openai.com/codex/config-advanced`)
- https://github.com/openai/codex, commit `970111f6cb62cc02fe36a82ea7d71b37a2889c97`: `codex-rs/config/src/loader/mod.rs`, `config/src/state.rs`, `config/src/project_trust.rs`, `config/src/config_toml.rs`, `config/src/mcp_types.rs`, `core/src/agents_md.rs`, `core/src/exec_policy.rs`, `hooks/src/engine/discovery.rs`, `ext/skills/src/host_roots.rs`, `codex-mcp/src/rmcp_client.rs`, `app-server-protocol/src/protocol/v2/thread.rs`, `app-server/src/request_processors/thread_processor.rs`, `app-server/src/config_manager.rs`, `features/src/lib.rs`, `login/src/auth/storage.rs`
