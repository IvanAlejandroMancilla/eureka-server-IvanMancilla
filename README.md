# eureka-server

Servidor de descubrimiento de servicios construido con **Spring Boot 3** y **Java 21**, basado en **Netflix Eureka** (vía Spring Cloud). Es el segundo componente que debe iniciarse en el ecosistema, justo después del `config-server`: los demás microservicios (`customer-service`, `product-service`) se registran en él al arrancar.

## ✨ ¿Qué hace?

En una arquitectura de microservicios, cada instancia puede levantarse en un host y puerto distintos, o escalarse a varias réplicas. En lugar de que cada servicio tenga hardcodeada la URL de los demás, `eureka-server` actúa como un **directorio central**:

1. Cada microservicio (`customer-service`, `product-service`), al arrancar, se **registra** en Eureka informando su nombre lógico (`spring.application.name`), host y puerto.
2. Eureka mantiene ese registro actualizado mediante *heartbeats* periódicos de cada instancia.
3. Cuando un servicio necesita comunicarse con otro (por ejemplo, `customer-service` llamando a `product-service` vía **OpenFeign**), en vez de usar una URL fija consulta a Eureka por el nombre lógico (`product-service`) y obtiene la ubicación real de una instancia disponible.

Esto permite que los servicios se descubran entre sí dinámicamente, sin URLs fijas, y habilita balanceo de carga entre múltiples instancias del mismo servicio.

## 🧱 Stack técnico

| Tecnología | Uso |
|---|---|
| Java 21 | Lenguaje |
| Spring Boot 3.3.2 | Framework base |
| Spring Cloud Netflix Eureka Server | Servidor de descubrimiento de servicios |
| Maven | Gestión de dependencias y build |

## 📁 Estructura del proyecto

```
src/main/java/com/ivanmancilla/eurekaserver
└── EurekaServerApplication.java   # Clase principal (@SpringBootApplication, @EnableEurekaServer)

src/main/resources
└── application.yml                # Puerto y configuración del servidor Eureka
```

La anotación `@EnableEurekaServer` es la que activa el servidor de registro y descubrimiento, junto con su dashboard web: no hay controllers ni lógica propia, el comportamiento lo aporta la dependencia `spring-cloud-starter-netflix-eureka-server`.

## ⚙️ Configuración (`application.yml`)

```yaml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  instance:
    hostname: localhost
```

- **`server.port`**: puerto en el que corre el dashboard y la API de Eureka (`8761`).
- **`eureka.client.register-with-eureka: false`**: evita que el propio servidor Eureka intente registrarse como cliente de sí mismo (es el rol por defecto de todo proyecto Spring Cloud con el starter de Eureka, y no aplica cuando el proyecto *es* el servidor).
- **`eureka.client.fetch-registry: false`**: por la misma razón, no necesita descargar el registro de instancias, ya que él mismo lo mantiene.

## ▶️ Cómo ejecutarlo

Debe levantarse **después** del `config-server` y **antes** de `customer-service` y `product-service`, ya que estos se registran en él al arrancar.

```bash
cd eureka-server
./mvnw spring-boot:run
```

## 🔍 Cómo probarlo

Con el servidor corriendo, se puede acceder al dashboard web de Eureka desde el navegador:

```
http://localhost:8761
```

Ahí se ve la lista de instancias registradas (`UP`) en tiempo real. Una vez que `customer-service` y `product-service` estén levantados, ambos deberían aparecer en esa lista con estado `UP`.

## 🔗 Relación con el resto del ecosistema

Ver el diagrama y la guía de arranque completa en el [README principal](../README.md) del repositorio.
