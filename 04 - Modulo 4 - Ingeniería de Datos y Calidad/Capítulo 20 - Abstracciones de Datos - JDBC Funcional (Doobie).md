## Capítulo 20. Abstracciones de Datos: JDBC Funcional (Doobie)

### 1. Explicación Teórica: Desacoplamiento de Efectos y Seguridad

**Doobie** es la solución idiomática en el ecosistema de Scala para la interacción con bases de datos relacionales a través de JDBC (Java Database Connectivity). Se describe como una **capa JDBC puramente funcional para Scala**.

#### El Problema de la Impureza en JDBC

En la programación tradicional, las operaciones de base de datos (E/S o I/O) son inherentemente **impuras** e **imperativas**. Un código que abre una conexión, ejecuta una consulta, maneja el resultado y luego cierra la conexión (a menudo en un bloque `try/catch/finally` en Java,) presenta varios problemas:

1. **Mutabilidad de Estado:** La conexión es un recurso mutable gestionado manualmente.
2. **Efectos Secundarios:** Las excepciones (como `SQLException`) interrumpen el flujo de ejecución del programa, rompiendo el principio de **transparencia referencial** de la Programación Funcional (PF).
3. **Gestión de Recursos (Resource Safety):** El desarrollador debe garantizar manualmente que la conexión se cierre, lo que es propenso a errores.

#### Doobie: El Efecto como Valor Inmutable

Doobie resuelve estos problemas encapsulando la interacción con la base de datos dentro de la **mónada de efectos** `ConnectionIO[A]`. Al igual que la mónada `IO` (de Cats Effect o ZIO) un valor `ConnectionIO` no ejecuta el código inmediatamente; es una **descripción inmutable** o un **"plano"** (_blueprint_) de la acción que debe realizarse en la base de datos.

La ejecución de este "plano" se delega a una capa externa, el **Transactor (`Transactor[F]`)**, que es responsable de:

- Obtener conexiones de un _pool_.
- Manejar las transacciones (commit/rollback).
- Garantizar el **cierre seguro de recursos**, eliminando la necesidad de bloques `finally` explícitos,,.

El enfoque funcional puro de Doobie utiliza el sistema de tipos avanzado de Scala para **comprobar la seguridad del tipo de las consultas SQL en tiempo de compilación**, previniendo errores comunes de mapeo entre Scala y la base de datos.

### 2. Sintaxis y Tipos Centrales

La sintaxis de Doobie se basa en _for-comprehensions_ y _string interpolators_ para expresar consultas SQL de forma segura y componible.

#### A. Tipos Clave

|Tipo|Descripción|Función Principal|
|:--|:--|:--|
|**`ConnectionIO[A]`**|La mónada de efecto principal. Representa una **descripción** de una acción que se ejecuta en el contexto de una conexión (transacción). `A` es el valor de retorno (ej. `Int`, `Usuario`).|Se utiliza para componer la lógica de negocio que interactúa con la DB.|
|**`Transactor[F]`**|El motor de ejecución. Convierte el valor `ConnectionIO[A]` en el efecto externo `F[A]` (donde `F` es típicamente `IO` de Cats Effect o ZIO), delegando la gestión de la conexión.|Se gestiona como una **dependencia** (Environment).|

#### B. Query DSL (String Interpolators)

Doobie utiliza _string interpolators_ especiales para inyectar variables de Scala de forma segura y construir consultas, evitando la inyección SQL (SQL Injection).

|Interpolador|Sintaxis Gramatical|Propósito|
|:--|:--|:--|
|**`sql` (Raw)**|`sql"SELECT * FROM tabla"`|Se usa para el texto SQL estático.|
|**`fr` (Fragmento)**|`fr"SELECT * FROM tabla WHERE id = ${miId}"`|El interpolador preferido. Permite la inyección segura de valores (los convierte en `?` en el SQL, y los valores se pasan como parámetros de JDBC).|

### 3. Ejemplos de Código Realistas: CRUD Funcional

Este ejemplo muestra la composición de operaciones de base de datos (creación de esquema, inserción, y consulta) utilizando la mónada `ConnectionIO`.

