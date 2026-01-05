# Manual Definitivo de Scala 3

## Indice General

Este documento proporciona una vista secuencial de todo el contenido del manual, con enlaces directos a cada capitulo en formato Markdown y PDF.

---

## Recursos Generales

| Recurso | Formato |
|---------|---------|
| [Guia Rapida (Cheatsheet)](./CHEATSHEET%20-%20Guia%20rapida.md) | Markdown |
| [Infografia del Ecosistema Scala](./Infografía_scala.png) | Imagen |
| [Manual Completo](./assets/pdf/ZZ%20-%20Manual%20Definitivo%20de%20Scala%203%20-%20De%20Cero%20a%20Ingeniería%20Reactiva.pdf) | PDF |
| [Indice PDF](./assets/pdf/00%20-%20Indice.pdf) | PDF |

---

## Introduccion

Fundamentos conceptuales y guia de transicion para desarrolladores Java/Python.

| Documento | Descripcion | Formato |
|-----------|-------------|---------|
| [Desmitificando la Programacion Funcional](./00%20-%20Introduccion/00%20-%20Desmitificando%20la%20Programación%20Funcional%20en%20Scala%20-%20Una%20Guía%20para%20Principiantes.md) | Los 4 pilares de la PF: inmutabilidad, expresiones, funciones como valores, errores como valores | Markdown |
| [Los Pilares Conceptuales de Scala](./00%20-%20Introduccion/00%20-%20Los%20Pilares%20Conceptuales%20de%20Scala%20-%20El%20Paradigma%20Híbrido.md) | Ontologia unificada POO + PF, jerarquia de tipos, sintaxis clave | Markdown |
| [Guia de Transicion: Java/Python a Scala 3](./00%20-%20Introduccion/02%20-%20Guía%20de%20Transición%20v2%20-%20De%20Java%20y%20Python%20a%20Scala%203.md) | Comparativa de codigo, `given`/`using`, `enum`, `extension` | Markdown |

---

## Modulo 1: The Scala Way (Fundamentos Hibridos)

Fundamentos del lenguaje, modelado de datos inmutables y programacion funcional basica.

| # | Capitulo | Descripcion | Markdown | PDF |
|---|----------|-------------|----------|-----|
| - | [Resumen Visual](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/00%20-%20Gráfico%20resumen%20Parte%201.md) | Diagrama Mermaid del modulo | Markdown | - |
| 1 | Configuracion y Tooling Esencial | Instalacion de JDK, REPL, configuracion de sbt y Scala CLI | [Capitulo 1](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%201%20-%20Configuración%20y%20Tooling%20Esencial.md) | [PDF](./assets/pdf/Capítulo%201%20-%20Configuración%20y%20Tooling%20Esencial.pdf) |
| 2 | El Paradigma Hibrido: POO Pura y PF | Todo es un objeto, funciones de primera clase, `val` vs `var`, inmutabilidad | [Capitulo 2](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%202%20-%20El%20Paradigma%20Híbrido%20-%20POO%20Pura%20y%20Programación%20Funcional%20(PF).md) | [PDF](./assets/pdf/Capítulo%202%20-%20El%20Paradigma%20Híbrido%20-%20POO%20Pura%20y%20Programación%20Funcional%20(PF).pdf) |
| 3 | Modelado de Datos: Clases, Objetos y Traits | `class`, `object`, `trait`, `case class`, Companion Objects | [Capitulo 3](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%203%20-%20Modelado%20de%20Datos%20-%20Clases,%20Objetos%20y%20Traits.md) | [PDF](./assets/pdf/Capítulo%203%20-%20Modelado%20de%20Datos%20-%20Clases,%20Objetos%20y%20Traits.pdf) |
| 4 | Funciones y Control de Flujo como Expresiones | Lambdas, HOFs (`map`, `filter`, `fold`), for-comprehensions | [Capitulo 4](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%204%20-%20Funciones%20y%20Control%20de%20Flujo%20como%20Expresiones.md) | [PDF](./assets/pdf/Capítulo%204%20-%20Funciones%20y%20Control%20de%20Flujo%20como%20Expresiones.pdf) |
| 5 | Estructuras de Datos y Colecciones Funcionales | `List`, `Set`, `Map`, operaciones funcionales inmutables | [Capitulo 5](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%205%20-%20Estructuras%20de%20Datos%20y%20Colecciones%20Funcionales.md) | [PDF](./assets/pdf/Capítulo%205%20-%20Estructuras%20de%20Datos%20y%20Colecciones%20Funcionales.pdf) |
| 6 | Pattern Matching y ADTs Basicos | `match`, `sealed trait`, `enum`, exhaustividad, desestructuracion | [Capitulo 6](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%206%20-%20Pattern%20Matching%20y%20Tipos%20de%20Datos%20Algebraicos%20(ADTs)%20Básicos.md) | [PDF](./assets/pdf/Capítulo%206%20-%20Pattern%20Matching%20y%20Tipos%20de%20Datos%20Algebraicos%20(ADTs)%20Básicos.pdf) |

