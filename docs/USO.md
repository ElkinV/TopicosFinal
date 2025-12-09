# Guía de Uso

Esta guía te ayudará a usar el proyecto Spring Apache Kafka de manera efectiva.

## Inicio Rápido

### 1. Verificar que Kafka esté ejecutándose

```bash
# Verificar que Kafka está activo
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### 2. Iniciar el Consumer

```bash
cd SpringKafkaConsumer
mvn spring-boot:run
```

Deberías ver en la consola:
```
Started SpringKafkaConsumerApplication
```

### 3. Iniciar el Provider

En una nueva terminal:

```bash
cd SpringKafkaProvider
mvn spring-boot:run
```

El Provider enviará automáticamente un mensaje al iniciar. Deberías ver en la consola del Consumer:

```
Received message: Topicos avanzados
```

## Enviar Mensajes Personalizados

### Modificar el Mensaje en el Provider

Edita `SpringKafkaProvider/src/main/java/com/kafka/provider/SpringKafkaProviderApplication.java`:

```java
@Bean
CommandLineRunner commandLineRunner(KafkaTemplate<String, String> kafkaTemplate){
    return args -> {
        kafkaTemplate.send("unProgramadorNace-topic", "Tu mensaje personalizado aquí");
    };
}
```

### Enviar Múltiples Mensajes

```java
@Bean
CommandLineRunner commandLineRunner(KafkaTemplate<String, String> kafkaTemplate){
    return args -> {
        for(int i = 0; i < 10; i++) {
            kafkaTemplate.send("unProgramadorNace-topic", "Mensaje número " + i);
        }
    };
}
```

### Enviar Mensajes con Clave

```java
kafkaTemplate.send("unProgramadorNace-topic", "clave-123", "Mensaje con clave");
```

Los mensajes con la misma clave irán a la misma partición, manteniendo el orden.

## Crear un REST Controller para Enviar Mensajes

Crea un controlador REST para enviar mensajes dinámicamente:

```java
package com.kafka.provider.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/kafka")
public class KafkaController {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @PostMapping("/send")
    public String sendMessage(@RequestBody String message) {
        kafkaTemplate.send("unProgramadorNace-topic", message);
        return "Mensaje enviado: " + message;
    }

    @PostMapping("/send/{key}")
    public String sendMessageWithKey(
            @PathVariable String key,
            @RequestBody String message) {
        kafkaTemplate.send("unProgramadorNace-topic", key, message);
        return "Mensaje enviado con clave " + key + ": " + message;
    }
}
```

Luego puedes enviar mensajes usando curl:

```bash
curl -X POST http://localhost:8080/api/kafka/send \
  -H "Content-Type: application/json" \
  -d "Hola desde REST"
```

## Procesar Diferentes Tipos de Mensajes

### Procesar Mensajes con Headers

Modifica el listener:

```java
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;

@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(
        @Payload String message,
        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
        @Header(KafkaHeaders.OFFSET) long offset) {
    LOGGER.info("Mensaje: {} | Tópico: {} | Partición: {} | Offset: {}", 
                message, topic, partition, offset);
}
```

### Procesar Objetos JSON

1. Crea una clase para el mensaje:

```java
public class Mensaje {
    private String contenido;
    private String autor;
    private LocalDateTime timestamp;
    
    // Getters y setters
}
```

2. Configura JSON serializer/deserializer en las configuraciones
3. Usa `KafkaTemplate<String, Mensaje>` y `@KafkaListener` con `Mensaje` como parámetro

## Monitoreo y Debugging

### Ver Mensajes en Kafka

Usa la consola de Kafka para ver mensajes:

```bash
bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic unProgramadorNace-topic \
  --from-beginning
```

### Ver Detalles del Tópico

```bash
bin/kafka-topics.sh \
  --describe \
  --topic unProgramadorNace-topic \
  --bootstrap-server localhost:9092
```

### Ver Grupos de Consumidores

```bash
bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list
```

### Ver Offset del Grupo

```bash
bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group-id \
  --describe
```

## Escenarios de Uso Comunes

### 1. Procesamiento en Lote

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(List<String> messages) {
    LOGGER.info("Recibidos {} mensajes", messages.size());
    for(String message : messages) {
        // Procesar cada mensaje
        processMessage(message);
    }
}
```

### 2. Procesamiento Asíncrono

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(String message) {
    CompletableFuture.runAsync(() -> {
        // Procesamiento pesado en otro thread
        heavyProcessing(message);
    });
}
```

### 3. Manejo de Errores

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(String message) {
    try {
        processMessage(message);
    } catch (Exception e) {
        LOGGER.error("Error procesando mensaje: {}", message, e);
        // Enviar a dead letter queue o reintentar
    }
}
```

### 4. Filtrado de Mensajes

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(String message) {
    if (message.startsWith("IMPORTANTE:")) {
        processImportantMessage(message);
    } else {
        processNormalMessage(message);
    }
}
```

## Mejores Prácticas

### 1. Idempotencia
Asegúrate de que el procesamiento de mensajes sea idempotente, ya que Kafka puede entregar el mismo mensaje múltiples veces.

### 2. Commit Manual
Para mayor control, considera commit manual de offsets:

```java
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(
        String message,
        Acknowledgment acknowledgment) {
    try {
        processMessage(message);
        acknowledgment.acknowledge(); // Commit manual
    } catch (Exception e) {
        // No hacer commit, el mensaje se reintentará
    }
}
```

### 3. Timeouts
Configura timeouts apropiados para evitar bloqueos:

```java
properties.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 30000);
properties.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 300000);
```

### 4. Logging
Usa logging apropiado para debugging:

```java
LOGGER.debug("Procesando mensaje: {}", message);
LOGGER.info("Mensaje procesado exitosamente");
LOGGER.error("Error procesando mensaje", exception);
```

## Solución de Problemas Comunes

### El Consumer no recibe mensajes

1. Verifica que el Consumer esté ejecutándose
2. Verifica que el grupo ID sea correcto
3. Verifica que el tópico exista
4. Revisa los logs del Consumer

### Mensajes duplicados

- Esto puede ocurrir si el Consumer falla después de procesar pero antes de hacer commit
- Implementa idempotencia en el procesamiento

### El Consumer está lento

1. Aumenta el número de particiones
2. Aumenta el número de consumidores en el grupo
3. Ajusta `MAX_POLL_RECORDS_CONFIG`
4. Optimiza el procesamiento de mensajes

### Error de conexión

1. Verifica que Kafka esté ejecutándose
2. Verifica la configuración de `bootstrapServers`
3. Verifica que los puertos estén abiertos
4. Revisa los logs de Kafka

## Ejemplos Avanzados

### Enviar Mensajes con Callback

```java
kafkaTemplate.send("unProgramadorNace-topic", message)
    .addCallback(
        result -> LOGGER.info("Mensaje enviado: {}", result),
        failure -> LOGGER.error("Error enviando mensaje", failure)
    );
```

### Procesar Mensajes con Retry

```java
@RetryableTopic(
    attempts = "3",
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
@KafkaListener(topics = {"unProgramadorNace-topic"}, groupId = "my-group-id")
public void listener(String message) {
    processMessage(message);
}
```

## Recursos Adicionales

- [Spring Kafka Documentation](https://docs.spring.io/spring-kafka/docs/current/reference/html/)
- [Kafka CLI Tools](https://kafka.apache.org/documentation/#basic_ops)
- [Kafka Best Practices](https://kafka.apache.org/documentation/#best_practices)

