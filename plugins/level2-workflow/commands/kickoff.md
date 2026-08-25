---
description: Arranca el flujo nivel 2 (plan -> validacion) para una feature o cambio.
---

Vas a coordinar el flujo nivel 2 (autonomia supervisada) para lo siguiente:

$ARGUMENTS

Sigue estos pasos, delegando en los subagentes. Recuerda: la construccion es una
cadena SECUENCIAL (cada paso depende del anterior); solo la revision final se
hace en PARALELO.

1. Delega en `product-analyst` para clarificar requerimientos y criterios de
   aceptacion. Hazme las preguntas que falten antes de continuar.
2. Con los requerimientos claros, delega en `architect` para el plan tecnico y
   presentamelo. NO toques codigo hasta que yo apruebe el plan.  [gate 1]
3. Tras mi aprobacion, delega SECUENCIALMENTE: `coder` implementa el plan, luego
   `tester` verifica. No los corras en paralelo: el tester necesita el codigo ya
   escrito.
4. Cuando el diff este listo, corre la revision en PARALELO: dispara a la vez
   `reviewer` (calidad), `security-auditor` (si el cambio es sensible) y
   `docs-writer` (documentacion). Son de lectura independiente sobre el mismo
   diff, asi que no chocan y ahorras tiempo.
5. Usa `debugger` si algo se rompe en cualquier punto (causa raiz + arreglo minimo).
6. Al final, junta los reportes, resume el diff y que se verifico, para que yo
   valide.  [gate 2]
