# REST API Development with Spring Boot

This guide provides a comprehensive overview of developing RESTful APIs using Spring Boot, covering everything from basic concepts to advanced patterns and best practices.

## Table of Contents

1. [Introduction to REST](#introduction-to-rest)
2. [Spring Web MVC Fundamentals](#spring-web-mvc-fundamentals)
3. [Building a Basic REST API](#building-a-basic-rest-api)
4. [HTTP Methods and Status Codes](#http-methods-and-status-codes)
5. [Request Mapping and Path Variables](#request-mapping-and-path-variables)
6. [Request Parameters and Request Body](#request-parameters-and-request-body)
7. [Response Handling](#response-handling)
8. [Content Negotiation](#content-negotiation)
9. [Exception Handling](#exception-handling)
10. [Validation](#validation)
11. [HATEOAS Implementation](#hateoas-implementation)
12. [Documentation with Swagger/OpenAPI](#documentation-with-swaggeropenapi)
13. [Versioning Strategies](#versioning-strategies)
14. [Rate Limiting and API Security](#rate-limiting-and-api-security)
15. [Testing REST APIs](#testing-rest-apis)
16. [Practical Case Study](#practical-case-study)

## Introduction to REST

REST (Representational State Transfer) is an architectural style for designing networked applications. RESTful APIs use HTTP requests to perform CRUD (Create, Read, Update, Delete) operations.

### Key Principles of REST

1. **Stateless**: Each request contains all information needed to process it
2. **Resource-Based**: Resources are identified by URIs
3. **Client-Server**: Separation of concerns between client and server
4. **Uniform Interface**: Consistent way to interact with resources
5. **Cacheable**: Responses must define themselves as cacheable or non-cacheable
6. **Layered System**: Client can't tell if it's connected directly to the end server

### Benefits of RESTful Architecture

- **Scalability**: Stateless nature enables horizontal scaling
- **Flexibility**: Separation of client and server
- **Independence**: Technology stack independence
- **Visibility**: Easy monitoring and debugging

## Spring Web MVC Fundamentals

Spring MVC is the web framework that powers Spring Boot's REST capabilities.

### Key Components

- **DispatcherServlet**: Central servlet that dispatches requests to handlers
- **HandlerMapping**: Maps requests to handlers
- **Controller**: Processes requests and builds responses
- **ViewResolver**: Resolves logical view names to actual views
- **Model**: Holds data to be displayed in the view

### Spring Boot Web Starter

The `spring-boot-starter-web` dependency includes:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This starter provides:
- Embedded web server (Tomcat by default)
- Auto-configuration for Spring MVC
- JSON serialization/deserialization support
- Static resource handling

## Building a Basic REST API

Let's create a simple API for managing a product catalog.

### Project Setup

1. Create a Spring Boot project with Spring Web dependency
2. Structure the project with proper packages:
   - `model` - Data classes
   - `repository` - Data access
   - `service` - Business logic
   - `controller` - API endpoints
   - `exception` - Custom exceptions and handlers

### Model Class

```java
package com.example.restapi.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Product {
    
    private Long id;
    private String name;
    private String description;
    private BigDecimal price;
    private Integer quantity;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors, getters and setters
    
    public Product() {
    }
    
    public Product(Long id, String name, String description, BigDecimal price, Integer quantity) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.price = price;
        this.quantity = quantity;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public Integer getQuantity() { return quantity; }
    public void setQuantity(Integer quantity) { this.quantity = quantity; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

### Repository Layer

```java
package com.example.restapi.repository;

import com.example.restapi.model.Product;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class ProductRepository {
    
    private final Map<Long, Product> productMap = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public ProductRepository() {
        // Add some sample products
        Product product1 = new Product(idGenerator.getAndIncrement(), "Laptop", "High-performance laptop", new BigDecimal("999.99"), 10);
        Product product2 = new Product(idGenerator.getAndIncrement(), "Smartphone", "Latest smartphone model", new BigDecimal("699.99"), 20);
        Product product3 = new Product(idGenerator.getAndIncrement(), "Headphones", "Noise-cancelling headphones", new BigDecimal("199.99"), 15);
        
        productMap.put(product1.getId(), product1);
        productMap.put(product2.getId(), product2);
        productMap.put(product3.getId(), product3);
    }
    
    public List<Product> findAll() {
        return new ArrayList<>(productMap.values());
    }
    
    public Optional<Product> findById(Long id) {
        return Optional.ofNullable(productMap.get(id));
    }
    
    public Product save(Product product) {
        if (product.getId() == null) {
            product.setId(idGenerator.getAndIncrement());
        }
        productMap.put(product.getId(), product);
        return product;
    }
    
    public void deleteById(Long id) {
        productMap.remove(id);
    }
}
```

### Service Layer

```java
package com.example.restapi.service;

import com.example.restapi.exception.ResourceNotFoundException;
import com.example.restapi.model.Product;
import com.example.restapi.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;

@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }
    
    public Product getProductById(Long id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found with id: " + id));
    }
    
    public Product createProduct(Product product) {
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());
        return productRepository.save(product);
    }
    
    public Product updateProduct(Long id, Product productDetails) {
        Product product = getProductById(id);
        
        product.setName(productDetails.getName());
        product.setDescription(productDetails.getDescription());
        product.setPrice(productDetails.getPrice());
        product.setQuantity(productDetails.getQuantity());
        product.setUpdatedAt(LocalDateTime.now());
        
        return productRepository.save(product);
    }
    
    public void deleteProduct(Long id) {
        // Check if product exists
        getProductById(id);
        productRepository.deleteById(id);
    }
}
```

### Exception Class

```java
package com.example.restapi.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### Controller Layer

```java
package com.example.restapi.controller;

import com.example.restapi.model.Product;
import com.example.restapi.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts() {
        List<Product> products = productService.getAllProducts();
        return new ResponseEntity<>(products, HttpStatus.OK);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        return new ResponseEntity<>(product, HttpStatus.OK);
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product createdProduct = productService.createProduct(product);
        return new ResponseEntity<>(createdProduct, HttpStatus.CREATED);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody Product product) {
        Product updatedProduct = productService.updateProduct(id, product);
        return new ResponseEntity<>(updatedProduct, HttpStatus.OK);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    }
}
```

### Main Application Class

```java
package com.example.restapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestApiApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(RestApiApplication.class, args);
    }
}
```

## HTTP Methods and Status Codes

HTTP methods represent the actions to be performed on resources:

### Common HTTP Methods

| Method  | Purpose                              | Idempotent | Safe |
|---------|--------------------------------------|------------|------|
| GET     | Retrieve a resource                  | Yes        | Yes  |
| POST    | Create a new resource                | No         | No   |
| PUT     | Update an existing resource          | Yes        | No   |
| DELETE  | Remove a resource                    | Yes        | No   |
| PATCH   | Partially update an existing resource| No         | No   |
| HEAD    | Same as GET but without body         | Yes        | Yes  |
| OPTIONS | Get supported methods for a URL      | Yes        | Yes  |

### Common HTTP Status Codes

| Code | Category      | Description                                  |
|------|---------------|----------------------------------------------|
| 200  | Success       | OK - Request succeeded                       |
| 201  | Success       | Created - Resource created successfully      |
| 204  | Success       | No Content - Request succeeded with no body  |
| 400  | Client Error  | Bad Request - Invalid syntax                 |
| 401  | Client Error  | Unauthorized - Authentication required       |
| 403  | Client Error  | Forbidden - No permission                    |
| 404  | Client Error  | Not Found - Resource doesn't exist           |
| 409  | Client Error  | Conflict - Request conflicts with state      |
| 500  | Server Error  | Internal Server Error - Unexpected condition |

In Spring Boot, the `ResponseEntity` class allows you to set both status codes and response bodies:

```java
@GetMapping("/{id}")
public ResponseEntity<Product> getProductById(@PathVariable Long id) {
    Product product = productService.getProductById(id);
    return new ResponseEntity<>(product, HttpStatus.OK);
}
```

## Request Mapping and Path Variables

Spring provides several annotations for mapping HTTP requests to controller methods:

### Request Mapping Annotations

- `@RequestMapping` - General purpose request mapping
- `@GetMapping` - Shortcut for `@RequestMapping(method = RequestMethod.GET)`
- `@PostMapping` - Shortcut for `@RequestMapping(method = RequestMethod.POST)`
- `@PutMapping` - Shortcut for `@RequestMapping(method = RequestMethod.PUT)`
- `@DeleteMapping` - Shortcut for `@RequestMapping(method = RequestMethod.DELETE)`
- `@PatchMapping` - Shortcut for `@RequestMapping(method = RequestMethod.PATCH)`

### Path Variables

Path variables extract values from the URI path:

```java
@GetMapping("/products/{id}")
public Product getProduct(@PathVariable Long id) {
    return productService.findById(id);
}
```

Multiple path variables can be used:

```java
@GetMapping("/categories/{categoryId}/products/{productId}")
public Product getCategoryProduct(
    @PathVariable Long categoryId,
    @PathVariable Long productId
) {
    return productService.findByCategoryAndProductId(categoryId, productId);
}
```

You can customize path variable names:

```java
@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(
    @PathVariable("userId") Long id,
    @PathVariable("orderId") Long orderNumber
) {
    return orderService.findByUserAndOrderId(id, orderNumber);
}
```

## Request Parameters and Request Body

### Request Parameters

Request parameters are extracted from the query string:

```java
@GetMapping("/products")
public List<Product> searchProducts(
    @RequestParam(required = false) String name,
    @RequestParam(defaultValue = "0") Integer minPrice,
    @RequestParam(defaultValue = "10") Integer page,
    @RequestParam(defaultValue = "10") Integer size
) {
    return productService.searchProducts(name, minPrice, page, size);
}
```

This method handles requests like:
- `/products?name=laptop`
- `/products?minPrice=500&page=2&size=20`

### Request Body

The `@RequestBody` annotation binds the request body to a method parameter:

```java
@PostMapping("/products")
public ResponseEntity<Product> createProduct(@RequestBody Product product) {
    Product savedProduct = productService.save(product);
    return new ResponseEntity<>(savedProduct, HttpStatus.CREATED);
}
```

## Response Handling

Spring Boot provides various ways to handle responses:

### Return Types

1. **ResponseEntity**: Complete control over response

```java
@GetMapping("/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);
    
    if (product == null) {
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
    
    return new ResponseEntity<>(product, HttpStatus.OK);
}
```

2. **Direct Object Return**: Spring automatically converts to JSON/XML

```java
@GetMapping("/simple/{id}")
public Product getProductSimple(@PathVariable Long id) {
    return productService.findById(id);
}
```

3. **ResponseEntity Builder Methods**: Fluent API for response building

```java
@GetMapping("/builder/{id}")
public ResponseEntity<Product> getProductBuilder(@PathVariable Long id) {
    Product product = productService.findById(id);
    
    return ResponseEntity
            .ok()
            .header("Custom-Header", "value")
            .body(product);
}

@PostMapping("/create")
public ResponseEntity<Product> createProduct(@RequestBody Product product) {
    Product saved = productService.save(product);
    
    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(saved.getId())
            .toUri();
    
    return ResponseEntity
            .created(location)
            .body(saved);
}
```

## Content Negotiation

Content negotiation allows clients to request data in different formats.

### Configuration

In `application.properties` or `application.yml`:

```properties
spring.mvc.contentnegotiation.favor-parameter=true
spring.mvc.contentnegotiation.parameter-name=format
```

### Supporting Multiple Formats

1. Add the necessary dependencies:

```xml
<!-- XML support -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

2. Use `produces` in the controller:

```java
@GetMapping(
    value = "/{id}",
    produces = { 
        MediaType.APPLICATION_JSON_VALUE,
        MediaType.APPLICATION_XML_VALUE 
    }
)
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);
    return ResponseEntity.ok(product);
}
```

Clients can request XML using:
- Content-Type header: `Content-Type: application/xml`
- Accept header: `Accept: application/xml`
- Format parameter: `/products/1?format=xml` (if enabled)

## Exception Handling

Spring Boot provides several ways to handle exceptions:

### @ExceptionHandler

Handle exceptions within a specific controller:

```java
@RestController
public class ProductController {
    
    // Regular controller methods
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

### @ControllerAdvice / @RestControllerAdvice

Handle exceptions globally across controllers:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Error Response Model

```java
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;
    
    // Constructor, getters, and setters
    
    public ErrorResponse(int status, String message, long timestamp) {
        this.status = status;
        this.message = message;
        this.timestamp = timestamp;
    }
    
    // Getters and setters
}
```

## Validation

Spring Boot integrates with Bean Validation (JSR-380) for input validation:

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### Model Validation

```java
public class ProductDTO {
    
    @NotNull(message = "Name cannot be null")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @NotNull(message = "Price cannot be null")
    @Positive(message = "Price must be positive")
    private BigDecimal price;
    
    @Min(value = 0, message = "Quantity cannot be negative")
    private Integer quantity;
    
    // Getters and setters
}
```

### Controller Validation

```java
@PostMapping
public ResponseEntity<Product> createProduct(@Valid @RequestBody ProductDTO productDTO) {
    Product product = convertToEntity(productDTO);
    Product savedProduct = productService.save(product);
    return new ResponseEntity<>(savedProduct, HttpStatus.CREATED);
}

private Product convertToEntity(ProductDTO productDTO) {
    // Conversion logic
    Product product = new Product();
    product.setName(productDTO.getName());
    product.setDescription(productDTO.getDescription());
    product.setPrice(productDTO.getPrice());
    product.setQuantity(productDTO.getQuantity());
    return product;
}
```

### Validation Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        ValidationErrorResponse error = new ValidationErrorResponse();
        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage("Validation failed");
        error.setTimestamp(System.currentTimeMillis());
        
        ex.getBindingResult().getFieldErrors().forEach(fieldError -> {
            error.addError(fieldError.getField(), fieldError.getDefaultMessage());
        });
        
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

### Validation Error Response

```java
public class ValidationErrorResponse {
    private int status;
    private String message;
    private long timestamp;
    private Map<String, String> errors = new HashMap<>();
    
    // Getters and setters
    
    public void addError(String field, String message) {
        errors.put(field, message);
    }
}
```

## HATEOAS Implementation

HATEOAS (Hypermedia as the Engine of Application State) makes APIs self-documenting through links.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

### Implementation

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/{id}")
    public EntityModel<Product> getProductById(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        
        return EntityModel.of(product,
            linkTo(methodOn(ProductController.class).getProductById(id)).withSelfRel(),
            linkTo(methodOn(ProductController.class).getAllProducts()).withRel("all-products")
        );
    }
    
    @GetMapping
    public CollectionModel<EntityModel<Product>> getAllProducts() {
        List<Product> products = productService.getAllProducts();
        
        List<EntityModel<Product>> productModels = products.stream()
            .map(product -> EntityModel.of(product,
                linkTo(methodOn(ProductController.class).getProductById(product.getId())).withSelfRel(),
                linkTo(methodOn(ProductController.class).getAllProducts()).withRel("all-products")
            ))
            .collect(Collectors.toList());
        
        return CollectionModel.of(productModels,
            linkTo(methodOn(ProductController.class).getAllProducts()).withSelfRel());
    }
}
```

### Sample HATEOAS Response

```json
{
    "id": 1,
    "name": "Laptop",
    "description": "High-performance laptop",
    "price": 999.99,
    "quantity": 10,
    "createdAt": "2023-01-15T10:30:45.123",
    "updatedAt": "2023-01-15T10:30:45.123",
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/products/1"
        },
        "all-products": {
            "href": "http://localhost:8080/api/products"
        }
    }
}
```

## Documentation with Swagger/OpenAPI

Swagger/OpenAPI provides automated API documentation:

### Dependencies

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.4</version>
</dependency>
```

### Configuration

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Product API")
                        .version("1.0")
                        .description("API for managing products")
                        .contact(new Contact()
                                .name("API Support")
                                .email("support@example.com"))
                )
                .externalDocs(new ExternalDocumentation()
                        .description("Product API Documentation")
                        .url("https://example.com/docs"));
    }
}
```

### API Documentation

```java
@RestController
@RequestMapping("/api/products")
@Tag(name = "Product Management", description = "Endpoints for managing products")
public class ProductController {
    
    @Operation(
        summary = "Get a product by ID",
        description = "Retrieves a product based on the provided ID",
        responses = {
            @ApiResponse(
                responseCode = "200", 
                description = "Successfully retrieved the product",
                content = @Content(mediaType = "application/json",
                    schema = @Schema(implementation = Product.class))
            ),
            @ApiResponse(
                responseCode = "404", 
                description = "Product not found",
                content = @Content(mediaType = "application/json",
                    schema = @Schema(implementation = ErrorResponse.class))
            )
        }
    )
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(
        @Parameter(description = "Product ID", required = true)
        @PathVariable Long id
    ) {
        Product product = productService.getProductById(id);
        return new ResponseEntity<>(product, HttpStatus.OK);
    }
    
    // Other methods with similarly detailed documentation
}
```

### Accessing Documentation

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`

## Versioning Strategies

API versioning ensures backward compatibility as APIs evolve.

### 1. URI Path Versioning

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 {
    // V1 implementation
}

@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 {
    // V2 implementation with changes
}
```

### 2. Request Parameter Versioning

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @GetMapping(params = "version=1")
    public ResponseEntity<ProductV1> getProductV1(@RequestParam Long id) {
        // Return v1 format
    }
    
    @GetMapping(params = "version=2")
    public ResponseEntity<ProductV2> getProductV2(@RequestParam Long id) {
        // Return v2 format
    }
}
```

### 3. Header Versioning

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @GetMapping(headers = "X-API-VERSION=1")
    public ResponseEntity<ProductV1> getProductV1(@RequestParam Long id) {
        // Return v1 format
    }
    
    @GetMapping(headers = "X-API-VERSION=2")
    public ResponseEntity<ProductV2> getProductV2(@RequestParam Long id) {
        // Return v2 format
    }
}
```

### 4. Media Type Versioning (Content Negotiation)

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @GetMapping(
        produces = "application/vnd.company.app-v1+json"
    )
    public ResponseEntity<ProductV1> getProductV1(@RequestParam Long id) {
        // Return v1 format
    }
    
    @GetMapping(
        produces = "application/vnd.company.app-v2+json"
    )
    public ResponseEntity<ProductV2> getProductV2(@RequestParam Long id) {
        // Return v2 format
    }
}
```

## Rate Limiting and API Security

### Rate Limiting with Bucket4j

Add dependencies:

```xml
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>4.10.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

Implement rate limiting:

```java
@Configuration
public class RateLimitingConfig {
    
    @Bean
    public Bucket createBucket() {
        long capacity = 20; // Maximum number of tokens
        long refillTokens = 10; // Tokens to refill
        long refillDuration = 1; // Duration in minutes
        
        Refill refill = Refill.greedy(refillTokens, Duration.ofMinutes(refillDuration));
        Bandwidth limit = Bandwidth.classic(capacity, refill);
        
        return Bucket4j.builder().addLimit(limit).build();
    }
}

@Component
public class RateLimitingInterceptor implements HandlerInterceptor {
    
    private final Bucket bucket;
    
    public RateLimitingInterceptor(Bucket bucket) {
        this.bucket = bucket;
    }
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (bucket.tryConsume(1)) {
            return true;
        }
        
        response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.getWriter().write("{\"message\":\"Rate limit exceeded. Try again later.\"}");
        return false;
    }
}

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    
    private final RateLimitingInterceptor rateLimitingInterceptor;
    
    public WebMvcConfig(RateLimitingInterceptor rateLimitingInterceptor) {
        this.rateLimitingInterceptor = rateLimitingInterceptor;
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(rateLimitingInterceptor)
                .addPathPatterns("/api/**");
    }
}
```

### API Key Authentication

```java
@Configuration
public class ApiKeyAuthFilter extends OncePerRequestFilter {
    
    private static final String API_KEY_HEADER = "X-API-KEY";
    private static final String API_KEY = "your-api-key"; // Store securely in configuration
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        
        String apiKey = request.getHeader(API_KEY_HEADER);
        
        if (API_KEY.equals(apiKey)) {
            filterChain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.getWriter().write("{\"message\":\"Invalid or missing API key\"}");
        }
    }
}

@Configuration
public class SecurityConfig {
    
    @Bean
    public FilterRegistrationBean<ApiKeyAuthFilter> apiKeyFilterRegistration() {
        FilterRegistrationBean<ApiKeyAuthFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new ApiKeyAuthFilter());
        registrationBean.addUrlPatterns("/api/*");
        return registrationBean;
    }
}
```

## Testing REST APIs

### Unit Testing Controllers

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private ProductService productService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void testGetProductById() throws Exception {
        // Given
        long productId = 1L;
        Product product = new Product();
        product.setId(productId);
        product.setName("Test Product");
        product.setPrice(new BigDecimal("99.99"));
        
        when(productService.getProductById(productId)).thenReturn(product);
        
        // When & Then
        mockMvc.perform(get("/api/products/{id}", productId))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id").value(productId))
                .andExpect(jsonPath("$.name").value("Test Product"))
                .andExpect(jsonPath("$.price").value(99.99));
        
        verify(productService).getProductById(productId);
    }
    
    @Test
    void testCreateProduct() throws Exception {
        // Given
        Product product = new Product();
        product.setName("New Product");
        product.setDescription("New Description");
        product.setPrice(new BigDecimal("129.99"));
        product.setQuantity(10);
        
        Product savedProduct = new Product();
        savedProduct.setId(1L);
        savedProduct.setName(product.getName());
        savedProduct.setDescription(product.getDescription());
        savedProduct.setPrice(product.getPrice());
        savedProduct.setQuantity(product.getQuantity());
        
        when(productService.createProduct(any(Product.class))).thenReturn(savedProduct);
        
        // When & Then
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(product)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("New Product"));
        
        verify(productService).createProduct(any(Product.class));
    }
}
```

### Integration Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProductIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private ProductRepository productRepository;
    
    @BeforeEach
    void setup() {
        productRepository.deleteAll();
    }
    
    @Test
    void testCreateAndGetProduct() {
        // Create a product
        Product product = new Product();
        product.setName("Integration Test Product");
        product.setDescription("Test Description");
        product.setPrice(new BigDecimal("199.99"));
        product.setQuantity(5);
        
        ResponseEntity<Product> createResponse = restTemplate.postForEntity(
                "/api/products",
                product,
                Product.class);
        
        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        assertNotNull(createResponse.getBody());
        assertNotNull(createResponse.getBody().getId());
        
        // Get the created product
        Long productId = createResponse.getBody().getId();
        ResponseEntity<Product> getResponse = restTemplate.getForEntity(
                "/api/products/{id}",
                Product.class,
                productId);
        
        assertEquals(HttpStatus.OK, getResponse.getStatusCode());
        assertEquals("Integration Test Product", getResponse.getBody().getName());
        assertEquals(0, new BigDecimal("199.99").compareTo(getResponse.getBody().getPrice()));
    }
}
```

## Practical Case Study

Let's build a complete e-commerce API with the following components:

### Domain Models

```java
// Product.java, Category.java, User.java, Order.java, OrderItem.java, etc.
```

### Data Access Layer

```java
// ProductRepository.java, CategoryRepository.java, UserRepository.java, etc.
```

### Service Layer

```java
// ProductService.java, OrderService.java, UserService.java, etc.
```

### Controller Layer

```java
// ProductController.java, CategoryController.java, OrderController.java, etc.
```

### Security Configuration

```java
// SecurityConfig.java
```

### API Documentation

```java
// OpenApiConfig.java
```

### Error Handling

```java
// GlobalExceptionHandler.java
```

### Test Suite

```java
// Unit tests and integration tests
```

This complete API would demonstrate:
- RESTful resource design
- Proper HTTP method usage
- Response status code usage
- Input validation
- Exception handling
- API documentation
- Security
- Performance considerations
- Testing strategies

## Conclusion

Building robust REST APIs with Spring Boot involves understanding REST principles, HTTP methods, status codes, content negotiation, exception handling, validation, documentation, and security. Spring Boot's powerful tools and abstractions make it easy to create well-designed APIs that follow best practices.

When designing your APIs, always consider:
- API consistency
- Backward compatibility
- Security
- Performance
- Documentation
- Testing

By following the patterns and examples in this guide, you'll be well-equipped to design and implement professional-grade REST APIs using Spring Boot.