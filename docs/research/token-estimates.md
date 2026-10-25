# Estimativas de tokens para leitura de código

Pesquisa realizada em 25 de outubro de 2026. Escopo: apoiar o desenho da versão 0.0.1 de um visualizador Electron, com código somente para leitura e snapshot semântico sob demanda. Nenhum projeto privado foi acessado. Não houve chamadas a APIs de modelos, uso de credenciais ou execução de scripts dos repositórios baixados.

## Números para planejamento

Os números abaixo representam o conteúdo dos arquivos em **uma passagem pelo texto**, sem prompts, nomes de arquivos, numeração de linhas, resultados adicionais de ferramentas ou respostas do agente. `k` significa mil tokens. As medições usam `tiktoken 0.12.0`, codificação `o200k_base`.

| Caso | Tokens de fonte em uma passagem | Natureza do número |
| --- | ---: | --- |
| Projeto privado, 7 mil linhas backend e 17 mil frontend, contando linhas físicas | Referência de **187k**; faixa de planejamento **150k a 270k** | Inferência de amostras TS/TSX públicas |
| Mesmo projeto, se as 24 mil linhas excluírem comentários e linhas em branco | Referência de **212k**; faixa de planejamento **180k a 320k** | Inferência com outro denominador de LOC |
| Motor Redis 7.2.4, escopo C/H definido abaixo | **1.290.281** | Medição local exata para esse corpus e tokenizador |
| Pequeno código de produção, Zustand 5.0.3, `src/` com 1.344 linhas físicas | **10.244** | Medição local exata; testes e documentação ficam fora |
| Projeto hipotético TS/TSX com 4 mil linhas físicas | **20k a 40k** | Extrapolação ilustrativa, sem medir um projeto de 4 mil linhas |

Para uma apresentação que precise de um único número aproximado do projeto privado, **200 mil tokens de fonte** é uma referência útil. A faixa conjunta de 150 mil a 320 mil deixa explícita a dúvida sobre como as linhas foram contadas. Não é intervalo estatístico nem garantia de encaixe em um modelo.

Mostrar arquivos locais, destacar sintaxe, navegar por símbolos ou renderizar um snapshot já salvo consome **zero tokens de LLM**, desde que essas operações sejam locais e não chamem um modelo. Gerar ou atualizar o snapshot por um agente consome os tokens que efetivamente entram e saem das chamadas. O modo somente leitura impede alterações, mas não elimina consumo de inferência.

## Corpus e revisões fixas

As versões históricas abaixo servem como amostras reproduzíveis. Não representam necessariamente as versões atuais nem recomendações de dependências. Taxonomy está arquivado, conforme seu README; continua útil como amostra de texto TSX.

