## Capítulo 7: Manejo de Ausencia y Errores Puros

### 1. Explicación Teórica: El Error como un Valor

El manejo de errores en Scala se alinea con los principios de la Programación Funcional (PF), donde las funciones ideales deben ser **puras** y carecer de **efectos secundarios**. Una consecuencia de esta filosofía es que los fallos, la ausencia de valores y los errores no deben interrumpir el flujo de control (como lo hacen las excepciones `throw`) ni introducir ambigüedad (como lo hace `null`).

En su lugar, Scala modela tanto la ausencia como el error como **valores de datos inmutables** que se encapsulan en estructuras de contenedores. Esto permite que los errores y los valores opcionales sean tratados como cualquier otro dato, haciendo que el código sea más seguro y componible.

#### A. Ausencia de Valor (`Option[T]`)

El tipo `Option[T]` es la solución idiomática de Scala para el problema de la **referencia nula (`null`)**, que es la causa principal de las famosas `NullPointerException`. `Option[T]` es un contenedor de valor seguro que garantiza que el desarrollador maneje la posible ausencia de un resultado.

- **Composición:** Funciona como una "colección" que contiene cero o un elemento. Por lo tanto, se puede transformar utilizando Funciones de Orden Superior (HOFs) como `map` y `flatMap`.

#### B. Manejo de Errores Puros (`Try[T]` y `Either[E, R]`)

En lugar de depender de bloques `try/catch` que detienen la ejecución, Scala proporciona contenedores que envuelven el resultado de una operación que puede fallar.

- **`Try[T]`:** Se utiliza principalmente para **capturar excepciones** existentes en código que interactúa con el mundo externo (I/O, librerías Java). Un `Try` resultará en un `Success(valor)` si la operación fue exitosa, o un `Failure(excepción)` si se lanzó una excepción.
- **`Either[E, R]`:** Es la estructura más flexible y preferida en el código funcional puro. `Either` tiene dos parámetros de tipo: `E` para la izquierda (Error) y `R` para la derecha (Resultado o Éxito). Este tipo fuerza al desarrollador a **tipar el error** de forma explícita (a diferencia de `Try`, que solo captura `Throwable`).

### 2. Sintaxis y Estructura Funcional

Tanto `Option`, `Try` como `Either` son **Tipos de Datos Algebraicos (ADTs)**, lo que significa que son tipos de suma (`Sum Types`) que encapsulan sus posibles estados internos.

#### A. Sintaxis de Option

`Option[T]` es un tipo base con dos subtipos (variantes) principales:

|Variante|Descripción|Sintaxis|
|:--|:--|:--|
|**`Some[T]`**|Indica que un valor está presente.|`val resultado = Some("Valor encontrado")`|
|**`None`**|Objeto _Singleton_ que indica la ausencia del valor.|`val ausente: Option[String] = None`|

#### B. Sintaxis de Either

`Either[E, R]` es un contenedor _sesgado a la derecha_ por convención, lo que significa que el valor de éxito se coloca en el lado `Right` y el valor de error en el lado `Left`.

|Variante|Descripción|Sintaxis|
|:--|:--|:--|
|**`Right[R]`**|Indica éxito. `R` es el valor de resultado.|`val exito = Right(42)`|
|**`Left[E]`**|Indica fallo. `E` es el valor o mensaje de error tipado.|`val fallo = Left("Conexión perdida")`|

El uso de `map`, `flatMap` o _for comprehensions_ en estos tipos permite encadenar operaciones sin _unwrapping_ (desenvolver) manualmente el contenedor ni verificar `null`. Si el contenedor es `None` o `Left`, el encadenamiento simplemente se salta la operación y propaga el estado de fallo.

### 3. Ejemplos de Código (Realistas)

#### A. Uso de `Option`: Manejo Seguro de una Búsqueda en `Map`

Buscar una clave en un `Map` es un caso clásico donde el resultado es incierto. El método `get` de `Map` devuelve automáticamente un `Option[V]`.

```scala
// Simula un Map de configuración inmutable (clave String -> valor Int)
val configuraciones = Map("timeout_ms" -> 5000, "retries" -> 3)

// 1. Acceso Seguro (devuelve Option[Int])
val timeoutOpt: Option[Int] = configuraciones.get("timeout_ms")
val portOpt: Option[Int] = configuraciones.get("port")

// 2. Composición funcional (Happy Path): usa map para duplicar el valor si existe
val nuevoTimeoutOpt = timeoutOpt.map(_ * 2)
// nuevoTimeoutOpt: Some(10000)

// 3. Manejo de ausencia con valor por defecto
val puertoFinal: Int =
  portOpt.getOrElse(8080) // Si es None, usa 8080. Si es Some(v), devuelve v.
// puertoFinal: 8080

// 4. Pattern Matching (alternativa idiomática a if/else)
val descripcion = puertoFinal match
  case 8080 => "Puerto por defecto"
  case t if t > 0 => s"Puerto configurado: $t"
  case _ => "Error de puerto"
```