---

## Modulo 2: Scala Funcional Moderna (Tipado y Abstracciones)

Sistema de tipos avanzado, abstracciones contextuales y metaprogramacion.

| # | Capitulo | Descripcion | Markdown | PDF |
|---|----------|-------------|----------|-----|
| - | [Resumen Visual](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/00%20-%20Gráfico%20resumen%20Parte%202.md) | Diagrama Mermaid del modulo | Markdown | - |
| 7 | Manejo de Ausencia y Errores Puros | `Option[T]`, `Either[E, R]`, `Try[T]`, composicion funcional | [Capitulo 7](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%207%20-%20Manejo%20de%20Ausencia%20y%20Errores%20Puros.md) | [PDF](./assets/pdf/Capítulo%207%20-%20Manejo%20de%20Ausencia%20y%20Errores%20Puros.pdf) |
| 8 | Abstracciones Contextuales (Scala 3) | `given`, `using`, Term Inference, derivacion de instancias | [Capitulo 8](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%208%20-%20Abstracciones%20Contextuales%20(Scala%203%20-%20%60given%60%20y%20%60using%60).md) | [PDF](./assets/pdf/Capítulo%208%20-%20Abstracciones%20Contextuales%20(Scala%203%20-%20%60given%60%20y%20%60using%60).pdf) |
| 9 | Type Classes y Metodos de Extension | Patron Type Class, `extension`, Context Bounds | [Capitulo 9](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%209%20-%20Type%20Classes%20y%20Métodos%20de%20Extensión.md) | [PDF](./assets/pdf/Capítulo%209%20-%20Type%20Classes%20y%20Métodos%20de%20Extensión.pdf) |
| 10 | Tipos Avanzados de Scala 3 | Union Types, Intersection Types, `opaque type` | [Capitulo 10](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%2010%20-%20Tipos%20Avanzados%20de%20Scala%203%20-%20Estructura%20y%20Flexibilidad.md) | [PDF](./assets/pdf/Capítulo%2010%20-%20Tipos%20Avanzados%20de%20Scala%203%20-%20Estructura%20y%20Flexibilidad.pdf) |
| 11 | Metaprogramacion y Optimizacion | `inline`, Macros, Quoting, Splicing | [Capitulo 11](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%2011%20-%20Metaprogramación%20y%20Optimización%20de%20Compilación.md) | [PDF](./assets/pdf/Capítulo%2011%20-%20Metaprogramación%20y%20Optimización%20de%20Compilación.pdf) |

---

## Modulo 3: El Ecosistema Industrial (Backend y Reactividad)

Concurrencia, sistemas de efectos, desarrollo web funcional y modelo de actores.

| # | Capitulo | Descripcion | Markdown | PDF |
|---|----------|-------------|----------|-----|
| - | [Resumen Visual](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/00%20-%20Gráfico%20resumen%20Parte%203.md) | Diagrama Mermaid del modulo | Markdown | - |
| 12 | Concurrencia: Futures y Monada IO | `Future`, `IO`, Fibers, `ExecutionContext` | [Capitulo 12](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2012%20-%20Concurrencia%20-%20Futures%20y%20la%20Mónada%20IO.md) | [PDF](./assets/pdf/Capítulo%2012%20-%20Concurrencia%20-%20Futures%20y%20la%20Mónada%20IO.pdf) |
| 13 | Arquitectura de Efectos: Cats Effect vs ZIO | Cats Effect, ZIO, `ZIO[R, E, A]`, `ZLayer` | [Capitulo 13](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2013%20-%20Arquitectura%20de%20Efectos%20-%20Cats%20Effect%20vs%20ZIO.md) | [PDF](./assets/pdf/Capítulo%2013%20-%20Arquitectura%20de%20Efectos%20-%20Cats%20Effect%20vs%20ZIO.pdf) |
| 14 | Desarrollo Web Funcional | http4s, Tapir, OpenAPI, endpoints type-safe | [Capitulo 14](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2014%20-%20Desarrollo%20Web%20Funcional.md) | [PDF](./assets/pdf/Capítulo%2014%20-%20Desarrollo%20Web%20Funcional.pdf) |
| 15 | Modelo de Actores: Pekko Fundamentos | `Behavior`, mensajes tipados, `ActorRef`, comunicacion asincrona | [Capitulo 15](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2015%20-%20Modelo%20de%20Actores%20-%20Pekko%20Fundamentos%20y%20Patrones.md) | [PDF](./assets/pdf/Capítulo%2015%20-%20Modelo%20de%20Actores%20-%20Pekko%20Fundamentos%20y%20Patrones.pdf) |
| 16 | Pekko: Tolerancia a Fallos y Estado | Supervision, Let-it-Crash, Event Sourcing | [Capitulo 16](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2016%20-%20Pekko%20-%20Tolerancia%20a%20Fallos%20y%20Estado%20Duradero.md) | [PDF](./assets/pdf/Capítulo%2016%20-%20Pekko%20-%20Tolerancia%20a%20Fallos%20y%20Estado%20Duradero.pdf) |
| 17 | Sistemas Distribuidos y Persistencia Reactiva | Cluster Sharding, Event Sourcing, Single-Writer Principle | [Capitulo 17](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2017%20-%20Sistemas%20Distribuidos%20y%20Persistencia%20Reactiva.md) | [PDF](./assets/pdf/Capítulo%2017%20-%20Sistemas%20Distribuidos%20y%20Persistencia%20Reactiva.pdf) |

