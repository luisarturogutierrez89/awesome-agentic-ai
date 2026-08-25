# crew

Equipo de subagentes para trabajar con Claude Code en modo "autonomia
supervisada" (nivel 2): los agentes hacen el trabajo, tu apruebas el plan y
validas el resultado.

## Pipeline

```
product-analyst -> architect -> [tu apruebas el plan] -> coder -> tester -> reviewer -> [tu validas]
```

Segun la situacion entran: `debugger` (cuando algo se rompe), `docs-writer`
(documentacion) y `security-auditor` (cambios sensibles).

## Subagentes

| Agente             | Cuando se usa                                              | Toca codigo |
|--------------------|-----------------------------------------------------------|-------------|
| product-analyst    | Al inicio: clarifica requerimientos y criterios de aceptacion | No       |
| architect          | Disena el plan tecnico antes de codear                    | No          |
| coder              | Implementa el plan aprobado                               | Si          |
| tester             | Escribe/corre pruebas y verifica                         | Si          |
| reviewer           | Revisa el diff (calidad general) antes de entregar       | No          |
| debugger           | Causa raiz + arreglo minimo cuando algo falla            | Si          |
| docs-writer        | Actualiza documentacion tras un cambio                   | Solo docs   |
| security-auditor   | Pase de seguridad sobre el diff en cambios sensibles     | No          |

## Componentes

- `agents/` — los 8 subagentes.
- `skills/level2-workflow/` — la disciplina de orquestacion (plan -> validacion).
- `commands/kickoff.md` — comando `/crew:kickoff <descripcion>` que
  dispara el pipeline completo.
