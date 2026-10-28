# Unidad 4 — Práctica guiada: Árboles y Grafos

**Materia:** Estructura de Datos (AED-1026)
**Unidad:** 4. Estructuras no lineales
**Temas:** 4.1 Árboles (clasificación, operaciones básicas sobre árboles binarios, aplicaciones) · 4.2 Grafos (representación, operaciones básicas)
**Lenguaje:** Python 3 — todo el código de este documento se ejecutó de verdad antes de publicarse

Hasta la Unidad 3 viste estructuras **lineales**: cada elemento tiene un predecesor y un sucesor claro (pila, cola, lista). En esta unidad entras a las estructuras **no lineales**: un elemento puede conectarse con varios otros a la vez. Dos ejemplos cubren casi todo lo que necesitas: el **árbol** (conexiones jerárquicas, sin ciclos) y el **grafo** (conexiones libres, con o sin ciclos).

## Antes de empezar: dos imágenes mentales

**Un árbol es como el organigrama de una empresa.** Hay un director general (la raíz), que tiene gerentes a su cargo (hijos), que a su vez tienen empleados a su cargo (nietos). Nadie le reporta a dos jefes a la vez, y no hay forma de que un empleado termine siendo, por una cadena de mandos, jefe de su propio jefe (eso sería un ciclo — y en un árbol **no existen los ciclos**).

**Un grafo es como un mapa de carreteras.** Durango se conecta con Torreón y con Mazatlán. Torreón se conecta con Saltillo. Pero también podrías ir de Saltillo de regreso a Torreón, y de ahí otra vez a Durango — nada prohíbe "dar la vuelta". Un mapa no tiene una única ciudad "de arriba": todas las ciudades están al mismo nivel, conectadas entre sí como les convenga a los ingenieros que trazaron las carreteras.

Esa es la diferencia completa entre los dos: el árbol tiene una jerarquía estricta y nunca regresas sobre tus pasos; el grafo no tiene jerarquía y sí puedes regresar.

```
ÁRBOL (organigrama)                 GRAFO (mapa de carreteras)

        Director                     Durango ---- Torreon
        /      \                        \            |
    Gerente   Gerente                    \           |
     /   \       \                     Mazatlan   Saltillo
 Emp.   Emp.    Emp.                       \          /
                                            Culiacan--Monterrey

Un solo camino del Director       Puedes ir y venir entre ciudades,
a cualquier empleado.             e incluso formar un circuito cerrado
Nunca hay vuelta atrás.           (Durango-Torreon-Saltillo-...-Durango).
```

---

## Parte 1 — Árboles binarios

### 1.1 Elementos de un árbol

Antes de clasificar árboles o programarlos, estos son los nombres que vas a usar todo el tiempo — tanto en este documento como en el examen:

- **Nodo**: cada elemento del árbol. Guarda un valor y, en un árbol binario, hasta dos referencias (a su hijo izquierdo y a su hijo derecho).
- **Raíz**: el único nodo que no tiene padre — el punto de entrada a todo el árbol (en el organigrama, el director general).
- **Padre**: el nodo inmediatamente arriba de otro. Cada nodo tiene exactamente un padre, excepto la raíz, que no tiene ninguno.
- **Hijo**: un nodo inmediatamente abajo de otro. En un árbol binario, cada nodo tiene como máximo 2 hijos (izquierdo y derecho).
- **Hermanos**: dos nodos que comparten el mismo padre.
- **Hoja**: un nodo sin hijos — el final de una rama.
- **Nodo interno**: cualquier nodo que no es hoja (tiene al menos un hijo).
- **Arista** (o rama): la conexión entre un padre y un hijo.
- **Subárbol**: cualquier nodo del árbol, junto con todos sus descendientes, visto como un árbol completo por sí mismo. Por eso la definición recursiva de árbol funciona: "un árbol es un nodo con dos subárboles (izquierdo y derecho), cada uno a su vez un árbol".
- **Nivel** (o profundidad): cuántas aristas hay que recorrer desde la raíz hasta ese nodo. La raíz está en el nivel 0.
- **Altura**: el nivel más profundo que alcanza el árbol — es decir, la distancia (en aristas) de la raíz a la hoja más lejana. Un árbol con un solo nodo tiene altura 0; un árbol vacío, por convención, altura -1 (así lo calcula la función `altura()` que vas a programar en la sección 1.3).
- **Grado de un nodo**: cuántos hijos tiene. En un árbol binario, el grado de cualquier nodo es 0, 1 o 2.
- **Ancestro / descendiente**: todo nodo en el camino de la raíz hasta un nodo dado es su ancestro; todo nodo "debajo" de uno dado es su descendiente.

