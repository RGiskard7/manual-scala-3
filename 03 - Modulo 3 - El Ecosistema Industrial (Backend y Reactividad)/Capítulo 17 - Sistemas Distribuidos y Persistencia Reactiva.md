## Capítulo 17: Sistemas Distribuidos y Persistencia Reactiva

### 1. Explicación Teórica: Coherencia y Resiliencia a Gran Escala

El diseño de sistemas distribuidos y reactivos en Scala, típicamente utilizando Apache Pekko (el _fork_ de código abierto del _toolkit_ Akka), se centra en la capacidad de escalar la concurrencia a través de múltiples máquinas y garantizar que el **estado interno de las entidades de negocio no se pierda** ante fallos (persistencia reactiva).

#### A. Sistemas Distribuidos con Cluster Sharding

Cuando un sistema necesita gestionar miles o millones de entidades con estado (como sesiones de usuario o agregados de Domain-Driven Design), una sola máquina (nodo) es insuficiente. **Cluster Sharding** (Fragmentación de Clúster) es la solución de Pekko para este problema de escalamiento horizontal, permitiendo distribuir estos actores con estado (_entidades_) a través de múltiples nodos.

- **Abstracción de Localización:** El desarrollador envía un mensaje a una entidad utilizando su **identificador lógico** (ej., `userId` o `PersistenceId`), sin necesidad de conocer su ubicación física en el clúster.
- **Principio Single-Writer:** El _Cluster Sharding_ es fundamental para la persistencia, ya que garantiza que solo **una instancia activa** del actor (entidad) para un `PersistenceId` dado exista en un momento dado en todo el clúster. Esto es vital para prevenir escrituras de eventos intercaladas que podrían corromper el estado.

#### B. Persistencia Reactiva con Event Sourcing

La **Persistencia Pekko** es el módulo que permite a los actores recuperar su estado tras un reinicio (debido a un fallo, una caída de nodo, o una supervisión que ordenó un `restart`). El patrón principal para lograr esto es **Event Sourcing** (Eventos Fuente).

- **Eventos Inmutables:** En lugar de guardar el estado actual (modelo CRUD), el actor persistente almacena una secuencia inmutable de **eventos** que representan todos los cambios de estado que han ocurrido. Los eventos se persisten añadiéndolos al almacenamiento (_journal_), y nunca se mutan.
- **Recuperación por Replay:** Si el actor se reinicia, su estado se **reconstruye reejecutando (replay) todos los eventos** almacenados en el _journal_, o desde el último _snapshot_ (punto de control) si existe. Esto garantiza que, incluso después de un fallo, la nueva instancia del actor recupere su memoria con precisión forense.

### 2. Sintaxis: EventSourcedBehavior y Cluster Sharding

#### A. Sintaxis de EventSourcedBehavior (Persistencia de Eventos)

La lógica central para el estado persistente se define mediante el `EventSourcedBehavior[Comando, Evento, Estado]`, el cual requiere cuatro componentes principales para su tipado seguro:

```scala
import org.apache.pekko.persistence.typed.scaladsl.{EventSourcedBehavior, Effect}
import org.apache.pekko.persistence.typed.PersistenceId

// 1. Comando, Evento, Estado (Modelos de Datos Inmutables/ADTs)
object EntidadCuenta {
  // Estado Inmutable (lo que el actor recuerda)
  final case class Estado(balance: BigDecimal)

  // Comandos (lo que el actor recibe y valida)
  sealed trait Comando
  final case class Depositar(monto: BigDecimal, replyTo: ActorRef[Respuesta]) extends Comando

  // Eventos (lo que se persiste)
  sealed trait Evento
  final case class Depositado(monto: BigDecimal) extends Evento

  // ... Respuesta ...

  // 2. Lógica para manejar Comandos -> Produce Efectos (Persistencia)
  val commandHandler: (Estado, Comando) => Effect[Evento, Estado] =
    (estado, comando) => comando match {
      case Depositar(monto, replyTo) =>
        // Persiste el evento antes de actualizar el estado o responder
        Effect.persist(Depositado(monto))
              // Efecto secundario: Responder solo después de la persistencia exitosa
              .thenReply(replyTo)(_ => StatusReply.Success("Depósito exitoso"))
    }

  // 3. Lógica para aplicar Eventos -> Produce Nuevo Estado
  val eventHandler: (Estado, Evento) => Estado =
    (estado, evento) => evento match {
      // SOLO actualización de estado, NUNCA efectos secundarios
      case Depositado(monto) => estado.copy(balance = estado.balance + monto)
    }

  // 4. Comportamiento Raíz
  def apply(id: String): Behavior[Comando] =
    EventSourcedBehavior[Comando, Evento, Estado](
      persistenceId = PersistenceId.ofUniqueId(id), // ID único
      emptyState = Estado(BigDecimal(0)),          // Estado inicial
      commandHandler = commandHandler,
      eventHandler = eventHandler
    )
}
```

