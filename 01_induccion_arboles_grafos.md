# Unidad 4 — Inducción: primer contacto con árboles y grafos

**Materia:** Estructura de Datos (AED-1026)
**Objetivo de esta práctica:** antes de construir las clases `Nodo`/`ArbolBinarioBusqueda`/`Grafo` (eso viene en [02_practica_arboles_grafos.md](02_practica_arboles_grafos.md)), entender con ejercicios sencillos **qué problema resuelve cada estructura** — sin clases, sin recursión complicada, solo diccionarios y listas que ya conoces.

Si nunca has visto árboles ni grafos, empieza aquí. Si ya hiciste esta práctica, pasa directo a la 02.

---

## Parte 0 — Sin computadora: dos dibujos

Antes de escribir código, dibuja en papel (o descríbelo en voz alta si estás en clase):

1. **El árbol genealógico de tu propia familia**, nada más 3 generaciones (abuelos → papás → tú y hermanos). Nota que cada persona tiene **un solo** padre/madre directo hacia arriba en el dibujo, y puede tener varios hijos hacia abajo.
2. **El mapa de las 5 ciudades que conoces más cerca de donde vives**, conectadas por las carreteras reales entre ellas. Nota que aquí **no hay un "arriba"** — cualquier ciudad puede conectar con cualquier otra, y puedes ir y regresar.

Eso que acabas de dibujar número 1 es un **árbol**. Eso que dibujaste número 2 es un **grafo**. Todo lo que sigue es poner esos dos dibujos en una computadora.

---

## Parte 1 — Un árbol sin clases: diccionarios anidados

No necesitas una clase `Nodo` para empezar a pensar en árboles. Un árbol es, ni más ni menos, **un valor con una lista de hijos, donde cada hijo es a su vez un valor con su propia lista de hijos**:

```python
familia = {
    "valor": "Abuelo",
    "hijos": [
        {"valor": "Tío", "hijos": []},
        {"valor": "Papá", "hijos": [
            {"valor": "Yo", "hijos": []},
            {"valor": "Hermano", "hijos": []},
        ]},
    ],
}
```

Lee esta estructura en voz alta: "Abuelo tiene dos hijos: Tío (sin hijos) y Papá (que tiene dos hijos: Yo y Hermano)". Es exactamente tu dibujo de la Parte 0, escrito con llaves y corchetes.

### 1.1 Recorrer el árbol: imprimirlo con sangría

```python
def imprimir_arbol(arbol, nivel=0):
    print("  " * nivel + "- " + arbol["valor"])
    for hijo in arbol["hijos"]:
        imprimir_arbol(hijo, nivel + 1)

imprimir_arbol(familia)
```

**Salida real:**

```
- Abuelo
  - Tío
  - Papá
    - Yo
    - Hermano
```

Fíjate cómo funciona la recursión aquí: la función se llama a sí misma con `nivel + 1` por cada hijo. No hace falta una pila ni una cola explícita — Python ya usa una pila internamente para recordar "en qué nodo iba" cada vez que profundiza un nivel más (esto es DFS, aunque todavía no lo hayamos nombrado así).

### 1.2 Contar cuántas personas hay en el árbol

```python
def contar_nodos(arbol):
    if arbol is None:
        return 0
    total = 1
    for hijo in arbol["hijos"]:
        total += contar_nodos(hijo)
    return total

print("Total de personas en el árbol:", contar_nodos(familia))
```

**Salida real:**

```
Total de personas en el árbol: 5
```

El patrón que acabas de usar — "yo cuento como 1, más lo que cuenten todos mis hijos" — es **la misma idea** detrás de casi cualquier operación sobre árboles: contar, sumar, buscar el máximo, calcular la altura. Cambia qué haces con el nodo actual, pero la forma de recorrer siempre es la misma.

### 1.3 Encontrar a cuántos "saltos" está alguien

```python
def profundidad_hasta(arbol, objetivo, nivel=0):
    if arbol["valor"] == objetivo:
        return nivel
    for hijo in arbol["hijos"]:
        resultado = profundidad_hasta(hijo, objetivo, nivel + 1)
        if resultado is not None:
            return resultado
    return None

print("Saltos de Abuelo a Hermano:", profundidad_hasta(familia, "Hermano"))
```

**Salida real:**

```
Saltos de Abuelo a Hermano: 2
```

