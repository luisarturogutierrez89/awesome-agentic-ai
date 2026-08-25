---
name: level2-workflow
description: Disciplina de orquestacion para construir o modificar codigo con supervision humana (nivel 2). Usalo cuando se planee o implemente una feature, un cambio no trivial o multi-archivo: define como delegar en los subagentes y donde van los dos puntos de validacion humana.
---

# Workflow nivel 2 · autonomia supervisada

Cuando la tarea sea construir o cambiar codigo no trivial, coordina asi:

1. **Planea primero.** Para cambios de mas de un archivo o no triviales, entra en
   plan mode: propon un plan y espera la aprobacion del humano antes de editar nada.

2. **Actua como orquestador.** Coordina y delega; manten el plan al frente de tu
   contexto y evita ensuciarlo con detalles de implementacion. No hagas tu el
   trabajo de los subagentes.

3. **Delega segun el paso:**
   - `product-analyst` para clarificar requerimientos y criterios de aceptacion,
     ANTES de planear lo tecnico.
   - `architect` para el plan tecnico.
   - `coder` para implementar el plan aprobado.
   - `tester` para verificar.
   - `reviewer` para revisar el diff antes de entregar.
   - `debugger` cuando algo se rompe (errores, pruebas rotas).
   - `docs-writer` para actualizar la documentacion tras un cambio.
   - `security-auditor` para un pase de seguridad en cambios sensibles.

4. **Secuencial para construir, paralelo para revisar.** La construccion es una
   cadena de dependencias: product-analyst -> architect -> coder -> tester. Corre
   esos pasos en orden, uno tras otro (no puedes codear antes del plan ni probar
   antes del codigo). En cambio, cuando el diff ya este listo, dispara la revision
   en PARALELO: `reviewer`, `security-auditor` y `docs-writer` leen el mismo diff
   de forma independiente, asi que corren a la vez sin chocar. No paralelices
   agentes que escriben en los mismos archivos.

5. **Cierra el loop antes de entregar.** El flujo no termina en el primer diff:
   se corrige solo, de forma acotada, antes de pasarle el control al humano.
   - Si el `tester` falla: `debugger` -> `tester` otra vez. Maximo 2 vueltas. No
     pases a la revision con las pruebas en rojo.
   - Si la revision arroja hallazgos CRITICOS o IMPORTANTES: `coder` (o
     `debugger`) los arregla, `tester` re-verifica, y se re-dispara solo al
     agente que los reporto. Maximo 2 vueltas. Los MENORES se reportan sin
     arreglar, para que el ciclo no se eternice.
   - Criterio de salida: pruebas en verde y cero hallazgos criticos o
     importantes. Lo que siga vivo tras 2 vueltas se escala al humano marcado
     como PENDIENTE; nunca se esconde ni se minimiza.
   - FRENO: si un arreglo exige cambiar el plan aprobado, el loop PARA y vuelve
     al gate 1. Un loop que rediseña la solucion solo ya no es nivel 2.

6. **Dos puntos de validacion humana:**
   - Gate 1: el humano aprueba el plan antes de ejecutar.
   - Gate 2: al terminar, resume el diff y que se verifico para que el humano
     valide el output.

7. **Seguridad.** Pregunta antes de operaciones destructivas de git y no hagas
   commits salvo que se pida explicitamente.
