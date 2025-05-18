# Spring Boot Fundamentals

## 1. Introduction to Spring Boot

### 1.1 What is Spring Boot?
- An extension of Spring Framework that simplifies application development
- Provides auto-configuration for Spring ecosystem
- Embedded server support (Tomcat, Jetty, Undertow)
- "Convention over configuration" approach
- Production-ready features (metrics, health checks, externalized configuration)

### 1.2 Spring Boot vs. Standard Spring
- **Spring Framework**: Provides core support for dependency injection, transaction management, web apps, data access, messaging, and testing
- **Spring Boot**: Simplifies Spring configuration with auto-configuration, starter dependencies, embedded server, and production-ready features

## 2. Setting Up a Spring Boot Project

### 2.1 Using Spring Initializr
- Web interface: [https://start.spring.io/](https://start.spring.io/)
- Command line with Spring Boot CLI
- IDE integration (IntelliJ, Eclipse, VS Code)

### 2.2 Maven Configuration
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>springboot-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-demo</name>
    <description>Demo project for Spring Boot</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.3 Gradle Configuration
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.5'
    id 'io.spring.dependency-management' version '1.1.3'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### 2.4 Project Structure
```
springboot-demo/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── demo/
│   │   │               ├── controller/
│   │   │               ├── model/
│   │   │               ├── repository/
│   │   │               ├── service/
│   │   │               └── DemoApplication.java
│   │   └── resources/
│   │       ├── static/
│   │       ├── templates/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── demo/
│                       └── DemoApplicationTests.java
├── .gitignore
├── mvnw
├── mvnw.cmd
└── pom.xml
```

## 3. Spring Boot Core Concepts

### 3.1 Auto-Configuration
- Spring Boot automatically configures Spring application based on:
  - Classpath dependencies
  - Property settings
  - Beans defined in your code

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

The `@SpringBootApplication` annotation combines:
- `@Configuration`: Tags the class as a source of bean definitions
- `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings
- `@ComponentScan`: Tells Spring to look for components in the current package and below

### 3.2 Starter Dependencies
- Curated sets of dependencies for specific functionality
- Common starters:
  - `spring-boot-starter-web`: Web development with Spring MVC
  - `spring-boot-starter-data-jpa`: JPA with Hibernate
  - `spring-boot-starter-security`: Spring Security
  - `spring-boot-starter-test`: Testing framework

### 3.3 Externalized Configuration
Spring Boot supports externalized configuration from multiple sources (in order of precedence):
1. Command-line arguments
2. JNDI attributes
3. Java System properties
4. OS environment variables
5. Profile-specific properties (`application-{profile}.properties`)
6. Application properties (`application.properties` or `application.yml`)
7. Default properties

#### application.properties example:
```properties
# Server configuration
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.org.springframework=INFO
logging.level.com.example=DEBUG

# Custom properties
app.name=My Spring Boot Application
app.description=A sample Spring Boot application
app.version=1.0.0
```

#### application.yml example:
```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    org.springframework: INFO
    com.example: DEBUG

app:
  name: My Spring Boot Application
  description: A sample Spring Boot application
  version: 1.0.0
```

### 3.4 Configuration Properties
- Type-safe configuration with `@ConfigurationProperties`

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private String name;
    private String description;
    private String version;
    
    // Getters and setters
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getDescription() {
        return description;
    }
    
    public void setDescription(String description) {
        this.description = description;
    }
    
    public String getVersion() {
        return version;
    }
    
    public void setVersion(String version) {
        this.version = version;
    }
}

// Enable configuration properties scanning
@SpringBootApplication
@EnableConfigurationProperties
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 3.5 Profiles
- Environment-specific configuration
- Activate with `spring.profiles.active` property

```properties
# Default configuration (application.properties)
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver

# Development profile (application-dev.properties)
spring.datasource.url=jdbc:mysql://localhost:3306/dev_db
spring.datasource.username=dev_user
spring.datasource.password=dev_pass

# Production profile (application-prod.properties)
spring.datasource.url=jdbc:mysql://prod-server:3306/prod_db
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

Activate profiles:
```bash
# Command line
java -jar app.jar --spring.profiles.active=dev

# Or in application.properties
spring.profiles.active=dev

# Or programmatically
SpringApplication app = new SpringApplication(DemoApplication.class);
app.setAdditionalProfiles("dev");
app.run(args);
```

### 3.6 Spring Boot DevTools
- Automatic restart when files change
- LiveReload server
- Property defaults tuned for development
- H2 console if H2 is on the classpath

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

## 4. Web Applications with Spring Boot

### 4.1 Creating REST Controllers
```java
package com.example.demo.controller;

import com.example.demo.model.User;
import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User createdUser = userService.createUser(user);
        return new ResponseEntity<>(createdUser, HttpStatus.CREATED);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id)
                .map(user -> new ResponseEntity<>(user, HttpStatus.OK))
                .orElse(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return new ResponseEntity<>(users, HttpStatus.OK);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.getUserById(id)
                .map(existingUser -> {
                    user.setId(id);
                    User updatedUser = userService.updateUser(user);
                    return new ResponseEntity<>(updatedUser, HttpStatus.OK);
                })
                .orElse(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        return userService.getUserById(id)
                .map(user -> {
                    userService.deleteUser(id);
                    return new ResponseEntity<Void>(HttpStatus.NO_CONTENT);
                })
                .orElse(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
}
```

### 4.2 Error Handling
```java
package com.example.demo.exception;

import java.time.LocalDateTime;

public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    
    // Constructors, getters, and setters
    public ErrorResponse() {
        this.timestamp = LocalDateTime.now();
    }
    
    public ErrorResponse(int status, String error, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }
    
    // Getters and setters
    // ...
}
```

```java
package com.example.demo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

```java
package com.example.demo.exception;

import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
            ResourceNotFoundException ex, HttpServletRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Not Found",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex, HttpServletRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### 4.3 Static Content
- Placed in `/src/main/resources/static/`
- Accessed directly from the root path
- Supports HTML, CSS, JavaScript, images, etc.

### 4.4 Template Engines
- Thymeleaf, FreeMarker, Mustache, JSP
- Configuration example for Thymeleaf:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```java
@Controller
public class WebController {
    
    @GetMapping("/greeting")
    public String greeting(@RequestParam(name="name", required=false, defaultValue="World") String name, Model model) {
        model.addAttribute("name", name);
        return "greeting"; // Resolves to /templates/greeting.html
    }
}
```

```html
<!DOCTYPE HTML>
<!-- src/main/resources/templates/greeting.html -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Greeting</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```

## 5. Running and Packaging Spring Boot Applications

### 5.1 Running Applications
- IDE: Run as Java application
- Command line: `./mvnw spring-boot:run` or `./gradlew bootRun`
- Executable JAR: `java -jar target/app.jar`

### 5.2 Creating Executable JAR/WAR
- Maven: `./mvnw clean package`
- Gradle: `./gradlew build`

### 5.3 Application Properties
```properties
# Banner (ASCII art at startup)
spring.main.banner-mode=off

# Server
server.port=8080
server.compression.enabled=true

# Tomcat
server.tomcat.max-threads=200
server.tomcat.accesslog.enabled=true

# Application context
server.servlet.context-path=/api
```

### 5.4 Customizing the Banner
Create a `banner.txt` file in resources directory:
```
  _____            _              ____              _   
 / ____|          (_)            |  _ \            | |  
| (___  _ __  _ __ _ _ __   __ _ | |_) | ___   ___ | |_ 
 \___ \| '_ \| '__| | '_ \ / _` ||  _ < / _ \ / _ \| __|
 ____) | |_) | |  | | | | | (_| || |_) | (_) | (_) | |_ 
