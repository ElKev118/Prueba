# TALLER 1: MÓDULO 1 - VIRTUALIZACIÓN DE LA CPU

## Objetivos

**Objetivo general:** Evaluar el impacto de distintas políticas de planificación de la CPU en el rendimiento de un sistema operativo mediante simulación.

**Objetivos Específicos:**
- Analizar los mecanismos y características de las políticas de planificación FCFS, SJF, STCF, Round Robin y Prioridades (MLFQ)
- Implementar estas políticas en un simulador para procesar una serie de tareas predefinidas
- Medir y comparar el rendimiento de cada política utilizando métricas como turnaround time y wait time promedio

---

## Parte 1: Políticas de Planificación

### Datos del Problema

| Process | CPU burst (μs) |
|---------|----------------|
| **P1**  | 135            |
| **P2**  | 102            |
| **P3**  | 56             |
| **P4**  | 148            |
| **P5**  | 125            |

**Supuestos:**
- Los cinco procesos llegan en $t = 0$ (simultáneamente)
- El intervalo de cambio de contexto es despreciable (0)
- Todos los procesos solo usan CPU (sin I/O)

---

### 1. FCFS (First Come, First Served) / FIFO

#### Procedimiento a Mano

**Principio:** Los procesos se ejecutan en el orden en que llegan.

**Orden de ejecución:** P1 → P2 → P3 → P4 → P5

#### Diagrama de Gantt

```
Tiempo: 0      135    237    293    441    566
       |---P1---|---P2---|---P3---|---P4---|---P5---|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 135        | 135         | 0       |
| **P2**  | 0         | 135       | 237        | 237         | 135     |
| **P3**  | 0         | 237       | 293        | 293         | 237     |
| **P4**  | 0         | 293       | 441        | 441         | 293     |
| **P5**  | 0         | 441       | 566        | 566         | 441     |
| **Promedio** | 0   | 179.2     | 354.4      | **334.4**    | **221.2** |

#### Fórmulas Utilizadas

- $T_{turnaround} = T_{completion} - T_{arrival}$
- $T_{ready} = T_{firstrun} - T_{arrival}$ (Tiempo de espera / Wait time)
- $T_{response} = T_{firstrun} - T_{arrival}$

#### Análisis FCFS

**Ventajas:**
- Simple de implementar
- Justo para procesos que llegan juntos

**Desventajas:**
- **Efecto Convoy:** Procesos cortos deben esperar a procesos largos
- Alto turnaround time promedio (334.4 μs)
- Pobre respuesta para procesos interactivos (espera de 441 μs para P5)

---

### 2. SJF (Shortest Job First)

#### Procedimiento a Mano

**Principio:** Ejecutar primero el proceso con la ráfaga de CPU más corta.

**Orden de ejecución (por tamaño de ráfaga):**
1. P3: 56 μs (más corto)
2. P2: 102 μs
3. P5: 125 μs
4. P1: 135 μs
5. P4: 148 μs (más largo)

#### Diagrama de Gantt

```
Tiempo: 0     56    158    283    418    566
       |---P3---|---P2---|---P5---|---P1---|---P4---|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 283       | 418        | 418         | 283     |
| **P2**  | 0         | 56        | 158        | 158         | 56      |
| **P3**  | 0         | 0         | 56         | 56          | 0       |
| **P4**  | 0         | 418       | 566        | 566         | 418     |
| **P5**  | 0         | 158       | 283        | 283         | 158     |
| **Promedio** | 0   | 183       | 296.2      | **296.2**    | **183.0** |

#### Análisis SJF

**Ventajas:**
- Mejor turnaround time promedio (296.2 μs) que FCFS
- Optimiza para procesos cortos
- Minimiza el número de procesos esperando

**Desventajas:**
- Requiere conocer los tiempos de ráfaga (irreal en la práctica)
- **Inanición potencial:** Procesos largos pueden esperar indefinidamente si llegan procesos cortos
- Pobre para procesos largos (P4 espera 418 μs)

---

### 3. Round Robin (RR) con Quantum = 40 μs

#### Procedimiento a Mano

**Principio:** Cada proceso obtiene una porción de tiempo (quantum) de 40 μs. Si no termina, va al final de la cola.

**Simulación paso a paso:**

| Tiempo | Proceso | Quantum usado | Restante | Acción          |
|--------|---------|--------------|----------|-----------------|
| 0-40   | P1      | 40           | 95       | → Cola          |
| 40-80  | P2      | 40           | 62       | → Cola          |
| 80-120 | P3      | 40           | 16       | → Cola          |
| 120-160| P4      | 40           | 108      | → Cola          |
| 160-200| P5      | 40           | 85       | → Cola          |
| 200-240| P1      | 40           | 55       | → Cola          |
| 240-280| P2      | 40           | 22       | → Cola          |
| 280-296| P3      | 16           | 0        | **COMPLETADO**  |
| 296-336| P4      | 40           | 68       | → Cola          |
| 336-376| P5      | 40           | 45       | → Cola          |
| 376-416| P1      | 40           | 15       | → Cola          |
| 416-438| P2      | 22           | 0        | **COMPLETADO**  |
| 438-478| P4      | 40           | 28       | → Cola          |
| 478-518| P5      | 40           | 5        | → Cola          |
| 518-533| P1      | 15           | 0        | **COMPLETADO**  |
| 533-561| P4      | 28           | 0        | **COMPLETADO**  |
| 561-566| P5      | 5            | 0        | **COMPLETADO**  |

