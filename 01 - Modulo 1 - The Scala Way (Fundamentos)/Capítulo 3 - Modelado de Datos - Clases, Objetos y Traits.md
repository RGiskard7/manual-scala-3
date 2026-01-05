## Capítulo 3: Modelado de Datos: Clases, Objetos y Traits

### 1. Explicación Teórica: El Molde de la POO Pura

Scala es un lenguaje que se adhiere al ideal de la **Programación Orientada a Objetos (POO) Pura**: **todo valor es un objeto**. Incluso los tipos que en Java serían primitivos (`Int`, `Double`) son objetos que descienden del tipo raíz `Any`.

En este contexto, las estructuras de Scala —`class`, `object`, y `trait`— sirven como los planos arquitectónicos para modelar el dominio de una aplicación, facilitando la encapsulación del estado y la lógica. La clave para un modelado idiomático en Scala radica en entender cómo estas estructuras se utilizan para **imponer la inmutabilidad**, **reemplazar los miembros estáticos** de Java y **favorecer la composición** sobre la herencia rígida.

### 2. Clases Estándar (`class`) y Constructores

Una `class` en Scala es el plano para crear instancias de objetos con su propio estado y métodos,.

#### El Constructor Primario

La principal diferencia con Java o Python es que **la declaración de la clase define el constructor primario**,.

- **Valores de Constructor Privados:** Si una variable de constructor se define sin `val` ni `var`, es un parámetro privado, no accesible fuera de la clase.
- **Campos Públicos Inmutables:** Si se antepone **`val`**, el parámetro se promociona automáticamente a un **campo público inmutable** (un _getter_ de solo lectura).
- **Campos Públicos Mutables:** Si se antepone **`var`**, el parámetro se convierte en un **campo público mutable** (con _getter_ y _setter_). **Esto se utiliza raramente** en código idiomático.

#### Sintaxis de la Clase Estándar

```scala
// El constructor primario toma dos argumentos: 'nombre' (val inmutable) y 'antiguedad' (privado).
// Se asume que no queremos que 'antiguedad' sea accesible o modificable desde fuera.
class Empleado(val nombre: String, antiguedad: Int) {

  // Campo privado interno. Se utiliza 'var' porque el estado interno de un actor
  // o clase simple a veces requiere mutabilidad controlada (aunque es mejor evitarlo).
  private var salario: Double = 0.0

  // Método público que accede a la propiedad pública 'nombre'.
  def obtenerNombre: String = nombre

  // Método auxiliar para el estado interno (opcional, para demostrar la mutabilidad).
  def establecerSalario(monto: Double): Unit = {
    salario = monto // La mutación de 'salario' ocurre internamente
  }
}

val programador = new Empleado("Ana", 5) // Instanciación
println(programador.nombre)             // Acceso al campo 'nombre' (que es un val)
// programador.antiguedad              // ERROR: 'antiguedad' es privado
```

### 3. Objetos Singleton (`object`) y Companion Objects

Scala elimina el concepto de miembros `static` que se encuentran en Java,. En su lugar, utiliza el patrón **Singleton** a nivel del lenguaje mediante la palabra clave **`object`**.

Un `object` garantiza que solo existe **una instancia** en la JVM. Esto es ideal para:

1. **Utilidades (Métodos Estáticos):** Funciones que no dependen del estado de ninguna instancia.
2. **Módulos (Namespaces):** Agrupar constantes y lógica.
3. **Entrada de Aplicación:** Un `object` que extiende `App` o contiene un método `main` es el punto de entrada de la aplicación.

#### Companion Objects

Cuando un `object` comparte el mismo nombre que una `class` y está definido en el mismo archivo, se convierte en su **Companion Object**. Esta relación es privilegiada: **el objeto y su clase pueden acceder a los miembros privados del otro**.

- **Patrón Factory (`apply`):** El uso más común del Companion Object es definir un método **`apply`**. Si el objeto tiene un método `apply`, se puede invocar como si fuera el constructor de la clase, **sin usar la palabra clave `new`**.

#### Sintaxis del Companion Object y `apply`

```scala
// Clase. Nótese que el constructor es privado.
class ConexiónDB private (val url: String, val timeout: Int)

// Companion Object: Accede al constructor privado y ofrece una factoría segura.
object ConexiónDB {

  // 1. Método apply para instanciar sin 'new'
  def apply(url: String): ConexiónDB =
    new ConexiónDB(url, 5000) // Llama al constructor privado con timeout por defecto

  // 2. Miembro que se comportaría como un "static final"
  val CONEXION_LOCAL = ConexiónDB("jdbc:local")
}

// Uso: Se llama a ConexiónDB.apply("url")
val prodDb = ConexiónDB("jdbc:produccion:9000")
```

### 4. La Potencia de los Case Classes (`case class`)

Los **Case Classes** son una de las características más distintivas de Scala y la forma idiomática de modelar **datos inmutables**,. Fueron diseñados específicamente para ser utilizados en conjunción con el _Pattern Matching_.

La adición de la palabra clave `case` delante de una clase instruye al compilador a generar automáticamente código repetitivo (_boilerplate_) que Java requeriría manualmente.

#### Características Automáticas de `case class`:

