## Capítulo 35: GraphQL con Caliban

### 1. Explicación Teórica

**GraphQL** ha transformado la forma en que los clientes consumen APIs al permitirles solicitar exactamente los datos que necesitan, eliminando los problemas de _over-fetching_ (recibir datos innecesarios) y _under-fetching_ (necesitar múltiples llamadas) típicos de REST. Mientras que REST se centra en recursos fijos en el servidor, GraphQL se basa en un esquema flexible impulsado por el cliente. En entornos industriales, esta flexibilidad reduce drásticamente la latencia percibida y simplifica la evolución de las interfaces frontend sin romper la compatibilidad con el backend.

En el ecosistema de **Scala 3**, la librería **Caliban** se posiciona como el estándar de oro para implementar GraphQL. Caliban adopta el paradigma de **"Código como Dato"** (similar a Tapir), donde el esquema de GraphQL no se escribe en un archivo de texto separado, sino que se deriva directamente de los tipos de Scala (case classes y enums). Esta integración garantiza una **seguridad de tipos total** en tiempo de compilación; si el código compila, el esquema de GraphQL es válido y consistente con la lógica de negocio. Además, al estar construido nativamente sobre **ZIO** (y compatible con **Cats Effect**), Caliban permite gestionar efectos secundarios, dependencias y concurrencia masiva mediante fibras de manera natural y eficiente.

### 2. Sintaxis y Estructura

La arquitectura de una API con Caliban se divide en tres pilares: el **Esquema** (definido por tipos Scala), el **Resolver** (la implementación de la lógica) y el **Intérprete**.

**Componentes Principales de Caliban:**

|Elemento|Propósito|Relación con Scala 3|
|:--|:--|:--|
|**Schema**|Define la estructura de tipos de la API.|Se deriva automáticamente de `case class` y `enum`.|
|**Query**|Operaciones de lectura de datos.|Mapeado a los campos de un objeto o clase inmutable.|
|**Mutation**|Operaciones que modifican el estado.|Funciones que devuelven efectos (ZIO/IO) con resultados.|
|**Subscription**|Flujos de datos en tiempo real.|Mapeado a `ZStream` o `fs2.Stream` para streaming funcional.|

**Configuración Base (sbt):**

```scala
libraryDependencies += "com.github.ghostdogpr" %% "caliban" % "2.9.0"
libraryDependencies += "com.github.ghostdogpr" %% "caliban-http4s" % "2.9.0" // Para integración con http4s
```

### 3. Ejemplos de Código Realistas

#### A. Definición del Dominio y Resolver (Type-Safe)

Utilizamos la potencia de las `case classes` de Scala 3 para definir un catálogo de productos inmutable.

```scala
import caliban.GraphQL.graphQL
import caliban.RootResolver
import zio._
import zio.stream.ZStream

// 1. Modelo de Dominio
case class Producto(id: String, nombre: String, precio: Double)
case class ProductoArgs(id: String)

// 2. Definición de la API (Queries y Subscriptions)
case class Queries(
  producto: ProductoArgs => Task[Producto],
  todosLosProductos: Task[List[Producto]]
)
case class Subscriptions(
  ofertasEnVivo: ZStream[Any, Nothing, Producto]
)

// 3. Implementación (Resolvers)
val queries = Queries(
  args => ZIO.succeed(Producto(args.id, "Teclado Mecánico", 150.0)),
  ZIO.succeed(List(Producto("1", "Mouse", 50.0)))
)
val subscriptions = Subscriptions(
  ZStream.tick(5.seconds).map(_ => Producto("99", "Oferta Flash", 10.0))
)

val api = graphQL(RootResolver(queries, subscriptions))
```

#### B. Integración con http4s y Ejecución

Caliban expone el intérprete que puede ser servido mediante rutas de **http4s**, aprovechando la concurrencia de fibra.

```scala
import caliban.Http4sAdapter
import org.http4s.blaze.server.BlazeServerBuilder
import org.http4s.server.Router

object GraphQLApp extends ZIOAppDefault:
  def run =
    for {
      interpreter <- api.interpreter
      _ <- BlazeServerBuilder[Task]
             .bindHttp(8080, "localhost")
             .withHttpApp(Router("/api/graphql" -> Http4sAdapter.makeHttpService(interpreter)).orNotFound)
             .resource
             .useForever
    } yield ()
```

### 4. Comparativa con Java/Python

|Aspecto|Java (GraphQL-Java)|Python (Graphene)|Scala (Caliban)|
|:--|:--|:--|:--|
|**Definición de Esquema**|Basada en archivos `.graphqls` o anotaciones.|Dinámica y mutable en tiempo de ejecución.|**Derivación estática** desde tipos nativos (Scala 3).|
|**Seguridad de Tipos**|Media; requiere sincronización manual con el código.|Nula; errores detectados solo al ejecutar.|**Máxima**; el compilador valida el contrato de la API.|
|**Efectos y Concurrencia**|`CompletableFuture` (pesado).|`asyncio` (Event Loop).|**Fibras ligeras** y composición de efectos puros (ZIO/IO).|
|**Manejo de Errores**|Excepciones que interrumpen el flujo.|Excepciones dinámicas.|**Errores como valores tipados** integrados en el esquema.|

### 5. Best Practices (The Scala Way)

1. **Aprovechar la Derivación Automática:** No escribas esquemas manualmente. Deja que Caliban use el mecanismo de **Mirrors** de Scala 3 para generar la estructura GraphQL a partir de tus `case classes`, evitando divergencias entre el código y la API.
2. **Modelar Errores con ADTs:** Define tus errores de negocio como miembros de un `enum` o `sealed trait`. Caliban los transformará en extensiones de error de GraphQL, permitiendo al cliente manejar fallos de forma estructurada.
3. **Usar ZLayer para Inyección de Dependencias:** Al igual que en el resto del ecosistema ZIO, utiliza `ZLayer` para inyectar servicios (repositorios, clientes de red) en tus resolvers de manera segura y desacoplada.
4. **Optimizar con Subscriptions para Tiempo Real:** Para notificaciones o actualizaciones frecuentes, utiliza `ZStream` en lugar de _polling_ constante. Esto reduce la carga del servidor y proporciona una experiencia reactiva superior.
5. **Documentación Automática:** Caliban genera introspección por defecto. Úsala junto con herramientas como **Tapir** para tener una fuente única de verdad para toda tu documentación técnica (OpenAPI + GraphQL Introspection).

---

**Metáfora de Ingeniería:** GraphQL con Caliban es como pasar de un **sistema de entrega de paquetes fijos (REST)**, donde recibes una caja pesada aunque solo necesites una llave, a tener un **asistente personal de alta precisión (Caliban)**. Tú le das una lista de deseos detallada (Query) y el asistente, guiado por un manual de reglas infalible (el sistema de tipos de Scala 3), recorre los almacenes del servidor y te entrega exactamente lo que pediste en un solo paquete, sin margen de error.