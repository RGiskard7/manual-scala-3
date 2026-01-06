## Capítulo 28: Migración de Scala 2 a Scala 3

### 1. Explicación Teórica

La transición de Scala 2 a Scala 3 representa un **cambio de época** en el ecosistema de la JVM, similar a la evolución de Python 2 a Python 3. No se trata de una simple actualización incremental, sino de una reescritura completa del compilador basada en el fundamento teórico del **Cálculo DOT** (Dependent Object Types), diseñada para garantizar la solidez del sistema de tipos. Esta evolución busca eliminar la verbosidad y la confusión de patrones antiguos, sustituyendo mecanismos sobrecargados por construcciones basadas en la **intención del programador**.

El pilar tecnológico que permite que esta migración no requiera una reescritura total e inmediata es el formato **TASTy (Typed Abstract Syntax Trees)**. A diferencia de los archivos `.class` tradicionales, que sufren de borrado de tipos (_type erasure_), los archivos `.tasty` contienen la representación completa del árbol de sintaxis abstracta tipado. Esto permite una **compatibilidad binaria bidireccional**: Scala 3 puede consumir librerías de Scala 2.13, y las versiones recientes de Scala 2 (2.13.6+) pueden leer archivos TASTy de Scala 3, facilitando una coexistencia fluida en proyectos industriales.

### 2. Sintaxis y Estructura

La migración se centra en tres áreas críticas: la limpieza sintáctica, el rediseño de las abstracciones contextuales y la eliminación de características obsoletas.

#### Cambios en Abstracciones Contextuales

Scala 3 descompone la palabra clave `implicit` en conceptos más precisos basados en la intención:

|Intención en Scala 2|Construcción en Scala 3|Propósito|
|:--|:--|:--|
|`implicit val` / `implicit object`|**`given`**|Define un valor canónico para un tipo en el contexto.|
|Parámetros `implicit`|**`using`**|Declara que una función requiere una instancia del contexto.|
|`implicit class`|**`extension`**|Añade métodos a tipos existentes sin herencia ni envoltorios.|
|Conversiones implícitas|**`Conversion[A, B]`**|Define transformaciones de tipo de forma segura y explícita.|

#### Herramientas de Migración Automática

El uso de herramientas especializadas es el estándar industrial para reducir el esfuerzo manual:

1. **`migrate-libs`**: Verifica la compatibilidad de las dependencias y sugiere versiones para Scala 3.
2. **`migrate-syntax`**: Aplica reglas de **Scalafix** para corregir incompatibilidades sintácticas (ej. quitar `procedure syntax` o cambiar literales de símbolos).
3. **`migrate-scalacOptions`**: Actualiza los flags del compilador a sus equivalentes modernos.
4. **`migrate`**: El paso final que añade tipos inferidos y argumentos implícitos necesarios para que el código compile en el nuevo algoritmo de inferencia.

### 3. Ejemplos de Código Realistas

#### A. Conversión de Implicits a Given/Using

Migración de un patrón de _Type Class_ tradicional a la sintaxis de Scala 3.

```scala
// Scala 2.13
// implicit val personOrdering: Ordering[Person] = ...
// def sort[T](list: List[T])(implicit ord: Ordering[T]): List[T] = ...

// Scala 3 (Idiomático)
case class Person(name: String, age: Int)

// Definición de la instancia (Provisión)
given personOrdering: Ordering[Person] with
  def compare(x: Person, y: Person): Int = x.name.compareTo(y.name)

// Consumo del contexto
def sortData[T](list: List[T])(using ord: Ordering[T]): List[T] =
  list.sorted(using ord)
```

#### B. Manejo de literales de Símbolos y Sintaxis

Scala 3 elimina la sintaxis de tick para símbolos y requiere el uso explícito de la clase `Symbol`.

```scala
// Scala 2.13 (Incompatible en Scala 3)
// val id = 'myIdentifier

// Scala 3
val id = Symbol("myIdentifier")

// Métodos de extensión modernos
extension (s: String)
  def isVocal: Boolean = "aeiou".contains(s.toLowerCase)
```

#### C. Configuración de Cross-Building en `build.sbt`

Estrategia para compilar incrementalmente un proyecto para ambas versiones.

```scala
// build.sbt
ThisBuild / crossScalaVersions := Seq("2.13.15", "3.3.4") // Scala LTS recomendada
ThisBuild / scalaVersion      := "2.13.15"

scalacOptions ++= {
  if (scalaVersion.value.startsWith("2.13")) Seq("-Xsource:3") // Detección temprana
  else Seq("-Xmax-inlines", "1024") // Optimización para macros de Scala 3
}
```

### 4. Comparativa con Java/Python

|Característica|Transición Java|Transición Python|Migración Scala 3|
|:--|:--|:--|:--|
|**Interoperabilidad**|Casi total entre versiones minor (bytecode).|Difícil entre 2 y 3 (runtime).|**Alta mediante TASTy** entre 2.13 y 3.x.|
|**Impacto Sintáctico**|Mínimo (adición de keywords).|Masivo (prints, strings, divisions).|**Estructural** (rediseño de `implicits`).|
|**Macros/Reflexión**|Reflexión estable en runtime.|Dinámico por naturaleza.|**Incompatible**; las macros de Scala 2 deben reescribirse.|
|**Estrategia de Soporte**|Versiones LTS concurrentes.|EoL de Python 2 tras una década.|**LTS & Next**; soporte prolongado para 2.13 y 3.3.|

### 5. Best Practices (The Scala Way)

1. **Migrar a Scala 2.13 Primero:** No intentes saltar de 2.11 o 2.12 directamente a 3. La versión 2.13 es el puente necesario para la compatibilidad de TASTy.
2. **Activar `-Xsource:3` en Scala 2:** Utiliza este flag en tu build de Scala 2.13 para detectar de forma temprana incompatibilidades de tipos e inferencia que fallarán en Scala 3.
3. **Migración por Módulos:** En proyectos grandes, utiliza una arquitectura multi-módulo en sbt. Empieza migrando los módulos más pequeños y con menos dependencias externas primero para ganar confianza.
4. **Priorizar la versión LTS (Long Term Support):** Para aplicaciones industriales, migra directamente a la versión LTS actual (ej. 3.3.x) para garantizar estabilidad en el _tooling_ y soporte extendido del compilador.
5. **Aislar Código Específico:** Si necesitas mantener compatibilidad con ambas versiones, utiliza directorios específicos como `src/main/scala-2` y `src/main/scala-3` para fragmentos de código divergentes (como macros o el uso de `Symbol`).

---

**Metáfora de Ingeniería:** Migrar a Scala 3 es como **actualizar los planos de una central eléctrica de papel a un modelo digital 3D (TASTy)**; aunque la electricidad (el bytecode) sigue fluyendo por los mismos cables, el nuevo formato permite a los ingenieros (el compilador) ver cada detalle estructural con una precisión que antes era imposible, detectando fallos de diseño antes de que ocurran.