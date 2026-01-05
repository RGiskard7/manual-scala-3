# 🚀 Scala 3 Engineering Field Guide

**Versión:** 2025 (Scala 3.3+ LTS) | **Enfoque:** Ingeniería, Data & Sistemas Reactivos.

---

## 1. 🏗️ Fundamentos y Sintaxis Operativa

Scala es un lenguaje **orientado a expresiones**. Todo bloque de código computa y devuelve un valor.

### Mutabilidad y Evaluación

|**Palabra Clave**|**Comportamiento**|**Uso en Ingeniería**|
|---|---|---|
|**`val`**|**Inmutable**. Se evalúa al definir.|**95% del código.** Thread-safe por defecto.|
|**`var`**|**Mutable**. Reasignable.|Evitar. Solo para optimización local en algoritmos críticos.|
|**`lazy val`**|**Inmutable + Perezoso**. Se evalúa en el 1er acceso.|Para singletons costosos, grafos cíclicos o configs.|
|**`def`**|**Método**. Se re-evalúa en cada llamada.|Definición de funciones y métodos de API.|

### Jerarquía de Tipos Unificada

- **`Any`**: Padre de todo.
    
- **`AnyVal`**: Primitivos (`Int`, `Boolean`, `Double`) -> No reservan memoria en el Heap (usualmente).
    
- **`AnyRef`**: Objetos (`List`, `String`, `MyClass`) -> Reservan memoria en el Heap.
    
- **`Nothing`**: Subtipo de todo. Representa una ejecución que **nunca termina** (excepción o loop infinito).
    

### Sintaxis: Scala 2 vs Scala 3

Scala

```scala
// Estilo Clásico (Braces)      // Estilo Moderno (Indentation)
if (x > 0) {                    if x > 0 then
  println("Positive")             println("Positive")
} else {                        else
  println("Negative")             println("Negative")
}                               // Sin llaves, basado en indentación
```

---

## 2. 🧬 Modelado de Dominio (ADTs)

Scala brilla modelando datos complejos mediante **Tipos de Datos Algebraicos (ADTs)**.

### Estructuras de Datos

|**Estructura**|**Semántica**|**Características Clave (Boilerplate-free)**|
|---|---|---|
|**`case class`**|**Producto (AND)**|Inmutable, `equals`, `hashCode`, `copy`, `unapply` (pattern matching), serializable.|
|**`enum`**|**Suma (OR)**|Define un conjunto finito de variantes. Reemplaza a `sealed trait` complejos.|
|**`trait`**|**Comportamiento**|Interfaces con implementación. Soporta herencia múltiple (Mixins).|
|**`object`**|**Singleton**|Instancia única. Reemplaza a `static`. Companion Object = Factory.|

### Patrón ADT (El estándar de modelado)

Scala

```scala
// Definición del Dominio
enum PaymentStatus:
  case Pending
  case Processed(timestamp: Long)           // Con datos
  case Failed(reason: String)               // Con datos

// Lógica de Negocio (Pattern Matching Exhaustivo)
def handle(status: PaymentStatus): String = status match
  case PaymentStatus.Pending      => "Wait..."
  case PaymentStatus.Processed(t) => s"Paid at $t"
  case PaymentStatus.Failed(r)    => s"Error: $r" // Si borras esta línea, falla la compilación
```

---

## 3. 🛡️ Sistema de Tipos Avanzado

Herramientas para hacer estados ilegales irrepresentables.

### Nuevos Tipos en Scala 3

|**Tipo**|**Sintaxis**|**Significado**|**Caso de Uso**|
|---|---|---|---|
|**Union**|`A|B`|Es A **O** es B.|
|**Intersection**|`A & B`|Es A **Y** es B.|Composición de servicios (`Service & Repository`).|
|**Opaque**|`opaque type ID = String`|Es `String` en runtime, `ID` en compile.|**Zero-overhead wrapper**. Seguridad sin coste de rendimiento.|

### Varianza (Guía Rápida)

- **Covarianza `[+T]`**: `List[Gato]` es una `List[Animal]`. (Para **Outputs** / Lectura).
    
- **Contravarianza `[-T]`**: `Vet[Animal]` sirve como `Vet[Gato]`. (Para **Inputs** / Escritura / Acciones).
    

---

## 4. 🔮 Abstracciones Contextuales (La "Magia" Controlada)

Reemplazo explícito y modular de los `implicits`.

