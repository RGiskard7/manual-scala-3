## Capítulo 15: Modelo de Actores: Pekko Fundamentos y Patrones

### 1. Explicación Teórica: El Modelo de Actores para la Concurrencia Masiva

El **Modelo de Actores** es un paradigma de concurrencia diseñado para simplificar la creación de sistemas distribuidos, concurrentes y tolerantes a fallos, abordando las dificultades históricas de la gestión manual de hilos y bloqueos en entornos de la JVM.

Scala es el lenguaje de implementación de muchos _frameworks_ importantes, incluyendo Akka, que históricamente popularizó este modelo. Tras un cambio de licencia de Akka a un modelo comercial en 2022, la comunidad _open source_ creó **Apache Pekko** como un _fork_ de la última versión bajo licencia Apache 2.0, garantizando el acceso a este modelo de programación.

#### Principios Fundamentales del Actor Model

1. **Aislamiento y Estado Privado:** Un **actor** es una unidad de computación ligera y autónoma que encapsula su propio estado interno y comportamiento. Los actores **no comparten memoria mutable**. Esta **inmutabilidad por defecto** es clave, ya que elimina toda una clase de problemas complejos y propensos a errores relacionados con la corrupción de datos y las condiciones de carrera (_race conditions_) en sistemas concurrentes.
2. **Comunicación Asíncrona:** Los actores se comunican exclusivamente enviándose **mensajes asíncronos** a un buzón dedicado (_mailbox_). El actor receptor procesa los mensajes uno a la vez en secuencia, lo que previene cuellos de botella y garantiza que el sistema siga siendo reactivo.
3. **Tolerancia a Fallos y Supervisión:** Los actores se organizan en una **jerarquía**. Si un actor hijo falla (por una excepción inesperada), el fallo se contiene y se notifica a su actor **supervisor** (padre). El supervisor implementa una estrategia predefinida (_Supervision Strategy_): reanudar, reiniciar (borrando el estado interno) o detener permanentemente el actor. Esta es la filosofía **"Let-It-Crash"** (dejar que falle), que permite sistemas auto-recuperables.

#### Pekko Typed (Actores Tipados)

La versión moderna de Pekko (Pekko Typed) utiliza el riguroso sistema de tipos de Scala para que los actores sean **seguros en tipos**. El tipo de mensaje que un actor acepta está explícitamente declarado en su definición (`Behavior[Command]`), y el compilador garantiza que solo se le envíen mensajes de ese tipo.

### 2. Sintaxis y Patrones Básicos

En Pekko Typed, un actor se define por su **comportamiento** (`Behavior`), no por una clase que implementa un método genérico `receive`. El comportamiento define cómo debe reaccionar el actor a los mensajes.

#### A. Definición del Protocolo (Mensajes)

La forma idiomática es utilizar un **Tipo de Suma (ADT)** para definir todos los comandos posibles que un actor puede recibir, utilizando un `sealed trait` (o `enum` en Scala 3) como tipo base, y `case class` para cada mensaje. Esto garantiza la exhaustividad en el _pattern matching_.

```scala
// 1. Definición del ADT para los comandos que el actor acepta
object ActorDeTareas {
  sealed trait Comando

  // Mensaje con datos y un 'replyTo' para la respuesta
  final case class AñadirTarea(tareaId: String, descripcion: String,
                               replyTo: ActorRef[Respuesta]) extends Comando

  // Mensaje sin datos, solo la acción
  case object ObtenerEstado extends Comando

  sealed trait Respuesta // ADT para posibles respuestas
  final case class TareaAñadida(tareaId: String)
}
```

#### B. Definición del Actor (Behavior)

El comportamiento inicial de un actor se construye típicamente usando `Behaviors.setup` (para inicialización o _context access_) o `Behaviors.receiveMessage`.

