# CoDriDe en Gemini CLI

Esta guía es para adoptar CoDriDe en un proyecto que usa [Gemini CLI](https://github.com/google-gemini/gemini-cli) en lugar de (o junto con) Claude Code. No asume familiaridad con Claude Code — todo aquí es nativo de Gemini CLI.

## Qué copias

Desde el [repositorio de CoDriDe](https://github.com/edilson-silva/codride), copia dos cosas a tu proyecto:

```bash
cp -r codride/.gemini codride/GEMINI.md /path/to/your-project/
```

- **`.gemini/commands/`** — los slash commands de CoDriDe (`/engineer:context`, `/product:spec`, etc.), un archivo TOML por comando, organizados en namespaces igual que los de Claude Code (`engineer/`, `product/`, `bootstrap/`, `meta/`).
- **`.gemini/agents/`** — los 8 agentes centrales de CoDriDe (revisor de código, verificador de master docs, redactor de documentación, planificador de pruebas, verificador de ADR, sincronizador de GitHub, desarrolladores Python/TypeScript).
- **`GEMINI.md`** — la visión general del pipeline, el elenco de agentes y los pasos de adopción, cargado automáticamente por Gemini CLI al inicio de cada sesión en este proyecto.

Nunca necesitas Claude Code, `.claude/` ni `CLAUDE.md` para nada de esto — `.gemini/` y `GEMINI.md` son completos y autocontenidos.

## Cómo empezar

Los mismos seis pasos que el Inicio Rápido de Claude Code, solo que ejecutados a través de Gemini CLI:

```
/engineer:doctor        # verificación previa
/meta:preferences       # idioma de la documentación, extensible más adelante
/engineer:discover      # genera docs/technical-context/
                         # luego escribe docs/business-context/ vía /bootstrap:business-docs
/warm-up                # inicia una sesión
```

De ahí en adelante, el pipeline completo (vía de producto, vía de ingeniería, `/engineer:pre-pr`, `/engineer:pr`) funciona exactamente como está documentado en el [README](../../README_ES.md) principal — nombres de comando, argumentos y el flujo general son idénticos. Los work items caen en `.gemini/work/<type>/<slug>/` (reflejando `.claude/work/` en Claude Code, pero sin compartirse con él — ver "Qué es diferente" abajo).

## Qué es diferente de Claude Code

**Los comandos orquestadores se ejecutan de forma secuencial, no en paralelo.** `/engineer:pre-pr` normalmente ejecuta `/engineer:validate` y `/engineer:review` de forma concurrente en Claude Code. En Gemini CLI, esta generación ejecuta cada paso de `/engineer:pre-pr` uno tras otro en su lugar. Esto no es una limitación de CoDriDe — Gemini CLI sí tiene despacho paralelo nativo de subagentes, pero es experimental y con problemas conocidos al momento de esta generación, así que CoDriDe usa por defecto el camino secuencial, más seguro, hasta que eso madure. Espera que `/engineer:pre-pr` tarde un poco más aquí que el mismo barrido en Claude Code; nada en las verificaciones en sí es diferente.

**Los nombres de herramientas de los agentes no están verificados.** El vocabulario propio de nombres de herramientas de Gemini CLI no se confirmó al momento de esta generación — el campo `tools:` del frontmatter de cada uno de los 8 agentes conserva los nombres de herramientas de Claude Code (`Read`, `Glob`, `Grep`, `Bash`, etc.) como un placeholder señalado, no como una suposición disfrazada de hecho. Cada archivo de agente tiene un comentario arriba de `tools:` que lo advierte. Si un agente no se comporta como se espera, ese es el primer lugar donde revisar — confirma los nombres reales de las herramientas en tu versión de Gemini CLI y actualiza en consecuencia.

**Sin soporte para Artifacts.** El `/meta:preferences` de Claude Code incluye una configuración para publicar Artifacts (páginas alojadas en claude.ai) — esa función no existe en Gemini CLI, así que está totalmente ausente aquí. En esta generación, `/meta:preferences` solo configura el idioma de la documentación.

**Sin verificación de actualización del objetivo generado.** El `/engineer:doctor` de Claude Code incluye una verificación de si un objetivo generado (como este) se desactualizó respecto a la fuente canónica upstream. Esa verificación es contabilidad interna de quien mantiene CoDriDe y no aplica a tu proyecto — está excluida de esta generación. Si sospechas que tu copia de `.gemini/`/`GEMINI.md` está desactualizada, pide a quien mantiene la fuente de tu CoDriDe que la regenere y la vuelva a copiar — eso no es algo que se haga desde dentro de tu propio proyecto.

**`/meta:create-agent` crea agentes nativos de Gemini.** Escribe archivos `.gemini/agents/project-<name>.md` con la misma advertencia de herramientas no verificadas que los 8 agentes centrales — genuinamente adaptado para Gemini CLI, no una traducción mecánica de la versión de Claude Code.

## Fuente de la verdad

Todo bajo `.gemini/` y `GEMINI.md` se genera a partir de la fuente canónica de CoDriDe en Claude Code, mantenida upstream. No edites a mano los archivos generados — los cambios se pierden la próxima vez que la fuente se regenere (un overwrite completo, nunca un merge). Si necesitas cambiar algo, eso sucede upstream, en el propio repositorio de CoDriDe.
