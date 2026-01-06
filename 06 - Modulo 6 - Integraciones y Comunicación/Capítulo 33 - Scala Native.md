## Capítulo 33: Scala Native

### 1. Explicación Teórica

**Scala Native** es un compilador optimizador y una implementación de la biblioteca estándar de Scala que traduce el código directamente a **código de máquina nativo** utilizando LLVM. A diferencia del modelo estándar que compila a Java Bytecode para la JVM, Scala Native emplea una compilación **Ahead-of-Time (AOT)**, lo que permite generar binarios ejecutables independientes. Este enfoque es crítico en entornos industriales para desarrollar herramientas de línea de comandos (CLI), microservicios de baja latencia y daemons de sistema que requieren un **arranque instantáneo** y un consumo de memoria mínimo.

La importancia de Scala Native radica en su capacidad para romper la dependencia de la infraestructura pesada de la JVM, permitiendo que Scala compita directamente con lenguajes como **Rust, Go y C++** en sistemas donde el tiempo de precalentamiento (warm-up) del JIT es inaceptable. Al integrarse con el **Scala Toolkit**, permite a los desarrolladores escribir lógica de sistemas de alto nivel con la seguridad del sistema de tipos de Scala 3, mientras mantienen la capacidad de interactuar directamente con bibliotecas de C mediante una capa de interoperabilidad de costo cero.

### 2. Sintaxis y Estructura

La arquitectura de Scala Native se apoya en el ecosistema de **Scala CLI** para facilitar la compilación y el empaquetado sin configuraciones complejas.

**Configuración de Empaquetado (Scala CLI):**

|Comando|Propósito|Resultado|
|:--|:--|:--|
|`scala-cli package . --native`|Compilación nativa estándar.|Binario ejecutable para la plataforma actual.|
|`scala-cli --power package --native-image`|Uso de GraalVM/Native Image.|Alternativa de binario estático optimizado.|

**Interoperabilidad con C:** Para llamar a funciones de C, Scala Native utiliza anotaciones especiales y tipos que mapean la memoria de forma segura. Se emplean tipos como `Ptr[Byte]` para punteros y la palabra clave `extern` para definir firmas de funciones externas.

### 3. Ejemplos de Código Realistas

#### A. Herramienta CLI de Alto Rendimiento (Scala Toolkit)

Este ejemplo crea una utilidad nativa que lista archivos filtrando por tamaño instantáneamente, aprovechando `os-lib`.

```scala
//> using scala "3.3.4"
//> using toolkit latest
//> using platform native

import os._

@main def listBigFiles(path: String, minSize: Long): Unit =
  // Operación de sistema de archivos rápida y nativa
  val target = Path(path, pwd)
  os.list(target)
    .filter(p => os.isFile(p) && os.size(p) > minSize)
    .foreach(p => println(s"${p.last} -> ${os.size(p)} bytes"))
```

#### B. Interoperabilidad con C (Llamada a `printf`)

Scala Native permite acceder a funciones del sistema operativo con una sobrecarga nula.

```scala
import scala.scalanative.unsafe._

// Definición del contrato con C
@extern
object stdio:
  def printf(format: CString, args: Any*): CInt = extern

@main def runNative(): Unit =
  Zone { implicit z =>
    // 'c' es un interpolador para crear CStrings inmutables
    stdio.printf(c"Hola desde Scala Native 3, el número es %d\n", 42)
  }
```

### 4. Comparativa con Java/Python

|Aspecto|Java (JVM)|Python (Interpreted)|Scala Native|
|:--|:--|:--|:--|
|**Tiempo de Arranque**|Lento (Carga de clases/JIT).|Rápido (pero ejecución lenta).|**Instantáneo (Binario nativo)**.|
|**Uso de Memoria**|Alto (Requiere Heap de la JVM).|Medio/Alto por objeto.|**Mínimo (Solo el código necesario)**.|
|**Distribución**|Requiere JRE instalado.|Requiere intérprete/entorno.|**Un solo archivo binario**.|
|**Interoperabilidad**|Vía JNI (Complejo y lento).|Vía C-Extensions (Manual).|**Nativa y directa (C-Interop)**.|

### 5. Best Practices (The Scala Way)

1. **Evitar la Reflexión de Runtime:** Dado que Scala Native es AOT, la reflexión pesada (como la usada en Spring) no es compatible. Prefiere la **derivación basada en Mirror** para serialización JSON con uPickle o Circe.
2. **Gestión de Memoria en Interop:** Al interactuar con C, utiliza el patrón `Zone` para delimitar el tiempo de vida de la memoria asignada manualmente, asegurando que se libere al final del bloque.
3. **Usar el Scala Toolkit para Portabilidad:** Aunque el código sea nativo, usa bibliotecas del Scala Toolkit. Esto garantiza que el mismo código pueda compilarse para JVM o JS si es necesario en el futuro.
4. **Inmutabilidad Rigurosa:** Aunque trabajes a bajo nivel, mantén el uso de `val` y `case classes`. Scala Native optimiza estas estructuras inmutables de forma muy eficiente en binarios.
5. **Aislar el Código Nativo:** Utiliza directorios `src/main/scala-native` para lógica que use `unsafe` o llamadas a C, manteniendo el resto del dominio en carpetas compartidas.

---

**Metáfora de Ingeniería:** Scala Native es como pasar de viajar en un **crucero transatlántico (JVM)**, que es inmenso, potente y lleno de servicios pero difícil de arrancar y estacionar, a pilotar una **moto de carreras monoplaza (Native)**. La moto solo lleva el motor y el combustible exacto que necesita para la pista, permitiéndote arrancar y alcanzar la máxima velocidad en una fracción de segundo.