```
                 Director           ← Raíz (nivel 0)
                /        \
          Gerente       Gerente     ← nivel 1, hijos de Director, hermanos entre sí
          /     \            \
       Emp.    Emp.          Emp.   ← nivel 2, hojas (sin hijos)
```

En este dibujo: "Director" es la raíz y tiene grado 2; los dos "Gerente" son hijos de Director y hermanos entre sí; las tres "Emp." son hojas; la altura del árbol es 2 (hay que bajar 2 aristas desde la raíz hasta la hoja más lejana); el subárbol del "Gerente" de la derecha es él mismo junto con su único "Emp." — un árbol de 2 nodos dentro del árbol grande.

### 1.2 Por qué un árbol y no una lista

Si guardas datos ordenados en una lista y buscas un valor, en el peor caso revisas **todos** los elementos (`O(n)`). Un árbol binario de búsqueda bien construido reduce eso a `O(log n)`: en cada paso descartas la mitad de lo que queda, igual que buscar una palabra en un diccionario físico.

### 1.3 Clasificación de árboles (4.1)

- **Árbol binario**: cada nodo tiene como máximo 2 hijos (izquierdo y derecho).
- **Árbol binario de búsqueda (ABB / BST)**: además, todo valor a la izquierda de un nodo es menor, y todo valor a la derecha es mayor o igual. Esto es lo que habilita la búsqueda rápida.
- **Árbol balanceado**: la altura del lado izquierdo y derecho de cada nodo difiere poco (idealmente ≤1). Garantiza que la búsqueda realmente sea `O(log n)`.
- **Árbol degenerado**: cada nodo tiene un solo hijo — en la práctica es una lista ligada disfrazada de árbol, y pierdes toda la ventaja de velocidad (lo vas a comprobar abajo con números reales).

### 1.4 Operaciones básicas sobre árboles binarios (4.1)

Construimos un Árbol Binario de Búsqueda con las operaciones fundamentales: **insertar**, **buscar**, y los tres **recorridos** clásicos (inorden, preorden, postorden).

```python
class Nodo:
    def __init__(self, valor):
        self.valor = valor
        self.izq = None
        self.der = None

class ArbolBinarioBusqueda:
    def __init__(self):
        self.raiz = None

    def insertar(self, valor):
        if self.raiz is None:
            self.raiz = Nodo(valor)
            return
        actual = self.raiz
        while True:
            if valor < actual.valor:
                if actual.izq is None:
                    actual.izq = Nodo(valor)
                    return
                actual = actual.izq
            else:
                if actual.der is None:
                    actual.der = Nodo(valor)
                    return
                actual = actual.der

    def buscar(self, valor):
        actual = self.raiz
        pasos = 0
        while actual is not None:
            pasos += 1
            if valor == actual.valor:
                return True, pasos
            actual = actual.izq if valor < actual.valor else actual.der
        return False, pasos

    def inorden(self, nodo=None, resultado=None):
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            self.inorden(nodo.izq, resultado)
            resultado.append(nodo.valor)
            self.inorden(nodo.der, resultado)
        return resultado

    def preorden(self, nodo=None, resultado=None):
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            resultado.append(nodo.valor)
            self.preorden(nodo.izq, resultado)
            self.preorden(nodo.der, resultado)
        return resultado

    def postorden(self, nodo=None, resultado=None):
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            self.postorden(nodo.izq, resultado)
            self.postorden(nodo.der, resultado)
            resultado.append(nodo.valor)
        return resultado

    def altura(self, nodo="__raiz__"):
        if nodo == "__raiz__":
            nodo = self.raiz
        if nodo is None:
            return -1
        return 1 + max(self.altura(nodo.izq), self.altura(nodo.der))

    def nivel_por_nivel(self):
        if self.raiz is None:
            return []
        niveles = []
        cola = [self.raiz]
        while cola:
            nivel = []
            siguiente = []
            for n in cola:
                nivel.append(n.valor)
                if n.izq: siguiente.append(n.izq)
                if n.der: siguiente.append(n.der)
            niveles.append(nivel)
            cola = siguiente
        return niveles
```

