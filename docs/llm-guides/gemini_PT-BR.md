# CoDriDe no Gemini CLI

Este guia é para adotar o CoDriDe num projeto usando o [Gemini CLI](https://github.com/google-gemini/gemini-cli) em vez de (ou junto com) o Claude Code. Ele não pressupõe familiaridade com o Claude Code — tudo aqui é nativo do Gemini CLI.

## O que você copia

Do [repositório do CoDriDe](https://github.com/edilson-silva/codride), copie duas coisas pro seu projeto:

```bash
cp -r codride/.gemini codride/GEMINI.md /path/to/your-project/
```

- **`.gemini/commands/`** — os slash commands do CoDriDe (`/engineer:context`, `/product:spec`, etc.), um arquivo TOML por comando, namespaced do mesmo jeito que os do Claude Code (`engineer/`, `product/`, `bootstrap/`, `meta/`).
- **`.gemini/agents/`** — os 8 agentes centrais do CoDriDe (revisor de código, checador de master docs, redator de documentação, planejador de testes, checador de ADR, sincronizador do GitHub, desenvolvedores Python/TypeScript).
- **`GEMINI.md`** — a visão geral do pipeline, o elenco de agentes e os passos de adoção, carregado automaticamente pelo Gemini CLI no início de toda sessão neste projeto.

Você nunca precisa do Claude Code, de `.claude/` ou de `CLAUDE.md` pra nada disso — `.gemini/` e `GEMINI.md` são completos e autocontidos.

## Como começar

Os mesmos seis passos do Início Rápido do Claude Code, só que rodando pelo Gemini CLI:

```
/engineer:doctor        # checagem pré-voo
/meta:preferences       # idioma da documentação, extensível pra mais depois
/engineer:discover      # gera docs/technical-context/
                         # depois escreva docs/business-context/ via /bootstrap:business-docs
/warm-up                # começa uma sessão
```

Daí em diante, o pipeline completo (trilha de produto, trilha de engenharia, `/engineer:pre-pr`, `/engineer:pr`) funciona exatamente como documentado no [README](../../README_PT-BR.md) principal — nomes de comando, argumentos e o fluxo geral são idênticos. Work items caem em `.gemini/work/<type>/<slug>/` (espelhando `.claude/work/` no Claude Code, mas sem ser compartilhado com ele — veja "O que é diferente" abaixo).

## O que é diferente do Claude Code

**Comandos orquestradores rodam sequencialmente, não em paralelo.** O `/engineer:pre-pr` normalmente roda `/engineer:validate` e `/engineer:review` de forma concorrente no Claude Code. No Gemini CLI, esta geração roda cada passo do `/engineer:pre-pr` um atrás do outro em vez disso. Isso não é uma limitação do CoDriDe — o Gemini CLI tem sim despacho paralelo nativo de subagentes, mas ele é experimental e com problemas conhecidos até o momento em que isto foi gerado, então o CoDriDe usa por padrão o caminho sequencial, mais seguro, até isso amadurecer. Espere que o `/engineer:pre-pr` demore um pouco mais aqui do que a mesma varredura no Claude Code; nada nas checagens em si é diferente.

**Os nomes das ferramentas dos agentes não estão verificados.** O vocabulário próprio de nomes de ferramentas do Gemini CLI não foi confirmado no momento desta geração — o campo `tools:` do frontmatter de cada um dos 8 agentes carrega os nomes de ferramentas do Claude Code (`Read`, `Glob`, `Grep`, `Bash`, etc.) como um placeholder sinalizado, não como um chute disfarçado de fato. Cada arquivo de agente tem um comentário acima de `tools:` avisando isso. Se um agente não se comportar como esperado, esse é o primeiro lugar pra checar — confirme os nomes reais das ferramentas na sua versão do Gemini CLI e atualize de acordo.

**Sem suporte a Artifacts.** O `/meta:preferences` do Claude Code inclui uma configuração pra publicar Artifacts (páginas hospedadas no claude.ai) — esse recurso não existe no Gemini CLI, então está totalmente ausente aqui. Nesta geração, o `/meta:preferences` só configura o idioma da documentação.

**Sem checagem de atualização do alvo gerado.** O `/engineer:doctor` do Claude Code inclui uma checagem de se um alvo gerado (como este) ficou dessincronizado da fonte canônica upstream. Essa checagem é contabilidade interna de quem mantém o próprio CoDriDe e não se aplica ao seu projeto — ela é excluída desta geração. Se você suspeitar que sua cópia de `.gemini/`/`GEMINI.md` está desatualizada, peça pra quem mantém a fonte do seu CoDriDe regenerar e recopiar — isso não é algo pra fazer de dentro do seu próprio projeto.

**O `/meta:create-agent` cria agentes nativos do Gemini.** Ele escreve arquivos `.gemini/agents/project-<name>.md` com a mesma ressalva de ferramentas não verificadas dos 8 agentes centrais — genuinamente adaptado pro Gemini CLI, não uma tradução mecânica da versão do Claude Code.

## Fonte da verdade

Tudo sob `.gemini/` e `GEMINI.md` é gerado a partir da fonte canônica do CoDriDe no Claude Code, mantida upstream. Não edite os arquivos gerados à mão — as mudanças se perdem na próxima vez que a fonte for regenerada (um overwrite completo, nunca um merge). Se você precisar mudar algo, isso acontece upstream, no próprio repositório do CoDriDe.
