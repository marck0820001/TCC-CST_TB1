# Ciencias de la Computación 2025-02

## Tópicos en Ciencias de la Computación

**Docente:** Luis Martin Canaval Sánchez  
**“Informe de Trabajo”**  
**Grupo 2**

### Relación de integrantes

| Miembro                              | Código     |
| ------------------------------------ | ---------- |
| Fuentes Rivera Onofre, Marco Antonio | u20211b693 |
| Huamán Saavedra, Willam Alexander    | u20211E088 |
| Vivas Alejandro, Renato Guillermo    | u202021644 |

**Octubre, 2025**

---

## 1. Introducción

El **Aircrew Assignment** es un problema clásico de operaciones y de inteligencia artificial que aparece de manera recurrente en aerolíneas, trenes y transporte marítimo con distintas variantes (_pairing_, _rostering_ y _line planning_). Su núcleo consiste en decidir qué personas asignar a cada servicio (vuelos, turnos o rutas) cumpliendo requisitos operativos (cupos, habilidades, calificaciones) y restricciones de descanso o disponibilidad. Esta clase de problemas se sitúa naturalmente en el terreno de los **Problemas de Satisfacción de Restricciones (CSP)**, pues la pregunta fundamental no es “cómo maximizar algo” sino, primero, si existe una asignación que satisfaga simultáneamente un conjunto de restricciones lógicas y combinatorias.

En esta versión académica del problema se pretende asignar **20 tripulantes** (10 _stewards_ y 10 _air hostesses_) a **10 vuelos**, donde cada vuelo exige: (i) un **cupo total** fijo, (ii) **mínimos por género**, (iii) **cobertura de idiomas** (al menos una persona que hable francés, español y alemán), y (iv) la restricción de que **ninguna persona puede volar en vuelos consecutivos**. Desde la perspectiva de modelado, estas condiciones se traducen en **restricciones globales** (cardinalidad y cobertura) y **restricciones de incompatibilidad temporal** (no adyacencia), que son especialmente adecuadas para un enfoque CSP con variables booleanas y sumas lineales.

Para modelar y resolver este caso utilizaremos **OR-Tools** con su solver **CP-SAT**, representando cada decisión de asignación como una variable booleana y expresando las reglas de **cupo por vuelo**, **mínimos por rol**, **cobertura de idiomas** y **no consecutividad** mediante **restricciones lineales y lógicas**. Aprovecharemos **estrategias de decisión** e _hints_ para priorizar primero los vuelos más restrictivos y los tripulantes multilingües, y validaremos la solución con **verificaciones automáticas** por restricción. El solucionador reporta estados de búsqueda (**FEASIBLE/OPTIMAL/INFEASIBLE**) y permite condicionar reglas mediante _enforcement literals_, lo que lo hace especialmente adecuado para problemas de **asignación de personal** como este. **Según la documentación oficial de OR-Tools (2024)**, CP-SAT combina técnicas de programación por restricciones, SAT y programación entera para trabajar eficientemente con **modelos discretos** con múltiples restricciones lógicas, justo el patrón que presenta Aircrew Assignment.

---

## 2. Objetivos

**OG:** Modelar y resolver el problema **Aircrew Assignment** como un CSP usando **OR-Tools**, evaluando **heurísticas** y **restricciones globales** para asignar tripulaciones a vuelos cumpliendo **cupos, roles e idiomas**, y justificando decisiones con **evidencia técnica** y **fuentes especializadas**.

- **OE1:** Analizar el problema **Aircrew Assignment** identificando **variables internas y externas**, **relaciones causa-efecto** y su impacto en la asignación de tripulaciones dentro del contexto de la computación.
- **OE2:** Modelar el problema como un **CSP**, definiendo **variables**, **dominios** y **restricciones** (cupos, roles, idiomas, no consecutivos) para representar formalmente el sistema.
- **OE3:** Desarrollar y **validar** una solución factible que cubra todos los vuelos **sin violar** las restricciones establecidas, demostrando **eficiencia** y **coherencia técnica**.
- **OE4:** **Comparar** alternativas de solución mediante **restricciones globales** y **heurísticas de búsqueda**, evaluando su **rendimiento** y seleccionando la más **óptima**.

---

## 3. Marco Teórico

### 3.1. Contexto del problema en operaciones aeronáuticas

La asignación de tripulaciones es un componente crítico del _airline scheduling_ porque impacta **puntualidad**, **costos** y **cumplimiento normativo** (descansos, roles, cobertura de idiomas). Recientemente se destaca una tendencia a **modelos integrados** que coordinan _pairing_ y _rostering_ con **ruteo de aeronaves** y **recuperación** ante disrupciones, pues el tratamiento separado puede degradar la calidad global del plan. Además, se enfatiza la **robustez**: planes que absorben ausencias o demoras con menor necesidad de replanificación de última hora. Estos enfoques combinan **optimización clásica** con **Programación por Restricciones (CP)** para capturar reglas complejas sin perder **factibilidad operativa** (Xu, 2024).

