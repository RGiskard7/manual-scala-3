# Manual Definitivo de Scala 3

## Indice General

Este documento proporciona una vista secuencial de todo el contenido del manual, con enlaces directos a cada capitulo.

---

## Recursos Generales

| Recurso | Descripcion |
|---------|-------------|
| [Guia Rapida (Cheatsheet)](./CHEATSHEET%20-%20Guia%20rapida.md) | Referencia condensada de sintaxis y patrones |
| [Infografia del Ecosistema Scala](./Infografía_scala.png) | Diagrama visual del ecosistema |

---

## Introduccion

Fundamentos conceptuales y guia de transicion para desarrolladores Java/Python.

| Documento | Descripcion |
|-----------|-------------|
| [Desmitificando la Programacion Funcional](./00%20-%20Introduccion/00%20-%20Desmitificando%20la%20Programación%20Funcional%20en%20Scala%20-%20Una%20Guía%20para%20Principiantes.md) | Los 4 pilares de la PF: inmutabilidad, expresiones, funciones como valores, errores como valores |
| [Los Pilares Conceptuales de Scala](./00%20-%20Introduccion/00%20-%20Los%20Pilares%20Conceptuales%20de%20Scala%20-%20El%20Paradigma%20Híbrido.md) | Ontologia unificada POO + PF, jerarquia de tipos, sintaxis clave |
| [Guia de Transicion: Java/Python a Scala 3](./00%20-%20Introduccion/02%20-%20Guía%20de%20Transición%20v2%20-%20De%20Java%20y%20Python%20a%20Scala%203.md) | Comparativa de codigo, `given`/`using`, `enum`, `extension` |

---

## Modulo 1: The Scala Way (Fundamentos Hibridos)

Fundamentos del lenguaje, modelado de datos inmutables y programacion funcional basica.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/00%20-%20Gráfico%20resumen%20Parte%201.md) | Diagrama Mermaid del modulo |
| 1 | [Configuracion y Tooling Esencial](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%201%20-%20Configuración%20y%20Tooling%20Esencial.md) | Instalacion de JDK, REPL, configuracion de sbt y Scala CLI |
| 2 | [El Paradigma Hibrido: POO Pura y PF](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%202%20-%20El%20Paradigma%20Híbrido%20-%20POO%20Pura%20y%20Programación%20Funcional%20(PF).md) | Todo es un objeto, funciones de primera clase, `val` vs `var`, inmutabilidad |
| 3 | [Modelado de Datos: Clases, Objetos y Traits](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%203%20-%20Modelado%20de%20Datos%20-%20Clases,%20Objetos%20y%20Traits.md) | `class`, `object`, `trait`, `case class`, Companion Objects |
| 4 | [Funciones y Control de Flujo como Expresiones](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%204%20-%20Funciones%20y%20Control%20de%20Flujo%20como%20Expresiones.md) | Lambdas, HOFs (`map`, `filter`, `fold`), for-comprehensions |
| 5 | [Estructuras de Datos y Colecciones Funcionales](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%205%20-%20Estructuras%20de%20Datos%20y%20Colecciones%20Funcionales.md) | `List`, `Set`, `Map`, operaciones funcionales inmutables |
| 6 | [Pattern Matching y ADTs Basicos](./01%20-%20Modulo%201%20-%20The%20Scala%20Way%20(Fundamentos)/Capítulo%206%20-%20Pattern%20Matching%20y%20Tipos%20de%20Datos%20Algebraicos%20(ADTs)%20Básicos.md) | `match`, `sealed trait`, `enum`, exhaustividad, desestructuracion |

---

## Modulo 2: Scala Funcional Moderna (Tipado y Abstracciones)

