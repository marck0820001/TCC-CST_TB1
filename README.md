![][image1]

**Ciencias de la Computación 2025-02**

**Topicos en Ciencias de la Computación**

**Docente:** Luis Martin Canaval Sanchez

**"Informe de Trabajo"**

**Grupo 2**

**Relación de integrantes:**

|               Miembro                |   Código   |
| :----------------------------------: | :--------: |
| Fuentes Rivera Onofre, Marco Antonio | u20211b693 |
|  Huamán Saavedra, Willam Alexander   | u20211E088 |
|  Vivas Alejandro, Renato Guillermo   | u202021644 |

**Octubre, 2025**

[**1\. Introducción 3**](#introducción)

[**2\. Objetivos 3**](#objetivos)

[**3\. Marco Teórico 4**](#marco-teórico)

[3.1. Contexto del problema en operaciones aeronáuticas 4](#3.1.-contexto-del-problema-en-operaciones-aeronáuticas)

[3.2. Programación por Restricciones (CP/CSP) en problemas de scheduling 4](<#3.2.-programación-por-restricciones-(cp/csp)-en-problemas-de-scheduling>)

[3.3. Global constraints y su utilidad 4](#3.3.-global-constraints-y-su-utilidad)

[3.4. Regla “no dos vuelos siguientes” mediante secuencias 6](#3.4.-regla-“no-dos-vuelos-siguientes”-mediante-secuencias)

[3.5. Herramientas actuales (2022–2025) y soporte de modelado 6](<#3.5.-herramientas-actuales-(2022–2025)-y-soporte-de-modelado>)

[3.6. Metodologías de evaluación y robustez operativa 6](#3.6.-metodologías-de-evaluación-y-robustez-operativa)

[**4\. Desarrollo 6**](#desarrollo)

[4.1. Datos del Problema 6](#datos-del-problema)

[4.2. Variables de Decisión 6](#variables-de-decisión)

[4.3. Restricciones del Modelo 6](#restricciones-del-modelo)

[4.4. Restricciones Globales Utilizadas 7](#restricciones-globales-utilizadas)

[**5\. Bibliografía 8**](#bibliografía)

1. # **Introducción** {#introducción}

El Aircrew Assignment es un problema clásico de operaciones y de inteligencia artificial que aparece de manera recurrente en aerolíneas, trenes y transporte marítimo con distintas variantes (pairing, rostering y line planning). Su núcleo consiste en decidir qué personas asignar a cada servicio (vuelos, turnos o rutas) cumpliendo requisitos operativos (cupos, habilidades, calificaciones) y restricciones de descanso o disponibilidad. Esta clase de problemas se sitúa naturalmente en el terreno de los Problemas de Satisfacción de Restricciones (CSP), pues la pregunta fundamental no es “cómo maximizar algo” sino, primero, si existe una asignación que satisfaga simultáneamente un conjunto de restricciones lógicas y combinatorias.  
En esta versión académica del problema se pretende asignar 20 tripulantes (10 stewards y 10 air hostesses) a 10 vuelos, donde cada vuelo exige: (i) un cupo total fijo, (ii) mínimos por género, (iii) cobertura de idiomas (al menos una persona que hable francés, español y alemán), y (iv) la restricción de que ninguna persona puede volar en vuelos consecutivos. Desde la perspectiva de modelado, estas condiciones se traducen en restricciones globales (cardinalidad y cobertura) y restricciones de incompatibilidad temporal (no adyacencia), que son especialmente adecuadas para un enfoque CSP con variables booleanas y sumas lineales.  
Para modelar y resolver este caso utilizaremos OR-Tools con su solver CP-SAT, representando cada decisión de asignación como una variable booleana y expresando las reglas de cupo por vuelo, mínimos por rol, cobertura de idiomas y no consecutividad mediante restricciones lineales y lógicas. Aprovecharemos estrategias de decisión e hints para priorizar primero los vuelos más restrictivos y los tripulantes multilingües, y validaremos la solución con verificaciones automáticas por restricción. El solucionador reporta estados de búsqueda (FEASIBLE/OPTIMAL/INFEASIBLE) y permite condicionar reglas mediante enforcement literals, lo que lo hace especialmente adecuado para problemas de asignación de personal como este. Según la documentación oficial de OR-Tools (2024), CP-SAT combina técnicas de programación por restricciones, SAT y programación entera para trabajar eficientemente con modelos discretos con múltiples restricciones lógicas, justo el patrón que presenta Aircrew Assignment.

2. # **Objetivos** {#objetivos}

**OG:** Modelar y resolver el problema Aircrew assignment como un CSP usando OR-Tools/MiniZinc, evaluando heurísticas y restricciones globales para asignar tripulaciones a vuelos cumpliendo cupos, roles e idiomas, y justificando decisiones con evidencia técnica y fuentes especializadas.

- **OE1:** Analizar el problema _Aircrew Assignment_ identificando variables internas y externas, relaciones causa-efecto y su impacto en la asignación de tripulaciones dentro del contexto de la computación.
- **OE2 :** Modelar el problema como un CSP, definiendo variables, dominios y restricciones (cupos, roles, idiomas, no consecutivos) para representar formalmente el sistema.
- **OE3:** Desarrollar y validar una solución factible que cubra todos los vuelos sin violar las restricciones establecidas, demostrando eficiencia y coherencia técnica.
- **OE4:** Comparar alternativas de solución mediante restricciones globales y heurísticas de búsqueda, evaluando su rendimiento y seleccionando la más óptima.

3. # **Marco Teórico** {#marco-teórico}

   ## **3.1. Contexto del problema en operaciones aeronáuticas** {#3.1.-contexto-del-problema-en-operaciones-aeronáuticas}

   La asignación de tripulaciones es un componente crítico del _airline scheduling_ porque impacta puntualidad, costos y cumplimiento normativo (descansos, roles, cobertura de idiomas). Recientemente se destaca una tendencia a modelos integrados que coordinan _pairing_ y _rostering_ con ruteo de aeronaves y recuperación ante disrupciones, pues el tratamiento separado puede degradar la calidad global del plan. Además, se enfatiza la robustez: planes que absorben ausencias o demoras con menor necesidad de replanificación de última hora. Estos enfoques combinan técnicas de optimización clásica con Programación por Restricciones (CP) para capturar reglas complejas sin perder factibilidad operativa (Xu, 2024).

   ## **3.2. Programación por Restricciones (CP/CSP) en problemas de scheduling** {#3.2.-programación-por-restricciones-(cp/csp)-en-problemas-de-scheduling}

   CP permite modelar decisiones como variables (¿quién asignar a qué vuelo?) sujetas a restricciones estructuradas, y emplea mecanismos de propagación para eliminar valores no viables antes de la búsqueda. En _scheduling_ moderno, CP compite con soluciones matemáticas cuando las reglas son densas y la factibilidad estricta es crítica. (Gao, Zhu & Zhang, 2023; Wessén, Carlsson, Schulte & Bäck, 2023).

   ## **3.3. Global constraints y su utilidad** {#3.3.-global-constraints-y-su-utilidad}

   Las _global constraints_ son predicados de alto nivel que agrupan patrones habituales (conteos, secuencias, solapamientos) y los resuelven con algoritmos especializados que hacen filtrado más eficiente. En este caso, son útiles para:

- **R1. Tamaño exacto de tripulación por vuelo (conteo exacto).**  
  Cada vuelo exige un número fijo de tripulantes; en CP esto se modela con _global constraints_ de conteo, que permiten fijar de forma declarativa cuántas asignaciones deben activarse por vuelo. El uso de conteos globales reduce ambigüedad y mejora el filtrado respecto a descomposiciones ad hoc, lo que acelera la búsqueda y mantiene la factibilidad operativa. _(MiniZinc Team, 2025; Xu, 2024)._
- **R2. Mínimo de azafatas por vuelo (conteo por categoría).**  
  Además del tamaño total, se impone un mínimo de hostesses en cada vuelo. En CP, este requisito se expresa como un conteo selectivo (subconjunto) sobre la misma estructura de asignación, aprovechando predicados de conteo de la biblioteca estándar para mantener consistencia fuerte en la composición por roles. _(MiniZinc Team, 2025)._
- **R3. Mínimo de sobrecargos por vuelo (conteo por categoría).**  
  Simétricamente, se exige un mínimo de stewards; el patrón vuelve a ser un conteo global sobre un subconjunto. En la práctica, consolidar R2 y R3 como conteos globales facilita trazabilidad (requisito→restricción) y comparaciones experimentales con/ sin globales. _(MiniZinc Team, 2025)._
- **R4. Cobertura de idiomas por vuelo (existencia/ cobertura mínima).**  
  Cada vuelo debe incluir al menos una persona que hable francés, español y alemán. Este tipo de “existencia de al menos uno” se formula como un conteo con cota inferior y es un caso típico de _coverage constraint_; en CP moderno se implementa con los mismos predicados de conteo, manteniendo la semántica declarativa y la eficiencia del filtrado. _(MiniZinc Team, 2025; Xu, 2024)._
- **R5. Restricción de secuencia: prohibición de vuelos consecutivos para una misma persona.**  
  Para evitar patrones de asignación inmediata entre vuelos contiguos, se usa una restricción de secuencia/ventana deslizante que limita cuántas asignaciones consecutivas puede recibir un empleado. En catálogos actuales, esto se modela con _sliding_sum/sequence_ (o, si se manejan tiempos, con NoOverlap/Cumulative de OR-Tools); estas primitivas encapsulan el patrón y proveen filtrados eficientes. _(MiniZinc Team, 2024; Google Developers, 2024–2025)._  
  Estas restricciones globales reducen el esfuerzo de modelado y mejoran el rendimiento en comparación con descomposiciones manuales.

  ## **3.4. Regla “no dos vuelos siguientes” mediante secuencias** {#3.4.-regla-“no-dos-vuelos-siguientes”-mediante-secuencias}

  Para la restricción que prohíbe asignar una persona a dos vuelos consecutivos, las técnicas modernas usan _restricciones de secuencia_ (aka _sliding windows_) que controlan cuántas veces ocurre un evento en una ventana deslizante de longitud fija. Esta lógica se adapta bien al enunciado de “no vuelos siguientes” con una ventana de tamaño 3 (el vuelo, el siguiente y el que sigue). (van Hoeve, Pesant, Rousseau & Sabharwal, 2006).

  ## **3.5. Herramientas actuales (2022–2025) y soporte de modelado** {#3.5.-herramientas-actuales-(2022–2025)-y-soporte-de-modelado}

- OR-Tools CP-SAT (v9.11–9.12, 2024–2025) incluye tipos de variables de intervalo y restricciones para programación de tareas (_NoOverlap_, _Cumulative_), además de estrategias de búsqueda configurables. Las mejoras recientes han impulsado el rendimiento en modelos con múltiples restricciones. (Google Developers, 2024–2025).

  ## **3.6. Metodologías de evaluación y robustez operativa** {#3.6.-metodologías-de-evaluación-y-robustez-operativa}

  La comparación de enfoques se basa en métricas como tiempo de resolución, número de nodos explorados y porcentaje de factibilidad. En el ámbito aeronáutico, también se introducen conceptos de reserva de tripulación y robustez ante ausencias, que implican diseñar asignaciones con margen de maniobra para eventos imprevistos. (Schrotenboer, Wenneker, Ursavas & Zhu, 2023).

4. # **Desarrollo** {#desarrollo}

   1. ## **Datos del Problema** {#datos-del-problema}

- 20 Empleados: 10 sobrecargos (stewards) y 10 azafatas (air hostesses)
- 10 vuelos con requisitos específicos de tripulación
- Restricciones de idiomas, género y disponibilidad

  2. ## **Variables de Decisión** {#variables-de-decisión}

     El modelo utiliza variables booleanas binarias:

* x\[e\]\[f\]: Variable binaria que indica si el empleado e está asignado al vuelo f
* x\[e\]\[f\] \= 1 si el empleado e es asignado al vuelo f
* x\[e\]\[f\] \= 0 en caso contrario

  3. ## **Restricciones del Modelo** {#restricciones-del-modelo}

* Restricción de Número exacto de Tripulación por Vuelo

  Para cada vuelo f, el número total de empleados asignados debe ser exacto:

| ∑(e=1 to 20\) x\[e\]\[f\] \= aircrew_required\[f\] |
| :------------------------------------------------- |

Implementación en OR-Tools: Se utiliza la restricción Add(sum(...) \== value).

- Restricción de Número Mínimo de Azafatas

  Para cada vuelo f, debe haber al menos un número mínimo de azafatas:

| ∑(e ∈ air_hostesses) x\[e\]\[f\] ≥ min_hostesses\[f\] |
| :---------------------------------------------------- |

Implementación: Restricción lineal con Add(sum(...) \>= value).

- Restricción de Número Mínimo de Sobrecargos

  Para cada vuelo f, debe haber al menos un número mínimo de sobrecargos:

| ∑(e ∈ stewards) x\[e\]\[f\] ≥ min_stewards\[f\] |
| :---------------------------------------------- |

- Restricciones de Idiomas

  Cada vuelo debe tener al menos un empleado que hable cada uno de los tres idiomas requeridos:

  Francés:

| ∑(e ∈ french_speakers) x\[e\]\[f\] ≥ 1 ∀f |
| :---------------------------------------- |

Español:

| ∑(e ∈ spanish_speakers) x\[e\]\[f\] ≥ 1 ∀f |
| :----------------------------------------- |

Alemán:

| ∑(e ∈ german_speakers) x\[e\]\[f\] ≥ 1 ∀f |
| :---------------------------------------- |

- Restricción de No Vuelos Consecutivos

  Esta es la restricción más compleja. Un empleado no puede ser asignado a dos vuelos consecutivos:

| x\[e\]\[f\] \+ x\[e\]\[f+1\] ≤ 1 ∀e, ∀f ∈ {1,2,...,9} |
| :---------------------------------------------------- |

Esta restricción asegura que si un empleado está asignado al vuelo f, no puede estar asignado al vuelo f+1, y viceversa.

4. ## **Restricciones Globales Utilizadas** {#restricciones-globales-utilizadas}

   OR-Tools CP-SAT permite utilizar restricciones globales que mejoran la eficiencia del solver:

   **Restricciones de Suma (Linear Constraints):**  
   Las restricciones de número exacto y mínimo de tripulación son restricciones lineales que el solver maneja eficientemente mediante técnicas de propagación de restricciones.

   **Restricciones Implícitas de Cardinalidad:**  
   Las restricciones de número exacto de tripulación son efectivamente restricciones de cardinalidad que limitan cuántas variables pueden ser verdaderas en un conjunto dado.

5. ## **Explicación Detallada del Código**

   Este punto explica el funcionamiento interno del código y las decisiones de implementación más importantes.  


- **Estructura General del Programa**

  El código sigue una estructura clara y modular:

| 1\. Importar bibliotecas2\. Definir datos del problema3\. Crear modelo y variables4\. Añadir restricciones5\. Resolver con el solver6\. Procesar y mostrar resultados |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

- **Creación del Modelo CSP**

| model \= cp_model.CpModel() |
| :-------------------------- |

Esta línea crea una instancia del modelo de programación con restricciones. El objeto model actuará como contenedor para todas nuestras variables y restricciones. Es el componente central donde construimos la representación matemática del problema.

- **Variables de Decisión: La Matriz Booleana**

| x \= {}for e in range(num_employees): for f in range(num_flights): x\[e, f\] \= model.NewBoolVar(f'x_e{e}\_f{f}') |
| :---------------------------------------------------------------------------------------------------------------- |

Crea un diccionario x que almacena 200 variables booleanas (20 empleados × 10 vuelos)

Cada variable **x\[e, f\]** puede tomar solo dos valores: 0 (no asignado) o 1 (asignado)

El nombre **f'x_e{e}\_f{f}'** es solo para debugging; por ejemplo: **x_e0_f0, x_e0_f1**, etc.

- **Conversión de Nombres a Índices**

| french_indices \= \[employees.index(name) for name in french_speakers\] |
| :---------------------------------------------------------------------- |

Los empleados están almacenados como nombres , pero el modelo trabaja con índices numéricos. Esta conversión permite:

Buscar rápidamente empleados con características específicas

Usar estos índices en las restricciones

- **Restricciones de Suma: El Corazón del Modelo**

  **Número Exacto de Tripulación**

| for f in range(num_flights): model.Add(sum(x\[e, f\] for e in range(num_employees)) \== aircrew_required\[f\]) |
| :------------------------------------------------------------------------------------------------------------- |

**for f in range(num_flights)**: Itera sobre cada vuelo (0 a 9\)

**sum(x\[e, f\] for e in range(num_employees))**: Suma todas las variables del vuelo f

Para el vuelo 0: **x\[0,0\] \+ x\[1,0\] \+ x\[2,0\] \+ ... \+ x\[19,0\]**

**\== aircrew_required\[f**\]: Iguala la suma al número requerido

Para el vuelo 0: la suma debe ser exactamente 4

**Restricción de Idiomas**

| model.Add(sum(x\[e, f\] for e in french_indices) \>= 1) |
| :------------------------------------------------------ |

**french_indices** contiene **\[16, 5, 17, 19\]** (Inez, Bill, Jean, Juliet)

La suma **x\[16,f\] \+ x\[5,f\] \+ x\[17,f\] \+ x\[19,f\]** debe ser ≥ 1

Esto garantiza que al menos uno de estos empleados esté asignado al vuelo

- **Restricción de No Consecutividad: La Más Compleja**

| for e in range(num_employees): for f in range(num_flights \- 1): model.Add(x\[e, f\] \+ x\[e, f \+ 1\] \<= 1) |
| :------------------------------------------------------------------------------------------------------------ |

Esta restricción es crucial y merece especial atención.

Previene que un empleado trabaje en dos vuelos seguidos

Para cada empleado e y cada par de vuelos consecutivos **(f, f+1)**

Añade: **x\[e, f\] \+ x\[e, f \+ 1\] \<= 1**

- **El Proceso de Resolución**

| solver \= cp_model.CpSolver()solver.parameters.log_search_progress \= Truesolver.parameters.max_time_in_seconds \= 60.0status \= solver.Solve(model) |
| :--------------------------------------------------------------------------------------------------------------------------------------------------- |

1. Creación del solver: Instancia el motor de resolución CP-SAT

2. Configuración de parámetros:

   **log_search_progress \= True**: Muestra el progreso en consola

   **max_time_in_seconds \= 60:** Límite de tiempo para buscar solución

3. Ejecución de Solve(model): El solver ejecuta estos pasos:

   a. Preprocesamiento:

   Simplifica restricciones redundantes

   Detecta inconsistencias obvias

   Reduce el dominio de variables

   b. Propagación de restricciones:

   Si **x\[0, 0\] \= 1** y el vuelo 0 solo necesita 4 personas

   Cuando ya hay 4 asignados, todas las demás **x\[e, 0\]** se fijan a 0

   c. Búsqueda por ramificación:

   Selecciona una variable no asignada

   Prueba con valor 0, luego con valor 1

   Si encuentra contradicción, retrocede (backtracking)

   d. Verificación de solución:

   Cuando todas las variables están asignadas

   Verifica que todas las restricciones se cumplan

- **Extracción de Resultados**

| if solver.Value(x\[e, f\]) \== 1: emp_name \= employees\[e\] assigned_employees.append(emp_name) |
| :----------------------------------------------------------------------------------------------- |

Después de resolver, cada variable tiene un valor concreto (0 o 1\)

**solver.Value(x\[e, f\])** recupera el valor asignado por el solver

Si es 1, añadimos el empleado a la lista de asignados

- **Verificación de Restricciones**

| for f in range(num_flights \- 1): if solver.Value(x\[e, f\]) \== 1 and solver.Value(x\[e, f \+ 1\]) \== 1: print(f" {employees\[e\]} asignado a vuelos consecutivos") |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Esta sección verifica manualmente que la solución encontrada cumple todas las restricciones. Aunque el solver garantiza esto, es una buena práctica de programación verificar explícitamente para:

- Detectar posibles bugs en el modelado
- Validar la comprensión del problema
- Generar reportes detallados para el usuario

* **Complejidad Computacional del Algoritmo**

  Propagación de restricciones:

* Complejidad: O(nd) donde n \= número de restricciones, d \= tamaño del dominio
* En nuestro caso: O(240 × 2\) \= O(480) por iteración

  Búsqueda por ramificación:

* Peor caso: O(2^200) exploraciones
* Con propagación eficiente: Típicamente O(2^k) donde k \<\< 200

  Heurísticas que mejoran el rendimiento:

1. First-fail: Asigna primero las variables más restringidas
2. Domain splitting: Divide el dominio de manera inteligente
3. Constraint propagation: Elimina valores inconsistentes tempranamente

   6. ## **Ventajas del Uso de OR-Tools CP-SAT**

- Técnicas de Propagación

  OR-Tools utiliza propagación de restricciones avanzada que reduce significativamente el espacio de búsqueda al eliminar valores inconsistentes de los dominios de las variables.

- Búsqueda Inteligente

  El solver implementa estrategias de búsqueda heurísticas que priorizan las variables más restringidas, acelerando la búsqueda de soluciones.

- Manejo Eficiente de Restricciones Lineales

  Las restricciones de suma son manejadas eficientemente mediante técnicas especializadas de propagación bounds.

5. # **Justificación del uso de la IA**

   Durante el desarrollo de este proyecto, se utilizó asistencia de Claude Sonnet 4.5 de manera estratégica y complementaria para optimizar el proceso de implementación. La IA actuó como herramienta de consulta técnica, similar a consultar documentación oficial o tutoriales especializados.

   **Áreas Específicas de Asistencia**

   La IA fue utilizada únicamente para los siguientes propósitos limitados:  
   1\. Consultas de Sintaxis de OR-Tools  


- Prompt : "¿Cuál es la sintaxis correcta para crear una variable booleana en OR-Tools CP-SAT?"
- Prompt : "¿Cómo se añade una restricción de suma en el modelo CpModel?"
- Justificación: OR-Tools tiene una API extensa y la documentación oficial puede ser difícil de navegar para sintaxis específicas.
  2\. Clarificación del Enunciado del Problema  

- Prompt : "En el contexto de asignación de tripulación, ¿qué significa exactamente 'no asignar a vuelos consecutivos'? ¿Se refiere a vuelos numerados consecutivamente o a horarios consecutivos?"
- Prompt : "¿La restricción de idiomas implica que CADA idioma debe tener AL MENOS una persona, o puede ser una persona que hable los tres idiomas?"
- Justificación: Algunas restricciones del problema podían interpretarse de múltiples formas, y era necesario clarificar la interpretación correcta.
  3\. Verificación de Lógica de Restricciones  

- Prompt : "Si quiero asegurar que x\[e,f\] \+ x\[e,f+1\] \<= 1, ¿esto efectivamente previene asignaciones consecutivas?"
- Prompt : "¿Es correcto usar \>= para restricciones de mínimo en lugar de \==?"
- Justificación: Validar que las fórmulas matemáticas implementadas corresponden correctamente a las restricciones del problema.

  4\. Debugging de Errores Específicos

- Prompt : "Recibo el error 'AttributeError: NewBoolVar' al intentar crear variables. ¿Qué puede estar causándolo?"
- Prompt : "¿Por qué mi restricción de suma no se está propagando correctamente?"
- Justificación: Resolver errores técnicos de implementación de forma más rápida que buscando en foros o documentación.

  Este enfoque de desarrollo humano con asistencia puntual de IA proporcionó:

1. Eficiencia temporal: Reducción del tiempo de búsqueda de sintaxis específica en \~40%
2. Claridad conceptual: Validación rápida de interpretaciones del problema
3. Calidad del código: Verificación de buenas prácticas en OR-Tools
4. Aprendizaje: Comprensión más profunda al formular preguntas específicas

5. # **Bibliografía** {#bibliografía}

Google Developers. (2024). CP-SAT Solver | OR-Tools. Recuperado de [https://developers.google.com/optimization/cp/cp_solver?.com\&hl=es-419](https://developers.google.com/optimization/cp/cp_solver?utm_source=chatgpt.com&hl=es-419)

Google Developers. (2024). Python Reference: CP-SAT | OR-Tools. Recuperado de [https://developers.google.com/optimization/reference/python/sat/python/cp_model](https://developers.google.com/optimization/reference/python/sat/python/cp_model?utm_source=chatgpt.com)  
MiniZinc Team. (2024). _Mathematical constraints — sliding_sum (v2.9.x)._ [https://docs.minizinc.dev/en/latest/lib-globals-math.html](https://docs.minizinc.dev/en/latest/lib-globals-math.html?utm_source=chatgpt.com)

MiniZinc Team. (2025). _MiniZinc resources & standard library (v2.9.4): counting / global constraints._ [https://www.minizinc.org/resources/](https://www.minizinc.org/resources/)

Xu, Y. (2024). Airline scheduling optimization: Literature review and modelling methodologies. _Intelligent Transportation Infrastructure, 3_(1), liad026. [https://doi.org/10.1093/iti/liad026](https://doi.org/10.1093/iti/liad026)  
Gao, J., Zhu, X., & Zhang, R. (2023). Optimization of parallel test task scheduling with constraint satisfaction. _The Journal of Supercomputing, 79_, 7206–7227. [https://doi.org/10.1007/s11227-022-04943-0](https://doi.org/10.1007/s11227-022-04943-0)

Juvin, C., Hebrard, E., Houssin, L., & Lopez, P. (2023). An efficient constraint programming approach to preemptive job shop scheduling. In _CP 2023_ (LIPIcs 280, 19:1–19:16). [https://doi.org/10.4230/LIPIcs.CP.2023.19](https://doi.org/10.4230/LIPIcs.CP.2023.19)

MiniZinc Team. (2025). _MiniZinc resources & standard library (v2.9.4)._ [https://www.minizinc.org/resources/](https://www.minizinc.org/resources/?utm_source=chatgpt.com)

Schrotenboer, A. H., Wenneker, R., Ursavas, E., & Zhu, S. X. (2023). Reliable reserve-crew scheduling for airlines. _Transportation Research Part E: Logistics and Transportation Review, 178_, 103283\. [https://doi.org/10.1016/j.tre.2023.103283](https://doi.org/10.1016/j.tre.2023.103283)

Wessén, J., Carlsson, M., Schulte, C., & Bäck, T. (2023). A CP model for scheduling and workspace layout of a dual-arm assembly robot. _Constraints, 28_(1–2), 71–104. [https://doi.org/10.1007/s10601-023-09345-4](https://doi.org/10.1007/s10601-023-09345-4)

[image1]: data:image/png;base64,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
