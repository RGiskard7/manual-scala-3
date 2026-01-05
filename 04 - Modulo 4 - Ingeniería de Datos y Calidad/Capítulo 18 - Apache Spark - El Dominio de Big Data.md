## Capítulo 18: Apache Spark: El Dominio de Big Data

### 1. Explicación Teórica: Scala como Lenguaje Nativo de Ingeniería de Datos

**Apache Spark** es una herramienta esencial y uno de los estándares del mercado para el entorno de **Big Data** y el análisis de datos a gran escala. Scala, acrónimo de _Scalable Language_, fue diseñado para la computación distribuida y el procesamiento de datos.

Scala es el lenguaje de implementación en el que se escribió originalmente Apache Spark. Históricamente, Spark revolucionó el procesamiento de Big Data al introducir abstracciones en memoria, como los **RDDs (Resilient Distributed Datasets)**, que son mucho más rápidas que los modelos anteriores como MapReduce.

Mientras que Python (a través de PySpark) suele ser el lenguaje preferido para la **exploración de datos** (data exploration), Scala es indiscutiblemente el lenguaje de la **ingeniería de datos a gran escala** (data engineering). Scala es el lenguaje de **primera clase** para Spark, utilizado para:

1. **Alto Rendimiento:** Escribir lógica de negocio (User Defined Functions o UDFs) que se ejecuta de forma nativa en la **Java Virtual Machine (JVM)**, eludiendo la costosa penalización de serialización y deserialización que se produce al comunicar con procesos externos de Python.
2. **Seguridad de Tipos:** Utilizar el avanzado sistema de tipos de Scala para modelar y validar esquemas de datos distribuidos en tiempo de compilación.

#### Las Abstracciones Fundamentales

Spark proporciona tres abstracciones principales para trabajar con datos distribuidos:

- **RDD (Resilient Distributed Dataset):** La abstracción original, que permite operaciones de bajo nivel y una vista de colecciones inmutables y distribuidas.
- **DataFrame:** Una vista de datos estructurados, similar a una tabla en una base de datos o un DataFrame en Python, que proporciona un lenguaje de dominio específico para la manipulación de datos estructurados. En Scala y Java, los DataFrames son esencialmente _Datasets de tipo Row_ (filas sin tipo estricto).
- **Dataset:** Una colección distribuida de objetos fuertemente tipados. Los Datasets combinan los beneficios de la optimización del motor de Spark (como los DataFrames) con la seguridad de tipos y la programación orientada a objetos de Scala.

### 2. Sintaxis: Inicialización y Estructuras de Datos

El punto de entrada a toda la funcionalidad de Spark es la clase **`SparkSession`**.

#### A. Inicialización de la Sesión

La sesión se crea mediante el constructor `SparkSession.builder()`, que centraliza el acceso a las APIs de RDDs, DataFrames y Datasets.

**Sintaxis (Creación de `SparkSession`):**

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("Nombre de mi Aplicación Spark")
  .config("spark.some.config.option", "some-value")
  .getOrCreate() // Obtiene una sesión existente o crea una nueva
```

#### B. Creación de DataFrames y Datasets

Una vez que se tiene una `SparkSession`, se pueden crear DataFrames a partir de un RDD existente, una tabla Hive o desde fuentes de datos como archivos JSON o Parquet.

**Sintaxis (Creación de DataFrame desde una fuente):**

```scala
// spark es una SparkSession existente
val df = spark.read.json("ruta/a/archivo.json")
```

Para trabajar con DataFrames de forma segura en tipos, se utiliza la clase **`case class`** de Scala para definir el esquema del dato. Los Datasets se crean a partir de _case classes_ proporcionando un _Encoder_ especializado.

**Sintaxis (Creación de Dataset tipado):**

```scala
case class Persona(nombre: String, edad: Long) // La case class define el esquema
import spark.implicits._ // Necesario para las conversiones implícitas (Encoder)