#### Diagrama de Gantt

```
Tiempo: 0    40   80  120  160  200  240  280 296 336 376 416 438 478 518 533 561 566
       |P1 |P2 |P3 |P4 |P5 |P1 |P2 |P3|P4  |P5  |P1  |P2 |P4  |P5  |P1  |P4 |P5|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 533        | 533         | 0       |
| **P2**  | 0         | 40        | 438        | 438         | 40      |
| **P3**  | 0         | 80        | 296        | 296         | 80      |
| **P4**  | 0         | 120       | 561        | 561         | 120     |
| **P5**  | 0         | 160       | 566        | 566         | 160     |
| **Promedio** | 0   | 80        | 478.8      | **478.8**    | **80.0** |

#### Análisis RR

**Ventajas:**
- **Mejor response time** (P1 responde inmediatamente, en promedio 80 μs)
- Más equitativo: todos los procesos reciben oportunidades iguales
- Adecuado para sistemas interactivos

**Desventajas:**
- **Pobre turnaround time** (478.8 μs) - peor que FCFS y SJF
- Overhead de cambios de contexto frecuentes (aunque los despreciamos aquí)
- El quantum elegido (40) es arbitrario y afecta significativamente el rendimiento

---

### 4. Comparación de Políticas y Conclusión

#### Tabla Comparativa

| Métrica | FCFS | SJF | RR (Q=40) |
|---------|------|-----|-----------|
| Turnaround Promedio (μs) | 334.4 | **296.2** | 478.8 |
| Wait Time Promedio (μs) | 221.2 | 183.0 | **80.0** |
| Response Time Promedio (μs) | 221.2 | 183.0 | **80.0** |
| Efecto Convoy | **Sí** | No | No |
| Inanición Potencial | No | **Sí** | No |
| Equidad | Regular | Pobre | **Excelente** |

#### Conclusión: ¿Cuál es la mejor política?

**Respuesta:** Depende del objetivo del sistema:

1. **Para sistemas batch (minimizar turnaround time):**
   - **SJF es la mejor** (296.2 μs)
   - Minimiza el tiempo total de ejecución
   - Pero requiere conocer los tiempos de ráfaga

2. **Para sistemas interactivos (response time):**
   - **RR es la mejor** (80.0 μs promedio)
   - Proporciona respuesta rápida a todos los usuarios
   - Más equitativo en la distribución del CPU

3. **Equilibrio:**
   - **MLFQ** (siguiente sección) combina lo mejor de ambos
   - Usa retroalimentación para adaptar la política automáticamente

**En conclusión con estos datos:** SJF es superior para turnaround time, pero RR es mejor para equity y response time. En la práctica, **MLFQ o una política adaptativa es preferible**.

---

### 5. STCF (Shortest Time-to-Completion First) - Procesos con llegadas escalonadas

#### Datos del Problema

| Process | Arrival time (μs) | CPU burst (μs) |
|---------|------------------|----------------|
| **P1**  | 0                | 135            |
| **P2**  | 145              | 102            |
| **P3**  | 200              | 56             |
| **P4**  | 300              | 148            |
| **P5**  | 400              | 125            |

#### Procedimiento a Mano

**Principio:** STCF es la versión preemptiva de SJF. Cuando llega un nuevo proceso, se compara el tiempo restante de todos los procesos (incluyendo el que está ejecutando) y se ejecuta el que requiere menos tiempo.

**Simulación cronológica:**

| Tiempo | Evento           | Procesos en Sistema | Proceso Ejecutando | Tiempo Restante |
|--------|-----------------|------------------|------------------|-----------------|
| 0      | Llega P1        | P1                | P1                | 135            |
| 0-145  | P1 ejecutando   | P1                | P1                | 0              |
| 145    | Llega P2        | P2                | P2                | 102            |
| 145-200| P2 ejecutando   | P2                | P2                | 47             |
| 200    | Llega P3, preempt| P2, P3           | P3 (56 < 47)      | 56             |
| 200-256| P3 ejecutando   | P2, P3            | P3                | 0              |
| 256    | Llega P4        | P2, P4            | P2 (47 < 148)     | 47             |
| 256-303| P2 ejecutando   | P2, P4            | P2                | 0              |
| 303    | Llega P5        | P4, P5            | P4 (148 < 125)    | 148            |
| 303-400| P4 ejecutando   | P4, P5            | P4                | 51             |
| 400    | P5 esperando    | P4, P5            | P4 (51 < 125)     | 51             |
| 400-451| P4 ejecutando   | P4, P5            | P4                | 0              |
| 451-576| P5 ejecutando   | P5                | P5                | 0              |

#### Diagrama de Gantt

```
Tiempo: 0      145  200  256 303  400  451  576
       |---P1---|---P2---|P3|---P2---|---P4---|---P5---|
                      (preempted)
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 145        | 145         | 0       |
| **P2**  | 145       | 145       | 303        | 158         | 0       |
| **P3**  | 200       | 200       | 256        | 56          | 0       |
| **P4**  | 300       | 303       | 451        | 151         | 3       |
| **P5**  | 400       | 451       | 576        | 176         | 51      |
| **Promedio** | 169   | 219.8     | 346.2      | **177.2**    | **10.8** |

