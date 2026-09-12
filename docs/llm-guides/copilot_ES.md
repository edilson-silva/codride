# CoDriDe en GitHub Copilot CLI

Esta guía es para adoptar CoDriDe en un proyecto que usa [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli) en lugar de (o junto con) Claude Code, Gemini CLI o Codex CLI. No asume familiaridad con Claude Code — todo aquí es nativo de Copilot CLI.

## Qué copias

Desde el [repositorio de CoDriDe](https://github.com/edilson-silva/codride), copia dos cosas a tu proyecto:

```bash
mkdir -p /path/to/your-project/.github
cp -r codride/.github/agents /path/to/your-project/.github/
cp codride/.github/copilot-instructions.md /path/to/your-project/.github/copilot-instructions.md
```

Revisa esta última línea antes de ejecutarla: muchos repositorios ya tienen su propio `.github/copilot-instructions.md` para la revisión de código de Copilot de GitHub — esto lo sobrescribiría silenciosamente. Si tu repositorio de destino ya tiene uno, combina los dos a mano en lugar de copiar a ciegas.

- **`.github/agents/`** — los comandos de CoDriDe *y* sus 8 agentes centrales, todos en un único directorio plano, un archivo `.agent.md` cada uno. Los comandos se nombran `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) para no colisionar sin un espacio de nombres basado en directorios; los 8 agentes centrales conservan sus nombres originales, planos (`branch-code-reviewer`, `adr-compliance-checker`, etc.), ya que nunca fueron comandos con namespace para empezar.
- **`.github/copilot-instructions.md`** — la visión general del pipeline, el elenco de agentes y los pasos de adopción, cargado automáticamente por Copilot CLI al inicio de cada sesión en este proyecto.

Nunca necesitas Claude Code, `.claude/` ni `CLAUDE.md` para nada de esto — `.github/agents/` y `.github/copilot-instructions.md` son completos y autocontenidos.

## Cómo empezar

Los mismos seis pasos que el Inicio Rápido de Claude Code, invocados como agentes de Copilot:

```
engineer-doctor      # verificación previa
meta-preferences     # idioma de la documentación, extensible más adelante
engineer-discover     # genera docs/technical-context/
                      # luego escribe docs/business-context/ vía bootstrap-business-docs
warm-up               # inicia una sesión
```

Invoca un agente con `/agent-name` interactivamente, con `--agent <name> --prompt "..."` de forma programática, o simplemente describe tu tarea en lenguaje natural — Copilot inferirá el agente correspondiente a partir de su `description`. De ahí en adelante, el pipeline completo (vía de producto, vía de ingeniería, `engineer-pre-pr`, `engineer-pr`) funciona exactamente como está documentado en el [README](../../README_ES.md) principal — mismo flujo general, mismos comandos por nombre, solo que se accede a ellos de forma distinta (ver "Sin sintaxis formal de argumentos" abajo). Los work items caen en `.copilot/work/<type>/<slug>/` — una ubicación específica de Copilot, no compartida con `.claude/work/`, `.gemini/work/` ni `.codex/work/` aunque sea el mismo proyecto.

## Qué es diferente de Claude Code

**Un formato de agente unificado, no dos.** Claude Code separa comandos (`.claude/commands/`) de subagentes (`.claude/agents/`) en directorios y formatos de archivo distintos. Copilot CLI tiene solo uno: `.agent.md`, distinguido por una bandera de frontmatter `user-invocable` — `true` para algo que invocas directamente (un comando), `false` para algo a lo que solo otro agente delega. Los 35 archivos — 27 comandos, 8 agentes centrales — viven todos en un único directorio plano, `.github/agents/`. Dos de los 8 agentes centrales son la excepción: `python-developer` y `typescript-developer` no tienen un comando invocador fijo en Claude Code (se usan "bajo demanda", invocados directamente por una persona), así que ambos conservan `user-invocable: true` aquí — la misma excepción que hace Codex CLI al convertirlos en Skills independientes.

**Delegación real a subagentes — nada se inserta directamente.** A diferencia de Codex CLI, Copilot CLI sí admite que un agente delegue a otro, así que esta traducción preserva cada relación comando→agente exactamente como funciona en Claude Code: `engineer-validate` sigue delegando a `branch-master-docs-checker`, `product-sync-github` sigue delegando a `github-project-sync`, y así sucesivamente — como archivos separados, no como lógica insertada.

**Los comandos orquestadores todavía se ejecutan de forma secuencial, no en paralelo — pero por una razón distinta a la de Gemini.** `engineer-pre-pr` normalmente ejecuta `engineer-validate` y `engineer-review` de forma concurrente en Claude Code. La concurrencia de subagentes de Copilot CLI es real y configurable (no experimental como la de Gemini) — pero está restringida a ciertos niveles de plan y configuraciones explícitas (v1.0.66+, facturación por uso), así que no se puede asumir presente para todo adoptante. Por esa razón, esta generación usa secuencial por defecto. Si tu plan tiene la concurrencia configurada, puedes habilitar tú mismo el despacho paralelo para `engineer-validate`/`engineer-review`.

**Sin sintaxis formal de argumentos.** Los comandos de Claude Code leen un bloque `#$ARGUMENTS` sustituido; Gemini CLI usa `{{args}}`. Los agentes de Copilot CLI no tienen ninguno de los dos — confirmado directamente contra la documentación oficial de GitHub, no asumido. Un agente no tiene ninguna forma templada de recibir parámetros; funciona enteramente a partir del contexto en lenguaje natural de tu pedido o de la bandera `--prompt`. Cada agente que solía leer un argumento (un slug de work item, un número de issue, una descripción de feature) ahora instruye a Copilot a extraer esa información de lo que realmente escribiste, y a preguntarte en lugar de adivinar si es ambiguo.

**Los nombres de herramientas de los agentes no están verificados.** El vocabulario propio de nombres de herramientas de Copilot CLI (`read`, `search`, etc.) se obtuvo de un ejemplo de la comunidad, no de la tabla de referencia completa y oficial de GitHub, al momento de esta generación. El campo `tools:` del frontmatter de cada agente conserva los nombres de herramientas de Claude Code como un placeholder señalado, no como una suposición disfrazada de hecho — un comentario arriba de `tools:` lo advierte. Si un agente no se comporta como se espera, ese es el primer lugar donde revisar.

**Sin soporte para Artifacts.** Igual que Gemini CLI y Codex CLI — el `/meta:preferences` de Claude Code configura la publicación de Artifacts (páginas alojadas en claude.ai); esa función no existe aquí, así que está totalmente ausente de `meta-preferences`.

**Sin verificación de actualización del objetivo generado.** Igual que Gemini CLI y Codex CLI — la verificación interna de contabilidad que marca un objetivo generado desactualizado en el `/engineer:doctor` de Claude Code no aplica a tu proyecto; está excluida de esta generación. Si tu copia de `.github/agents/`/`.github/copilot-instructions.md` parece desactualizada, pide a quien mantiene la fuente de tu CoDriDe que la regenere.

**`meta-create-agent` crea agentes nativos de Copilot.** Escribe archivos `.github/agents/project-<name>.agent.md`, usando el frontmatter real de Copilot — incluyendo un campo `tools:` genuino (a diferencia de Codex, que no tiene ninguno) y, para capacidades respaldadas por MCP, `mcp-servers:` en lugar de una entrada de herramienta individual.

**Bono: Copilot CLI también lee `AGENTS.md`, `CLAUDE.md` y `GEMINI.md` directamente, si están presentes.** Los combina y deduplica con `.github/copilot-instructions.md`, sin un orden de precedencia definido entre ellos. Esto es una ventaja si tu proyecto llega a tener más de un objetivo de CoDriDe en paralelo — no es algo de lo que esta generación dependa, ya que quien adopta solo Copilot nunca tiene esos otros archivos.

## Fuente de la verdad

Todo bajo `.github/agents/` y `.github/copilot-instructions.md` se genera a partir de la fuente canónica de CoDriDe en Claude Code, mantenida upstream. No edites a mano los archivos generados — los cambios se pierden la próxima vez que la fuente se regenere (un overwrite completo, nunca un merge). Si necesitas cambiar algo, eso sucede upstream, en el propio repositorio de CoDriDe.