**Qué hace cada recorrido** (para que lo expliques en clase sin tener que adivinarlo viendo el código):

- **Inorden** (izquierda → nodo → derecha): en un ABB, **siempre** entrega los valores ordenados de menor a mayor. Es la prueba de que el árbol está bien construido.
- **Preorden** (nodo → izquierda → derecha): entrega primero la raíz. Se usa para **copiar/serializar** un árbol (reconstruir el mismo árbol desde la lista preorden).
- **Postorden** (izquierda → derecha → nodo): entrega la raíz al final. Se usa cuando necesitas procesar los hijos antes que el padre, por ejemplo **borrar un árbol de memoria** o evaluar una expresión matemática representada como árbol.

### 1.5 Ejecución real: insertar, recorrer, y comparar un árbol balanceado contra uno degenerado

```python
valores = [50, 30, 70, 20, 40, 60, 80, 10]
arbol = ArbolBinarioBusqueda()
for v in valores:
    arbol.insertar(v)

print("Insertados:", valores)
print("Inorden (ordenado):", arbol.inorden())
print("Preorden:", arbol.preorden())
print("Postorden:", arbol.postorden())
print("Altura:", arbol.altura())
print("Por niveles:", arbol.nivel_por_nivel())
print("Buscar 60:", arbol.buscar(60))
print("Buscar 65:", arbol.buscar(65))
```

**Salida real:**

```
Insertados: [50, 30, 70, 20, 40, 60, 80, 10]
Inorden (ordenado): [10, 20, 30, 40, 50, 60, 70, 80]
Preorden: [50, 30, 20, 10, 40, 70, 60, 80]
Postorden: [10, 20, 40, 30, 60, 80, 70, 50]
Altura: 3
Por niveles: [[50], [30, 70], [20, 40, 60, 80], [10]]
Buscar 60: (True, 3)
Buscar 65: (False, 3)
```

Así quedó el árbol en memoria después de las 8 inserciones — es exactamente lo que dice `nivel_por_nivel()`:

```
                    50
                  /    \
               30        70
              /  \      /  \
            20    40  60    80
            /
          10
```

Para ubicar visualmente cómo trabaja `buscar(60)`: empieza en 50 (60 > 50, va a la derecha), llega a 70 (60 < 70, va a la izquierda), llega a 60 (coincide) — 3 pasos, siguiendo siempre **una sola rama** hacia abajo, nunca revisa el lado contrario.

Fíjate: **inorden entregó los 8 valores perfectamente ordenados**, sin que nadie los ordenara explícitamente — es una propiedad automática del ABB. Y buscar el 60 tomó solo 3 pasos de 8 posibles.

Ahora el experimento que justifica por qué "balanceado" no es un detalle cosmético — insertamos los **mismos 8 valores**, pero ya ordenados de entrada:

```python
degenerado = ArbolBinarioBusqueda()
for v in sorted(valores):
    degenerado.insertar(v)

print("Altura degenerado:", degenerado.altura(), "vs balanceado:", arbol.altura())
print("Buscar 80 en degenerado:", degenerado.buscar(80))
print("Buscar 80 en balanceado:", arbol.buscar(80))
```

**Salida real:**

```
Altura degenerado: 7 vs balanceado: 3
Buscar 80 en degenerado: (True, 8)
Buscar 80 en balanceado: (True, 3)
```

