# ✈ EcoVuelo — Sistema de Rutas Aéreas

Modelado de una red de vuelos entre 6 ciudades de la Sierra ecuatoriana
usando un **grafo no dirigido y ponderado**, con una interfaz de escritorio
en **Tkinter** que dibuja y actualiza el mapa en vivo.

**Unidad 4 — Grafos** · Estructura de Datos · 2026-1S
**Autor:** Freddy Erazo — Ingeniería en Ciencia de Datos e Inteligencia Artificial

---

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| [`Erazo_Freddy_Unidad4_Grafos.ipynb`](Erazo_Freddy_Unidad4_Grafos.ipynb) | Notebook con el código completo (clase de grafo + GUI) |
| [`Erazo_Freddy_Unidad4_Grafos.pdf`](Erazo_Freddy_Unidad4_Grafos.pdf) | Exportación en PDF del notebook ejecutado |
| [`Explicacion_Codigo_GrafoEcoVuelo.md`](Explicacion_Codigo_GrafoEcoVuelo.md) | Explicación detallada, módulo por módulo, del código |

## Cómo ejecutarlo

```bash
pip install numpy matplotlib
jupyter notebook Erazo_Freddy_Unidad4_Grafos.ipynb
```

Ejecuta las celdas en orden. La última celda abre la ventana de escritorio
(`tkinter`, incluido con Python); al cerrarla, el control vuelve al notebook.

---

## 1. Idea general

El proyecto modela una red de vuelos entre 6 ciudades de la Sierra ecuatoriana
(Quito, Latacunga, Ambato, Riobamba, Guaranda, Ibarra) usando un **grafo no
dirigido y ponderado**:

- **No dirigido** → si hay ruta Quito–Ambato, también hay Ambato–Quito.
- **Ponderado** → cada arista guarda la distancia en km entre dos ciudades.

El código está dividido en tres módulos (tres celdas principales del
notebook):

| Módulo | Contenido |
|---|---|
| 1 | Clase `GrafoEcoVuelo`: la estructura de datos y sus operaciones básicas |
| 2 | Creación del grafo inicial con las 6 ciudades y sus rutas reales |
| 3 | Interfaz gráfica de escritorio (Tkinter) que usa la clase del Módulo 1 |

---

## 2. Módulo 1 — Clase `GrafoEcoVuelo`

### Representación interna

```python
self.vertices = {}   # { ciudad: { vecino: km } }
```

El grafo se representa como una **lista de adyacencia** implementada con un
diccionario de diccionarios: cada ciudad (vértice) apunta a otro diccionario
donde las claves son las ciudades vecinas y los valores son el peso de la
arista (la distancia en km). Esta estructura permite:

- Verificar si dos ciudades están conectadas en O(1).
- Consultar el peso de una arista en O(1).
- Recorrer los vecinos de una ciudad iterando directamente sobre su
  diccionario.

### Métodos de la clase

| # | Método | Operación de grafo | Qué hace |
|---|---|---|---|
| 1 | `anadir_ciudad(ciudad)` | Agregar vértice | Crea la entrada `ciudad: {}` si no existe |
| 2 | `eliminar_ciudad(ciudad)` | Eliminar vértice | Borra el vértice y recorre todos los demás para quitar las aristas que apuntaban a él |
| 3 | `anadir_ruta(origen, destino, km)` | Agregar arista ponderada | Escribe la arista **en ambos sentidos** (`vertices[origen][destino]` y `vertices[destino][origen]`), reflejando que el grafo no es dirigido |
| 4 | `eliminar_ruta(origen, destino)` | Eliminar arista | Borra la entrada en ambos diccionarios |
| 5 | `buscar_ruta(origen, destino)` | Verificar arista | Comprueba existencia de conexión directa |
| 6 | `buscar_ciudad(ciudad)` | Verificar vértice | Comprueba si el vértice existe |
| 7 | `destinos_directos(ciudad)` | Obtener adyacentes | Lista las claves del diccionario de esa ciudad (sus vecinos) |
| 8 | `consultar_distancia(origen, destino)` | Obtener peso | Devuelve el valor almacenado en la arista |
| 9 | `calcular_conectividad(ciudad)` | Grado del vértice | `len(self.vertices[ciudad])`, es decir cuántas rutas directas tiene |
| 10a | `bfs(inicio)` | Recorrido en anchura | Recorrido por niveles usando una **cola** (lista con `pop(0)`) |
| 10b | `dfs(inicio)` | Recorrido en profundidad | Recorrido recursivo (`_dfs` es el helper privado) que se hunde por una rama antes de retroceder |
| — | `mostrar_red()` | — | Imprime todo el diccionario de adyacencia de forma legible |