#### Análisis STCF

**Ventajas:**
- **Mejor turnaround time** (177.2 μs) - mucho mejor que FCFS, SJF sin preemption, RR
- Minimiza el tiempo de espera de procesos cortos
- Optimal para minimizar turnaround time cuando todos los tiempos son conocidos

**Desventajas:**
- Requiere conocer los tiempos de ráfaga futuros
- Puede causar preemption frecuente (overhead)
- No es adecuado para interactivos (pobre response time para procesos largos)
- P5 espera 51 μs después de llegar

---

## Parte 2: MLFQ (Multi-Level Feedback Queue)

### Datos del Problema

| Process | Arrival time (μs) | Execution time (μs) |
|---------|------------------|-------------------|
| **P0**  | 0                | 8                 |
| **P1**  | 1                | 4                 |
| **P2**  | 2                | 9                 |
| **P3**  | 3                | 5                 |

### Configuración MLFQ

**Tres colas de prioridad:**
- **Q2** (máxima prioridad): Quantum = 1 μs
- **Q1** (media prioridad): Quantum = 2 μs
- **Q0** (baja prioridad): Quantum = 4 μs

**Reglas:**
1. Si prioridad(A) > prioridad(B); A se ejecuta
2. Si prioridad(A) = prioridad(B); RR entre A y B
3. Cuando un trabajo llega, se ubica en Q2 (máxima prioridad)
4. Cuando un trabajo consume su allotment en un nivel, baja de prioridad
5. Boost de prioridad: NO APLICA (S = 0)

### Procedimiento a Mano

#### Simulación Cronológica

| Tiempo | Evento | Q2 | Q1 | Q0 | Ejecutando | Restante | Acción |
|--------|--------|----|----|----|-----------|---------|----|
| 0      | P0 llega | P0 | | | P0 | 8 | Ejecuta Q2 |
| 1      | P0 quantum (1), P1 llega | P1 | P0 | | P1 | 4 | Ejecuta Q2 |
| 2      | P1 quantum (1), P2 llega | P2 | P0 | | P2 | 9 | Ejecuta Q2 |
| 3      | P2 quantum (1), P3 llega | P3 | P0 | | P3 | 5 | Ejecuta Q2 |
| 4      | P3 quantum (1), baja | | P0 | P3 | P0 | 7 | Ejecuta Q1 |
| 5      | P0 quantum (1) | | P0 | P3 | P0 | 6 | Ejecuta Q1 (total 2) |
| 6      | P0 baja, P1 quantum (1) | | P1 | P3, P0 | P1 | 3 | Ejecuta Q1 |
| 7      | P1 quantum (1) | | P1 | P3, P0 | P1 | 2 | Ejecuta Q1 (total 2) |
| 8      | P1 baja, P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 8 | Ejecuta Q1 |
| 9      | P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 7 | Ejecuta Q1 (total 2) |
| 10     | P2 baja, P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 4 | Ejecuta Q1 |
| 11     | P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 3 | Ejecuta Q1 (total 2) |
| 12     | P3 baja, P0 quantum (1) | | P0 | P2, P1, P3 | P0 | 6 | Ejecuta Q1 |
| 13     | P0 quantum (1) | | P0 | P2, P1, P3 | P0 | 5 | Ejecuta Q1 (total 2) |
| 14     | P0 baja, P1 quantum (1) | | P1 | P2, P3, P0 | P1 | 2 | Ejecuta Q1 |
| 15     | P1 quantum (1) | | P1 | P2, P3, P0 | P1 | 1 | Ejecuta Q1 (total 2) |
| 16     | P1 baja, P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 7 | Ejecuta Q1 |
| 17     | P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 6 | Ejecuta Q1 (total 2) |
| 18     | P2 baja, P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 3 | Ejecuta Q1 |
| 19     | P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 2 | Ejecuta Q1 (total 2) |
| 20     | P3 baja, P0 quantum (4) | | | P2, P1, P3, P0 | P0 | 5 | Ejecuta Q0 |
| 24     | P0 quantum (4) | | | P2, P1, P3, P0 | P0 | 1 | Ejecuta Q0 (total 4) |
| 25     | P0 DONE, P1 quantum (4) | | | P2, P3, P1 | P1 | 1 | Ejecuta Q0 |
| 26     | P1 DONE, P3 quantum (4) | | | P2, P3 | P3 | 0 | P3 DONE |
| 26     | P2 quantum (4) | | | P2 | P2 | 6 | Ejecuta Q0 |
| 30     | P2 quantum (4) | | | P2 | P2 | 2 | Ejecuta Q0 (total 4) |
| 34     | P2 quantum (2) | | | P2 | P2 | 0 | P2 DONE |

