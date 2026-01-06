## Capítulo 26: Scala.js - Programación Frontend Funcional

### 1. Explicación Teórica

**Scala.js** representa la extensión del ecosistema Scala más allá de la JVM, permitiendo compilar código fuente de Scala 3 directamente a **JavaScript** optimizado. Esta tecnología es fundamental en entornos industriales porque elimina la barrera tradicional entre el desarrollo backend y frontend, permitiendo que los equipos utilicen el mismo lenguaje, librerías y, lo más importante, los mismos **modelos de datos** en todo el stack tecnológico. Al ejecutarse sobre el motor de JavaScript del navegador o Node.js, Scala.js ofrece la robustez del tipado estático en un entorno históricamente propenso a errores de ejecución.

La arquitectura de una aplicación Scala.js moderna se aleja del modelo de "DOM virtual" de frameworks como React para adoptar la **Programación Reactiva Funcional (FRP)**. A través de librerías como **Laminar**, los desarrolladores gestionan el estado de la interfaz de usuario mediante **observables** (Airstream), donde los flujos de datos se vinculan directamente a los elementos del DOM. Esto no solo mejora el rendimiento al evitar cálculos de diferencia de árboles (diffing), sino que garantiza que la interfaz de usuario sea una representación pura y coherente del estado de la aplicación en todo momento.

### 2. Sintaxis y Estructura

La estructura de un proyecto Scala.js profesional utiliza el plugin `sbt-crossproject` para organizar el código en tres segmentos lógicos: `shared` (lógica de dominio), `js` (frontend) y `jvm` (backend).

**Configuración de Dependencias (build.sbt):** En Scala.js, se utiliza el operador `%%%` para asegurar que sbt descargue la versión compatible con JavaScript de la librería.

|Elemento|Sintaxis / Herramienta|Propósito|
|:--|:--|:--|
|**Plugin Core**|`addSbtPlugin("org.scala-js" % "sbt-scalajs" % "1.18.2")`|Habilita la compilación a JS.|
|**Cross-Project**|`crossProject(JSPlatform, JVMPlatform)`|Define módulos compartidos entre JS y JVM.|
|**Dependencias JS**|`libraryDependencies += "com.raquo" %%% "laminar" % "17.2.0"`|Librerías específicas para el navegador.|
|**Interoperabilidad**|`@js.native`, `js.Dynamic`|Acceso a variables y funciones globales de JS.|

### 3. Ejemplos de Código Realistas

#### A. Modelos Compartidos y Serialización (Carpeta `shared/`)

Utilizamos **uPickle** para definir modelos que se serializan automáticamente en ambos extremos del stack.

```scala
import upickle.default.{ReadWriter, derives}

// Modelo definido una sola vez para Backend y Frontend
case class Usuario(id: String, nombre: String, email: String) derives ReadWriter

// Lógica de validación compartida
object Validador:
  def esEmailValido(email: String): Boolean = email.contains("@")
```

#### B. Interfaz Reactiva con Laminar (Carpeta `js/`)

Laminar utiliza el sistema de observables para manejar eventos de usuario sin "glitches" de estado.

```scala
import com.raquo.laminar.api.L._
import org.scalajs.dom

// Un contador reactivo simple
val contador = Var(0)

def appElement() =
  div(
    h1("Gestión de Usuarios"),
    button(
      tpe := "button",
      "Incrementar: ",
      child.text <-- contador.signal.map(_.toString), // Enlace de datos
      onClick --> { _ => contador.update(_ + 1) }    // Manejo de eventos
    ),
    p(
      child.text <-- contador.signal.map(v => s"El valor actual es $v")
    )
  )

// Renderizado en el DOM
render(dom.document.getElementById("app"), appElement())
```

#### C. Integración con APIs usando sttp (Carpeta `js/`)

Scala.js permite realizar peticiones asíncronas de forma segura en tipos integrando `sttp` y `uPickle`.

```scala
import sttp.client3._
import sttp.client3.upickle._

def buscarUsuario(id: String): Unit =
  val request = basicRequest
    .get(uri"http://api.miempresa.com/usuarios/$id")
    .response(asJson[Usuario]) // uPickle maneja el parseo automáticamente

  // sttp usa internamente las promesas de JS mapeadas a Future de Scala
  println(s"Iniciando petición para usuario $id...")
```

### 4. Comparativa con Java/Python

|Característica|Java (GWT/TeaVM)|Python (PyScript/Brython)|Scala.js|
|:--|:--|:--|:--|
|**Paradigma**|Imperativo pesado.|Scripting interpretado.|**Funcional Reactivo (FRP)**.|
|**Seguridad de Tipos**|Alta, pero verbosa.|Nula/Baja (Runtime).|**Máxima** (Scala 3 > TypeScript).|
|**Código Compartido**|Complejo (limitado a Java puro).|Pesado (requiere runtime en el navegador).|**Nativo y ligero** vía `shared/`.|
|**Rendimiento**|Regular (JS generado verboso).|Lento (interpretado en el cliente).|**Excelente** (optimizador avanzado).|
|**Interoperabilidad**|Difícil con ecosistema npm.|Buena con ecosistema Python.|**Total** vía `ScalablyTyped`.|

### 5. Best Practices (The Scala Way)

1. **Centralizar la Verdad en `shared/`**: Mueve todos los modelos de datos, DTOs y lógica de validación al módulo compartido. Esto garantiza que un cambio en el backend rompa la compilación del frontend si hay una incompatibilidad, evitando errores de integración en runtime.
2. **Preferir Laminar sobre React Wrappers**: Aunque existen librerías como `Slinky` para React, `Laminar` es la opción "Scala pura" que no hereda las limitaciones de JavaScript y ofrece una gestión de memoria automática superior.
3. **Usar ScalablyTyped para librerías JS**: No escribas _facades_ manualmente para librerías de npm. Utiliza `ScalablyTyped` para generar automáticamente tipos de Scala a partir de definiciones de TypeScript (.d.ts).
4. **Optimizar con `uPickle`**: Para aplicaciones con alta transferencia de datos, usa `uPickle` en lugar de `circe`. Es significativamente más rápido en Scala.js al aprovechar el parser de JSON nativo del navegador.
5. **Distinguir Etapas de Linkeado**: Usa `fastLinkJS` durante el desarrollo para recompilaciones rápidas y `fullLinkJS` para producción, ya que este último aplica optimizaciones avanzadas de eliminación de código muerto (Dead Code Elimination).

---

**Metáfora de Ingeniería:** Desarrollar el frontend con Scala.js es como construir un edificio con **sensores de tensión integrados en cada viga**; a diferencia de JavaScript (donde solo te das cuenta de que algo está mal cuando el edificio se agrieta), el sistema de tipos de Scala actúa como un sistema de monitorización constante que impide que coloques una pieza donde no encaja perfectamente..