|_____/| .__/|_|  |_|_| |_|\__, ||____/ \___/ \___/ \__|
       | |                  __/ |                       
       |_|                 |___/                        
```

### 5.5 Command Line Runner
```java
@SpringBootApplication
public class DemoApplication implements CommandLineRunner {
    
    private final UserService userService;
    
    @Autowired
    public DemoApplication(UserService userService) {
        this.userService = userService;
    }
    
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
    
    @Override
    public void run(String... args) throws Exception {
        // Code to run at application startup
        System.out.println("Initializing application data...");
        
        // Create initial users
        User user1 = new User(null, "Admin User", "admin@example.com");
        userService.createUser(user1);
        
        System.out.println("Application data initialized!");
    }
}
```

## 6. Spring Boot Actuator

### 6.1 Overview
- Production-ready features to monitor and manage applications
- Health checks, metrics, info, environment, etc.

### 6.2 Setup
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 6.3 Endpoints
- `/actuator/health` - Application health
- `/actuator/info` - Application info
- `/actuator/metrics` - Application metrics
- `/actuator/env` - Environment variables
- `/actuator/loggers` - Logger configuration
- `/actuator/mappings` - Request mappings
- `/actuator/beans` - Spring beans

### 6.4 Configuration
```properties
# Enable all endpoints
management.endpoints.web.exposure.include=*

