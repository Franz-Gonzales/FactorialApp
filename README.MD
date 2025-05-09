# FactorialApp - Práctica Jenkins

Este repositorio contiene una aplicación Java que calcula el factorial de un número, configurada con Maven y gestionada mediante un pipeline de Jenkins. La práctica utiliza Docker para ejecutar Jenkins con Java 21 y sigue las instrucciones del documento "Practica Jenkins.pdf".

## Requisitos Previos

- **Docker Desktop**: Para ejecutar el contenedor de Jenkins.
- **Git**: Para clonar y gestionar el repositorio.
- **Maven**: Para compilar, probar y empaquetar el proyecto.
- **Java 21 o superior**: Para desarrollo local (se usa Java 23 localmente, pero Jenkins usa Java 21).
- **IntelliJ IDEA** (opcional): Para desarrollo y pruebas locales.
- **Cuenta de GitHub**: Para alojar el repositorio.

## Configuración del Entorno

### 1. Configurar Jenkins con Docker
- **Descargar la imagen de Jenkins**:
  ```powershell
  docker pull jenkins/jenkins:lts-jdk21
  

# Crear el contenedor:
    powershell
    docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts-jdk21

# Acceder a Jenkins:
 ## Abre http://localhost:8080 en el navegador.
 ## Obtén la contraseña inicial:
 powershell

    - docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

- Pega la contraseña en la interfaz y selecciona Install suggested plugins.
- Crea un usuario administrador y completa la configuración.

# Instalar Maven en el contenedor:
- Accede como root:
  ```powershell
    powershell
        docker exec -it --user root jenkins bash


- Instala Maven:
    ```powershell
    apt update
    apt install -y maven

- Verifica:
    ```powershell
    mvn --version

- Sal del contenedor:
    ```powershell
    exit

# Instalar Git en el contenedor (si es necesario):
    docker exec -it --user root jenkins bash
    apt install -y git
    git --version
    exit


# 2. Configuración del Proyecto Java
   Dependencias (pom.xml)

   Archivo pom.xml configurado para Java 21:
   
```xml
    <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>FactorialApp</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.11.0</junit.version>
    </properties>
    <dependencies>
        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!-- Compilar y ejecutar pruebas -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.3.1</version>
            </plugin>
            <!-- Especificar Java 21 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <source>21</source>
                    <target>21</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

# Código Java
- Archivo src/main/java/com/example/Factorial.java

```java
package com.example;

public class Factorial {
    public static long calcular(int numero) {
        if (numero < 0) {
            throw new IllegalArgumentException("El número debe ser >= 0");
        }
        long resultado = 1;
        for (int i = 2; i <= numero; i++) {
            resultado *= i;
        }
        return resultado;
    }
}
```

- Archivo src/test/java/com/example/FactorialTest.java
```java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class FactorialTest {
    @Test
    void testFactorialDeCero() {
        assertEquals(1, Factorial.calcular(0));
    }

    @Test
    void testFactorialDeCinco() {
        assertEquals(120, Factorial.calcular(5));
    }

    @Test
    void testFactorialNegativo() {
        assertThrows(IllegalArgumentException.class, () -> Factorial.calcular(-1));
    }
}
```


## Comandos Locales
- Instalar Maven (si no está instalado):
```powershell
    winget install Apache.Maven
```

- Probar el proyecto localmente
```powershell
    cd FactorialApp
    mvn clean test
```

- Empaquetar localmente:
```powershell
    mvn package
```

# 3. Configuración del Repositorio GitHub
   - Crear el repositorio:
        En GitHub, crea un repositorio público llamado FactorialApp.
   - Clonar el repositorio:

    powershell
    git clone https://github.com/Franz-Gonzales/FactorialApp.git
    cd FactorialApp

    
Estructura del proyecto:

    mkdir -p src/main/java/com/example src/test/java/com/example

Subir cambios:
```powershell
    git add .
    git commit -m "Initial commit with FactorialApp for Java 21"
    git push origin main
```


# 4. Configuración del Pipeline en Jenkins
   Script del Pipeline:
```groovy
pipeline {
    agent any
    stages {
        stage('Clonar') {
            steps {
                git branch: 'main', url: 'https://github.com/Franz-Gonzales/FactorialApp.git'
            }
        }
        stage('Compilar') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Pruebas') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Empaquetar') {
            steps {
                sh 'mvn package'
            }
        }
    }
    post {
        success {
            echo 'Build y pruebas exitosas'
        }
        failure {
            echo 'Falló el proceso'
        }
    }
}
```

