## Capítulo 14: Desarrollo Web Funcional

### 1. Explicación Teórica: El Paradigma de los Efectos Puros en Web

El desarrollo web funcional en Scala se fundamenta en los mismos principios de inmutabilidad y transparencia referencial que definen la Programación Funcional Pura (PFP), extendiendo el manejo seguro de la concurrencia al ámbito de la red y las operaciones de E/S.

#### El Monad IO como Servidor

En lugar de depender de los tradicionales _frameworks_ de Arquitectura Modelo-Vista-Controlador (MVC) que manejan la lógica y el estado de forma mutable (como Play Framework, que sigue siendo una opción robusta), el desarrollo moderno se enfoca en el **Modelo de Programación de Efectos**.

Esto significa que:

1. **Las operaciones de red (E/S)**, como recibir una petición HTTP o conectarse a una base de datos, se modelan como **valores de datos inmutables** (descripciones de una acción).
2. Estos valores de efecto se encapsulan en una **Mónada IO** (por ejemplo, `IO` de Cats Effect o `ZIO` de ZIO) permitiendo componer el flujo de la aplicación de manera declarativa.
3. La concurrencia es gestionada de manera eficiente mediante **Fibras** (_Fibers_) o hilos ligeros, lo que permite un rendimiento superior al modelo de hilos pesados (_OS Threads_) de la JVM tradicional.

#### El Código como Contrato de API (Tapir)

La abstracción se extiende a la capa de la API con librerías que tratan la definición de _endpoints_ HTTP como **datos** y no como código ejecutable. Librerías como **Tapir** permiten definir de forma declarativa un _endpoint_ (qué recibe, qué devuelve, qué errores puede generar).

Este **código como dato** (o _Code as Data Paradigm_) permite que el compilador derive automáticamente la documentación OpenAPI (Swagger), la implementación del cliente, y la validación, todo a partir de una única fuente de verdad.

---

### 2. Sintaxis: El Desacoplamiento de la Definición del Enrutamiento

En un _stack_ funcional puro (típicamente **Cats Effect con http4s** o **ZIO con ZIO HTTP**), la sintaxis se enfoca en dos niveles: la definición inmutable del contrato y la composición de los efectos de servicio.

#### A. Definición Declarativa del Endpoint (Tapir)

La librería Tapir proporciona un Domain Specific Language (DSL) para describir el contrato de una API de manera _type-safe_, desacoplada de cualquier servidor o motor de efectos.

|Característica|Sintaxis Gramatical (Adaptada de Tapir)|
|:--|:--|
|**Endpoint Base**|`val endpoint = endpoint`|
|**Input (Ruta y Parámetros)**|`.in("api" / "recursos" / path[Tipo]("nombre"))`|
|**Output (Respuesta Exitosa)**|`.out(jsonBody[TipoRespuesta])`|

#### B. Composición del Servicio (http4s y IO)

La librería **http4s** (construida sobre Cats Effect) define los _servicios HTTP_ como `HttpRoutes[F]`, donde `F` es típicamente el efecto `IO`. La sintaxis utiliza `match` expression o patrones (como `GET -> Root`) para enrutar la petición a una función que devuelve un efecto.

```scala
// El servicio se define como rutas de HTTPRoutes[IO]
val service: HttpRoutes[IO] = HttpRoutes.of[IO]:
  // Usa Pattern Matching para enrutar por método y path
  case GET -> Root / "weather" =>
    // fetch1 y fetch2 devuelven un Future o IO (efectos)
    for {
      winner <- fetch1.race(fetch2).timeout(10.seconds)
      response <- Ok(WeatherReport.from(winner))
    } yield response
```

---

### 3. Ejemplos de Código (Generación de API con Tapir)

Este ejemplo muestra cómo se define un _endpoint_ de búsqueda de reportes (`GET /api/report/{reportId}`) y cómo esta única definición se usa para generar tanto la documentación como el servidor, utilizando la seguridad de tipos para el modelo de datos.