|**Keyword**|**Función**|**Traducción Mental**|
|---|---|---|
|**`given`**|Define una instancia.|_"Aquí hay un valor canónico de este tipo para quien lo pida"._|
|**`using`**|Requiere una instancia.|_"Necesito que el compilador me inyecte esto automáticamente"._|
|**`extension`**|Añade métodos.|_"Añádele el método `toJson` a la clase `String` sin heredar"._|

### Patrón Type Class (Polimorfismo Ad-hoc)

Scala

```scala
// 1. Contrato
trait Show[A]:
  def show(a: A): String

// 2. Instancia (Given)
given Show[Int] with
  def show(a: Int) = s"Número: $a"

// 3. Uso (Extension + Using)
extension [A](a: A)(using s: Show[A])
  def ver: String = s.show(a)

// Resultado
10.ver // "Número: 10"
```

---

## 5. 🌊 Colecciones Funcionales

Nunca uses bucles `for` o `while` para transformar datos. Usa combinadores.

|**Operación**|**Descripción**|**Snippet (One-liner)**|
|---|---|---|
|**`map`**|Transforma 1 a 1.|`users.map(_.email)`|
|**`flatMap`**|Transforma 1 a N (aplana).|`users.flatMap(_.orders)`|
|**`filter`**|Selecciona elementos.|`nums.filter(_ > 10)`|
|**`foldLeft`**|Reduce a un solo valor (acumulador).|`nums.foldLeft(0)(_ + _)`|
|**`collect`**|Filtra + Transforma (Pattern Match).|`data.collect { case i: Int => i * 2 }`|

### For-Comprehensions = flatMap + map

Scala

```scala
// Esto...
val result = for
  u <- users if u.active  // withFilter
  o <- u.orders           // flatMap
yield o.id                // map

// ...es azúcar sintáctico para:
users.withFilter(_.active).flatMap(u => u.orders.map(o => o.id))
```

---

## 6. ⚡ Arquitectura Reactiva (Concurrencia)

### Future vs. IO (El Gran Debate)

|**Característica**|**Future[T] (Std Lib)**|**IO[A] (Cats Effect / ZIO)**|
|---|---|---|
|**Modelo**|**Eager** (Se ejecuta al definirse).|**Lazy** (Es una descripción/blueprint).|
|**Hilo**|Atado a un `ExecutionContext` (OS Thread).|Se ejecuta en **Fibers** (Green Threads ligeros).|
|**Transparencia**|Rompe la transparencia referencial (Memoizado).|Pura. Transparencia Referencial total.|
|**Uso**|Legacy, Scripts simples.|**Arquitectura Moderna**, Microservicios.|

### Apache Pekko (Modelo de Actores)

Sistema distribuido basado en paso de mensajes.

- **Protocolo:** Definido con `enum` o `sealed trait`.
    
- **Behavior:** Define cómo reacciona el actor al próximo mensaje (`Behaviors.receiveMessage`).
    
- **Supervisión:** Estrategia "Let it Crash" (Reiniciar componente fallido).
    

---

## 7. 💾 Ingeniería de Datos (Spark & Doobie)

### Apache Spark (Scala API)

- **DataFrame:** Datos distribuidos con esquema en runtime (menos seguro).
    
    Scala
    
    ```scala
    df.select("name", "age").filter($"age" > 18) // Error si "age" no existe salta en Runtime
    ```
    
- **Dataset[T]:** Datos distribuidos fuertemente tipados (seguro).
    
    Scala
    
    ```scala
    ds.filter(user => user.age > 18) // Error salta en Compile Time
    ```
    

### Doobie (JDBC Funcional)

Acceso a DB puro. Nada se ejecuta sin `transact`.

Scala

```scala
val program: ConnectionIO[List[User]] =
  sql"select name from users".query[User].to[List]

// Ejecución controlada (El "Fin del Mundo")
program.transact(xa).unsafeRunSync()
```

---

## 🛑 Anti-Patrones vs. Buenas Prácticas

|**❌ NO HAGAS ESTO (Java Style)**|**✅ HAZ ESTO (Scala Way)**|**Por qué**|
|---|---|---|
|`return` explícito|Última expresión del bloque|`return` rompe el flujo en lambdas y refactorizaciones.|
|`null`|`Option[T]`|Evita `NullPointerException` con el compilador.|
|`throw new Exception`|`Either[Error, T]` o `Try[T]`|Trata el error como dato, no como interrupción.|
|`var` en clases|`case class` + `copy`|La inmutabilidad facilita la concurrencia.|
|Casteo `asInstanceOf[T]`|Pattern Matching|Es seguro y el compilador verifica tipos.|
|Bucles `while`|Recursión (`@tailrec`) o `fold`|Mantiene el estado inmutable y evita side-effects.|