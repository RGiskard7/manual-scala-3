## Capítulo 4: Funciones y Control de Flujo como Expresiones

### 1. Explicación Teórica: La Supremacía de la Expresión

Este capítulo marca la inmersión en la Programación Funcional (PF), que, junto con la Programación Orientada a Objetos (POO) pura, forma el paradigma híbrido de Scala. Para un desarrollador de Scala, las funciones y el control de flujo se conceptualizan bajo un principio fundamental: la **Programación Orientada a Expresiones** (_Expression-Oriented Programming_).

#### Funciones como Ciudadanos de Primera Clase

En la PF, las funciones no son meras rutinas o subprogramas; son **valores de primera clase**. Esto significa que una función puede ser:

1. Asignada a una variable.
2. Pasada como argumento a otras funciones (Funciones de Orden Superior o HOFs).
3. Devuelta como resultado de otra función.

#### Todo Devuelve un Valor

En Scala, **casi todas las estructuras se evalúan y devuelven un valor** (son expresiones), a diferencia de las _instrucciones_ (_statements_) que simplemente ejecutan una acción sin retornar un resultado.

La consecuencia más significativa de este enfoque es el **Retorno Implícito**: el valor de una función o bloque de código es automáticamente el valor de la **última expresión evaluada** dentro de dicho bloque. Esto elimina la necesidad de utilizar la palabra clave `return` en la mayoría de los casos.

---

### 2. Sintaxis y Uso de Funciones

#### Definición Básica y Retorno Implícito

La definición de funciones utiliza la palabra clave `def`. La sintaxis especifica el nombre, los parámetros de entrada con su tipo obligatorio, y opcionalmente el tipo de retorno, seguido del signo de igualdad '=' que indica que se devuelve un valor.

| Característica            | Sintaxis Gramatical                                                   |
| :------------------------ | :-------------------------------------------------------------------- |
| **Definición de Función** | `def nombre(parámetro: Tipo, ...): TipoRetorno = expresión`           |
| **Retorno Implícito**     | La última línea (expresión) dentro del cuerpo es el valor de retorno. |

#### Ejemplo de Función (Cálculo Financiero)

Este ejemplo ilustra el retorno implícito y la capacidad de las funciones de encapsular lógica compleja.

```scala
/**
 * Calcula el interés compuesto anual de un préstamo.
 * No requiere 'return'; la última línea es el valor de retorno implícito.
 * @param principal Monto inicial.
 * @param tasaInteres Tasa anual (ej: 0.05 para 5%).
 * @param anios Número de años.
 */
def calcularInteres(principal: Double, tasaInteres: Double, anios: Int): Double = {
  // El compilador infiere que este bloque retornará un Double

  // Expresión de cálculo (la última línea del bloque)
  principal * math.pow(1 + tasaInteres, anios)
}

// Uso:
val inversionInicial = 1000.0
val valorFinal = calcularInteres(inversionInicial, 0.05, 10)
// valorFinal: 1628.8946267774424 (Retorno implícito del cálculo)
```

#### Funciones Anónimas y Shorthand

Las **funciones anónimas** (o _lambdas_) son funciones sin nombre, utilizadas comúnmente como argumentos de otras funciones.

```scala
// Forma explícita (similar a Function1 trait)
val incrementador: Int => Int = (x: Int) => x + 1

// Forma idiomática (el compilador infiere el tipo)
val duplicador = (x: Int) => x * 2

// Forma más corta usando placeholder (_) si el argumento se usa una vez
val triplicador = _ * 3

// Uso:
val resultado = triplicador(7) // 21
```

---

### 3. Control de Flujo como Expresiones

Las estructuras de control tradicionales (`if-else`, bloques de código) se utilizan para **producir un valor** en el contexto funcional de Scala.

#### If/Else como Expresión

El `if-else` es una expresión que devuelve un valor que puede ser asignado directamente a un `val` inmutable.

| Comparativa: Java vs. Scala | Java (Instrucción / Ternario)                                       | Scala (Expresión)                                        |
| :-------------------------- | :------------------------------------------------------------------ | :------------------------------------------------------- |
| **Declaración**             | `String status = (edad >= 18) ? "Adulto" : "Menor";` (Usa ternario) | **`val status = if (edad >= 18) "Adulto" else "Menor"`** |

#### Ejemplo de Código (Lógica de Negocio)

```scala
// Definición inmutable de la tasa de comisión
def obtenerTasaComision(volumenVentas: Double): Double =
  val tasa =
    if (volumenVentas > 50000.0) then
      0.05 // 5% para ventas altas
    else if (volumenVentas > 10000.0) then
      0.03 // 3% para ventas medias
    else
      0.01 // 1% para ventas bajas

  // La última línea (tasa) se convierte en el valor de retorno implícito.
  tasa

val comisionVendedor =
  obtenerTasaComision(60000.0) * 60000.0
// comisionVendedor: 3000.0 (Calculado mediante una expresión if-else)
```

