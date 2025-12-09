# Arquitectura del Proyecto

Este documento describe la arquitectura, diseño y flujo de datos del proyecto Spring Apache Kafka.

## Visión General

El proyecto implementa un patrón de mensajería asíncrona utilizando Apache Kafka como broker de mensajes. Está compuesto por dos aplicaciones Spring Boot independientes que se comunican a través de Kafka.

## Diagrama de Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                        Apache Kafka Cluster                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Tópico: unProgramadorNace-topic        │  │
│  │  ┌──────────┐  ┌──────────┐                              │  │
│  │  │Partición 0│  │Partición 1│  (2 particiones)            │  │
│  │  └──────────┘  └──────────┘                              │  │
│  │       │              │                                     │  │
│  │       └──────┬───────┘                                     │  │
│  │              │                                             │  │
│  │         (2 réplicas cada una)                              │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
         ▲                                    │
         │                                    │
         │                                    ▼
┌─────────────────────┐            ┌─────────────────────┐
│                     │            │                     │
│ SpringKafkaProvider │            │ SpringKafkaConsumer │
│   (Productor)       │            │   (Consumidor)      │
│                     │            │                     │
│ Puerto: 8080        │            │ Puerto: 8081        │
└─────────────────────┘            └─────────────────────┘
```

## Componentes Principales

### 1. SpringKafkaProvider (Productor)

#### Responsabilidades
- Crear y configurar el tópico de Kafka
- Enviar mensajes al tópico
- Configurar el productor de Kafka

#### Componentes Internos

```
SpringKafkaProvider
│
├── SpringKafkaProviderApplication
│   └── CommandLineRunner: Envía mensaje al iniciar
│
├── config/
│   ├── KafkaProviderConfig
│   │   ├── ProducerFactory: Factory para crear productores
│   │   └── KafkaTemplate: Template para enviar mensajes
│   │
│   └── KafkaTopicConfig
│       └── NewTopic: Configuración del tópico
│
└── resources/
    └── application.properties: Configuración de la aplicación
```

#### Flujo de Datos (Provider)

```
1. Aplicación inicia
   │
2. KafkaTopicConfig crea el tópico (si no existe)
   │
3. KafkaProviderConfig configura el productor
   │
4. CommandLineRunner ejecuta
   │
5. KafkaTemplate envía mensaje
   │
6. Mensaje llega a Kafka
```

### 2. SpringKafkaConsumer (Consumidor)

#### Responsabilidades
- Escuchar mensajes del tópico
- Procesar mensajes recibidos
- Configurar el consumidor de Kafka

#### Componentes Internos

```
SpringKafkaConsumer
│
├── SpringKafkaConsumerApplication
│   └── Punto de entrada de la aplicación
│
├── config/
│   └── KafkaConsumerConfig
│       ├── ConsumerFactory: Factory para crear consumidores
│       └── KafkaListenerContainerFactory: Factory para listeners
│
├── listener/
│   └── KafkaConsumerListener
│       └── @KafkaListener: Método que procesa mensajes
│
└── resources/
    └── application.properties: Configuración de la aplicación
```

#### Flujo de Datos (Consumer)

```
1. Aplicación inicia
   │
2. KafkaConsumerConfig configura el consumidor
   │
3. KafkaConsumerListener se registra
   │
4. Escucha activa en el tópico
   │
5. Mensaje llega de Kafka
   │
6. Listener procesa el mensaje
   │
7. Log registra el mensaje
```

## Flujo Completo de Mensajes

```
┌─────────────────────────────────────────────────────────────┐
│                    Flujo de Mensajes                        │
└─────────────────────────────────────────────────────────────┘

1. Provider inicia
   │
   ├─> Crea tópico "unProgramadorNace-topic" (si no existe)
   │   ├─> 2 particiones
   │   └─> 2 réplicas
   │
   └─> Envía mensaje: "Topicos avanzados"
       │
       └─> Kafka recibe mensaje
           │
           ├─> Almacena en partición 0 o 1 (según hash de clave)
           │
           └─> Replica a los brokers configurados

2. Consumer inicia
   │
   ├─> Se conecta a Kafka
   │
   ├─> Se une al grupo "my-group-id"
   │
   └─> Comienza a escuchar el tópico
       │
       └─> Kafka entrega mensajes
           │
           └─> Listener procesa cada mensaje
               │
               └─> Log: "Received message: Topicos avanzados"
