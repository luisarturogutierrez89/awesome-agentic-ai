---
name: level2-workflow
description: Disciplina de orquestacion para CONSTRUIR una feature o un cambio multi-archivo con supervision humana (nivel 2), delegando en el equipo de subagentes. Usalo solo cuando el usuario pida explicitamente implementar, construir, migrar o refactorizar algo no trivial. NO lo uses para responder preguntas sobre el codigo, explicar como funciona algo, arreglar un typo o un cambio de una linea, ni en conversacion general: convocar al equipo para eso cuesta mucho mas de lo que aporta.
---

# Workflow nivel 2 · autonomia supervisada

Cuando la tarea sea construir o cambiar codigo no trivial, coordina asi:

1. **Triage antes de convocar.** Mide la tarea primero. Si es trivial — un
   archivo, sin comportamiento nuevo, sin tocar seguridad ni contratos de API —
   hazla directo y dilo; no levantes el pipeline. El equipo completo es para
   features y cambios multi-archivo o sensibles. Este paso es el que evita
   gastar 8 subagentes en un fix de una linea.

2. **Planea primero, y endurece el plan.** Para cambios de mas de un archivo o no
   triviales, entra en plan mode: propon un plan y espera la aprobacion del
   humano antes de editar nada. Plan mode es preferible porque bloquea los edits
   en el harness, no solo por instruccion.
   El plan puede venir del planificador nativo o de `architect`; en cualquier
   caso, si el cambio toca autenticacion, sesiones, secretos, entradas de
   usuario, subida de archivos o permisos, NO lo lleves al gate 1 sin su seccion
   de restricciones de seguridad. Si el plan nativo no la trae, delega en
   `architect` para que la agregue: complementa el plan, no lo rehace.

3. **Actua como orquestador.** Coordina y delega; manten el plan al frente de tu
   contexto y evita ensuciarlo con detalles de implementacion. No hagas tu el
   trabajo de los subagentes.

4. **Delega segun el paso:**
   - `product-analyst` para clarificar requerimientos y criterios de aceptacion,
     ANTES de planear lo tecnico.
   - `architect` para el plan tecnico.
   - `coder` para implementar el plan aprobado.
   - `tester` para verificar.
   - `reviewer` para revisar el diff antes de entregar.
   - `debugger` cuando algo se rompe (errores, pruebas rotas).
   - `docs-writer` para actualizar la documentacion tras un cambio.
   - `security-auditor` para un pase de seguridad en cambios sensibles.

5. **Secuencial para construir, paralelo para revisar.** La construccion es una
   cadena de dependencias: product-analyst -> architect -> coder -> tester. Corre
   esos pasos en orden, uno tras otro (no puedes codear antes del plan ni probar
   antes del codigo). En cambio, cuando el diff ya este listo, dispara la revision
   en PARALELO: `reviewer`, `security-auditor` y `docs-writer` leen el mismo diff
   de forma independiente, asi que corren a la vez sin chocar. No paralelices
   agentes que escriben en los mismos archivos.

6. **Cierra el loop antes de entregar.** El flujo no termina en el primer diff:
   se corrige solo, de forma acotada, antes de pasarle el control al humano.
   - Si el `tester` falla: `debugger` -> `tester` otra vez. Maximo 2 vueltas. No
     pases a la revision con las pruebas en rojo.
   - Si la revision arroja hallazgos BLOQUEANTES: `coder` (o `debugger`) los
     arregla, `tester` re-verifica, y se re-dispara solo al agente que los
     reporto. Maximo 2 vueltas. Bloqueante = critico/importante del `reviewer` y
     critico/ALTO del `security-auditor`: las dos escalas cuentan, un "alto" de
     seguridad no es menor. Cuenta tambien como bloqueante, con cualquier
     etiqueta, lo que cruce la frontera entre inquilinos o inutilice un
     mecanismo de escape de seguridad: esos se gradúan por la frontera que
     rompen, no por lo dificil que sea llegar. Los MENORES (y medio/bajo) se reportan sin arreglar,
     para que el ciclo no se eternice.
   - Pasa los hallazgos al `coder` EN LOTES por area si son mas de tres. Todos
     de golpe le agotan el contexto y hay que relanzarlo: llamadas extra que no
     arreglan nada nuevo.
   - Criterio de salida: pruebas en verde y cero hallazgos bloqueantes vivos. Lo que siga vivo tras 2 vueltas se escala al humano marcado
     como PENDIENTE; nunca se esconde ni se minimiza.
   - FRENO: si un arreglo exige cambiar el plan aprobado, el loop PARA y vuelve
     al gate 1. Un loop que rediseña la solucion solo ya no es nivel 2.

7. **Dos puntos de validacion humana:**
   - Gate 1: el humano aprueba el plan antes de ejecutar.
   - Gate 2: al terminar, resume el diff y que se verifico para que el humano
     valide el output.

8. **Seguridad temprano y contexto ligero.** Dos economias que importan:
   - Las restricciones de seguridad van en el PLAN, no solo en la auditoria
     final. Un hallazgo atrapado por el `architect` cuesta una fraccion de lo
     que cuesta devolverle el trabajo al `coder` para rehacerlo.
   - No acumules los reportes completos de los subagentes en tu contexto:
     quedate con la conclusion accionable y suelta el detalle. Si te quedas sin
     ventana a media tarea, el trabajo se corta donde mas duele.

9. **Operaciones de git.** Pregunta antes de operaciones destructivas y no hagas
   commits salvo que se pida explicitamente.
