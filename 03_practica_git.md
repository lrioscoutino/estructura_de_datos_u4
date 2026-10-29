# Unidad 4 — Práctica guiada: Git como grafo dirigido acíclico

**Materia:** Estructura de Datos (AED-1026)
**Unidad:** 4. Estructuras no lineales
**Tema:** 4.2 Grafos — aplicación real de grafo dirigido acíclico (DAG)
**Público:** estudiantes que **nunca han usado Git** — no se asume ningún conocimiento previo
**Lenguaje/herramienta:** Git (línea de comandos) — todos los comandos de este documento se ejecutaron de verdad antes de publicarse

## Por qué Git está en la práctica de grafos

Ya viste en `02_practica_arboles_grafos.md` que un grafo es un conjunto de vértices conectados por aristas, sin la restricción jerárquica de un árbol. El historial de un repositorio de Git es exactamente eso: **cada commit es un vértice**, y **cada commit "apunta" a su padre** (o padres, si es un merge) con una arista dirigida.

Es un caso particular de grafo con dos propiedades importantes:

- **Dirigido**: la arista va del commit nuevo hacia el commit padre (nunca al revés — no puedes "des-commitear" el pasado).
- **Acíclico**: nunca puedes volver, siguiendo flechas, a un commit que ya visitaste. A esto se le llama **DAG** (Directed Acyclic Graph, grafo dirigido acíclico).

La diferencia con el árbol de la sección 1 es la que ya conoces: en un árbol cada nodo tiene **un solo padre**. En Git, un commit normal también tiene un solo padre — pero un **commit de merge tiene dos (o más) padres**, y ahí es donde Git deja de ser un árbol y se convierte en grafo de verdad. Lo vas a ver ocurrir en vivo en esta práctica.

---

## Parte 0 — Preparar el terreno (nunca has usado Git)

Git no viene instalado por defecto en todos los sistemas. Si al correr `git --version` te da "comando no encontrado", instálalo primero (`sudo apt install git` en Ubuntu/Debian). Si ya tienes Git, verifica la versión:

```bash
git --version
```

**Salida real (en el entorno donde se verificó esta práctica):**

```
git version 2.43.0
```

Git necesita saber quién eres antes de dejarte guardar cambios (esto se configura **una sola vez** por máquina):

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu_correo@ejemplo.com"
```

No hay salida visible si todo salió bien — Git solo guarda esos datos en un archivo de configuración.

---

## Parte 1 — Tu primer repositorio y tu primer commit

Crea una carpeta nueva y conviértela en un repositorio de Git:

```bash
mkdir mi_primer_repo
cd mi_primer_repo
git init
```

**Salida real:**

```
Initialized empty Git repository in /tmp/git_demo/.git/
```

`git init` no crea ningún commit todavía — solo prepara una carpeta oculta `.git/` donde Git va a guardar **todo** el historial (el grafo completo vive ahí dentro, no en tus archivos visibles).

Ahora crea un archivo y dile a Git que lo "vigile":

```bash
echo "hola" > a.txt
git add a.txt
git status
```

**Salida real:**

```
En la rama master

No hay commits aún

Cambios a ser confirmados:
  (usa "git rm --cached <archivo>..." para sacar del área de preparación)
	nuevo archivo:   a.txt
```

`git add` no guarda el commit todavía — solo mueve el archivo al **área de preparación** (staging area), la sala de espera antes de confirmar. `git status` siempre te dice en qué estado está tu carpeta; vas a usarlo constantemente.

Ahora sí, el primer commit — el primer **vértice** de tu grafo:

```bash
git commit -m "primer commit"
```

**Salida real:**

```
[master (commit-raíz) 82bf357] primer commit
 1 file changed, 1 insertion(+)
 create mode 100644 a.txt
```

Fíjate en `82bf357`: es el identificador único de este commit (un hash SHA-1, los primeros 7 caracteres mostrados). Cada vértice de tu grafo tiene un identificador así, nunca se repite.

Modifica el archivo y haz un segundo commit, para empezar a formar la cadena:

```bash
echo "mundo" >> a.txt
git add a.txt
git commit -m "segundo commit"
git log --oneline
```

**Salida real:**

```
[master 8526080] segundo commit
 1 file changed, 1 insertion(+)
