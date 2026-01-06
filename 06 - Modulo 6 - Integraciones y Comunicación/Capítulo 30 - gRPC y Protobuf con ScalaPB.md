## Capítulo 30: gRPC y Protobuf con ScalaPB

### 1. Explicación Teórica

**gRPC** es un marco de trabajo de alto rendimiento para llamadas a procedimientos remotos (RPC) que utiliza **HTTP/2** para el transporte y **Protocol Buffers (Protobuf)** como lenguaje de definición de interfaz. En entornos industriales, gRPC es fundamental para la arquitectura de microservicios, ya que permite definir contratos estrictos y tipados que facilitan la interoperabilidad entre lenguajes y garantizan la eficiencia en la transmisión de datos binarios. A diferencia de las APIs REST tradicionales basadas en JSON, gRPC reduce drásticamente el uso de CPU y ancho de banda al evitar el overhead de la serialización de texto.

En el ecosistema de **Scala 3**, la integración con gRPC se logra principalmente a través de **ScalaPB**, que traduce las definiciones de Protobuf en **Case Classes** e infraestructuras de servicio nativas. Este capítulo se conecta con los conceptos previos de **Efectos Puros** y **Streaming**, ya que utilizaremos **fs2-grpc** para integrar gRPC con el modelo de evaluación _pull-based_ de **FS2**. Esta combinación permite gestionar flujos de datos masivos con **backpressure automático** y concurrencia basada en **fibras**, asegurando que los servicios sean resilientes y reactivos ante cargas masivas.

### 2. Sintaxis y Estructura

La configuración de un proyecto con ScalaPB requiere definir los mensajes en archivos `.proto` y configurar el compilador en `build.sbt`. ScalaPB genera automáticamente los codecs necesarios aprovechando la potencia del sistema de tipos de Scala 3.

**Componentes Principales:**

|Elemento|Propósito|Relación en Scala|
|:--|:--|:--|
|**Message**|Define la estructura del dato binario.|Genera una **Case Class** inmutable.|
|**Service**|Define los endpoints y el tipo de comunicación.|Genera un **Trait** con métodos de efecto.|
|**Stream**|Define flujos unidireccionales o bidireccionales.|Se mapea a `fs2.Stream[F, A]`.|

**Configuración básica en `build.sbt`:**

```scala
// Habilita la generación de código ScalaPB
Compile / PB.targets := Seq(
  scalapb.gen(grpc = true) -> (Compile / sourceManaged).value,
  // Genera infraestructura compatible con fs2-grpc
  fs2grpclib.gen(grpc = true) -> (Compile / sourceManaged).value
)
```

### 3. Ejemplos de Código Realistas

#### A. Definición del Contrato (`usuarios.proto`)

Definimos un servicio de gestión de usuarios y un flujo de eventos en tiempo real.

```scala
syntax = "proto3";
package com.miempresa.v1;

message UserRequest { string id = 1; }
message UserResponse { string nombre = 1; bool activo = 2; }

message Evento { string tipo = 1; string timestamp = 2; }

service UsuarioService {
  // RPC Simple (Unario)
  rpc GetUsuario(UserRequest) returns (UserResponse);
  // Streaming Bidireccional
  rpc StreamEventos(stream Evento) returns (stream Evento);
}
```

#### B. Implementación del Servicio con `fs2-grpc`

Utilizamos la mónada **IO** de **Cats Effect** para manejar los efectos secundarios de forma segura.

```scala
import cats.effect.IO
import fs2.Stream
import com.miempresa.v1.usuarios._

class UsuarioServiceImpl extends UsuarioServiceFs2Grpc[IO, Any]:
  // Implementación unaria
  def getUsuario(request: UserRequest, ctx: Any): IO[UserResponse] =
    IO.pure(UserResponse(nombre = s"Usuario-${request.id}", activo = true))

  // Streaming bidireccional con FS2
  def streamEventos(request: Stream[IO, Evento], ctx: Any): Stream[IO, Evento] =
    request.evalTap(evt => IO.println(s"Recibido: ${evt.tipo}"))
      .map(evt => evt.copy(tipo = s"Procesado: ${evt.tipo}")) // Transformación pura
```

#### C. Servidor con Autenticación (Interceptors)

Para añadir seguridad, gRPC utiliza interceptores que actúan como capas intermedias para validar credenciales.

```scala
import io.grpc.{Metadata, ServerInterceptor, ServerCall, ServerCallHandler}

class AuthInterceptor extends ServerInterceptor:
  def interceptCall[Req, Resp](
    call: ServerCall[Req, Resp],
    headers: Metadata,
    next: ServerCallHandler[Req, Resp]
  ): ServerCall.Listener[Req, Resp] =
    val token = headers.get(Metadata.Key.of("auth-token", Metadata.ASCII_STRING_MARSHALLER))
    if token == "secreto" then next.startCall(call, headers)
    else throw new RuntimeException("No autorizado") // Filosofía Let-It-Crash
```

### 4. Comparativa con Java/Python

|Aspecto|Java (gRPC Java)|Python (gRPC Python)|Scala (ScalaPB + FS2)|
|:--|:--|:--|:--|
|**Modelo de Concurrencia**|`Futures` o hilos pesados del SO.|Bucle de eventos (`Asyncio`).|**Fibras ligeras** (IO) con concurrencia masiva.|
|**Streaming**|Basado en `StreamObserver` (callbacks complejos).|Generadores de Python.|**Streams declarativos** (FS2) con backpressure nativo.|
|**Seguridad de Tipos**|Alta, pero verbosa en la construcción de mensajes.|Nula/Baja (Tipado dinámico).|**Exhaustiva**; genera ADTs y Case Classes para Pattern Matching.|
|**Manejo de Errores**|Excepciones que interrumpen el flujo.|Excepciones.|**Errores como valores** (Either/IO) integrados en el flujo.|

### 5. Best Practices (The Scala Way)

1. **Centralizar la Verdad en el `.proto`**: Trata el archivo de Protocol Buffers como la única fuente de verdad (Code as Data) para generar tanto el servidor como el cliente y la documentación.
2. **Manejo de Recursos con `Resource`**: Al iniciar el servidor gRPC, envuélvelo en un `Resource` de Cats Effect para garantizar que el puerto se cierre correctamente incluso ante fallos críticos.
3. **Preferir Streaming para Datos Pesados**: Para transferencias de archivos o logs masivos, utiliza streams bidireccionales en lugar de mensajes unarios gigantes para mantener un uso de memoria constante (MBs de RAM para GBs de datos).
4. **Inmutabilidad Rigurosa**: Nunca intentes mutar los mensajes generados por ScalaPB; utiliza el método `.copy()` para crear nuevas versiones de los mensajes, manteniendo la transparencia referencial.
5. **Validación en el "Borde"**: Valida los campos de los mensajes de entrada usando **precondiciones** (`require`) o modelando los resultados fallidos como valores `Left` antes de procesar la lógica de negocio.

---

**Metáfora de Ingeniería:** Implementar gRPC con ScalaPB es como sustituir el envío de **cartas escritas a mano (JSON)** por un sistema de **tubos neumáticos de alta velocidad**. Las cartas pueden ser leídas por cualquiera y son lentas de procesar, mientras que los tubos neumáticos transportan cápsulas herméticas (binario) que solo encajan en terminales específicos (contratos de Protobuf), garantizando que el paquete llegue intacto, a la velocidad del sonido y sin riesgo de que el contenido se confunda por el camino.