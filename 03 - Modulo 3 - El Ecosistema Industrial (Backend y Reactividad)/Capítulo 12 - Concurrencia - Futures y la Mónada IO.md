## Capítulo 12: Concurrencia: Futures y la Mónada IO

### 1. Explicación Teórica: El Futuro y el Efecto como Valor

La Programación Funcional (PF) y el diseño de sistemas distribuidos han hecho de la **concurrencia segura** una de las principales fortalezas de Scala. Scala resuelve muchos de los problemas de la programación concurrente que históricamente presentaba Java.

#### El Futuro (Future): La Promesa Asíncrona

El tipo `Future` forma parte de la biblioteca estándar de Scala y es la herramienta más común para manejar la **asincronía**. Un `Future[T]` es un contenedor o **mónada que representa un valor de tipo `T` que estará disponible en algún momento**.

- **Composición No Bloqueante:** Permite componer operaciones asíncronas de manera no bloqueante. Las operaciones se pueden encadenar utilizando funciones como `map` y `flatMap` (o la sintaxis azúcar de _for-comprehensions_) para realizar trabajo secuencial en diferentes hilos sin caer en el "infierno de _callbacks_" (_callback hell_).
- **ExecutionContext:** Para ejecutar un `Future`, se requiere un **`ExecutionContext`**, que actúa como un _pool_ de hilos (thread pool) al que se delega la tarea de ejecutar el código asíncrono.

#### La Mónada IO y las Fibras (Fibers): La Concurrencia Pura

El manejo avanzado de la concurrencia en Scala se basa en el principio de que los efectos secundarios (como las operaciones de entrada/salida o de red) deben tratarse como **valores de datos inmutables**, no como interrupciones o efectos implícitos. Esto se logra mediante la **mónada IO** (Input/Output).

- **Sistemas de Efectos:** Librerías como **Cats Effect** y **ZIO** encapsulan el trabajo concurrente y los efectos secundarios dentro de un tipo `IO`. Un valor `IO` no ejecuta inmediatamente la lógica, sino que es una **descripción** o **plano** (_blueprint_) de un flujo de trabajo concurrente.
- **Fibras (_Fibers_):** Estos sistemas de efectos puros utilizan **fibras** (_Fibers_) o hilos ligeros (_lightweight green threads_) en lugar de los pesados hilos del sistema operativo (JVM Threads). Las fibras permiten una **concurrencia masiva** (miles de fibras ejecutándose en un número reducido de hilos físicos) con una eficiencia de recursos muy superior.

### 2. Sintaxis: Futures y For-Comprehensions

La sintaxis de `Future` es la base para la programación asíncrona en Scala, aprovechando la expresividad de las _for-comprehensions_ para que el código asíncrono se lea de forma secuencial.

#### Estructura de `Future` y Composición

```scala
import scala.concurrent.{Future, ExecutionContext}
import scala.util.{Try, Success}

// 1. Definición del ExecutionContext (Generalmente implicito)
// Necesario para que el Future sepa dónde ejecutar el código.
implicit val ec: ExecutionContext = ExecutionContext.global

// 2. Definición de una operación asíncrona (IO)
def fetchUserData(id: Int): Future[String] = Future {
  // Simulación de un delay de red
  Thread.sleep(100)
  if (id == 1) "Usuario(Ana)" else throw new Exception("ID no encontrado")
}

// 3. Composición Secuencial con for-comprehension
// El flatMap implícito permite encadenar Futures
val procesoCompleto: Future[String] =
  for {
    // Primera operación asíncrona (Future[String])
    userData <- fetchUserData(1)
    // Segunda operación asíncrona que depende del resultado anterior
    procesado <- Future { s"Datos procesados: ${userData.toUpperCase}" }
  } yield procesado

// Resultado: Future(Success(Datos procesados: USUARIO(ANA)))
```

### 3. Ejemplos de Código Realistas: Mónada IO (Cats Effect / ZIO)

Los sistemas de efectos puros representan la forma idiomática (_Scala Way_) de gestionar la asincronía, concurrencia, y gestión de recursos en aplicaciones de alto rendimiento, como servidores web o microservicios.

Utilizaremos la estructura funcional de composición para una operación que requiere múltiples pasos asíncronos (`flatMap`), garantizando que la lógica se lee de forma secuencial y que todos los efectos están tipados.