8526080 segundo commit
82bf357 primer commit
```

`git log --oneline` imprime el grafo en el único orden en que un grafo dirigido con un solo "extremo" se puede leer en línea recta: del más nuevo al más viejo, siguiendo las flechas "apunta a su padre" hacia atrás. Ahora mismo tu grafo es la forma más simple posible: dos vértices, una arista.

```
[82bf357] <---- [8526080]
 primer           segundo
 commit           commit
(padre)           (hijo, HEAD)
```

---

## Parte 2 — Ramas: el grafo deja de ser una línea recta

Hasta ahora tu grafo es una lista ligada disfrazada — cada commit tiene exactamente un hijo. Una **rama** (`branch`) es simplemente una etiqueta que apunta a un commit; cuando haces un commit nuevo estando en una rama, la etiqueta se mueve al nuevo commit. Si creas una segunda rama desde el mismo punto, el mismo vértice puede empezar a tener **dos caminos distintos saliendo de él** — ahí el grafo deja de ser una cadena.

```bash
git branch rama-nueva
git checkout rama-nueva
echo "cambio en rama" > b.txt
git add b.txt
git commit -m "commit en rama-nueva"
```

**Salida real:**

```
Switched to branch 'rama-nueva'
[rama-nueva 4c32aa4] commit en rama-nueva
 1 file changed, 1 insertion(+)
 create mode 100644 b.txt
```

Ahora regresa a `master` (la rama original) y haz **otro** commit distinto desde el mismo punto de partida:

```bash
git checkout master
echo "cambio en master" > c.txt
git add c.txt
git commit -m "commit en master"
git log --oneline --all --graph --decorate
```

**Salida real:**

```
* 4d95d15 (HEAD -> master) commit en master
| * 4c32aa4 (rama-nueva) commit en rama-nueva
|/
* 8526080 segundo commit
* 82bf357 primer commit
```

Ahí está: el commit `8526080` tiene **dos hijos** (`4d95d15` y `4c32aa4`) — algo que un árbol binario nunca permitiría sin romper sus reglas, pero que un grafo general acepta sin problema. `git log --graph` literalmente te dibuja el grafo en texto: cada `*` es un vértice, las líneas `|` y `/` son las aristas.

```
                 4d95d15 (master)
                /
8526080 ------+
     \         \
      82bf357   4c32aa4 (rama-nueva)
```

---

## Parte 3 — Merge: el momento en que un commit tiene dos padres

Esto es lo que convierte a Git en un grafo de verdad y no solo en un árbol invertido. Une las dos ramas con un **merge**:

```bash
git merge rama-nueva -m "merge rama-nueva"
git log --oneline --all --graph --decorate
```

**Salida real:**

```
Merge made by the 'ort' strategy.
 b.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 b.txt
*   4b50083 (HEAD -> master) merge rama-nueva
|\
| * 4c32aa4 (rama-nueva) commit en rama-nueva
* | 4d95d15 commit en master
|/
* 8526080 segundo commit
* 82bf357 primer commit
```

El nuevo commit `4b50083` tiene **dos padres**: `4d95d15` y `4c32aa4`. Puedes confirmarlo preguntándole directamente al objeto commit:

```bash
git cat-file -p HEAD
```

**Salida real:**

```
tree f3c119251e79d3a75b773f6600c669b7b723dc91
parent 4d95d15c3fb8c322aa353bc1445e9b9f63902273
parent 4c32aa4da576373778913babd9b7446fa059af2b
author Demo <demo@demo.com> 1793233642 -0600
committer Demo <demo@demo.com> 1793233642 -0600

merge rama-nueva
```

Dos líneas `parent` — eso es, al pie de la letra, un vértice con grado de entrada 2 en un grafo dirigido. Ningún árbol binario de la sección 1 permite esto; un grafo sí.

Ahora el grafo completo tiene una forma de diamante:

```
            82bf357
               |
            8526080
             /    \
       4d95d15    4c32aa4
             \    /
            4b50083  (el merge: dos padres)
