# AI Engineering Levels — Workflow nivel 2 para Claude Code

Un marketplace de Claude Code con un plugin, **crew**, que empaqueta
un equipo de subagentes para trabajar en modo "autonomia supervisada" (nivel 2):
los agentes planean, implementan, prueban y revisan; tu apruebas el plan y
validas el resultado.

## Que trae el plugin

- **8 subagentes**: `product-analyst`, `architect`, `coder`, `tester`,
  `reviewer`, `debugger`, `docs-writer`, `security-auditor`.
- **1 skill de orquestacion** (`level2-workflow`): la disciplina plan -> validacion,
  que Claude carga solo cuando la tarea es planear o construir.
- **1 comando de arranque** (`/crew:kickoff`): dispara el pipeline
  completo para una feature.

## Instalar (cualquier dev, cualquier maquina)

Hay dos formas equivalentes. **Desde la terminal**:

```bash
claude plugin marketplace add luisarturogutierrez89/awesome-agentic-ai
claude plugin install crew@ai-eng-levels
```

O **dentro de una sesion de Claude Code** (arranca `claude` primero, y ahi
escribes):

```
/plugin marketplace add luisarturogutierrez89/awesome-agentic-ai
/plugin install crew@ai-eng-levels
```

Ojo: `/plugin` solo existe DENTRO de la sesion. Si lo escribes en tu shell vas a
ver `zsh: no such file or directory: /plugin`; ahi usa la variante `claude plugin`.

Reinicia la sesion para que cargue. Para actualizar despues de un cambio en el
repo: `claude plugin update crew@ai-eng-levels` (el ID completo con
marketplace; `claude plugin update crew` a secas falla con "Plugin not found").

## Usarlo

Arranca una sesion en plan mode dentro de tu proyecto y dispara el flujo:

```
claude --permission-mode plan
```

```
/crew:kickoff clasificar el tipo de recibo antes de parsearlo
```

Veras al `product-analyst` clarificar requerimientos, luego al `architect`
proponer un plan que tu apruebas (gate 1), y despues coder/tester/reviewer
ejecutar, para que tu valides el diff al final (gate 2).

## Si lo forkeas: personaliza

1. En `.claude-plugin/marketplace.json`: cambia `owner.name`, `owner.url` y el
   `name` del marketplace (`ai-eng-levels`). El `name` debe ser kebab-case y
   unico para ti: cada usuario solo puede registrar un marketplace por nombre.
2. En `plugins/crew/.claude-plugin/plugin.json`: pon tu `author.name`.
3. Valida antes de compartir:
   ```
   claude plugin validate .
   ```

## Versionado

`plugin.json` trae `version: 0.2.0`. Sube ese numero en cada release para que
los usuarios reciban la actualizacion. Si prefieres que sigan siempre el ultimo
commit, quita el campo `version` y Claude Code usara el SHA del commit.

## Usarlo en el trabajo (equipos gestionados)

En una maquina administrada por tu empresa, los "managed settings" pueden
restringir que marketplaces se pueden agregar (`strictKnownMarketplaces`). Si
esta bloqueado, no podras agregar un marketplace personal ahi: pide a quien
administra Claude Code que lo permita, o que lo distribuya a todo el equipo via
Organization settings > Plugins. Para compartir con tu equipo por repo, se puede
declarar en el `.claude/settings.json` del proyecto con `extraKnownMarketplaces`
y `enabledPlugins` para que se registre solo al confiar en la carpeta.

## Avisos

- **No metas nada confidencial** en este repo si va a ser publico: nada de
  documentos internos, specs de clientes, credenciales ni datos de la empresa.
  Los agentes aqui son genericos a proposito.
- Trata cualquier plugin de terceros como codigo que corres con confianza:
  revisalo y fijalo a una version.

## Licencia

MIT (ajustala si quieres).
