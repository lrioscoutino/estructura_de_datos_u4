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
        self.izq = None   # referencia al hijo izquierdo (o None si no tiene)
        self.der = None   # referencia al hijo derecho (o None si no tiene)

class ArbolBinarioBusqueda:
    def __init__(self):
        self.raiz = None  # árbol vacío: ningún nodo todavía

    def insertar(self, valor):
        # Caso base: árbol vacío → el nuevo valor se vuelve la raíz.
        if self.raiz is None:
            self.raiz = Nodo(valor)
            return
        # Recorremos desde la raíz comparando contra cada nodo: si el valor
        # es menor nos movemos a la izquierda, si no a la derecha, hasta
        # encontrar un hueco (None) donde colgar el nuevo nodo.
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
        # Misma lógica de comparación que insertar(), pero sin crear nodos:
        # en cada paso descartamos la mitad del árbol que no puede contener
        # el valor buscado. Cuenta los pasos para medir qué tan rápido fue.
        actual = self.raiz
        pasos = 0
        while actual is not None:
            pasos += 1
            if valor == actual.valor:
                return True, pasos
            actual = actual.izq if valor < actual.valor else actual.der
        return False, pasos

    def inorden(self, nodo=None, resultado=None):
        # izquierda → nodo → derecha. En un ABB esto siempre entrega los
        # valores ordenados de menor a mayor (ver explicación abajo).
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            self.inorden(nodo.izq, resultado)
            resultado.append(nodo.valor)
            self.inorden(nodo.der, resultado)
        return resultado

    def preorden(self, nodo=None, resultado=None):
        # nodo → izquierda → derecha. Visita primero la raíz, útil para
        # copiar/serializar el árbol (se puede reconstruir desde esta lista).
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            resultado.append(nodo.valor)
            self.preorden(nodo.izq, resultado)
            self.preorden(nodo.der, resultado)
        return resultado

    def postorden(self, nodo=None, resultado=None):
        # izquierda → derecha → nodo. Visita la raíz al final, útil cuando
        # hay que procesar los hijos antes que el padre (borrar, evaluar).
        if resultado is None:
            resultado = []
            nodo = self.raiz
        if nodo:
            self.postorden(nodo.izq, resultado)
            self.postorden(nodo.der, resultado)
            resultado.append(nodo.valor)
        return resultado

    def altura(self, nodo="__raiz__"):
        # Altura = 1 + la altura del hijo más profundo. Árbol vacío = -1,
        # para que un nodo solo (sin hijos) salga en 1 + max(-1,-1) = 0.
        if nodo == "__raiz__":
            nodo = self.raiz
        if nodo is None:
            return -1
        return 1 + max(self.altura(nodo.izq), self.altura(nodo.der))

    def nivel_por_nivel(self):
        # Recorrido por anchura (BFS) sobre el árbol: se procesa un nivel
        # completo (una "cola" de nodos) antes de pasar al siguiente nivel.
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

### 1.7 Implementaciones reales en tecnología actual

No son ejemplos de libro de texto — son árboles corriendo en producción, hoy, en herramientas que ya usas:

