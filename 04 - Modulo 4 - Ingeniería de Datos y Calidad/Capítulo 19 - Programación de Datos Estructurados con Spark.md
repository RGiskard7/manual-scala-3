## Capítulo 19: Programación de Datos Estructurados con Spark

### 1. Explicación Teórica: DataFrames, Datasets y el Optimizador Catalyst

La **Programación de Datos Estructurados** en Apache Spark se realiza predominantemente a través de las abstracciones de **DataFrame** y **Dataset**. Estas estructuras son fundamentales para el procesamiento de Big Data, ya que permiten que el motor de optimización de Spark, llamado **Catalyst**, genere planes de ejecución altamente eficientes.

#### DataFrames (Datasets sin tipo)

Un **DataFrame** es esencialmente un `Dataset` de tipo `Row` (`Dataset[Row]`). Proporciona una vista estructurada de los datos, similar a una tabla en una base de datos relacional. Las operaciones en DataFrames son llamadas "transformaciones sin tipo" (_untyped transformations_) porque las comprobaciones de nombres de columnas y esquemas se realizan en **tiempo de ejecución** (_runtime_). Esto ofrece flexibilidad y concisión, pero con menor seguridad de tipos.

#### Datasets (Datasets fuertemente tipados)

El **Dataset** es la abstracción predilecta en Scala. Representa una colección distribuida de objetos fuertemente tipados. Los Datasets son la unión perfecta entre la eficiencia de los DataFrames y la **seguridad de tipos y la programación orientada a objetos de Scala**.

Para definir el esquema de un Dataset de forma segura en tipos, se utiliza la **`case class`** de Scala. Spark utiliza un mecanismo llamado **Encoder** para serializar y deserializar estos objetos de Scala. Los _Encoders_ están codificados dinámicamente y utilizan un formato que permite a Spark realizar operaciones clave (como filtrado, ordenación y _hashing_) sin necesidad de deserializar completamente los bytes de vuelta a objetos en cada paso, lo que resulta en una abstracción de alto rendimiento.

### 2. Sintaxis y Operaciones Fundamentales

La manipulación de DataFrames y Datasets se basa en una API expresiva que se traduce en optimizaciones del motor Catalyst. Para empezar a trabajar con DataFrames o Datasets, es crucial tener una instancia de `SparkSession` e importar las conversiones implícitas (los _Encoders_).

**Requisito de Importación:**

```scala
import spark.implicits._
// Habilita los Encoders y la notación de columna $""
```

#### A. Operaciones sin Tipo (DataFrames / `Dataset[Row]`)

Estas operaciones manipulan los datos haciendo referencia a las columnas mediante _strings_ (cadenas de texto) o notación `$`.

|Operación|Descripción|Sintaxis (DataFrame)|
|:--|:--|:--|
|**Selección**|Selecciona columnas específicas.|`df.select("nombre", "edad")`|
|**Expresiones**|Ejecuta cálculos o transformaciones en las columnas.|`df.select( $"name", $"age" + 1 )`|
|**Filtrado**|Retiene filas que cumplen una condición (_predicate_).|`df.filter( $"age" > 21 )`|
|**Agregación**|Agrupa por una columna y aplica una función de agregación.|`df.groupBy("age").count()`|

#### B. Operaciones con Tipo (Datasets / `Dataset[T]`)

Cuando se utiliza un `Dataset[T]` (donde T es una `case class`), se puede acceder a los campos directamente utilizando la sintaxis de programación funcional de Scala (lambdas y HOFs como `map` y `filter`).

|Operación|Descripción|Sintaxis (Dataset[T])|
|:--|:--|:--|
|**Filtrado**|Usa lambdas sobre el tipo `T`.|`ds.filter(p => p.edad > 21)`|
|**Transformación**|Crea un nuevo objeto `T` inmutable por cada elemento.|`ds.map(p => p.copy(edad = p.edad + 1))`|

### 3. Ejemplos de Código Realistas

Utilizaremos la modelización de un equipo deportivo de Spark, mostrando cómo las uniones (_joins_) son esenciales en la ingeniería de datos.

