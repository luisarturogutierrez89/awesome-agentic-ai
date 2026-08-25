---
description: Arranca el flujo nivel 2 (plan -> validacion) para una feature o cambio.
---

Vas a coordinar el flujo nivel 2 (autonomia supervisada) para lo siguiente:

$ARGUMENTS

Sigue estos pasos, delegando en los subagentes. La construccion es una cadena
SECUENCIAL; la revision corre en PARALELO; y ambas cierran con un loop de
correccion acotado ANTES de pasarme el control a mi.

0. TRIAGE PRIMERO. Antes de convocar a nadie, evalua el tamaño real de la tarea.
   Si es trivial — un archivo, sin comportamiento nuevo, sin tocar seguridad ni
   contratos de API (un typo, renombrar algo, ajustar una constante, un fix de
   una linea) — hazlo TU directamente, avisame que lo hiciste sin pipeline, y
   termina. Convocar 8 subagentes para un cambio de una linea cuesta mucho mas
   de lo que aporta. El pipeline completo es para features y cambios
   multi-archivo o sensibles. Ante la duda, preguntame cual de los dos quiero.

1. Delega en `product-analyst` para clarificar requerimientos y criterios de
   aceptacion. Hazme las preguntas que falten antes de continuar.
2. Con los requerimientos claros, delega en `architect` para el plan tecnico y
   presentamelo. NO toques codigo hasta que yo apruebe el plan.  [gate 1]
3. Tras mi aprobacion, delega SECUENCIALMENTE: `coder` implementa el plan, luego
   `tester` verifica. No los corras en paralelo: el tester necesita el codigo ya
   escrito.
4. LOOP DE PRUEBAS (maximo 2 vueltas). Si el `tester` reporta fallos, delega en
   `debugger` (causa raiz + arreglo minimo) y vuelve a correr `tester`. NO
   avances a la revision con las pruebas en rojo. Si tras 2 vueltas sigue
   fallando, PARA y reportame que falla y que se intento.
5. Con las pruebas en verde, corre la revision en PARALELO: dispara a la vez
   `reviewer` (calidad), `security-auditor` (si el cambio es sensible) y
   `docs-writer` (documentacion). Son de lectura independiente sobre el mismo
   diff, asi que no chocan y ahorras tiempo.
6. LOOP DE CORRECCION (maximo 2 vueltas). Si la revision arroja hallazgos
   CRITICOS o IMPORTANTES:
   a. Delega el arreglo en `coder`, o en `debugger` si es un fallo de
      comportamiento. Arregla SOLO esos hallazgos; no amplies el alcance.
   b. Vuelve a correr `tester` para confirmar que el arreglo no rompio nada.
   c. Re-dispara solo al agente cuyos hallazgos se atendieron, para verificar
      que quedaron resueltos.
   Los hallazgos MENORES no se arreglan: se reportan tal cual.
7. CRITERIO DE SALIDA: pruebas en verde y cero hallazgos criticos o importantes.
   Si despues de 2 vueltas queda alguno vivo, no lo escondas ni lo minimices:
   escalamelo marcado como PENDIENTE, con lo que se intento.
8. FRENO. Si un arreglo requiere cambiar el plan aprobado (no es un fix acotado),
   PARA el loop y regresa al gate 1 conmigo. No cambies la solucion por tu
   cuenta: eso ya no es nivel 2.
9. CUIDA TU CONTEXTO. Cada subagente te devuelve un reporte; si los acumulas
   completos te quedas sin ventana antes de terminar. Quedate con la conclusion
   accionable de cada uno (que hay que arreglar y donde) y suelta el detalle.
   Pideles reportes concisos.
10. Al final juntame los reportes y resume: el diff, que se verifico, cuantas
    vueltas de correccion hubo, que se arreglo en ellas y que quedo
    pendiente. Di tambien que NO se pudo verificar y por que.  [gate 2]