# Enable specific endpoints
# management.endpoints.web.exposure.include=health,info,metrics,loggers

# Disable specific endpoints
management.endpoints.web.exposure.exclude=env,beans

# Base path for actuator endpoints
management.endpoints.web.base-path=/manage

# Health endpoint
management.endpoint.health.show-details=always

# Info endpoint
info.app.name=@project.name@
info.app.description=@project.description@
info.app.version=@project.version@
info.app.java.version=@java.version@
```

### 6.5 Custom Health Indicator
```java
package com.example.demo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        // Check database connectivity
        boolean isHealthy = checkDatabaseConnection();
        
        if (isHealthy) {
            return Health.up()
                    .withDetail("database", "Database is up and running")
                    .build();
        } else {
            return Health.down()
                    .withDetail("database", "Database connection failed")
                    .build();
        }
    }
    
    private boolean checkDatabaseConnection() {
        // Implementation to check database connectivity
        return true; // Placeholder
    }
}
```

## 7. Testing in Spring Boot

### 7.1 Test Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 7.2 Unit Testing Controllers
```java
package com.example.demo.controller;

import com.example.demo.model.User;
import com.example.demo.service.UserService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.hamcrest.Matchers.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void getUserById_ShouldReturnUser() throws Exception {
        // Arrange
        User user = new User(1L, "Test User", "test@example.com");
        when(userService.getUserById(1L)).thenReturn(Optional.of(user));
        
        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
               .andExpect(status().isOk())
               .andExpect(content().contentType(MediaType.APPLICATION_JSON))
               .andExpect(jsonPath("$.id", is(1)))
               .andExpect(jsonPath("$.name", is("Test User")))
               .andExpect(jsonPath("$.email", is("test@example.com")));
    }
    
    @Test
    void getUserById_UserNotFound_ShouldReturn404() throws Exception {
        // Arrange
        when(userService.getUserById(1L)).thenReturn(Optional.empty());
        
        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
               .andExpect(status().isNotFound());
    }
    
    @Test
    void getAllUsers_ShouldReturnList() throws Exception {
        // Arrange
        List<User> users = Arrays.asList(
            new User(1L, "User 1", "user1@example.com"),
            new User(2L, "User 2", "user2@example.com")
        );
        when(userService.getAllUsers()).thenReturn(users);
        
        // Act & Assert
        mockMvc.perform(get("/api/users"))
               .andExpect(status().isOk())
               .andExpect(content().contentType(MediaType.APPLICATION_JSON))
               .andExpect(jsonPath("$", hasSize(2)))
               .andExpect(jsonPath("$[0].id", is(1)))
               .andExpect(jsonPath("$[0].name", is("User 1")))
               .andExpect(jsonPath("$[1].id", is(2)))
               .andExpect(jsonPath("$[1].name", is("User 2")));
    }
    
    @Test
    void createUser_ShouldReturnCreated() throws Exception {
        // Arrange
        User userToCreate = new User(null, "New User", "new@example.com");
        User createdUser = new User(1L, "New User", "new@example.com");
        
        when(userService.createUser(any(User.class))).thenReturn(createdUser);
        
        // Act & Assert
        mockMvc.perform(post("/api/users")
               .contentType(MediaType.APPLICATION_JSON)
               .content(objectMapper.writeValueAsString(userToCreate)))
               .andExpect(status().isCreated())
               .andExpect(jsonPath("$.id", is(1)))
               .andExpect(jsonPath("$.name", is("New User")))
               .andExpect(jsonPath("$.email", is("new@example.com")));
    }
}
```

### 7.3 Integration Testing
```java
package com.example.demo;