```scala
// Usando un tipo IO (similar en Cats Effect o ZIO) para encapsular efectos.
// Esto es un 'blueprint', no se ejecuta hasta que se le indica.

// Definición de efectos puros (solo descripción de la acción)
def leerConfig(key: String): IO[String] = IO {
  // Simula la lectura de un archivo o DB
  println(s"Leyendo $key...")
  if (key == "db") "jdbc://prod/db" else throw new Exception("Config no válida")
}

def conectar(url: String): IO[Connection] = IO {
  println(s"Intentando conectar a $url...")
  Connection(url)
}

// Case Class simple para el ejemplo (asumimos que existe)
case class Connection(url: String)

// Flujo principal de negocio modelado como un único efecto componible
val inicializarDB: IO[Connection] =
  for {
    // Paso 1: Obtener la URL (asíncrono/efecto)
    url <- leerConfig("db")
    // Paso 2: Conectarse (asíncrono/efecto, depende del paso 1)
    conn <- conectar(url)
    // Paso 3: Devolver el resultado final
  } yield conn

// La ejecución se realiza de forma controlada "al final del universo"
// (requiere librerías Typelevel o ZIO para correr el efecto).
// Cuando se ejecuta, imprimirá "Leyendo db...", "Intentando conectar a..." y devolverá Right(Connection).
```

### 4. Comparativa (Java y Python)

El enfoque de Scala para la concurrencia está fundamentalmente determinado por su adopción de la inmutabilidad y los tipos de efectos, contrastando con el modelo tradicional basado en mutabilidad y manejo de excepciones de Java.

|Aspecto|Java (Tradicional/Imperativo)|Python (Dinámico/Asyncio)|Scala (Idiomático/Funcional)|
|:--|:--|:--|:--|
|**Manejo de Concurrencia**|Hilos pesados (OS Threads) y manejo manual de bloqueos (`synchronized`, `Lock`).|Basado en _Event Loop_ (Asyncio).|**Fibras/Fibers** ligeras (millones en pocos hilos) y modelos de efectos (`IO Monad`),,.|
|**Composición Asíncrona**|Uso de `CompletableFuture` o `Future` (verboso/híbrido), a menudo requiere anidación.|Uso de `await`/`async` o _coroutines_.|**Monads** (`Future`, `IO`). El uso de `for-comprehensions` permite encadenamiento secuencial legible.|
|**Seguridad de Concurrencia**|Baja. Propensa a _deadlocks_ y _race conditions_ debido al estado mutable compartido.|Media.|**Alta.** Inmutabilidad por defecto garantiza que los datos compartidos son seguros para hilos,.|
|**Manejo de Errores**|Excepciones (`throw`/`try-catch`) que interrumpen el flujo.|Excepciones.|**Errores como Valores Tipados** (`Either`, `ZIO[R, E, A]`), forzando al desarrollador a manejar el fallo de forma compositiva,.|

### 5. Best Practices (_The Scala Way_)

La forma idiomática de construir sistemas concurrentes y asíncronos en Scala ha evolucionado más allá de `Future` para maximizar la composición y la pureza funcional:

1. **Preferir IO Monad sobre Future:** Aunque `Future` es útil para encapsular operaciones I/O sencillas, los sistemas modernos de Scala 3 (como Cats Effect o ZIO) prefieren la **mónada IO**. El `IO` ofrece control total sobre la ejecución del efecto, gestión garantizada de recursos y manejo de interrupción de tareas.
2. **Abrazar la Inmutabilidad:** La base de la concurrencia segura es la inmutabilidad,. Evita el uso de `var` y colecciones mutables (`Array`, `mutable.ListBuffer`) en contextos de concurrencia y utiliza `val` y colecciones inmutables (`List`, `Vector`).
3. **Composición con For-Comprehensions:** Utiliza siempre _for-comprehensions_ para encadenar operaciones asíncronas (`Future` o `IO`). Esto transforma una cadena de `flatMap` potencialmente difícil de leer en una secuencia lógica y legible de pasos.
4. **Modelar Fallos Explícitamente:** No uses excepciones para los errores esperados. En su lugar, utiliza tipos como `Either` o el tipo `ZIO[R, E, A]` (donde `E` es el tipo de error tipado) para que el compilador garantice que los posibles fallos se manejen correctamente.

> En el contexto de la concurrencia, `Future` es como un temporizador que se enciende y te notifica cuando ha terminado (pero no controla el hilo que lo ejecuta). En cambio, la mónada `IO` (ZIO/Cats Effect) es como escribir un guion de película completo: describes cada acción (`IO`) de principio a fin, cómo deben interactuar los actores concurrentes (`Fibers`), cómo manejar los fallos y qué recursos limpiar, pero la película solo comienza a rodar cuando tú lo decides, lo que garantiza que la ejecución siempre es predecible y segura.