### BFS explicado

```python
cola = [inicio]
vistos = {inicio}
while cola:
    actual = cola.pop(0)      # saca el primero (FIFO)
    visitados.append(actual)
    for vecino in self.vertices[actual]:
        if vecino not in vistos:
            vistos.add(vecino)
            cola.append(vecino)
```

Usa el patrón clásico de BFS: una cola FIFO y un conjunto `vistos` para no
procesar el mismo nodo dos veces. Visita primero todos los vecinos directos
de `inicio`, luego los vecinos de esos vecinos, y así por niveles.

### DFS explicado

```python
def _dfs(self, nodo, vistos, visitados):
    vistos.add(nodo)
    visitados.append(nodo)
    for vecino in self.vertices[nodo]:
        if vecino not in vistos:
            self._dfs(vecino, vistos, visitados)   # recursión = pila implícita
```

En vez de una cola, DFS usa la **pila de llamadas de la recursión**: entra en
un vecino, y desde ahí sigue entrando en el siguiente vecino no visitado
antes de volver a los hermanos del nivel anterior.

---

## 3. Módulo 2 — Datos iniciales

Crea el objeto `grafo = GrafoEcoVuelo()`, agrega las 6 ciudades y luego 6
rutas con distancias reales en km:

```
Quito – Latacunga   89 km
Quito – Ibarra      115 km
Latacunga – Ambato   47 km
Ambato – Riobamba    51 km
Riobamba – Guaranda  65 km
Guaranda – Ambato    72 km
```

Con esto el grafo queda conectado (se puede llegar de cualquier ciudad a
cualquier otra), aunque no todas las ciudades tienen vuelo **directo** entre
sí (por ejemplo Ibarra solo conecta con Quito).

---

## 4. Módulo 3 — Interfaz de escritorio (Tkinter)

Esta es la parte más extensa del código: una ventana `tkinter` independiente
que envuelve la clase `GrafoEcoVuelo` del Módulo 1 con controles visuales y
un mapa dibujado con `matplotlib`.

### 4.1 Estructura de la ventana

```
┌─────────────────────────────┬───────────────────────────┐
│ Panel izquierdo (col_izq)   │ Panel derecho (col_der)   │
│  - Encabezado                │  - Mapa del grafo         │
│  - Pestañas (Notebook):      │    (matplotlib embebido   │
│      🏙 Ciudades             │     con FigureCanvasTkAgg)│
│      ✈ Rutas                 │                           │
│      🔍 Recorridos           │                           │
│  - Cuadro de "Resultado"     │                           │
└─────────────────────────────┴───────────────────────────┘
```

La clase que arma todo esto es `VentanaEcoVuelo`. Su `__init__` construye el
layout, y `iniciar()` llama a `self.root.mainloop()` para abrir la ventana.

### 4.2 Posicionamiento de las ciudades en el mapa

`_POS_BASE` fija coordenadas (x, y) normalizadas entre 0 y 1 para las 6
ciudades originales, aproximando su ubicación geográfica real.

El método `_posicion(ciudad)`:

- Si la ciudad ya tiene posición guardada en `_pos_cache`, la reutiliza (para
  que no "salte" de lugar cada vez que se redibuja el mapa).
- Si es una ciudad **nueva**, la coloca en el **centroide** (promedio de
  coordenadas) de las ciudades con las que se conecta, con un pequeño
  desplazamiento aleatorio para que no quede exactamente sobre una línea.
- Si no tiene ninguna conexión todavía, la ubica en un punto aleatorio del
  lienzo.

### 4.3 Dibujo del grafo — `_dibujar()`

Se ejecuta cada vez que cambia algo (agregar/eliminar ciudad o ruta, elegir
origen/destino, ejecutar BFS/DFS). Hace lo siguiente:

1. Limpia el eje y calcula la posición de cada ciudad.
2. Dibuja las **aristas** (líneas) con su etiqueta de distancia en km. El
   color y grosor de cada línea cambian si esa arista forma parte de un
   camino resaltado (`_h_camino`) o es la conexión origen–destino
   seleccionada.
3. Dibuja los **nodos** (círculos) con la inicial de la ciudad y su nombre al
   lado. El color del nodo indica su rol actual: azul por defecto, verde si
   es el origen resaltado, naranja si es el destino, o un degradado si forma
   parte de un recorrido BFS/DFS (más oscuro cuanto más lejos en el
   recorrido).

Es decir, el mapa es una representación visual en vivo del mismo diccionario
`self.vertices` de la clase `GrafoEcoVuelo`.

### 4.4 Dijkstra — `_ruta_mas_corta(origen, destino)`

Cuando el usuario busca una ruta o distancia entre dos ciudades que **no**
tienen vuelo directo, la interfaz calcula el camino más corto con el
**algoritmo de Dijkstra**, usando un heap (`heapq`) como cola de prioridad:

```python
cola = [(0, origen)]
while cola:
    d, actual = heapq.heappop(cola)      # siempre saca la distancia menor conocida
    ...
    for vecino, km in self.grafo.vertices[actual].items():
        nueva_d = d + km
        if nueva_d < distancias.get(vecino, infinito):
            distancias[vecino] = nueva_d
            previos[vecino] = actual      # para reconstruir el camino
            heapq.heappush(cola, (nueva_d, vecino))
```

Al final reconstruye el camino recorriendo el diccionario `previos` desde el
destino hacia el origen y lo invierte. Esto es lo que permite, por ejemplo,
calcular la ruta más corta entre Ibarra y Guaranda aunque no exista un vuelo
directo entre ellas.

### 4.5 Las tres pestañas

**🏙 Ciudades** (`_armar_tab_ciudades`)
- Añadir, eliminar, buscar ciudad; ver su conectividad (grado) y sus destinos
  directos.
- Al añadir una ciudad nueva se abre un diálogo (`_pedir_conexiones`) que
  obliga a indicar con qué ciudad(es) existentes se conecta y a qué
  distancia, para que el grafo nunca quede con un vértice aislado (excepto la
  primera ciudad del sistema).

**✈ Rutas** (`_armar_tab_rutas`)
- Combos de Origen/Destino: al elegirlos se resaltan en el mapa en vivo
  (`_on_seleccion_od`).
- Botones para añadir/actualizar o eliminar una ruta directa.
- "Buscar ruta" y "Distancia": si existe vuelo directo lo informan; si no,
  usan Dijkstra (`_ruta_mas_corta`) para proponer la mejor ruta indirecta y
  resaltarla en el mapa.

**🔍 Recorridos** (`_armar_tab_recorridos`)
- Ejecuta BFS o DFS desde la ciudad elegida (misma lógica que los métodos
  `bfs`/`dfs` de la clase, reimplementada aquí para además ir resaltando el
  camino recorrido sobre el mapa) y muestra el orden de visita.

### 4.6 Flujo de datos

Todo el módulo 3 opera sobre el **mismo objeto `grafo`** creado en el Módulo
2. La GUI no duplica datos: cada botón llama a los métodos de
`GrafoEcoVuelo` (o a una lógica equivalente para Dijkstra/BFS/DFS con
resaltado) y luego llama a `self._dibujar()` para reflejar el cambio en el
mapa.

---

## 5. Resumen de conceptos de grafos aplicados

- **Grafo no dirigido ponderado** representado como lista de adyacencia.
- **Operaciones básicas**: insertar/eliminar vértice, insertar/eliminar
  arista, consultar adyacencia y peso, grado de un vértice.
- **Recorridos**: BFS (cola) y DFS (recursión/pila).
- **Camino más corto con pesos**: algoritmo de Dijkstra con cola de
  prioridad (`heapq`).
- **Visualización**: capa adicional (no es teoría de grafos en sí) que
  traduce el estado del grafo a un dibujo interactivo con `matplotlib` +
  `tkinter`.
