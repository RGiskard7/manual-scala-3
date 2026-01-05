## Capítulo 21: Diseño de Estructuras y Algoritmos Avanzados

### 1. Explicación Teórica: Inmutabilidad, Algoritmos Funcionales y Complejidad

El diseño avanzado de estructuras y algoritmos en Scala se basa en tres pilares interconectados que definen la Programación Funcional Pura (PFP): la **inmutabilidad**, la **composición declarativa** y la **eficiencia algorítmica**.

#### Estructuras de Datos Persistentes e Inmutables

En Scala, las colecciones son, por defecto, **inmutables**. Esto significa que cualquier "modificación" de una estructura existente (como añadir un elemento a una lista) no altera la original, sino que **devuelve una nueva colección**. Esta práctica es crucial para crear **sistemas concurrentes seguros** al eliminar el riesgo de condiciones de carrera (_race conditions_) asociadas al estado mutable compartido.

Cuando se diseña software avanzado, es fundamental elegir la estructura de datos correcta basándose en el análisis de rendimiento:

- **Listas (`List`):** Son la colección fundamental inmutable y están optimizadas para la **recursión** y las operaciones de anteposición (agregar al inicio o `cons`).
- **Vectores (`Vector`):** Se prefieren sobre `List` en escenarios donde se necesita acceso por índice con una complejidad garantizada de $O(1)$.
- **Estructuras Personalizadas (_Homegrown Collections_):** Para lograr la verdadera maestría, el desarrollador debe ser capaz de implementar estructuras complejas (como árboles binarios de búsqueda inmutables o conjuntos) utilizando los principios funcionales (`case class`, `sealed trait`).

#### Algoritmos Funcionales y Complejidad (O(n))

En lugar de construir algoritmos utilizando iteración imperativa (`for` o `while` con variables mutables `var`), Scala emplea dos técnicas principales para el diseño de algoritmos a gran escala:

1. **Funciones de Orden Superior (HOFs):** Métodos como `map`, `filter`, `fold` y `reduce` permiten definir la lógica **declarativamente** (el _qué_ hacer) en lugar de imperativamente (el _cómo_ iterar).
2. **Recursión:** Es el mecanismo preferido para reemplazar los bucles iterativos. En un contexto funcional, el algoritmo llama a la función sobre un subconjunto de los datos hasta alcanzar un caso base.

El análisis teórico de la **complejidad** $O(n)$ (notación de orden) es esencial para determinar la eficiencia de estas estructuras y algoritmos, asegurando que las operaciones clave (como inserción, búsqueda o eliminación) cumplan con los requisitos de rendimiento esperados (por ejemplo, $O(1)$ para las operaciones de pilas y colas).

### 2. Sintaxis y Estructura: Modelado Inmutable y Recursión

#### A. Tipos de Datos Algebraicos (ADTs) para Estructuras

Las estructuras de datos inmutables se modelan utilizando **ADTs (Tipos de Datos Algebraicos)**, combinando `sealed trait` (o `enum` en Scala 3) con `case class`.

```scala
// El ADT base para una Lista Simple (similar a List.scala)
sealed trait MyList[+T]

// Tipo de Producto 1: La lista vacía (Singleton Object)
case object MyNil extends MyList[Nothing]

// Tipo de Producto 2: La lista no vacía (Case Class Cons)
final case class MyCons[+T](
    head: T,           // Primer elemento (immutable)
    tail: MyList[T]    // Resto de la lista (immutable)
) extends MyList[T]
```

#### B. Sintaxis de Reducción Funcional (`reduce`)

Para aplicar un algoritmo de **reducción** (que resume toda una colección en un único valor), se utiliza `reduce` o `fold`.

```scala
// Definición genérica de una operación de reducción (requiere un tipo A)
def sumar(list: MyList[Int]): Int =
  // Map/Reduce se aplican internamente usando Pattern Matching y recursión.
  list match {
    // Caso base: La lista vacía devuelve 0 (usando foldLeft con un valor inicial de 0).
    case MyNil => 0
    // Caso recursivo: Suma la cabeza y llama recursivamente a la suma sobre la cola.
    case MyCons(h, t) => h + sumar(t)
    // Nota: El compilador exige la declaración del tipo de retorno (Int) para la recursión.
  }
```

### 3. Ejemplos de Código: Reducción (Algoritmo `reduce`)

El algoritmo `reduce` (`foldLeft`/`foldRight`) es la forma canónica de aplicar una lógica sobre toda la colección para obtener un valor final,. Aquí simulamos la implementación de `reduceLeft` para encontrar el elemento máximo en una lista (algoritmo de búsqueda lineal, $O(n)$).

**Caso de Uso Realista:** Encontrar el jugador con la puntuación más alta en una lista de resultados de un juego.