#### Diagrama de Gantt

```
Tiempo: 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 24 25 26 30 34
       |P0|P1|P2|P3|P0|P0|P1|P1|P2|P2|P3|P3|P0|P0|P1|P1|P2|P2|P3|P3|P0   |P1|P3|P2      |
       Q2 Q2 Q2 Q2 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q0   Q0 Q0 Q0
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P0**  | 0         | 0         | 25         | 25          | 0       |
| **P1**  | 1         | 1         | 26         | 25          | 0       |
| **P2**  | 2         | 2         | 34         | 32          | 0       |
| **P3**  | 3         | 3         | 26         | 23          | 0       |
| **Promedio** | 1.5   | 1.5       | 27.75      | **26.25**    | **0.0** |

#### Análisis MLFQ

**Ventajas:**
- Excelente turnaround time (26.25 μs) combinado con buen response time
- Adaptation automática: procesos cortos obtienen alta prioridad
- No requiere conocer los tiempos de ráfaga
- Equidad: todos obtienen oportunidades

**Características observadas:**
- P1 y P3 (procesos cortos) completan rápidamente
- P2 (proceso largo) es desplazado a Q0 pero eventualmente completa
- El quantum variable por cola ($1, 2, 4$ μs) es optimal para este patrón
- Tiempos de ready = 0 para todos (excelente interactividad)

---

## Instrucciones para Usar los Scripts

### scheduler.py

**Ubicación:** [GitHub OSTEP Homework](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched)

**Uso en Windows o Linux:**

```bash
# 1. Descargar el script
# Opción A: Clonar todo el repositorio
git clone https://github.com/remzi-arpacidusseau/ostep-homework.git
cd ostep-homework/cpu-sched

# Opción B: Descargar solo scheduler.py
curl -O https://raw.githubusercontent.com/remzi-arpacidusseau/ostep-homework/master/cpu-sched/scheduler.py

# 2. Ejecutar el programa (sin respuestas)
python scheduler.py -p FCFS -l 135,102,56,148,125

# 3. Ver respuestas
python scheduler.py -p FCFS -l 135,102,56,148,125 -c

# 4. Probar otras políticas
python scheduler.py -p SJF -l 135,102,56,148,125 -c
python scheduler.py -p RR -l 135,102,56,148,125 -q 40 -c
```

**Parámetros principales:**
- `-p` : Política (FCFS, SJF, RR)
- `-l` : Lista de tiempos de ráfaga (comma-separated)
- `-q` : Quantum para Round Robin
- `-c` : Mostrar respuestas calculadas
- `-h` : Ayuda completa

### mlfq.py

**Ubicación:** [GitHub OSTEP Homework](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched-mlfq)

**Uso en Windows o Linux:**

```bash
# 1. Descargar el script
cd ../cpu-sched-mlfq
curl -O https://raw.githubusercontent.com/remzi-arpacidusseau/ostep-homework/master/cpu-sched-mlfq/mlfq.py

# 2. Ejecutar el simulador (sin respuestas)
python mlfq.py -l 0,8,0:1,4,0:2,9,0:3,5,0 -Q 1,2,4 -A 1,2,4

# 3. Ver respuestas
python mlfq.py -l 0,8,0:1,4,0:2,9,0:3,5,0 -Q 1,2,4 -A 1,2,4 -c
```

**Formato del job list (-l):** `startTime,runTime,ioFreq:startTime2,runTime2,ioFreq2:...`

**Parámetros principales:**
- `-l` : Lista de jobs en formato especificado
- `-Q` : Quantum para cada cola (comma-separated)
- `-A` : Allotment para cada cola (en time slices)
- `-c` : Mostrar respuestas
- `-h` : Ayuda completa

---

## Conclusiones Generales del Taller

1. **No existe política perfecta:** Cada una optimiza una métrica diferente
2. **Trade-offs:** Mejorar response time requiere afectar turnaround time
3. **MLFQ es práctica:** Combina lo mejor de múltiples políticas
4. **Adaptación es clave:** Los planificadores modernos se adaptan al comportamiento
5. **Métricas importan:** Elegir la métrica correcta es fundamental para el diseño

---

## Referencias

- **Libro:** Operating Systems: Three Easy Pieces (Arpaci-Dusseau)
  - Capítulo 7: Scheduling
  - Capítulo 8: Multi-Level Feedback Queue

- **Código:** [OSTEP Homework - cpu-sched](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched)

- **Diapositivas del curso:** Planificación de Procesos (Danny Múnera, 2024)








# TALLER 1: MÓDULO 1 - VIRTUALIZACIÓN DE LA CPU

## Objetivos

**Objetivo general:** Evaluar el impacto de distintas políticas de planificación de la CPU en el rendimiento de un sistema operativo mediante simulación.

**Objetivos Específicos:**
- Analizar los mecanismos y características de las políticas de planificación FCFS, SJF, STCF, Round Robin y Prioridades (MLFQ)
- Implementar estas políticas en un simulador para procesar una serie de tareas predefinidas
- Medir y comparar el rendimiento de cada política utilizando métricas como turnaround time y wait time promedio

---

## Parte 1: Políticas de Planificación

### Datos del Problema

| Process | CPU burst (μs) |
|---------|----------------|
| **P1**  | 135            |
| **P2**  | 102            |
| **P3**  | 56             |
| **P4**  | 148            |
| **P5**  | 125            |

**Supuestos:**
- Los cinco procesos llegan en $t = 0$ (simultáneamente)
- El intervalo de cambio de contexto es despreciable (0)
- Todos los procesos solo usan CPU (sin I/O)

---

### 1. FCFS (First Come, First Served) / FIFO

#### Procedimiento a Mano

**Principio:** Los procesos se ejecutan en el orden en que llegan.

**Orden de ejecución:** P1 → P2 → P3 → P4 → P5

#### Diagrama de Gantt

```
Tiempo: 0      135    237    293    441    566
       |---P1---|---P2---|---P3---|---P4---|---P5---|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 135        | 135         | 0       |