Esto es exactamente lo que vas a hacer en la práctica 02, pero ahí en vez de un diccionario con `"valor"`/`"hijos"` vas a usar una clase `Nodo` con `.izq`/`.der` — la clase es solo una forma más disciplinada de escribir lo mismo que ya hiciste aquí.

**Ejercicio para ti:** agrega una persona más al diccionario `familia` (un primo, hijo de "Tío") y corre de nuevo `contar_nodos` y `profundidad_hasta` para confirmar que el conteo y la profundidad cambian como esperas.

---

## Parte 2 — Un grafo sin clases: diccionario de listas

Un grafo es todavía más simple de representar que un árbol: **un diccionario donde cada clave es un nodo, y su valor es la lista de sus vecinos directos**.

```python
amistades = {
    "Ana": ["Beto", "Caro"],
    "Beto": ["Ana", "Caro"],
    "Caro": ["Ana", "Beto", "Dani"],
    "Dani": ["Caro"],
}

for persona, amigos in amistades.items():
    print(f"{persona} conoce a: {amigos}")
```

**Salida real:**

```
Ana conoce a: ['Beto', 'Caro']
Beto conoce a: ['Ana', 'Caro']
Caro conoce a: ['Ana', 'Beto', 'Dani']
Dani conoce a: ['Caro']
```

Fíjate en algo que **no puede pasar en un árbol**: "Ana" aparece en la lista de "Beto", y "Beto" aparece en la lista de "Ana" — es la misma conexión vista desde los dos lados (porque es una amistad no dirigida). Y si sigues las conexiones Ana → Caro → Beto → Ana, **regresas a donde empezaste**. Eso es un **ciclo**, y en un árbol genealógico sería imposible (nadie es su propio ancestro).

### 2.1 Detectar si existe un ciclo desde un nodo

```python
def tiene_ciclo_simple(grafo, inicio):
    visitados = set()
    pila = list(grafo[inicio])
    while pila:
        actual = pila.pop()
        if actual == inicio:
            return True
        if actual not in visitados:
            visitados.add(actual)
            pila.extend(grafo[actual])
    return False

print("¿Hay un ciclo que regresa a Ana?:", tiene_ciclo_simple(amistades, "Ana"))
```

**Salida real:**

```
¿Hay un ciclo que regresa a Ana?: True
```

Esta función usa una **lista como pila** (`.pop()` sin índice saca el último elemento) para explorar vecino por vecino — es, en esencia, un DFS simplificado. En la práctica 02 vas a ver la versión completa con `visitar`/`BFS`/`DFS` sobre una clase `Grafo`, pero la idea — "explora vecinos, no repitas los que ya visitaste, si regresas al inicio hay ciclo" — es la misma que acabas de programar aquí con 10 líneas.

**Ejercicio para ti:** agrega una quinta persona, "Eva", que solo conoce a "Dani" (sin regreso: `"Eva": ["Dani"]`, pero no agregues "Eva" a la lista de "Dani"). Corre `tiene_ciclo_simple(amistades, "Eva")` y explica por qué el resultado es diferente al de Ana — pista: revisa si las conexiones son realmente de ida y vuelta.

---

## Qué llevarte de esta práctica antes de seguir

| | Árbol (`familia`) | Grafo (`amistades`) |
|---|---|---|
| Forma de los datos | `{"valor":..., "hijos":[...]}` anidado | `{nodo: [vecinos]}` plano |
| ¿Puede regresar a donde empezó? | Nunca | Puede (ciclo) |
| ¿Cuántos "padres" por nodo? | Máximo uno | Sin límite |
| La operación que ya programaste | Recorrer + contar + medir profundidad | Explorar vecinos + detectar ciclo |

Con esta base, en [02_practica_arboles_grafos.md](02_practica_arboles_grafos.md) vas a:
- Reemplazar el diccionario `{"valor", "hijos"}` por una clase `Nodo` con `.izq`/`.der`, y agregar la regla de orden (menor a la izquierda) que convierte un árbol cualquiera en un **Árbol Binario de Búsqueda**.
- Reemplazar la exploración manual con pila por los algoritmos con nombre propio: **BFS** (anchura, con cola), **DFS** (profundidad, con pila o recursión) y **Dijkstra** (ruta más barata cuando las conexiones tienen peso).

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM
- docs.python.org/3/tutorial/datastructures.html (diccionarios y listas, base de esta inducción)