| Fonte primária | Revisão medida | Recorte |
| --- | --- | --- |
| [NestJS](https://github.com/nestjs/nest/tree/d0fb875e23e3e6baae8537efd41d54709acade0c) | `d0fb875e23e3e6baae8537efd41d54709acade0c`, tag `v10.4.15` | `packages/core/**/*.ts` e, separadamente, arquivos TS/TSX dentro de `sample/*/src/` |
| [Taxonomy](https://github.com/shadcn-ui/taxonomy/tree/298a8857c7128a0d121e7f699dfd729f23b3966d) | `298a8857c7128a0d121e7f699dfd729f23b3966d` | Arquivos versionados `.tsx` e `.ts`, contados separadamente |
| [Zustand](https://github.com/pmndrs/zustand/tree/3f9127f4ebc9c846ac6ace3f502af199280ae884) | `3f9127f4ebc9c846ac6ace3f502af199280ae884`, tag `v5.0.3` | Todos os `.ts` de `src/`, incluindo declarações mantidas no projeto |
| [Redis](https://github.com/redis/redis/tree/d2c8a4b91e8c0e6aefd1f5bc0bf582cddbe046b7) | `d2c8a4b91e8c0e6aefd1f5bc0bf582cddbe046b7`, tag `7.2.4` | Arquivos `.c` e `.h` diretamente em `src/`, com as exclusões abaixo |

Nest fornece tanto código interno de framework quanto exemplos de backend. Taxonomy fornece componentes, páginas, layouts e rotas de uma aplicação Next.js. Seu TSX inclui componentes de servidor, conteúdo de interface e SVG embutido; não representa exclusivamente código executado no navegador. Zustand é uma biblioteca pequena no recorte de produção, não uma aplicação completa nem um repositório inteiro com menos de 5 mil linhas contando testes e documentação.

### Regras de inclusão

- Apenas arquivos versionados por Git entram na seleção. Cada arquivo conta uma vez, mesmo se outro arquivo tiver conteúdo idêntico.
- Texto UTF-8 original, comentários, cabeçalhos e espaços em branco permanecem na entrada do tokenizador. Não há minificação, remoção de comentários ou reformatação.
- Dependências, vendor, builds, arquivos minificados, binários e lockfiles ficam fora. A seleção por extensão também exclui CSS, imagens, Markdown/MDX, manifests e configurações JSON do corpus TS/TSX.
- Testes em diretórios `test`, `tests` e `__tests__`, além de arquivos `.spec.*` e `.test.*`, ficam fora. Exemplos de backend do Nest entram somente na amostra identificada como exemplos.
- Os dois `graphql.schema.ts` gerados dos exemplos Nest ficam fora. Importar código gerado não torna o arquivo importador gerado.
- No Redis, `deps/`, `tests/`, módulos de exemplo em `src/modules/`, documentação e scripts ficam fora. Testes condicionais embutidos em arquivos de produção, como blocos `REDIS_TEST`, permanecem: ler o arquivo inteiro também lê esses blocos.

O escopo principal Redis contém código do servidor, estruturas de dados, persistência, replicação, Sentinel, suporte a módulos e verificadores AOF/RDB. É o código do próprio motor de banco, não uma aplicação que usa Redis nem dados armazenados em execução.

Exclusões adicionais do Redis, aplicadas literalmente pelo script:

- Clientes e ferramentas: `redis-cli.c`, `redis-benchmark.c`, `cli_commands.c`, `cli_commands.h`, `cli_common.c`, `cli_common.h`, `sparkline.c`, `sparkline.h`.
- Auxiliares incorporados ou adaptados de terceiros: `crc16.c`, `geohash.c`, `geohash.h`, `geohash_helper.c`, `geohash_helper.h`, `lzf.h`, `lzfP.h`, `lzf_c.c`, `lzf_d.c`, `mt19937-64.c`, `mt19937-64.h`, `pqsort.c`, `pqsort.h`, `rand.c`, `rand.h`, `setproctitle.c`, `sha1.c`, `sha1.h`, `sha256.c`, `sha256.h`, `siphash.c`, `strl.c`. Os cabeçalhos dos próprios arquivos registram sua origem. Esta é uma delimitação operacional do corpus, não uma auditoria completa de autoria.
- Tabela e arte: `crc16_slottable.h` e `asciilogo.h`. Auxiliar de testes: `testhelp.h`.
- `commands.def` fica fora por ser gerado. O [Makefile fixado](https://github.com/redis/redis/blob/d2c8a4b91e8c0e6aefd1f5bc0bf582cddbe046b7/src/Makefile#L438) mostra sua geração a partir de `src/commands/*.json`. Esses JSON são definições de comandos, não dados de usuários, e recebem uma contagem adicional separada.

## Medições

Linha física significa cada linha delimitada por LF, incluindo a última se não terminar em LF. O [cloc](https://github.com/AlDanial/cloc), versão 2.10, separa linhas em branco, comentários e código. A classificação é a do cloc, não uma análise semântica. Em todos os grupos, a soma dessas três categorias coincide com a contagem física independente.

| Corpus | Arquivos | Linhas físicas | Em branco | Comentários | Código, cloc | Tokens `o200k_base` | Tokens `cl100k_base` |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Nest core | 172 | 13.627 | 1.302 | 871 | 11.454 | 90.343 | 89.406 |
| Nest exemplos | 281 | 4.366 | 542 | 65 | 3.759 | 26.704 | 26.216 |
| Taxonomy TSX | 94 | 6.253 | 612 | 8 | 5.633 | 51.877 | 50.672 |
| Taxonomy TS | 31 | 1.258 | 173 | 44 | 1.041 | 7.016 | 6.952 |
| Zustand `src/` | 15 | 1.344 | 150 | 77 | 1.117 | 10.244 | 10.147 |
| Redis motor C/H | 131 | 131.508 | 13.599 | 31.972 | 85.937 | 1.290.281 | 1.285.028 |
| Redis C/H incluindo clientes e ferramentas | 139 | 144.307 | 14.556 | 33.235 | 96.516 | 1.407.985 | 1.402.187 |
| Redis definições de comandos JSON | 392 | 23.626 | 2 | 0 | 23.624 | 133.092 | 133.073 |

As duas linhas C/H do Redis se sobrepõem; não devem ser somadas. Motor mais definições JSON resulta em **1.423.373 tokens**. C/H com clientes, ferramentas e JSON resulta em **1.541.077 tokens**. Nenhum desses totais inclui todo o checkout indiscriminadamente.

Usei [`tiktoken`](https://github.com/openai/tiktoken) localmente, somando `len(encoding.encode_ordinary(texto))` por arquivo. A codificação principal é `o200k_base`; `cl100k_base` mostra a sensibilidade a outra codificação pública. Seus [vocabulários e padrões estão definidos na versão 0.12.0](https://github.com/openai/tiktoken/blob/0.12.0/tiktoken_ext/openai_public.py). A soma por arquivo pode diferir ligeiramente da tokenização de um texto concatenado, além do acréscimo de separadores e metadados no uso real.

**Não são medições do consumo de Codex ou Claude Code.** Codex e Claude Code são agentes que podem usar modelos diferentes. Não estabeleci correspondência entre o tokenizador escolhido e um modelo concreto da assinatura. A diferença entre as duas codificações ficou abaixo de 2,4% nestas amostras, mas isso não limita o erro para Claude nem para outro modelo. A documentação Anthropic oferece [contagem por modelo e mensagem estruturada](https://platform.claude.com/docs/en/build-with-claude/token-counting); esse endpoint não foi chamado.

## Como estimar as 24 mil linhas privadas

As proporções abaixo usam o texto integral dos arquivos no numerador. A coluna por linha de código **continua incluindo tokens dos comentários e espaços em branco do arquivo**; apenas muda o denominador.

| Amostra | Tokens por linha física | Tokens por linha de código, cloc |
| --- | ---: | ---: |
| Backend, Nest core | 6,63 | 7,89 |
| Backend, exemplos Nest | 6,12 | 7,10 |
| Interface, Taxonomy TSX | 8,30 | 9,21 |
| Suporte TS, Taxonomy | 5,58 | 6,74 |
| Biblioteca TS, Zustand | 7,62 | 9,17 |

Aplicando as médias exatas do Nest core e Taxonomy TSX:

```text
Se forem linhas físicas:
7.000 × (90.343 / 13.627) + 17.000 × (51.877 / 6.253)
≈ 187.446 tokens de fonte

Se forem linhas de código segundo cloc:
7.000 × (90.343 / 11.454) + 17.000 × (51.877 / 5.633)
≈ 211.773 tokens de fonte
```

Para planejamento, amplio as proporções observadas para acomodar outro estilo de código. Com linhas físicas, assumo 5 a 9 tokens por linha de backend e 7 a 12 por linha de frontend: 154 mil a 267 mil tokens, arredondados para **150 mil a 270 mil**. Com linhas de código, assumo 6 a 12 e 8 a 14, respectivamente: 178 mil a 322 mil, arredondados para **180 mil a 320 mil**.

Essas margens são escolhas de planejamento, não percentis de uma população. Backend e frontend privados foram assumidos comparáveis a TS e TSX. Uma aplicação com SQL extenso, grandes objetos literais, SVG embutido, traduções, linhas muito longas ou outra linguagem pode ficar fora delas. Nem a amostra de um framework nem um único frontend representam toda aplicação pessoal.

Também não existe teto universal de tokens para um projeto com menos de 5 mil linhas. Usar provisoriamente 5 a 10 tokens por linha física sugere 5 mil a 10 mil tokens para 1 mil linhas, ou 20 mil a 40 mil para 4 mil. Isso só extrapola a ordem de grandeza das amostras TS/TSX; arquivos muito densos podem exceder a faixa.

## Volume da fonte, contexto e consumo da sessão

O volume de fonte é uma propriedade do corpus e do tokenizador. A janela de contexto limita o conteúdo utilizável em uma requisição; precisa acomodar instruções, histórico, ferramentas e espaço para geração. Ela não equivale à quantidade acumulada de tokens de uma sessão. Os limites dependem do modelo. A [documentação OpenAI sobre contexto](https://developers.openai.com/api/docs/guides/conversation-state#managing-the-context-window) distingue entrada, saída e tokens de raciocínio aplicáveis. Portanto, 200 mil tokens de fonte não garantem que o projeto inteiro caiba em uma chamada.

Agentes pesquisam e leem arquivos conforme a tarefa. Claude Code documenta o ciclo de exploração, ferramentas e [conteúdo que ocupa o contexto](https://code.claude.com/docs/en/how-claude-code-works). O [Codex CLI](https://learn.chatgpt.com/docs/codex/cli) também trabalha com exploração do repositório e ferramentas locais. Abrir o projeto não implica enviar todos os arquivos. Em sentido inverso, um trecho já lido pode reaparecer no histórico de chamadas seguintes sem uma nova leitura do disco.

Uma fórmula ilustrativa para a sessão:

```text
S = tokens do corpus completo em uma passagem
f = fração dos tokens da fonte efetivamente explorada, entre 0 e 1
r = número médio de aparições desse material nas entradas do modelo
H = soma das demais entradas em todas as chamadas
O = soma de tokens gerados em todas as chamadas

Entrada acumulada I ≈ S × f × r + H
Total acumulado T ≈ I + O
```

`r` inclui reapresentação no histórico, além de releituras explícitas. `H` inclui prompts, instruções, esquemas de ferramentas, caminhos, linhas numeradas, resumos e saídas anteriores reapresentadas, sem contar novamente o código já coberto por `S × f × r`. `O` inclui chamadas de ferramentas geradas, texto intermediário e snapshot final; tokens de raciocínio precisam ser considerados quando o modelo os contabiliza. `f` é fração de tokens, não porcentagem de arquivos. Nenhum desses parâmetros foi medido em sessões reais de agente.

| Cenário assumido | S | f | r | H | O | Entrada I | Total T |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Ler todo o pequeno corpus Zustand uma vez | 10.244 | 100% | 1 | 4.000 | 2.000 | 14.244 | 16.244 |
| Explorar uma parte do projeto privado | 200.000 | 10% | 2 | 10.000 | 4.000 | 50.000 | 54.000 |
| Snapshot mais abrangente do projeto privado | 200.000 | 50% | 3 | 20.000 | 8.000 | 320.000 | 328.000 |
| Investigar um subsistema do Redis | 1.290.281 | 5% | 2 | 20.000 | 6.000 | ≈ 149.028 | ≈ 155.028 |

Esses cenários explicam a aritmética. Não predizem duração, qualidade do snapshot, quantidade necessária de turnos ou cobertura semântica. Um snapshot que exige entendimento global pode precisar de muito mais que os 5% ou 10% assumidos.

Entrada em cache continua sendo entrada reutilizada. Por exemplo, se uma sessão tem 50 mil tokens de entrada acumulada e 60% deles obtêm cache hit, são 30 mil tokens de entrada em cache e 20 mil sem cache hit. O corpus não encolhe e a janela não aumenta. A [documentação oficial de prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching) explica o reaproveitamento de contexto e a contabilização específica da API. Cache hit não é garantido, e a telemetria pode distinguir escrita e leitura de cache.

Não converto tokens em dinheiro, mensagens disponíveis ou porcentagem de assinatura. Seriam necessários o modelo, a conta e as regras documentadas do produto; a contagem local não fornece essa conversão. Uma futura medição do agente deve registrar modelo, versão, tokens de entrada, entrada em cache, saída e raciocínio disponíveis na telemetria, além dos arquivos realmente examinados.

## Reprodução

O diretório temporário desta execução é `/tmp/visconde-token-research.5WrCHh`. Ele contém os checkouts, `measure.py`, `results.json`, listas de arquivos e manifestos SHA-256. O relatório abaixo permite reproduzir o trabalho mesmo depois da remoção desse diretório. Nenhum desses artefatos temporários entra no repositório do aplicativo.

Use Python 3.13, `tiktoken==0.12.0` e `cloc` 2.10. Nesta execução, Python foi 3.13.12. Instale apenas as ferramentas de medição; não execute builds, instalações npm ou scripts dos projetos amostrados. Os downloads dos vocabulários públicos do tokenizador não fazem inferência.

Crie um diretório temporário vazio e execute:

```bash
uv venv --python 3.13 .venv
uv pip install --python .venv/bin/python 'tiktoken==0.12.0'
git -c core.hooksPath=/dev/null clone --depth 1 --branch 7.2.4 https://github.com/redis/redis.git redis
git -c core.hooksPath=/dev/null clone --depth 1 --branch v10.4.15 https://github.com/nestjs/nest.git nest
git -c core.hooksPath=/dev/null clone --depth 1 --branch v5.0.3 https://github.com/pmndrs/zustand.git zustand
git init taxonomy
git -C taxonomy remote add origin https://github.com/shadcn-ui/taxonomy.git
git -C taxonomy fetch --depth 1 origin 298a8857c7128a0d121e7f699dfd729f23b3966d
git -C taxonomy -c core.hooksPath=/dev/null checkout --detach FETCH_HEAD
```

Confira `git rev-parse HEAD` em cada checkout contra a tabela de revisões. Salve o script a seguir como `measure.py` ao lado deles e execute `.venv/bin/python measure.py`.

```python
import hashlib
import json
import subprocess
from pathlib import Path

import tiktoken

BASE = Path(__file__).resolve().parent
ENCODINGS = {name: tiktoken.get_encoding(name) for name in ('o200k_base', 'cl100k_base')}
VENDOR = set('crc16.c geohash.c geohash.h geohash_helper.c geohash_helper.h lzf.h lzfP.h lzf_c.c lzf_d.c mt19937-64.c mt19937-64.h pqsort.c pqsort.h rand.c rand.h setproctitle.c sha1.c sha1.h sha256.c sha256.h siphash.c strl.c'.split())
GENERATED = {'crc16_slottable.h', 'asciilogo.h'}

def tracked(repo):
    return [Path(p) for p in subprocess.check_output(['git', '-C', str(BASE / repo), 'ls-files', '-z']).decode().split('\0') if p]

def ts_source(p):
    return (p.suffix in {'.ts', '.tsx'}
            and not {'test', 'tests', '__tests__', 'node_modules', 'dist', 'build', 'vendor', 'generated', '.next'} & set(p.parts)
            and not any(s in p.name for s in ('.spec.', '.test.', '.min.'))
            and p.name != 'graphql.schema.ts')

groups = {}
nest = tracked('nest')
groups['nest_core'] = ('nest', [p for p in nest if str(p).startswith('packages/core/') and ts_source(p)])
groups['nest_samples'] = ('nest', [p for p in nest if p.parts[0] == 'sample' and 'src' in p.parts and ts_source(p)])
taxonomy = [p for p in tracked('taxonomy') if ts_source(p)]
groups['taxonomy_tsx'] = ('taxonomy', [p for p in taxonomy if p.suffix == '.tsx'])
groups['taxonomy_ts'] = ('taxonomy', [p for p in taxonomy if p.suffix == '.ts'])
groups['zustand_src'] = ('zustand', [p for p in tracked('zustand') if p.parts[0] == 'src' and ts_source(p)])
redis = tracked('redis')
groups['redis_c_h'] = ('redis', [p for p in redis if p.parent == Path('src') and p.suffix in {'.c', '.h'} and p.name not in VENDOR | GENERATED | {'testhelp.h'}])
CLIENT_TOOLS = set('redis-cli.c redis-benchmark.c cli_commands.c cli_commands.h cli_common.c cli_common.h sparkline.c sparkline.h'.split())
groups['redis_engine'] = ('redis', [p for p in groups['redis_c_h'][1] if p.name not in CLIENT_TOOLS])
groups['redis_commands_json'] = ('redis', [p for p in redis if str(p).startswith('src/commands/') and p.suffix == '.json'])

report = {'tiktoken': tiktoken.__version__, 'cloc': subprocess.check_output(['cloc', '--version']).decode().strip(), 'groups': {}}
for name, (repo, paths) in groups.items():
    paths = sorted(paths)
    listfile = BASE / f'{name}.files'
    listfile.write_text(''.join(str(BASE / repo / p) + '\n' for p in paths))
    cloc = json.loads(subprocess.check_output(['cloc', '--json', '--quiet', '--skip-uniqueness', '--by-file', f'--list-file={listfile}']))
    rows = []
    for p in paths:
        raw = (BASE / repo / p).read_bytes()
        assert b'\0' not in raw
        source = raw.decode('utf-8')
        rows.append({'path': p.as_posix(), 'sha256': hashlib.sha256(raw).hexdigest(), 'bytes': len(raw), 'physical': source.count('\n') + int(bool(source) and not source.endswith('\n')), **{enc: len(tokenizer.encode_ordinary(source)) for enc, tokenizer in ENCODINGS.items()}})
    totals = {key: sum(row[key] for row in rows) for key in ('bytes', 'physical', *ENCODINGS)}
    totals.update({key: cloc['SUM'][key] for key in ('blank', 'comment', 'code')})
    assert totals['physical'] == totals['blank'] + totals['comment'] + totals['code'], (name, totals)
    manifest = ''.join(f"{row['sha256']}  {row['path']}\n" for row in rows)
    (BASE / f'{name}.manifest').write_text(manifest)
    report['groups'][name] = {'repo': repo, 'commit': subprocess.check_output(['git', '-C', str(BASE / repo), 'rev-parse', 'HEAD']).decode().strip(), 'files': len(rows), 'totals': totals, 'manifest_sha256': hashlib.sha256(manifest.encode()).hexdigest(), 'per_file': rows}
    print(name, len(rows), json.dumps(totals), flush=True)
(BASE / 'results.json').write_text(json.dumps(report, indent=2) + '\n')
```

Cada manifesto contém linhas `sha256  caminho_relativo`, ordenadas por caminho e terminadas em LF. Estes são os hashes dos manifestos medidos, para conferir seleção e conteúdo:

| Grupo | SHA-256 do manifesto |
| --- | --- |
| `nest_core` | `ebb456e5a7199e3adc14e7125ba7bccc7320570e7954bbb2f81537821077c194` |
| `nest_samples` | `934db22fee30bf82c86ac72fff8ede35c625f4800083c405af3ec66dc648c143` |
| `taxonomy_tsx` | `094f0763fefb559079aeed43ec01fedfe08f68f2a901f58622a1fa9ddf828c0f` |
| `taxonomy_ts` | `9b3ed30258d1e6515c7de975b24be408985c954aa913cd081041e11dc09ba126` |
| `zustand_src` | `613f3b91cf8df99499245c41bdd35259838ad376417fd941ba5df37eec6c5fdd` |
| `redis_c_h` | `7a8ac5930307238e081cf85a4363fb945e8b44de0cbc456c23170e5639d8ebb1` |
| `redis_engine` | `eff53a1b96b5ce2b114565cf895a976ce78ab6c266b1e475ffddb17918b80046` |
| `redis_commands_json` | `97d8c512c7815cae2b687241fc0a3a183959cbe675830e811cbcc63e10bb5c12` |