| **P2**  | 0         | 135       | 237        | 237         | 135     |
| **P3**  | 0         | 237       | 293        | 293         | 237     |
| **P4**  | 0         | 293       | 441        | 441         | 293     |
| **P5**  | 0         | 441       | 566        | 566         | 441     |
| **Promedio** | 0   | 179.2     | 354.4      | **334.4**    | **221.2** |

#### Fórmulas Utilizadas

- $T_{turnaround} = T_{completion} - T_{arrival}$
- $T_{ready} = T_{firstrun} - T_{arrival}$ (Tiempo de espera / Wait time)
- $T_{response} = T_{firstrun} - T_{arrival}$

#### Análisis FCFS

**Ventajas:**
- Simple de implementar
- Justo para procesos que llegan juntos

**Desventajas:**
- **Efecto Convoy:** Procesos cortos deben esperar a procesos largos
- Alto turnaround time promedio (334.4 μs)
- Pobre respuesta para procesos interactivos (espera de 441 μs para P5)

---

### 2. SJF (Shortest Job First)

#### Procedimiento a Mano

**Principio:** Ejecutar primero el proceso con la ráfaga de CPU más corta.

**Orden de ejecución (por tamaño de ráfaga):**
1. P3: 56 μs (más corto)
2. P2: 102 μs
3. P5: 125 μs
4. P1: 135 μs
5. P4: 148 μs (más largo)

#### Diagrama de Gantt

```
Tiempo: 0     56    158    283    418    566
       |---P3---|---P2---|---P5---|---P1---|---P4---|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 283       | 418        | 418         | 283     |
| **P2**  | 0         | 56        | 158        | 158         | 56      |
| **P3**  | 0         | 0         | 56         | 56          | 0       |
| **P4**  | 0         | 418       | 566        | 566         | 418     |
| **P5**  | 0         | 158       | 283        | 283         | 158     |
| **Promedio** | 0   | 183       | 296.2      | **296.2**    | **183.0** |

#### Análisis SJF

**Ventajas:**
- Mejor turnaround time promedio (296.2 μs) que FCFS
- Optimiza para procesos cortos
- Minimiza el número de procesos esperando

**Desventajas:**
- Requiere conocer los tiempos de ráfaga (irreal en la práctica)
- **Inanición potencial:** Procesos largos pueden esperar indefinidamente si llegan procesos cortos
- Pobre para procesos largos (P4 espera 418 μs)

---

### 3. Round Robin (RR) con Quantum = 40 μs

#### Procedimiento a Mano

**Principio:** Cada proceso obtiene una porción de tiempo (quantum) de 40 μs. Si no termina, va al final de la cola.

**Simulación paso a paso:**

| Tiempo | Proceso | Quantum usado | Restante | Acción          |
|--------|---------|--------------|----------|-----------------|
| 0-40   | P1      | 40           | 95       | → Cola          |
| 40-80  | P2      | 40           | 62       | → Cola          |
| 80-120 | P3      | 40           | 16       | → Cola          |
| 120-160| P4      | 40           | 108      | → Cola          |
| 160-200| P5      | 40           | 85       | → Cola          |
| 200-240| P1      | 40           | 55       | → Cola          |
| 240-280| P2      | 40           | 22       | → Cola          |
| 280-296| P3      | 16           | 0        | **COMPLETADO**  |
| 296-336| P4      | 40           | 68       | → Cola          |
| 336-376| P5      | 40           | 45       | → Cola          |
| 376-416| P1      | 40           | 15       | → Cola          |
| 416-438| P2      | 22           | 0        | **COMPLETADO**  |
| 438-478| P4      | 40           | 28       | → Cola          |
| 478-518| P5      | 40           | 5        | → Cola          |
| 518-533| P1      | 15           | 0        | **COMPLETADO**  |
| 533-561| P4      | 28           | 0        | **COMPLETADO**  |
| 561-566| P5      | 5            | 0        | **COMPLETADO**  |

#### Diagrama de Gantt

```
Tiempo: 0    40   80  120  160  200  240  280 296 336 376 416 438 478 518 533 561 566
       |P1 |P2 |P3 |P4 |P5 |P1 |P2 |P3|P4  |P5  |P1  |P2 |P4  |P5  |P1  |P4 |P5|
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 533        | 533         | 0       |
| **P2**  | 0         | 40        | 438        | 438         | 40      |
| **P3**  | 0         | 80        | 296        | 296         | 80      |
| **P4**  | 0         | 120       | 561        | 561         | 120     |
| **P5**  | 0         | 160       | 566        | 566         | 160     |
| **Promedio** | 0   | 80        | 478.8      | **478.8**    | **80.0** |