```

## Patrones de Diseño Utilizados

### 1. Factory Pattern
- `ProducerFactory` y `ConsumerFactory` crean instancias de productores y consumidores
- `KafkaListenerContainerFactory` crea contenedores para listeners

### 2. Template Pattern
- `KafkaTemplate` encapsula la lógica de envío de mensajes
- Proporciona métodos de alto nivel para interactuar con Kafka

### 3. Listener Pattern
- `@KafkaListener` implementa el patrón observer/listener
- Permite procesamiento asíncrono de mensajes

### 4. Configuration Pattern
- Clases `@Configuration` centralizan la configuración
- Facilita la gestión y modificación de configuraciones

## Configuración de Kafka

### Tópico: unProgramadorNace-topic

```
┌─────────────────────────────────────────┐
│  Tópico: unProgramadorNace-topic        │
├─────────────────────────────────────────┤
│  Particiones: 2                          │
│  Réplicas: 2                             │
│  Cleanup Policy: DELETE                  │
│  Retention: 24 horas                     │
│  Max Message Size: ~1 MB                 │
│  Max Segment Size: 1 GB                  │
└─────────────────────────────────────────┘
```

### Particiones y Paralelismo

```
Partición 0 ──┐
              ├──> Consumer Group: my-group-id
Partición 1 ──┘      (1 consumidor puede procesar ambas)
```

Con más consumidores en el mismo grupo:

```
Partición 0 ──> Consumer 1
Partición 1 ──> Consumer 2
```

## Escalabilidad

### Escalado Horizontal del Consumer

```
┌─────────────────────────────────────────┐
│  Consumer Group: my-group-id            │
├─────────────────────────────────────────┤
│  Consumer 1 ──> Partición 0             │
│  Consumer 2 ──> Partición 1             │
│  Consumer 3 ──> (idle, esperando)       │
└─────────────────────────────────────────┘
```

**Regla**: El número de consumidores activos no puede exceder el número de particiones.

### Escalado del Provider

Múltiples instancias del Provider pueden enviar mensajes al mismo tópico. Kafka distribuirá los mensajes entre las particiones.

## Consideraciones de Diseño

### 1. Desacoplamiento
- Provider y Consumer son completamente independientes
- No se conocen entre sí
- Se comunican solo a través de Kafka

### 2. Tolerancia a Fallos
- Si el Consumer falla, los mensajes permanecen en Kafka
- El Consumer puede recuperarse y procesar mensajes desde donde se quedó
- Las réplicas garantizan disponibilidad

### 3. Rendimiento
- Las particiones permiten procesamiento paralelo
- Los mensajes se distribuyen entre particiones
- Múltiples consumidores pueden procesar en paralelo

### 4. Orden de Mensajes
- Los mensajes dentro de una partición mantienen el orden
- Entre particiones, el orden no está garantizado
- Para mantener orden global, usa 1 partición (reduce paralelismo)

## Extensiones Posibles

### 1. Múltiples Tópicos
```
Provider ──> Tópico 1 ──> Consumer 1
          └─> Tópico 2 ──> Consumer 2
          └─> Tópico 3 ──> Consumer 3
```

### 2. Procesamiento de Errores
```
Consumer ──> Procesa mensaje
          │
          ├─> Éxito ──> Commit offset
          │
          └─> Error ──> Dead Letter Queue
```

### 3. Transformación de Mensajes
```
Provider ──> Tópico A ──> Consumer/Transformer ──> Tópico B ──> Consumer Final
```

## Diagrama de Secuencia

```
Provider                Kafka                Consumer
   │                     │                      │
   │─── Iniciar ────────>│                      │
   │                     │                      │
   │─── Crear Tópico ───>│                      │
   │<── Tópico Creado ───│                      │
   │                     │                      │
   │─── Enviar Msg ─────>│                      │
   │                     │                      │
   │                     │                      │─── Iniciar ──────>│
   │                     │                      │                   │
   │                     │                      │─── Suscribirse ──>│
   │                     │                      │                   │
   │                     │<───── Poll ───────────│                   │
   │                     │                      │                   │
   │                     │────── Msg ───────────>│                   │
   │                     │                      │                   │
   │                     │                      │─── Procesar ─────>│
   │                     │                      │                   │
   │                     │<───── Commit ────────│                   │
```

## Conclusión

Esta arquitectura proporciona:
- ✅ Desacoplamiento entre componentes
- ✅ Escalabilidad horizontal
- ✅ Tolerancia a fallos
- ✅ Procesamiento asíncrono
- ✅ Flexibilidad para extender

El diseño permite agregar fácilmente más productores, consumidores o tópicos según las necesidades del sistema.

