# Simulador de Sistemas Operativos

Aplicación web interactiva, sin dependencias ni instalación, que simula y visualiza algoritmos clásicos de un sistema operativo. Está hecha con HTML, CSS y JavaScript puro y se ejecuta completamente en el navegador.

Incluye dos módulos:

| Módulo | Algoritmo | Qué muestra |
|---|---|---|
| **Gestión de memoria** | Reemplazo de páginas **Óptimo** | Tabla de marcos (frames) paso a paso y total de fallos de página |
| **Gestión de memoria** | Reemplazo de páginas **Clock** (reloj) | Tabla de marcos con bit de referencia, posición donde se reemplazó y total de fallos |
| **Planificación de CPU** | **FCFS** (First Come, First Served) | Diagrama de Gantt, estado de la cola en cada instante, tiempos de retorno y espera |
| **Planificación de CPU** | **SJF** (Shortest Job First, no expropiativo) | Diagrama de Gantt, tabla de resultados y comparativa contra FCFS |

---

## Tabla de contenido

- [Estructura del proyecto](#estructura-del-proyecto)
- [Cómo ejecutarlo](#cómo-ejecutarlo)
- [Guía de uso](#guía-de-uso)
- [Arquitectura y flujo de la aplicación](#arquitectura-y-flujo-de-la-aplicación)
- [Algoritmos de memoria](#algoritmos-de-memoria)
  - [Óptimo](#algoritmo-óptimo)
  - [Clock](#algoritmo-clock)
- [Algoritmos de planificación de CPU](#algoritmos-de-planificación-de-cpu)
  - [FCFS](#fcfs)
  - [SJF](#sjf)
  - [Métricas y convenciones](#métricas-y-convenciones)
- [Ejemplos de resultados](#ejemplos-de-resultados)
- [Interfaz y estilos](#interfaz-y-estilos)
- [Limitaciones conocidas y posibles mejoras](#limitaciones-conocidas-y-posibles-mejoras)

---

## Estructura del proyecto

```
Emulacion/
├── index.html      # Las 4 pantallas (inicio, módulos, memoria, CPU) y el contenedor de resultados
├── CSS/
│   └── styles.css  # Tema oscuro "cyber-minimalist", diapositivas, tarjetas y tablas
└── JS/
    └── app.js      # Navegación, algoritmos, renderizado de tablas y eventos
```

## Cómo ejecutarlo

No requiere servidor ni compilación. Abre `index.html` en un navegador moderno.

> **Importante:** los resultados se abren en una **pestaña nueva**. Si el navegador bloquea las ventanas emergentes, la aplicación mostrará un aviso; permítelas para este sitio y vuelve a pulsar el botón.

Las tipografías (*Urbanist* y *Space Mono*, de Google Fonts) y los iconos (*Font Awesome* 6.5.1, de cdnjs) se cargan desde internet. Sin conexión la aplicación funciona igual, pero con las fuentes del sistema y sin iconos.

---

## Guía de uso

La aplicación funciona como una presentación de 4 pantallas:

```
Inicio  →  Clasificación de sistemas  →  Gestión de memoria
                                      →  Planificación CPU
```

### Pantalla de memoria

Tiene tres parámetros que aplican a las simulaciones **aleatorias**:

| Parámetro | Opciones | Significado |
|---|---|---|
| **Frames** | 4 u 8 | Número de marcos de memoria disponibles |
| **Procesos** | 8 o 16 | Cantidad de páginas distintas que pueden solicitarse |
| **Tiempo** | 10 a 70 | Longitud de la secuencia de solicitudes (se valida al ejecutar) |

Botones:

- **Simulación aleatoria** (Óptimo): genera una secuencia al azar y ejecuta el algoritmo.
- **Aleatorio** (Clock): igual, pero con el algoritmo Clock.
- **Datos Excel** (Clock): usa una secuencia fija de 20 solicitudes con 4 marcos, **ignorando** los parámetros anteriores (el botón muestra un recordatorio al pasar el cursor).

### Pantalla de CPU

- **FCFS → Seleccionar:** genera 10 procesos aleatorios y los simula.
- **SJF → Aleatorio / Ejemplo:** simula 10 procesos aleatorios o un conjunto fijo de ejemplo.

> **Consejo:** con 8 marcos y 8 procesos todas las páginas caben en memoria, así que solo habrá fallos por la primera carga de cada página y ambos algoritmos darán el mismo resultado. Para ver diferencias claras entre ellos, usa **4 marcos y 16 procesos**.

---

## Arquitectura y flujo de la aplicación

```mermaid
flowchart LR
    A["Clic en un botón<br/>(DOMContentLoaded)"] --> B["Algoritmo<br/>(lógica pura)"]
    B --> C["Función de renderizado<br/>genera HTML"]
    C --> D["#modal-cuerpo<br/>(contenedor oculto)"]
    D --> E["abrirModal()"]
    E --> F["Nueva pestaña con<br/>estilos incluidos"]
    E --> G["Limpia #modal-cuerpo"]
```

`app.js` está organizado en bloques con comentarios de sección:

1. **Motor de interfaz:** `navegarA()` y `abrirModal()`.
2. **Módulo 1, paginación:** Óptimo y Clock (lógica y renderizado).
3. **Módulo 2, planificación de CPU:** FCFS y SJF (lógica y renderizado).
4. **Event listeners:** conectan los botones del HTML con la lógica.

### Navegación

`navegarA(id)` quita la clase `active` de todas las `.slide-container` y se la agrega a la pantalla destino. El CSS se encarga de la transición (opacidad, escala y visibilidad).

### Cómo se muestran los resultados (`abrirModal`)

El contenedor `#modal-cuerpo` funciona como un **área de trabajo temporal**:

1. Cada algoritmo genera su HTML (o, en FCFS y SJF, inyecta primero el esqueleto de tablas y luego las rellena por `id`) dentro de `#modal-cuerpo`.
2. `abrirModal(titulo)` lee ese HTML y abre una pestaña nueva con `window.open`. Si el navegador la bloquea (`window.open` devuelve `null`), muestra un aviso al usuario y termina sin error.
3. Escribe con `document.write` un documento completo que **incluye su propio CSS** (el mismo tema), de modo que la pestaña se ve idéntica a la aplicación.
4. Cierra el flujo de escritura y deja `#modal-cuerpo` vacío para la siguiente simulación.

Como el contenido se copia como HTML estático, la pestaña de resultados no es interactiva: es una "fotografía" de la simulación.

---

## Algoritmos de memoria

Ambos reciben el número de marcos, y generan o reciben una **secuencia de referencias** (la página solicitada en cada instante). Un **fallo de página** ocurre cuando la página solicitada no está en ningún marco.

### Algoritmo Óptimo

Función: `simulacionOptimaLogica(numFrames, numProcesos, totalTiempos)`.

Es el algoritmo de **Belady**: ante un fallo sin marcos libres, reemplaza la página **que se volverá a usar más tarde** (o que no se volverá a usar). Requiere conocer el futuro, por eso no se puede implementar en un sistema real y sirve como **referencia teórica** del mejor resultado posible.

Funcionamiento paso a paso:

1. Genera la secuencia: cada solicitud es una página aleatoria entre `1` y `numProcesos`.
2. Los marcos empiezan vacíos (`"-"`).
3. Para cada instante `t`:
   - **Acierto:** si la página ya está en un marco, no hay fallo; solo se copia el estado de los marcos.
   - **Fallo:** se cuenta, se marca con `X` y se coloca la página:
     - si hay un marco vacío, se usa el primero libre;
     - si no, se recorre cada marco buscando **cuándo se usará de nuevo** su página (`referencia.indexOf(pagina, t + 1)`). Si no se usa nunca más, se le asigna `9999` (infinito). Se reemplaza el marco con la distancia mayor. En caso de empate gana el primero.
4. En cada instante se guarda una "fotografía" de todos los marcos.

Devuelve: `referencia`, `filaFallos`, `matrizFrames`, `numFrames`, `totalTiempos` y `totalFallos`.

**Tabla de salida** (`renderizarTablaPaginacion`): fila **Demanda** (página solicitada), fila **Fallo** (`X` en rojo), una fila por marco (`F1`, `F2`...) y fila **Tiempo**. Arriba se muestra el total de fallos.

### Algoritmo Clock

Función: `ejecutarSimulacionReloj(numFrames, numProcesos, duracionTiempo, secuenciaDemanda = null)`.

Es una **aproximación eficiente de LRU**. Los marcos forman una **lista circular** y cada uno guarda un **bit de referencia (uso)**. Un puntero ("manecilla del reloj") indica dónde empezar a buscar víctima.

Cada marco es un par `[pagina, bit]`, inicialmente `[null, 0]`. Si no se pasa una secuencia, se genera una aleatoria con páginas entre `0` y `numProcesos - 1`.

Para cada solicitud:

1. **Acierto:** si la página está en algún marco, su bit pasa a `1`.
2. **Fallo:** se cuenta y:
   - si hay un marco vacío, se carga la página ahí con bit `1`;
   - si no, el puntero recorre los marcos:
     - si el bit es `0` → **se reemplaza** esa página (se carga la nueva con bit `1`) y el puntero avanza al siguiente marco;
     - si el bit es `1` → se le da una "segunda oportunidad": se pone en `0` y se avanza.
   
   El recorrido siempre termina, porque tras una vuelta completa todos los bits quedan en `0`.
3. Se guarda una copia del estado de los marcos y la posición del reemplazo.

**Tabla de salida** (`renderizarTablaClock`): igual que la del Óptimo, pero cada celda muestra `página (bit)`. Un asterisco amarillo `*` marca el marco **donde el puntero realizó el reemplazo** (no aparece cuando se rellena un marco vacío ni en los aciertos).

---

## Algoritmos de planificación de CPU

### FCFS

Funciones: `prepararProcesosAleatorios()`, `simularFCFS()` y varias de renderizado.

Atiende los procesos **en orden de llegada**, sin interrumpirlos (**no expropiativo**). Es simple y justo, pero sufre el **efecto convoy**: un proceso largo retrasa a los cortos que llegan detrás.

**Datos de entrada** (generados al azar):

- 10 procesos, identificados `A` a `J`.
- **Tiempo de ejecución:** entre 2 y 9 unidades.
- **Tiempo de llegada:** el primer proceso llega en `0` y los demás reciben tiempos **únicos** entre 0 y 14. Después se ordenan por llegada.

**Simulación unidad por unidad** (`simularFCFS`), repitiendo hasta que terminen todos:

1. Los procesos cuyo tiempo de llegada coincide con el instante actual entran a la cola.
2. Si la CPU está libre y hay cola, el primero de la cola pasa a ejecutarse y se registra su **tiempo de comienzo**.
3. Se guarda una fotografía de la cola para ese instante.
4. Se registra el estado de cada proceso en el Gantt:
   - `E`: ejecutándose;
   - `L`: listo, esperando en la cola;
   - `F`: justo terminó.
5. Se descuenta una unidad al proceso en ejecución. Si llega a cero, se registra su **tiempo de fin** y se libera la CPU.
6. Avanza el reloj.

**Salidas en pantalla:** tabla de **Entrada** (ID, ejecución, llegada), tabla de **Resultados** (comienzo, fin, retorno, espera) con los promedios, el **diagrama de Gantt** y el **estado de la cola** en cada instante (el proceso al frente de la cola es el que aparece más abajo en cada columna).

### SJF

Funciones: `sjf(input)`, `fcfs(input)` (para comparar), `runSJF()` y renderizado.

Es **SJF no expropiativo**: cuando la CPU queda libre, elige entre los procesos ya llegados el de **menor ráfaga**, y lo ejecuta hasta el final. Minimiza el tiempo de espera promedio, pero puede causar **inanición** en procesos largos si siguen llegando cortos.

**Implementación:**

- Los procesos se ordenan por llegada y se van agregando a una **cola de prioridad** (un *min-heap* binario implementado a mano con `hpush` y `hpop`).
- Criterio de prioridad (`cmp`): primero **menor ráfaga**, luego **menor llegada**, luego **orden alfabético del ID**, para que los empates sean deterministas.
- Bucle principal: agrega los procesos que ya llegaron; si no hay ninguno listo, **salta el reloj** a la siguiente llegada (CPU ociosa); si los hay, extrae el más corto, lo ejecuta de `start` a `end = start + ráfaga` y registra sus métricas.

**Datos de entrada:**

- **Ejemplo:** 10 procesos fijos (`A` a `J`) definidos en `loadDefault()`.
- **Aleatorio:** 10 procesos con ráfagas de 1 a 9 y llegadas crecientes (cada una suma entre 0 y 3 respecto a la anterior).

**Salidas en pantalla:** tabla de entrada, **diagrama de Gantt** (`L` espera, `E` ejecución, `F` instante de finalización), tabla de resultados con promedios y una **comparativa SJF vs FCFS** (la función `fcfs` calcula la planificación FCFS sobre los mismos datos) con la diferencia y cuál resulta mejor en espera y retorno.

### Métricas y convenciones

| Métrica | Fórmula |
|---|---|
| **Tiempo de retorno** | instante de finalización − tiempo de llegada |
| **Tiempo de espera** | tiempo de retorno − tiempo de ejecución (equivale a *inicio − llegada* sin interrupciones) |

> **Convención de la columna "Fin":** en la tabla de **FCFS**, "Fin" es la **última unidad de tiempo en que el proceso se ejecutó**, por lo que el retorno se calcula como `Fin − Llegada + 1`. En la tabla de **SJF**, "Fin" es el **instante en que termina** (`inicio + ráfaga`), y el retorno es `Fin − Llegada`. Ambas convenciones dan el mismo retorno; solo cambia cómo se escribe el valor de "Fin".

---

## Ejemplos de resultados

### Clock con los "Datos Excel"

Secuencia fija de 20 solicitudes con 4 marcos:

```
5 2 3 2 4 1 3 2 5 5 8 9 6 0 2 11 5 9 6 2
```

Resultado: **16 fallos de página**. Los fallos ocurren en los instantes 1, 2, 3, 5, 6, 9 y del 11 al 20.

### SJF con el conjunto de ejemplo

Orden de ejecución: `A [0-5]`, `F [5-8]`, `D [8-13]`, `H [13-16]`, `C [16-18]`, `B [18-22]`, `J [22-29]`, `I [29-37]`, `E [37-45]`, `G [45-55]`. Tiempo total: 55 unidades.

| Métrica | SJF | FCFS | Diferencia | Mejor |
|---|---|---|---|---|
| Espera promedio | 11.60 | 18.60 | 7.00 | SJF |
| Retorno promedio | 17.10 | 24.10 | 7.00 | SJF |

---

## Interfaz y estilos

- **Tema "cyber-minimalist":** fondo casi negro con acentos amarillo (`--primary: #FFEA00`) y cian (`--secondary: #00E5FF`), definidos como variables CSS en `:root`.
- **Diapositivas:** cada pantalla es una `.slide-container` de **1280 × 720 px**, posicionada de forma absoluta y apilada. Solo la que tiene la clase `active` es visible, con una transición de 0.5 s.
- **Escalado automático:** la función `ajustarEscala()` calcula un factor según el tamaño de la ventana y lo guarda en la variable CSS `--escala`, que usan las reglas `transform: scale(...)` de las diapositivas. El factor nunca supera `1`, así que en pantallas de 1366 × 768 o mayores la apariencia es idéntica a la original; en ventanas más pequeñas las diapositivas se reducen proporcionalmente para caber completas. Se recalcula al redimensionar la ventana.
- **Componentes:** tarjetas (`.tile`, `.action-card`), botones (`.btn-ui` y variantes) y tablas (`.tabla-neon`).
- **Gantt:** `E` aparece con fondo cian, `L` en gris y `F` con fondo amarillo.
- **Fallos de página:** se resaltan en rojo (`.fallo-critico`).
- **Aislamiento:** la pestaña de resultados incluye una copia de las variables y estilos de tabla, por lo que no depende de `styles.css`.

---

## Limitaciones conocidas y posibles mejoras

Los algoritmos se verificaron comparándolos con implementaciones independientes sobre miles de entradas aleatorias (Óptimo, Clock, SJF y FCFS), y la aplicación se probó completa en un navegador. Lo siguiente **no son errores de cálculo**: son decisiones de diseño a tener en cuenta y mejoras opcionales.

### Comportamiento a tener en cuenta

- **El botón "Datos Excel" es fijo** (4 marcos, 16 procesos, 20 solicitudes) e ignora los selectores de la pantalla.
- **Convención de "Fin" distinta** entre FCFS y SJF (ver [Métricas y convenciones](#métricas-y-convenciones)).
- **Numeración de páginas distinta:** el Óptimo usa páginas de `1` a `N` y Clock de `0` a `N-1` (la secuencia "Excel" incluye la página `0`). Para unificarlas basta con quitar el `+ 1` de la línea que genera `referencia` en `simulacionOptimaLogica`.
- **Óptimo y Clock no se comparan sobre la misma secuencia:** cada uno genera la suya. Compartir una secuencia y mostrar ambos resultados permitiría una comparativa directa, como la de SJF vs FCFS.
- **Número de procesos de CPU fijo** (10 en ambos módulos), sin opción de configurarlo.
- **Móviles en vertical:** al escalar la diapositiva completa a un ancho de unos 390 px queda muy pequeña (factor ≈ 0.29). Se recomienda pantalla de escritorio o el móvil en horizontal.
