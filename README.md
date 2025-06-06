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


# Análisis de complejidad

**Complejidad temporal del enfoque lógico (Prolog, AFD)**

- Cada símbolo (bit) en la cadena se consume una sola vez y se hace una única llamada a `move/3`.

- Suponiendo que la lista de bits tiene longitud n, haremos n llamadas recursivas a `aux_automata/2`.

- Cada llamada a `move/3` es una búsqueda en los hechos definidos. Como `move/3` está “indexado” por el primer argumento (estado actual) y luego el tercer argumento (símbolo), Prolog lo resuelve en tiempo aproximado O(1) o O(k) donde k es constante (número de hechos por estado).

- Por tanto, en total, la complejidad es **O(n)** en el número de símbolos de la cadena.


**Complejidad espacial**

- El único “overhead” es la recursión en la pila de Prolog de profundidad máxima n, más el espacio necesario para almacenar la lista de bits.
  
- Espacio auxiliar de la pila: O(n).


# Alternativa con otro paradigma: Imperativo (Python)

Para comparar, veamos cómo se resolvería el mismo problema “¿un binario es divisible por 3?” con un enfoque imperativo en Python, usando una función que recorre la cadena y mantiene el residuo:

```
def es_divisible_por_3(binario: str) -> bool:
    """
    Recibe una cadena de caracteres '0' y '1'.
    Devuelve True si el número binario es divisible por 3, False en otro caso.
    """

    residuo = 0
    for caracter in binario:
        # Convertimos el caracter a entero (0 o 1)
        bit = int(caracter)
        # Nuevo residuo = (2 * residuo + bit) mod 3
        residuo = (2 * residuo + bit) % 3

    # Si al final residuo es 0, es divisible
    return (residuo == 0)


if __name__ == "__main__":
    pruebas = ["0", "11", "10", "110", "1001", "100", "111011", "10110"]
    for b in pruebas:
        print(f"{b} →", "Divisible" if es_divisible_por_3(b) else "NO divisible")
```

- Recorremos cada caracter de la cadena una sola vez: n iteraciones.

- Cada iteración realiza una operación aritmética y una asignación: O(1).

- En total: O(n).

No hay recursión ni búsquedas extra; tan solo se guarda un entero residuo que evoluciona. Espacialmente, solo se reserva O(1) para la variable residuo además del espacio para la cadena de entrada.

| **Caracteristica**                     | **Prolog (AFD Declarativo)**                                              | **Python (Imperativo)**                                                    |
| -------------------------------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Expresión de la lógica**             | Muy concisa: solo hechos `move/3`, `final/1`                              | Algo más verbosa: bucle + cálculos                                         |
| **Nivel de abstracción**               | Alto: “digo qué” es el autómata, no “cómo”                                | Medio: se define explícitamente el cálculo del residuo                     |
| **Complejidad temporal**               | O(n)                                                                      | O(n)                                                                       |
| **Complejidad espacial**               | O(n) por recursión en pila (lista de bits)                                | O(1) operativo + O(n) para la cadena                                       |
| **Escalabilidad (cadenas muy largas)** | Buena, salvo profundidad de recursión en Prolog (n estados en la pila)    | Excelente, sin límite de recursión (iteración)                             |
| **Detección de errores de alfabeto**   | Si aparece un símbolo distinto de ‘0’ o ‘1’, Prolog falla automáticamente | En Python hay que validar explícitamente el input (o capturar excepciones) |


# Conclusión

Después de esta ultima entrega, llego a la conclusión de que cuando se trata de prototipar rápidamente un reconocimiento formal mediante un DFA, Prolog resulta realmente práctico. En cuestión de minutos puedo definir las transiciones y los estados finales con apenas unas líneas de código, sin preocuparme por bucles ni manejar variables de estado. Esto me permite centrarme en la lógica del autómata en lugar de en la sintaxis del lenguaje, lo que agiliza mucho el desarrollo de pruebas conceptuales y ejemplos sencillos.

Sin embargo, si se piensa en llevar ese mismo reconocimiento a un entorno de producción, especialmente cuando las cadenas de entrada pueden ser muy largas, un enfoque imperativo como el de Python me parece más efectivo. Con Python se puede procesar cada bit en un bucle iterativo y mantener un único valor de “residuo” sin necesidad de usar recursión. De esa forma de trabajar se evita el consumo de pila que existiría en Prolog con llamadas recursivas para cada símbolo, y además se integra de forma natural con otras bibliotecas o módulos que muy probablemente pueda usar en un proyecto real.

En ambos casos el tiempo de ejecución crece linealmente con la longitud de la cadena (O(n)), pero la diferencia clave está en el espacio: la implementación en Python solo utiliza O(1) de memoria adicional para el residuo, mientras que Prolog reserva O(n) en la pila para cada llamada recursiva. Por eso, para exploraciones rápidas y ejemplos teóricos, Prolog es ideal; en cambio, para manejar volúmenes grandes de datos o incluir la función dentro de una aplicación más compleja, Python (u otro lenguaje imperativo) es, en mi experiencia, la opción más eficiente y fácil de integrar.

# REFERENCIAS

- Hopcroft, J. E., Motwani, R., & Ullman, J. D. (2006). Introduction to Automata Theory, Languages, and Computation (3ª ed.). Pearson Education.
- Sipser, M. (2013). Introduction to the Theory of Computation (3ª ed.). Cengage Learning.
- Sterling, L., & Shapiro, E. (1994). The Art of Prolog: Advanced Programming Techniques (2ª ed.). MIT Press.
- SWI-Prolog Development Team. (2021). SWI-Prolog Reference Manual. Retrieved from https://www.swi-prolog.org/pldoc/man?section=manual
- Lutz, M. (2013). Learning Python (5ª ed.). O’Reilly Media.
- Ullman, J. D., & Aho, A. V. (2003). Foundations of Computer Science. Pearson.