## Crear el pipeline:
    
- En Jenkins, ve a New Item, nombra el proyecto FactorialApp-Pipeline, selecciona Pipeline, pega el script y guarda.

## Ejecutar el pipeline:
- Haz clic en Build Now y verifica la salida en Console Output.

# 5. Opcional: Archivar el JAR

-   Agrega esta etapa al pipeline para archivar el archivo generado:
```groovy
    stage('Archivar') {
      steps {
          archiveArtifacts artifacts: 'target/FactorialApp-1.0-SNAPSHOT.jar', allowEmptyArchive: true
      }
    }
```


# Notas
- La imagen jenkins/jenkins:lts-jdk21 se utilizó porque no había una versión oficial con Java 23 disponible.
- Las pruebas se realizaron localmente con IntelliJ IDEA y pasaron correctamente.
- El pipeline se ejecutó con éxito el 9 de mayo de 2025, confirmando la funcionalidad del proyecto.


---

### Informe de Resultados

```markdown
# Informe de Resultados - Práctica Jenkins

## Objetivo
El objetivo de esta práctica fue configurar un entorno de integración continua (CI) utilizando Jenkins con Docker, desarrollar una aplicación Java para calcular el factorial de un número, integrar pruebas unitarias con JUnit 5, gestionar el código en un repositorio de GitHub y automatizar el proceso con un pipeline.

## Metodología
1. **Configuración del entorno**:
   - Se utilizó la imagen `jenkins/jenkins:lts-jdk21` en Docker para ejecutar Jenkins, adaptada manualmente para incluir Maven.
   - Se instalaron los plugins sugeridos y se configuró un usuario administrador en la interfaz de Jenkins.

2. **Desarrollo del proyecto**:
   - Se creó un proyecto Maven con el grupo `org.example`, artefacto `FactorialApp` y versión `1.0-SNAPSHOT`.
   - Se implementó la clase `Factorial.java` para calcular el factorial y se añadieron pruebas unitarias en `FactorialTest.java` usando JUnit 5.11.0.
   - El `pom.xml` se configuró para Java 21, con plugins `maven-surefire-plugin` (3.3.1) y `maven-compiler-plugin` (3.13.0).

3. **Gestión del repositorio**:
   - Se creó el repositorio `https://github.com/Franz-Gonzales/FactorialApp.git` en GitHub.
   - El código se desarrolló localmente con IntelliJ IDEA, se probaron las pruebas unitarias y se subieron los cambios a la rama `main`.

4. **Configuración del pipeline**:
   - Se definió un pipeline con las etapas `Clonar`, `Compilar`, `Pruebas` y `Empaquetar`, especificando la rama `main` para evitar errores de clonación.
   - Se ejecutaron múltiples builds, resolviendo inicialmente un error relacionado con la rama predeterminada (`master` vs. `main`).

## Resultados
- **Ejecución exitosa**: El build #3 del pipeline se completó con éxito el 9 de mayo de 2025 a las 20:04:45, con una duración de 57 segundos. Esto confirma que las etapas de clonación, compilación, pruebas y empaquetado funcionaron correctamente.
- **Pruebas unitarias**: Las pruebas locales en IntelliJ IDEA pasaron exitosamente, validando la funcionalidad del cálculo del factorial (casos: 0, 5, y número negativo).
- **Artefacto generado**: Se generó el archivo `FactorialApp-1.0-SNAPSHOT.jar` en el directorio `target` del workspace de Jenkins, verificable en `/var/jenkins_home/workspace/FactorialApp-Pipeline/target/`.
- **Mensaje de éxito**: El pipeline mostró el mensaje `Build y pruebas exitosas` en la sección `post success` (según la configuración del script).

## Conclusiones
- La práctica se llevó a cabo con éxito, adaptando el uso de Java 21 en lugar de Java 17 (especificado en el documento) debido a la disponibilidad de la imagen `jenkins/jenkins:lts-jdk21`.
- El problema inicial de clonación se resolvió especificando la rama `main` en el pipeline, lo que destaca la importancia de alinear la configuración con la estructura del repositorio.
- El uso de Docker y Jenkins automatizó eficazmente el proceso de integración continua, cumpliendo con los objetivos establecidos.

## Recomendaciones
- Considerar la creación de una imagen personalizada de Docker con Java 23 para futuras prácticas si se requiere una versión más reciente.
- Agregar una etapa de archivado en el pipeline (`archiveArtifacts`) para facilitar el acceso al JAR generado.
- Documentar las capturas de pantalla de la consola y el workspace como evidencia adicional.
