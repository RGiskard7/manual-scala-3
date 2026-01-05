## Capítulo 9: Type Classes y Métodos de Extensión

### 1. Explicación Teórica: Desacoplamiento de Comportamiento

Este capítulo abarca dos pilares de la Programación Funcional Avanzada en Scala 3: las **Type Classes** (Clases de Tipos) y los **Métodos de Extensión**, ambos íntimamente ligados al mecanismo de **Abstracciones Contextuales** (`given`/`using`) que vimos anteriormente.

#### Type Classes (Clases de Tipos): Composición de Comportamiento

Una **Type Class** es un patrón de diseño que permite añadir nuevo comportamiento a cualquier tipo de dato, incluso a aquellos cuyo código fuente no controlamos (como `String` o `Int` de la librería estándar), sin recurrir a la herencia o al subtipado.

La estructura de una Type Class sigue la regla de la **separación de intereses** (decoupling):

1. **Definición de la API (el Contrato):** Se define un _trait_ con uno o más parámetros de tipo genérico. Esto declara el comportamiento deseado (ej. cómo mostrarse, cómo ordenarse, cómo serializarse).
    
2. **Provisión de la Implementación (la Instancia):** Se utiliza la construcción **`given`** para proveer una implementación **canónica** de ese _trait_ para un tipo de dato específico (ej. `given Showable[Persona]`).
    

Este patrón es análogo a la interfaz `java.util.Comparator<T>` en Java, pero con una diferencia crucial: Scala utiliza el sistema de tipos y la inferencia automática (`given`/`using`) para inyectar la implementación necesaria en tiempo de compilación, en lugar de obligar al programador a pasar la implementación manualmente en cada llamada.

#### Métodos de Extensión: Enriquecimiento de Tipos

Los **Métodos de Extensión** (_Extension Methods_) son una característica de Scala 3 diseñada para **retroactivamente agregar nuevos métodos** a tipos ya existentes.

En versiones anteriores (Scala 2), esta funcionalidad se lograba mediante _clases implícitas_ (`implicit class`), un mecanismo potente pero que podía ser confuso y verboso. La palabra clave **`extension`** en Scala 3 formaliza este concepto directamente en el lenguaje, haciéndolo más claro y proveyendo mejores mensajes de error.

La combinación de _Extension Methods_ y _Type Classes_ es fundamental: permite que una vez que se ha probado que un tipo `A` conforma a una Type Class (es decir, existe un `given TC[A]`), se puedan invocar métodos de extensión en ese tipo `A` como si siempre hubieran sido parte de su API.

### 2. Sintaxis y Composición

#### A. Sintaxis de la Type Class (Trait Parametrizado)

Una Type Class es un _trait_ con al menos un parámetro de tipo que representa el tipo que se está extendiendo.

Scala

```scala
// 1. Definición de la Type Class (TC): Describe la capacidad
trait Sumable[T]:
  def sumar(a: T, b: T): T
```

#### B. Sintaxis de la Instancia (Provisión con `given`)

Se usa `given` para proporcionar la implementación canónica de la Type Class para un tipo concreto (ej. `Int`).

Scala

```scala
// 2. Implementación de la TC para Int (la Instancia Given)
given IntSumable: Sumable[Int] with
  def sumar(a: Int, b: Int): Int = a + b
  
// Esta instancia es el valor que el compilador inyectará.
```

#### C. Sintaxis del Consumo (Consumo con `using` y Context Bounds)

Hay dos formas idiomáticas de indicar que una función requiere una instancia `given`:

1. **Parámetros Contextuales (`using`):** Explícita y clara, pero puede ser verborrágica.
    
2. **Límites Contextuales (`[T: TC]`) (Context Bounds):** Más concisa, usada cuando la función solo necesita que la Type Class exista para usar sus métodos de extensión.
    

Scala

```scala
// Consumo con 'using' (parámetro nombrado)
def combinarValores[T](lista: List[T])(using s: Sumable[T]): T =
  lista.reduce(s.sumar) 

// Consumo con Límite Contextual (shorthand para decir que Sumable[T] debe existir)
// def combinarValores[T: Sumable](lista: List[T]): T = ...
```

#### D. Sintaxis del Método de Extensión (`extension`)

Los Métodos de Extensión se declaran fuera de la definición de la clase, pero actúan como si fueran internos.

Scala

```scala
// 3. Declaración del Método de Extensión (añadir método a String)
extension (s: String)
  // Nuevo método para String
  def esPalindromo: Boolean = s == s.reverse
  
val resultado = "ana".esPalindromo // Invocado como un método nativo
```

### 3. Ejemplos de Código Realistas

#### A. Type Class y Consumo (Ordenación Personalizada)

Definimos una Type Class `Ordering` para ordenar objetos `Persona` de forma predeterminada por su apellido, y luego consumimos ese comportamiento.

Scala