- **Git**: cada commit no apunta directo a tus archivos, apunta a un **árbol de objetos "tree"**: una carpeta es un `tree` que contiene referencias a otros `tree` (subcarpetas) y a `blob` (contenido de archivo). Cuando corres `git commit`, Git recorre tu proyecto con la misma lógica recursiva que programaste en la Parte 1 — "yo soy una carpeta, mi contenido es lo mío más lo de mis subcarpetas" — y por eso dos commits que comparten la mayoría de sus archivos reutilizan los mismos nodos `tree` sin duplicarlos.
- **El DOM del navegador**: cada página web que abres se representa internamente como un árbol — `<html>` es la raíz, `<body>` y `<head>` son sus hijos, cada `<div>` un nodo con sus propios hijos. Cuando JavaScript hace `document.querySelector()` o cuando el navegador "pinta" la página, está recorriendo este árbol. React usa una copia de este árbol (el "Virtual DOM" / React Fiber) para calcular el mínimo de cambios necesarios antes de tocar el DOM real — una optimización que depende directamente de poder comparar dos árboles rápido.
- **Compiladores e intérpretes**: cuando escribes código, antes de ejecutarlo se convierte en un **Árbol de Sintaxis Abstracta (AST)** — `(3 + 5) * 2` se vuelve un árbol de operaciones, exactamente como viste en la sección 1.6. Python, V8 (el motor de JavaScript de Chrome) y cualquier compilador moderno recorren ese árbol para generar el código final o interpretarlo nodo por nodo.
- **Autocompletado y corrector ortográfico**: un **Trie** (árbol de prefijos) guarda palabras letra por letra compartiendo el camino común — "casa" y "caso" comparten las primeras 3 letras en el mismo camino del árbol. Es la estructura detrás del autocompletado de tu teclado y de la búsqueda por prefijo en bases de código (ej. VS Code buscando símbolos).
- **Blockchain — Árboles de Merkle**: Bitcoin y Ethereum no guardan todas las transacciones de un bloque en una lista plana; las organizan en un árbol binario donde cada nodo es el hash de sus dos hijos, hasta llegar a un único hash raíz ("Merkle root"). Esto permite verificar que una transacción específica pertenece a un bloque revisando solo `O(log n)` hashes en vez de descargar el bloque completo — la misma ventaja de velocidad que ya comprobaste con el ABB en la sección 1.5, aplicada a criptografía.
- **Inteligencia artificial — árboles de búsqueda de juego**: motores de ajedrez (Stockfish) y el algoritmo que usó AlphaGo exploran un **árbol de jugadas posibles** (minimax con poda alfa-beta) — cada nodo es un estado del tablero, cada hijo una jugada posible desde ahí.

---

## Parte 2 — Grafos

### 2.1 Por qué un grafo y no un árbol

Un árbol no permite ciclos ni que un nodo tenga más de un "padre". Pero el mundo real sí tiene eso: una ciudad conectada por carretera a varias otras, cuentas de usuario que se siguen mutuamente, páginas web que se enlazan entre sí formando ciclos. Para eso existe el **grafo**: un conjunto de **vértices** (nodos) conectados por **aristas** (conexiones), sin la restricción jerárquica del árbol.

### 2.2 Componentes de un grafo

Igual que hiciste con los árboles en la sección 1.1, antes de clasificar o programar grafos conviene fijar el vocabulario:

- **Vértice** (o nodo): cada elemento del grafo. En el mapa de ciudades, cada ciudad es un vértice.
- **Arista** (o arco): la conexión entre dos vértices. En el mapa, cada carretera es una arista.
- **Adyacencia**: dos vértices son adyacentes si existe una arista directa entre ellos — son "vecinos".
- **Grado de un vértice**: cuántas aristas le llegan o salen. En un grafo **dirigido** se divide en **grado de entrada** (aristas que llegan) y **grado de salida** (aristas que salen); en uno **no dirigido** es solo un número.
- **Peso**: el valor asociado a una arista (distancia, costo, tiempo). Si no se especifica, se asume que todas las aristas "cuestan" lo mismo.
- **Camino**: una secuencia de vértices donde cada uno está conectado al siguiente por una arista. `Durango → Torreón → Saltillo` es un camino.
- **Ciclo**: un camino que empieza y termina en el mismo vértice sin repetir aristas en medio (por ejemplo, Durango → Torreón → Saltillo → Monterrey → Culiacán → Mazatlán → Durango).
- **Componente conexo**: un subconjunto de vértices donde todos son alcanzables entre sí por algún camino. Un grafo puede tener varios "pedazos" separados — cada pedazo es un componente conexo.
- **Subgrafo**: un grafo más pequeño formado con algunos de los vértices y aristas del grafo original (lo mismo que un subárbol, pero sin la restricción jerárquica).

```
   Durango --265km-- Torreon
      \                 |
      310km            270km
        \                |
      Mazatlan        Saltillo
          \               |
         215km           85km
            \              |
          Culiacan------900km------Monterrey

Vértices: 6 ciudades.  Aristas: 6 carreteras (cada una con peso = distancia).
Grado de Durango = 2 (se conecta con Torreón y Mazatlán).
Durango-Torreon-Saltillo-Monterrey-Culiacan-Mazatlan-Durango = un ciclo (vuelve al inicio).
Todo el mapa es UN SOLO componente conexo: desde cualquier ciudad llegas a cualquier otra.
```

