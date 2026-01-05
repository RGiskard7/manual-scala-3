## Capítulo 23: Tooling de Ingeniería y Arquitectura (sbt y Modularidad)

### 1. Explicación Teórica: El Dominio del Build Tool y el Desacoplamiento

El ecosistema de ingeniería de Scala se define por el uso de herramientas de construcción (_build tools_) que gestionan dependencias, compilación y empaquetado, siendo **sbt (Scala Build Tool)** la elección histórica para proyectos de escala industrial. Recientemente, **Scala CLI** ha emergido como la herramienta moderna por defecto para _scripts_ y prototipos.

#### Modularidad y sbt

Para proyectos grandes y complejos, la modularidad se implementa dividiendo el código en **múltiples submódulos o subproyectos**. sbt gestiona esta arquitectura, permitiendo que la compilación se ejecute en **paralelo** entre los diferentes módulos, lo cual es crucial para reducir los tiempos de compilación de grandes bases de código. Los proyectos de Scala suelen estructurarse en paquetes que corresponden a diferentes capítulos o áreas de la aplicación.

#### Tooling de Calidad y Estándares

Además de las herramientas de compilación, los ingenieros de Scala utilizan _tooling_ avanzado para garantizar la calidad del código:

1. **sbt:** Gestiona la estructura de directorios, dependencias y tareas avanzadas.
2. **Scala CLI / Coursier:** Herramientas modernas de instalación (`cs`) y _scripting_ ligero.
3. **Metals / IntelliJ IDEA:** Entornos de desarrollo, siendo **Metals** la opción ligera preferida para Scala 3 y **IntelliJ IDEA** la opción para grandes bases de código.
4. **Scalafix:** Una herramienta poderosa para _linting_ y refactorización, utilizada para **imponer estándares de código** y automatizar refactorizaciones masivas.

#### Modularidad Arquitectónica y Desacoplamiento

La verdadera ingeniería de software implica **reducir el acoplamiento** entre módulos. El estilo de Scala, impulsado por la Programación Funcional Pura (PFP), busca escribir código débilmente acoplado. Un patrón clave para lograr esto es el **Principio del Mínimo Conocimiento (Law of Demeter)**:

- Un módulo debe depender solo de la **interfaz mínima** (un _trait_ o _boundary_) que necesita de otro módulo, en lugar de depender de la implementación completa de ese módulo.
- Esto previene que un cambio en una implementación subyacente de un módulo A fuerce una **recompilación en cascada** innecesaria de un módulo B, si B solo dependía de la interfaz abstracta.

---

### 2. Sintaxis: sbt y Declaración de Dependencias

#### A. Configuración Básica de sbt (build.sbt)

El archivo `build.sbt` define los ajustes y dependencias del proyecto. Para incluir dependencias y configurar la versión de Scala, se utiliza la sintaxis específica del DSL (Domain Specific Language) de sbt.

**Sintaxis (build.sbt):**

```scala
lazy val root = project
  .in(file(".")) // Proyecto raíz en el directorio actual
  .settings(
    // Versión de Scala (ej. para Scala 3)
    scalaVersion := "3.3.3",

    // Dependencias de librerías
    libraryDependencies ++= Seq(
      // Cats Effect para manejo de efectos
      "org.typelevel" %% "cats-effect" % "3.5.4",

      // Dependencia de ScalaCheck para testing PBT
      "org.typelevel" %% "scalacheck" % "1.18.0" % Test,

      // Apache Spark (ej. para ingeniería de datos)
      "org.apache.spark" %% "spark-sql" % "3.5.1"
    )
  )
```

#### B. Declaración de Módulos (Multi-Project)

Para la modularidad, `sbt` utiliza `lazy val` para definir cada submódulo, especificando el código fuente y las dependencias inter-módulo.

```scala
// Definición del Proyecto Raíz (que agrega los submódulos)
lazy val root = project
  .in(file("."))
  .aggregate(core, service, cli) // Agrega todos los módulos

// Definición del Módulo 'core'
lazy val core = project
  .in(file("core"))
  .settings(name := "core-module")

// Definición del Módulo 'service'
lazy val service = project
  .in(file("service"))
  // Dependencia: Service depende de Core
  .dependsOn(core)
```

---

### 3. Ejemplos de Código: Desacoplamiento con Traits

El patrón idiomático de Scala para el desacoplamiento fuerte se logra separando la definición de la API (un `trait` abstracto) de su implementación. Si el `Módulo B` necesita usar solo un método de `Módulo A`, definimos una interfaz abstracta en `Módulo B` y usamos la inyección de dependencias (`given` en Scala 3) para enlazar la implementación en tiempo de composición o inicio.

**Escenario Realista:** Desacoplar el módulo `Reportes` del módulo `BaseDeDatos` para que los cambios en la implementación de `BaseDeDatos` (ej., cambiar de JDBC a Cassandra) no recompilen `Reportes`.

#### 1. Módulo Core/Interfaces (Dependencia Mínima)

En el `core`, definimos solo lo que la lógica de negocio necesita (Ley de Deméter).

