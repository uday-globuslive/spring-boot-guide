# Spring Core Fundamentals

## 1. Introduction to Spring

### 1.1 What is Spring?
- A lightweight Java framework that provides comprehensive infrastructure support for developing Java applications.
- Core Features: 
  - Dependency Injection (DI)
  - Aspect-Oriented Programming (AOP)
  - Declarative transaction management
  - Integration with other frameworks

### 1.2 Environment Setup
- **Prerequisites**:
  - JDK 17 or higher
  - Maven or Gradle build tool
  - IDE (IntelliJ IDEA, Eclipse, VS Code)

```bash
# Install JDK (Ubuntu)
sudo apt install openjdk-17-jdk

# Verify installation
java -version
javac -version

# Install Maven
sudo apt install maven

# Verify Maven
mvn -version
```

## 2. Spring Core Concepts

### 2.1 Inversion of Control (IoC)
- **Definition**: Design principle where control flow is inverted - framework calls your code, not the other way around.
- **Benefits**:
  - Decoupling of execution and implementation
  - Modularity and testability
  - Configuration externalized from code

### 2.2 Dependency Injection
- **Definition**: Pattern where objects receive dependencies rather than creating them.
- **Types**:
  - Constructor Injection (preferred)
  - Setter Injection
  - Field Injection (less recommended)

### 2.3 Spring Bean Lifecycle
- **Definition**: Beans are objects managed by Spring IoC container.
- **Lifecycle Phases**:
  - Instantiation
  - Populating properties
  - BeanNameAware, BeanFactoryAware callbacks
  - PreInitialization (BeanPostProcessors)
  - InitMethod
  - PostInitialization
  - Bean ready for use
  - Destruction

### 2.4 Spring Bean Scopes
- **Singleton**: Default scope, one instance per Spring container
- **Prototype**: New instance each time requested
- **Request**: One instance per HTTP request
- **Session**: One instance per HTTP session
- **Application**: One instance per ServletContext
- **WebSocket**: One instance per WebSocket

## 3. Spring Configuration

### 3.1 XML-based Configuration
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                          http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.example.service.UserServiceImpl">
        <property name="userRepository" ref="userRepository"/>
    </bean>
    
    <bean id="userRepository" class="com.example.repository.UserRepositoryImpl"/>
</beans>
```

### 3.2 Java-based Configuration
```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserRepository userRepository() {
        return new UserRepositoryImpl();
    }
    
    @Bean
    public UserService userService() {
        return new UserServiceImpl(userRepository());
    }
}
```

### 3.3 Annotation-based Configuration
```java
@Component
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // Implementation methods
}
```

### 3.4 Component Scanning
```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Configuration methods
}
```

## 4. Spring AOP (Aspect-Oriented Programming)

### 4.1 AOP Concepts
- **Aspect**: Modularization of a concern that cuts across multiple classes
- **Join Point**: Point during program execution
- **Advice**: Action taken at a particular join point
- **Pointcut**: Expression that matches join points
- **Weaving**: Process of linking aspects with application objects

### 4.2 Types of Advice
- **Before**: Runs before the method execution
- **After**: Runs after the method execution (regardless of outcome)
- **AfterReturning**: Runs after successful method execution
- **AfterThrowing**: Runs after method throws an exception
- **Around**: Most powerful, surrounds method execution

### 4.3 AOP Implementation Example
```java
@Aspect
@Component
public class LoggingAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);
    
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        logger.info("Executing: " + joinPoint.getSignature().getName());
    }
    
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        logger.info("Method returned: " + result);
    }
    
    @Around("@annotation(com.example.annotation.Loggable)")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - start;
        
        logger.info(joinPoint.getSignature() + " executed in " + executionTime + "ms");
        return result;
    }
}
```

## 5. Spring Expression Language (SpEL)
- **Definition**: Expression language supporting querying and manipulating object graphs at runtime.
- **Usage**:
  - Property configuration
  - Conditional expression evaluation
  - Collection manipulation

```java
@Value("#{systemProperties['user.region']}")
private String region;