// Crear un Dataset tipado a partir de una secuencia en memoria
val caseClassDS = Seq(Persona("Ana", 32)).toDS()
```

### 3. Ejemplos de Código: Modelado de Datos

El siguiente ejemplo demuestra el flujo para inicializar Spark y cargar un DataFrame desde un archivo, luego mostrando cómo acceder y manipular los datos de forma _untyped_ (DataFrame) y _typed_ (Dataset).

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.Dataset
import org.apache.spark.sql.Row

// Definición del modelo de datos para el Dataset tipado
case class Persona(nombre: String, edad: Long)

object BigDataApp extends App {

  // 1. Inicialización de SparkSession (Entry Point)
  val spark: SparkSession = SparkSession
    .builder()
    .appName("Dominio Big Data")
    .master("local[*]") // Usar el máximo de núcleos locales para ejecución
    .getOrCreate()

  // Importar implicits para habilitar .toDS() y la notación $""
  import spark.implicits._

  // 2. Crear un DataFrame (untyped) leyendo un archivo JSON (asumiendo people.json existe)
  // df es de tipo DataFrame (que es un Dataset[Row])
  val df = spark.read.json("ejemplos/recursos/people.json")

  println("--- 2A. DataFrame (Sin tipar) ---")
  df.printSchema() // Imprime el esquema en formato de árbol

  // Operaciones untyped (DataFrame Operations)
  // Seleccionar solo la columna "name"
  df.select("name").show()

  // 3. Crear un Dataset (typed) a partir del DataFrame
  // Intentar convertir el DataFrame a un Dataset[Persona]
  val peopleDS: Dataset[Persona] = df.as[Persona]

  println("--- 3A. Dataset (Tipado) ---")
  peopleDS.printSchema()

  // 4. Transformaciones funcionales en el Dataset tipado
  // Incrementar la edad por 1 y seleccionar solo aquellos mayores de 21
  peopleDS
    .filter(_.edad > 21) // Utiliza la propiedad 'edad' con tipado seguro
    .map(p => p.copy(edad = p.edad + 1)) // Crea una copia inmutable del objeto Persona
    .show()

  // 5. Ejecutar consultas SQL programáticamente
  df.createOrReplaceTempView("personas") // Registrar el DataFrame como vista temporal
  val sqlDF: DataFrame = spark.sql("SELECT nombre, edad FROM personas WHERE edad IS NOT NULL") // Ejecutar consulta SQL
  sqlDF.show()
}
```

### 4. Comparativa con Java o Python

Scala es la opción preferida para la ingeniería de datos sobre otros lenguajes de la JVM y sobre Python, principalmente por la **seguridad estática** y la **eficiencia en el _runtime_**.

|Aspecto|Java (API de Spark)|Python (PySpark)|Scala (API de Spark)|
|:--|:--|:--|:--|
|**Tipo de Estructura Preferida**|`Dataset<Row>` (DataFrame) o `Dataset<JavaBean>` (Dataset).|DataFrame (pandas-like).|**Dataset[CaseClass]**. Usa _case classes_ para una vista tipada de los datos.|
|**Seguridad de Tipos y Esquema**|Estático, pero más verboso (requiere JavaBeans).|**Dinámico.** El esquema se verifica en _runtime_ (esquema flexible).|**Estático y Conciso.** El compilador verifica el esquema de los `Datasets` en tiempo de compilación.|
|**Rendimiento de UDFs (Lógica)**|**Nativo JVM.** Rendimiento de código Java puro.|**Penalización.** Requiere serialización de datos entre la JVM (Spark) y el proceso de Python.|**Nativo JVM.** Las funciones Scala se compilan a _bytecode_ de Java, resultando en alto rendimiento.|
|**Concisión**|Muy verboso (necesita _getters_, _setters_, y `Serializable`).|Conciso (lenguaje dinámico).|**Muy Conciso.** Se utiliza sintaxis funcional (HOFs, _for-comprehensions_) y _case classes_.|

### 5. Best Practices (_The Scala Way_)

La forma idiomática de trabajar con Big Data utilizando Scala y Spark aprovecha la robustez del sistema de tipos para garantizar la corrección del código antes de ejecutar flujos de datos masivos:

1. **Priorizar `Dataset[T]`:** Siempre que sea posible, define tu esquema de datos mediante **`case class`** y trabaja con la abstracción **`Dataset[T]`** en lugar del `DataFrame` (que es `Dataset[Row]`). Esto te permite usar la **seguridad de tipos en tiempo de compilación** para atrapar errores de esquema o de lógica de acceso a columnas que de otro modo explotarían en _runtime_.
2. **Utilizar `spark.implicits._`:** Esta importación es esencial, ya que proporciona los `Encoder` necesarios para que Spark pueda serializar y deserializar tus `case class` de Scala de forma eficiente cuando se convierten a `Dataset` y viceversa.
3. **Composición Funcional para Transformaciones:** Utiliza funciones de orden superior (`map`, `filter`, `groupBy`) y **`for-comprehensions`** para la manipulación de DataFrames/Datasets. Esto es más legible y se traduce a un plan de ejecución optimizado por el motor Catalyst de Spark.
4. **UDFs en Scala:** Para la lógica compleja (User Defined Functions), implementalas en Scala. Esto garantiza que la UDF se ejecute en el entorno nativo de la JVM, **maximizando el rendimiento** y evitando el costo de comunicación y serialización con procesos de lenguajes externos (como Python).
5. **Entorno REPL para Prototipado:** Para probar transformaciones de datos rápidamente, utiliza el `spark-shell` (que se ejecuta en Scala) para iterar y probar la lógica de procesamiento antes de integrarla en la aplicación completa.