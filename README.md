# Descripción

**Problema elegido**: Dado un número escrito en base binaria (una cadena que solo contiene 0 y 1), determinar si ese número es divisible por 3.
- Contexto y relevancia
  1. En sistemas embebidos o hardware (p. ej., ALU de un procesador), a menudo se necesitan comprobaciones rápidas de divisibilidad sin convertir a decimal. Un AFD permite verificar “divisible entre 3” procesando bit a bit y manteniendo únicamente el residuo módulo 3.

  2. En redes de comunicaciones, ciertas codificaciones binarias requieren comprobaciones de redundancia basadas en divisibilidad, y es más eficiente hacerlo con un autómata que con aritmética de enteros, sobre todo cuando se trabaja con secuencias muy largas.

- Paradigma que utilizaremos: Paradigma lógico (Prolog).
Definiremos hechos `move/3` que representan las transiciones de estados del AFD, el o los estados finales con `final/1`, y luego un predicado `automata/1` (y sus auxiliares) para recorrer la cadena.


# Modelos (lógica y diagrama)

**Explicación del AFD**

Para decidir si un número binario b<sub>n−1</sub>…b<sub>1</sub>b<sub>0</sub> (cada b<sub>i</sub> ∈ {0,1}) es divisible por 3, basta con leer los bits de izquierda a derecha y calcular el residuo módulo 3 que va cambiando. Si al final el residuo es 0, es divisible.

- Cada vez que “vimos” un bit x en el número actual con residuo r, el nuevo residuo será (2·r + x) mod 3 (porque al desplazar a la izquierda en binario, el valor anterior se multiplica por 2, y luego sumamos el bit x).

Definimos 3 estados:

- S0: residuo = 0

- S1: residuo = 1

- S2: residuo = 2

**Estado inicial**: S0 (antes de leer ningún bit, el residuo de “cadena vacía” = 0).

**Estado(es) final(es)**: S0 (solo si al terminar, residuo = 0, aceptamos que es divisible por 3).

Las transiciones según el bit leído (0 o 1) quedan definidas por:
```
    Estado actual  |  Bit leído  |  Nuevo estado  |  Cálculo del residuo
   ---------------|-------------|----------------|--------------------------------
      S0 (r=0)    |     0       |     S0 (r=0)   |  (2·0 + 0) mod 3 = 0  
      S0 (r=0)    |     1       |     S1 (r=1)   |  (2·0 + 1) mod 3 = 1  
      S1 (r=1)    |     0       |     S2 (r=2)   |  (2·1 + 0) mod 3 = 2  
      S1 (r=1)    |     1       |     S0 (r=0)   |  (2·1 + 1) mod 3 = 3 mod 3 = 0  
      S2 (r=2)    |     0       |     S1 (r=1)   |  (2·2 + 0) mod 3 = 4 mod 3 = 1  
      S2 (r=2)    |     1       |     S2 (r=2)   |  (2·2 + 1) mod 3 = 5 mod 3 = 2  
```

**Diagrama del AFD**

A continuación se muestra el diagrama del autómata con sus transiciones y estados finales. El estado S0 (residuo 0) está dibujado con doble círculo, indicando el estado de aceptación:


Explicación del diagrama

![automata binarios](https://github.com/user-attachments/assets/8740fbf1-d9bd-4394-bb35-fdc3f4f3cd3b)


El círculo S0 (residuo 0) es doble porque es el estado final.

Desde S0, con 0 volvemos a S0; con 1 vamos a S1.

Desde S1, con 0 pasamos a S2; con 1 volvemos a S0.

Desde S2, con 0 vamos a S1; con 1 permanecemos en S2. 



