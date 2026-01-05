# Guía de Transición: De Java/Python a Scala 3

Esta guía está diseñada para ayudar a los desarrolladores con experiencia en lenguajes imperativos (Java) o dinámicos (Python) a adoptar la filosofía y el _idioma_ de Scala 3.

## 1. El Cambio de Mentalidad (Filosofía)

### Paradigma Híbrido: POO + PF

Scala es un lenguaje multiparadigma que **fusiona la Programación Orientada a Objetos (POO) y la Programación Funcional (PF) en una sola ontología**. Esta unificación significa que **todo valor es un objeto** y **toda función es un valor de primera clase**. Scala utiliza la madurez de la **Máquina Virtual de Java (JVM)** para ofrecer abstracciones funcionales más potentes. Este diseño permite crear sistemas altamente expresivos, seguros en tipos y capaces de manejar concurrencia compleja de manera eficiente.

### Programación Orientada a Expresiones (_Expression-Oriented Programming_)

En lenguajes como Java o Python, las estructuras de control como `if` o `for` son a menudo _instrucciones_ (_statements_) que ejecutan un comando y no devuelven un valor. En Scala, **casi todo es una expresión** que se evalúa a un valor.

| Concepto              | Java/Python (Instrucción)                                                                                              | Scala (Expresión)                                                                                                                                          |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **If/Else**           | Se utiliza para ejecutar bloques de código, a menudo requiere una variable mutable externa para capturar el resultado. | La estructura `if-else` **devuelve un valor** que puede asignarse directamente a una variable.                                                             |
| **Bloques de Código** | Un bloque `{}` ejecuta instrucciones. El `return` es necesario.                                                        | Un bloque `{}` **evalúa la última expresión** dentro del bloque como su valor de retorno. Esto a menudo elimina la necesidad de la palabra clave `return`. |

### Inmutabilidad por Defecto: `val` vs. `var`

La inmutabilidad no es una convención de estilo, sino un **principio de ingeniería fundamental** en Scala para construir sistemas **concurrentes seguros**. Los objetos inmutables son intrínsecamente seguros para hilos (_thread-safe_), eliminando la necesidad de mecanismos complejos de bloqueo (_locks_).

- **`val` (Value):** Es una **referencia inmutable** cuyo valor no puede cambiar después de la asignación. Es la **opción por defecto** y equivale a usar `final` en Java.
- **`var` (Variable):** Es una **referencia mutable** que puede reasignarse. Su uso es **fuertemente desaconsejado** y debe limitarse a casos muy específicos (ej. optimización de bajo nivel o dentro de un actor, donde el estado es controlado).

## 2. La Piedra Rosetta (Comparativa de Código)

Esta tabla ilustra cómo se realizan tareas comunes utilizando el enfoque idiomático de Scala 3, contrastándolo con el código equivalente en Java y Python.

| Tarea                                    | Código Java / Python                                                                                     | Código Scala 3 Idiomático                                                                                                                                                                                                                                                |
| :--------------------------------------- | :------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Clase de Datos Inmutable**             | **Java (Record):** `public record Persona(String nombre, int edad) {}`                                   | **Scala (`case class`):** `case class Persona(nombre: String, edad: Int)`. Genera automáticamente constructor, _equals_, _hashCode_, _toString_ y método `copy`.                                                                                                         |
| **Manejo de Nulos (Ausencia)**           | **Java:** `if (nombre != null) { ... } else { ... }` (Riesgo de `NullPointerException`).                 | **Scala (`Option`):** Utiliza `Option[String]` (`Some(valor)` o `None`). `nombreOption.map(n => s"Hola $n")` `nombreOption.getOrElse("Desconocido")`.                                                                                                                    |
| **Transformación Funcional (Listas)**    | **Python:** `[x * 2 for x in lista if x % 2 == 0]`                                                       | **Scala (`map`/`filter`):** `lista.filter(_ % 2 == 0).map(_ * 2)`. **Funciones de Orden Superior (HOFs)** sin bucles explícitos.                                                                                                                                         |
| **Control de Flujo Avanzado (`switch`)** | **Java (`switch`):** Utiliza valores constantes; carece de desestructuración de objetos.                 | **Scala (`match` / Pattern Matching):** `comando match {``&nbsp;&nbsp;case Comando.Mover(x, y) => s"Moviendo a $x, $y"``&nbsp;&nbsp;case Comando.Salir => "Finalizado"``&nbsp;&nbsp;case _ => "Error"``}`Permite **desestructurar** `case classes` y patrones complejos. |
| **Objeto Único (`static`)**              | **Java:** `public static final String NOMBRE = "App";`                                                   | **Scala (`object`):** `object Configuracion { val NOMBRE = "App" }`. `object` es un **Singleton** que reemplaza a los miembros `static` de Java.                                                                                                                         |
| **Concurrencia y Asincronía**            | **Java (Threading):** `new Thread(new Runnable() { ... })` (Bloqueo, gestión manual de hilos y _locks_). | **Scala (Future / IO / Fibras):** `Future { heavyComputation() }`o mejor, usando efectos puros: `IO.delay { heavyComputation() }.map(...)`,. Utiliza **Fibras** (o _green threads_) para concurrencia ligera y segura.                                                   |
| **Streams (Lectura de Archivo)**         | **Python:** `with open('file.txt', 'r') as f: ...`                                                       | **Scala (FS2/IO):** `readInputStream[IO](IO(inps), 4096)` `.through(Files[IO].writeAll(Path("file.txt")))`. La gestión de recursos (`bracket`) es garantizada por el sistema de efectos.                                                                                 |
| **Conexiones HTTP (Load Balancer)**      | **Java (Servlets/Netty):** Código imperativo y gestión explícita de _threading_.                         | **Scala (Cats Effect/Http4s):** Implementación funcional de bajo nivel para enrutamiento.`HttpRoutes.of[IO]: { case GET -> Root / "path" => ... }`. Los _endpoints_ se describen como valores tipados y componibles.                                                     |