import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class UserIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @BeforeEach
    void setup() {
        userRepository.deleteAll();
    }
    
    @Test
    void userLifecycle() throws Exception {
        // Create a user
        User userToCreate = new User(null, "Integration Test", "integration@example.com");
        
        // Test POST /api/users
        String createResponse = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userToCreate)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name", is("Integration Test")))
                .andExpect(jsonPath("$.email", is("integration@example.com")))
                .andReturn().getResponse().getContentAsString();
        
        // Extract user ID from response
        User createdUser = objectMapper.readValue(createResponse, User.class);
        Long userId = createdUser.getId();
        
        // Test GET /api/users/{id}
        mockMvc.perform(get("/api/users/" + userId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id", is(userId.intValue())))
                .andExpect(jsonPath("$.name", is("Integration Test")));
        
        // Test PUT /api/users/{id}
        User userToUpdate = new User(userId, "Updated User", "updated@example.com");
        mockMvc.perform(put("/api/users/" + userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userToUpdate)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is("Updated User")))
                .andExpect(jsonPath("$.email", is("updated@example.com")));
        
        // Test DELETE /api/users/{id}
        mockMvc.perform(delete("/api/users/" + userId))
                .andExpect(status().isNoContent());
        
        // Verify user is deleted
        mockMvc.perform(get("/api/users/" + userId))
                .andExpect(status().isNotFound());
    }
}
```

### 7.4 TestRestTemplate
```java
package com.example.demo;

import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserRestTemplateTest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    private String getBaseUrl() {
        return "http://localhost:" + port + "/api/users";
    }
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    void testCreateUser() {
        // Arrange
        User user = new User();
        user.setName("Test User");
        user.setEmail("test@example.com");
        
        // Act
        ResponseEntity<User> response = restTemplate.postForEntity(getBaseUrl(), user, User.class);
        
        // Assert
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertNotNull(response.getBody().getId());
        assertEquals("Test User", response.getBody().getName());
    }
    
    @Test
    void testGetUserById() {
        // Arrange - Create a user first
        User user = new User();
        user.setName("Get Test");
        user.setEmail("get@example.com");
        
        User createdUser = restTemplate.postForObject(getBaseUrl(), user, User.class);
        
        // Act
        ResponseEntity<User> response = restTemplate.getForEntity(
                getBaseUrl() + "/" + createdUser.getId(), User.class);
        
        // Assert
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals(createdUser.getId(), response.getBody().getId());
        assertEquals("Get Test", response.getBody().getName());
    }
    
    @Test
    void testUpdateUser() {
        // Arrange - Create a user first
        User user = new User();
        user.setName("Update Test");
        user.setEmail("update@example.com");
        
        User createdUser = restTemplate.postForObject(getBaseUrl(), user, User.class);
        
        // Update user
        createdUser.setName("Updated Name");
        
        // Act
        HttpEntity<User> requestEntity = new HttpEntity<>(createdUser);
        ResponseEntity<User> response = restTemplate.exchange(
                getBaseUrl() + "/" + createdUser.getId(), HttpMethod.PUT, requestEntity, User.class);
        
        // Assert
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("Updated Name", response.getBody().getName());
    }
    
    @Test
    void testDeleteUser() {
        // Arrange - Create a user first
        User user = new User();
        user.setName("Delete Test");
        user.setEmail("delete@example.com");
        
        User createdUser = restTemplate.postForObject(getBaseUrl(), user, User.class);
        
        // Act
        restTemplate.delete(getBaseUrl() + "/" + createdUser.getId());
        
        // Assert - Try to get the deleted user
        ResponseEntity<User> response = restTemplate.getForEntity(
                getBaseUrl() + "/" + createdUser.getId(), User.class);
        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
    }
}
```

## 8. Building a Complete Spring Boot Application

Here is a simplified application structure for a user management system:

### 8.1 Entity
```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    // Constructors
    public User() {}
    
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    @Override
    public String toString() {
        return "User{id=" + id + ", name='" + name + "', email='" + email + "'}";
    }
}
```

### 8.2 Repository
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameContainingIgnoreCase(String name);
    boolean existsByEmail(String email);
}
```

