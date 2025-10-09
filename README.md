# **Aircrew Assignment: Modelado como CSP con OR-Tools**

**Ciencias de la Computación 2025-02**

**Topicos en Ciencias de la Computación**

**Docente:** Luis Martin Canaval Sanchez

**"Informe de Trabajo"**

**Grupo 2**

### Relación de integrantes:

| Miembro                              | Código     |
| :----------------------------------- | :--------- |
| Fuentes Rivera Onofre, Marco Antonio | u20211b693 |
| Huamán Saavedra, Willam Alexander    | u20211E088 |
| Vivas Alejandro, Renato Guillermo    | u202021644 |

**Octubre, 2025**

## 1. Introducción

El **Aircrew Assignment** es un problema clásico en **operaciones aeronáuticas** y **inteligencia artificial** que aparece de manera recurrente en aerolíneas, trenes y transporte marítimo con variantes como **pairing**, **rostering** y **line planning**. Su núcleo consiste en asignar personas a cada servicio (vuelos, turnos o rutas), cumpliendo con requisitos operativos como cupos, habilidades, calificaciones y restricciones de descanso o disponibilidad.

Este tipo de problemas se sitúa naturalmente en el campo de los **Problemas de Satisfacción de Restricciones (CSP)**, donde la pregunta fundamental no es "cómo maximizar algo", sino si existe una asignación que satisfaga simultáneamente un conjunto de restricciones lógicas y combinatorias.

En esta versión académica del problema se pretende asignar **20 tripulantes** (10 stewards y 10 air hostesses) a **10 vuelos**, con las siguientes restricciones:

- Cupo total fijo por vuelo.
- Mínimos por género.
- Cobertura de idiomas (francés, español, alemán).
- Restricción de que ninguna persona puede volar en vuelos consecutivos.

Para modelar y resolver este caso utilizaremos **OR-Tools** con su **solver CP-SAT**, representando cada decisión de asignación como una variable booleana y expresando las restricciones de cupo, género, idiomas y no consecutividad mediante restricciones lineales y lógicas.

## 2. Objetivos

- **OG:** Modelar y resolver el problema **Aircrew Assignment** como un **CSP** usando OR-Tools/MiniZinc, evaluando heurísticas y restricciones globales para asignar tripulaciones a vuelos, cumpliendo con cupos, roles e idiomas, y justificando las decisiones con evidencia técnica y fuentes especializadas.

### Objetivos Específicos

- **OE1:** Analizar el problema **Aircrew Assignment**, identificando variables internas y externas, relaciones causa-efecto y su impacto en la asignación de tripulaciones dentro del contexto de la computación.
- **OE2:** Modelar el problema como un **CSP**, definiendo variables, dominios y restricciones (cupos, roles, idiomas, no consecutivos) para representar formalmente el sistema.
- **OE3:** Desarrollar y validar una solución factible que cubra todos los vuelos sin violar las restricciones establecidas, demostrando eficiencia y coherencia técnica.
- **OE4:** Comparar alternativas de solución mediante restricciones globales y heurísticas de búsqueda, evaluando su rendimiento y seleccionando la más óptima.

## 3. Marco Teórico

### 3.1. Contexto del problema en operaciones aeronáuticas

La asignación de tripulantes es un componente crítico del **airline scheduling**, ya que impacta la **puntualidad**, los **costos** y el **cumplimiento normativo** (descansos, roles, cobertura de idiomas). Recientemente, se destaca la tendencia hacia modelos **integrados** que coordinan **pairing** y **rostering** con el ruteo de aeronaves y recuperación ante disrupciones, ya que tratar estos aspectos de manera separada puede degradar la calidad global del plan. Además, se enfatiza la **robustez** en los...

### 3.2. Programación por Restricciones (CP/CSP) en problemas de scheduling

La **Programación por Restricciones (CP)** permite modelar decisiones como variables (¿quién asigna a qué vuelo?), sujetas a restricciones estructuradas. CP se utiliza ampliamente en **scheduling** cuando las reglas son densas y la factibilidad estricta es crítica.

### 3.3. Global Constraints y su utilidad

Las **global constraints** agrupan patrones comunes (como conteos, secuencias, solapamientos) y los resuelven de manera eficiente utilizando algoritmos especializados.

#### Ejemplos de restricciones globales usadas:

- **R1**. Tamaño exacto de tripulación por vuelo.
- **R2**. Mínimo de azafatas por vuelo.
- **R3**. Mínimo de sobrecargos por vuelo.
- **R4**. Cobertura de idiomas por vuelo (existencia mínima).
- **R5**. Restricción de secuencia: prohibición de vuelos consecutivos para una misma persona.

### 3.4. Regla “no dos vuelos siguientes” mediante secuencias

La restricción de **no asignar una persona a dos vuelos consecutivos** se modela mediante restricciones de secuencia (**sliding windows**), que controlan cuántas veces ocurre un evento dentro de una ventana deslizante de longitud fija.