```

---

## Parte 4 — El árbol escondido dentro de cada commit (bonus: conecta con la Parte 1 del documento 02)

Cada commit no solo apunta a su padre — también apunta a un **árbol de archivos** (lo que viste en 02, sección 1.7, "Git: árbol de objetos"). Pídele a Git que te muestre el árbol del commit actual:

```bash
git cat-file -p HEAD^{tree}
```

**Salida real:**

```
100644 blob 70e300cdc0a6474be66e31790145902efd3dcd77	a.txt
100644 blob 8511ba0045fc132b8cd29d57c574a69a47724161	b.txt
100644 blob adcfafc93c6d9e7d5b92ac708e5e618dcdef9e7d	c.txt
```

Esto confirma algo importante: **dentro de Git conviven las dos estructuras de esta unidad al mismo tiempo**. El historial de commits es un **grafo** (permite dos padres, permite ramas que se separan y se vuelven a unir). El contenido de archivos de cada commit es un **árbol** (cada carpeta es un nodo con hijos, sin ciclos, exactamente como el árbol de objetos "tree"/"blob" que ya viste en la sección 1.7 de `02_practica_arboles_grafos.md`).

---

## Parte 5 — Comandos que vas a usar todos los días (referencia rápida)

| Comando | Qué hace | Analogía con el grafo |
|---|---|---|
| `git init` | Crea el repositorio (el grafo vacío) | Crear el lienzo donde van a vivir los vértices |
| `git add <archivo>` | Prepara un cambio para el próximo commit | Aún no crea ningún vértice |
| `git commit -m "msj"` | Crea un vértice nuevo, con arista hacia su padre | `insertar()` de un nodo nuevo al final |
| `git log --oneline --graph --all` | Dibuja el grafo completo en texto | El equivalente a `nivel_por_nivel()` o un recorrido impreso |
| `git branch <nombre>` | Crea una etiqueta que apunta a un commit | Marcar un vértice para empezar un camino nuevo desde ahí |
| `git checkout <rama>` | Mueve dónde estás "parado" en el grafo | Mover el puntero `actual` que usabas al recorrer el árbol |
| `git merge <rama>` | Crea un vértice con dos padres, uniendo dos caminos | El único punto donde el grafo deja de parecer un árbol |
| `git status` | Te dice en qué estado está tu carpeta de trabajo | — |
| `git diff` | Muestra línea por línea qué cambió y no se ha guardado | — |

**Ejemplo real de `git diff`** (después de modificar `a.txt` sin hacer commit todavía):

```
diff --git a/a.txt b/a.txt
index 70e300c..6616fda 100644
--- a/a.txt
+++ b/a.txt
@@ -1,2 +1,3 @@
 hola
 mundo
+linea nueva
```

La línea con `+` al inicio es contenido nuevo; nada se borró, por eso no hay líneas con `-`.

---

## Cierre: qué te llevas de esta práctica

1. Un repositorio de Git **es** un grafo dirigido acíclico: cada commit es un vértice, cada "apunta a su padre" es una arista dirigida.
2. Mientras no hagas merges, el grafo se ve como una cadena (lista ligada). En cuanto creas una rama, un vértice gana más de un hijo. En cuanto haces un merge, un vértice gana más de un padre — ahí deja de parecerse a un árbol.
3. `git log --graph` literalmente te imprime el dibujo del grafo que llevas estudiando toda la unidad — no es una metáfora, es la misma estructura de datos.
4. Dentro de cada commit vive, además, un **árbol** de archivos (visto en `02_practica_arboles_grafos.md`, sección 1.7) — Git usa las dos estructuras no lineales de esta unidad al mismo tiempo, cada una para un problema distinto.

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM
- git-scm.com/book — *Pro Git*, capítulo "Git Internals" (objetos commit/tree/blob, DAG de commits)
- Salidas de comandos verificadas en un repositorio Git real creado para esta práctica