#### Análisis RR

**Ventajas:**
- **Mejor response time** (P1 responde inmediatamente, en promedio 80 μs)
- Más equitativo: todos los procesos reciben oportunidades iguales
- Adecuado para sistemas interactivos

**Desventajas:**
- **Pobre turnaround time** (478.8 μs) - peor que FCFS y SJF
- Overhead de cambios de contexto frecuentes (aunque los despreciamos aquí)
- El quantum elegido (40) es arbitrario y afecta significativamente el rendimiento

---

### 4. Comparación de Políticas y Conclusión

#### Tabla Comparativa

| Métrica | FCFS | SJF | RR (Q=40) |
|---------|------|-----|-----------|
| Turnaround Promedio (μs) | 334.4 | **296.2** | 478.8 |
| Wait Time Promedio (μs) | 221.2 | 183.0 | **80.0** |
| Response Time Promedio (μs) | 221.2 | 183.0 | **80.0** |
| Efecto Convoy | **Sí** | No | No |
| Inanición Potencial | No | **Sí** | No |
| Equidad | Regular | Pobre | **Excelente** |

#### Conclusión: ¿Cuál es la mejor política?

**Respuesta:** Depende del objetivo del sistema:

1. **Para sistemas batch (minimizar turnaround time):**
   - **SJF es la mejor** (296.2 μs)
   - Minimiza el tiempo total de ejecución
   - Pero requiere conocer los tiempos de ráfaga

2. **Para sistemas interactivos (response time):**
   - **RR es la mejor** (80.0 μs promedio)
   - Proporciona respuesta rápida a todos los usuarios
   - Más equitativo en la distribución del CPU

3. **Equilibrio:**
   - **MLFQ** (siguiente sección) combina lo mejor de ambos
   - Usa retroalimentación para adaptar la política automáticamente

**En conclusión con estos datos:** SJF es superior para turnaround time, pero RR es mejor para equity y response time. En la práctica, **MLFQ o una política adaptativa es preferible**.

---

### 5. STCF (Shortest Time-to-Completion First) - Procesos con llegadas escalonadas

#### Datos del Problema

| Process | Arrival time (μs) | CPU burst (μs) |
|---------|------------------|----------------|
| **P1**  | 0                | 135            |
| **P2**  | 145              | 102            |
| **P3**  | 200              | 56             |
| **P4**  | 300              | 148            |
| **P5**  | 400              | 125            |

#### Procedimiento a Mano

**Principio:** STCF es la versión preemptiva de SJF. Cuando llega un nuevo proceso, se compara el tiempo restante de todos los procesos (incluyendo el que está ejecutando) y se ejecuta el que requiere menos tiempo.

**Simulación cronológica:**

| Tiempo | Evento           | Procesos en Sistema | Proceso Ejecutando | Tiempo Restante |
|--------|-----------------|------------------|------------------|-----------------|
| 0      | Llega P1        | P1                | P1                | 135            |
| 0-145  | P1 ejecutando   | P1                | P1                | 0              |
| 145    | Llega P2        | P2                | P2                | 102            |
| 145-200| P2 ejecutando   | P2                | P2                | 47             |
| 200    | Llega P3, preempt| P2, P3           | P3 (56 < 47)      | 56             |
| 200-256| P3 ejecutando   | P2, P3            | P3                | 0              |
| 256    | Llega P4        | P2, P4            | P2 (47 < 148)     | 47             |
| 256-303| P2 ejecutando   | P2, P4            | P2                | 0              |
| 303    | Llega P5        | P4, P5            | P4 (148 < 125)    | 148            |
| 303-400| P4 ejecutando   | P4, P5            | P4                | 51             |
| 400    | P5 esperando    | P4, P5            | P4 (51 < 125)     | 51             |
| 400-451| P4 ejecutando   | P4, P5            | P4                | 0              |
| 451-576| P5 ejecutando   | P5                | P5                | 0              |

#### Diagrama de Gantt