#### B. Uso de `Either`: Conversión de String y Propagación de Fallo

Este ejemplo muestra cómo una función que intenta convertir una cadena en un número (una operación que puede fallar con una `NumberFormatException`) puede modelarse usando `Either`, lo que fuerza a tipar el error (`String`) y a manejarlo explícitamente.

```scala
// Definimos un ADT para tipificar los posibles errores (aunque un String simple también funciona)
enum ErrorDominio:
  case FormatoInvalido(entrada: String)
  case DatoFaltante

// Función que intenta convertir una String a Int, devolviendo Either[Error, Resultado]
def parseAndValidate(entrada: String): Either[ErrorDominio, Int] =
  try {
    val valor = entrada.toInt
    if (valor <= 0)
      Left(ErrorDominio.FormatoInvalido(entrada))
    else
      Right(valor) // Éxito: envuelto en Right
  } catch {
    case _: NumberFormatException =>
      Left(ErrorDominio.FormatoInvalido(entrada)) // Fallo: envuelto en Left
  }

val resultadoValido = parseAndValidate("123")
// Right(123)

val resultadoInvalido = parseAndValidate("abc")
// Left(FormatoInvalido("abc"))

// Encadenamiento funcional usando for comprehension (azúcar sintáctico para flatMap/map)
// La for comprehension se detiene si encuentra un Left (fallo)
val proceso = for {
  num1 <- parseAndValidate("100") // Debe ser Right(100)
  num2 <- parseAndValidate("10")  // Debe ser Right(10)
} yield num1 / num2
// proceso: Right(10)
```

### 4. Comparativa con Java y Python

|Aspecto|Java (Imperativo)|Python (Dinámico)|Scala (Funcional/Puro)|
|:--|:--|:--|:--|
|**Ausencia de Valor**|**`null`**. Causa errores de tipo `NullPointerException`.|`None` o `null`.|**`Option[T]`**. Permite componer el código (`.map`) sin verificación explícita de nulidad.|
|**Manejo de Errores**|**`throw`** / `try-catch`. Interrumpe el flujo de ejecución (`checked/unchecked exceptions`).|`raise` / `try-except`.|**`Either[E, R]`** / **`Try[T]`**. El fallo es un **valor** que se propaga, no una interrupción.|
|**Composición**|Difícil encadenar operaciones que pueden fallar sin múltiples bloques `try-catch`.|Composición limitada en el manejo de errores.|Uso de `map` o `flatMap` sobre `Option`/`Either` para construir secuencias de operaciones de forma declarativa.|
|**Tipado del Error**|La mayoría son `Throwable` o `Exception` genéricos.|Errores basados en la clase.|`Either` permite **tipar explícitamente el error** (`E`) con un `String`, una _Case Class_ o un `enum`.|

### 5. Best Practices (_The Scala Way_)

1. **Eliminación de `null`:** Nunca se debe devolver o aceptar `null` en un código Scala idiomático. Se debe utilizar **`Option[T]`** en su lugar para modelar cualquier valor que pueda estar ausente.
2. **Transparencia de Errores:** Evitar lanzar excepciones (`throw`) desde funciones puras o métodos que definen la lógica de negocio. En su lugar, el **error debe ser un valor** que se retorne dentro de un contenedor, garantizando la **transparencia referencial**.
3. **Elección del Contenedor:**
    - Usar **`Try[T]`** cuando el objetivo es interactuar con una API existente de Java que lanza excepciones que no se pueden modificar, ya que `Try` está diseñado para _capturar_ `Throwable`.
    - Usar **`Either[E, R]`** para definir la lógica interna y APIs que requieren errores tipados específicos, ya que `Either` permite modelar `E` (Error) como una _case class_ o _enum_.
4. **Composición Funcional:** Utilizar **`map` y `flatMap`** (o _for comprehensions_, que son azúcar sintáctico para estos) para encadenar operaciones sobre estos contenedores. Este enfoque permite al desarrollador enfocarse en el "camino feliz" (el flujo de éxito) y delegar la propagación de errores al sistema de tipos.