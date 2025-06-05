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

# Implementación
```
% Hechos move(CurrentState, NextState, Symbol):
%   representan las transiciones del DFA que calcula residuo mod 3.
move(s0, s0, '0').   % (2*0 + 0) mod 3 = 0  → sigue en s0
move(s0, s1, '1').   % (2*0 + 1) mod 3 = 1  → pasa a s1

move(s1, s2, '0').   % (2*1 + 0) mod 3 = 2  → pasa a s2
move(s1, s0, '1').   % (2*1 + 1) mod 3 = 0  → pasa a s0

move(s2, s1, '0').   % (2*2 + 0) mod 3 = 1  → pasa a s1
move(s2, s2, '1').   % (2*2 + 1) mod 3 = 2  → sigue en s2

% Sólo el estado s0 (residuo = 0) es final/aceptación.
final(s0).

% Predicado principal:
%   automata(ListaDeSimbolosBinarios)
%   → Si la lista contiene sólo '0' y '1', simula el DFA y muestra si es divisible.
automata(Lista) :-
    aux_automata(Lista, s0).

% Caso base: cadena vacía y EstadoActual es final → aceptamos
aux_automata([], EstadoActual) :-
    final(EstadoActual),
    writeln('La cadena BINARIA es divisible por 3.').

% Caso base inverso: cadena vacía y EstadoActual no es final → rechazamos
aux_automata([], EstadoActual) :-
    \+ final(EstadoActual),
    writeln('La cadena BINARIA NO es divisible por 3.'), !, fail.

% Caso recursivo: consumimos el primer símbolo y aplicamos move/3
aux_automata([Simbolo | Resto], EstadoActual) :-
    move(EstadoActual, EstadoSiguiente, Simbolo),
    aux_automata(Resto, EstadoSiguiente).

% -------------------------------
% Ejemplos de consulta (pégalas en el recuadro de Query de SWISH):
%
% ?- automata(['0']).            % 0 en binario → divisible
% ?- automata(['1','1']).        % 3 en binario → divisible
% ?- automata(['1','0']).        % 2 en binario → NO divisible
% ?- automata(['1','1','0']).    % 6 en binario → divisible
% ?- automata(['1','0','0','1']).% 9 en binario → divisible
% ?- automata(['1','0','0']).    % 4 en binario → NO divisible
% ?- automata(['1','1','1','0','1','1']).  % 59 en binario → NO divisible
% ?- automata(['1','0','1','1','0']).      % 22 en binario → NO divisible
% ?- automata([]).               % cadena vacía → divisible (interpreta 0)
% -------------------------------
```

Estos seis hechos (`move/3`) definen la función de transición del autómata. Cada cláusula `move(EstadoActual, EstadoSiguiente, Símbolo)` indica:

- **EstadoActual**: el estado en el que se encuentra el automáta antes de leer el próximo bit.

- **Símbolo**: el carácter que el autómata lee (en este caso, `'0'` o `'1'`).

- **EstadoSiguiente**: a qué nuevo estado salta el autómata después de procesar ese símbolo.

Imaginemos que “s0”, “s1” y “s2” representan el resto actual (módulo 3) que llevamos al interpretar la parte de la cadena ya leída. En efecto:

- **s0** equivale a “resto 0”

- **s1** equivale a “resto 1”

- **s2** equivale a “resto 2”

En resumen:

- move/3 define la tabla de transición del AFD, basada en cómo se calcula el resto módulo 3 al añadir un bit.

- final/1 declara qué estado(es) son de aceptación (en este caso, solo s0).

- automata/1 es la puerta de entrada: toma una lista de bits y arranca la simulación en el estado inicial s0.

- aux_automata/2 hace el recorrido recursivo:
  
  1. Si la lista está vacía, revisa si el estado actual es final (acepta o rechaza).
  2. Si quedan símbolos, consume el primero, usa move/3 para pasar al siguiente estado y llama recursivamente con el resto de la lista.

Este programa es un ejemplo clásico de cómo, en Prolog, se puede implementar de forma muy sencilla un proyecto de teoría de autómatas: en pocas líneas describimos la lógica de un DFA, y Prolog se encarga automáticamente de recorrer la lista de entrada y aplicar “backtracking” o “fallo” cuando no existe transición válida.

# PRUEBAS

Para ejecutar el programa hay que abrir el archivo autonama_binario.pl en prolog o copiar el codigo en otra terminal de prolog.

**Pruebas Exitosas**:

2. 1111 (15)
4. 11110 (30)
5. 1000010 (66)

![prueba_exitosa](https://github.com/user-attachments/assets/13cd8edf-e09c-4787-aeb5-dd04d21ff870)


**Pruebas No Exitosas**:

1. 100 (4)
2. 1011 (11)
3. 111011 (59) 
   
![prueba_error](https://github.com/user-attachments/assets/e6c0f4a1-12c1-45b7-a159-75e24f24d290)