### 2.3 Tipos de grafos

- **Dirigido vs. no dirigido**: en un grafo **dirigido**, cada arista tiene un sentido único (seguir a alguien en redes sociales no implica que te sigan de vuelta). En uno **no dirigido**, la conexión funciona en ambos sentidos (una carretera se recorre de ida y vuelta). Ya programaste los dos con el parámetro `dirigido` de la clase `Grafo`.
- **Ponderado vs. no ponderado**: **ponderado** cuando sus aristas tienen peso (la red de ciudades, con distancias en km); **no ponderado** cuando todas las conexiones valen lo mismo (quién sigue a quién en redes sociales, sin medir "qué tan fuerte" es esa conexión).
- **Cíclico vs. acíclico**: **cíclico** si existe al menos un camino que regresa a su punto de partida (el mapa de ciudades lo es). **Acíclico** si eso es imposible — el caso más importante es el **DAG** (grafo dirigido acíclico), que ya viste en acción en `03_practica_git.md`: el historial de Git nunca puede "regresar" a un commit anterior.
- **Conexo vs. no conexo**: **conexo** si hay un camino entre cualquier par de vértices (como la red de ciudades completa). **No conexo** si el grafo se divide en dos o más componentes conexos sin ninguna arista que los una — por ejemplo, dos redes sociales distintas que nunca se han conectado entre sí.
- **Simple vs. multigrafo**: un grafo **simple** permite como máximo una arista entre cada par de vértices y ninguna arista de un vértice a sí mismo. Un **multigrafo** permite varias aristas paralelas entre el mismo par (por ejemplo, dos vuelos distintos entre las mismas dos ciudades, cada uno con su propio horario y precio).
- **Completo**: todo vértice está conectado directamente con todos los demás. Un grupo de chat donde todos pueden escribirle a todos directamente es un grafo completo.
- **Bipartito**: los vértices se pueden separar en dos grupos, de forma que las aristas **solo** conectan vértices de un grupo con vértices del otro grupo (nunca dentro del mismo grupo). Ejemplo real: una app de freelancers, donde un grupo son "clientes" y el otro "trabajadores", y las aristas son contrataciones — nunca hay una arista cliente-cliente ni trabajador-trabajador.

| Tipo | Ejemplo de esta práctica / de la vida real |
|---|---|
| Dirigido | `dependencias` (sección 2.6) — instalar Docker antes de desplegar |
| No dirigido | `red` de ciudades (sección 2.5) — carreteras de ida y vuelta |
| Ponderado | La misma `red` de ciudades — cada arista tiene distancia en km |
| No ponderado | Un grafo de "quién sigue a quién" sin medir intensidad |
| Acíclico (DAG) | El historial de commits de Git (`03_practica_git.md`) |
| Cíclico | La red de ciudades — puedes volver a Durango dando la vuelta |
| Bipartito | Clientes↔trabajadores en una app de freelancers |

### 2.4 Representación de grafos (4.2)

Hay dos formas estándar de representar un grafo en memoria:

- **Matriz de adyacencia**: una tabla de `n × n` donde la celda `[i][j]` indica si hay conexión entre el vértice `i` y el `j`. Simple, pero desperdicia memoria si el grafo tiene pocas conexiones (`O(n²)` siempre).
- **Lista de adyacencia**: cada vértice guarda solo la lista de sus vecinos directos. Es la representación que usa casi todo software real (redes sociales, mapas, rutas) porque solo gasta memoria proporcional a las conexiones que realmente existen.

Implementamos con **lista de adyacencia**, la opción que vas a encontrar en código real:

```python
from collections import deque

class Grafo:
    def __init__(self, dirigido=False):
        self.dirigido = dirigido
        self.lista_adyacencia = {}  # vértice -> lista de (vecino, peso)

    def agregar_vertice(self, v):
        # setdefault evita borrar vecinos ya existentes si el vértice
        # ya estaba en el diccionario.
        self.lista_adyacencia.setdefault(v, [])

    def agregar_arista(self, u, v, peso=1):
        self.agregar_vertice(u)
        self.agregar_vertice(v)
        self.lista_adyacencia[u].append((v, peso))
        # Si NO es dirigido, la conexión también se registra al revés
        # (v -> u), porque se puede recorrer en ambos sentidos.
        if not self.dirigido:
            self.lista_adyacencia[v].append((u, peso))
```

