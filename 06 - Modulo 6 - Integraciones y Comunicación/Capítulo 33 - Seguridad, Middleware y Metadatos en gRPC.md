## Capítulo 33: Seguridad, Middleware y Metadatos en gRPC

### 1. Explicación Teórica

Una vez que un servicio gRPC está operativo (ya sea con Futures o FS2), el siguiente desafío en un entorno industrial es la gestión de **aspectos transversales** (cross-cutting concerns): autenticación, autorización, logging, métricas y tracing distribuido. En gRPC, estos aspectos no se mezclan con la lógica de negocio del servicio, sino que se implementan mediante **Interceptors**.

Los **Interceptors** funcionan como el patrón "Middleware" en servidores web tradicionales: son capas de cebolla que envuelven la ejecución del servicio. Cada solicitud debe atravesar estas capas antes de llegar a la implementación del método RPC.

Además, gRPC no utiliza "Headers HTTP" en el sentido tradicional para enviar tokens o credenciales, sino una abstracción llamada **Metadata**. La Metadata es un mapa de clave-valor binario o de texto que viaja junto con la llamada RPC, permitiendo inyectar información de contexto (como JWTs o Tracing IDs) sin contaminar la firma de los métodos del servicio.

### 2. Sintaxis y Estructura

Para implementar seguridad y middleware, interactuamos con las interfaces nativas de `io.grpc`.

**Conceptos Clave:**

|**Componente**|**Descripción**|**Equivalente HTTP**|
|---|---|---|
|**ServerInterceptor**|Interfaz para interceptar llamadas entrantes.|Middleware / Filter|
|**Metadata**|Mapa de pares clave-valor (ASCII o Binario).|Headers|
|**Metadata.Key**|Llave tipada para acceder a valores en la Metadata.|Header Name|
|**Context**|Almacenamiento local del hilo (ThreadLocal) para pasar datos del interceptor al servicio (ej. UserID).|Request Attribute|

### 3. Ejemplos de Código Realistas

#### A. Implementación de un AuthInterceptor

Este interceptor (basado en el código original del proyecto) inspecciona la Metadata de cada llamada para buscar un token de autorización. Si el token es inválido, rechaza la llamada inmediatamente (Fail Fast) antes de que toque la lógica de negocio.

Scala

```scala
import io.grpc.{Metadata, ServerInterceptor, ServerCall, ServerCallHandler, Status}

// Definimos la clave esperada en la metadata (similar a "Authorization" en HTTP)
// Metadata.ASCII_STRING_MARSHALLER indica que el valor es texto plano
val AUTH_TOKEN_KEY: Metadata.Key[String] = 
  Metadata.Key.of("auth-token", Metadata.ASCII_STRING_MARSHALLER)

class AuthInterceptor extends ServerInterceptor:
  override def interceptCall[Req, Resp](
    call: ServerCall[Req, Resp],
    headers: Metadata,
    next: ServerCallHandler[Req, Resp]
  ): ServerCall.Listener[Req] =
    // 1. Extraer el token de los metadatos
    val token = headers.get(AUTH_TOKEN_KEY)

    // 2. Validar el token (Lógica simplificada para el ejemplo)
    if token == "secreto-super-seguro" then
      // 3. Éxito: Pasar el control al siguiente eslabón de la cadena (el servicio)
      // Aquí también podríamos inyectar el UserID en el Contexto
      next.startCall(call, headers)
    else
      // 4. Fallo: Cerrar la llamada inmediatamente con estado UNAUTHENTICATED
      call.close(Status.UNAUTHENTICATED.withDescription("Token inválido o ausente"), headers)
      
      // Retornar un listener vacío (no procesamos nada más)
      new ServerCall.Listener[Req]() {}
```

#### B. Registro del Interceptor en el Servidor

Un interceptor no hace nada si no se "cablea" al servicio. Es fundamental utilizar `ServerInterceptors.intercept` para envolver la definición del servicio.

Scala

```scala
import io.grpc.ServerInterceptors
import io.grpc.netty.shaded.io.grpc.netty.NettyServerBuilder
import com.miempresa.v1.usuarios.UsuarioServiceGrpc // Tu trait generado

object SecureServer extends App:
  
  // 1. Instanciamos la lógica de negocio (del Cap 31 o 32)
  val servicioBase = new UsuarioServiceImpl()
  
  // 2. Envolvemos el servicio con el Interceptor
  // El orden importa: los interceptores se ejecutan de fuera hacia adentro
  val servicioSeguro = ServerInterceptors.intercept(servicioBase, new AuthInterceptor)

  // 3. Montamos el servidor con el servicio ya interceptado
  val server = NettyServerBuilder
    .forPort(9999)
    .addService(servicioSeguro)
    .build()
    .start()

  println("Servidor gRPC Seguro iniciado en puerto 9999")
  server.awaitTermination()
```

### 4. Comparativa: Middleware en gRPC vs HTTP

|**Característica**|**HTTP / REST (ej. http4s/Play)**|**gRPC (Interceptors)**|
|---|---|---|
|**Formato de Headers**|Texto plano (Strings).|**Metadata Tipada** (puede ser binaria).|
|**Propagación**|Manual o vía librerías de headers.|**Context** automático (ThreadLocal seguro).|
|**Flujo de Error**|Códigos de estado HTTP (401, 403, 500).|**Status Codes gRPC** (Status.UNAUTHENTICATED, etc.).|
|**Posición**|Middlewares globales o por ruta.|Envuelven la **Definición del Servicio** completo.|

### 5. Best Practices (The Scala Way)

1. **Fail-Fast en Seguridad:** El interceptor debe ser la primera línea de defensa. Si una solicitud no tiene credenciales, recházala con `call.close(...)` y `Status.UNAUTHENTICATED`. No permitas que la llamada consuma recursos del servidor (CPU/Memoria) instanciando la lógica de negocio.
    
2. **Nunca Token en Texto Plano:** Aunque el ejemplo usa un string simple, en producción gRPC debe correr siempre sobre **TLS/SSL**. Los tokens Bearer o API Keys viajan en la Metadata y son visibles si no se cifra el canal de transporte.
    
3. **Uso de Context Keys:** Si el interceptor valida al usuario, no pases el `userId` como un argumento más al método del servicio. Utiliza `io.grpc.Context` para almacenar el usuario autenticado y recupéralo dentro del servicio de forma estática y segura.
    
4. **Separación de Responsabilidades:** No mezcles autenticación (¿quién eres?) con logging o métricas. Crea múltiples interceptores pequeños (`AuthInterceptor`, `LoggingInterceptor`, `PrometheusInterceptor`) y encadénalos. Esto facilita el testing unitario de cada pieza de infraestructura.
    

---

Metáfora de Ingeniería:

Un Interceptor en gRPC funciona como el Control de Seguridad de un Aeropuerto.

- El **Pasajero** (la Request) y su **Equipaje** (el Payload) llegan a la terminal.
    
- Antes de poder acercarse a la **Puerta de Embarque** (el Servicio/Método), deben pasar por el arco de seguridad.
    
- El agente de seguridad (Interceptor) verifica el **Pasaporte y Tarjeta de Embarque** (Metadata).
    
- Si todo está en orden, el pasajero pasa. Si no, es detenido ahí mismo y escoltado a la salida (Call Closed), sin importar cuán importante sea su viaje, protegiendo así la integridad de la zona estéril (el Servidor).