Sistema de tipos avanzado, abstracciones contextuales y metaprogramacion.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/00%20-%20Gráfico%20resumen%20Parte%202.md) | Diagrama Mermaid del modulo |
| 7 | [Manejo de Ausencia y Errores Puros](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%207%20-%20Manejo%20de%20Ausencia%20y%20Errores%20Puros.md) | `Option[T]`, `Either[E, R]`, `Try[T]`, composicion funcional |
| 8 | [Abstracciones Contextuales (Scala 3)](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%208%20-%20Abstracciones%20Contextuales%20(Scala%203%20-%20%60given%60%20y%20%60using%60).md) | `given`, `using`, Term Inference, derivacion de instancias |
| 9 | [Type Classes y Metodos de Extension](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%209%20-%20Type%20Classes%20y%20Métodos%20de%20Extensión.md) | Patron Type Class, `extension`, Context Bounds |
| 10 | [Tipos Avanzados de Scala 3](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%2010%20-%20Tipos%20Avanzados%20de%20Scala%203%20-%20Estructura%20y%20Flexibilidad.md) | Union Types, Intersection Types, `opaque type` |
| 11 | [Metaprogramacion y Optimizacion](./02%20-%20Modulo%202%20-%20Scala%20Funcional%20Moderna/Capítulo%2011%20-%20Metaprogramación%20y%20Optimización%20de%20Compilación.md) | `inline`, Macros, Quoting, Splicing |

---

## Modulo 3: El Ecosistema Industrial (Backend y Reactividad)

Concurrencia, sistemas de efectos, desarrollo web funcional y modelo de actores.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/00%20-%20Gráfico%20resumen%20Parte%203.md) | Diagrama Mermaid del modulo |
| 12 | [Concurrencia: Futures y Monada IO](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2012%20-%20Concurrencia%20-%20Futures%20y%20la%20Mónada%20IO.md) | `Future`, `IO`, Fibers, `ExecutionContext` |
| 13 | [Arquitectura de Efectos: Cats Effect vs ZIO](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2013%20-%20Arquitectura%20de%20Efectos%20-%20Cats%20Effect%20vs%20ZIO.md) | Cats Effect, ZIO, `ZIO[R, E, A]`, `ZLayer` |
| 14 | [Desarrollo Web Funcional](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2014%20-%20Desarrollo%20Web%20Funcional.md) | http4s, Tapir, OpenAPI, endpoints type-safe |
| 15 | [Modelo de Actores: Pekko Fundamentos](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2015%20-%20Modelo%20de%20Actores%20-%20Pekko%20Fundamentos%20y%20Patrones.md) | `Behavior`, mensajes tipados, `ActorRef`, comunicacion asincrona |
| 16 | [Pekko: Tolerancia a Fallos y Estado](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2016%20-%20Pekko%20-%20Tolerancia%20a%20Fallos%20y%20Estado%20Duradero.md) | Supervision, Let-it-Crash, Event Sourcing |
| 17 | [Sistemas Distribuidos y Persistencia Reactiva](./03%20-%20Modulo%203%20-%20El%20Ecosistema%20Industrial%20(Backend%20y%20Reactividad)/Capítulo%2017%20-%20Sistemas%20Distribuidos%20y%20Persistencia%20Reactiva.md) | Cluster Sharding, Event Sourcing, Single-Writer Principle |

---

## Modulo 4: Ingenieria de Datos y Calidad (Maestria Aplicada)

Big Data con Apache Spark, persistencia funcional, algoritmos avanzados y testing.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/00%20-%20Gráfico%20resumen%20Parte%204.md) | Diagrama Mermaid del modulo |
| 18 | [Apache Spark: El Dominio de Big Data](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2018%20-%20Apache%20Spark%20-%20El%20Dominio%20de%20Big%20Data.md) | `SparkSession`, RDD, DataFrame, `Dataset[T]` |
| 19 | [Programacion de Datos Estructurados con Spark](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2019%20-%20Programación%20de%20Datos%20Estructurados%20con%20Spark.md) | Joins, agregaciones, Catalyst optimizer |
| 20 | [Abstracciones de Datos: JDBC Funcional (Doobie)](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2020%20-%20Abstracciones%20de%20Datos%20-%20JDBC%20Funcional%20(Doobie).md) | `ConnectionIO`, `Transactor`, SQL seguro |
| 21 | [Diseno de Estructuras y Algoritmos Avanzados](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2021%20-%20Diseño%20de%20Estructuras%20y%20Algoritmos%20Avanzados.md) | Inmutabilidad, recursion, complejidad O(n) |
| 22 | [Testing de Calidad y Property-Based Testing](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2022%20-%20Testing%20de%20Calidad%20y%20Property-Based%20Testing.md) | ScalaCheck, generadores, invariantes |
| 23 | [Tooling de Ingenieria y Arquitectura](./04%20-%20Modulo%204%20-%20Ingeniería%20de%20Datos%20y%20Calidad/Capítulo%2023%20-%20Tooling%20de%20Ingeniería%20y%20Arquitectura%20(sbt%20y%20Modularidad).md) | sbt multi-project, Scalafix, modularidad |