- **Grafo no dirigido**: la conexión va en ambos sentidos (una carretera se recorre en los dos sentidos).
- **Grafo dirigido**: la conexión tiene un solo sentido (seguir a alguien en redes sociales no implica que te sigan de vuelta).
- **Peso**: un valor asociado a la arista (distancia en km, costo, tiempo). Un grafo sin peso asume que todas las conexiones "cuestan" lo mismo.

### Antes de ver el algoritmo: qué es un "heap" y por qué Dijkstra lo necesita

Dijkstra, abajo, usa `import heapq`. Un **heap** (montículo) es una estructura que mantiene siempre accesible, en `O(1)`, **el elemento más pequeño** que contiene, sin tener que ordenar toda la colección cada vez que agregas algo nuevo. `heapq.heappush` mete un elemento manteniendo esa propiedad; `heapq.heappop` saca siempre el más pequeño.

¿Para qué le sirve a Dijkstra? El algoritmo necesita, en cada paso, visitar la ciudad **más cercana que todavía no ha explorado** — sin un heap tendrías que revisar manualmente todas las distancias pendientes cada vez (lento); con un heap, pedirlo es instantáneo. Es la misma idea que una cola, pero en vez de "el que llegó primero" (FIFO), es "el que tiene el valor más chico" — por eso a veces se le llama **cola de prioridad**.

### 2.5 Operaciones básicas sobre grafos (4.2): BFS, DFS, camino más corto

```python
    def bfs(self, inicio):
        # Recorrido por ANCHURA: usa una cola (deque, FIFO). Se visita el
        # vértice inicial, se encolan sus vecinos, y se procesa la cola en
        # el mismo orden en que entraron — por eso avanza "nivel por nivel"
        # y encuentra el camino con menos saltos.
        visitados = {inicio}
        orden = []
        cola = deque([inicio])
        while cola:
            actual = cola.popleft()          # saca el más antiguo en la cola
            orden.append(actual)
            for vecino, _ in self.lista_adyacencia[actual]:
                if vecino not in visitados:
                    visitados.add(vecino)    # marcar antes de encolar evita duplicados
                    cola.append(vecino)
        return orden

    def dfs(self, inicio):
        # Recorrido por PROFUNDIDAD: usa recursión (que internamente es una
        # pila de llamadas). Se mete lo más hondo posible por un vecino
        # antes de regresar a explorar los demás — por eso "se hunde" en
        # una rama completa antes de probar otra.
        visitados = set()
        orden = []
        def recorrer(v):
            visitados.add(v)
            orden.append(v)
            for vecino, _ in self.lista_adyacencia[v]:
                if vecino not in visitados:
                    recorrer(vecino)          # baja un nivel más antes de seguir con los demás vecinos
        recorrer(inicio)
        return orden

    def hay_camino(self, origen, destino):
        # Reutiliza BFS: si destino aparece en el recorrido desde origen,
        # es alcanzable. No importa BFS o DFS para esta pregunta.
        return destino in self.bfs(origen)

    def dijkstra(self, inicio):
        # Camino MÁS CORTO (menor costo total) con pesos. Usa un heap (cola
        # de prioridad) para siempre sacar el vértice pendiente más cercano.
        import heapq
        distancias = {v: float('inf') for v in self.lista_adyacencia}
        distancias[inicio] = 0
        heap = [(0, inicio)]
        while heap:
            dist_actual, actual = heapq.heappop(heap)   # el más cercano pendiente
            if dist_actual > distancias[actual]:
                continue   # ya existe una ruta mejor registrada; esta entrada quedó obsoleta
            for vecino, peso in self.lista_adyacencia[actual]:
                nueva = dist_actual + peso
                if nueva < distancias[vecino]:
                    distancias[vecino] = nueva          # se encontró una ruta más corta
                    heapq.heappush(heap, (nueva, vecino))
        return distancias
```

