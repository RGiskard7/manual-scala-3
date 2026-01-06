## Capítulo 25: Serialización JSON - Circe y uPickle

### 1. Explicación Teórica

En la arquitectura de microservicios y APIs modernas, la serialización es el proceso crítico de traducir modelos de dominio inmutables a formatos de intercambio de datos, principalmente **JSON**. En Scala 3, este proceso ha evolucionado desde el uso intensivo de macros complejas hacia un sistema basado en la **derivación por Mirror**, donde el compilador sintetiza metadatos de las _case classes_ y _enums_ en tiempo de compilación, eliminando el _overhead_ en tiempo de ejecución.

La elección de una librería de serialización en un entorno industrial suele balancear dos necesidades: **integración con el ecosistema** y **rendimiento extremo**. **Circe** es la opción predilecta para proyectos basados en el stack functional (Typelevel), ofreciendo una integración nativa con Cats y http4s mediante el uso de _Type Classes_ puras. Por otro lado, **uPickle** destaca como el líder en rendimiento, superando a Circe y librerías basadas en Java (como Jackson) por márgenes significativos al evitar la construcción de árboles de sintaxis abstracta (AST) intermedios cuando se serializan tipos estáticos directamente.

### 2. Sintaxis y Estructura

La serialización en Scala 3 se apoya en la palabra clave `derives`, que permite al compilador generar automáticamente las instancias necesarias para convertir datos.

**Tipos y Dependencias Principales:**

|Librería|Tipo de Clase Central|Filosofía Industrial|
|:--|:--|:--|
|**Circe**|`Encoder[A]` / `Decoder[A]`|Basada en Cats, funcional y altamente transformable.|
|**uPickle**|`ReadWriter[A]`|Rápida, intuitiva y con cero dependencias externas.|

**Comparativa de Rendimiento (MacBook Pro i7):**

- **uPickle:** Lectura (~613,727 ops/ms) / Escritura (~1,041,798 ops/ms).
- **Circe:** Lectura (~132,519 ops/ms) / Escritura (~441,906 ops/ms).

### 3. Ejemplos de Código Realistas

#### A. Derivación en Scala 3 con Circe y http4s

Circe utiliza el mecanismo de derivación nativo de Scala 3 para crear codecs de forma concisa.

```scala
import io.circe.Codec
import io.circe.syntax._

// El compilador genera el codec usando Mirror en tiempo de compilación
case class Producto(id: String, precio: Double) derives Codec.AsObject

val laptop = Producto("macbook-m3", 2500.0)
val json = laptop.asJson.noSpaces // {"id":"macbook-m3","precio":2500.0}
```

#### B. ADTs y Jerarquías Selladas con uPickle

uPickle maneja las jerarquías de tipos (Sum Types) añadiendo un campo especial `$type` para identificar la variante durante la deserialización.

```scala
import upickle.default._

// Definición de un ADT (Algebraic Data Type)
sealed trait Evento derives ReadWriter
case class Login(user: String) extends Evento
case class Logout(user: String) extends Evento

// Serialización con etiqueta de tipo
val loginEvent = write(Login("admin"))
// Resultado: {"$type":"Login","user":"admin"}
```

#### C. Integración con Tapir

Tapir permite definir el cuerpo de una petición como un dato tipado, delegando la serialización a Circe o uPickle según se prefiera.

```scala
import sttp.tapir._
import sttp.tapir.json.circe._ // O .upickle para uPickle

// Definición del endpoint como "Código como Dato"
val productoEndpoint = endpoint
  .post
  .in("api" / "productos")
  .in(jsonBody[Producto]) // Serialización automática garantizada por el compilador
  .out(stringBody)
```

### 4. Comparativa con Java/Python

|Aspecto|Java (Jackson/Gson)|Python (json/pydantic)|Scala (Circe/uPickle)|
|:--|:--|:--|:--|
|**Mecanismo**|Reflexión pesada en _runtime_.|Dinámico y mutable.|Derivación estática por **Mirror**.|
|**Seguridad**|Errores comunes de casting al leer JSON.|Sin chequeo en compilación.|**Seguridad de tipos total** en tiempo de compilación.|
|**Inmutabilidad**|Difícil con modelos inmutables sin configuraciones extra.|Mutable por defecto.|**Inmutabilidad rigurosa**; los cambios generan nuevas copias.|
|**Rendimiento**|Penalización por metadatos dinámicos.|Lento debido al intérprete.|**Costo cero** en la abstracción (uPickle).|

### 5. Best Practices (The Scala Way)

1. **Preferir Derivación Semi-automática en APIs:** Aunque la derivación automática (`generic.auto`) es cómoda para prototipos, en producción se recomienda la semi-automática o el uso de `derives`. Esto evita que un cambio menor en una _case class_ rompa la compatibilidad binaria sin previo aviso del compilador.
2. **Manejar Errores como Valores:** No lances excepciones si la deserialización falla. Circe devuelve un `Either[ParsingFailure, A]`, lo que permite componer el manejo de errores funcionalmente usando `map` o `flatMap`.
3. **Default values para Evolución de Esquemas:** Define valores por defecto en los constructores de tus _case classes_. Librerías como uPickle utilizarán estos valores si un campo falta en el JSON, permitiendo una evolución segura del esquema sin romper clientes antiguos.
4. **Uso de Snake Case mediante Configuración:** Evita renombrar campos manualmente. Utiliza módulos de configuración (como `generic-extras` en Circe o configuraciones personalizadas en uPickle) para transformar automáticamente `camelCase` de Scala a `snake_case` de JSON.

---

**Metáfora de Ingeniería:** La serialización en Scala 3 es como un **molde de fundición de alta precisión**; el compilador (el Mirror) conoce cada relieve y curva de tu dato (la _case class_) y crea una réplica perfecta en JSON sin necesidad de inspeccionar la pieza manualmente cada vez que sale de la fábrica.