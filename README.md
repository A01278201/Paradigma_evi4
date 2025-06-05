# Descripción

**Problema elegido**: Dado un número escrito en base binaria (una cadena que solo contiene 0 y 1), determinar si ese número es divisible por 3.
- Contexto y relevancia
  1. En sistemas embebidos o hardware (p. ej., ALU de un procesador), a menudo se necesitan comprobaciones rápidas de divisibilidad sin convertir a decimal. Un AFD permite verificar “divisible entre 3” procesando bit a bit y manteniendo únicamente el residuo módulo 3.

  2. En redes de comunicaciones, ciertas codificaciones binarias requieren comprobaciones de redundancia basadas en divisibilidad, y es más eficiente hacerlo con un autómata que con aritmética de enteros, sobre todo cuando se trabaja con secuencias muy largas.

- Paradigma que utilizaremos: Paradigma lógico (Prolog).
Definiremos hechos move/3 que representan las transiciones de estados del AFD, el o los estados finales con final/1, y luego un predicado automata/1 (y sus auxiliares) para recorrer la cadena.




