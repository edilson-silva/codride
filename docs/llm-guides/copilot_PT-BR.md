# CoDriDe no GitHub Copilot CLI

Este guia é para adotar o CoDriDe num projeto usando o [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli) em vez de (ou junto com) o Claude Code, o Gemini CLI ou o Codex CLI. Ele não pressupõe familiaridade com o Claude Code — tudo aqui é nativo do Copilot CLI.

## O que você copia

Do [repositório do CoDriDe](https://github.com/edilson-silva/codride), copie duas coisas pro seu projeto:

```bash
mkdir -p /path/to/your-project/.github
cp -r codride/.github/agents /path/to/your-project/.github/
cp codride/.github/copilot-instructions.md /path/to/your-project/.github/copilot-instructions.md
```

Confira essa última linha antes de rodar: muitos repositórios já têm um `.github/copilot-instructions.md` próprio pra revisão de código do Copilot da GitHub — isso sobrescreveria ele silenciosamente. Se o seu repositório de destino já tiver um, mescle os dois à mão em vez de copiar às cegas.

- **`.github/agents/`** — os comandos do CoDriDe *e* seus 8 agentes centrais, todos num único diretório plano, um arquivo `.agent.md` cada. Os comandos são nomeados `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) pra ficarem livres de colisão sem um namespace baseado em diretório; os 8 agentes centrais mantêm seus nomes originais, planos (`branch-code-reviewer`, `adr-compliance-checker`, etc.), já que nunca foram comandos namespaced pra começo de conversa.
- **`.github/copilot-instructions.md`** — a visão geral do pipeline, o elenco de agentes e os passos de adoção, carregado automaticamente pelo Copilot CLI no início de toda sessão neste projeto.

Você nunca precisa do Claude Code, de `.claude/` ou de `CLAUDE.md` pra nada disso — `.github/agents/` e `.github/copilot-instructions.md` são completos e autocontidos.

## Como começar

Os mesmos seis passos do Início Rápido do Claude Code, invocados como agentes do Copilot:

```
engineer-doctor      # checagem pré-voo
meta-preferences     # idioma da documentação, extensível pra mais depois
engineer-discover     # gera docs/technical-context/
                      # depois escreva docs/business-context/ via bootstrap-business-docs
warm-up               # começa uma sessão
```

Invoque um agente com `/agent-name` interativamente, com `--agent <name> --prompt "..."` programaticamente, ou só descreva sua tarefa em linguagem natural — o Copilot vai inferir o agente correspondente a partir da `description` dele. Daí em diante, o pipeline completo (trilha de produto, trilha de engenharia, `engineer-pre-pr`, `engineer-pr`) funciona exatamente como documentado no [README](../../README_PT-BR.md) principal — mesmo fluxo geral, mesmos comandos pelo nome, só acessados de forma diferente (veja "Sem sintaxe formal de argumento" abaixo). Work items caem em `.copilot/work/<type>/<slug>/` — um local específico do Copilot, não compartilhado com `.claude/work/`, `.gemini/work/` ou `.codex/work/` mesmo no mesmo projeto.

## O que é diferente do Claude Code

**Um formato unificado de agente, não dois.** O Claude Code separa comandos (`.claude/commands/`) de subagentes (`.claude/agents/`) em diretórios e formatos de arquivo distintos. O Copilot CLI tem só um: `.agent.md`, distinguido por uma flag de frontmatter `user-invocable` — `true` pra algo que você invoca diretamente (um comando), `false` pra algo que só outro agente delega. Os 35 arquivos — 27 comandos, 8 agentes centrais — ficam todos num único diretório plano, `.github/agents/`. Dois dos 8 agentes centrais são a exceção: `python-developer` e `typescript-developer` não têm um comando chamador fixo no Claude Code (são usados "sob demanda", invocados diretamente por um humano), então ambos mantêm `user-invocable: true` aqui — a mesma exceção que o Codex CLI faz ao transformá-los em Skills independentes.

**Delegação real a subagentes — nada é embutido.** Diferente do Codex CLI, o Copilot CLI suporta sim um agente delegando pra outro, então esta tradução preserva cada relação comando→agente exatamente como funciona no Claude Code: `engineer-validate` continua delegando pro `branch-master-docs-checker`, `product-sync-github` continua delegando pro `github-project-sync`, e assim por diante — como arquivos separados, não como lógica embutida.

**Comandos orquestradores ainda rodam sequencialmente, não em paralelo — mas por um motivo diferente do Gemini.** O `engineer-pre-pr` normalmente roda `engineer-validate` e `engineer-review` de forma concorrente no Claude Code. A concorrência de subagentes do Copilot CLI é real e configurável (não experimental como a do Gemini) — mas ela é restrita a determinados planos e configurações explícitas (v1.0.66+, faturamento por uso), então não dá pra presumir que ela está presente pra todo adotante. Por esse motivo, esta geração usa sequencial por padrão. Se o seu plano tiver concorrência configurada, você mesmo pode habilitar o despacho paralelo pro `engineer-validate`/`engineer-review`.

**Sem sintaxe formal de argumento.** Os comandos do Claude Code leem um bloco `#$ARGUMENTS` substituído; o Gemini CLI usa `{{args}}`. Os agentes do Copilot CLI não têm nenhum dos dois — confirmado direto na documentação oficial da GitHub, não presumido. Um agente não tem forma nenhuma templated de receber parâmetros; ele funciona inteiramente a partir do contexto em linguagem natural do seu pedido ou da flag `--prompt`. Todo agente que costumava ler um argumento (um slug de work item, um número de issue, uma descrição de feature) agora instrui o Copilot a extrair essa informação do que você realmente digitou, e a perguntar em vez de chutar se estiver ambíguo.

**Os nomes das ferramentas dos agentes não estão verificados.** O vocabulário próprio de nomes de ferramentas do Copilot CLI (`read`, `search`, etc.) foi obtido de um exemplo da comunidade, não da tabela de referência completa e oficial da GitHub, no momento em que isto foi gerado. O campo `tools:` do frontmatter de cada agente carrega os nomes de ferramentas do Claude Code como um placeholder sinalizado, não como um chute disfarçado de fato — um comentário acima de `tools:` avisa isso. Se um agente não se comportar como esperado, esse é o primeiro lugar pra checar.

**Sem suporte a Artifacts.** Igual ao Gemini CLI e ao Codex CLI — o `/meta:preferences` do Claude Code configura a publicação de Artifacts (páginas hospedadas no claude.ai); esse recurso não existe aqui, então está totalmente ausente do `meta-preferences`.

**Sem checagem de atualização do alvo gerado.** Igual ao Gemini CLI e ao Codex CLI — a checagem interna de contabilidade que sinaliza um alvo gerado desatualizado no `/engineer:doctor` do Claude Code não se aplica ao seu projeto; ela é excluída desta geração. Se sua cópia de `.github/agents/`/`.github/copilot-instructions.md` parecer desatualizada, peça pra quem mantém a fonte do seu CoDriDe regenerá-la.

**O `meta-create-agent` cria agentes nativos do Copilot.** Ele escreve arquivos `.github/agents/project-<name>.agent.md`, usando o frontmatter real do Copilot — incluindo um campo `tools:` genuíno (diferente do Codex, que não tem nenhum) e, pra capacidades apoiadas em MCP, `mcp-servers:` em vez de uma entrada de ferramenta individual.

**Bônus: o Copilot CLI também lê `AGENTS.md`, `CLAUDE.md` e `GEMINI.md` diretamente, se estiverem presentes.** Ele combina e deduplica esses arquivos com o `.github/copilot-instructions.md`, sem uma ordem de precedência definida entre eles. Isso é um bônus se o seu projeto carregar mais de um alvo do CoDriDe lado a lado — não é algo de que esta geração dependa, já que quem adota só o Copilot nunca tem esses outros arquivos.

## Fonte da verdade

Tudo sob `.github/agents/` e `.github/copilot-instructions.md` é gerado a partir da fonte canônica do CoDriDe no Claude Code, mantida upstream. Não edite os arquivos gerados à mão — as mudanças se perdem na próxima vez que a fonte for regenerada (um overwrite completo, nunca um merge). Se você precisar mudar algo, isso acontece upstream, no próprio repositório do CoDriDe.