### 3.2. Programación por Restricciones (CP/CSP) en problemas de _scheduling_

CP permite modelar decisiones como variables (¿quién asignar a qué vuelo?) sujetas a **restricciones estructuradas**, y emplea **propagación** para eliminar valores no viables antes de la búsqueda. En _scheduling_ moderno, CP **compite** con soluciones matemáticas cuando las reglas son **densas** y la factibilidad estricta es **crítica**. (Gao, Zhu & Zhang, 2023; Wessén, Carlsson, Schulte & Bäck, 2023).

### 3.3. _Global constraints_ y su utilidad

Las _global constraints_ son **predicados de alto nivel** que agrupan patrones habituales (conteos, secuencias, solapamientos) y los resuelven con **algoritmos especializados** que hacen filtrado más eficiente. En este caso, son útiles para:

- **R1. Tamaño exacto de tripulación por vuelo (conteo exacto).**  
  Cada vuelo exige un número fijo de tripulantes; en CP esto se modela con _global constraints_ de conteo, que permiten fijar de forma declarativa cuántas asignaciones deben activarse por vuelo. El uso de conteos globales reduce ambigüedad y mejora el filtrado respecto a descomposiciones _ad hoc_, lo que acelera la búsqueda y mantiene la factibilidad operativa. (MiniZinc Team, 2025; Xu, 2024).

- **R2. Mínimo de azafatas por vuelo (conteo por categoría).**  
  Además del tamaño total, se impone un mínimo de _hostesses_ en cada vuelo. En CP, este requisito se expresa como un conteo selectivo (subconjunto) sobre la misma estructura de asignación, aprovechando predicados de conteo de la biblioteca estándar para mantener **consistencia fuerte** en la composición por roles. (MiniZinc Team, 2025).

- **R3. Mínimo de sobrecargos por vuelo (conteo por categoría).**  
  Simétricamente, se exige un mínimo de _stewards_; el patrón vuelve a ser un conteo global sobre un subconjunto. En la práctica, consolidar R2 y R3 como conteos globales facilita **trazabilidad** (requisito → restricción) y **comparaciones experimentales** con/sin _globales_. (MiniZinc Team, 2025).

- **R4. Cobertura de idiomas por vuelo (existencia/cobertura mínima).**  
  Cada vuelo debe incluir al menos una persona que hable **francés, español y alemán**. Este tipo de “existencia de al menos uno” se formula como un conteo con cota inferior y es un caso típico de _coverage constraint_; en CP moderno se implementa con los mismos predicados de conteo, manteniendo la **semántica declarativa** y la **eficiencia** del filtrado. (MiniZinc Team, 2025; Xu, 2024).

- **R5. Restricción de secuencia: prohibición de vuelos consecutivos para una misma persona.**  
  Para evitar asignación inmediata entre vuelos contiguos, se usa una **restricción de secuencia/ventana deslizante** que limita cuántas asignaciones consecutivas puede recibir un empleado. En catálogos actuales, esto se modela con `sliding_sum/sequence` (o, si se manejan tiempos, con `NoOverlap/Cumulative` de OR-Tools); estas primitivas encapsulan el patrón y proveen **filtrados eficientes**. (MiniZinc Team, 2024; Google Developers, 2024–2025).

Estas restricciones globales reducen el **esfuerzo de modelado** y mejoran el **rendimiento** en comparación con descomposiciones manuales.

### 3.4. Regla “no dos vuelos siguientes” mediante secuencias

Para la restricción que **prohíbe** asignar una persona a dos vuelos consecutivos, las técnicas modernas usan **restricciones de secuencia** (_sliding windows_) que controlan cuántas veces ocurre un evento en una ventana deslizante de longitud fija. Esta lógica se adapta bien al enunciado de “no vuelos siguientes” con una **ventana de tamaño 3** (el vuelo, el siguiente y el que sigue). (van Hoeve, Pesant, Rousseau & Sabharwal, 2006).

### 3.5. Herramientas actuales (2022–2025) y soporte de modelado

- **OR-Tools CP-SAT (v9.11–9.12, 2024–2025)** incluye tipos de **variables de intervalo** y **restricciones** para programación de tareas (`NoOverlap`, `Cumulative`), además de **estrategias de búsqueda** configurables. Las mejoras recientes han impulsado el **rendimiento** en modelos con múltiples restricciones. (Google Developers, 2024–2025).

### 3.6. Metodologías de evaluación y robustez operativa

La comparación de enfoques se basa en métricas como **tiempo de resolución**, **número de nodos explorados** y **porcentaje de factibilidad**. En el ámbito aeronáutico, también se introducen conceptos de **reserva de tripulación** y **robustez** ante ausencias, que implican diseñar asignaciones con **margen de maniobra** para eventos imprevistos. (Schrotenboer, Wenneker, Ursavas & Zhu, 2023).

