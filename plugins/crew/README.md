# crew

Equipo de subagentes para trabajar con Claude Code en modo "autonomia
supervisada" (nivel 2): los agentes hacen el trabajo, tu apruebas el plan y
validas el resultado.

## Cuando NO usar el pipeline

El equipo completo son 8 subagentes, y cada uno corre sus propias peticiones. Para
un typo, un rename o un fix de una linea cuesta mucho mas de lo que aporta: el
flujo hace triage primero y resuelve directo los cambios triviales. El pipeline
es para features y cambios multi-archivo o sensibles.

## Resultados medidos

Cuatro issues reales de un monorepo TypeScript (React + Fastify + Postgres),
midiendo con el panel de uso de Claude Code. Son porcentajes del consumo total,
no tokens absolutos.

| | #36 auth | #21 contenido | #45 privacidad |
|---|---|---|---|
| Version del plugin | inicial | 0.5.0 | 0.6.0 |
| Despachos de agentes | 9 | 21 | **11** |
| Pasadas de `coder` | 2 | 5 | **1** |
| Rondas de correccion | 1 (manual) | 3 | **0** |
| Hallazgos critico/alto | 3 | 2 criticos + 7 altos | **0** |
| `coder` (% del consumo) | 40% | 10% | 10% |
| `architect` | no corrio | no corrio | **1%** |
| Plugin (% del consumo) | 63% | 74% | 67% |

**Lo caro es implementar; la supervision es barata.** Una llamada a `coder`
costo ~7x lo que una auditoria de seguridad y ~20x lo que un plan. De ahi el
diseño: las restricciones de seguridad se fijan en el PLAN, no en la auditoria
posterior. En el #45 `architect` costo **1% del consumo** y atrapo cuatro
discrepancias del plan —incluida una que habria llegado rota a produccion—
antes de escribir una sola linea. `coder` corrio una vez en vez de cinco.

Que si aporta valor, medido: en el #21 la revision encontro dos fallos criticos
de autorizacion (cualquier `viewer` podia publicar y borrar via API directa) mas
siete altos, todos arreglados por los loops antes de llegar al humano.

**Advertencias honestas.** Las tareas no son equivalentes: el #21 fue full-stack
con superficie nueva y el #45 fue frontend sobre una base ya auditada, asi que
parte de la caida a cero hallazgos es el tamaño del cambio, no solo el plugin.
Son cuatro corridas, no un experimento controlado. Y el `security-auditor` cobra
su ~10% cada vez, encuentre algo o no: es una prima de seguro, no un gasto que
se recupera en cada corrida.

**El pipeline no es tu mayor gasto.** En estas mediciones, el consumo dominante
fueron sesiones de 8+ horas y contextos arriba de 150k, no los subagentes. Usa
`/clear` al cambiar de tarea antes de venir a recortar agentes.

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
