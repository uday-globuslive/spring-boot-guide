# Data Access with Spring

This guide provides a comprehensive overview of data access techniques in Spring and Spring Boot, covering everything from basic JDBC to advanced ORM solutions and reactive database access.

## Table of Contents

1. [Introduction to Data Access in Spring](#introduction-to-data-access-in-spring)
2. [Spring JDBC](#spring-jdbc)
3. [Spring Data JPA](#spring-data-jpa)
4. [Hibernate Integration](#hibernate-integration)
5. [Transaction Management](#transaction-management)
6. [Data Validation](#data-validation)
7. [NoSQL Databases](#nosql-databases)
8. [Spring Data MongoDB](#spring-data-mongodb)
9. [Spring Data Redis](#spring-data-redis)
10. [Spring Data Elasticsearch](#spring-data-elasticsearch)
11. [Spring Data REST](#spring-data-rest)
12. [Reactive Data Access](#reactive-data-access)
13. [Connection Pooling](#connection-pooling)
14. [Database Migrations](#database-migrations)
15. [Practical Case Study](#practical-case-study)

## Introduction to Data Access in Spring

Spring offers comprehensive support for data access technologies, making database operations more manageable and less error-prone.

### Key Data Access Components in Spring

1. **JDBC Support**: Simplifies JDBC operations and exception handling
2. **Transaction Management**: Provides consistent transaction management across different APIs
3. **ORM Integration**: Seamless integration with ORMs like Hibernate, JPA, etc.
4. **Spring Data**: High-level abstractions for various data access technologies

### Spring Data Access Principles

- **Template Pattern**: `JdbcTemplate`, `JpaTemplate`, etc., simplify common operations
- **Exception Translation**: Converts technology-specific exceptions to Spring's `DataAccessException` hierarchy
- **Resource Management**: Automatic handling of connections, sessions, etc.
- **Repository Abstraction**: Common interface for different data access technologies

## Spring JDBC

Spring JDBC simplifies database access by reducing boilerplate code and providing exception translation.

### JDBC Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<!-- Database driver (e.g., MySQL) -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>
```

### Database Configuration

In `application.properties` or `application.yml`:

```properties
# MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# H2 in-memory database
# spring.datasource.url=jdbc:h2:mem:testdb
# spring.datasource.driver-class-name=org.h2.Driver
# spring.datasource.username=sa
# spring.datasource.password=
# spring.h2.console.enabled=true
```

### Using JdbcTemplate

```java
@Service
public class ProductService {
    
    private final JdbcTemplate jdbcTemplate;
    
    @Autowired
    public ProductService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    // Query for a list of objects
    public List<Product> findAll() {
        return jdbcTemplate.query(
            "SELECT id, name, description, price, quantity FROM products",
            (rs, rowNum) -> {
                Product product = new Product();
                product.setId(rs.getLong("id"));
                product.setName(rs.getString("name"));
                product.setDescription(rs.getString("description"));
                product.setPrice(rs.getBigDecimal("price"));
                product.setQuantity(rs.getInt("quantity"));
                return product;
            }
        );
    }
    
    // Query for a single object
    public Optional<Product> findById(Long id) {
        try {
            return Optional.of(jdbcTemplate.queryForObject(
                "SELECT id, name, description, price, quantity FROM products WHERE id = ?",
                new Object[]{id},
                (rs, rowNum) -> {
                    Product product = new Product();
                    product.setId(rs.getLong("id"));
                    product.setName(rs.getString("name"));
                    product.setDescription(rs.getString("description"));
                    product.setPrice(rs.getBigDecimal("price"));
                    product.setQuantity(rs.getInt("quantity"));
                    return product;
                }
            ));
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
    
    // Insert a new record
    public Product create(Product product) {
        KeyHolder keyHolder = new GeneratedKeyHolder();
        
        jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection.prepareStatement(
                "INSERT INTO products (name, description, price, quantity) VALUES (?, ?, ?, ?)",
                Statement.RETURN_GENERATED_KEYS
            );
            ps.setString(1, product.getName());
            ps.setString(2, product.getDescription());
            ps.setBigDecimal(3, product.getPrice());
            ps.setInt(4, product.getQuantity());
            return ps;
        }, keyHolder);
        
        product.setId(keyHolder.getKey().longValue());
        return product;
    }
    
    // Update an existing record
    public void update(Product product) {
        jdbcTemplate.update(
            "UPDATE products SET name = ?, description = ?, price = ?, quantity = ? WHERE id = ?",
            product.getName(),
            product.getDescription(),
            product.getPrice(),
            product.getQuantity(),
            product.getId()
        );
    }
    
    // Delete a record
    public void delete(Long id) {
        jdbcTemplate.update("DELETE FROM products WHERE id = ?", id);
    }
    
    // Count records
    public int count() {
        return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM products", Integer.class);
    }
}
```

### Using NamedParameterJdbcTemplate

```java
@Service
public class OrderService {
    
    private final NamedParameterJdbcTemplate jdbcTemplate;
    
    @Autowired
    public OrderService(NamedParameterJdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public List<Order> findByCustomerId(Long customerId) {
        Map<String, Object> params = new HashMap<>();
        params.put("customerId", customerId);
        
        return jdbcTemplate.query(
            "SELECT id, customer_id, order_date, status, total FROM orders WHERE customer_id = :customerId",
            params,
            (rs, rowNum) -> {
                Order order = new Order();
                order.setId(rs.getLong("id"));
                order.setCustomerId(rs.getLong("customer_id"));
                order.setOrderDate(rs.getTimestamp("order_date").toLocalDateTime());
                order.setStatus(rs.getString("status"));
                order.setTotal(rs.getBigDecimal("total"));
                return order;
            }
        );
    }
    
    public void batchUpdate(List<Order> orders) {
        String sql = "UPDATE orders SET status = :status WHERE id = :id";
        
        SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(orders.toArray());
        jdbcTemplate.batchUpdate(sql, batch);
    }
}
```

### Simplifying with RowMapper

```java
// Define a reusable RowMapper
public class ProductRowMapper implements RowMapper<Product> {
    @Override
    public Product mapRow(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getLong("id"));
        product.setName(rs.getString("name"));
        product.setDescription(rs.getString("description"));
        product.setPrice(rs.getBigDecimal("price"));
        product.setQuantity(rs.getInt("quantity"));
        return product;
    }
}

// Use the RowMapper in queries
@Service
public class ProductService {
    
    private final JdbcTemplate jdbcTemplate;
    private final ProductRowMapper productRowMapper = new ProductRowMapper();
    
    @Autowired
    public ProductService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public List<Product> findAll() {
        return jdbcTemplate.query(
            "SELECT id, name, description, price, quantity FROM products",
            productRowMapper
        );
    }
    
    public Optional<Product> findById(Long id) {
        try {
            return Optional.of(jdbcTemplate.queryForObject(
                "SELECT id, name, description, price, quantity FROM products WHERE id = ?",
                new Object[]{id},
                productRowMapper
            ));
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
}
```

## Spring Data JPA

Spring Data JPA simplifies JPA-based data access by reducing boilerplate code and providing repository interfaces.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Database driver (e.g., MySQL) -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>
```

### JPA Configuration

```properties
# JPA/Hibernate properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

### Entity Class

```java
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(length = 500)
    private String description;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;
    
    @Column
    private Integer quantity;
    
    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
    
    // Constructors, getters, and setters
}

@Entity
@Table(name = "categories")
public class Category {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String name;
    
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Product> products = new ArrayList<>();
    
    // Constructors, getters, and setters
}
```

### Repository Interface

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // Find products by name containing the given string (case-insensitive)
    List<Product> findByNameContainingIgnoreCase(String name);
    
    // Find products with price less than the given value
    List<Product> findByPriceLessThan(BigDecimal price);
    
    // Find products by category name
    List<Product> findByCategoryName(String categoryName);
    
    // Find products with stock (quantity) less than the given value
    List<Product> findByQuantityLessThan(Integer quantity);
    
    // Find products created after the given date
    List<Product> findByCreatedAtAfter(LocalDateTime date);
    
    // Custom query using JPQL
    @Query("SELECT p FROM Product p WHERE p.price BETWEEN :minPrice AND :maxPrice")
    List<Product> findProductsByPriceRange(@Param("minPrice") BigDecimal minPrice, 
                                          @Param("maxPrice") BigDecimal maxPrice);
    
    // Custom native SQL query
    @Query(value = "SELECT * FROM products p WHERE p.quantity > :minQuantity ORDER BY p.name", 
           nativeQuery = true)
    List<Product> findProductsInStock(@Param("minQuantity") Integer minQuantity);
    
    // Update query
    @Modifying
    @Transactional
    @Query("UPDATE Product p SET p.quantity = :quantity WHERE p.id = :id")
    void updateProductQuantity(@Param("id") Long id, @Param("quantity") Integer quantity);
    
    // Pagination example
    Page<Product> findByPriceGreaterThan(BigDecimal price, Pageable pageable);
}
```

### Service Layer

```java
@Service
@Transactional(readOnly = true)
public class ProductService {
    
    private final ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public List<Product> findAllProducts() {
        return productRepository.findAll();
    }
    
    public Optional<Product> findProductById(Long id) {
        return productRepository.findById(id);
    }
    
    public List<Product> findProductsByName(String name) {
        return productRepository.findByNameContainingIgnoreCase(name);
    }
    
    public List<Product> findProductsByPriceRange(BigDecimal minPrice, BigDecimal maxPrice) {
        return productRepository.findProductsByPriceRange(minPrice, maxPrice);
    }
    
    public Page<Product> findProductsByPriceWithPagination(BigDecimal price, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        return productRepository.findByPriceGreaterThan(price, pageable);
    }
    
    @Transactional
    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }
    
    @Transactional
    public void updateProductQuantity(Long id, Integer quantity) {
        productRepository.updateProductQuantity(id, quantity);
    }
    
    @Transactional
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
}
```

### Using Specifications

```java
// Entity with JPA Static Metamodel
// Requires annotation processor for metamodel generation
@Entity
@Table(name = "products")
public class Product {
    // Same as above
}

// Specifications for dynamic queries
public class ProductSpecifications {
    
    public static Specification<Product> nameContains(String name) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.like(
                criteriaBuilder.lower(root.get("name")), 
                "%" + name.toLowerCase() + "%"
            );
    }
    
    public static Specification<Product> priceBetween(BigDecimal minPrice, BigDecimal maxPrice) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.between(
                root.get("price"), 
                minPrice, 
                maxPrice
            );
    }
    
    public static Specification<Product> inCategory(String categoryName) {
        return (root, query, criteriaBuilder) -> {
            Join<Product, Category> categoryJoin = root.join("category");
            return criteriaBuilder.equal(categoryJoin.get("name"), categoryName);
        };
    }
    
    public static Specification<Product> inStock() {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.greaterThan(root.get("quantity"), 0);
    }
}

// Repository with Specification support
public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
    // Other methods as above
}

// Service using specifications
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public List<Product> findProductsByCriteria(String name, BigDecimal minPrice, BigDecimal maxPrice, 
                                               String categoryName, Boolean inStock) {
        
        Specification<Product> spec = Specification.where(null);
        
        if (name != null) {
            spec = spec.and(ProductSpecifications.nameContains(name));
        }
        
        if (minPrice != null && maxPrice != null) {
            spec = spec.and(ProductSpecifications.priceBetween(minPrice, maxPrice));
        }
        
        if (categoryName != null) {
            spec = spec.and(ProductSpecifications.inCategory(categoryName));
        }
        
        if (inStock != null && inStock) {
            spec = spec.and(ProductSpecifications.inStock());
        }
        
        return productRepository.findAll(spec);
    }
}
```

## Hibernate Integration

Spring Boot seamlessly integrates with Hibernate as the default JPA provider.

### Advanced Hibernate Features

#### Custom UserType

```java
// Define a custom type for handling JSON data
public class JsonType implements UserType {
    
    @Override
    public int[] sqlTypes() {
        return new int[] { Types.LONGVARCHAR };
    }
    
    @Override
    public Class returnedClass() {
        return Map.class;
    }
    
    @Override
    public boolean equals(Object x, Object y) {
        return Objects.equals(x, y);
    }
    
    @Override
    public int hashCode(Object x) {
        return Objects.hashCode(x);
    }
    
    @Override
    public Object nullSafeGet(ResultSet rs, String[] names, SharedSessionContractImplementor session, Object owner)
            throws SQLException {
        String json = rs.getString(names[0]);
        if (rs.wasNull() || json == null) {
            return new HashMap<String, Object>();
        }
        
        try {
            return new ObjectMapper().readValue(json, Map.class);
        } catch (Exception e) {
            throw new HibernateException("Error deserializing JSON", e);
        }
    }
    
    @Override
    public void nullSafeSet(PreparedStatement st, Object value, int index, SharedSessionContractImplementor session)
            throws SQLException {
        if (value == null) {
            st.setNull(index, Types.LONGVARCHAR);
            return;
        }
        
        try {
            String json = new ObjectMapper().writeValueAsString(value);
            st.setString(index, json);
        } catch (Exception e) {
            throw new HibernateException("Error serializing JSON", e);
        }
    }
    
    @Override
    public Object deepCopy(Object value) {
        if (value == null) {
            return null;
        }
        
        try {
            return new ObjectMapper().readValue(
                new ObjectMapper().writeValueAsString(value),
                Map.class
            );
        } catch (Exception e) {
            throw new HibernateException("Error cloning JSON", e);
        }
    }
    
    @Override
    public boolean isMutable() {
        return true;
    }
    
    @Override
    public Serializable disassemble(Object value) {
        try {
            return value == null ? null : new ObjectMapper().writeValueAsString(value);
        } catch (Exception e) {
            throw new HibernateException("Error serializing JSON", e);
        }
    }
    
    @Override
    public Object assemble(Serializable cached, Object owner) {
        try {
            return cached == null ? null : new ObjectMapper().readValue((String) cached, Map.class);
        } catch (Exception e) {
            throw new HibernateException("Error deserializing JSON", e);
        }
    }
    
    @Override
    public Object replace(Object original, Object target, Object owner) {
        return deepCopy(original);
    }
}

// Using the custom type in an entity
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Type(type = "com.example.types.JsonType")
    @Column(name = "attributes", columnDefinition = "json")
    private Map<String, Object> attributes = new HashMap<>();
    
    // Other fields, getters, and setters
}
```

#### Second-Level Cache

```properties
# Enable second-level cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.use_query_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider
```

```java
@Entity
@Table(name = "products")
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    // Entity fields and methods
}

@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final EntityManager entityManager;
    
    @Autowired
    public ProductService(ProductRepository productRepository, EntityManager entityManager) {
        this.productRepository = productRepository;
        this.entityManager = entityManager;
    }
    
    @Transactional(readOnly = true)
    public List<Product> findProductsByPriceRangeCached(BigDecimal minPrice, BigDecimal maxPrice) {
        Session session = entityManager.unwrap(Session.class);
        Query<Product> query = session.createQuery(
            "FROM Product p WHERE p.price BETWEEN :minPrice AND :maxPrice",
            Product.class
        );
        query.setParameter("minPrice", minPrice);
        query.setParameter("maxPrice", maxPrice);
        query.setCacheable(true);
        query.setCacheRegion("product.price.range");
        
        return query.getResultList();
    }
}
```

#### Custom Naming Strategy

```java
@Configuration
public class HibernateConfig {
    
    @Bean
    public PhysicalNamingStrategy physicalNamingStrategy() {
        return new CustomPhysicalNamingStrategy();
    }
}

public class CustomPhysicalNamingStrategy implements PhysicalNamingStrategy {
    
    @Override
    public Identifier toPhysicalCatalogName(Identifier name, JdbcEnvironment jdbcEnvironment) {
        return apply(name, jdbcEnvironment);
    }
    
    @Override
    public Identifier toPhysicalSchemaName(Identifier name, JdbcEnvironment jdbcEnvironment) {
        return apply(name, jdbcEnvironment);
    }
    
    @Override
    public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment jdbcEnvironment) {
        return apply(name, jdbcEnvironment);
    }
    
    @Override
    public Identifier toPhysicalSequenceName(Identifier name, JdbcEnvironment jdbcEnvironment) {
        return apply(name, jdbcEnvironment);
    }
    
    @Override
    public Identifier toPhysicalColumnName(Identifier name, JdbcEnvironment jdbcEnvironment) {
        return apply(name, jdbcEnvironment);
    }
    
    private Identifier apply(Identifier name, JdbcEnvironment jdbcEnvironment) {
        if (name == null) {
            return null;
        }
        
        String text = name.getText();
        // Convert camel case to snake case
        String snakeCase = text.replaceAll("([a-z])([A-Z])", "$1_$2").toLowerCase();
        return Identifier.toIdentifier(snakeCase);
    }
}
```

## Transaction Management

Spring provides a consistent transaction management API across different transaction APIs.

### Declarative Transaction Management

```java
@Service
@Transactional // Class-level transaction settings
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    
    @Autowired
    public OrderService(OrderRepository orderRepository, ProductRepository productRepository) {
        this.orderRepository = orderRepository;
        this.productRepository = productRepository;
    }
    
    // Inherits class-level transaction settings
    public Order createOrder(Order order) {
        // Validate order
        validateOrder(order);
        
        // Update product inventory
        for (OrderItem item : order.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                                 .orElseThrow(() -> new RuntimeException("Product not found"));
            
            int newQuantity = product.getQuantity() - item.getQuantity();
            if (newQuantity < 0) {
                throw new RuntimeException("Insufficient inventory for product: " + product.getName());
            }
            
            product.setQuantity(newQuantity);
            productRepository.save(product);
        }
        
        // Save order
        return orderRepository.save(order);
    }
    
    @Transactional(readOnly = true) // Override class-level settings
    public List<Order> findOrdersByCustomerId(Long customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
    
    @Transactional(timeout = 30) // Set transaction timeout
    public void processLargeOrders() {
        // Process time-consuming operations
    }
    
    @Transactional(rollbackFor = {SQLException.class}, 
                   noRollbackFor = {DataNotFoundException.class})
    public void complexOperation() {
        // Operations with custom rollback rules
    }
    
    private void validateOrder(Order order) {
        // Validation logic
    }
}
```

### Transaction Propagation Behaviors

```java
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final AuditService auditService;
    
    @Autowired
    public ProductService(ProductRepository productRepository, AuditService auditService) {
        this.productRepository = productRepository;
        this.auditService = auditService;
    }
    
    @Transactional
    public Product updateProduct(Product product) {
        // This operation runs in the current transaction
        Product savedProduct = productRepository.save(product);
        
        // This operation requires a new transaction
        auditService.logProductUpdate(savedProduct);
        
        return savedProduct;
    }
}

@Service
public class AuditService {
    
    private final AuditRepository auditRepository;
    
    @Autowired
    public AuditService(AuditRepository auditRepository) {
        this.auditRepository = auditRepository;
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logProductUpdate(Product product) {
        AuditLog log = new AuditLog();
        log.setEntityType("Product");
        log.setEntityId(product.getId());
        log.setAction("UPDATE");
        log.setTimestamp(LocalDateTime.now());
        log.setDetails("Product updated: " + product.getName());
        
        auditRepository.save(log);
    }
}
```

### Programmatic Transaction Management

```java
@Service
public class ReportService {
    
    private final TransactionTemplate transactionTemplate;
    private final JdbcTemplate jdbcTemplate;
    
    @Autowired
    public ReportService(PlatformTransactionManager transactionManager, JdbcTemplate jdbcTemplate) {
        this.transactionTemplate = new TransactionTemplate(transactionManager);
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public void generateReport() {
        // Execute in a transaction
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                try {
                    // Step 1: Create temporary tables
                    jdbcTemplate.execute("CREATE TEMPORARY TABLE temp_report AS SELECT * FROM products WHERE quantity > 0");
                    
                    // Step 2: Process data
                    jdbcTemplate.update("UPDATE temp_report SET status = 'IN_STOCK'");
                    
                    // Step 3: Export to final report table
                    jdbcTemplate.update("INSERT INTO product_reports SELECT * FROM temp_report");
                } catch (Exception e) {
                    // Explicit rollback if needed
                    status.setRollbackOnly();
                    throw e;
                } finally {
                    // Clean up
                    jdbcTemplate.execute("DROP TEMPORARY TABLE IF EXISTS temp_report");
                }
            }
        });
    }
    
    public ReportSummary generateReportWithResult() {
        return transactionTemplate.execute(status -> {
            try {
                // Report generation logic
                
                // Return summary
                ReportSummary summary = new ReportSummary();
                summary.setTotalProducts(100);
                summary.setInStock(80);
                summary.setOutOfStock(20);
                return summary;
            } catch (Exception e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

## Data Validation

Spring provides robust validation support integrated with JPA entities.

### Bean Validation (JSR-380)

```java
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Product name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    @Column(nullable = false)
    private String name;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    @Column
    private String description;
    
    @NotNull(message = "Price is required")
    @Positive(message = "Price must be greater than zero")
    @Digits(integer = 8, fraction = 2, message = "Price can have up to 8 integer digits and 2 decimal places")
    @Column(nullable = false)
    private BigDecimal price;
    
    @Min(value = 0, message = "Quantity cannot be negative")
    @Column
    private Integer quantity;
    
    @NotNull(message = "Category is required")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;
    
    // Other fields, getters, and setters
}
```

### Validation in Service Layer

```java
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final Validator validator;
    
    @Autowired
    public ProductService(ProductRepository productRepository, Validator validator) {
        this.productRepository = productRepository;
        this.validator = validator;
    }
    
    @Transactional
    public Product saveProduct(Product product) {
        // Validate the product
        Set<ConstraintViolation<Product>> violations = validator.validate(product);
        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
        
        // Additional business validation
        if (product.getPrice().compareTo(BigDecimal.ZERO) == 0 && "Premium".equals(product.getCategory().getName())) {
            throw new IllegalArgumentException("Premium products cannot be free");
        }
        
        return productRepository.save(product);
    }
}
```

### Custom Validation Constraints

```java
// Custom constraint annotation
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueProductNameValidator.class)
public @interface UniqueProductName {
    String message() default "Product name already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
public class UniqueProductNameValidator implements ConstraintValidator<UniqueProductName, String> {
    
    private final ProductRepository productRepository;
    
    @Autowired
    public UniqueProductNameValidator(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    @Override
    public boolean isValid(String name, ConstraintValidatorContext context) {
        if (name == null) {
            return true; // Let @NotNull handle this
        }
        
        return !productRepository.existsByNameIgnoreCase(name);
    }
}

// Using the custom constraint
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Product name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    @UniqueProductName
    @Column(nullable = false)
    private String name;
    
    // Other fields, getters, and setters
}
```

## NoSQL Databases

Spring Data provides consistent support for various NoSQL databases.

### Common Spring Data Abstractions

1. **Repository Interfaces**: Common interfaces across datastores
2. **Template Classes**: Low-level data access operations
3. **Converters**: Map between domain types and store-specific types
4. **Mapping Annotations**: Define the mapping between objects and storage

## Spring Data MongoDB

MongoDB is a document-oriented NoSQL database supported by Spring Data.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### MongoDB Configuration

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/ecommerce
spring.data.mongodb.auto-index-creation=true
```

### MongoDB Document

```java
@Document(collection = "products")
public class Product {
    
    @Id
    private String id;
    
    @Indexed(unique = true)
    private String sku;
    
    @NotBlank
    @Indexed
    private String name;
    
    private String description;
    
    @NotNull
    private BigDecimal price;
    
    private Integer quantity;
    
    @DBRef
    private Category category;
    
    private Map<String, Object> attributes = new HashMap<>();
    
    @CreatedDate
    private Date createdAt;
    
    @LastModifiedDate
    private Date updatedAt;
    
    // Constructors, getters, and setters
}

@Document(collection = "categories")
public class Category {
    
    @Id
    private String id;
    
    @Indexed(unique = true)
    private String name;
    
    private String description;
    
    // Constructors, getters, and setters
}
```

### MongoDB Repository

```java
public interface ProductRepository extends MongoRepository<Product, String> {
    
    List<Product> findByNameContainingIgnoreCase(String name);
    
    List<Product> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
    
    List<Product> findByCategoryName(String categoryName);
    
    List<Product> findByQuantityGreaterThan(Integer minQuantity);
    
    @Query("{ 'attributes.color': ?0 }")
    List<Product> findByColor(String color);
    
    @Query("{ 'price': { $lt: ?0 }, 'quantity': { $gt: 0 } }")
    List<Product> findInStockOnSale(BigDecimal maxPrice);
    
    @Query(value = "{ 'category.name': ?0 }", count = true)
    long countByCategory(String categoryName);
    
    @Query(value = "{ 'quantity': 0 }", fields = "{ 'name': 1, 'sku': 1 }")
    List<Product> findOutOfStockProductNamesAndSkus();
    
    // Text search (requires text index on the collection)
    @Query("{ $text: { $search: ?0 } }")
    List<Product> searchProducts(String searchText);
}
```

### MongoDB Service

```java
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final MongoTemplate mongoTemplate;
    
    @Autowired
    public ProductService(ProductRepository productRepository, MongoTemplate mongoTemplate) {
        this.productRepository = productRepository;
        this.mongoTemplate = mongoTemplate;
    }
    
    public List<Product> findAllProducts() {
        return productRepository.findAll();
    }
    
    public Optional<Product> findProductById(String id) {
        return productRepository.findById(id);
    }
    
    public List<Product> findProductsByName(String name) {
        return productRepository.findByNameContainingIgnoreCase(name);
    }
    
    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }
    
    public void deleteProduct(String id) {
        productRepository.deleteById(id);
    }
    
    // Advanced query using MongoTemplate
    public List<Product> findProductsByAttributes(Map<String, Object> attributes) {
        Query query = new Query();
        
        attributes.forEach((key, value) -> {
            query.addCriteria(Criteria.where("attributes." + key).is(value));
        });
        
        return mongoTemplate.find(query, Product.class);
    }
    
    // Aggregation example
    public List<CategorySales> getProductSalesByCategory() {
        TypedAggregation<Product> aggregation = Aggregation.newAggregation(
            Product.class,
            Aggregation.match(Criteria.where("quantity").gt(0)),
            Aggregation.group("category.name")
                        .count().as("productCount")
                        .sum("price").as("totalValue"),
            Aggregation.project("productCount", "totalValue")
                        .and("_id").as("categoryName"),
            Aggregation.sort(Sort.Direction.DESC, "totalValue")
        );
        
        AggregationResults<CategorySales> results = mongoTemplate.aggregate(
            aggregation, CategorySales.class);
            
        return results.getMappedResults();
    }
    
    public class CategorySales {
        private String categoryName;
        private long productCount;
        private BigDecimal totalValue;
        
        // Getters and setters
    }
}
```

## Spring Data Redis

Redis is an in-memory data structure store that can be used as a database, cache, and message broker.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### Redis Configuration

```properties
spring.redis.host=localhost
spring.redis.port=6379
```

### Redis Entity

```java
@RedisHash("products")
public class Product implements Serializable {
    
    @Id
    private String id;
    
    private String name;
    
    private String description;
    
    private BigDecimal price;
    
    private Integer quantity;
    
    @Reference
    private Category category;
    
    @TimeToLive
    private Long ttl; // Time to live in seconds
    
    // Constructors, getters, and setters
}

@RedisHash("categories")
public class Category implements Serializable {
    
    @Id
    private String id;
    
    private String name;
    
    // Constructors, getters, and setters
}
```

### Redis Repository

```java
public interface ProductRedisRepository extends CrudRepository<Product, String> {
    
    List<Product> findByName(String name);
    
    List<Product> findByCategory_Name(String categoryName);
}
```

### Redis Service

```java
@Service
public class ProductCacheService {
    
    private final StringRedisTemplate stringRedisTemplate;
    private final RedisTemplate<String, Product> redisTemplate;
    private final ProductRedisRepository productRedisRepository;
    
    @Autowired
    public ProductCacheService(StringRedisTemplate stringRedisTemplate,
                              RedisTemplate<String, Product> redisTemplate,
                              ProductRedisRepository productRedisRepository) {
        this.stringRedisTemplate = stringRedisTemplate;
        this.redisTemplate = redisTemplate;
        this.productRedisRepository = productRedisRepository;
    }
    
    // Basic operations with RedisRepository
    public Product cacheProduct(Product product) {
        return productRedisRepository.save(product);
    }
    
    public Optional<Product> getProductFromCache(String id) {
        return productRedisRepository.findById(id);
    }
    
    public void removeProductFromCache(String id) {
        productRedisRepository.deleteById(id);
    }
    
    // Operations with RedisTemplate
    public void setProductInCache(String key, Product product) {
        redisTemplate.opsForValue().set(key, product);
    }
    
    public void setProductInCacheWithExpiry(String key, Product product, long timeout, TimeUnit unit) {
        redisTemplate.opsForValue().set(key, product, timeout, unit);
    }
    
    public Product getProductFromCacheByKey(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    
    // Hash operations
    public void addProductToHash(String hashKey, String itemKey, Product product) {
        redisTemplate.opsForHash().put(hashKey, itemKey, product);
    }
    
    public Map<Object, Object> getAllProductsFromHash(String hashKey) {
        return redisTemplate.opsForHash().entries(hashKey);
    }
    
    // List operations
    public void addProductToList(String listKey, Product product) {
        redisTemplate.opsForList().rightPush(listKey, product);
    }
    
    public List<Product> getAllProductsFromList(String listKey) {
        return redisTemplate.opsForList().range(listKey, 0, -1);
    }
    
    // Set operations
    public void addProductToSet(String setKey, Product product) {
        redisTemplate.opsForSet().add(setKey, product);
    }
    
    public Set<Product> getAllProductsFromSet(String setKey) {
        return redisTemplate.opsForSet().members(setKey);
    }
    
    // Sorted set operations
    public void addProductToSortedSet(String sortedSetKey, Product product, double score) {
        redisTemplate.opsForZSet().add(sortedSetKey, product, score);
    }
    
    public Set<Product> getProductsByScoreRange(String sortedSetKey, double min, double max) {
        return redisTemplate.opsForZSet().rangeByScore(sortedSetKey, min, max);
    }
}
```

## Spring Data Elasticsearch

Elasticsearch is a distributed, RESTful search and analytics engine.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

### Elasticsearch Configuration

```properties
spring.elasticsearch.rest.uris=http://localhost:9200
```

### Elasticsearch Document

```java
@Document(indexName = "products")
public class ProductDocument {
    
    @Id
    private String id;
    
    @Field(type = FieldType.Text, analyzer = "standard")
    private String name;
    
    @Field(type = FieldType.Text, analyzer = "standard")
    private String description;
    
    @Field(type = FieldType.Keyword)
    private String sku;
    
    @Field(type = FieldType.Float)
    private BigDecimal price;
    
    @Field(type = FieldType.Integer)
    private Integer quantity;
    
    @Field(type = FieldType.Keyword)
    private String category;
    
    @Field(type = FieldType.Object)
    private Map<String, Object> attributes = new HashMap<>();
    
    @Field(type = FieldType.Date)
    private Date createdAt;
    
    // Constructors, getters, and setters
}
```

### Elasticsearch Repository

```java
public interface ProductElasticsearchRepository extends ElasticsearchRepository<ProductDocument, String> {
    
    List<ProductDocument> findByName(String name);
    
    List<ProductDocument> findByNameContaining(String name);
    
    List<ProductDocument> findByCategory(String category);
    
    List<ProductDocument> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
    
    @Query("{\"bool\": {\"must\": [{\"match\": {\"name\": \"?0\"}}, {\"range\": {\"price\": {\"lte\": \"?1\"}}}]}}")
    List<ProductDocument> findByNameAndMaxPrice(String name, BigDecimal maxPrice);
}
```

### Elasticsearch Service

```java
@Service
public class ProductSearchService {
    
    private final ProductElasticsearchRepository productRepository;
    private final ElasticsearchOperations elasticsearchOperations;
    
    @Autowired
    public ProductSearchService(ProductElasticsearchRepository productRepository,
                              ElasticsearchOperations elasticsearchOperations) {
        this.productRepository = productRepository;
        this.elasticsearchOperations = elasticsearchOperations;
    }
    
    public ProductDocument indexProduct(ProductDocument product) {
        return productRepository.save(product);
    }
    
    public void deleteProductFromIndex(String id) {
        productRepository.deleteById(id);
    }
    
    public List<ProductDocument> findAllProducts() {
        Iterable<ProductDocument> products = productRepository.findAll();
        List<ProductDocument> productList = new ArrayList<>();
        products.forEach(productList::add);
        return productList;
    }
    
    public Optional<ProductDocument> findProductById(String id) {
        return productRepository.findById(id);
    }
    
    // Basic search
    public List<ProductDocument> searchProducts(String query) {
        Query searchQuery = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.multiMatchQuery(query, "name", "description"))
            .build();
        
        SearchHits<ProductDocument> searchHits = elasticsearchOperations.search(
            searchQuery, ProductDocument.class);
        
        List<ProductDocument> products = new ArrayList<>();
        searchHits.forEach(hit -> products.add(hit.getContent()));
        
        return products;
    }
    
    // Advanced search with filters and sorting
    public List<ProductDocument> searchProductsWithFilters(String query, String category, 
                                                         BigDecimal minPrice, BigDecimal maxPrice,
                                                         String sortField, String sortDirection) {
        // Build bool query
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        // Add query condition
        if (query != null && !query.isEmpty()) {
            boolQuery.must(QueryBuilders.multiMatchQuery(query, "name", "description"));
        }
        
        // Add filters
        if (category != null && !category.isEmpty()) {
            boolQuery.filter(QueryBuilders.termQuery("category", category));
        }
        
        if (minPrice != null || maxPrice != null) {
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
            if (minPrice != null) {
                rangeQuery.gte(minPrice.doubleValue());
            }
            if (maxPrice != null) {
                rangeQuery.lte(maxPrice.doubleValue());
            }
            boolQuery.filter(rangeQuery);
        }
        
        // Build search query
        NativeSearchQueryBuilder searchQueryBuilder = new NativeSearchQueryBuilder()
            .withQuery(boolQuery);
        
        // Add sorting
        if (sortField != null && !sortField.isEmpty()) {
            Sort.Direction direction = Sort.Direction.ASC;
            if (sortDirection != null && sortDirection.equalsIgnoreCase("desc")) {
                direction = Sort.Direction.DESC;
            }
            searchQueryBuilder.withSort(Sort.by(direction, sortField));
        }
        
        // Execute search
        Query searchQuery = searchQueryBuilder.build();
        SearchHits<ProductDocument> searchHits = elasticsearchOperations.search(
            searchQuery, ProductDocument.class);
        
        List<ProductDocument> products = new ArrayList<>();
        searchHits.forEach(hit -> products.add(hit.getContent()));
        
        return products;
    }
    
    // Aggregation example: count of products by category
    public Map<String, Long> getProductCountByCategory() {
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
            .addAggregation(AggregationBuilders.terms("categories").field("category"))
            .build();
        
        SearchHits<ProductDocument> searchHits = elasticsearchOperations.search(searchQuery, ProductDocument.class);
        Aggregations aggregations = searchHits.getAggregations();
        
        Map<String, Long> result = new HashMap<>();
        if (aggregations != null) {
            Terms categoryAgg = aggregations.get("categories");
            categoryAgg.getBuckets().forEach(bucket -> {
                result.put(bucket.getKeyAsString(), bucket.getDocCount());
            });
        }
        
        return result;
    }
}
```

## Spring Data REST

Spring Data REST automatically exposes Spring Data repositories as RESTful resources.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

### Configuration

```properties
spring.data.rest.base-path=/api
spring.data.rest.default-page-size=20
spring.data.rest.max-page-size=100
```

### Repository Exposure

```java
@RepositoryRestResource(path = "products")
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    @RestResource(path = "by-name", rel = "by-name")
    List<Product> findByNameContainingIgnoreCase(@Param("name") String name);
    
    @RestResource(path = "by-price-range", rel = "by-price-range")
    List<Product> findByPriceBetween(
        @Param("min") BigDecimal minPrice, 
        @Param("max") BigDecimal maxPrice
    );
    
    @RestResource(exported = false)
    void deleteById(Long id);
}

@RepositoryRestResource(path = "categories")
public interface CategoryRepository extends JpaRepository<Category, Long> {
    
    @RestResource(path = "by-name", rel = "by-name")
    Optional<Category> findByNameIgnoreCase(@Param("name") String name);
}
```

### Customizing the REST API

```java
@Configuration
public class RestConfig implements RepositoryRestConfigurer {
    
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config, CorsRegistry cors) {
        config.setBasePath("/api");
        
        // Expose entity IDs
        config.exposeIdsFor(Product.class, Category.class);
        
        // Configure CORS
        cors.addMapping("/**")
            .allowedOrigins("*")
            .allowedMethods("GET", "POST", "PUT", "DELETE");
        
        // Set default page size
        config.setDefaultPageSize(20);
        
        // Configure projection
        config.projectionConfiguration()
              .addProjection(ProductSummary.class);
    }
}
```

### Projections

```java
@Projection(name = "summary", types = { Product.class })
public interface ProductSummary {
    Long getId();
    String getName();
    BigDecimal getPrice();
    String getCategoryName();
    
    // Derived attribute
    @Value("#{target.price.multiply(0.1)}")
    BigDecimal getTax();
}
```

### Events

```java
@Component
@RepositoryEventHandler
public class ProductEventHandler {
    
    private final Logger logger = LoggerFactory.getLogger(ProductEventHandler.class);
    
    @HandleBeforeCreate
    public void handleProductBeforeCreate(Product product) {
        // Pre-create validation or modification
        if (product.getQuantity() == null) {
            product.setQuantity(0);
        }
        
        logger.info("Creating product: {}", product.getName());
    }
    
    @HandleAfterCreate
    public void handleProductAfterCreate(Product product) {
        logger.info("Product created: {}", product.getName());
    }
    
    @HandleBeforeSave
    public void handleProductBeforeSave(Product product) {
        // Pre-update validation or modification
        product.setUpdatedAt(LocalDateTime.now());
        
        logger.info("Updating product: {}", product.getName());
    }
    
    @HandleAfterSave
    public void handleProductAfterSave(Product product) {
        logger.info("Product updated: {}", product.getName());
    }
    
    @HandleBeforeDelete
    public void handleProductBeforeDelete(Product product) {
        logger.info("Deleting product: {}", product.getName());
    }
    
    @HandleAfterDelete
    public void handleProductAfterDelete(Product product) {
        logger.info("Product deleted: {}", product.getName());
    }
}
```

### Validators

```java
@Component
public class ProductValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return Product.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        Product product = (Product) target;
        
        if (product.getName() == null || product.getName().isEmpty()) {
            errors.rejectValue("name", "name.empty", "Product name is required");
        }
        
        if (product.getPrice() == null) {
            errors.rejectValue("price", "price.empty", "Product price is required");
        } else if (product.getPrice().compareTo(BigDecimal.ZERO) < 0) {
            errors.rejectValue("price", "price.negative", "Product price cannot be negative");
        }
    }
}

@Configuration
public class ValidatorConfig implements InitializingBean {
    
    private final ProductValidator productValidator;
    private final RepositoryRestConfiguration configuration;
    
    @Autowired
    public ValidatorConfig(ProductValidator productValidator, RepositoryRestConfiguration configuration) {
        this.productValidator = productValidator;
        this.configuration = configuration;
    }
    
    @Override
    public void afterPropertiesSet() {
        configuration.getValidatorList().add(productValidator);
    }
}
```

## Reactive Data Access

Spring Data offers reactive support for certain databases using Reactive Streams specifications.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### Configuration

```properties
spring.r2dbc.url=r2dbc:h2:mem:///testdb
spring.r2dbc.username=sa
spring.r2dbc.password=
```

### Reactive Entity

```java
@Table("products")
public class Product {
    
    @Id
    private Long id;
    
    private String name;
    
    private String description;
    
    private BigDecimal price;
    
    private Integer quantity;
    
    // Constructors, getters, and setters
}
```

### Reactive Repository

```java
public interface ReactiveProductRepository extends ReactiveCrudRepository<Product, Long> {
    
    Flux<Product> findByNameContainingIgnoreCase(String name);
    
    Flux<Product> findByPriceGreaterThanEqual(BigDecimal minPrice);
    
    Mono<Product> findByNameIgnoreCase(String name);
    
    @Query("SELECT * FROM products WHERE price BETWEEN :minPrice AND :maxPrice")
    Flux<Product> findByPriceRange(BigDecimal minPrice, BigDecimal maxPrice);
}
```

### Reactive Service

```java
@Service
public class ReactiveProductService {
    
    private final ReactiveProductRepository productRepository;
    private final DatabaseClient databaseClient;
    
    @Autowired
    public ReactiveProductService(ReactiveProductRepository productRepository, DatabaseClient databaseClient) {
        this.productRepository = productRepository;
        this.databaseClient = databaseClient;
    }
    
    public Flux<Product> findAllProducts() {
        return productRepository.findAll();
    }
    
    public Mono<Product> findProductById(Long id) {
        return productRepository.findById(id);
    }
    
    public Flux<Product> findProductsByName(String name) {
        return productRepository.findByNameContainingIgnoreCase(name);
    }
    
    public Mono<Product> saveProduct(Product product) {
        return productRepository.save(product);
    }
    
    public Mono<Void> deleteProduct(Long id) {
        return productRepository.deleteById(id);
    }
    
    // Using DatabaseClient for more control
    public Flux<Product> findProductsInStock() {
        return databaseClient.sql("SELECT * FROM products WHERE quantity > 0")
                            .map((row, metadata) -> {
                                Product product = new Product();
                                product.setId(row.get("id", Long.class));
                                product.setName(row.get("name", String.class));
                                product.setDescription(row.get("description", String.class));
                                product.setPrice(row.get("price", BigDecimal.class));
                                product.setQuantity(row.get("quantity", Integer.class));
                                return product;
                            })
                            .all();
    }
    
    // Transactional reactive operations
    @Transactional
    public Mono<Product> updateProductStock(Long id, Integer newQuantity) {
        return productRepository.findById(id)
                .switchIfEmpty(Mono.error(new RuntimeException("Product not found")))
                .flatMap(product -> {
                    product.setQuantity(newQuantity);
                    return productRepository.save(product);
                });
    }
}
```

### Reactive Controller

```java
@RestController
@RequestMapping("/api/reactive/products")
public class ReactiveProductController {
    
    private final ReactiveProductService productService;
    
    @Autowired
    public ReactiveProductController(ReactiveProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public Flux<Product> getAllProducts() {
        return productService.findAllProducts();
    }
    
    @GetMapping("/{id}")
    public Mono<ResponseEntity<Product>> getProductById(@PathVariable Long id) {
        return productService.findProductById(id)
                .map(product -> ResponseEntity.ok(product))
                .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/search")
    public Flux<Product> searchProducts(@RequestParam String name) {
        return productService.findProductsByName(name);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<Product> createProduct(@RequestBody Product product) {
        return productService.saveProduct(product);
    }
    
    @PutMapping("/{id}")
    public Mono<ResponseEntity<Product>> updateProduct(@PathVariable Long id, @RequestBody Product product) {
        return productService.findProductById(id)
                .flatMap(existingProduct -> {
                    existingProduct.setName(product.getName());
                    existingProduct.setDescription(product.getDescription());
                    existingProduct.setPrice(product.getPrice());
                    existingProduct.setQuantity(product.getQuantity());
                    return productService.saveProduct(existingProduct);
                })
                .map(updatedProduct -> ResponseEntity.ok(updatedProduct))
                .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteProduct(@PathVariable Long id) {
        return productService.deleteProduct(id);
    }
    
    @GetMapping("/in-stock")
    public Flux<Product> getProductsInStock() {
        return productService.findProductsInStock();
    }
    
    @PatchMapping("/{id}/stock")
    public Mono<ResponseEntity<Product>> updateProductStock(@PathVariable Long id, @RequestParam Integer quantity) {
        return productService.updateProductStock(id, quantity)
                .map(updatedProduct -> ResponseEntity.ok(updatedProduct))
                .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    // Server-Sent Events (SSE) example
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Product> streamProducts() {
        return productService.findAllProducts()
                .delayElements(Duration.ofSeconds(1));
    }
}
```

## Connection Pooling

Connection pooling is essential for efficient database access in production applications.

### HikariCP Configuration

HikariCP is the default connection pool in Spring Boot.

```properties
# HikariCP specific settings
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.max-lifetime=1800000
```

### Custom HikariCP Configuration

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariConfig hikariConfig() {
        return new HikariConfig();
    }
    
    @Bean
    public DataSource dataSource(HikariConfig hikariConfig) {
        return new HikariDataSource(hikariConfig);
    }
}
```

### Connection Pool Monitoring

```java
@Configuration
public class DataSourceMonitoringConfig {
    
    private final DataSource dataSource;
    
    @Autowired
    public DataSourceMonitoringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Bean
    public ConnectionPoolMetrics connectionPoolMetrics() {
        if (dataSource instanceof HikariDataSource) {
            return new HikariPoolMetrics((HikariDataSource) dataSource);
        }
        throw new IllegalStateException("Unsupported datasource type: " + dataSource.getClass().getName());
    }
    
    public static class HikariPoolMetrics {
        private final HikariDataSource hikariDataSource;
        
        public HikariPoolMetrics(HikariDataSource hikariDataSource) {
            this.hikariDataSource = hikariDataSource;
        }
        
        public HikariPoolMXBean getPoolMetrics() {
            return hikariDataSource.getHikariPoolMXBean();
        }
        
        @Scheduled(fixedDelay = 60000) // Run every minute
        public void logMetrics() {
            HikariPoolMXBean poolMetrics = getPoolMetrics();
            System.out.println("Connection Pool Metrics:");
            System.out.println("Active connections: " + poolMetrics.getActiveConnections());
            System.out.println("Idle connections: " + poolMetrics.getIdleConnections());
            System.out.println("Total connections: " + poolMetrics.getTotalConnections());
            System.out.println("Waiting threads: " + poolMetrics.getThreadsAwaitingConnection());
        }
    }
}
```

## Database Migrations

Database migrations help manage database schema changes across environments and deployments.

### Flyway Integration

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
spring.flyway.baseline-version=0
```

Migration scripts:
- `V1__create_tables.sql`
- `V2__add_indexes.sql`
- `V3__add_constraints.sql`

```sql
-- V1__create_tables.sql
CREATE TABLE categories (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);

CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    quantity INT DEFAULT 0,
    category_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- V2__add_indexes.sql
CREATE INDEX idx_product_name ON products(name);
CREATE INDEX idx_product_price ON products(price);
CREATE INDEX idx_product_category ON products(category_id);

-- V3__add_constraints.sql
ALTER TABLE products ADD CONSTRAINT check_price_positive CHECK (price >= 0);
ALTER TABLE products ADD CONSTRAINT check_quantity_non_negative CHECK (quantity >= 0);
```

### Liquibase Integration

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

```properties
spring.liquibase.enabled=true
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
```

```xml
<!-- db.changelog-master.xml -->
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.4.xsd">
                        
    <include file="classpath:db/changelog/changes/01-create-tables.xml"/>
    <include file="classpath:db/changelog/changes/02-add-indexes.xml"/>
    <include file="classpath:db/changelog/changes/03-add-constraints.xml"/>
    
</databaseChangeLog>

<!-- 01-create-tables.xml -->
<changeSet id="01" author="developer">
    <createTable tableName="categories">
        <column name="id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true" nullable="false"/>
        </column>
        <column name="name" type="VARCHAR(100)">
            <constraints nullable="false" unique="true"/>
        </column>
        <column name="description" type="TEXT"/>
    </createTable>
    
    <createTable tableName="products">
        <column name="id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true" nullable="false"/>
        </column>
        <column name="name" type="VARCHAR(100)">
            <constraints nullable="false"/>
        </column>
        <column name="description" type="TEXT"/>
        <column name="price" type="DECIMAL(10, 2)">
            <constraints nullable="false"/>
        </column>
        <column name="quantity" type="INT" defaultValue="0"/>
        <column name="category_id" type="BIGINT">
            <constraints foreignKeyName="fk_product_category"
                          references="categories(id)"/>
        </column>
        <column name="created_at" type="TIMESTAMP" defaultValueComputed="CURRENT_TIMESTAMP"/>
        <column name="updated_at" type="TIMESTAMP" defaultValueComputed="CURRENT_TIMESTAMP"/>
    </createTable>
</changeSet>
```

## Practical Case Study

Let's build a complete e-commerce data access layer with:

- Product catalog (JPA)
- User management (JPA)
- Shopping cart (Redis)
- Order processing (JPA with transactions)
- Product search (Elasticsearch)
- Cache management (Redis)
- Data change tracking and auditing

### Domain Model

```java
// Product entity, Category entity, User entity, Order entity, etc.
```

### Repository Layer

```java
// ProductRepository, CategoryRepository, UserRepository, etc.
```

### Service Layer

```java
// ProductService, UserService, OrderService, etc.
```

### Cache Configuration

```java
// Redis cache configuration for product catalog
```

### Search Configuration

```java
// Elasticsearch integration for product search
```

### Audit Configuration

```java
// JPA auditing for data change tracking
```

### Transaction Management

```java
// Cross-repository transaction management for order processing
```

## Conclusion

This guide has covered a wide range of data access techniques in Spring and Spring Boot, from simple JDBC operations to complex NoSQL and reactive data access. Key takeaways include:

1. **Choose the Right Tools**: Spring offers various data access technologies, each with its own strengths and weaknesses.
2. **Use Spring Data**: It provides a consistent programming model across different data stores.
3. **Implement Proper Transaction Management**: Ensure data consistency with proper transaction boundaries.
4. **Consider Performance**: Use appropriate caching, connection pooling, and indexing strategies.
5. **Plan for Evolution**: Implement database migration tools to manage schema changes.
6. **Test Thoroughly**: Data access code requires comprehensive testing.

By following the patterns and examples in this guide, you'll be well-equipped to implement robust and efficient data access in your Spring applications.