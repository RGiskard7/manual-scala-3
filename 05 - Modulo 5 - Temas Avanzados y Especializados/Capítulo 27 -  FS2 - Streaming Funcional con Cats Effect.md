## Capítulo 27: FS2 - Streaming Funcional con Cats Effect

### 1. Explicación Teórica

**FS2 (Functional Streams for Scala)** es una librería de procesamiento de flujos de datos puramente funcional, diseñada para gestionar flujos discretos de valores con soporte nativo para efectos secundarios y concurrencia. A diferencia de los streams de la biblioteca estándar de Scala, que carecen de capacidades avanzadas de control de flujo, FS2 ofrece una infraestructura robusta para el **manejo de recursos, backpressure automático y concurrencia basada en fibras**.

La importancia de FS2 en entornos industriales radica en su capacidad para procesar conjuntos de datos masivos (como archivos de varios GB) utilizando una cantidad mínima de memoria RAM (unos pocos MB) mediante el procesamiento de ventanas de datos manejables. FS2 actúa como el corazón del stack de Typelevel, integrándose profundamente con **Cats Effect** para suspender efectos y con **http4s** para manejar cuerpos de peticiones HTTP como flujos de bytes. Los streams en FS2 son **descripciones inmutables** (blueprints); el código define qué debe suceder, pero la ejecución real se pospone hasta que el flujo se compila y se ejecuta en el "fin del universo".

### 2. Sintaxis y Estructura

El tipo central de la librería es `Stream[F, O]`, donde **`F`** representa el tipo de efecto (normalmente `IO` de Cats Effect) y **`O`** representa el tipo de salida.

**Tipos y Operaciones Fundamentales:**

|Operación|Sintaxis|Propósito|
|:--|:--|:--|
|**Stream Puro**|`Stream[Pure, O]`|Flujos que no requieren evaluar efectos (no pueden fallar).|
|**Stream con Efectos**|`Stream.eval(IO(val))`|Crea un flujo que evalúa un efecto y emite el resultado.|
|**Compilación**|`stream.compile.drain`|Transforma la descripción del flujo en un único efecto ejecutable.|
|**Transformación**|`Pipe[F, I, O]`|Alias para una función `Stream[F, I] => Stream[F, O]`.|

**Modelo de Evaluación:** FS2 utiliza un modelo basado en **Pull**. Los consumidores "tiran" de los datos desde arriba hacia abajo; si el consumidor no solicita datos, el productor no los genera, lo que crea un mecanismo de **backpressure intrínseco**.

### 3. Ejemplos de Código Realistas

#### A. Procesamiento Seguro de Archivos Grandes

Utilizando el patrón **bracket**, FS2 garantiza que los recursos (como manejadores de archivos) se liberen correctamente incluso ante fallos o interrupciones.

```scala
import cats.effect.{IO, IOApp}
import fs2.{Stream, text}
import fs2.io.file.{Files, Path}

object FileProcessor extends IOApp.Simple:
  val sourcePath = Path("datos_masivos.csv")
  val targetPath = Path("procesado.txt")

  // Stream que lee, transforma y escribe con memoria constante
  val pipeline: Stream[IO, Unit] =
    Files[IO].readAll(sourcePath) // Flujo de bytes
      .through(text.utf8.decode)   // Convertir a String
      .through(text.lines)        // Separar por líneas
      .filter(_.nonEmpty)
      .map(_.toUpperCase)         // Lógica de negocio
      .intersperse("\n")
      .through(text.utf8.encode)   // Volver a bytes
      .through(Files[IO].writeAll(targetPath))

  def run: IO[Unit] = pipeline.compile.drain // Ejecución del "blueprint"
```

#### B. API Streaming y Concurrencia con `parEvalMap`

FS2 permite ejecutar efectos en paralelo sobre los elementos del flujo, manteniendo el control sobre el número de fibras activas para no saturar el sistema.

```scala
import cats.effect.IO
import fs2.Stream

case class User(id: Int, name: String)

def fetchFromApi(id: Int): IO[User] =
  IO.println(s"Consultando usuario $id...") >> IO.pure(User(id, s"User-$id"))

val userStream = Stream.range(1, 100)
  .parEvalMap(16)(fetchFromApi) // Ejecuta hasta 16 consultas simultáneas en paralelo
  .filter(_.id % 2 == 0)
  .take(10) // Detiene el flujo tras obtener 10 resultados
```

### 4. Comparativa con Java/Python

|Característica|Java (Stream API)|Python (Generators/Asyncio)|Scala (FS2)|
|:--|:--|:--|:--|
|**Backpressure**|No nativo (basado en bloqueos).|Manual o limitado en `asyncio`.|**Automático y nativo** vía modelo Pull.|
|**Manejo de Efectos**|Mezcla lógica con I/O impuro.|Corrutinas `async/await`.|**Referencialmente transparente** (Efectos como valores).|
|**Seguridad de Recursos**|Bloques `try-with-resources`.|Context managers (`with`).|Patrón **Bracket/Resource** garantizado por el runtime.|
|**Concurrencia**|`parallelStream` (difícil de controlar).|Limitada por el GIL o procesos pesados.|**Masiva vía Fibras** (millones de flujos ligeros).|

### 5. Best Practices (The Scala Way)

1. **Nunca usar `toList` en flujos de producción:** Intentar convertir un stream con efectos o infinito directamente a una lista causará un error de compilación o un `OutOfMemoryError` en runtime. Siempre utiliza `.compile.toVector` o similares para materializar el efecto.
2. **Manejar recursos con `bracket` u `onFinalize`:** No intentes limpiar archivos o conexiones dentro de un `handleErrorWith`. Solo `bracket` garantiza que la acción de liberación se ejecute ante cualquier tipo de terminación (éxito, error o cancelación).
3. **Aprovechar los `Chunks` para I/O pesado:** Internamente, FS2 agrupa elementos en `Chunks` para optimizar el rendimiento. Al escribir transformaciones de bajo nivel, trabajar con `Chunks` reduce el overhead de asignación de objetos.
4. **Separar descripción de ejecución:** Mantén tus streams como valores `val` o métodos `def` que devuelven descripciones puras. La llamada a `unsafeRunSync()` o la extensión de `IOApp` debe ser el único lugar donde el programa "cobra vida".

---

**Metáfora de Ingeniería:** Un stream de FS2 es como una **línea de ensamblaje automatizada**; tú no mueves las piezas manualmente (instrucciones imperativas), sino que diseñas los rieles y los sensores (descripciones de efectos). El sistema solo consume energía (CPU/RAM) cuando hay una caja al final de la línea esperando a ser llenada (Pull), y si la salida se atasca, toda la maquinaria se detiene automáticamente para evitar desbordamientos (Backpressure).