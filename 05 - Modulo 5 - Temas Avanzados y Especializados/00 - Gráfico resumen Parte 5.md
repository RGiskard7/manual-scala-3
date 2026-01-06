````mermaid
graph TD

%% Definición de Estilos Profesionales
classDef concept fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#000;
classDef infra fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#000;
classDef code fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#000;
classDef data fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#000;

%% Nodo Raíz
M5["<b>PARTE 5: TEMAS AVANZADOS Y ESPECIALIZADOS</b><br/><i>Testing, Serialización, Frontend, Streaming y Optimización</i>"]

%% Subgrafo: Testing Tradicional
subgraph Testing ["Testing Tradicional: ScalaTest y MUnit"]
    ScalaTest["<b>ScalaTest</b><br/><i>Framework completo con múltiples estilos DSL</i>"]
    MUnit["<b>MUnit</b><br/><i>Testing minimalista y rápido del Scala Toolkit</i>"]
    AsyncTest["<b>Testing Asíncrono</b><br/><i>Soporte nativo para IO, ZIO y Fibras</i>"]
    PekkoTest["<b>TestProbe (Pekko)</b><br/><i>Verificación de mensajes entre actores</i>"]
end

%% Subgrafo: Serialización JSON
subgraph JSON ["Serialización JSON: Circe y uPickle"]
    Circe["<b>Circe</b><br/><i>Encoder/Decoder basados en Cats y Type Classes</i>"]
    UPickle["<b>uPickle</b><br/><i>Alto rendimiento sin dependencias externas</i>"]
    Derives["<b>derives Codec</b><br/><i>Derivación automática via Mirror en compilación</i>"]
end

%% Subgrafo: Scala.js
subgraph Frontend ["Scala.js: Frontend Funcional"]
    ScalaJS["<b>Scala.js Compiler</b><br/><i>Compilación de Scala 3 a JavaScript optimizado</i>"]
    Laminar["<b>Laminar (FRP)</b><br/><i>UI reactiva sin Virtual DOM</i>"]
    Shared["<b>Módulo shared/</b><br/><i>Modelos compartidos entre Backend y Frontend</i>"]
    ScalablyTyped["<b>ScalablyTyped</b><br/><i>Facades automáticas desde TypeScript</i>"]
end

%% Subgrafo: FS2 Streaming
subgraph Streaming ["FS2: Streaming Funcional"]
    StreamF["<b>Stream F, O</b><br/><i>Descripción inmutable de flujos con efectos</i>"]
    Pull["<b>Modelo Pull</b><br/><i>Backpressure automático y nativo</i>"]
    Bracket["<b>Bracket/Resource</b><br/><i>Gestión segura de recursos en flujos</i>"]
    ParEval["<b>parEvalMap</b><br/><i>Concurrencia controlada sobre elementos</i>"]
end

%% Subgrafo: Migración
subgraph Migracion ["Migración: Scala 2 a Scala 3"]
    TASTy["<b>TASTy Format</b><br/><i>Interoperabilidad binaria bidireccional</i>"]
    GivenUsing["<b>implicit → given/using</b><br/><i>Rediseño de abstracciones contextuales</i>"]
    ScalaMigrate["<b>scala-migrate</b><br/><i>Herramienta automática de conversión</i>"]
    XSource3["<b>-Xsource:3</b><br/><i>Detección temprana en Scala 2.13</i>"]
end

%% Subgrafo: Optimización
subgraph Optimizacion ["Optimizaciones y Rendimiento"]
    InlineMod["<b>inline def</b><br/><i>Expansión en compilación sin overhead de llamada</i>"]
    OpaqueT["<b>opaque type</b><br/><i>Abstracciones de costo cero en runtime</i>"]
    JMH["<b>JMH Benchmarks</b><br/><i>Medición precisa evitando distorsiones del JIT</i>"]
    FlameGraph["<b>Flame Graphs</b><br/><i>Identificación visual de hot paths</i>"]
end

%% Conexiones y Relaciones Técnicas
M5 --> Testing
M5 --> JSON
M5 --> Frontend
M5 --> Streaming
M5 --> Migracion
M5 --> Optimizacion

ScalaTest -->|Complementado por| MUnit
MUnit -->|Soporta| AsyncTest
AsyncTest -->|Integra con| PekkoTest

Circe -->|Alternativa rápida| UPickle
Derives -->|Genera| Circe
Derives -->|Genera| UPickle

ScalaJS -->|Renderiza con| Laminar
Shared -->|Compilado por| ScalaJS
ScalablyTyped -->|Genera facades para| Laminar

StreamF -->|Usa modelo| Pull
Bracket -->|Garantiza limpieza en| StreamF
ParEval -->|Paraleliza| StreamF

TASTy -->|Habilita| CrossBuild["'Cross-compilation 2.13/3.x'"]
GivenUsing -->|Migrado via| ScalaMigrate
XSource3 -->|Prepara para| TASTy

InlineMod -->|Evita overhead en| HotPaths["'Hot Paths'"]
OpaqueT -->|Elimina allocations en| HotPaths
JMH -->|Mide| HotPaths
FlameGraph -->|Visualiza| HotPaths

%% Asignación de Clases de Estilo
class M5,AsyncTest,Pull,TASTy,GivenUsing concept;
class ScalaTest,MUnit,ScalaJS,JMH,ScalaMigrate infra;
class Circe,UPickle,Derives,Laminar,StreamF,InlineMod,OpaqueT code;
class Shared,Bracket,ParEval,XSource3,FlameGraph,PekkoTest,ScalablyTyped data;
````

