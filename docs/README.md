# Documentación del Proyecto Spring Apache Kafka

Bienvenido a la documentación completa del proyecto Spring Apache Kafka. Esta documentación te guiará a través de todos los aspectos del proyecto.

## 📚 Índice de Documentación

### 1. [README Principal](../README.md)
Documentación general del proyecto, incluyendo descripción, arquitectura básica, requisitos y uso rápido.

### 2. [Guía de Instalación](INSTALACION.md)
Instrucciones detalladas paso a paso para instalar y configurar:
- Java 17
- Maven
- Apache Kafka
- Configuración del entorno
- Solución de problemas comunes de instalación

### 3. [Guía de Configuración](CONFIGURACION.md)
Documentación completa sobre todas las configuraciones disponibles:
- Configuración de aplicaciones
- Configuración de Kafka Producer
- Configuración de Kafka Consumer
- Configuración de tópicos
- Variables de entorno
- Perfiles de Spring
- Configuración de seguridad

### 4. [Arquitectura del Proyecto](ARQUITECTURA.md)
Descripción detallada de la arquitectura:
- Diagramas de arquitectura
- Componentes principales
- Flujo de datos
- Patrones de diseño utilizados
- Escalabilidad
- Consideraciones de diseño

### 5. [Guía de Uso](USO.md)
Guía práctica para usar el proyecto:
- Inicio rápido
- Envío de mensajes
- Procesamiento de mensajes
- Monitoreo y debugging
- Escenarios de uso comunes
- Mejores prácticas
- Solución de problemas

## 🚀 Inicio Rápido

Si eres nuevo en el proyecto, te recomendamos seguir este orden:

1. **Lee el [README Principal](../README.md)** para entender el proyecto
2. **Sigue la [Guía de Instalación](INSTALACION.md)** para configurar tu entorno
3. **Revisa la [Guía de Uso](USO.md)** para comenzar a usar el proyecto
4. **Consulta la [Guía de Configuración](CONFIGURACION.md)** cuando necesites personalizar
5. **Explora la [Arquitectura](ARQUITECTURA.md)** para entender el diseño

## 📖 Estructura de la Documentación

```
docs/
├── README.md              # Este archivo (índice)
├── INSTALACION.md         # Guía de instalación detallada
├── CONFIGURACION.md       # Todas las configuraciones disponibles
├── ARQUITECTURA.md        # Arquitectura y diseño del proyecto
└── USO.md                 # Guía práctica de uso
```

## 🎯 Casos de Uso

### Quiero instalar el proyecto
→ Ve a [Guía de Instalación](INSTALACION.md)

### Quiero entender cómo funciona
→ Ve a [Arquitectura del Proyecto](ARQUITECTURA.md)

### Quiero personalizar la configuración
→ Ve a [Guía de Configuración](CONFIGURACION.md)

### Quiero empezar a usar el proyecto
→ Ve a [Guía de Uso](USO.md)

### Tengo un problema
→ Revisa la sección de "Solución de Problemas" en:
  - [Guía de Instalación](INSTALACION.md#solución-de-problemas)
  - [Guía de Uso](USO.md#solución-de-problemas-comunes)

## 🔍 Búsqueda Rápida

### Configuración de Kafka
- Producer: [CONFIGURACION.md#configuración-de-kafka-producer](CONFIGURACION.md#configuración-de-kafka-producer)
- Consumer: [CONFIGURACION.md#configuración-de-kafka-consumer](CONFIGURACION.md#configuración-de-kafka-consumer)
- Tópicos: [CONFIGURACION.md#configuración-del-tópico](CONFIGURACION.md#configuración-del-tópico)

### Ejemplos de Código
- Enviar mensajes: [USO.md#enviar-mensajes-personalizados](USO.md#enviar-mensajes-personalizados)
- Procesar mensajes: [USO.md#procesar-diferentes-tipos-de-mensajes](USO.md#procesar-diferentes-tipos-de-mensajes)
- REST API: [USO.md#crear-un-rest-controller-para-enviar-mensajes](USO.md#crear-un-rest-controller-para-enviar-mensajes)

### Arquitectura
- Componentes: [ARQUITECTURA.md#componentes-principales](ARQUITECTURA.md#componentes-principales)
- Flujo de datos: [ARQUITECTURA.md#flujo-completo-de-mensajes](ARQUITECTURA.md#flujo-completo-de-mensajes)
- Escalabilidad: [ARQUITECTURA.md#escalabilidad](ARQUITECTURA.md#escalabilidad)

## 💡 Consejos

1. **Primera vez**: Sigue el orden sugerido arriba
2. **Configuración**: Guarda tus configuraciones personalizadas
3. **Logs**: Revisa los logs cuando tengas problemas
4. **Kafka CLI**: Aprende a usar las herramientas CLI de Kafka para debugging
5. **Testing**: Prueba con mensajes simples antes de implementar lógica compleja

## 📝 Contribuir a la Documentación

Si encuentras errores o quieres mejorar la documentación:

1. Identifica el archivo que necesita cambios
2. Sugiere mejoras o correcciones
3. Mantén el formato y estilo consistente
4. Incluye ejemplos cuando sea posible

## 🔗 Enlaces Útiles

- [Documentación oficial de Spring Kafka](https://docs.spring.io/spring-kafka/docs/current/reference/html/)
- [Documentación oficial de Apache Kafka](https://kafka.apache.org/documentation/)
- [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)

## ❓ Preguntas Frecuentes

### ¿Necesito Kafka instalado localmente?
Sí, necesitas tener Kafka y Zookeeper ejecutándose. Ver [Guía de Instalación](INSTALACION.md).

### ¿Puedo usar Docker en lugar de instalar Kafka?
Sí, puedes usar Docker Compose. La configuración sería similar, solo cambia `bootstrapServers` a la dirección del contenedor.

### ¿Cómo cambio el nombre del tópico?
Modifica `KafkaTopicConfig.java` y el listener en `KafkaConsumerListener.java`. Ver [Guía de Configuración](CONFIGURACION.md).

### ¿Puedo tener múltiples consumidores?
Sí, puedes tener múltiples instancias del Consumer con el mismo `groupId`. Ver [Arquitectura](ARQUITECTURA.md#escalabilidad).

### ¿Los mensajes se pierden si el Consumer está apagado?
No, los mensajes se mantienen en Kafka según la política de retención configurada. El Consumer los procesará cuando se reinicie.

---

**Última actualización**: Esta documentación corresponde a la versión 1.0-SNAPSHOT del proyecto.