```
Tiempo: 0      145  200  256 303  400  451  576
       |---P1---|---P2---|P3|---P2---|---P4---|---P5---|
                      (preempted)
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P1**  | 0         | 0         | 145        | 145         | 0       |
| **P2**  | 145       | 145       | 303        | 158         | 0       |
| **P3**  | 200       | 200       | 256        | 56          | 0       |
| **P4**  | 300       | 303       | 451        | 151         | 3       |
| **P5**  | 400       | 451       | 576        | 176         | 51      |
| **Promedio** | 169   | 219.8     | 346.2      | **177.2**    | **10.8** |

#### Análisis STCF

**Ventajas:**
- **Mejor turnaround time** (177.2 μs) - mucho mejor que FCFS, SJF sin preemption, RR
- Minimiza el tiempo de espera de procesos cortos
- Optimal para minimizar turnaround time cuando todos los tiempos son conocidos

**Desventajas:**
- Requiere conocer los tiempos de ráfaga futuros
- Puede causar preemption frecuente (overhead)
- No es adecuado para interactivos (pobre response time para procesos largos)
- P5 espera 51 μs después de llegar

---

## Parte 2: MLFQ (Multi-Level Feedback Queue)

### Datos del Problema

| Process | Arrival time (μs) | Execution time (μs) |
|---------|------------------|-------------------|
| **P0**  | 0                | 8                 |
| **P1**  | 1                | 4                 |
| **P2**  | 2                | 9                 |
| **P3**  | 3                | 5                 |

### Configuración MLFQ

**Tres colas de prioridad:**
- **Q2** (máxima prioridad): Quantum = 1 μs
- **Q1** (media prioridad): Quantum = 2 μs
- **Q0** (baja prioridad): Quantum = 4 μs

**Reglas:**
1. Si prioridad(A) > prioridad(B); A se ejecuta
2. Si prioridad(A) = prioridad(B); RR entre A y B
3. Cuando un trabajo llega, se ubica en Q2 (máxima prioridad)
4. Cuando un trabajo consume su allotment en un nivel, baja de prioridad
5. Boost de prioridad: NO APLICA (S = 0)

### Procedimiento a Mano

#### Simulación Cronológica

| Tiempo | Evento | Q2 | Q1 | Q0 | Ejecutando | Restante | Acción |
|--------|--------|----|----|----|-----------|---------|----|
| 0      | P0 llega | P0 | | | P0 | 8 | Ejecuta Q2 |
| 1      | P0 quantum (1), P1 llega | P1 | P0 | | P1 | 4 | Ejecuta Q2 |
| 2      | P1 quantum (1), P2 llega | P2 | P0 | | P2 | 9 | Ejecuta Q2 |
| 3      | P2 quantum (1), P3 llega | P3 | P0 | | P3 | 5 | Ejecuta Q2 |
| 4      | P3 quantum (1), baja | | P0 | P3 | P0 | 7 | Ejecuta Q1 |
| 5      | P0 quantum (1) | | P0 | P3 | P0 | 6 | Ejecuta Q1 (total 2) |
| 6      | P0 baja, P1 quantum (1) | | P1 | P3, P0 | P1 | 3 | Ejecuta Q1 |
| 7      | P1 quantum (1) | | P1 | P3, P0 | P1 | 2 | Ejecuta Q1 (total 2) |
| 8      | P1 baja, P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 8 | Ejecuta Q1 |
| 9      | P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 7 | Ejecuta Q1 (total 2) |
| 10     | P2 baja, P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 4 | Ejecuta Q1 |
| 11     | P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 3 | Ejecuta Q1 (total 2) |
| 12     | P3 baja, P0 quantum (1) | | P0 | P2, P1, P3 | P0 | 6 | Ejecuta Q1 |
| 13     | P0 quantum (1) | | P0 | P2, P1, P3 | P0 | 5 | Ejecuta Q1 (total 2) |
| 14     | P0 baja, P1 quantum (1) | | P1 | P2, P3, P0 | P1 | 2 | Ejecuta Q1 |
| 15     | P1 quantum (1) | | P1 | P2, P3, P0 | P1 | 1 | Ejecuta Q1 (total 2) |
| 16     | P1 baja, P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 7 | Ejecuta Q1 |
| 17     | P2 quantum (1) | | P2 | P3, P0, P1 | P2 | 6 | Ejecuta Q1 (total 2) |
| 18     | P2 baja, P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 3 | Ejecuta Q1 |
| 19     | P3 quantum (1) | | P3 | P2, P0, P1 | P3 | 2 | Ejecuta Q1 (total 2) |
| 20     | P3 baja, P0 quantum (4) | | | P2, P1, P3, P0 | P0 | 5 | Ejecuta Q0 |
| 24     | P0 quantum (4) | | | P2, P1, P3, P0 | P0 | 1 | Ejecuta Q0 (total 4) |
| 25     | P0 DONE, P1 quantum (4) | | | P2, P3, P1 | P1 | 1 | Ejecuta Q0 |
| 26     | P1 DONE, P3 quantum (4) | | | P2, P3 | P3 | 0 | P3 DONE |
| 26     | P2 quantum (4) | | | P2 | P2 | 6 | Ejecuta Q0 |
| 30     | P2 quantum (4) | | | P2 | P2 | 2 | Ejecuta Q0 (total 4) |
| 34     | P2 quantum (2) | | | P2 | P2 | 0 | P2 DONE |

#### Diagrama de Gantt

```
Tiempo: 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 24 25 26 30 34
       |P0|P1|P2|P3|P0|P0|P1|P1|P2|P2|P3|P3|P0|P0|P1|P1|P2|P2|P3|P3|P0   |P1|P3|P2      |
       Q2 Q2 Q2 Q2 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q1 Q0   Q0 Q0 Q0
