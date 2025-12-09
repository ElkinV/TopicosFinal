# Guía de Instalación Detallada

Esta guía te ayudará a instalar y configurar todo lo necesario para ejecutar el proyecto Spring Apache Kafka.

## Requisitos del Sistema

### Java
- **Versión requerida**: Java 17 o superior
- **Verificar instalación**:
  ```bash
  java -version
  ```
- **Descargar**: [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) o [OpenJDK](https://openjdk.org/)

### Maven
- **Versión requerida**: Maven 3.6 o superior
- **Verificar instalación**:
  ```bash
  mvn -version
  ```
- **Descargar**: [Apache Maven](https://maven.apache.org/download.cgi)

### Apache Kafka
- **Versión recomendada**: 2.8.0 o superior
- **Descargar**: [Apache Kafka Downloads](https://kafka.apache.org/downloads)

## Instalación Paso a Paso

### 1. Instalar Java 17

#### Windows
1. Descarga el instalador desde el sitio oficial
2. Ejecuta el instalador y sigue las instrucciones
3. Configura la variable de entorno `JAVA_HOME`
4. Agrega `%JAVA_HOME%\bin` al `PATH`

#### Linux/Mac
```bash
# Usando SDKMAN
sdk install java 17.0.0-tem

# O usando Homebrew (Mac)
brew install openjdk@17
```

### 2. Instalar Maven

#### Windows
1. Descarga Maven desde el sitio oficial
2. Extrae el archivo ZIP
3. Configura la variable de entorno `MAVEN_HOME`
4. Agrega `%MAVEN_HOME%\bin` al `PATH`

#### Linux/Mac
```bash
# Usando Homebrew (Mac)
brew install maven

# O descarga y configura manualmente
export MAVEN_HOME=/ruta/a/maven
export PATH=$PATH:$MAVEN_HOME/bin
```

### 3. Instalar Apache Kafka

#### Windows
1. Descarga Kafka desde el sitio oficial
2. Extrae el archivo en una ubicación (ej: `C:\kafka`)
3. Navega a la carpeta de Kafka

#### Linux/Mac
```bash
# Descarga y extrae
wget https://downloads.apache.org/kafka/2.8.0/kafka_2.13-2.8.0.tgz
tar -xzf kafka_2.13-2.8.0.tgz
cd kafka_2.13-2.8.0
```

### 4. Configurar Kafka

#### Verificar la configuración de Zookeeper
El archivo `config/zookeeper.properties` debería tener una configuración similar a:

```properties
dataDir=/tmp/zookeeper
clientPort=2181
maxClientCnxns=0
```

#### Verificar la configuración de Kafka
El archivo `config/server.properties` debería tener:

```properties
broker.id=0
listeners=PLAINTEXT://localhost:9092
log.dirs=/tmp/kafka-logs
zookeeper.connect=localhost:2181
```

**Nota para Windows**: Puede que necesites cambiar las rutas de `log.dirs` y `dataDir` a rutas de Windows.

## Iniciar los Servicios

### 1. Iniciar Zookeeper

#### Windows
```bash
cd C:\kafka
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

#### Linux/Mac
```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

Deberías ver un mensaje indicando que Zookeeper está ejecutándose en el puerto 2181.

### 2. Iniciar Kafka

Abre una **nueva terminal** y ejecuta:

#### Windows
```bash
cd C:\kafka
bin\windows\kafka-server-start.bat config\server.properties
```

#### Linux/Mac
```bash
bin/kafka-server-start.sh config/server.properties
```

Deberías ver mensajes de inicio indicando que Kafka está ejecutándose en el puerto 9092.

### 3. Verificar que Kafka está funcionando

En una nueva terminal:

```bash
# Listar tópicos (debería estar vacío inicialmente)
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# O en Windows
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

## Compilar el Proyecto

Desde la raíz del proyecto:

```bash
mvn clean install
```

Esto compilará ambos módulos (Provider y Consumer).

## Ejecutar las Aplicaciones

### Opción 1: Desde la raíz del proyecto

```bash
# Terminal 1 - Consumer
mvn spring-boot:run -pl SpringKafkaConsumer

# Terminal 2 - Provider
mvn spring-boot:run -pl SpringKafkaProvider
```

### Opción 2: Desde cada módulo

```bash
# Terminal 1 - Consumer
cd SpringKafkaConsumer
mvn spring-boot:run

# Terminal 2 - Provider
cd SpringKafkaProvider
mvn spring-boot:run
```

### Opción 3: Usando JARs

```bash
# Compilar JARs
mvn clean package

# Ejecutar Consumer
java -jar SpringKafkaConsumer/target/SpringKafkaConsumer-0.0.1-SNAPSHOT.jar

# Ejecutar Provider
java -jar SpringKafkaProvider/target/SpringKafkaProvider-0.0.1-SNAPSHOT.jar
```

## Solución de Problemas

### Error: "Connection refused"
- Verifica que Kafka y Zookeeper estén ejecutándose
- Verifica que los puertos 2181 (Zookeeper) y 9092 (Kafka) estén disponibles

### Error: "Topic already exists"
- Esto es normal si el tópico ya fue creado previamente
- Puedes eliminarlo con:
  ```bash
  bin/kafka-topics.sh --delete --topic unProgramadorNace-topic --bootstrap-server localhost:9092
  ```

### Error: "Address already in use"
- Otro proceso está usando el puerto
- Cambia el puerto en `application.properties` o detén el proceso que lo está usando

### Windows: Problemas con rutas
- Asegúrate de usar rutas absolutas en `config/server.properties` y `config/zookeeper.properties`
- Ejemplo: `log.dirs=C:\\kafka-logs`

## Verificación Final

1. ✅ Zookeeper ejecutándose en puerto 2181
2. ✅ Kafka ejecutándose en puerto 9092
3. ✅ Consumer ejecutándose en puerto 8081
4. ✅ Provider ejecutándose en puerto 8080
5. ✅ Mensaje recibido en la consola del Consumer

Si todos estos pasos se completan correctamente, ¡tu instalación está lista!

