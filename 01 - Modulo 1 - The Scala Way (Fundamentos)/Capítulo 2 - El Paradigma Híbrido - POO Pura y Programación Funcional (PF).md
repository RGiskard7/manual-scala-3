## Capítulo 2: El Paradigma Híbrido: POO Pura y Programación Funcional (PF)

### 1. Explicación Teórica: La Fusión de Paradigmas

Scala (acrónimo de _Scalable Language_) fue diseñado por Martin Odersky con el propósito central de **fusionar la Programación Orientada a Objetos (POO) y la Programación Funcional (PF) en una sola ontología**. Este enfoque multiparadigma permite a los desarrolladores aprovechar la madurez de la POO en la JVM y la expresividad y seguridad de la PF.

#### POO Pura (Todo es un Objeto)

A diferencia de Java o C++, que distinguen entre tipos primitivos y objetos de referencia, Scala adopta un modelo de objetos **puro**: **todo valor es un objeto** y **toda operación es una llamada a un método**.

- **Jerarquía Unificada:** Todos los tipos en Scala descienden de un tipo raíz común llamado `Any`. Los tipos de valor (`Int`, `Boolean`), conocidos como `AnyVal`, y los tipos de referencia (`AnyRef`, equivalentes a `java.lang.Object`), están unidos bajo esta misma jerarquía. Esta uniformidad permite la escritura de código genérico que funciona consistentemente con números y objetos complejos.

#### Programación Funcional (Funciones de Primera Clase)

Scala incorpora la PF al tratar las **funciones como valores de primera clase**. Esto significa que una función es tratada como cualquier otro dato (como un entero o una cadena de texto).

- **Composición Funcional:** Las funciones pueden ser **asignadas a variables**, pasadas como argumentos a otras funciones (Funciones de Orden Superior o HOFs), o devueltas como resultado. HOFs como `map`, `filter`, y `reduce` son esenciales para transformar colecciones de datos de manera declarativa, sin recurrir a bucles imperativos.
- **La Clave Arquitectónica:** Internamente, una función de Scala (`FunctionX`) es compilada a un objeto, una instancia de un _trait_ con un método `apply`. Esta es la razón técnica por la que la JVM, diseñada para objetos, puede ejecutar sin problemas el paradigma funcional.

#### El Sello de Inmutabilidad: `val` vs. `var`

La inmutabilidad no es una mera preferencia estilística, sino una **decisión de ingeniería fundamental** en Scala para construir **sistemas concurrentes seguros**.

|Palabra Clave|Propósito|Mutabilidad|Uso Idiomático|
|:--|:--|:--|:--|
|**`val`**|**Valor** (Value). Es una referencia inmutable. Su valor **no puede cambiar** una vez asignado.|Inmutable|**Usar por defecto**. Equivale a `final` en Java.|
|**`var`**|**Variable** (Variable). Es una referencia mutable. Su valor puede ser reasignado.|Mutable|**Evitar**, se limita a casos de optimización o manejo de estado controlado (ej. dentro de un Actor).|

#### Programación Orientada a Expresiones (_Expression-Oriented Programming_)

En Scala, **casi todas las estructuras de control se evalúan y devuelven un valor** (son expresiones).

- **Retorno Implícito:** El valor de una función o bloque de código (`{}`) es el valor de la **última expresión evaluada**. Esto hace que la palabra clave `return` sea innecesaria en la mayoría de los casos.

### 2. Sintaxis y Ejemplos de Código (Scala 3)

Los siguientes ejemplos ilustran la inmutabilidad por defecto y el uso de estructuras de control como expresiones, pilares de la programación idiomática en Scala 3.

#### Inferencia de Tipos e Inmutabilidad

Scala es fuertemente tipado estáticamente, pero el compilador es **lo suficientemente potente para inferir el tipo** basándose en el valor asignado.

```scala
// El compilador infiere Int (entero)
val numeroMagico = 42
// numeroMagico: Int = 42

// Intentar reasignar un val causa un error de compilación
// numeroMagico = 99
// Error: reassignment to val

// Uso de var (mutable)
var contador: Int = 0
// El uso de var está desaconsejado, pero es necesario para la mutabilidad explícita.
contador = contador + 1 // Válido
```

#### Control de Flujo como Expresiones

La estructura `if-else` y los bloques de código devuelven un valor.

```scala
// Definición de una función simple
// El compilador infiere que retorna String, eliminando la necesidad de escribir ': String'
def clasificarEdad(edad: Int) =
  // If-Else como expresión (no se necesita el operador ternario de C/Java).
  val tipo =
    if edad >= 65 then
      "Jubilado"
    else if edad >= 18 then
      "Adulto"
    else
      "Menor"

  // La última expresión del bloque (tipo) es retornada implícitamente.
  tipo

// Bloque de código como expresión
val resultadoBloque: Int = {
  val a = 5 // Variable local dentro del bloque
  val b = 7
  a + b // El valor del bloque es 12
}
```

### 3. Comparativa (Java y Python)

El paradigma híbrido de Scala resuelve problemas de verbosidad y seguridad inherentes a la POO tradicional y a los lenguajes dinámicos.

|Concepto|Java (Imperativo/POO)|Python (Dinámico)|Scala (Híbrido/Funcional)|
|:--|:--|:--|:--|
|**Inmutabilidad**|Opcional, requiere `final`. Mutable por defecto.|Por convención; las variables son reasignables.|**`val` (inmutable) es el valor por defecto**. Mutabilidad explícita con `var`.|
|**Funciones**|Métodos atados a clases (`static` o instancia).|Valores de primera clase (lambdas).|**Valores de primera clase** encapsulados en objetos (`FunctionX` traits).|
|**Jerarquía de Tipos**|Distinción entre primitivos (`int`) y objetos (`Integer`).|Tipado dinámico.|**Todo es un objeto** (`Any`), eliminando la necesidad de _autoboxing_.|
|**Control de Flujo**|Estructuras como `if` son **instrucciones** (_statements_). Necesita `return` para devolver valor de un método.|Instrucciones o expresiones especializadas.|**Todo es una expresión**. El valor de retorno es implícito.|

### 4. Best Practices (_The Scala Way_)

La forma idiomática de programar en Scala se centra en maximizar la inmutabilidad y la expresividad compositiva que ofrece la PF:

1. **Imponer la Inmutabilidad:** Declarar siempre **`val`** a menos que la mutabilidad sea estrictamente necesaria. Esto minimiza la posibilidad de _race conditions_ y simplifica la lógica en sistemas concurrentes.
2. **Abrazar las Expresiones:** Evitar el uso explícito de `return` y modelar la lógica de negocios utilizando estructuras de control que retornan valor (`if-else`, `match`), asignando directamente el resultado a un `val`.
3. **Composición Funcional sobre Iteración:** Para trabajar con colecciones (`List`, `Vector`, `Set`), utilizar siempre **Funciones de Orden Superior (HOFs)** como `map`, `filter`, y `fold` en lugar de bucles `for` o `while` al estilo imperativo.
4. **Diseño Modular:** Utilizar **`object`** (Singleton) para funciones utilitarias y métodos que habrían sido `static` en Java, manteniendo la semántica de la POO pura.
5. **Seguridad de Tipos sin Redundancia:** Confiar en la **inferencia de tipos** del compilador para un código conciso. Solo declarar el tipo explícitamente cuando se define una API pública o cuando la inferencia podría resultar ambigua.