#### B. Sintaxis de Cluster Sharding (Distribución)

El _Cluster Sharding_ se inicializa registrando el `Behavior` de la entidad en el `ActorSystem` de cada nodo. Luego, el sistema de mensajería utiliza una referencia especial (`EntityRef`) para encontrar la ubicación de la entidad en el clúster.

```scala
import org.apache.pekko.cluster.sharding.typed.scaladsl.{ClusterSharding, Entity, EntityTypeKey}
import org.apache.pekko.actor.typed.ActorSystem

// Asumimos que 'system' es el ActorSystem con Cluster activado.
val system: ActorSystem[_] = ???

// 1. Definir la clave de la entidad
val TipoCuenta = EntityTypeKey[EntidadCuenta.Comando]("CuentaBancaria")

// 2. Inicializar el Cluster Sharding en el nodo.
ClusterSharding(system).init(
  Entity(TipoCuenta) { entityContext =>
    // entityContext.entityId es el ID lógico de la entidad (ej: "Cuenta-123")
    EntidadCuenta(entityContext.entityId)
  }
)

// 3. Obtener una referencia de entidad para comunicación (EntityRef)
// Esto abstrae la ubicación real de la entidad en el clúster.
val cuentaRef = ClusterSharding(system).entityRefFor(TipoCuenta, "Cuenta-ABC-123")

// Enviar un comando a la entidad, el sharding lo ruteará.
cuentaRef ! EntidadCuenta.Depositar(100.00, ...)
```

### 3. Ejemplos de Código: El Flujo de Event Sourcing

El ejemplo ilustra el flujo de comando a evento y la naturaleza de los _side effects_ (efectos secundarios) en el `commandHandler`.

```scala
import org.apache.pekko.persistence.typed.scaladsl.Effect

// Componentes del Event Sourcing (predefinidos en la Sección 2A)

// --- EJEMPLO DE LÓGICA DE NEGOCIO ---

// 1. El actor recibe un comando (ej: Depositar)
// Llama al commandHandler: (Estado(500), Depositar(100, replyTo))

val commandHandlerEjemplo: (EntidadCuenta.Estado, EntidadCuenta.Comando) => Effect[EntidadCuenta.Evento, EntidadCuenta.Estado] =
    (estado, comando) => comando match {
      case EntidadCuenta.Depositar(monto, replyTo) =>
        // Validación: (Si el monto es positivo, continúa)
        if (monto > 0)
          // Efecto 1: Persistir el evento (se envía al journal)
          Effect.persist(EntidadCuenta.Depositado(monto))
                .thenRun { nuevoEstado =>
                  // Side Effect (Ej: Logear o notificar)
                  println(s"Nuevo balance tras persistencia: ${nuevoEstado.balance}")
                }
                // Efecto 2: Responder al solicitante (solo después del éxito de la persistencia)
                .thenReply(replyTo)(_ => StatusReply.Success("OK"))
        else
          // Efecto 3: No persistir y solo responder con error (read-only command)
          Effect.reply(replyTo)(StatusReply.Error("Monto inválido"))
    }

// 2. Si Effect.persist fue exitoso, se llama al eventHandler:
// (Estado(500), Depositado(100)) -> Nuevo Estado(600)

val eventHandlerEjemplo: (EntidadCuenta.Estado, EntidadCuenta.Evento) => EntidadCuenta.Estado =
    (estado, evento) => evento match {
      case EntidadCuenta.Depositado(monto) =>
        // Estado anterior: Estado(balance = 500)
        // Nuevo estado: Estado(balance = 600)
        estado.copy(balance = estado.balance + monto)
    }

// 3. Recuperación: Si el actor se reinicia, el sistema aplica:
// emptyState (0) -> eventHandler(0, Depositado(100)) -> eventHandler(100, Depositado(50)) -> ...
// El estado se reconstruye secuencialmente.
```