### 8.3 Service
```java
public interface UserService {
    User createUser(User user);
    Optional<User> getUserById(Long id);
    List<User> getAllUsers();
    List<User> getUsersByName(String name);
    User updateUser(User user);
    void deleteUser(Long id);
    boolean isEmailTaken(String email);
}
```

```java
@Service
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    @Override
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    @Override
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    @Override
    public List<User> getUsersByName(String name) {
        return userRepository.findByNameContainingIgnoreCase(name);
    }
    
    @Override
    public User updateUser(User user) {
        if (!userRepository.existsById(user.getId())) {
            throw new ResourceNotFoundException("User not found with id: " + user.getId());
        }
        return userRepository.save(user);
    }
    
    @Override
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }
        userRepository.deleteById(id);
    }
    
    @Override
    public boolean isEmailTaken(String email) {
        return userRepository.existsByEmail(email);
    }
}
```

### 8.4 Controller
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        if (userService.isEmailTaken(user.getEmail())) {
            throw new IllegalArgumentException("Email is already in use");
        }
        User createdUser = userService.createUser(user);
        return new ResponseEntity<>(createdUser, HttpStatus.CREATED);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id)
                .map(user -> new ResponseEntity<>(user, HttpStatus.OK))
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers(
            @RequestParam(required = false) String name) {
        List<User> users;
        if (name != null && !name.isEmpty()) {
            users = userService.getUsersByName(name);
        } else {
            users = userService.getAllUsers();
        }
        return new ResponseEntity<>(users, HttpStatus.OK);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, @Valid @RequestBody User user) {
        
        if (!id.equals(user.getId())) {
            throw new IllegalArgumentException("ID in path must match ID in request body");
        }
        
        // Check if email is taken by a different user
        if (userService.isEmailTaken(user.getEmail()) &&
                !userService.getUserById(id).get().getEmail().equals(user.getEmail())) {
            throw new IllegalArgumentException("Email is already in use");
        }
        
        User updatedUser = userService.updateUser(user);
        return new ResponseEntity<>(updatedUser, HttpStatus.OK);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Map<String, Boolean>> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        
        Map<String, Boolean> response = new HashMap<>();
        response.put("deleted", Boolean.TRUE);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}
```

### 8.5 Exception Handler
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
            ResourceNotFoundException ex, HttpServletRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Not Found",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(
            IllegalArgumentException ex, HttpServletRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity.badRequest().body(errors);
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<Map<String, String>> handleConstraintViolationException(
            ConstraintViolationException ex) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getConstraintViolations().forEach(violation -> {
            String propertyPath = violation.getPropertyPath().toString();
            String fieldName = propertyPath.substring(propertyPath.lastIndexOf('.') + 1);
            String errorMessage = violation.getMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity.badRequest().body(errors);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex, HttpServletRequest request) {
        
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                ex.getMessage(),
                request.getRequestURI()
        );
        
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### 8.6 Application Properties
```properties
# Application
spring.application.name=user-management-system

# Server
server.port=8080
server.servlet.context-path=/

# Database Configuration
spring.datasource.url=jdbc:h2:mem:userdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA/Hibernate
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.org.springframework=INFO
logging.level.com.example.demo=DEBUG
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
info.app.name=User Management System
info.app.description=A Spring Boot demo application
info.app.version=1.0.0
```

### 8.7 Main Application Class
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserManagementApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(UserManagementApplication.class, args);
    }
}
```