````mermaid
graph TD

%% Definición de Estilos Profesionales
classDef concept fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#000;
classDef infra fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#000;
classDef code fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#000;
classDef data fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#000;

%% Nodo Raíz
M4["<b>PARTE 4: INGENIERÍA DE DATOS Y CALIDAD</b><br/><i>Maestría aplicada en Big Data, Persistencia y Validación</i>"]

%% Subgrafo: Apache Spark (Big Data)
subgraph Spark ["Apache Spark: Dominio de Big Data"]
    SparkS["<b>SparkSession</b><br/><i>Punto de entrada unificado para RDD, DF y DS</i>"]
    Catalyst["<b>Optimizador Catalyst</b><br/><i>Generación de planes de ejecución eficientes</i>"]
    DatasetT["<b>Dataset T</b><br/><i>Seguridad de tipos en compilación mediante case classes</i>"]
    UDFNative["<b>Native UDFs</b><br/><i>Ejecución directa en JVM sin penalización de serialización</i>"]
end

%% Subgrafo: Doobie (Persistencia Funcional)
subgraph Persistencia ["Doobie: Persistencia Pura y JDBC"]
    ConnIO["<b>ConnectionIO A</b><br/><i>Descripción inmutable de una acción en base de datos</i>"]
    Transactor["<b>Transactor F</b><br/><i>Motor que ejecuta el 'plano' en un efecto externo F</i>"]
    SafeSQL["<b>SQL Interpolators (fr)</b><br/><i>Prevención de inyección SQL y validación en compilación</i>"]
end

%% Subgrafo: Estructuras y Algoritmos
subgraph Algoritmos ["Diseño de Algoritmos Avanzados"]
    PersistentData["<b>Estructuras Persistentes</b><br/><i>Inmutabilidad rigurosa para seguridad concurrente</i>"]
    Complexity["<b>Análisis O(n)</b><br/><i>Garantía de eficiencia en operaciones fundamentales</i>"]
    TCO["<b>Recursión de Cola (TCO)</b><br/><i>Optimización para evitar 'Stack Overflow'</i>"]
    HOFsAlg["<b>HOFs (fold, reduce)</b><br/><i>Transformación declarativa de datos a gran escala</i>"]
end

%% Subgrafo: Calidad y ScalaCheck
subgraph Calidad ["Testing Basado en Propiedades (PBT)"]
    PBT["<b>Property-Based Testing</b><br/><i>Validación de invariantes lógicas sobre miles de casos</i>"]
    Generators["<b>Generadores (Gen)</b><br/><i>Creación automática de datos aleatorios y casos de borde</i>"]
    Invariantes["<b>Invariantes Lógicos</b><br/><i>Reglas universales que el código debe mantener siempre</i>"]
end

%% Subgrafo: Tooling de Ingeniería
subgraph Tooling ["Tooling y Modularidad Industrial"]
    SBTMulti["<b>sbt Multi-project</b><br/><i>Compilación paralela y gestión modular de dependencias</i>"]
    Scalafix["<b>Scalafix</b><br/><i>Linting avanzado y refactorización automatizada</i>"]
    Demeter["<b>Ley de Deméter (Desacoplamiento)</b><br/><i>Reducción de dependencias entre fronteras de módulos</i>"]
end

%% Conexiones y Relaciones Técnicas
M4 --> Spark
M4 --> Persistencia
M4 --> Algoritmos
M4 --> Calidad
M4 --> Tooling

SparkS -->|Gestiona| Catalyst
Catalyst -->|Optimiza| DatasetT
DatasetT -->|Usa| UDFNative

ConnIO -->|Delegado a| Transactor
SafeSQL -->|Produce| ConnIO

PersistentData -->|Analizado via| Complexity
Complexity -->|Optimizado con| TCO
HOFsAlg -->|Opera sobre| PersistentData

PBT -->|Requiere| Generators
Generators -->|Valida| Invariantes

SBTMulti -->|Maximiza| CompPar["'Compilación Incremental'"]
Scalafix -->|Impone| Standards["'Estándares de Código'"]
Demeter -->|Minimiza| SBTMulti

%% Asignación de Clases de Estilo
class M4,Catalyst,PBT,Invariantes,Demeter concept;
class SparkS,Transactor,SBTMulti,Scalafix infra;
class DatasetT,UDFNative,ConnIO,Generators,TCO,HOFsAlg code;
class SafeSQL,PersistentData,Complexity data;
````