- **BFS (recorrido por anchura)**: usa una **cola** (justo la estructura de la Unidad 3) — explora nivel por nivel. Responde "¿cuál es el camino con **menos saltos**?".
- **DFS (recorrido por profundidad)**: usa recursión, que internamente es una **pila** (otra estructura de la Unidad 3) — se mete lo más hondo posible antes de retroceder. Responde "¿existe algún camino?" y se usa para detectar ciclos.
- **Dijkstra**: responde "¿cuál es el camino de **menor costo total**?" cuando las aristas tienen peso — no es lo mismo que "menos saltos": un camino con 2 saltos puede costar más que uno con 4 saltos si las distancias son grandes.

### 2.6 Ejecución real: una red de ciudades (grafo no dirigido con peso)

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

### 2.7 Ejecución real: grafo dirigido (dependencias de despliegue)

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

### 2.8 Aplicaciones de grafos (4.2)

- **Mapas y GPS**: ciudades/intersecciones = vértices, calles = aristas con peso (distancia o tiempo) → Dijkstra o A*.
- **Redes sociales**: usuarios = vértices, "sigue a" = arista dirigida. BFS calcula "amigos en común" o "grados de separación".
- **Redes de computadoras**: dispositivos = vértices, enlaces = aristas → encontrar la mejor ruta para un paquete de datos.
- **Gestión de dependencias** (paquetes `npm`/`pip`, tareas de un proyecto): grafo dirigido donde un ciclo significa un error de diseño (dependencia circular).
- **Motores de recomendación**: productos/usuarios conectados por "compró junto con" — se recorre el grafo para sugerir productos relacionados.

### 2.9 Implementaciones reales en tecnología actual

- **Google Maps / Waze**: el grafo de calles no tiene pesos fijos — se recalculan en tiempo real con datos de tráfico de millones de teléfonos, y el algoritmo real no es Dijkstra puro sino variantes como **A\*** (Dijkstra + una estimación de qué tan cerca está el destino, para explorar menos nodos) y **Contraction Hierarchies**, que precomputan "atajos" entre intersecciones importantes para responder rutas de miles de km en milisegundos.
- **El "Social Graph" de Meta/Facebook**: toda la plataforma se modela como un grafo gigantesco — usuarios, páginas, grupos, publicaciones, todo son vértices, y "le gusta", "sigue", "es amigo de" son aristas. Internamente usan un sistema propio llamado **TAO** (The Associations and Objects) construido específicamente para servir consultas de grafo a la escala de miles de millones de nodos, algo que una base de datos relacional normal no puede hacer eficientemente.
- **Bases de datos de grafos (Neo4j, Amazon Neptune)**: a diferencia de PostgreSQL (donde relacionar tablas requiere `JOIN`s costosos), estas bases de datos guardan los datos *como* un grafo desde el disco — cada "salto" entre nodos conectados es `O(1)` en vez de un `JOIN`. Se usan para detección de fraude bancario (encontrar anillos de cuentas conectadas sospechosamente) y para sistemas de recomendación en tiempo real.
- **Grafos de conocimiento (Knowledge Graphs)**: el panel de información que aparece a la derecha cuando buscas algo en Google ("Google Knowledge Graph") es un grafo gigante donde los nodos son entidades del mundo real (personas, lugares, conceptos) y las aristas son sus relaciones ("nació en", "dirigió la película"). Wikidata es la versión abierta y pública de esta misma idea.
- **Grafos de cómputo en Inteligencia Artificial**: cuando entrenas una red neuronal con TensorFlow o PyTorch, el modelo completo se representa internamente como un **grafo dirigido de operaciones** (cada capa es un nodo, los datos fluyen por las aristas). El entrenamiento (`backpropagation`) literalmente recorre ese grafo al revés, de la salida hacia la entrada, calculando gradientes — un recorrido de grafo con nombre propio en el mundo del Machine Learning.
- **Internet mismo**: el protocolo de enrutamiento **BGP** que conecta a todos los proveedores de internet del mundo modela routers y redes como un grafo dirigido con pesos, y cada router corre su propia versión de un algoritmo de camino más corto (similar a Dijkstra) para decidir por dónde reenviar cada paquete que recibe.

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
