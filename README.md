# Spring Apache Kafka - Proyecto Final

Este proyecto es una aplicación de ejemplo que demuestra cómo integrar Apache Kafka con Spring Boot. El proyecto está estructurado como un proyecto Maven multi-módulo que incluye un **Productor (Provider)** y un **Consumidor (Consumer)** de Kafka.

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Uso](#uso)
- [Configuración de Kafka](#configuración-de-kafka)
- [Componentes Principales](#componentes-principales)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)

## 📖 Descripción General

Este proyecto implementa un sistema de mensajería asíncrona utilizando Apache Kafka. Consta de dos módulos independientes:

1. **SpringKafkaProvider**: Aplicación que actúa como productor de mensajes, enviando mensajes a un tópico de Kafka.
2. **SpringKafkaConsumer**: Aplicación que actúa como consumidor de mensajes, recibiendo y procesando mensajes del tópico.

## 🏗️ Arquitectura

```
┌─────────────────────┐         ┌──────────┐         ┌─────────────────────┐
│                     │         │          │         │                     │
│ SpringKafkaProvider │ ──────> │  Kafka   │ ──────> │ SpringKafkaConsumer │
│   (Puerto 8080)     │         │ (9092)   │         │   (Puerto 8081)     │
│                     │         │          │         │                     │
└─────────────────────┘         └──────────┘         └─────────────────────┘
```

## 🔧 Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

- **Java 17** o superior
- **Maven 3.6+**
- **Apache Kafka** (versión 2.8.0 o superior)
- **Zookeeper** (incluido con Kafka)

## 📦 Instalación

### 1. Clonar el Repositorio

```bash
git clone <url-del-repositorio>
cd spring-apache-kafka
```

### 2. Instalar Apache Kafka

Si aún no tienes Kafka instalado, puedes descargarlo desde [Apache Kafka Downloads](https://kafka.apache.org/downloads).

### 3. Iniciar Kafka y Zookeeper

```bash
# Iniciar Zookeeper (en una terminal)
bin/zookeeper-server-start.sh config/zookeeper.properties

# Iniciar Kafka (en otra terminal)
bin/kafka-server-start.sh config/server.properties
```

**Nota para Windows:**
```bash
# Zookeeper
bin\windows\zookeeper-server-start.bat config\zookeeper.properties

# Kafka
bin\windows\kafka-server-start.bat config\server.properties
```

### 4. Compilar el Proyecto

```bash
mvn clean install
```

## ⚙️ Configuración

### Configuración del Provider (application.properties)

El archivo `SpringKafkaProvider/src/main/resources/application.properties` contiene:

```properties
spring.application.name=SpringKafkaProvider
server.port=8080
spring.kafka.bootstrapServers=localhost:9092
```

### Configuración del Consumer (application.properties)

El archivo `SpringKafkaConsumer/src/main/resources/application.properties` contiene:

```properties
spring.application.name=SpringKafkaConsumer
server.port=8081
spring.kafka.bootstrapServers=localhost:9092
```

## 📁 Estructura del Proyecto

```
spring-apache-kafka/
├── pom.xml                          # POM padre del proyecto
├── README.md                        # Este archivo
│
├── SpringKafkaProvider/             # Módulo Productor
│   ├── pom.xml
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/kafka/provider/
│           │       ├── SpringKafkaProviderApplication.java
│           │       └── config/
│           │           ├── KafkaProviderConfig.java
│           │           └── KafkaTopicConfig.java
│           └── resources/
│               └── application.properties
│
└── SpringKafkaConsumer/            # Módulo Consumidor
    ├── pom.xml
    └── src/
        └── main/
            ├── java/
            │   └── com/kafka/consumer/
            │       ├── SpringKafkaConsumerApplication.java
            │       ├── config/
            │       │   └── KafkaConsumerConfig.java
            │       └── listener/
            │           └── KafkaConsumerListener.java
            └── resources/
                └── application.properties
```

## 🚀 Uso

### 1. Iniciar el Consumidor

En una terminal, ejecuta:

```bash
cd SpringKafkaConsumer
mvn spring-boot:run
```

O desde la raíz del proyecto:

```bash
mvn spring-boot:run -pl SpringKafkaConsumer
```

El consumidor estará escuchando en el puerto **8081** y esperando mensajes del tópico `unProgramadorNace-topic`.

### 2. Iniciar el Productor

En otra terminal, ejecuta:

```bash
cd SpringKafkaProvider
mvn spring-boot:run
```

O desde la raíz del proyecto:

```bash
mvn spring-boot:run -pl SpringKafkaProvider
```

El productor se iniciará en el puerto **8080** y automáticamente enviará un mensaje al tópico al iniciar.

### 3. Verificar los Mensajes

Deberías ver en la consola del consumidor un mensaje como:

```
Received message: Topicos avanzados
```

## 🔍 Configuración de Kafka

### Configuración del Tópico

El tópico se crea automáticamente con la siguiente configuración (definida en `KafkaTopicConfig.java`):

- **Nombre del tópico**: `unProgramadorNace-topic`
- **Particiones**: 2
- **Réplicas**: 2
- **Política de limpieza**: DELETE (borra mensajes antiguos)
- **Tiempo de retención**: 86400000 ms (24 horas)
- **Tamaño máximo del segmento**: 1073741824 bytes (1 GB)
- **Tamaño máximo de mensaje**: 1000012 bytes (~1 MB)

### Verificar el Tópico Manualmente

Puedes verificar que el tópico se haya creado correctamente usando la CLI de Kafka:

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

Para ver detalles del tópico:

```bash
bin/kafka-topics.sh --describe --topic unProgramadorNace-topic --bootstrap-server localhost:9092
```

## 🧩 Componentes Principales

### SpringKafkaProvider

#### `KafkaProviderConfig.java`
Configura el productor de Kafka:
- Define las propiedades del productor
- Configura los serializadores (StringSerializer)
- Crea el `KafkaTemplate` para enviar mensajes

#### `KafkaTopicConfig.java`
Configura el tópico de Kafka:
- Define el nombre, particiones y réplicas
- Establece políticas de retención y limpieza
- Configura límites de tamaño de mensajes y segmentos

#### `SpringKafkaProviderApplication.java`
Aplicación principal que:
- Inicia la aplicación Spring Boot
- Envía un mensaje de prueba al tópico al iniciar

### SpringKafkaConsumer

#### `KafkaConsumerConfig.java`
Configura el consumidor de Kafka:
- Define las propiedades del consumidor
- Configura los deserializadores
- Crea el `KafkaListenerContainerFactory`

#### `KafkaConsumerListener.java`
Listener que procesa los mensajes:
- Escucha el tópico `unProgramadorNace-topic`
- Procesa mensajes del grupo `my-group-id`
- Registra los mensajes recibidos en los logs

## 🛠️ Tecnologías Utilizadas

- **Spring Boot 3.0.6**: Framework de aplicación
- **Spring Kafka**: Integración de Kafka con Spring
- **Apache Kafka**: Sistema de mensajería distribuida
- **Maven**: Gestor de dependencias y construcción
- **Java 17**: Lenguaje de programación
- **SLF4J**: Logging

## 📝 Dependencias Principales

Las dependencias principales se encuentran en el `pom.xml` padre:

- `spring-boot-starter-web`: Para aplicaciones web Spring Boot
- `spring-kafka`: Integración de Kafka con Spring
- `spring-boot-devtools`: Herramientas de desarrollo
- `spring-kafka-test`: Utilidades de testing para Kafka

## 🔄 Flujo de Mensajes

1. El **Provider** inicia y crea el tópico si no existe
2. El **Provider** envía un mensaje al tópico `unProgramadorNace-topic`
3. El **Consumer** está escuchando el tópico
4. Cuando llega un mensaje, el listener lo procesa y lo registra en los logs

## 📚 Recursos Adicionales

- [Documentación de Spring Kafka](https://docs.spring.io/spring-kafka/docs/current/reference/html/)
- [Documentación de Apache Kafka](https://kafka.apache.org/documentation/)
- [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)


## 📄 Licencia

Este proyecto es de código abierto y está disponible para uso educativo y de demostración.

## 👨‍💻 Autores : Elkin Vasquez, Gybram Llamas


**Nota**: Asegúrate de que Kafka y Zookeeper estén ejecutándose antes de iniciar las aplicaciones Spring Boot.