Mismos 8 valores, mismo contenido — pero buscar el 80 tomó **8 pasos en el degenerado contra 3 en el balanceado**. El degenerado se convirtió en una lista ligada: insertar datos ya ordenados en un ABB simple (sin rebalanceo automático) es el peor caso posible. Por eso existen variantes como AVL o árboles rojo-negro, que rebalancean solos — quedan fuera del alcance de esta materia, pero ahora entiendes **por qué existen**.

Así se ve la diferencia lado a lado — el mismo contenido, dos formas radicalmente distintas:

```
BALANCEADO (altura 3)                DEGENERADO (altura 7)

        50                            10
      /    \                            \
   30        70                          20
  /  \      /  \                           \
20    40  60    80                          30
/                                              \
10                                              40
                                                   \
                                                    50
                                                       \
                                                        60
                                                           \
                                                            70
                                                               \
                                                                80

3 saltos como máximo               Hasta 8 saltos: cada nodo tiene
entre la raíz y cualquier hoja.    un solo hijo — es una lista ligada
                                   con forma de árbol.
```

### 1.6 Aplicaciones de árboles (4.1)

- **Sistemas de archivos**: carpetas y subcarpetas son, literalmente, un árbol (ya lo viste de forma real en la Unidad 5 de Sistemas Operativos, con inodos).
- **Árboles de decisión**: cada nodo es una pregunta, cada rama una respuesta — base de algoritmos de clasificación en Machine Learning.
- **Índices de bases de datos** (B-Tree / B+Tree): PostgreSQL y MySQL usan variantes de árboles balanceados para que una consulta `WHERE id = X` no tenga que revisar la tabla completa.
- **Árboles de expresiones**: una expresión como `(3 + 5) * 2` se representa como árbol, y el **postorden** la evalúa directamente.
- **Organigramas**: ya construiste uno en la Unidad 2 (recursividad) — un organigrama *es* un árbol general (no binario, porque un jefe puede tener más de 2 subordinados).

---

## Parte 2 — Grafos

### 2.1 Por qué un grafo y no un árbol

Un árbol no permite ciclos ni que un nodo tenga más de un "padre". Pero el mundo real sí tiene eso: una ciudad conectada por carretera a varias otras, cuentas de usuario que se siguen mutuamente, páginas web que se enlazan entre sí formando ciclos. Para eso existe el **grafo**: un conjunto de **vértices** (nodos) conectados por **aristas** (conexiones), sin la restricción jerárquica del árbol.

### 2.2 Representación de grafos (4.2)

Hay dos formas estándar de representar un grafo en memoria:

- **Matriz de adyacencia**: una tabla de `n × n` donde la celda `[i][j]` indica si hay conexión entre el vértice `i` y el `j`. Simple, pero desperdicia memoria si el grafo tiene pocas conexiones (`O(n²)` siempre).
- **Lista de adyacencia**: cada vértice guarda solo la lista de sus vecinos directos. Es la representación que usa casi todo software real (redes sociales, mapas, rutas) porque solo gasta memoria proporcional a las conexiones que realmente existen.

Implementamos con **lista de adyacencia**, la opción que vas a encontrar en código real:

```python
from collections import deque

class Grafo:
    def __init__(self, dirigido=False):
        self.dirigido = dirigido
        self.lista_adyacencia = {}

    def agregar_vertice(self, v):
        self.lista_adyacencia.setdefault(v, [])

    def agregar_arista(self, u, v, peso=1):
        self.agregar_vertice(u)
        self.agregar_vertice(v)
        self.lista_adyacencia[u].append((v, peso))
        if not self.dirigido:
            self.lista_adyacencia[v].append((u, peso))
```

- **Grafo no dirigido**: la conexión va en ambos sentidos (una carretera se recorre en los dos sentidos).
- **Grafo dirigido**: la conexión tiene un solo sentido (seguir a alguien en redes sociales no implica que te sigan de vuelta).
- **Peso**: un valor asociado a la arista (distancia en km, costo, tiempo). Un grafo sin peso asume que todas las conexiones "cuestan" lo mismo.