```

#### Tabla de Métricas

| Process | T_arrival | T_firstrun | T_completion | T_turnaround | T_ready |
|---------|-----------|-----------|-------------|--------------|---------|
| **P0**  | 0         | 0         | 25         | 25          | 0       |
| **P1**  | 1         | 1         | 26         | 25          | 0       |
| **P2**  | 2         | 2         | 34         | 32          | 0       |
| **P3**  | 3         | 3         | 26         | 23          | 0       |
| **Promedio** | 1.5   | 1.5       | 27.75      | **26.25**    | **0.0** |

#### Análisis MLFQ

**Ventajas:**
- Excelente turnaround time (26.25 μs) combinado con buen response time
- Adaptation automática: procesos cortos obtienen alta prioridad
- No requiere conocer los tiempos de ráfaga
- Equidad: todos obtienen oportunidades

**Características observadas:**
- P1 y P3 (procesos cortos) completan rápidamente
- P2 (proceso largo) es desplazado a Q0 pero eventualmente completa
- El quantum variable por cola ($1, 2, 4$ μs) es optimal para este patrón
- Tiempos de ready = 0 para todos (excelente interactividad)

---

## Instrucciones para Usar los Scripts

### scheduler.py

**Ubicación:** [GitHub OSTEP Homework](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched)

**Uso en Windows o Linux:**

```bash
# 1. Descargar el script
# Opción A: Clonar todo el repositorio
git clone https://github.com/remzi-arpacidusseau/ostep-homework.git
cd ostep-homework/cpu-sched

# Opción B: Descargar solo scheduler.py
curl -O https://raw.githubusercontent.com/remzi-arpacidusseau/ostep-homework/master/cpu-sched/scheduler.py

# 2. Ejecutar el programa (sin respuestas)
python scheduler.py -p FCFS -l 135,102,56,148,125

# 3. Ver respuestas
python scheduler.py -p FCFS -l 135,102,56,148,125 -c

# 4. Probar otras políticas
python scheduler.py -p SJF -l 135,102,56,148,125 -c
python scheduler.py -p RR -l 135,102,56,148,125 -q 40 -c
```

**Parámetros principales:**
- `-p` : Política (FCFS, SJF, RR)
- `-l` : Lista de tiempos de ráfaga (comma-separated)
- `-q` : Quantum para Round Robin
- `-c` : Mostrar respuestas calculadas
- `-h` : Ayuda completa

### mlfq.py

**Ubicación:** [GitHub OSTEP Homework](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched-mlfq)

**Uso en Windows o Linux:**

```bash
# 1. Descargar el script
cd ../cpu-sched-mlfq
curl -O https://raw.githubusercontent.com/remzi-arpacidusseau/ostep-homework/master/cpu-sched-mlfq/mlfq.py

# 2. Ejecutar el simulador (sin respuestas)
python mlfq.py -l 0,8,0:1,4,0:2,9,0:3,5,0 -Q 1,2,4 -A 1,2,4

# 3. Ver respuestas
python mlfq.py -l 0,8,0:1,4,0:2,9,0:3,5,0 -Q 1,2,4 -A 1,2,4 -c
```

**Formato del job list (-l):** `startTime,runTime,ioFreq:startTime2,runTime2,ioFreq2:...`

**Parámetros principales:**
- `-l` : Lista de jobs en formato especificado
- `-Q` : Quantum para cada cola (comma-separated)
- `-A` : Allotment para cada cola (en time slices)
- `-c` : Mostrar respuestas
- `-h` : Ayuda completa

---

## Conclusiones Generales del Taller

1. **No existe política perfecta:** Cada una optimiza una métrica diferente
2. **Trade-offs:** Mejorar response time requiere afectar turnaround time
3. **MLFQ es práctica:** Combina lo mejor de múltiples políticas
4. **Adaptación es clave:** Los planificadores modernos se adaptan al comportamiento
5. **Métricas importan:** Elegir la métrica correcta es fundamental para el diseño

---

## Referencias

- **Libro:** Operating Systems: Three Easy Pieces (Arpaci-Dusseau)
  - Capítulo 7: Scheduling
  - Capítulo 8: Multi-Level Feedback Queue

- **Código:** [OSTEP Homework - cpu-sched](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched)

- **Diapositivas del curso:** Planificación de Procesos (Danny Múnera, 2024)