```scala
import org.apache.spark.sql.{SparkSession, Dataset, Row}
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._

// 1. Inicialización de Spark (asumiendo sparkSession ya existe)
val spark: SparkSession = ???
import spark.implicits._ // Habilita Encoders y la notación $"columna"

// 2. Definición del Esquema (Usando Case Class para seguridad de tipos)
case class Kid(id: Int, nombre: String, teamId: Int)
case class Team(id: Int, nombreEquipo: String, capitanId: Int)

// 3. Creación de Datasets de ejemplo (Simulación de datos)
val kidsDS: Dataset[Kid] = Seq(
  Kid(1, "Ana", 10),
  Kid(2, "Juan", 20),
  Kid(3, "Eva", 10),
  Kid(4, "Leo", 99) // Kid sin equipo (ID 99 no existe)
).toDS()

val teamsDS: Dataset[Team] = Seq(
  Team(10, "Los Invencibles", 1),
  Team(20, "Estrellas de Rock", 2)
).toDS()

// --- OPERACIONES FUNDAMENTALES ---

// A. Filtrado y Agregación (Typed - Seguro en Tipos)
println("--- Agregación: Conteo de Miembros por Equipo ---")
val conteo = kidsDS
  .groupBy(_.teamId) // Agrupa usando el campo de la case class
  .count()          // Cuenta los miembros en cada grupo
conteo.show()
// Resultado: (teamId=10, count=2), (teamId=20, count=1), (teamId=99, count=1)

// B. Unión de DataFrames (Inner Join)
// Queremos la lista de niños con el nombre de su equipo.
// Transformamos a DataFrame (Dataset[Row]) para usar la API de columna en la unión.
val condicionUnion = kidsDS("teamId") === teamsDS("id") // Columna de Kid === Columna de Team

println("--- Inner Join: Solo niños con equipo existente ---")
val kidsTeamsDF = kidsDS.toDF() // Convertir a DataFrame (Dataset[Row])
  .join(teamsDS.toDF(), condicionUnion, "inner")
  .select($"nombre" as "Nombre_Niño", $"nombreEquipo")

kidsTeamsDF.show()
/*
+-------------+-------------+
|Nombre_Niño|nombreEquipo|
+-------------+-------------+
|Ana          |Los Invencibles|
|Eva          |Los Invencibles|
|Juan         |Estrellas de Rock|
+-------------+-------------+
*/

// C. Unión Externa Izquierda (Left Outer Join)
// Queremos todos los niños, incluso si no tienen equipo (usando Left Outer Join)
println("--- Left Outer Join: Incluyendo al niño sin equipo ---")
val allKidsTeamsDF = kidsDS.toDF()
  .join(teamsDS.toDF(), condicionUnion, "left_outer")
  .select($"nombre" as "Nombre_Niño", $"nombreEquipo")

allKidsTeamsDF.show()
/*
+-------------+-------------+
|Nombre_Niño|nombreEquipo|
+-------------+-------------+
|Ana          |Los Invencibles|
|Eva          |Los Invencibles|
|Juan         |Estrellas de Rock|
|Leo          |null            | // Leo no tiene equipo (null)
+-------------+-------------+
*/

// D. Anti-Join (Patrón de Filtrado)
// Queremos encontrar solo a los niños que NO están en un equipo.
println("--- Anti-Join: Niños sin equipo ---")
// Left Anti Join devuelve las filas del lado izquierdo para las que NO hay una coincidencia en el derecho.
val lonelyKidsDF = kidsDS.toDF()
  .join(teamsDS.toDF(), condicionUnion, "left_anti")
  .select($"nombre" as "Nombre_Niño")

lonelyKidsDF.show()
// Resultado: Leo
```

### 4. Comparativa con Java y Python

La principal ventaja de usar Scala para la programación de datos estructurados con Spark reside en la robustez de su sistema de tipos en tiempo de compilación.

|Característica|Java (API de Spark)|Python (PySpark)|Scala (API de Spark)|
|:--|:--|:--|:--|
|**Estructura Preferida**|`Dataset[Row]` (DataFrame) o `Dataset[JavaBean]`.|`DataFrame` (dinámico).|**`Dataset[CaseClass]`**.|
|**Seguridad del Esquema**|Estático, pero requiere _JavaBeans_ (más verboso que `case class`).|**Dinámico**. Los errores de esquema (ej. escribir mal el nombre de una columna) se detectan en _runtime_.|**Estático.** El compilador verifica que la `case class` y la estructura de datos coincidan en **tiempo de compilación**.|
|**Expresividad en HOFs**|Requiere clases anónimas o _lambdas_ con `MapFunction` y `Encoder` explícito.|Usa _lambdas_ (`lambda x: ...`) y sintaxis de Python.|**Conciso y funcional.** Utiliza directamente lambdas de Scala (ej. `.filter(_.edad > 21)`).|
|**UDFs (Lógica de Negocio)**|Nativo JVM.|Penalización de rendimiento debido a la serialización de datos entre la JVM y el proceso Python.|**Nativo JVM.** Rendimiento superior para la lógica compleja.|

### 5. Best Practices (_The Scala Way_)

La forma idiomática de manipular datos estructurados en Spark con Scala se basa en la seguridad de tipos y la composición funcional:

1. **Modelar Siempre con `case class` y `Dataset[T]`:** La mejor práctica es utilizar **`case class`** para definir el esquema de los datos y trabajar con **`Dataset[T]`**. Esto traslada la detección de errores de esquema (ej. columna inexistente) del _runtime_ al **tiempo de compilación**, lo cual es un sello distintivo de la ingeniería de datos robusta en Scala.
2. **Usar `spark.implicits._`:** Esta importación es necesaria para obtener los `Encoder` necesarios para que Spark pueda mapear las `case class` hacia y desde el formato interno optimizado.
3. **Priorizar la API Columnar y Funciones Integradas:** Siempre que sea posible, utiliza las funciones optimizadas de la API columnar de Spark (ej. `$"columna" + 1`, `groupBy`) o las funciones integradas de `org.apache.spark.sql.functions` (ej. `count`, `sum`). Estas funciones son procesadas por el optimizador Catalyst, que genera el _bytecode_ más eficiente. **Evita las UDFs** (User Defined Functions) a menos que la lógica no pueda expresarse de otra manera, ya que las UDFs suelen ser menos optimizables.
4. **Composición con HOFs y `for-comprehensions`:** Utiliza `map`, `filter`, `flatMap` o las _for-comprehensions_ (que son azúcar sintáctico para los monads subyacentes) para aplicar transformaciones a los Datasets. Este estilo declarativo maximiza la legibilidad y la capacidad de optimización del código.

> La programación de datos estructurados en Scala es como construir una base de datos distribuida con un **tipado estricto**. Al definir el esquema con `case class` antes de procesar un terabyte de datos, le das al compilador un plano arquitectónico (el `Dataset[T]`), permitiendo que el optimizador de Spark (Catalyst) construya la tubería de procesamiento más rápida posible, garantizando que si el código compila, es menos propenso a fallar a gran escala.