```scala
// Definido en el módulo 'Reportes'
// Solo declara la función esencial que Reportes necesita del servicio 'Usuario'.
// (Simulamos un trait llamado "Gate" o "Boundary" para el desacoplamiento)
trait ServicioUsuario {
  // Solo necesitamos saber si un usuario existe, no todos sus detalles
  def existe(id: String): Boolean
}

// Clase de negocio que necesita la dependencia inyectada.
// En Scala 3, usamos 'using' para inyectar la implementación de ServicioUsuario.
class GeneradorReporte(using usuarioService: ServicioUsuario) {
  def generar(userId: String): String =
    if usuarioService.existe(userId) then
      s"Reporte generado para ID: $userId"
    else
      s"Error: Usuario $userId no encontrado"
}
```

#### 2. Módulo de Implementación (Provee la Lógica)

En el módulo `BaseDeDatos`, se define y proporciona la implementación concreta utilizando un `given` o una `case class` que implementa el _trait_.

```scala
// Definido en el módulo 'BaseDeDatos'
import scala.collection.concurrent.TrieMap

// Implementación real del ServicioUsuario
case class ServicioUsuarioImpl() extends ServicioUsuario:
  // Simulación de una DB en memoria
  private val usuarios = TrieMap("A42" -> true, "B99" -> false)

  override def existe(id: String): Boolean =
    usuarios.contains(id)

// Proveer la instancia canónica de la interfaz (dado/given)
// Esto es lo que el compilador inyectará donde se pida 'using ServicioUsuario'.
given servicioUsuario: ServicioUsuario = ServicioUsuarioImpl()
```

Si la implementación de `ServicioUsuarioImpl` cambia (ej., cambia de `TrieMap` a `Doobie` o `Pekko Persistence`), el módulo `Reportes` no requiere cambios en su definición, lo que **minimiza la compilación incremental**.

---

### 4. Comparativa con Java o Python

La arquitectura de _tooling_ y modularidad de Scala, especialmente en combinación con el tipado estático y sbt, ofrece ventajas específicas en proyectos grandes.

|Característica|Java (Maven/Gradle)|Python (Scripts/Monolitos)|Scala (sbt/Scala CLI)|
|:--|:--|:--|:--|
|**Definición de Dependencias**|Archivos externos (XML o Groovy), a menudo verbosos.|`pip` (gestión de paquetes), acoplamiento en _runtime_.|**sbt DSL** conciso. **Scala CLI Directivas** para scripts (`//> using dependency`).|
|**Compilación Modular**|Maven/Gradle soporta módulos, pero la velocidad depende de la configuración.|N/A (Interpretado/Ejecución).|**Compilación en Paralelo** nativa a través de sbt.|
|**Desacoplamiento**|Inyección de dependencias en _runtime_ (Spring, CDI) o uso manual de `interface`.|Tipado dinámico (seguridad baja).|**Desacoplamiento por Tipo** (`trait` y `given/using`). La inyección se resuelve en **tiempo de compilación**, reduciendo errores.|
|**Rendimiento de Lógica**|Lógica compilada a _bytecode_ JVM.|Penalización por la comunicación entre el intérprete Python y la JVM (ej. PySpark).|**Scala UDFs/Lógica Pura** se ejecuta como _bytecode_ JVM nativo, optimizado para el alto rendimiento.|

---

### 5. Best Practices (_The Scala Way_)

1. **Modularidad Primero con sbt:** Utiliza sbt para dividir grandes aplicaciones en **múltiples submódulos lógicos**. Organiza el código en módulos (`core`, `api`, `persistence`) y utiliza `dependsOn` para definir el grafo de dependencias. Esto maximiza la **compilación incremental** y el paralelismo.
2. **Abstracción de Dependencias (Law of Demeter):** Implementa el desacoplamiento fuerte asegurándote de que los módulos dependan solo de las **interfaces abstractas (traits)** necesarias, no de las clases de implementación concretas. La inyección se debe gestionar mediante **Abstracciones Contextuales (`given/using`)**.
3. **Tooling Específico:** Usa **sbt** para la gestión de proyectos complejos y **Scala CLI** para _scripts_ y tareas rápidas. Para el desarrollo, **Metals** es el IDE recomendado para el entorno moderno de Scala 3.
4. **Estándares Automatizados:** Incorpora **Scalafix** en el proceso de desarrollo para **forzar estándares de código y evitar la deuda técnica**. Las herramientas de la metaprogramación (como Scalafix) son fundamentales en la arquitectura moderna de Scala.
5. **Reutilización del `case class`:** La inmutabilidad de los modelos (`case class`) permite que sean compartidos de forma segura a través de los límites de los módulos sin riesgo de concurrencia.

> En la arquitectura de Scala, la herramienta de construcción (sbt) no es solo un compilador; es el **arquitecto principal** que define la estructura y el paralelismo del sistema. Al dividir el proyecto en módulos desacoplados por interfaces abstractas, se asegura que los cambios en una sola unidad no derriben ni ralenticen toda la fábrica, permitiendo que la compilación y la evolución sean rápidas y seguras.