# crew

Equipo de subagentes para trabajar con Claude Code en modo "autonomia
supervisada" (nivel 2): los agentes hacen el trabajo, tu apruebas el plan y
validas el resultado.

## Cuando NO usar el pipeline

El equipo completo son 8 subagentes, y cada uno corre sus propias peticiones. Para
un typo, un rename o un fix de una linea cuesta mucho mas de lo que aporta: el
flujo hace triage primero y resuelve directo los cambios triviales. El pipeline
es para features y cambios multi-archivo o sensibles.

Referencia de costo relativo, medida en una migracion de autenticacion real
(9 despachos): una llamada a `coder` costo ~7x lo que una auditoria de seguridad
y ~20x lo que un plan. Lo caro es implementar; la supervision es barata. Por eso
conviene mover la seguridad al PLAN en vez de dejarla a la auditoria final: un
hallazgo temprano evita rehacer la implementacion.

## Pipeline

```
product-analyst -> architect -> [GATE 1: tu apruebas el plan]
                                          |
                                          v
                              coder -> tester <--+
                                          |      | falla: debugger (max 2)
                                          +------+
                                          |
                                     pruebas en verde
                                          |
                                          v
                         reviewer + security-auditor + docs-writer
                              (en paralelo, sobre el mismo diff)
                                          |
                              critico o importante? --si--> arregla + re-testea
                                          |                        (max 2 vueltas)
                                          no
                                          v
                              [GATE 2: tu validas el diff]
```

El flujo se auto-corrige antes de entregarte nada: no te pasa un diff con las
pruebas en rojo ni con hallazgos criticos sin atender. Lo que no logre resolver
en 2 vueltas te lo escala marcado como PENDIENTE. Los hallazgos menores se
reportan sin arreglar. Si un arreglo exige cambiar el plan, el loop se detiene y
vuelve al gate 1: el rediseño lo apruebas tu.

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