## 3. Sintaxis de Scala 3 (Novedades)

Scala 3 (proyecto **Dotty**) trajo una reescritura del compilador con el objetivo de simplificar la sintaxis y mejorar la seguridad. Los cambios más notables son:

### Abstracciones Contextuales (`given` / `using`)

El mecanismo de `implicit` de Scala 2, a menudo confuso, fue dividido en palabras clave que reflejan su **intención**,. Esto es fundamental para _Type Classes_ y la inyección de dependencias (DI) funcional.

- **`given`:** Se utiliza para **definir** o **proporcionar** un valor canónico de un tipo al contexto. Reemplaza a `implicit val/def`.
- **`using`:** Se utiliza para **consumir** ese valor, declarando que una función requiere un valor de un tipo específico del contexto. El compilador lo inyecta automáticamente si lo encuentra en el alcance.

```scala
// 1. Definición (dado que existe un Ordenador para Ints)
trait Ordenador[T]:
  def compara(a: T, b: T): Int // Solo una abstracción

given Ordenador[Int] with // Se define la instancia dada (canonical value)
  def compara(a: Int, b: Int): Int = a - b

// 2. Consumo (usando el Ordenador)
def ordenar[T](lista: List[T])(using ord: Ordenador[T]): List[T] =
  lista.sortWith((a, b) => ord.compara(a, b) < 0)

// El compilador inyecta automáticamente el 'given' Ordenador[Int] al llamar a ordenar.
val numeros = List(3, 1, 4).ordenar // Resulta en List(1, 3, 4)
```

### Nueva Sintaxis de Indentación (Opcional)

Scala 3 permite prescindir de las llaves (`{}`) en favor de una **sintaxis basada en indentación** (similar a Python). El compilador añade marcadores de inicio (`indent`) y fin (`outdent`) basados en el nivel de sangría, aunque se pueden usar delimitadores explícitos como `end`.

```scala
// Sintaxis tradicional (Scala 2/Java Style)
if (edad > 18) {
  println("Adulto")
} else {
  println("Menor")
}

// Sintaxis de indentación (Scala 3 Style)
def clasificar(edad: Int): Unit =
  if edad > 18 then // 'then' es opcional pero ayuda a la legibilidad
    println("Adulto")
  else
    println("Menor")

// [Ejemplo de Bucle For con indentación]
def listarNumeros(xs: List[Int]): Unit =
  for x <- xs do // 'do' es el marcador de inicio de la región
    if x > 0 then
      println(s"Positivo: $x")
    else
      println(s"No positivo: $x")
```

### Enumeraciones y ADTs con `enum`

Scala 3 introduce `enum` como una estructura de primera clase, reemplazando la necesidad de usar `sealed trait` + `case class` para modelar **Tipos de Datos Algebraicos (ADTs)**. Los `enums` pueden tener parámetros y métodos.

```scala
// Antes: sealed trait Comando + 3 case classes
// Ahora: Enum simple,
enum Comando:
  case Mover(x: Int, y: Int)
  case Escribir(texto: String)
  case Salir

// Uso en Pattern Matching
val cmd = Comando.Mover(1, 1)

val descripcion = cmd match
  case Comando.Salir => "Finalizar"
  case Comando.Mover(x, y) => s"Movimiento a ($x, $y)"
  case Comando.Escribir(txt) => s"Escribiendo: $txt"
// El compilador advierte si falta algún caso del 'enum'.
```

### Métodos de Extensión (`extension`)

Permiten añadir nuevos métodos a tipos existentes (incluidos tipos que no controlas, como `String`), sin recurrir a la sintaxis críptica de `implicit class` de Scala 2.

```scala
// Extiende el tipo String con un método de mayúsculas
extension (s: String)
  def enMayusculas: String = s.toUpperCase()

val saludo = "hola".enMayusculas // "HOLA"
```