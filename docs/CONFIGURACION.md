# Guía de Configuración

Esta guía detalla todas las configuraciones disponibles en el proyecto y cómo personalizarlas.

## Configuración de Aplicación

### SpringKafkaProvider

Archivo: `SpringKafkaProvider/src/main/resources/application.properties`

```properties
# Nombre de la aplicación
spring.application.name=SpringKafkaProvider

# Puerto del servidor
server.port=8080

# Servidores de Kafka (bootstrap servers)
spring.kafka.bootstrapServers=localhost:9092
```

### SpringKafkaConsumer

Archivo: `SpringKafkaConsumer/src/main/resources/application.properties`

```properties
# Nombre de la aplicación
spring.application.name=SpringKafkaConsumer

# Puerto del servidor
server.port=8081

# Servidores de Kafka (bootstrap servers)
spring.kafka.bootstrapServers=localhost:9092
```

## Configuración de Kafka Producer

Archivo: `SpringKafkaProvider/src/main/java/com/kafka/provider/config/KafkaProviderConfig.java`

### Propiedades Configuradas

- **BOOTSTRAP_SERVERS_CONFIG**: Dirección del servidor Kafka (por defecto: `localhost:9092`)
- **KEY_SERIALIZER_CLASS_CONFIG**: Serializador para las claves (`StringSerializer`)
- **VALUE_SERIALIZER_CLASS_CONFIG**: Serializador para los valores (`StringSerializer`)

### Personalización

Puedes agregar más propiedades del productor:

```java
properties.put(ProducerConfig.ACKS_CONFIG, "all"); // Esperar confirmación de todas las réplicas
properties.put(ProducerConfig.RETRIES_CONFIG, 3); // Número de reintentos
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384); // Tamaño del batch
properties.put(ProducerConfig.LINGER_MS_CONFIG, 1); // Tiempo de espera antes de enviar
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // Memoria del buffer
```

## Configuración de Kafka Consumer

Archivo: `SpringKafkaConsumer/src/main/java/com/kafka/consumer/config/KafkaConsumerConfig.java`

### Propiedades Configuradas

- **BOOTSTRAP_SERVERS_CONFIG**: Dirección del servidor Kafka
- **KEY_DESERIALIZER_CLASS_CONFIG**: Deserializador para las claves (`StringDeserializer`)
- **VALUE_DESERIALIZER_CLASS_CONFIG**: Deserializador para los valores (`StringDeserializer`)

### Personalización

Puedes agregar más propiedades del consumidor:

```java
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group-id"); // ID del grupo
properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // earliest, latest, none
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true); // Auto-commit de offsets
properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000); // Intervalo de auto-commit
properties.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500); // Máximo de registros por poll
```

## Configuración del Tópico

Archivo: `SpringKafkaProvider/src/main/java/com/kafka/provider/config/KafkaTopicConfig.java`

### Configuración Actual

```java
NewTopic newTopic = TopicBuilder.name("unProgramadorNace-topic")
    .partitions(2)        // Número de particiones
    .replicas(2)          // Número de réplicas
    .configs(configurations)
    .build();
```

### Propiedades del Tópico

- **CLEANUP_POLICY_CONFIG**: 
  - `delete`: Borra mensajes antiguos (por defecto)
  - `compact`: Mantiene solo el mensaje más reciente por clave

- **RETENTION_MS_CONFIG**: 
  - Tiempo de retención de mensajes en milisegundos
  - Por defecto: `86400000` (24 horas)
  - `-1` significa retención ilimitada

- **SEGMENT_BYTES_CONFIG**: 
  - Tamaño máximo del segmento de log
  - Por defecto: `1073741824` bytes (1 GB)

- **MAX_MESSAGE_BYTES_CONFIG**: 
  - Tamaño máximo de cada mensaje
  - Por defecto: `1000012` bytes (~1 MB)

### Personalización del Tópico

