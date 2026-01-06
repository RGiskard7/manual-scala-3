````mermaid
graph TD

%% Definición de Estilos Profesionales
classDef concept fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#000;
classDef infra fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#000;
classDef code fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#000;
classDef data fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#000;

%% Nodo Raíz
M6["<b>PARTE 6: INTEGRACIONES Y COMUNICACIÓN</b><br/><i>gRPC, Kafka, GraphQL y Compilación Nativa</i>"]

%% Subgrafo: gRPC y Protobuf
subgraph GRPC ["gRPC y Protobuf con ScalaPB"]
    Protobuf["<b>Protocol Buffers</b><br/><i>Definición de contratos binarios tipados</i>"]
    ScalaPB["<b>ScalaPB</b><br/><i>Generación de Case Classes desde .proto</i>"]
    FS2Grpc["<b>fs2-grpc</b><br/><i>Streaming bidireccional con backpressure</i>"]
    Interceptors["<b>Interceptors</b><br/><i>Middleware para autenticación y logging</i>"]
end

%% Subgrafo: Kafka Streaming
subgraph Kafka ["Kafka Streaming Funcional"]
    FS2Kafka["<b>fs2-kafka</b><br/><i>Consumidores y productores funcionales</i>"]
    Backpressure["<b>Modelo Pull</b><br/><i>Backpressure automático nativo</i>"]
    Commits["<b>commitBatchWithin</b><br/><i>Confirmación eficiente de offsets en lote</i>"]
    Avro["<b>Avro/Vulcan</b><br/><i>Serialización con evolución de esquemas</i>"]
end

%% Subgrafo: GraphQL
subgraph GraphQL ["GraphQL con Caliban"]
    Caliban["<b>Caliban</b><br/><i>Esquemas GraphQL derivados de tipos Scala</i>"]
    Queries["<b>Queries/Mutations</b><br/><i>Operaciones de lectura y escritura tipadas</i>"]
    Subscriptions["<b>Subscriptions</b><br/><i>Streams en tiempo real con ZStream</i>"]
    Introspection["<b>Introspection</b><br/><i>Documentación automática del esquema</i>"]
end

%% Subgrafo: Scala Native
subgraph Native ["Scala Native"]
    AOT["<b>Compilación AOT</b><br/><i>Binarios nativos sin dependencia de JVM</i>"]
    CInterop["<b>C-Interop</b><br/><i>Llamadas directas a funciones de sistema</i>"]
    ScalaCLI["<b>Scala CLI</b><br/><i>Empaquetado nativo simplificado</i>"]
    Toolkit["<b>Scala Toolkit</b><br/><i>Librerías portables JVM/JS/Native</i>"]
end

%% Conexiones y Relaciones Técnicas
M6 --> GRPC
M6 --> Kafka
M6 --> GraphQL
M6 --> Native

Protobuf -->|Genera| ScalaPB
ScalaPB -->|Integra con| FS2Grpc
FS2Grpc -->|Protegido por| Interceptors

FS2Kafka -->|Usa modelo| Backpressure
Commits -->|Optimiza| FS2Kafka
Avro -->|Serializa para| FS2Kafka

Caliban -->|Define| Queries
Caliban -->|Soporta| Subscriptions
Introspection -->|Documenta| Caliban

AOT -->|Produce binarios via| ScalaCLI
CInterop -->|Accede a| SysLibs["'Librerías del Sistema'"]
Toolkit -->|Portable entre| Plataformas["'JVM / JS / Native'"]

%% Asignación de Clases de Estilo
class M6,Backpressure,AOT concept;
class ScalaPB,FS2Kafka,Caliban,ScalaCLI infra;
class Protobuf,FS2Grpc,Commits,Queries,Subscriptions,CInterop code;
class Interceptors,Avro,Introspection,Toolkit data;
````