---

## Modulo 4: Ingenieria de Datos y Calidad (Maestria Aplicada)

Big Data con Apache Spark, persistencia funcional, algoritmos avanzados y testing.

| # | Capitulo | Descripcion | Markdown | PDF |
|---|----------|-------------|----------|-----|
| - | [Resumen Visual](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/00%20-%20Gráfico%20resumen%20Parte%204.md) | Diagrama Mermaid del modulo | Markdown | - |
| 18 | Apache Spark: El Dominio de Big Data | `SparkSession`, RDD, DataFrame, `Dataset[T]` | [Capitulo 18](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2018%20-%20Apache%20Spark%20-%20El%20Dominio%20de%20Big%20Data.md) | [PDF](./assets/pdf/Capítulo%2018%20-%20Apache%20Spark%20-%20El%20Dominio%20de%20Big%20Data.pdf) |
| 19 | Programacion de Datos Estructurados con Spark | Joins, agregaciones, Catalyst optimizer | [Capitulo 19](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2019%20-%20Programación%20de%20Datos%20Estructurados%20con%20Spark.md) | [PDF](./assets/pdf/Capítulo%2019%20-%20Programación%20de%20Datos%20Estructurados%20con%20Spark.pdf) |
| 20 | Abstracciones de Datos: JDBC Funcional (Doobie) | `ConnectionIO`, `Transactor`, SQL seguro | [Capitulo 20](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2020%20-%20Abstracciones%20de%20Datos%20-%20JDBC%20Funcional%20(Doobie).md) | [PDF](./assets/pdf/Capítulo%2020%20-%20Abstracciones%20de%20Datos%20-%20JDBC%20Funcional%20(Doobie).pdf) |
| 21 | Diseno de Estructuras y Algoritmos Avanzados | Inmutabilidad, recursion, complejidad O(n) | [Capitulo 21](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2021%20-%20Diseño%20de%20Estructuras%20y%20Algoritmos%20Avanzados.md) | [PDF](./assets/pdf/Capítulo%2021%20-%20Diseño%20de%20Estructuras%20y%20Algoritmos%20Avanzados.pdf) |
| 22 | Testing de Calidad y Property-Based Testing | ScalaCheck, generadores, invariantes | [Capitulo 22](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2022%20-%20Testing%20de%20Calidad%20y%20Property-Based%20Testing.md) | [PDF](./assets/pdf/Capítulo%2022%20-%20Testing%20de%20Calidad%20y%20Property-Based%20Testing.pdf) |
| 23 | Tooling de Ingenieria y Arquitectura | sbt multi-project, Scalafix, modularidad | [Capitulo 23](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2023%20-%20Tooling%20de%20Ingeniería%20y%20Arquitectura%20(sbt%20y%20Modularidad).md) | [PDF](./assets/pdf/Capítulo%2023%20-%20Tooling%20de%20Ingeniería%20y%20Arquitectura%20(sbt%20y%20Modularidad).pdf) |

---

## Recursos Adicionales (PDF)

Documentos complementarios disponibles en formato PDF.

| Documento | Descripcion |
|-----------|-------------|
| [Manual Completo](./assets/pdf/ZZ%20-%20Manual%20Definitivo%20de%20Scala%203%20-%20De%20Cero%20a%20Ingeniería%20Reactiva.pdf) | Compilacion completa del manual en un solo documento |
| [Scala: Seguridad y Escala](./assets/pdf/ZZ%20-%20Scala_3_Seguridad_y_Escala.pdf) | Documento sobre seguridad de tipos y escalabilidad |
| [Scala y el Imperativo Arquitectonico](./assets/pdf/ZZ%20-%20Scala_y_el_Imperativo_Arquitectónico.pdf) | Perspectiva arquitectonica del lenguaje |

---

## Navegacion

- [Volver al README](./README.md)
- [Guia Rapida (Cheatsheet)](./CHEATSHEET%20-%20Guia%20rapida.md)