### 3.5. Herramientas actuales (2022–2025) y soporte de modelado

**OR-Tools CP-SAT** (v9.11–9.12, 2024–2025) es una herramienta que incluye tipos de variables de intervalo y restricciones para programación de tareas, además de contar con estrategias de búsqueda configurables que mejoran el rendimiento en modelos con múltiples restricciones.

### 3.6. Metodologías de evaluación y robustez operativa

La evaluación de la solución se basa en **tiempo de resolución**, **número de nodos explorados** y **porcentaje de factibilidad**. En el ámbito aeronáutico, también se introducen conceptos como la **reserva de tripulación** y **robustez** ante ausencias, para mejorar la flexibilidad de las asignaciones.

## 4. Desarrollo

### 4.1. Datos del Problema

- **20 empleados**: 10 sobrecargos (stewards) y 10 azafatas (air hostesses).
- **10 vuelos** con requisitos específicos de tripulación.
- **Restricciones**: idiomas, género y disponibilidad.

### 4.2. Variables de Decisión

El modelo utiliza **variables booleanas** binarias para asignar empleados a vuelos:

- **x[e][f]** = 1 si el empleado **e** está asignado al vuelo **f**.
- **x[e][f]** = 0 en caso contrario.

### 4.3. Restricciones del Modelo

#### Restricción de número exacto de tripulantes por vuelo:

Para cada vuelo **f**, el número total de empleados asignados debe ser exacto:

```
∑(e=1 to 20) x[e][f] = aircrew_required[f]
```

#### Restricción de mínimo de azafatas por vuelo:

Para cada vuelo **f**, debe haber al menos un número mínimo de azafatas:

```
∑(e ∈ air_hostesses) x[e][f] ≥ min_hostesses[f]
```

#### Restricción de número mínimo de sobrecargos por vuelo:

```
∑(e ∈ stewards) x[e][f] ≥ min_stewards[f]
```

#### Restricciones de Idiomas:

Cada vuelo debe tener al menos un empleado que hable cada uno de los tres idiomas requeridos (Francés, Español, Alemán):

```
∑(e ∈ french_speakers) x[e][f] ≥ 1 ∀f
∑(e ∈ spanish_speakers) x[e][f] ≥ 1 ∀f
∑(e ∈ german_speakers) x[e][f] ≥ 1 ∀f
```

#### Restricción de no vuelos consecutivos:

Esta restricción asegura que un empleado no puede ser asignado a dos vuelos consecutivos:

```
x[e][f] + x[e][f+1] ≤ 1 ∀e, ∀f ∈ {1, 2, ..., 9}
```

### 4.4. Restricciones Globales Utilizadas

Las **restricciones globales** en **OR-Tools CP-SAT** permiten mejorar la eficiencia del solver al reducir el espacio de búsqueda mediante técnicas avanzadas de propagación.

### 4.5. Explicación Detallada del Código

El código sigue una estructura modular, que incluye la importación de bibliotecas, la definición de los datos del problema, la creación del modelo y las variables, y la adición de restricciones. Posteriormente, se resuelve el modelo y se procesan los resultados.

### 4.6. Ventajas del Uso de OR-Tools CP-SAT

El uso de OR-Tools CP-SAT permite aprovechar técnicas de **propagación de restricciones avanzadas**, **búsqueda inteligente** y un **manejo eficiente de restricciones lineales**, lo que hace que la solución sea más eficiente.

## 5. Justificación del Uso de la IA

Durante el desarrollo de este proyecto, se utilizó asistencia de **Claude Sonnet 4.5** para optimizar el proceso de implementación y resolver dudas técnicas sobre sintaxis y restricciones. Esta herramienta fue utilizada para mejorar la eficiencia en el desarrollo y asegurar la corrección del código.

## 6. Bibliografía

- Google Developers. (2024). [CP-SAT Solver | OR-Tools](https://developers.google.com/optimization/cp/cp_solver)
- Google Developers. (2024). [Python Reference: CP-SAT | OR-Tools](https://developers.google.com/optimization/reference/python/sat/python/cp_model)
- MiniZinc Team. (2024). [Mathematical constraints — sliding_sum (v2.9.x)](https://docs.minizinc.dev/en/latest/lib-globals-math.html)
- Xu, Y. (2024). **Airline scheduling optimization: Literature review and modelling methodologies**. Intelligent Transportation Infrastructure, 3(1), liad026. [https://doi.org/10.1093/iti/liad026](https://doi.org/10.1093/iti/liad026)
- Gao, J., Zhu, X., & Zhang, R. (2023). Optimization of parallel test task scheduling with constraint satisfaction. The Journal of Supercomputing, 79, 7206–7227. [https://doi.org/10.1007/s11227-022-04943-0](https://doi.org/10.1007/s11227-022-04943-0)
