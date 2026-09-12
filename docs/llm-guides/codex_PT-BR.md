# CoDriDe no Codex CLI

Este guia é para adotar o CoDriDe num projeto usando o [Codex CLI](https://developers.openai.com/codex) em vez de (ou junto com) o Claude Code ou o Gemini CLI. Ele não pressupõe familiaridade com o Claude Code — tudo aqui é nativo do Codex CLI.

## O que você copia

Do [repositório do CoDriDe](https://github.com/edilson-silva/codride), copie duas coisas pro seu projeto:

```bash
cp -r codride/.agents codride/AGENTS.md /path/to/your-project/
```

- **`.agents/skills/`** — as skills do CoDriDe, um diretório por skill (`.agents/skills/<name>/SKILL.md`), nomeadas `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) pra ficarem livres de colisão sem um namespace baseado em diretório. Duas skills — `python-developer`, `typescript-developer` — mantêm um nome plano, já que nunca foram namespaced.
- **`AGENTS.md`** — a visão geral do pipeline, o mapeamento de agente pra skill, e os passos de adoção, carregado automaticamente pelo Codex CLI (varrido a partir do seu diretório de trabalho até a raiz do repositório).

Você nunca precisa do Claude Code, de `.claude/` ou de `CLAUDE.md` pra nada disso — `.agents/` e `AGENTS.md` são completos e autocontidos.

## Como começar

Os mesmos seis passos do Início Rápido do Claude Code, invocados como Codex Skills:

```
engineer-doctor      # checagem pré-voo
meta-preferences     # idioma da documentação, extensível pra mais depois
engineer-discover     # gera docs/technical-context/
                      # depois escreva docs/business-context/ via bootstrap-business-docs
warm-up               # começa uma sessão
```

Invoque uma skill mencionando ela explicitamente (`$engineer-doctor`) ou só descreva sua tarefa em linguagem natural — o Codex vai selecionar a skill correspondente implicitamente quando seu pedido bater com a descrição dela. Daí em diante, o pipeline completo funciona exatamente como documentado no [README](../../README_PT-BR.md) principal — mesmo fluxo geral, mesmos comandos pelo nome, só acessados de forma diferente. Work items caem em `.codex/work/<type>/<slug>/` — um local específico do Codex, não compartilhado com `.claude/work/` ou `.gemini/work/` mesmo no mesmo projeto.

## O que é diferente do Claude Code

Este alvo tem duas diferenças estruturais, não só cosméticas — leia as duas antes de supor que um comando se comporta como no Claude Code.

**Nenhuma delegação a subagentes.** O Claude Code tem 8 subagentes centrais, cada um um arquivo separado pro qual o orquestrador pode delegar — incluindo rodar alguns de forma concorrente (o passo `validate`+`review` do `/engineer:pre-pr`). O Codex CLI não tem mecanismo de delegação nenhum, nem mesmo delegação sequencial pra um arquivo separado. Seis desses agentes são embutidos diretamente na única skill que costumava invocá-los — `engineer-validate`, `engineer-review`, `engineer-sync-docs`, `engineer-coverage`, `engineer-discover`, `engineer-work` e `product-sync-github` (sete skills, já que `engineer-discover` e `engineer-work` carregam cada uma, de forma independente, uma cópia completa da lógica de checagem de conformidade com ADR do sexto agente, duplicada em vez de compartilhada — o Codex também não tem mecanismo de referência pra isso). Os dois agentes sob demanda sem chamador fixo (`python-developer`, `typescript-developer`) viraram skills independentes, invocadas diretamente em vez de delegadas.

**Sem sintaxe formal de argumento.** Os comandos do Claude Code leem um bloco `#$ARGUMENTS` substituído; o Gemini CLI usa `{{args}}`. As Codex Skills não têm nenhum dos dois — confirmado direto na documentação oficial do Codex, não presumido. Uma Skill não tem forma nenhuma templated de receber parâmetros; ela funciona inteiramente a partir do contexto em linguagem natural do seu pedido. Toda skill que costumava ler um argumento (um slug de work item, um número de issue, uma descrição de feature) agora instrui o Codex a extrair essa informação do que você realmente digitou, e a perguntar em vez de chutar se estiver ambíguo. Na prática: em vez de digitar algo equivalente a `/engineer:context feat/csv-order-export`, você diria algo como "comece o context pra feat/csv-order-export" e a skill lê o slug da sua frase.

**A nomeação das skills não tem namespace.** `.agents/skills/<name>/` é plano — não existe um subdiretório por namespace como o Claude Code (`.claude/commands/engineer/`) ou o Gemini CLI (`.gemini/commands/engineer/`) têm. As skills do CoDriDe são nomeadas `<namespace>-<name>` em vez disso (`engineer-validate`, `product-validate` — ambas existem e não colidem) pra preservar a mesma distinção sem uma pasta pra fazer isso.

**Sem suporte a Artifacts.** Igual ao Gemini CLI — o `/meta:preferences` do Claude Code configura a publicação de Artifacts (páginas hospedadas no claude.ai); esse recurso não existe aqui, então está totalmente ausente do `meta-preferences`.

**Sem checagem de atualização do alvo gerado.** Igual ao Gemini CLI — a checagem interna de contabilidade que sinaliza um alvo gerado desatualizado no `/engineer:doctor` do Claude Code não se aplica ao seu projeto; ela é excluída desta geração. Se sua cópia de `.agents/`/`AGENTS.md` parecer desatualizada, peça pra quem mantém a fonte do seu CoDriDe regenerá-la.

**O `meta-create-agent` cria skills nativas do Codex.** Ele escreve arquivos `.agents/skills/project-<name>/SKILL.md`, com uma omissão deliberada: os subagentes do Claude Code carregam um allowlist `tools:` que delimita exatamente o que aquele agente pode acessar; as Codex Skills não têm um campo equivalente de restrição de ferramentas, então o `meta-create-agent` não tenta inventar um — uma skill que você cria é instruções mais frontmatter, nada além disso.

## Fonte da verdade

Tudo sob `.agents/` e `AGENTS.md` é gerado a partir da fonte canônica do CoDriDe no Claude Code, mantida upstream. Não edite os arquivos gerados à mão — as mudanças se perdem na próxima vez que a fonte for regenerada (um overwrite completo, nunca um merge). Se você precisar mudar algo, isso acontece upstream, no próprio repositório do CoDriDe.