```scala
import org.apache.pekko
import pekko.actor.typed.scaladsl.{Behaviors, ActorContext}
import pekko.actor.typed.{ActorRef, Behavior}

object ActorDeTareas {
  import ActorDeTareas._ // Importar los comandos definidos arriba

  // El estado interno inmutable del actor (Modelado con una case class)
  case class Estado(tareas: Map[String, String])

  // La función de fábrica que define el comportamiento
  def apply(): Behavior[Comando] =
    Behaviors.setup { context: ActorContext[Comando] =>
      context.log.info("Actor de Tareas iniciado.")
      // Inicia con un estado de tareas vacío
      manejarTareas(Estado(Map.empty))
    }

  // Comportamiento funcional: El estado es un parámetro inmutable (FP style)
  private def manejarTareas(estado: Estado): Behavior[Comando] =
    Behaviors.receiveMessage { mensaje: Comando =>
      mensaje match {
        case AñadirTarea(id, desc, responderA) =>
          context.log.info("Recibido comando AñadirTarea: {}", id)

          // Crea un nuevo estado inmutable
          val nuevoEstado = estado.copy(tareas = estado.tareas + (id -> desc))

          // Responde al remitente (necesita el replyTo)
          responderA ! TareaAñadida(id)

          // Devuelve el nuevo comportamiento con el estado actualizado
          manejarTareas(nuevoEstado)

        case ObtenerEstado =>
          context.log.info("Estado actual: {} tareas", estado.tareas.size)
          // Si no cambia el estado o comportamiento, se devuelve Behaviors.same
          Behaviors.same
      }
    }
}
```

#### C. Comunicación y Referencias

La comunicación se realiza mediante el operador `!` (bang o _tell_), y es asíncrona. Para un patrón de solicitud-respuesta (_request-response_), la petición debe incluir explícitamente una referencia de actor (`ActorRef[T]`) a la que enviar la respuesta, como `replyTo`.

```scala
// Ejemplo de envío de un comando desde otro actor o desde el ActorSystem
val actorRef: ActorRef[ActorDeTareas.Comando] = ??? // Obtención de la referencia

// Se crea una sonda (probe) para recibir la respuesta (Técnica de testing/integración)
val sondaRespuesta: ActorRef[ActorDeTareas.Respuesta] = ???

// Envío asíncrono del mensaje:
actorRef ! ActorDeTareas.AñadirTarea("001", "Implementar módulo", sondaRespuesta)
```

### 3. Patrones Avanzados: Estado y Tolerancia a Fallos

#### A. Manejo de Estado: Estilos Funcional vs. OO

Pekko permite dos estilos para manejar el estado interno de un actor:

1. **Estilo Funcional (FP Style):** El estado se pasa como un **parámetro inmutable** a la función que define el `Behavior`. Cuando el actor procesa un mensaje que modifica el estado, devuelve una llamada recursiva a la misma función (`manejarTareas(nuevoEstado)`), la cual se convierte en el **nuevo comportamiento** para el siguiente mensaje. **Esta es la forma preferida en Scala idiomático**, ya que es inherentemente inmutable.
2. **Estilo Orientado a Objetos (OO Style):** El estado se mantiene como un campo mutable (`var`) dentro de la clase que define el comportamiento. Al recibir un mensaje, el actor muta la variable (`var`) y devuelve `Behaviors.same`. Aunque es más familiar para los desarrolladores de Java, el encapsulamiento del actor garantiza que sigue siendo _thread-safe_ porque solo un mensaje se procesa a la vez.

#### B. Tolerancia a Fallos y Jerarquía (Supervisión)

La jerarquía de actores es el mecanismo de **tolerancia a fallos** de Pekko. El actor padre (supervisor) es el responsable de decidir cómo manejar los fallos del hijo. El `Behaviors.supervise` se utiliza para envolver el comportamiento del actor y aplicar la estrategia.

```scala
import pekko.actor.typed.SupervisorStrategy
import scala.concurrent.duration._

// 1. Definición del actor hijo que puede fallar
object ServicioHijo {
  def apply(): Behavior[String] = Behaviors.setup { context =>
    context.log.info("Servicio Hijo iniciado.")
    Behaviors.receiveMessage {
      case "fallar" => throw new RuntimeException("Error inesperado en servicio.")
      case _ => Behaviors.same
    }
  }
}

// 2. Actor Padre que supervisa y aplica una estrategia
object SupervisorServicio {
  def apply(): Behavior[String] =
    Behaviors.setup { context =>
      // Estrategia: Reiniciar el actor hijo en caso de cualquier fallo (Exception)
      val estrategia = SupervisorStrategy.restart.withLimit(maxNrOfRetries = 10,
                                                            withinTimeRange = 10.seconds)

      val hijo = context.spawn(
        Behaviors.supervise(ServicioHijo()).onFailure[Exception](estrategia),
        name = "HijoSupervisado"
      )

      Behaviors.receiveMessage {
        case "pruebaFallo" => hijo ! "fallar"; Behaviors.same
        case _ => Behaviors.unhandled
      }
    }
}
```