```java
@Bean
public NewTopic generateTopic() {
    Map<String, String> configurations = new HashMap<>();
    
    // Política de limpieza
    configurations.put(TopicConfig.CLEANUP_POLICY_CONFIG, TopicConfig.CLEANUP_POLICY_COMPACT);
    
    // Retención de 7 días
    configurations.put(TopicConfig.RETENTION_MS_CONFIG, "604800000");
    
    // Compresión
    configurations.put(TopicConfig.COMPRESSION_TYPE_CONFIG, "snappy");
    
    return TopicBuilder.name("mi-topico-personalizado")
            .partitions(3)      // Más particiones para mayor paralelismo
            .replicas(3)        // Más réplicas para mayor disponibilidad
            .configs(configurations)
            .build();
}
```

## Configuración del Listener

Archivo: `SpringKafkaConsumer/src/main/java/com/kafka/consumer/listener/KafkaConsumerListener.java`

### Configuración Actual

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(String message) {
    LOGGER.info("Received message: " + message);
}
```

### Opciones de Personalización

```java
@KafkaListener(
    topics = {"unProgramadorNace-topic"},
    groupId = "my-group-id",
    containerFactory = "consumer",  // Factory personalizada
    autoStartup = "true",            // Iniciar automáticamente
    concurrency = "3"                // Número de threads concurrentes
)
public void listener(
    @Payload String message,                    // Cuerpo del mensaje
    @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,  // Tópico
    @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,  // Partición
    @Header(KafkaHeaders.OFFSET) long offset   // Offset
) {
    LOGGER.info("Received message: {} from topic: {}, partition: {}, offset: {}", 
                message, topic, partition, offset);
}
```

## Configuración de Múltiples Tópicos

Para escuchar múltiples tópicos:

```java
@KafkaListener(topics = {"topic1", "topic2", "topic3"}, groupId = "my-group-id")
public void listener(String message) {
    LOGGER.info("Received message: " + message);
}
```

## Configuración de Particiones Específicas

Para escuchar particiones específicas:

```java
@KafkaListener(
    topicPartitions = @TopicPartition(
        topic = "unProgramadorNace-topic",
        partitions = {"0", "1"}
    ),
    groupId = "my-group-id"
)
public void listener(String message) {
    LOGGER.info("Received message: " + message);
}
```

## Configuración de Perfiles

Puedes crear diferentes perfiles para desarrollo, producción, etc.

### application-dev.properties

```properties
spring.kafka.bootstrapServers=localhost:9092
server.port=8080
```

### application-prod.properties

```properties
spring.kafka.bootstrapServers=kafka-prod-1:9092,kafka-prod-2:9092,kafka-prod-3:9092
server.port=8080
```

Activar un perfil:

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

## Variables de Entorno

Puedes usar variables de entorno en `application.properties`:

```properties
spring.kafka.bootstrapServers=${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
server.port=${SERVER_PORT:8080}
```

Esto usa la variable de entorno `KAFKA_BOOTSTRAP_SERVERS` o `localhost:9092` como valor por defecto.

## Configuración de Logging

Agrega a `application.properties`:

```properties
# Nivel de logging
logging.level.com.kafka=DEBUG
logging.level.org.apache.kafka=INFO
logging.level.org.springframework.kafka=DEBUG

# Formato de logs
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
```

## Configuración de Seguridad (SASL/SSL)

Para configurar Kafka con seguridad:

```java
properties.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
properties.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
properties.put(SaslConfigs.SASL_JAAS_CONFIG, 
    "org.apache.kafka.common.security.plain.PlainLoginModule required " +
    "username=\"usuario\" password=\"contraseña\";");
```

## Mejores Prácticas

1. **Particiones**: Usa un número de particiones que sea múltiplo del número de consumidores
2. **Réplicas**: En producción, usa al menos 3 réplicas
3. **Retención**: Configura según tus necesidades de negocio
4. **Group ID**: Usa nombres descriptivos para los grupos de consumidores
5. **Offset Reset**: Usa `earliest` en desarrollo y `latest` en producción
6. **Acknowledgment**: En producción, usa `acks=all` para garantizar la durabilidad