#### Bloques de Código como Expresiones

Un bloque de código delimitado por llaves `{}` se comporta como una expresión, retornando el valor de su última línea. Esto es útil para encapsular variables locales (`val`) temporales que se usan solo para calcular el valor final.

```scala
val resultadoComplejo: Double = {
  // Inicialización de valores locales (solo accesibles dentro de este bloque)
  val valorBase = 1000.0
  val ajusteImpuesto = 1.18

  // La última expresión es el valor devuelto por el bloque
  valorBase * ajusteImpuesto
} // resultadoComplejo: 1180.0
```

---

### 4. Iteración Funcional: HOFs y For Comprehensions

En Scala, se **evita la iteración imperativa** mediante bucles `for` o `while` al estilo Java/Python, ya que a menudo requieren el uso de variables mutables (`var`). La **forma idiomática** de trabajar con colecciones es a través de las **Funciones de Orden Superior (HOFs)**.

#### Funciones de Orden Superior (HOFs)

HOFs como `map`, `filter`, `fold` y `reduce` permiten transformar colecciones de manera declarativa y segura, devolviendo nuevas colecciones inmutables.

|Tarea|Código Imperativo (Java/Python style)|Código Funcional Idiomático (Scala)|
|:--|:--|:--|
|**Filtrar y Mapear**|Bucle explícito con asignación mutable.|**Composición de HOFs**: `list.filter(...).map(...)`|
|**Reducción**|Inicializar un acumulador (`var`).|**Uso de `fold` o `reduce`**.|

#### Ejemplo de Código (Procesamiento de Logs)

```scala
// Lista inmutable de códigos de estado HTTP
val codigosHTTP = List(200, 404, 500, 201, 403, 200, 301)

// 1. Uso de filter (predicado) para encontrar errores
val erroresCliente = codigosHTTP.filter(codigo => codigo >= 400 && codigo < 500)
// List(404, 403)

// 2. Uso de map para convertir cada código a su descripción (String)
val descripciones = erroresCliente.map(codigo => s"Error $codigo")
// List("Error 404", "Error 403")

// 3. Uso de foldLeft (reduce) para sumar la cantidad total de errores
// 0 es el valor inicial (seed), (total, _) es la función acumuladora
val conteoErrores = erroresCliente.foldLeft(0)((total, _) => total + 1)
// 3 (Se puede simplificar a .size, pero muestra el uso de fold/reduce)
```

#### For Comprehensions (Azúcar Sintáctico)

Una **for comprehension** no es un bucle; es azúcar sintáctico para encadenar llamadas a `map`, `flatMap` y `filter`. Se utiliza para operaciones complejas donde la composición directa con `map/flatMap` se volvería difícil de leer. El uso de `yield` es obligatorio para que la comprensión retorne una colección,.

```scala
val listaNumeros = List(1, 2, 3)
val listaLetras = List("a", "b")

// Generar todas las combinaciones (producto cartesiano) de numero y letra
val combinaciones =
  for {
    numero <- listaNumeros // flatMap implícito
    letra <- listaLetras    // map implícito
  } yield (numero, letra) // Resultado esperado (Tuple)

// combinaciones: List((1,a), (1,b), (2,a), (2,b), (3,a), (3,b))
```

---

### 5. Best Practices (_The Scala Way_)

El uso idiomático de Scala exige priorizar las abstracciones funcionales:

- **Inmutabilidad y Expresiones:** Siempre que sea posible, **modela la lógica de negocios utilizando expresiones** que devuelven un valor (`if-else`, `{bloques}`, `match`) y asigna el resultado a un `val` (valor inmutable), evitando así el estado mutable.
- **HOFs sobre Imperativo:** Para cualquier manipulación de colecciones (listas, conjuntos, mapas), utiliza `map`, `filter`, `flatMap` o `fold`,. La recursión es el mecanismo preferido en lugar de los bucles `while`.
- **Funciones Anónimas y Concisión:** Aprovecha la inferencia de tipos para mantener las declaraciones de funciones anónimas concisas, utilizando _placeholders_ (`_`) cuando sea apropiado, especialmente en llamadas a HOFs.

> **Comparativa con Java/Python:** En Scala, la lógica es declarativa. No estás diciendo _cómo_ iterar (instrucción mutable), sino _qué_ transformación aplicar a los datos (expresión inmutable). Esto se asemeja a describir una receta matemática donde el resultado está garantizado por la composición de sus partes, sin pasos intermedios que puedan arruinar el plato.