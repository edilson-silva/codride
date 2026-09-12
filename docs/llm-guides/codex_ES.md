# CoDriDe en Codex CLI

Esta guía es para adoptar CoDriDe en un proyecto que usa [Codex CLI](https://developers.openai.com/codex) en lugar de (o junto con) Claude Code o Gemini CLI. No asume familiaridad con Claude Code — todo aquí es nativo de Codex CLI.

## Qué copias

Desde el [repositorio de CoDriDe](https://github.com/edilson-silva/codride), copia dos cosas a tu proyecto:

```bash
cp -r codride/.agents codride/AGENTS.md /path/to/your-project/
```

- **`.agents/skills/`** — las skills de CoDriDe, un directorio por skill (`.agents/skills/<name>/SKILL.md`), nombradas `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) para no colisionar sin un espacio de nombres basado en directorios. Dos skills — `python-developer`, `typescript-developer` — conservan un nombre plano, ya que nunca estuvieron en un namespace.
- **`AGENTS.md`** — la visión general del pipeline, el mapeo de agente a skill, y los pasos de adopción, cargado automáticamente por Codex CLI (recorrido desde tu directorio de trabajo hasta la raíz del repositorio).

Nunca necesitas Claude Code, `.claude/` ni `CLAUDE.md` para nada de esto — `.agents/` y `AGENTS.md` son completos y autocontenidos.

## Cómo empezar

Los mismos seis pasos que el Inicio Rápido de Claude Code, invocados como Codex Skills:

```
engineer-doctor      # verificación previa
meta-preferences     # idioma de la documentación, extensible más adelante
engineer-discover     # genera docs/technical-context/
                      # luego escribe docs/business-context/ vía bootstrap-business-docs
warm-up               # inicia una sesión
```

Invoca una skill mencionándola explícitamente (`$engineer-doctor`) o simplemente describe tu tarea en lenguaje natural — Codex seleccionará la skill correspondiente implícitamente cuando tu pedido coincida con su descripción. De ahí en adelante, el pipeline completo funciona exactamente como está documentado en el [README](../../README_ES.md) principal — mismo flujo general, mismos comandos por nombre, solo que se accede a ellos de forma distinta. Los work items caen en `.codex/work/<type>/<slug>/` — una ubicación específica de Codex, no compartida con `.claude/work/` ni `.gemini/work/` aunque sea el mismo proyecto.

## Qué es diferente de Claude Code

Este objetivo tiene dos diferencias estructurales, no solo cosméticas — lee ambas antes de asumir que un comando se comporta como en Claude Code.

**Ninguna delegación a subagentes.** Claude Code tiene 8 subagentes centrales, cada uno un archivo separado al que el orquestador puede delegar — incluyendo ejecutar algunos de forma concurrente (el paso `validate`+`review` de `/engineer:pre-pr`). Codex CLI no tiene ningún mecanismo de delegación, ni siquiera delegación secuencial a un archivo separado. Seis de esos agentes se insertan directamente en la única skill que solía invocarlos — `engineer-validate`, `engineer-review`, `engineer-sync-docs`, `engineer-coverage`, `engineer-discover`, `engineer-work` y `product-sync-github` (siete skills, ya que `engineer-discover` y `engineer-work` cargan cada una, de forma independiente, una copia completa de la lógica de verificación de cumplimiento de ADR del sexto agente, duplicada en lugar de compartida — Codex tampoco tiene un mecanismo de referencia para eso). Los dos agentes bajo demanda sin invocador fijo (`python-developer`, `typescript-developer`) se convirtieron en skills independientes, invocadas directamente en lugar de delegadas.

**Sin sintaxis formal de argumentos.** Los comandos de Claude Code leen un bloque `#$ARGUMENTS` sustituido; Gemini CLI usa `{{args}}`. Las Codex Skills no tienen ninguno de los dos — confirmado directamente contra la documentación oficial de Codex, no asumido. Una Skill no tiene ninguna forma templada de recibir parámetros; funciona enteramente a partir del contexto en lenguaje natural de tu pedido. Cada skill que solía leer un argumento (un slug de work item, un número de issue, una descripción de feature) ahora instruye a Codex a extraer esa información de lo que realmente escribiste, y a preguntarte en lugar de adivinar si es ambiguo. En la práctica: en lugar de escribir algo equivalente a `/engineer:context feat/csv-order-export`, dirías algo como "inicia el context para feat/csv-order-export" y la skill lee el slug de tu frase.

**El nombrado de las skills no tiene namespace.** `.agents/skills/<name>/` es plano — no existe un subdirectorio por espacio de nombres como sí tienen Claude Code (`.claude/commands/engineer/`) o Gemini CLI (`.gemini/commands/engineer/`). Las skills de CoDriDe se nombran `<namespace>-<name>` en su lugar (`engineer-validate`, `product-validate` — ambas existen y no colisionan) para preservar la misma distinción sin una carpeta que lo haga.

**Sin soporte para Artifacts.** Igual que Gemini CLI — el `/meta:preferences` de Claude Code configura la publicación de Artifacts (páginas alojadas en claude.ai); esa función no existe aquí, así que está totalmente ausente de `meta-preferences`.

**Sin verificación de actualización del objetivo generado.** Igual que Gemini CLI — la verificación interna de contabilidad que marca un objetivo generado desactualizado en el `/engineer:doctor` de Claude Code no aplica a tu proyecto; está excluida de esta generación. Si tu copia de `.agents/`/`AGENTS.md` parece desactualizada, pide a quien mantiene la fuente de tu CoDriDe que la regenere.

**`meta-create-agent` crea skills nativas de Codex.** Escribe archivos `.agents/skills/project-<name>/SKILL.md`, con una omisión deliberada: los subagentes de Claude Code llevan un allowlist `tools:` que delimita exactamente qué puede acceder ese agente; las Codex Skills no tienen un campo equivalente de restricción de herramientas, así que `meta-create-agent` no intenta inventar uno — una skill que creas es instrucciones más frontmatter, nada más.

## Fuente de la verdad

Todo bajo `.agents/` y `AGENTS.md` se genera a partir de la fuente canónica de CoDriDe en Claude Code, mantenida upstream. No edites a mano los archivos generados — los cambios se pierden la próxima vez que la fuente se regenere (un overwrite completo, nunca un merge). Si necesitas cambiar algo, eso sucede upstream, en el propio repositorio de CoDriDe.