```scala
case class Persona(nombre: String, apellido: String, edad: Int)

// 1. Provisión de la Instancia Given (Ordering es un Type Class estándar)
// Define cómo ordenar una Persona.
given StandardPersonOrdering: Ordering[Persona] with
  // Implementa el método compare de Ordering
  override def compare(x: Persona, y: Persona): Int =
    x.apellido.compareTo(y.apellido) 

// 2. Consumo de la Type Class (usando límite contextual)
// La lista.sorted internamente requiere un Ordering[T] usando la cláusula 'using'.
def ordenarPorContexto[T: Ordering](lista: List[T]): List[T] =
  lista.sorted

val personas = List(
  Persona("Ana", "Smith", 30),
  Persona("Bob", "Jones", 25)
)

// El compilador inyecta StandardPersonOrdering automáticamente.
val listaOrdenada = ordenarPorContexto(personas)
// Resultado: Bob Jones, Ana Smith (ordenado por apellido)
```

#### B. Métodos de Extensión con Composición (Requisito Contextual)

Este ejemplo muestra una Type Class genérica (`Combinator`) que, a través de un Extension Method, añade la funcionalidad de sumarse a cualquier `List` si existe un `given Combinator[A]` para el tipo de elemento `A`.

Scala

```scala
// Type Class de Combinación (puede ser una suma, una concatenación, etc.)
trait Combinator[A]:
  def combine(x: A, y: A): A

// Instancia Given para Int
given IntCombinator: Combinator[Int] with
  def combine(x: Int, y: Int): Int = x + y

// Extension Method: Añade 'sumarTodo' a cualquier List[A] si A es Combinator
extension [A] (list: List[A])
  // El método requiere que exista una instancia Combinator[A] en el contexto
  def sumarTodo(using c: Combinator[A]): A =
    list.reduce((a, b) => c.combine(a, b))

val numeros = List(10, 20, 30)

// El compilador inyecta IntCombinator, permitiendo que List[Int] llame a sumarTodo
val resultadoSuma = numeros.sumarTodo 
// resultadoSuma: 60
```

### 4. Comparativa con Java y Python

La Type Class y los Extension Methods de Scala 3 abordan el dilema de añadir funcionalidad sin modificar la clase original, con ventajas significativas sobre los enfoques de lenguajes tradicionales.

|**Característica**|**Java (Interfaces/Anotaciones)**|**Python (Mixins/Decoradores)**|**Scala 3 (Type Classes/Extension)**|
|---|---|---|---|
|**Desacoplamiento**|Débil. Se requiere herencia explícita (`implements`) o pasar la lógica manualmente (`Comparator`).|Mixins de clases (acoplamiento estructural) o _monkey patching_ (dinámico, inseguro).|**Completo.** La Type Class se define externamente y la implementación se inyecta automáticamente en compilación.|
|**Extensión de API**|Inexistente para tipos cerrados (`String`). Requiere utilidades estáticas (`StringUtils.isPalindrome(s)`).|Flexible pero dinámico y no tipado estáticamente.|**Estático y Tipo-Seguro.** `extension` añade métodos como si fueran nativos (`"ana".esPalindromo`).|
|**Resolución de Dependencias**|En Java, las implementaciones de interfaces deben resolverse en _runtime_ o mediante anotaciones de DI (ej. Spring `@Autowired`).|No aplica (dinámico).|**Inferencia de Términos.** El compilador encuentra y resuelve la dependencia (`given`) en tiempo de compilación.|

### 5. Best Practices (_The Scala Way_)

La forma idiomática de usar estas abstracciones en Scala 3 maximiza la seguridad de tipos y la concisión del código:

1. **Utilizar `given` sobre `implicit`:** Aunque Scala 3 soporta el mecanismo `implicit` de Scala 2 para la compatibilidad, la forma moderna y recomendada para definir instancias canónicas (Type Class Instances o dependencias) es usar la palabra clave **`given`**.
    
2. **Usar `extension` para Enriquecer Tipos:** Reemplazar el uso de `implicit class` (si se migra de Scala 2) con la sintaxis **`extension`** de primera clase. Esto mejora la claridad y la inferencia de tipos.
    
3. **Consumo con Límites Contextuales:** Cuando una función solo necesita la Type Class para habilitar un método de extensión (es decir, no necesita usar la instancia explícitamente), se prefiere la sintaxis de **Límites Contextuales** (ej. `[T: Ordering]`) sobre el parámetro `using` completo. Esta es una forma concisa de decirle al compilador: "Asegúrate de que esta Type Class exista en el contexto".
    
4. **Combinar para la Composición:** La Type Class debe definir la capacidad (`trait`), la instancia `given` debe proporcionarla, y el `extension` debe exponer la nueva funcionalidad en un formato elegante e _infix_ (ej. `list.sumarTodo`).
    

> El patrón Type Class + Extension Methods es el **mecanismo de inyección de comportamiento a nivel de tipo de Scala**. En lugar de forzar a una clase a heredar (`extends`), se le permite _componer_ capacidades (el `trait`), y el compilador, actuando como un gestor de dependencias experto, inyecta la implementación correcta (`given`) para cualquier llamada que lo requiera (`using`).