1. **Constructor Simplificado:** El método `apply` se genera en el Companion Object, permitiendo la instanciación sin `new`.
2. **Inmutabilidad:** Todos los parámetros del constructor primario se convierten implícitamente en **campos `val` públicos**.
3. **Igualdad Estructural (`equals` y `hashCode`):** Compara el contenido de los campos, no solo la referencia.
4. **Representación en Cadena (`toString`):** Genera una representación legible que incluye los nombres de los campos.
5. **Copia Modificada (`copy`):** Crea un nuevo objeto inmutable a partir del original, permitiendo la modificación de campos específicos.
6. **Desestructuración (`unapply`):** Genera un método para la **desestructuración** automática en el _Pattern Matching_.

#### Sintaxis de Case Class

```scala
// Declaración concisa para modelar un registro de datos inmutable.
case class Transacción(id: Long, monto: BigDecimal, moneda: String, completada: Boolean = false)

// 1. Uso sin 'new' (gracias a apply en el Companion Object generado)
val tx1 = Transacción(100, BigDecimal(500.0), "USD")

// 2. Creación de una copia modificada (el original tx1 es inmutable)
// Se usa tx1 como la base, solo se cambia 'completada'.
val tx2 = tx1.copy(completada = true)
// tx2 es un nuevo objeto: Transacción(100, 500.0, "USD", true)
```

#### Comparativa con Clases de Datos en Java/Python

| Característica              | Java (POO/JDK 17+)                        | Python (Dinámico/dataclass)                 | Scala (Idiomático/Case Class)                           |
| :-------------------------- | :---------------------------------------- | :------------------------------------------ | :------------------------------------------------------ |
| **Declaración**             | `public record T(...) {}`                 | `@dataclass class T:`                       | **`case class T(...)`**.                                |
| **Inmutabilidad**           | Garantizada (por `record`/`final`).       | Por convención o librerías externas.        | **Garantizada por defecto**.                            |
| **Método `copy`**           | No generado automáticamente.              | No generado automáticamente.                | **Generado automáticamente**.                           |
| **Uso en Pattern Matching** | Parcial (disponible en `switch` de Java). | No aplica directamente (lenguaje dinámico). | **Integración nativa y potente** (gracias a `unapply`). |

### 5. Traits (`trait`): Composición y Abstracción

Los **Traits** son el mecanismo de Scala para la **herencia de comportamiento** y actúan como interfaces enriquecidas,.

- **Flexibilidad:** Un `trait` puede contener **métodos abstractos** (sin implementación) y **métodos concretos** (con implementación).
- **Mixins:** Una clase solo puede heredar de una única clase base (herencia simple), pero puede **mezclar (`mix in`) múltiples traits** usando la palabra clave **`with`**.
- **Composición:** Este mecanismo de _mixins_ resuelve el problema de la herencia múltiple de Java, favoreciendo la **composición de comportamientos** modulares.

#### Traits con Argumentos (Scala 3)

Una de las adiciones significativas en Scala 3 es que los `traits` ahora pueden aceptar **parámetros de constructor**. Esto permite que el _trait_ capture información al ser mezclado en una clase, haciendo que la composición sea más potente.

#### Sintaxis de Traits y Uso

```scala
// 1. Definición de un trait (comportamiento)
trait Validador[T] { // T es un tipo genérico que se validará
  def validar(data: T): Boolean
  // Método concreto (con implementación por defecto)
  def esRequerido: Boolean = true
}

// 2. Trait con argumento (Scala 3)
trait Trazabilidad(val idOperacion: String):
  def log(mensaje: String): Unit = println(s"[$idOperacion] $mensaje")

// 3. Clase que hereda de una clase base y mezcla dos traits (mixins)
class Servicio(nombre: String)
    extends ServicioBase(nombre) // Hereda de una clase base (máximo una)
    with Validador[String]       // Mezcla el primer trait
    with Trazabilidad("TX-456")  // Mezcla el segundo trait (con argumento de constructor)
{
  override def validar(data: String): Boolean = data.nonEmpty // Implementa el método abstracto

  def procesar(data: String): Unit =
    if (validar(data)) log(s"Datos válidos: $data")
}
```

### 6. Best Practices (_The Scala Way_)

El modelado de datos en Scala es un acto de equilibrio entre la concisión y la expresividad, dictado por el paradigma funcional-orientado a objetos.

| Objetivo de Modelado               | Herramienta Idiomática             | Justificación                                                                                                                                    |
| :--------------------------------- | :--------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Modelos de Datos Inmutables**    | **`case class`**                   | Genera automáticamente _boilerplate_ (`equals`, `copy`) y habilita la desestructuración y la inmutabilidad por defecto,.                         |
| **Comportamiento Reutilizable**    | **`trait`** y `with`               | Permite la composición modular de funcionalidades sin la rigidez de la herencia de clases,.                                                      |
| **Miembros Estáticos y Factories** | **`object`** (Singleton/Companion) | Reemplaza a `static` y debe contener los métodos de fábrica (`apply`) para la clase, asegurando una única instancia de la utilidad.              |
| **Inmutabilidad de Campos**        | **`val`** en constructores         | **Declarar `val` por defecto** en el constructor primario; evita `var` a menos que sea un estado interno de baja visibilidad (como en un Actor). |

> El enfoque de Scala es usar **`case class`** para _qué es_ el dato (modelado) y **`trait`** para _qué puede hacer_ el dato (comportamiento). Al preferir los `trait`s para la herencia y la composición, el desarrollador se enfoca en ensamblar capacidades, como si estuviera construyendo con bloques de Lego.