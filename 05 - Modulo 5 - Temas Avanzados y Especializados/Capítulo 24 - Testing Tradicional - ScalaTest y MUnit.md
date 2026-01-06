## Capítulo 24: Testing Tradicional - ScalaTest y MUnit

### 1. Explicación Teórica

El ecosistema de **testing en Scala 3** se caracteriza por una transición desde herramientas pesadas y cargadas de DSLs complejos hacia marcos de trabajo más ligeros que priorizan el uso de "Scala puro". En entornos industriales, el testing no es solo una fase de verificación, sino un componente crítico para catch bugs tempranos, mejorar la calidad del código y facilitar la colaboración técnica en sistemas distribuidos de alta complejidad.

La arquitectura moderna de pruebas en Scala se sustenta en la coexistencia de dos pilares: **ScalaTest**, el estándar de la industria que ofrece un enfoque de "baterías incluidas", y **MUnit**, un marco de trabajo minimalista y rápido diseñado para integrarse estrechamente con el **Scala Toolkit**. Mientras que en lenguajes como Java o Python las pruebas suelen ser imperativas y basadas en excepciones, en Scala se integran con los conceptos de **inmutabilidad y tipos avanzados**, permitiendo validar incluso la lógica asíncrona de efectos (IO, ZIO) y la resiliencia de sistemas de actores como Pekko.

### 2. Sintaxis y Estructura

Para implementar estas herramientas, la configuración comienza en el archivo de construcción de **sbt**. Se recomienda desactivar el búfer de logs de sbt (`logBuffered`) para obtener una salida legible suite por suite durante la ejecución en paralelo.

**Dependencias en `build.sbt`:**

|Framework|Configuración de Dependencia|Propósito|
|:--|:--|:--|
|**ScalaTest**|`libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.19" % Test`|Pruebas completas con múltiples estilos.|
|**MUnit**|`libraryDependencies += "org.scalameta" %% "munit" % "1.1.0" % Test`|Testing ligero, rápido y "plain Scala".|

**Estructura Base de una Suite:**

- **ScalaTest**: Normalmente extiende `AnyFunSuite` o `AnyWordSpec` para definir pruebas con nombres descriptivos y aserciones tipadas.
- **MUnit**: Utiliza la clase `FunSuite`, eliminando la sobrecarga cognitiva de los DSLs pesados y centrándose en el reporte detallado de errores (diffs).

### 3. Ejemplos de Código Realistas

#### A. Testing de un Repositorio Inmutable con ScalaTest

Este ejemplo utiliza `BeforeAndAfterAll` para gestionar el ciclo de vida de un recurso compartido (como una conexión a base de datos simulada).

```scala
import org.scalatest.funsuite.AnyFunSuite
import org.scalatest.BeforeAndAfterAll

// Modelo de dominio inmutable
case class Usuario(id: Int, nombre: String)

class UsuarioRepositorySpec extends AnyFunSuite with BeforeAndAfterAll:
  // Fixture de nivel de suite
  override def beforeAll(): Unit =
    println("Inicializando conexión a base de datos...")

  test("El repositorio debe permitir insertar y recuperar un usuario"):
    val repo = Map(1 -> Usuario(1, "Alice")) // Simulación inmutable
    val resultado = repo.get(1)

    assert(resultado.contains(Usuario(1, "Alice"))) // Aserción estándar

  override def afterAll(): Unit =
    println("Cerrando recursos de base de datos...")
```

#### B. Testing Asíncrono de Efectos (ZIO/Cats Effect) con MUnit

MUnit destaca en el testing de concurrencia y efectos puros, permitiendo devolver directamente un efecto (`IO` o `Task`) en el cuerpo del test.

```scala
import munit.FunSuite
import cats.effect.IO

class WebServiceSpec extends FunSuite:
  // MUnit maneja nativamente la ejecución de efectos asíncronos
  test("La API de saludo debe retornar un mensaje válido"):
    val logic: IO[String] = IO.pure("Hola Scala 3")

    logic.map { greeting =>
      assertEquals(greeting, "Hola Scala 3") // Reporte de errores con diff
    }
```

#### C. Testing de Actores (Pekko)

Para probar actores, se utiliza un `TestProbe` que actúa como un buzón para verificar el intercambio de mensajes asíncronos.

```scala
import org.apache.pekko.actor.testkit.typed.scaladsl.ScalaTestWithActorTestKit
import org.scalatest.wordspec.AnyWordSpecLike

class DeviceActorSpec extends ScalaTestWithActorTestKit with AnyWordSpecLike:
  "Un actor de dispositivo" should:
    "responder con la temperatura grabada" in:
      val probe = createTestProbe[RespondTemperature]()
      val deviceActor = spawn(Device("grupo1", "termometro1"))

      deviceActor ! RecordTemperature(requestId = 42, 25.5, probe.ref)
      // Verifica la recepción del mensaje asíncrono
      probe.expectMessage(TemperatureRecorded(42))
```

### 4. Comparativa con Java/Python

|Aspecto|Java (JUnit/Mockito)|Python (Pytest)|Scala (ScalaTest/MUnit)|
|:--|:--|:--|:--|
|**Filosofía**|Basado en ejemplos (Example-Based).|Dinámico y flexible, basado en funciones.|Híbrido: Basado en ejemplos o en propiedades (PBT).|
|**Aserciones**|`assertEquals(exp, act)`.|`assert x == y`.|Aserciones ricas o `assertEquals` con diffs visuales.|
|**Asincronía**|Compleja, requiere `CompletableFuture` o hilos manuales.|Basada en `async/await` (Asyncio).|Nativa para **Fibras** y Mónadas de efectos (IO/ZIO).|
|**Manejo de Recursos**|Bloques `try-finally` o `@BeforeClass`.|Fixtures de `pytest`.|Patrón **Bracket** y Fixtures funcionales inmutables.|

### 5. Best Practices (The Scala Way)

1. **Preferir Inmutabilidad en Fixtures:** En lugar de reasignar variables mutables (`var`) en métodos `beforeEach`, utiliza el patrón `FixtureContext` o las `FunFixture` de MUnit para proveer estados frescos e inmutables a cada prueba.
2. **Modelar Errores como Valores:** No lances excepciones para validar fallos esperados en el dominio. Modela los resultados como `Either` u `Option` y pruébalos usando los matchers específicos para estos tipos.
3. **Gestión Segura de Recursos:** Para pruebas que involucren I/O o red, utiliza el método `bracket` (adquisición, uso, liberación) en lugar de depender únicamente de `afterAll`, garantizando que los recursos se limpien incluso ante interrupciones críticas.
4. **Aprovechar el Paralelismo de sbt:** Permite que sbt ejecute las suites en paralelo (comportamiento por defecto) para maximizar el uso de la CPU, pero marca como `Serial` aquellas suites que compartan estado mutable externo para evitar condiciones de carrera.
5. **Testing de Efectos en el "Borde":** Compón tu lógica de efectos usando `map` y `flatMap` dentro del test, y permite que el framework de pruebas ejecute la operación final (el "fin del universo") para mantener la transparencia referencial.

---

**Metáfora de Ingeniería:** Hacer testing en Scala 3 es como pasar de inspeccionar una pieza terminada con un calibrador manual (Java/Python) a diseñar un **sistema de inspección automatizado por láser**; el compilador y los tipos actúan como guías de precisión que aseguran que las piezas (funciones y efectos) encajen perfectamente antes de que la cinta transportadora (el _runtime_) empiece a moverse.