### 4. Comparativa con Java y Python

La arquitectura de efectos puros y actores de Scala/Pekko ofrece una solución integrada para problemas de concurrencia y estado distribuido que en Java o Python requieren librerías de infraestructura externa y coordinación manual.

|Aspecto|Java (Spring/JEE)|Python (Estructuras Dinámicas)|Scala (Pekko Actor Model)|
|:--|:--|:--|:--|
|**Escalamiento de Estado**|Alto acoplamiento con la infraestructura (Bases de datos, caches distribuidos). Requiere coordinación manual (ej. Redis, ZooKeeper).|N/A (Generalmente no se usa para sistemas distribuidos de estado).|**Cluster Sharding:** Distribución transparente y gestionada de entidades con estado, asegurando un escritor único (Single-Writer Principle).|
|**Tolerancia a Fallos**|Bloques `try-catch` y excepciones. La interrupción por excepción termina el hilo.|`try-except`.|**Supervisión Jerárquica ("Let-It-Crash"):** Los fallos se aíslan al actor y el supervisor decide reiniciar o reanudar.|
|**Recuperación de Estado**|Pérdida de estado en memoria en caso de reinicio de la aplicación o hilo.|Pérdida de estado en memoria.|**Event Sourcing:** El estado se reconstruye mediante la **reproducción de eventos** inmutables.|
|**Resiliencia vs. Kubernetes**|Kubernetes ofrece tolerancia a fallos a nivel de contenedor (coarse-grained). La pérdida de estado es un problema para aplicaciones _stateful_.|N/A|**Sinergia:** Pekko ofrece resiliencia a nivel de aplicación (fine-grained) (reinicio de actores/Fibers). Permite a los servicios _stateful_ comportarse _como si fueran stateless_ en Kubernetes.|

### 5. Best Practices (_The Scala Way_)

1. **Modelado con Event Sourcing:** Para cualquier entidad cuyo estado deba sobrevivir a fallos o reinicios, utiliza **`EventSourcedBehavior`**. Este patrón es superior a la persistencia de estado duradero (Durable State) cuando se requiere trazabilidad y auditoría de la entidad.
2. **Pureza en `eventHandler`:** El `eventHandler` es la única fuente de verdad para el estado y **no debe contener efectos secundarios** (I/O, logs, llamadas a otros actores). La única responsabilidad del `eventHandler` es crear un **nuevo estado inmutable** a partir del estado antiguo y el evento.
3. **Encadenamiento de Efectos Secundarios:** Los efectos secundarios (como enviar una respuesta al remitente o logear una acción) deben encadenarse **después** de la persistencia del evento exitosa, utilizando combinadores como **`Effect.persist(...).thenRun(...)`** o **`Effect.thenReply(...)`**.
4. **Cluster Sharding para el Escalado:** Si la aplicación se ejecuta en un clúster, el uso de **`ClusterSharding`** es idiomático para distribuir la carga y garantizar el **Single-Writer Principle** necesario para la integridad de `EventSourcedBehavior`.

> La Persistencia Reactiva y los Sistemas Distribuidos de Pekko son una manifestación del principio de **inmutabilidad aplicada a la infraestructura**. Al tratar los cambios de estado como un registro inmutable (Event Sourcing) y al distribuirlos bajo la estricta vigilancia del Cluster Sharding, se construye un sistema que no teme a los fallos, sino que los utiliza como una oportunidad para demostrar su capacidad de **auto-recuperación y escalabilidad**.