```scala
// Importaciones de librerías de efectos y Tapir
import io.circe.generic.auto._ // Para derivar JSON (Codec)
import sttp.tapir._           // Definición de Endpoints
import sttp.tapir.json.circe._

// 1. Modelo de Datos (Tipo Producto Inmutable)
case class Reporte(id: String, contenido: String)
// Se asume que Reporte deriva un Codec para JSON.

// 2. Definición del Endpoint (La Declaración)
// Un endpoint que toma un String (reportId) en la ruta y devuelve un Reporte en JSON
val buscarReporteEndpoint: PublicEndpoint[String, String, Reporte, Any] =
  endpoint
    .name("Buscar Reporte por ID")
    .description("Busca un reporte específico usando su identificador.")
    .in("api" / "report" / path[String]("reportId")) // Input: /api/report/{reportId}
    .errorOut(stringBody)                          // Error: Devuelve un String simple (ej: "404 No Encontrado")
    .out(jsonBody[Reporte])                        // Output: Devuelve un objeto Reporte en formato JSON

// 3. Lógica de Negocio (Manejo del Efecto)
// Función que simula la búsqueda y devuelve un efecto IO[Reporte]
def fetchReport(reportId: String): IO[Either[String, Reporte]] = IO {
  if (reportId == "5ca1a-78fc8d6")
    Right(Reporte(reportId, "Análisis de tráfico de microservicios."))
  else
    Left("Reporte no encontrado.")
}

// 4. Derivación de la Plataforma (Generación en Compilación/Inicio)

// Generar Documentación (OpenAPI / Swagger)
val apiDocs = docsReader // Asumiendo que docsReader está en scope (de Tapir)
  .toOpenAPI(buscarReporteEndpoint, "Servicio de Reportes", "1.0.0")

// Iniciar Servidor (Se asume un builder compatible con Tapir, como http4s o ZIO HTTP)
val server = serverBuilder(port = "8080") // Asumiendo serverBuilder
    .addEndpoint(buscarReporteEndpoint.serverLogic(reportId => fetchReport(reportId)))
    .start()
```

---

### 4. Comparativa con Java o Python

La principal diferencia del desarrollo web en Scala, especialmente con _stacks_ funcionales puros (Cats Effect, ZIO, http4s, Tapir), es la **fase en la que se comprueba la corrección del código**.

| Característica                   | Java (Spring/Jakarta EE)                                                                                              | Python (Django/Flask)                                          | Scala (Funcional Pura)                                                                                                                       |
| :------------------------------- | :-------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| **Manejo de E/S y Concurrencia** | Basado en _OS Threads_ o `CompletableFuture` (híbrido).                                                               | Modelos de _event loop_ (Asyncio).                             | **Fibras (_Fibers_)/Monads IO**. Concurrencia masiva y gestión de efectos puros.                                                             |
| **Definición de API**            | Uso de anotaciones (`@GetMapping`, `@Path`). La documentación (Swagger) se genera mediante _reflection_ en _runtime_. | Rutas definidas en archivos de configuración; tipado dinámico. | **Declarativa (Tapir)**. El _endpoint_ es un **valor** Scala tipado.                                                                         |
| **Seguridad del Contrato**       | La validación del cuerpo/ruta se comprueba en _runtime_; la documentación se desincroniza fácilmente.                 | Sin chequeo estático.                                          | **Chequeo en Compilación**. La definición de Tapir es _type-safe_, garantizando que la documentación y el cliente coincidan con el servidor. |
| **Estado**                       | El estado es mutable por defecto; requiere bloqueos (`synchronized`) para ser _thread-safe_.                          | Mutable.                                                       | **Inmutabilidad por defecto**. Las estructuras de efectos garantizan la seguridad sin bloqueos.                                              |

---

### 5. Best Practices (_The Scala Way_)

Para el desarrollo web en Scala, el enfoque idiomático (el _Scala Way_) exige priorizar las arquitecturas que maximizan la seguridad de tipos y la composicionalidad de los efectos:

1. **Elegir un _Stack_ de Efectos:** **Prioriza el uso de Cats Effect (con http4s)** o **ZIO** sobre _frameworks_ MVC tradicionales como Play. Estos _stacks_ proporcionan un control de concurrencia superior mediante el uso de Fibras (Fibers) y garantizan que los efectos secundarios sean gestionados de forma explícita y segura,.
2. **Abstracción Declarativa con Tapir:** Utiliza **Tapir** para definir tus _endpoints_ de API. Esto permite que el _código de la API se trate como un valor de datos_, facilitando la generación automática de la documentación (OpenAPI) y la creación de clientes _type-safe_.
3. **Full Stack y Case Classes:** Aprovecha Scala.js para el _frontend_ y **reutiliza los modelos de datos inmutables (`case class`)** y los tipos de error tipados (`Either`) en ambos extremos del _stack_. Esto garantiza que los datos enviados desde el _backend_ coincidan exactamente con la estructura esperada por el _frontend_, eliminando errores de serialización en _runtime_.

> En el desarrollo web funcional, estás construyendo una cadena de montaje de efectos. En lugar de ejecutar inmediatamente las acciones (instrucciones imperativas), defines un contrato (Tapir) y un plan de acción (la composición de IO o ZIO). El compilador, a través del tipado estático, se convierte en el inspector de calidad que verifica que cada pieza del plan de acción se conecta perfectamente con la siguiente, garantizando que el servidor sea robusto y que las respuestas coincidan con la documentación antes de que el código se ejecute.