---

## Modulo 5: Temas Avanzados y Especializados

Testing tradicional, serializacion JSON, desarrollo frontend con Scala.js, streaming funcional y optimizacion.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/00%20-%20Gráfico%20resumen%20Parte%205.md) | Diagrama Mermaid del modulo |
| 24 | [Testing Tradicional: ScalaTest y MUnit](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2024%20-%20Testing%20Tradicional%20-%20ScalaTest%20y%20MUnit.md) | Frameworks de testing, fixtures, aserciones, testing asincrono |
| 25 | [Serializacion JSON: Circe y uPickle](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2025%20-%20Serialización%20JSON%20-%20Circe%20y%20uPickle.md) | Derivacion automatica, codecs, rendimiento, integracion con APIs |
| 26 | [Scala.js: Frontend Funcional](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2026%20-%20Scala.js%20-%20Programación%20Frontend%20Funcional.md) | Compilacion a JS, Laminar, modelos compartidos, ScalablyTyped |
| 27 | [FS2: Streaming Funcional](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2027%20-%20%20FS2%20-%20Streaming%20Funcional%20con%20Cats%20Effect.md) | Stream[F, O], backpressure, bracket, concurrencia con parEvalMap |
| 28 | [Migracion de Scala 2 a Scala 3](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2028%20-%20Migración%20de%20Scala%202%20a%20Scala%203.md) | TASTy, given/using, scala-migrate, cross-building |
| 29 | [Optimizaciones Avanzadas y Rendimiento](./05%20-%20Modulo%205%20-%20Temas%20Avanzados%20y%20Especializados/Capítulo%2029%20-%20Optimizaciones%20Avanzadas%20y%20Rendimiento.md) | inline, opaque types, JMH, flame graphs, async-profiler |

---

## Modulo 6: Integraciones y Comunicacion

gRPC, Kafka, GraphQL y compilacion nativa para arquitecturas de microservicios.

| # | Capitulo | Descripcion |
|---|----------|-------------|
| - | [Resumen Visual](./06%20-%20Modulo%206%20-%20Integraciones%20y%20Comunicación/00%20-%20Gráfico%20resumen%20Parte%206.md) | Diagrama Mermaid del modulo |
| 30 | [gRPC y Protobuf con ScalaPB](./06%20-%20Modulo%206%20-%20Integraciones%20y%20Comunicación/Capítulo%2030%20-%20gRPC%20y%20Protobuf%20con%20ScalaPB.md) | Protocol Buffers, fs2-grpc, streaming bidireccional, interceptors |
| 31 | [Kafka Streaming Funcional](./06%20-%20Modulo%206%20-%20Integraciones%20y%20Comunicación/Capítulo%2031%20-%20Kafka%20Streaming%20Funcional%20(fs2-kafka).md) | fs2-kafka, backpressure, commits en lote, Avro/Vulcan |
| 32 | [GraphQL con Caliban](./06%20-%20Modulo%206%20-%20Integraciones%20y%20Comunicación/Capítulo%2032%20-%20GraphQL%20con%20Caliban.md) | Esquemas type-safe, queries, mutations, subscriptions |
| 33 | [Scala Native](./06%20-%20Modulo%206%20-%20Integraciones%20y%20Comunicación/Capítulo%2033%20-%20Scala%20Native.md) | Compilacion AOT, C-Interop, Scala CLI, binarios nativos |

---

## Navegacion

- [Volver al README](./README.md)
- [Guia Rapida (Cheatsheet)](./CHEATSHEET%20-%20Guia%20rapida.md)