```scala
case class Jugador(nombre: String, score: Int)

// Definimos la lista inmutable usando la estructura MyCons/MyNil (del punto 2)
val resultados = MyCons(
  Jugador("Max", 5000),
  MyCons(Jugador("Ana", 8000),
    MyCons(Jugador("Leo", 2500), MyNil)
  )
)

// Función genérica recursiva que simula reduce/fold para encontrar el máximo.
// Requiere que el tipo T sea comparable (usando un Type Class implícito: Ordering[T])
def encontrarMaximo(list: MyList[Jugador]): Option[Jugador] = {

  def foldRec(actual: Jugador, tail: MyList[Jugador]): Jugador =
    tail match {
      case MyNil => actual // Caso base: si la cola está vacía, devuelve el 'actual'.

      // Caso recursivo: Compara el 'actual' con la 'cabeza' de la cola.
      case MyCons(h, t) =>
        val siguienteMax = if (h.score > actual.score) h else actual
        foldRec(siguienteMax, t) // Llama recursivamente con el nuevo máximo.
    }

  list match {
    case MyNil => None // Si la lista está vacía, no hay máximo.
    case MyCons(h, t) => Some(foldRec(h, t))
  }
}

// Ejecución y resultado (usando Pattern Matching para desestructurar el Option):
val jugadorGanador = encontrarMaximo(resultados) match {
  case Some(j) => s"Ganador: ${j.nombre} con ${j.score}"
  case None    => "No hay resultados."
}

// Resultado esperado: Ganador: Ana con 8000
```

Este ejemplo usa **Pattern Matching** para desestructurar las `case class` (`MyCons` y `MyNil`),, y la **recursión** para iterar sobre la estructura inmutable, un patrón algorítmico fundamental en PF.

### 4. Comparativa con Java y Python

La principal diferencia en el diseño de estructuras y algoritmos en Scala radica en su adherencia al paradigma funcional y el sistema de tipos estático:

| Aspecto                  | Java (Imperativo/Tradicional)                                                                        | Python (Dinámico/Comprensiones)                                      | Scala (Funcional/Idiomático)                                                                                                         |
| :----------------------- | :--------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| **Estructuras de Datos** | Predominan estructuras mutables (`ArrayList`, `Array`).                                              | Estructuras dinámicas (listas) mutables por defecto.                 | **Inmutabilidad por defecto** (`List`, `Vector`),. Las "modificaciones" siempre crean una nueva estructura.                          |
| **Algoritmos/Iteración** | Uso de **bucles `for`/`while`** y mutación en su lugar (_in-place_).                                 | Bucles, o _list comprehensions_ para inmutabilidad simple.           | **HOFs** (`map`, `fold`) y **recursión**,. Se evita la mutación y el estado intermedio.                                              |
| **Seguridad de Tipos**   | El _casting_ y la reflexión son necesarios para estructuras complejas.                               | Tipado dinámico; los errores de estructura se detectan en _runtime_. | **Seguridad estática total.** El `Pattern Matching` desestructura tipos complejos (`case class`) en compilación.                     |
| **Notación O(n)**        | Usualmente se asume; la complejidad puede verse comprometida por la mutabilidad y la sincronización. | N/A.                                                                 | Se espera que el diseño de estructuras de datos inmutables y persistentes considere y garantice la complejidad algorítmica ($O(n)$). |

### 5. Best Practices (_The Scala Way_)

1. **Inmutabilidad Rigurosa:** El diseño de cualquier estructura de datos debe comenzar con **`val`** y **`case class`**,. La mutabilidad (`var`) solo debe usarse en contextos de micro-optimización de bajo nivel o dentro de una unidad encapsulada (como un Actor).
2. **HOFs para Algoritmos:** Evita escribir bucles `for` o `while` al estilo imperativo, ya que requieren `var`. Para cualquier algoritmo de transformación o reducción, utiliza los métodos `map`, `filter`, `flatMap` y `fold`. El uso de `for-comprehensions` es azúcar sintáctico para estas HOFs y mejora la legibilidad en cadenas complejas.
3. **Recursión vs. Iteración:** Utiliza la **recursión** como la técnica fundamental para la iteración en lógica funcional. Para evitar errores de desbordamiento de pila (_Stack Overflow_), la recursión debe ser **recursión de cola optimizada (TCO)**, donde la llamada recursiva es la última operación en la función.
4. **Modelado con ADTs:** Al crear estructuras de datos complejas (árboles, grafos, listas anidadas), utiliza el binomio **`sealed trait/enum`** y **`case class`** para modelar sus variantes. Esto permite usar el **Pattern Matching** para desestructurar la lógica de los algoritmos de forma concisa y segura en tipos,.
5. **Eficiencia de Colecciones:** Aunque `List` es la opción por defecto, usa **`Vector`** si necesitas un acceso rápido (cercano a $O(1)$) a elementos por índice, ya que su rendimiento es superior al de `List` para esta operación.

> El diseño avanzado de algoritmos en Scala no es solo una cuestión de velocidad; es una cuestión de **verificación y composicionalidad**. Al utilizar estructuras inmutables y composición funcional, el desarrollador construye un algoritmo como si estuviera encadenando operaciones matemáticas (HOFs), donde la seguridad y la corrección del tipo son probadas por el compilador, mientras que la eficiencia algorítmica ($O(n)$) se mantiene gracias a las propiedades de la estructura subyacente.