En este ejemplo, si el `ServicioHijo` recibe "fallar", lanzará una excepción, pero el `SupervisorServicio` lo reiniciará hasta 10 veces en un período de 10 segundos, siguiendo la estrategia definida. El padre solo reinicia la instancia del hijo, manteniendo el resto del sistema en funcionamiento.

### 4. Comparativa con Java y Python

|Aspecto|Java (Modelos Thread/Lock)|Python (Event Loop/Asyncio)|Scala (Pekko/Actor Model)|
|:--|:--|:--|:--|
|**Modelo de Concurrencia**|Hilos pesados (_OS Threads_). Uso de `synchronized` y `Lock` para sincronización.|Bucle de eventos, _coroutines_ (Python 3+).|**Actores/Fibras Ligeras.** Se ejecuta concurrencia masiva en un _pool_ de hilos reducido.|
|**Manejo de Estado Compartido**|**Mutable.** Propenso a _race conditions_ y _deadlocks_.|Mutable por defecto.|**Inmutable por defecto.** El estado está **encapsulado** en el actor; no hay memoria compartida, eliminando bloqueos.|
|**Manejo de Errores**|Excepciones (`try/catch`) que interrumpen la ejecución del hilo.|Excepciones/manejo de errores.|**Supervisión Jerárquica.** Los fallos (excepciones inesperadas) se aíslan y se gestionan reactivamente mediante estrategias.|
|**Protocolo de Comunicación**|Llamadas a métodos sincrónicas.|Llamadas a funciones o métodos (usualmente asíncronos en Asyncio).|**Mensajes Asíncronos** (`!`). Requiere un `ActorRef` explícito (`replyTo`) para las respuestas, lo que garantiza el tipado.|

### 5. Best Practices (_The Scala Way_)

1. **Modelar Comandos como ADTs:** Utiliza `sealed trait`/`enum` y `case class` para definir el protocolo de mensajes de tu actor. Esto garantiza que los mensajes sean **inmutables** y permite al compilador verificar si se cubren todos los casos en el _pattern matching_ del actor.
2. **Estado Funcional por Defecto:** Aunque Pekko permite el uso de `var` encapsuladas, la forma idiomática y recomendada en Scala es definir el comportamiento como una **función recursiva que acepta y devuelve el estado como un parámetro inmutable** (`Behavior[Comando] = manejarTareas(nuevoEstado)`).
3. **Comunicación Explícita:** Nunca confíes en un `sender()` implícito. Si el actor debe responder, el mensaje de solicitud (`Command`) debe incluir explícitamente el campo `replyTo: ActorRef[Respuesta]`, asegurando la seguridad de tipos para la respuesta.
4. **Diseño Jerárquico para la Resiliencia:** Estructura la aplicación en una jerarquía lógica de actores. Define la **Supervision Strategy** (`Behaviors.supervise`) en el actor padre para gestionar los fallos de sus hijos (ej. `SupervisorStrategy.restart`), aplicando el principio "Let-It-Crash".
5. **Cluster Sharding para Escalabilidad:** Para sistemas distribuidos que gestionan muchas entidades con estado (como sesiones de usuario o entidades DDD), utiliza **Cluster Sharding** para distribuir transparentemente los actores por el clúster, sin que el remitente necesite conocer su ubicación física.

> El Actor Model es una **fábrica de resiliencia**: en lugar de intentar evitar que las máquinas fallen, asume que fallarán y diseña un sistema que puede reiniciar instantáneamente las unidades de trabajo más pequeñas (actores) bajo la estricta vigilancia de un supervisor, protegiendo la integridad del sistema completo.