@Value("#{userService.findById(1)?.name ?: 'Unknown'}")
private String userName;
```

## 6. Spring Events
- **Framework Events**: ContextRefreshedEvent, ContextStartedEvent, etc.
- **Custom Events**: Create your own event classes

```java
// Custom event
public class UserCreatedEvent extends ApplicationEvent {
    private final User user;
    
    public UserCreatedEvent(Object source, User user) {
        super(source);
        this.user = user;
    }
    
    public User getUser() {
        return user;
    }
}

// Event publisher
@Service
public class UserServiceImpl implements UserService {
    private final ApplicationEventPublisher eventPublisher;
    
    @Autowired
    public UserServiceImpl(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }
    
    @Override
    public User createUser(User user) {
        // Save user logic
        eventPublisher.publishEvent(new UserCreatedEvent(this, user));
        return user;
    }
}

// Event listener
@Component
public class UserEventListener {
    
    @EventListener
    public void handleUserCreated(UserCreatedEvent event) {
        // Handle the event
        System.out.println("User created: " + event.getUser().getName());
    }
}
```

## 7. Building a Basic Spring Application

### 7.1 Project Structure
```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           ├── config/
│   │           │   └── AppConfig.java
│   │           ├── model/
│   │           │   └── User.java
│   │           ├── repository/
│   │           │   ├── UserRepository.java
│   │           │   └── UserRepositoryImpl.java
│   │           ├── service/
│   │           │   ├── UserService.java
│   │           │   └── UserServiceImpl.java
│   │           └── Application.java
│   └── resources/
│       └── application.properties
└── test/
    └── java/
        └── com/
            └── example/
                ├── repository/
                │   └── UserRepositoryImplTest.java
                └── service/
                    └── UserServiceImplTest.java
```

### 7.2 Maven Configuration (pom.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                            http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-core-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <spring.version>6.0.10</spring.version>
    </properties>

    <dependencies>
        <!-- Spring Core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!-- Spring AOP -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.19</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.9.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

### 7.3 Model
```java
package com.example.model;

public class User {
    private Long id;
    private String name;
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

### 7.4 Repository
```java
package com.example.repository;

import com.example.model.User;
import java.util.List;
import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
    List<User> findAll();
    void deleteById(Long id);
}
```

```java
package com.example.repository;

import com.example.model.User;
import org.springframework.stereotype.Repository;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class UserRepositoryImpl implements UserRepository {
    
    private final Map<Long, User> users = new ConcurrentHashMap<>();
    private final AtomicLong idCounter = new AtomicLong(1);
    
    @Override
    public User save(User user) {
        if (user.getId() == null) {
            user.setId(idCounter.getAndIncrement());
        }
        users.put(user.getId(), user);
        return user;
    }
    
    @Override
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(users.get(id));
    }
    
    @Override
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
    
    @Override
    public void deleteById(Long id) {
        users.remove(id);
    }
}
```

### 7.5 Service
```java
package com.example.service;

import com.example.model.User;
import java.util.List;
import java.util.Optional;

public interface UserService {
    User createUser(User user);
    Optional<User> getUserById(Long id);
    List<User> getAllUsers();
    void deleteUser(Long id);
}
```

```java
package com.example.service;

import com.example.model.User;
import com.example.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

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
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### 7.6 Configuration
```java
package com.example.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan(basePackages = "com.example")
@EnableAspectJAutoProxy
public class AppConfig {
    // Additional beans if needed
}
```

### 7.7 Main Application
```java
package com.example;

import com.example.config.AppConfig;
import com.example.model.User;
import com.example.service.UserService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.List;

public class Application {
    
