## Capítulo 32: gRPC Reactivo y Streaming con FS2

### 1. Explicación Teórica: El Problema del Future y la Necesidad de Backpressure

En el Capítulo 31, exploramos la implementación de gRPC basada en `scala.concurrent.Future`, la cual es adecuada para interacciones simples de tipo solicitud-respuesta (unarias),. Sin embargo, el modelo de `Future` presenta limitaciones críticas cuando se enfrenta a flujos de datos masivos o infinitos. Los `Futures` son **eager** (se ejecutan inmediatamente al crearse) y carecen de mecanismos nativos de **backpressure**,. En un sistema de alto rendimiento, si un productor envía datos más rápido de lo que el consumidor puede procesarlos, el sistema corre el riesgo de sufrir un **OutOfMemoryError** al intentar buferizar una cantidad ilimitada de información en la memoria RAM,.

Para resolver esto, el ecosistema de Scala 3 utiliza **FS2 (Functional Streams for Scala)** a través de la librería `fs2-grpc`. A diferencia de los modelos basados en "push", FS2 opera bajo un modelo **pull-based**,. En esta arquitectura, el consumidor es quien "tira" de los datos desde arriba hacia abajo según su capacidad; si el consumidor no solicita más elementos, el productor se detiene automáticamente,. Esto permite procesar gigabytes de datos utilizando apenas unos pocos megabytes de RAM, garantizando la resiliencia y el manejo seguro de recursos mediante el uso de fibras ligeras en lugar de hilos pesados de la JVM,,.

### 2. Sintaxis y Estructura

La integración con `fs2-grpc` transforma radicalmente las firmas de los servicios generados. Mientras que el paradigma clásico devolvía `Future[Res]`, el paradigma funcional genera rasgos (**traits**) polimórficos sobre un tipo de efecto `F[_]`,.

|Tipo de llamada Protobuf|Firma Clásica (Future)|Firma Reactiva (FS2 + IO)|
|:--|:--|:--|
|**RPC Unario**|`def call(req: Req): Future[Res]`|`def call(req: Req, ctx: A): F[Res]`|
|**Streaming (Server)**|`StreamObserver` (Callbacks)|`def call(req: Req, ctx: A): Stream[F, Res]`|
|**Streaming (Bidi)**|`StreamObserver` (Complejo)|`def call(req: Stream[F, Req], ctx: A): Stream[F, Res]`|

**Cambio de paradigma en la implementación:** Se utiliza la mónada **IO** de Cats Effect para encapsular los efectos secundarios de forma pura y segura,. Los flujos se definen como descripciones inmutables (**blueprints**) que solo "cobran vida" cuando se compilan y ejecutan en el punto de entrada de la aplicación,.

### 3. Ejemplo Realista: Servicio de "Live Feed" de Órdenes

Implementamos un servicio que recibe un flujo continuo de solicitudes y devuelve un flujo procesado en tiempo real utilizando `IOApp` y la gestión de recursos mediante `Resource`,.

```scala
import cats.effect.{IO, IOApp, Resource}
import fs2.Stream
import io.grpc.Metadata
import com.rockthejvm.protos.orders._ // Clases generadas por ScalaPB

// 1. Implementación del Servicio con FS2-gRPC
class OrderStreamImpl extends OrderFs2Grpc[IO, Metadata]:
  override def sendOrderStream(
    request: Stream[IO, OrderRequest],
    ctx: Metadata
  ): Stream[IO, OrderReply] =
    // El modelo pull-based garantiza backpressure automático aquí
    request.evalTap(req => IO.println(s"Recibida orden: ${req.orderid}"))
      .map { req =>
        OrderReply(
          orderid = req.orderid,
          total = req.items.map(_.amount).reduceOption(_ + _).getOrElse(0.0)
        )
      }

// 2. Montaje del Servidor usando Resource y IOApp
object LiveFeedServer extends IOApp.Simple:
  import io.grpc.netty.shaded.io.grpc.netty.NettyServerBuilder
  import fs2.grpc.syntax.all._

  // Gestión segura del ciclo de vida del servidor
  val serverResource: Resource[IO, Unit] =
    for {
      serviceDef <- OrderFs2Grpc.bindServiceResource[IO](new OrderStreamImpl)
      _ <- NettyServerBuilder
        .forPort(9999)
        .addService(serviceDef)
        .resource[IO] // Convierte el servidor en un recurso seguro
        .evalMap(server => IO(server.start()))
    } yield ()

  val run: IO[Unit] = serverResource.useForever // Mantiene el servidor vivo indefinidamente
```

### 4. Comparativa: Clásico vs. Reactivo

|Característica|gRPC Clásico (Chapter 31)|gRPC Reactivo (FS2)|
|:--|:--|:--|
|**Control de Flujo**|Manual o inexistente (riesgo de OOM).|**Backpressure automático** (Pull model).|
|**Recursos**|Cierre manual de canales/servidores.|Gestión vía **Resource/Bracket** (Garantizado).|
|**Concurrencia**|Hilos pesados (OS Threads).|**Fibras masivas** (Concurrencia masiva eficiente).|
|**Manejo de Errores**|Excepciones que interrumpen hilos.|**Errores como valores** integrados en el stream.|

### 5. Best Practices (The Scala Way)

1. **Manejo de Recursos con `Resource`:** Al iniciar servidores o clientes gRPC, envuélvelos siempre en un `Resource`. Esto garantiza que los puertos y conexiones se liberen correctamente incluso ante fallos críticos o señales de interrupción del sistema,.
2. **Evitar `handleErrorWith` para Limpieza:** No intentes limpiar recursos (como archivos o sockets) dentro de un bloque de manejo de errores. Utiliza siempre el patrón **bracket** u `onFinalize`, ya que son los únicos que garantizan la liberación ante cualquier tipo de terminación (éxito, error o cancelación),,.
3. **Preferir Streaming Bidireccional para Datos Pesados:** Para transferencias de archivos o logs masivos, utiliza streams en lugar de mensajes unarios gigantes para mantener un uso de memoria constante (MBs para procesar GBs),.
4. **No usar `toList` en Producción:** Nunca intentes convertir un stream infinito o con efectos directamente a una lista (`toList`), ya que esto romperá las semánticas de streaming y provocará un crash por falta de memoria. Utiliza `.compile.drain` o `.compile.toVector` para materializar el efecto de forma controlada,.
5. **Validación en el "Borde":** Utiliza precondiciones (`require`) o modela fallos como valores `Left` antes de entrar en la lógica de procesamiento pesada del stream para fallar rápido y de forma segura.

---

**Metáfora de Ingeniería:** Implementar gRPC con FS2 es como añadir **válvulas de presión inteligentes** a un sistema de **tubos neumáticos de alta velocidad**. Mientras que un sistema simple dispara cápsulas de datos sin control hasta que la tubería estalla (OOM), las "válvulas" de FS2 detectan cuánta presión puede soportar la terminal de destino. Si la terminal se atasca, el flujo se detiene en toda la línea automáticamente (Backpressure), evitando cualquier daño estructural y asegurando que cada paquete llegue intacto solo cuando el sistema está listo para recibirlo.