---

## 4. Desarrollo

### 4.1. Datos del Problema

- **20 empleados:** 10 sobrecargos (_stewards_) y 10 azafatas (_air hostesses_).
- **10 vuelos** con requisitos específicos de tripulación.
- **Restricciones** de idiomas, género y disponibilidad.

### 4.2. Variables de Decisión

El modelo utiliza **variables booleanas** binarias:

- `x[e][f]`: Variable binaria que indica si el empleado **e** está asignado al vuelo **f**.
- `x[e][f] = 1` si el empleado e es asignado al vuelo f.
- `x[e][f] = 0` en caso contrario.

### 4.3. Restricciones del Modelo

**Restricción de número exacto de tripulación por vuelo**  
Para cada vuelo _f_, el número total de empleados asignados debe ser exacto:

```
∑ (e = 1..20) x[e][f] = aircrew_required[f]
```

_Implementación en OR-Tools:_ `model.Add(sum(...) == value)`.

**Restricción de número mínimo de azafatas**  
Para cada vuelo _f_, debe haber al menos un número mínimo de azafatas:

```
∑ (e ∈ air_hostesses) x[e][f] ≥ min_hostesses[f]
```

_Implementación en OR-Tools:_ `model.Add(sum(...) >= value)`.

**Restricción de número mínimo de sobrecargos**  
Para cada vuelo _f_, debe haber al menos un número mínimo de sobrecargos:

```
∑ (e ∈ stewards) x[e][f] ≥ min_stewards[f]
```

**Restricciones de idiomas**  
Cada vuelo debe tener al menos un empleado que hable cada uno de los tres idiomas requeridos:

- **Francés:**
  ```
  ∑ (e ∈ french_speakers) x[e][f] ≥ 1   ∀ f
  ```
- **Español:**
  ```
  ∑ (e ∈ spanish_speakers) x[e][f] ≥ 1  ∀ f
  ```
- **Alemán:**
  ```
  ∑ (e ∈ german_speakers) x[e][f] ≥ 1   ∀ f
  ```

**Restricción de no vuelos consecutivos**  
Un empleado no puede ser asignado a dos vuelos consecutivos:

```
x[e][f] + x[e][f+1] ≤ 1    ∀ e, ∀ f ∈ {1,2,...,9}
```

Esta restricción asegura que si un empleado está asignado al vuelo _f_, no puede estar asignado al vuelo _f+1_, y viceversa.

### 4.4. Restricciones Globales Utilizadas

**Restricciones de suma (Linear Constraints).**  
Las restricciones de número exacto y mínimo de tripulación son **lineales** que el solver maneja eficientemente mediante **propagación**.

**Restricciones implícitas de cardinalidad.**  
Las restricciones de número exacto de tripulación son efectivamente **cardinalidad** que limitan cuántas variables pueden ser verdaderas en un conjunto dado.

---

## 5. Bibliografía

- Google Developers. (2024). **CP-SAT Solver | OR-Tools**.  
  https://developers.google.com/optimization/cp/cp_solver

- Google Developers. (2024). **Python Reference: CP-SAT | OR-Tools**.  
  https://developers.google.com/optimization/reference/python/sat/python/cp_model

- MiniZinc Team. (2024). **Mathematical constraints — sliding_sum (v2.9.x)**.  
  https://docs.minizinc.dev/en/latest/lib-globals-math.html

- MiniZinc Team. (2025). **MiniZinc resources & standard library (v2.9.4): counting / global constraints**.  
  https://www.minizinc.org/resources/

- Xu, Y. (2024). **Airline scheduling optimization: Literature review and modelling methodologies**. _Intelligent Transportation Infrastructure_, 3(1), liad026.  
  https://doi.org/10.1093/iti/liad026

- Gao, J., Zhu, X., & Zhang, R. (2023). **Optimization of parallel test task scheduling with constraint satisfaction**. _The Journal of Supercomputing_, 79, 7206–7227.  
  https://doi.org/10.1007/s11227-022-04943-0

- Juvin, C., Hebrard, E., Houssin, L., & Lopez, P. (2023). **An efficient constraint programming approach to preemptive job shop scheduling**. In _CP 2023_ (LIPIcs 280, 19:1–19:16).  
  https://doi.org/10.4230/LIPIcs.CP.2023.19

- Schrotenboer, A. H., Wenneker, R., Ursavas, E., & Zhu, S. X. (2023). **Reliable reserve-crew scheduling for airlines**. _Transportation Research Part E: Logistics and Transportation Review_, 178, 103283.  
  https://doi.org/10.1016/j.tre.2023.103283

- Wessén, J., Carlsson, M., Schulte, C., & Bäck, T. (2023). **A CP model for scheduling and workspace layout of a dual-arm assembly robot**. _Constraints_, 28(1–2), 71–104.  
  https://doi.org/10.1007/s10601-023-09345-4

---

**Fin del documento.**