### Antes de ver el algoritmo: qué es un "heap" y por qué Dijkstra lo necesita

Dijkstra, abajo, usa `import heapq`. Un **heap** (montículo) es una estructura que mantiene siempre accesible, en `O(1)`, **el elemento más pequeño** que contiene, sin tener que ordenar toda la colección cada vez que agregas algo nuevo. `heapq.heappush` mete un elemento manteniendo esa propiedad; `heapq.heappop` saca siempre el más pequeño.

¿Para qué le sirve a Dijkstra? El algoritmo necesita, en cada paso, visitar la ciudad **más cercana que todavía no ha explorado** — sin un heap tendrías que revisar manualmente todas las distancias pendientes cada vez (lento); con un heap, pedirlo es instantáneo. Es la misma idea que una cola, pero en vez de "el que llegó primero" (FIFO), es "el que tiene el valor más chico" — por eso a veces se le llama **cola de prioridad**.

### 2.3 Operaciones básicas sobre grafos (4.2): BFS, DFS, camino más corto

```python
    def bfs(self, inicio):
        visitados = {inicio}
        orden = []
        cola = deque([inicio])
        while cola:
            actual = cola.popleft()
            orden.append(actual)
            for vecino, _ in self.lista_adyacencia[actual]:
                if vecino not in visitados:
                    visitados.add(vecino)
                    cola.append(vecino)
        return orden

    def dfs(self, inicio):
        visitados = set()
        orden = []
        def recorrer(v):
            visitados.add(v)
            orden.append(v)
            for vecino, _ in self.lista_adyacencia[v]:
                if vecino not in visitados:
                    recorrer(vecino)
        recorrer(inicio)
        return orden

    def hay_camino(self, origen, destino):
        return destino in self.bfs(origen)

    def dijkstra(self, inicio):
        import heapq
        distancias = {v: float('inf') for v in self.lista_adyacencia}
        distancias[inicio] = 0
        heap = [(0, inicio)]
        while heap:
            dist_actual, actual = heapq.heappop(heap)
            if dist_actual > distancias[actual]:
                continue
            for vecino, peso in self.lista_adyacencia[actual]:
                nueva = dist_actual + peso
                if nueva < distancias[vecino]:
                    distancias[vecino] = nueva
                    heapq.heappush(heap, (nueva, vecino))
        return distancias
```

- **BFS (recorrido por anchura)**: usa una **cola** (justo la estructura de la Unidad 3) — explora nivel por nivel. Responde "¿cuál es el camino con **menos saltos**?".
- **DFS (recorrido por profundidad)**: usa recursión, que internamente es una **pila** (otra estructura de la Unidad 3) — se mete lo más hondo posible antes de retroceder. Responde "¿existe algún camino?" y se usa para detectar ciclos.
- **Dijkstra**: responde "¿cuál es el camino de **menor costo total**?" cuando las aristas tienen peso — no es lo mismo que "menos saltos": un camino con 2 saltos puede costar más que uno con 4 saltos si las distancias son grandes.

### 2.4 Ejecución real: una red de ciudades (grafo no dirigido con peso)

Así se ve la red que vamos a construir (el número en cada conexión es la distancia en km):

```
          Durango
          /     \
       265        310
        /           \
   Torreon       Mazatlan
      |              |
     270            215
      |              |
  Saltillo       Culiacan
      |              |
      85            900
      |              |
       \            /
        Monterrey
```

```python
red = Grafo(dirigido=False)
red.agregar_arista("Durango", "Torreon", 265)
red.agregar_arista("Durango", "Mazatlan", 310)
red.agregar_arista("Torreon", "Saltillo", 270)
red.agregar_arista("Mazatlan", "Culiacan", 215)
red.agregar_arista("Saltillo", "Monterrey", 85)
red.agregar_arista("Culiacan", "Monterrey", 900)

print("BFS desde Durango:", red.bfs("Durango"))
print("DFS desde Durango:", red.dfs("Durango"))
print("¿Hay camino Durango -> Monterrey?:", red.hay_camino("Durango", "Monterrey"))

distancias = red.dijkstra("Durango")
for ciudad, km in distancias.items():
    print(f"  Durango -> {ciudad}: {km} km")
```

