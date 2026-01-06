## Capítulo 31: Kafka Streaming Funcional (fs2-kafka)

### 1. Explicación Teórica

**Apache Kafka** se ha consolidado como el estándar industrial para la mensajería distribuida y el procesamiento de flujos de datos a gran escala, siendo la columna vertebral de arquitecturas de microservicios y pipelines de datos en empresas de primer nivel. En el ecosistema de Scala 3, la librería **fs2-kafka** permite integrar la potencia de Kafka con el modelo de **streaming puramente funcional** de FS2.

Este capítulo representa la culminación del aprendizaje sobre **concurrencia y efectos**, aplicando los conceptos de **Fibras** y la **mónada IO** (vistos en los capítulos 12 y 13) al mundo del procesamiento de eventos en tiempo real. A diferencia de los clientes tradicionales, `fs2-kafka` utiliza un modelo basado en **Pull**, lo que garantiza un mecanismo de **backpressure intrínseco**: el consumidor solo "tira" de los datos de Kafka cuando tiene capacidad de procesamiento, evitando desbordamientos de memoria y garantizando la resiliencia del sistema.

### 2. Sintaxis y Estructura

La integración de Kafka con FS2 se basa en la configuración de tipos seguros y la gestión de recursos mediante `Resource`.

**Componentes y Tipos Principales:**

|Elemento|Tipo / Clase|Propósito|
|:--|:--|:--|
|**Consumer**|`KafkaConsumer[F, K, V]`|Cliente para leer registros de tópicos de Kafka.|
|**Producer**|`KafkaProducer[F, K, V]`|Cliente para enviar registros de forma asíncrona y segura.|
|**Settings**|`ConsumerSettings` / `ProducerSettings`|Configuración de brokers, grupos de consumo y serializadores.|
|**Committable**|`CommittableConsumerRecord`|Registro que incluye la capacidad de confirmar el offset tras el proceso.|

**Configuración Base (sbt):**

```scala
libraryDependencies += "com.github.fd4s" %% "fs2-kafka" % "3.5.1"
```

### 3. Ejemplos de Código Realistas

#### A. Pipeline de Consumo y Procesamiento (The Functional Way)

Este ejemplo demuestra cómo consumir eventos de un tópico, transformarlos y gestionar los offsets de manera segura, utilizando el patrón **bracket** implícito en los recursos de FS2.

```scala
import cats.effect.{IO, IOApp}
import fs2.kafka._
import scala.concurrent.duration._

object KafkaStreamingApp extends IOApp.Simple:
  // Configuración del consumidor
  val consumerSettings = ConsumerSettings[IO, String, String]
    .withBootstrapServers("localhost:9092")
    .withGroupId("grupo-procesamiento-1")
    .withAutoOffsetReset(AutoOffsetReset.Earliest)

  val run: IO[Unit] =
    KafkaConsumer.stream(consumerSettings)
      .subscribeTo("topico-eventos")
      .records
      .mapAsync(16) { committable =>
        // Procesamiento funcional inmutable
        val record = committable.record
        IO.println(s"Procesando: ${record.value}").as(committable.offset)
      }
      .through(commitBatchWithin(500, 15.seconds)) // Commits eficientes en lote
      .compile
      .drain
```

#### B. Serialización con Avro y Vulcan

En entornos industriales, se prefiere **Avro** sobre JSON para garantizar la evolución del esquema y reducir el ancho de banda. Vulcan es la librería que mapea _case classes_ de Scala a esquemas Avro usando el sistema de tipos.

```scala
import lbialy.vulcan._ // Basado en inlines y derivación de Scala 3

case class EventoUsuario(id: String, accion: String) derives Codec

// El esquema se deriva en tiempo de compilación para máxima seguridad de tipos
```

### 4. Comparativa con Java/Python

|Aspecto|Java (Kafka Client)|Python (kafka-python)|Scala (fs2-kafka)|
|:--|:--|:--|:--|
|**Paradigma**|Imperativo (Callbacks/Loops).|Dinámico (Bloqueante por defecto).|**Declarativo (Streaming Funcional)**.|
|**Backpressure**|Manual (Pause/Resume).|Limitada o inexistente.|**Automática (Modelo Pull)**.|
|**Manejo de Errores**|Try-catch que interrumpe hilos.|Excepciones en runtime.|**Errores como Valores Tipados**.|
|**Gestión de Recursos**|Bloques try-with-resources.|Context Managers (`with`).|**Resource/Bracket** (Seguridad total).|

### 5. Best Practices (The Scala Way)

1. **Preferir `mapAsync` sobre `map` para I/O:** Utiliza `mapAsync` o `parEvalMap` para ejecutar efectos de base de datos o llamadas API en paralelo mientras consumes de Kafka, controlando el número de fibras para evitar saturar el sistema.
2. **Offsets: Commits en Lote (`Batching`):** No confirmes (commit) cada mensaje individualmente; usa `commitBatchWithin` para agrupar confirmaciones. Esto mejora drásticamente el rendimiento del broker al reducir las escrituras en el tópico de offsets internos.
3. **Exactly-Once mediante Idempotencia:** El procesamiento _Exactly-once_ real se logra mejor diseñando sumideros (sinks) idempotentes. Si necesitas la semántica nativa de Kafka, configura `withTransactionalId` en el productor, pero ten en cuenta el aumento de latencia.
4. **Inmutabilidad de los Mensajes:** Trata cada registro de Kafka como una **case class inmutable**. Si necesitas transformar el dato, usa `.copy()` para crear una nueva versión, manteniendo la transparencia referencial.
5. **Evitar el "Auto-commit":** Desactiva el auto-commit de Kafka y confía en el flujo de FS2 para manejar los offsets. Esto garantiza que un mensaje no se marque como leído si el procesamiento funcional falla antes de completarse.

---

**Metáfora de Ingeniería:** Apache Kafka es como un **río infinito de pergaminos** que fluye constantemente; usar el cliente de Java o Python es como intentar atrapar pergaminos con las manos mientras corres por la orilla (estado mutable y riesgo de pérdida). Usar **fs2-kafka** es construir una **represa inteligente con compuertas automatizadas**: el río solo entrega pergaminos cuando tus escribas (fibras) están listos para leerlos, y cada pergamino se archiva (commit) solo cuando la traducción ha sido verificada y sellada por el compilador.