    public static void main(String[] args) {
        // Create Spring application context
        AnnotationConfigApplicationContext context = 
            new AnnotationConfigApplicationContext(AppConfig.class);
        
        // Get UserService bean
        UserService userService = context.getBean(UserService.class);
        
        // Create users
        User user1 = new User(null, "John Doe", "john.doe@example.com");
        User user2 = new User(null, "Jane Smith", "jane.smith@example.com");
        
        userService.createUser(user1);
        userService.createUser(user2);
        
        // Retrieve all users
        List<User> users = userService.getAllUsers();
        System.out.println("All users:");
        users.forEach(System.out::println);
        
        // Close context
        context.close();
    }
}
```

## 8. Testing Spring Applications

### 8.1 Unit Testing
```java
package com.example.service;

import com.example.model.User;
import com.example.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class UserServiceImplTest {
    
    @Mock
    private UserRepository userRepository;
    
    private UserService userService;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        userService = new UserServiceImpl(userRepository);
    }
    
    @Test
    void createUser_ShouldSaveUser() {
        // Arrange
        User user = new User(null, "Test User", "test@example.com");
        User savedUser = new User(1L, "Test User", "test@example.com");
        when(userRepository.save(user)).thenReturn(savedUser);
        
        // Act
        User result = userService.createUser(user);
        
        // Assert
        assertNotNull(result);
        assertEquals(savedUser.getId(), result.getId());
        assertEquals(savedUser.getName(), result.getName());
        verify(userRepository, times(1)).save(user);
    }
    
    @Test
    void getUserById_WhenUserExists_ShouldReturnUser() {
        // Arrange
        Long userId = 1L;
        User user = new User(userId, "Test User", "test@example.com");
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        
        // Act
        Optional<User> result = userService.getUserById(userId);
        
        // Assert
        assertTrue(result.isPresent());
        assertEquals(user.getId(), result.get().getId());
        verify(userRepository, times(1)).findById(userId);
    }
    
    @Test
    void getUserById_WhenUserDoesNotExist_ShouldReturnEmpty() {
        // Arrange
        Long userId = 1L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // Act
        Optional<User> result = userService.getUserById(userId);
        
        // Assert
        assertTrue(result.isEmpty());
        verify(userRepository, times(1)).findById(userId);
    }
    
    @Test
    void getAllUsers_ShouldReturnAllUsers() {
        // Arrange
        List<User> users = Arrays.asList(
            new User(1L, "User 1", "user1@example.com"),
            new User(2L, "User 2", "user2@example.com")
        );
        when(userRepository.findAll()).thenReturn(users);
        
        // Act
        List<User> result = userService.getAllUsers();
        
        // Assert
        assertEquals(2, result.size());
        assertEquals(users, result);
        verify(userRepository, times(1)).findAll();
    }
    
    @Test
    void deleteUser_ShouldCallRepository() {
        // Arrange
        Long userId = 1L;
        
        // Act
        userService.deleteUser(userId);
        
        // Assert
        verify(userRepository, times(1)).deleteById(userId);
    }
}
```

### 8.2 Integration Testing
```java
package com.example;

import com.example.config.AppConfig;
import com.example.model.User;
import com.example.service.UserService;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = AppConfig.class)
class ApplicationIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    void testCreateAndRetrieveUser() {
        // Create a user
        User user = new User(null, "Integration Test", "integration@example.com");
        User createdUser = userService.createUser(user);
        
        assertNotNull(createdUser.getId());
        assertEquals(user.getName(), createdUser.getName());
        
        // Retrieve user by ID
        Optional<User> retrievedUserOpt = userService.getUserById(createdUser.getId());
        
        assertTrue(retrievedUserOpt.isPresent());
        User retrievedUser = retrievedUserOpt.get();
        assertEquals(createdUser.getId(), retrievedUser.getId());
        assertEquals(createdUser.getName(), retrievedUser.getName());
        assertEquals(createdUser.getEmail(), retrievedUser.getEmail());
    }
    
    @Test
    void testGetAllUsers() {
        // Create test users
        userService.createUser(new User(null, "Test User 1", "test1@example.com"));
        userService.createUser(new User(null, "Test User 2", "test2@example.com"));
        
        // Get all users
        List<User> users = userService.getAllUsers();
        
        // We cannot assert exact size because of shared state between tests
        assertFalse(users.isEmpty());
    }
}
```