```scala
import cats.effect.IO // Efecto externo (monad IO)
import doobie._
import doobie.implicits._ // Habilita los interpoladores 'sql' y 'fr'
import doobie.util.transactor.Transactor
import doobie.hikari.HikariTransactor // Transactor de conexión

// 1. Modelo de Datos (representación de la fila de la DB)
case class Usuario(id: Long, nombre: String, email: String)

// 2. Definición del Transactor (Configuración - Se haría una vez al inicio del sistema)
// El Transactor es la dependencia que sabe cómo conectarse a la DB.
val xa: Transactor[IO] = Transactor.fromDriverManager[IO](
  "org.postgresql.Driver",
  "jdbc:postgresql:mydb",
  "user",
  "password"
)

// --- OPERACIONES EN CONNECTIONIO ---

// 3. Crear el Esquema (Efecto de IO)
// Un ConnectionIO[Int] que representa el número de filas modificadas (1 si se crea).
val crearTabla: ConnectionIO[Int] =
  sql"""
    CREATE TABLE IF NOT EXISTS usuarios (
      id BIGSERIAL PRIMARY KEY,
      nombre VARCHAR NOT NULL,
      email VARCHAR NOT NULL
    )
  """.update.run

// 4. Inserción Segura de Datos (Inyección de Parámetros)
// Usamos el interpolador 'fr' (fragmento) para inyectar variables de Scala.
def insertarUsuario(nombre: String, email: String): ConnectionIO[Usuario] =
  val insercion = fr"INSERT INTO usuarios (nombre, email) VALUES ($nombre, $email)"

  insercion.update.withUniqueGeneratedKeys[Long]("id").map { idGenerado =>
    Usuario(idGenerado, nombre, email) // Devuelve el objeto Usuario con el ID real.
  }

// 5. Consulta de Datos (Mapeo de Filas)
// .query[Usuario] indica a Doobie cómo mapear las columnas al case class Usuario.
val obtenerUsuarios: ConnectionIO[List[Usuario]] =
  sql"SELECT id, nombre, email FROM usuarios".query[Usuario].to[List]

// --- COMPOSICIÓN Y EJECUCIÓN ---

// 6. Composición Funcional (for-comprehension)
// Componemos las acciones de DB como un único ConnectionIO[List[Usuario]].
val flujoCompleto: ConnectionIO[List[Usuario]] =
  for {
    _ <- crearTabla                                 // Se ejecuta (produce un Int, pero lo ignoramos)
    _ <- insertarUsuario("Ana García", "ana@test.com")
    _ <- insertarUsuario("Luis Pérez", "luis@test.com")
    usuarios <- obtenerUsuarios                      // Obtenemos la lista
  } yield usuarios

// 7. Ejecución (al "final del universo")
// Transactor.transact lo envuelve en una transacción segura y lo ejecuta.
val resultadoIO: IO[List[Usuario]] = flujoCompleto.transact(xa)

// Para correr la aplicación, se llamaría a 'resultadoIO.unsafeRunSync()' o se integraría
// en un framework de efectos como IOApp (Cats Effect) o ZIOApp (ZIO).
```

### 4. Comparativa con Java o Python

La diferencia fundamental entre Doobie y las herramientas tradicionales de acceso a bases de datos es el **cambio de paradigma de imperativo a puramente funcional**, lo que aumenta la seguridad en tiempo de compilación y la resiliencia en _runtime_.

|Aspecto|Java (JDBC o JPA/Hibernate)|Python (SQLAlchemy ORM/Raw)|Scala (Doobie)|
|:--|:--|:--|:--|
|**Manejo de I/O**|Imperativo. Requiere `try-catch` para excepciones y `finally` para cerrar `Connection` y `ResultSet`,.|Orientado a objetos o procedural.|**Funcional Puro.** La interacción con la DB se modela como un valor `ConnectionIO`.|
|**Transacciones**|Gestión manual de `.setAutoCommit(false)` y `.commit()`/`.rollback()`.|Gestión manual o por capa de abstracción de ORM.|**Automática y Declarativa.** El `.transact(xa)` envuelve todo el bloque `ConnectionIO` en una única transacción atómica.|
|**Seguridad de SQL**|Inyección SQL es un riesgo si se construye el _string_ de consulta manualmente.|Similar a Java.|**Seguridad en Tipos.** El interpolador `fr` garantiza que los parámetros de Scala sean _bind_ de JDBC, previniendo inyección SQL.|
|**Composición**|Difícil de componer pasos transaccionales sin anidación de bloques o _callbacks_.|Limitada.|**Composición Monádica.** Los pasos se encadenan de forma secuencial y legible con _for-comprehensions_ (`map`/`flatMap`).|
|**Recursos**|Recolección de basura de objetos de conexión lentos (JVM).|N/A.|**Gestión de Recursos Segura.** Utiliza abstracciones funcionales (_Resource_) del motor de efectos (`IO`) para garantizar que la conexión se cierre automáticamente.|

### 5. Best Practices (_The Scala Way_)

El uso de Doobie en Scala 3 representa el camino funcional ideal para manejar la persistencia:

1. **Mantenimiento de la Pureza:** La lógica de negocio que interactúa con la base de datos debe residir exclusivamente en el tipo **`ConnectionIO[A]`**. Esto garantiza que las operaciones de la base de datos sean puras, inmutables y no se ejecuten hasta que se llamen a `transact`.
2. **Uso de `case class` para Mapeo:** Define siempre las filas de la base de datos con **`case class`**,. Doobie puede inferir automáticamente cómo mapear las columnas al `case class` con seguridad de tipos (siempre y cuando los nombres de las columnas coincidan).
3. **Prevención de Inyección SQL:** **Nunca utilices interpolación de _strings_ estándar (ej. `s"..."`) para construir consultas SQL que incluyan variables de usuario.** Utiliza siempre el interpolador **`fr`** de Doobie para inyectar valores de forma segura como parámetros de JDBC.
4. **Inyección de Dependencias (Transactor):** El `Transactor[F]` es un recurso de infraestructura que debe tratarse como una **dependencia**. En arquitecturas funcionales modernas (como ZIO o Cats Effect), el Transactor se gestiona como un recurso global disponible a través del sistema de inyección de dependencias (`ZLayer` en ZIO o `Resource[IO, Transactor]` en Cats Effect).

> Doobie actúa como un **notario funcional** entre tu código Scala y la base de datos. En lugar de ejecutar órdenes directamente (imperativo), tú le entregas a Doobie una **descripción** detallada de lo que debe hacer (el `ConnectionIO`). El compilador, mediante la seguridad de tipos, certifica que esa descripción es correcta y segura (libre de inyección SQL), y el Transactor se encarga de ejecutarla en un **entorno controlado y a prueba de fallos** (la transacción).