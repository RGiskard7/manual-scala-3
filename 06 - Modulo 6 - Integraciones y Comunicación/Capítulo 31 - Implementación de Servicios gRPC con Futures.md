## Capítulo 31: Implementación de Servicios gRPC con Futures

### 1. Explicación Teórica

La implementación de gRPC en Scala siguiendo el estilo "clásico" se basa en el uso de **`scala.concurrent.Future`**, la herramienta estándar de la biblioteca de Scala para gestionar la asincronía. En este paradigma, **ScalaPB** actúa como el puente que traduce las definiciones de un archivo `.proto` en artefactos de código nativos y tipados.

Bajo el capó, cada mensaje definido como `message` en Protobuf se convierte en una **`case class`** inmutable que extiende `GeneratedMessage`, proporcionando de forma gratuita métodos de serialización, igualdad estructural y copias funcionales. Por su parte, cada `service` se traduce en un **`trait`** (interfaz) dentro de un objeto compañero (por ejemplo, `DataServiceGrpc.DataService`), donde cada método definido en el contrato RPC devuelve un `Future`. Este enfoque es una envoltura ligera sobre la biblioteca oficial `grpc-java`, lo que lo hace ideal para proyectos que buscan minimizar dependencias externas o que provienen de ecosistemas centrados en Java.

### 2. Sintaxis y Estructura

La firma de los métodos generados sigue un patrón predecible que permite al desarrollador centrarse exclusivamente en la lógica de negocio.

**Firma del método generado:**

```scala
// Definición automática por ScalaPB en el trait del servicio
def procesarDatos(request: DataRequest): Future[DataResponse]
```

Para implementar la lógica, simplemente se debe crear una clase que extienda el trait base generado:

|Componente Protobuf|Artefacto Scala Generado|Uso en Implementación|
|:--|:--|:--|
|`message Request`|`case class Request`|DTO inmutable de entrada.|
|`service MyService`|`trait MyService`|Interfaz a extender para el servidor.|
|`rpc Method`|`def Method(...): Future`|Método asíncrono a sobreescribir.|

**Extensión del Trait Base:**

```scala
import scala.concurrent.Future
import com.miempresa.api.v1.procesador._

// Implementación del servidor extendiendo el trait generado
class ProcesadorImpl extends ProcesadorServiceGrpc.ProcesadorService:
  override def procesarDatos(req: DataRequest): Future[DataResponse] =
    // Lógica asíncrona aquí
    Future.successful(DataResponse("Éxito", true))
```

### 3. Ejemplo Realista

#### A. Implementación del Servidor (`DataService`)

Utilizamos un `ExecutionContext` para delegar la ejecución de las tareas a un pool de hilos.

```scala
import scala.concurrent.{ExecutionContext, Future}
import com.miempresa.api.v1.procesador._

class DataServiceImpl(using ec: ExecutionContext)
  extends ProcesadorServiceGrpc.ProcesadorService:

  override def procesarDatos(request: DataRequest): Future[DataResponse] =
    // Simulamos un procesamiento asíncrono
    Future {
      if request.id.isEmpty then
        throw new IllegalArgumentException("ID no puede estar vacío")
      DataResponse(resultado = s"ID ${request.id} procesado", exitoso = true)
    }.recover {
      case e: Exception => DataResponse(resultado = e.getMessage, exitoso = false)
    }
```

#### B. Cliente (Stub) y Manejo de Respuestas

El cliente utiliza un "Stub" para realizar la llamada remota como si fuera un método local.

```scala
import scala.util.{Success, Failure}
import io.grpc.ManagedChannelBuilder
import com.miempresa.api.v1.procesador._

// 1. Configuración del canal y el stub asíncrono
val channel = ManagedChannelBuilder.forAddress("localhost", 9999).usePlaintext().build()
val stub = ProcesadorServiceGrpc.stub(channel) //

// 2. Llamada asíncrona devolviendo un Future
val request = DataRequest(id = "tx-99")
val respuestaFuture: Future[DataResponse] = stub.procesarDatos(request)

// 3. Manejo del resultado (Opción A: onComplete)
respuestaFuture.onComplete {
  case Success(res) => println(s"Servidor respondió: ${res.resultado}")
  case Failure(ex)  => println(s"Error en la llamada RPC: ${ex.getMessage}")
}

// 4. Composición (Opción B: For-comprehension)
val procesoCompuesto = for
  res1 <- stub.procesarDatos(DataRequest("ID-1"))
  res2 <- stub.procesarDatos(DataRequest("ID-2"))
yield (res1, res2)
```

### 4. Comparativa con Java/Python

|Característica|Python (`asyncio`)|Java (`CompletableFuture`)|Scala (`Future`)|
|:--|:--|:--|:--|
|**Modelo de Ejecución**|Single-threaded Event Loop.|Thread Pool manual o ForkJoin.|**Thread Pool vía `ExecutionContext`**.|
|**Evaluación**|Lazy (necesita `await`).|Eager (inicia al crearse).|**Eager** (inicia inmediatamente).|
|**Interoperabilidad**|Dinámica; riesgo en runtime.|Verbosa; basada en reflexión.|**Estática y sólida** gracias a ScalaPB.|
|**Streaming**|Generadores simples.|`StreamObserver` (callbacks complejos).|Limitado en este modelo; requiere FS2 para complejidad.|

### 5. Best Practices (The Scala Way)

1. **Gestión estricta del `ExecutionContext`:** Evita usar `ExecutionContext.global` en entornos de producción para servicios gRPC. Define pools de hilos dedicados para el manejo de RPC para evitar que tareas intensas de CPU bloqueen hilos de transporte de red.
2. **Fallar el Future, no lanzar excepciones:** Dentro de la lógica del servicio, prefiere devolver un `Future.failed(exception)` o capturar errores con `.recover` en lugar de lanzar excepciones directamente. Esto asegura que el error se propague correctamente a través de la cadena asíncrona sin romper el hilo de ejecución.
3. **Inmutabilidad de Mensajes:** Los mensajes generados por ScalaPB son **thread-safe** por ser inmutables. Utiliza siempre `.copy()` para crear versiones modificadas del mensaje, manteniendo la transparencia referencial.
4. **Validación Precoz:** Valida los campos obligatorios de la solicitud Protobuf inmediatamente al recibirla usando precondiciones. Aunque Protobuf 3 trata casi todo como opcional, tu lógica de negocio debe forzar la integridad antes de iniciar procesos costosos.

---

**Metáfora de Ingeniería:** Implementar gRPC con `Future` es como un **Sistema de Tickets de Encargo** en una cafetería moderna. Tú entregas tu pedido (la solicitud RPC) y el cajero te devuelve un ticket numerado (el `Future`). El ticket es una promesa de que obtendrás tu café cuando esté listo. Mientras tanto, puedes leer el periódico o hablar por teléfono (ejecutar otras tareas). Cuando tu número aparece en la pantalla (completitud del `Future`), simplemente intercambias el ticket por el producto final (la respuesta).