**Salida real:**

```
BFS desde Durango: ['Durango', 'Torreon', 'Mazatlan', 'Saltillo', 'Culiacan', 'Monterrey']
DFS desde Durango: ['Durango', 'Torreon', 'Saltillo', 'Monterrey', 'Culiacan', 'Mazatlan']
¿Hay camino Durango -> Monterrey?: True

  Durango -> Durango: 0 km
  Durango -> Torreon: 265 km
  Durango -> Mazatlan: 310 km
  Durango -> Saltillo: 535 km
  Durango -> Culiacan: 525 km
  Durango -> Monterrey: 620 km
```

Nota algo importante: BFS y DFS visitan los vértices en **orden distinto** (ambos son válidos, dependen del orden en que se exploran los vecinos), pero Dijkstra te da el dato que de verdad importa en una app de mapas: la ruta Durango→Monterrey por **Saltillo cuesta 620 km**, aunque pasando por Culiacán también "hay camino" (lo confirmó `hay_camino`), ese camino es mucho más largo (525 + 900 = 1425 km). BFS/DFS solo contestan *si existe* un camino; Dijkstra contesta *cuál es el mejor*.

### 2.5 Ejecución real: grafo dirigido (dependencias de despliegue)

Un caso donde la dirección de la arista importa: no puedes desplegar una app antes de instalar Docker.

```python
dependencias = Grafo(dirigido=True)
dependencias.agregar_arista("instalar_ubuntu", "configurar_red")
dependencias.agregar_arista("configurar_red", "instalar_docker")
dependencias.agregar_arista("instalar_docker", "desplegar_app")
dependencias.agregar_arista("crear_usuario", "desplegar_app")

print("DFS desde instalar_ubuntu:", dependencias.dfs("instalar_ubuntu"))
```

**Salida real:**

```
DFS desde instalar_ubuntu: ['instalar_ubuntu', 'configurar_red', 'instalar_docker', 'desplegar_app']
```

El DFS siguió el orden correcto de dependencias. Este patrón (grafo dirigido + recorrido) es exactamente cómo herramientas reales como `make`, Docker Compose o un pipeline de CI deciden en qué orden ejecutar pasos que dependen unos de otros.

### 2.6 Aplicaciones de grafos (4.2)

- **Mapas y GPS**: ciudades/intersecciones = vértices, calles = aristas con peso (distancia o tiempo) → Dijkstra o A*.
- **Redes sociales**: usuarios = vértices, "sigue a" = arista dirigida. BFS calcula "amigos en común" o "grados de separación".
- **Redes de computadoras**: dispositivos = vértices, enlaces = aristas → encontrar la mejor ruta para un paquete de datos.
- **Gestión de dependencias** (paquetes `npm`/`pip`, tareas de un proyecto): grafo dirigido donde un ciclo significa un error de diseño (dependencia circular).
- **Motores de recomendación**: productos/usuarios conectados por "compró junto con" — se recorre el grafo para sugerir productos relacionados.

---

## Comparación rápida: árbol vs. grafo

| | Árbol | Grafo |
|---|---|---|
| Ciclos | No permitidos | Permitidos |
| Padres por nodo | Máximo 1 (excepto la raíz, que no tiene) | Sin límite |
| Ejemplo | Sistema de archivos, organigrama | Mapa de carreteras, red social |
| Operación típica | Recorridos (inorden/pre/post), búsqueda `O(log n)` | BFS/DFS, camino más corto (Dijkstra) |

Dato clave para no confundirse: **todo árbol es un grafo** (un caso particular, sin ciclos y con un único camino entre cualquier par de nodos), pero no todo grafo es un árbol.

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM
- Cormen, T., Leiserson, C., Rivest, R., Stein, C. — *Introduction to Algorithms* (cap. de árboles de búsqueda binaria y algoritmos de grafos)
- docs.python.org/3/library/heapq.html (usado en la implementación de Dijkstra)
