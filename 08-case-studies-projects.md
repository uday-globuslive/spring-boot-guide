# Spring Boot Case Studies and Projects Guide

This guide provides comprehensive case studies and step-by-step project implementations for various types of Spring Boot applications. Each case study includes detailed explanations, architecture diagrams, code examples, and implementation guidance.

## Table of Contents

- [E-Commerce Application](#e-commerce-application)
- [Banking System](#banking-system)
- [Content Management System](#content-management-system)
- [Real-time Chat Application](#real-time-chat-application)
- [Task Management System](#task-management-system)
- [Microservices Architecture](#microservices-architecture)

## E-Commerce Application

### Overview

This case study explores building a complete e-commerce platform using Spring Boot, covering product catalog management, shopping cart functionality, order processing, and payment integration.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│    Frontend     │      │   API Gateway   │      │  Product Service│
│    (React.js)   │◄────►│  (Spring Cloud) │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │  Order Service  │      │  User Service   │
                         │  (Spring Boot)  │◄────►│  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │ Payment Service │      │ Inventory Svc   │
                         │  (Spring Boot)  │      │  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
```

### Key Components

1. **Product Service**: Manages product catalog and inventory
2. **Order Service**: Handles order creation and processing
3. **User Service**: Manages customer accounts and authentication
4. **Payment Service**: Integrates with payment gateways
5. **API Gateway**: Routes requests to appropriate microservices

### Implementation Steps

#### 1. Project Setup

Create a new Spring Boot project with the required dependencies:

```bash
mkdir e-commerce-app
cd e-commerce-app
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=product-service product-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=order-service order-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=user-service user-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=payment-service payment-service
spring init --dependencies=cloud-gateway,eureka-client --name=api-gateway api-gateway
spring init --dependencies=eureka-server --name=discovery-server discovery-server
```

#### 2. Product Service Implementation

```java
// product-service/src/main/java/com/example/model/Product.java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String name;
    
    private String description;
    
    @Positive
    private BigDecimal price;
    
    @Min(0)
    private Integer stockQuantity;
    
    @Enumerated(EnumType.STRING)
    private ProductCategory category;
    
    private String imageUrl;
    
    // Getters and setters provided by Lombok
}

// product-service/src/main/java/com/example/controller/ProductController.java
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
        return ResponseEntity.ok(productService.findAll());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(productService.save(product));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id, 
            @Valid @RequestBody Product product) {
        return productService.findById(id)
            .map(existingProduct -> {
                BeanUtils.copyProperties(product, existingProduct, "id");
                return ResponseEntity.ok(productService.save(existingProduct));
            })
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        return productService.findById(id)
            .map(product -> {
                productService.delete(product);
                return ResponseEntity.noContent().<Void>build();
            })
            .orElse(ResponseEntity.notFound().build());
    }
}
```

#### 3. Order Service Implementation

```java
// order-service/src/main/java/com/example/model/Order.java
@Entity
@Table(name = "orders")
@Data
@NoArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long userId;
    
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "order")
    private List<OrderItem> items = new ArrayList<>();
    
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    private String shippingAddress;
    
    private String paymentId;
}

// order-service/src/main/java/com/example/service/OrderService.java
@Service
@Transactional
public class OrderService {
    private final OrderRepository orderRepository;
    private final ProductServiceClient productServiceClient;
    private final PaymentServiceClient paymentServiceClient;
    
    @Autowired
    public OrderService(
            OrderRepository orderRepository,
            ProductServiceClient productServiceClient,
            PaymentServiceClient paymentServiceClient) {
        this.orderRepository = orderRepository;
        this.productServiceClient = productServiceClient;
        this.paymentServiceClient = paymentServiceClient;
    }
    
    public Order createOrder(OrderRequest orderRequest) {
        // Validate product availability
        for (OrderItemRequest item : orderRequest.getItems()) {
            Product product = productServiceClient.getProduct(item.getProductId());
            if (product.getStockQuantity() < item.getQuantity()) {
                throw new InsufficientStockException(
                    "Not enough stock for product: " + product.getName());
            }
        }
        
        // Create order
        Order order = new Order();
        order.setUserId(orderRequest.getUserId());
        order.setStatus(OrderStatus.PENDING);
        order.setShippingAddress(orderRequest.getShippingAddress());
        
        // Add order items
        BigDecimal totalAmount = BigDecimal.ZERO;
        List<OrderItem> orderItems = new ArrayList<>();
        
        for (OrderItemRequest item : orderRequest.getItems()) {
            Product product = productServiceClient.getProduct(item.getProductId());
            
            OrderItem orderItem = new OrderItem();
            orderItem.setOrder(order);
            orderItem.setProductId(product.getId());
            orderItem.setProductName(product.getName());
            orderItem.setQuantity(item.getQuantity());
            orderItem.setUnitPrice(product.getPrice());
            orderItem.setSubtotal(
                product.getPrice().multiply(new BigDecimal(item.getQuantity())));
            
            orderItems.add(orderItem);
            totalAmount = totalAmount.add(orderItem.getSubtotal());
        }
        
        order.setItems(orderItems);
        order.setTotalAmount(totalAmount);
        order.setCreatedAt(LocalDateTime.now());
        
        // Process payment
        PaymentRequest paymentRequest = new PaymentRequest();
        paymentRequest.setOrderId(order.getId());
        paymentRequest.setAmount(totalAmount);
        paymentRequest.setPaymentMethod(orderRequest.getPaymentMethod());
        
        PaymentResponse paymentResponse = paymentServiceClient.processPayment(paymentRequest);
        order.setPaymentId(paymentResponse.getPaymentId());
        
        if (paymentResponse.isSuccessful()) {
            order.setStatus(OrderStatus.CONFIRMED);
        } else {
            order.setStatus(OrderStatus.PAYMENT_FAILED);
        }
        
        // Update inventory
        for (OrderItemRequest item : orderRequest.getItems()) {
            productServiceClient.updateStock(
                item.getProductId(), item.getQuantity());
        }
        
        return orderRepository.save(order);
    }
    
    // Other methods
}
```

## Banking System

### Overview

This case study demonstrates building a secure banking application using Spring Boot, covering account management, transactions, security features, and reporting.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│    Frontend     │      │  API Gateway    │      │ Account Service │
│    (Angular)    │◄────►│  (Spring Cloud) │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │Transaction Svc  │      │ Customer Service│
                         │  (Spring Boot)  │◄────►│  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │ Reporting Svc   │      │  Auth Service   │
                         │  (Spring Boot)  │      │  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
```

### Key Components

1. **Customer Service**: Manages customer profiles and authentication
2. **Account Service**: Handles account creation and management
3. **Transaction Service**: Processes financial transactions
4. **Reporting Service**: Generates statements and reports
5. **Auth Service**: Implements OAuth2 security

### Implementation Steps

#### 1. Project Setup

```bash
mkdir banking-system
cd banking-system
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=customer-service customer-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=account-service account-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=transaction-service transaction-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=reporting-service reporting-service
spring init --dependencies=web,security,oauth2-resource-server --name=auth-service auth-service
spring init --dependencies=cloud-gateway,eureka-client --name=api-gateway api-gateway
spring init --dependencies=eureka-server --name=discovery-server discovery-server
```

#### 2. Customer Service Implementation

```java
// customer-service/src/main/java/com/example/model/Customer.java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String firstName;
    
    @NotBlank
    private String lastName;
    
    @Email
    @NotBlank
    @Column(unique = true)
    private String email;
    
    @NotBlank
    private String phoneNumber;
    
    @JsonIgnore
    private String password;
    
    @OneToOne(cascade = CascadeType.ALL)
    private Address address;
    
    @Temporal(TemporalType.DATE)
    private Date dateOfBirth;
    
    @Enumerated(EnumType.STRING)
    private CustomerStatus status;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}

// customer-service/src/main/java/com/example/service/CustomerService.java
@Service
@Transactional
public class CustomerService {
    private final CustomerRepository customerRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Autowired
    public CustomerService(
            CustomerRepository customerRepository,
            PasswordEncoder passwordEncoder) {
        this.customerRepository = customerRepository;
        this.passwordEncoder = passwordEncoder;
    }
    
    public Customer registerCustomer(CustomerRegistrationRequest request) {
        if (customerRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException("Email already in use");
        }
        
        Customer customer = new Customer();
        customer.setFirstName(request.getFirstName());
        customer.setLastName(request.getLastName());
        customer.setEmail(request.getEmail());
        customer.setPhoneNumber(request.getPhoneNumber());
        customer.setPassword(passwordEncoder.encode(request.getPassword()));
        customer.setDateOfBirth(request.getDateOfBirth());
        customer.setStatus(CustomerStatus.PENDING_VERIFICATION);
        customer.setCreatedAt(LocalDateTime.now());
        
        Address address = new Address();
        address.setStreet(request.getStreet());
        address.setCity(request.getCity());
        address.setState(request.getState());
        address.setZipCode(request.getZipCode());
        address.setCountry(request.getCountry());
        
        customer.setAddress(address);
        
        return customerRepository.save(customer);
    }
    
    // Other methods
}
```

#### 3. Account Service Implementation

```java
// account-service/src/main/java/com/example/model/Account.java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String accountNumber;
    
    private Long customerId;
    
    @Enumerated(EnumType.STRING)
    private AccountType type;
    
    private BigDecimal balance;
    
    @Enumerated(EnumType.STRING)
    private AccountStatus status;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    private BigDecimal dailyTransactionLimit;
    
    private Currency currency;
}

// account-service/src/main/java/com/example/service/AccountService.java
@Service
@Transactional
public class AccountService {
    private final AccountRepository accountRepository;
    private final CustomerServiceClient customerServiceClient;
    
    @Autowired
    public AccountService(
            AccountRepository accountRepository,
            CustomerServiceClient customerServiceClient) {
        this.accountRepository = accountRepository;
        this.customerServiceClient = customerServiceClient;
    }
    
    public Account createAccount(CreateAccountRequest request) {
        // Verify customer exists
        CustomerResponse customerResponse = 
            customerServiceClient.getCustomer(request.getCustomerId());
        
        if (CustomerStatus.ACTIVE != customerResponse.getStatus()) {
            throw new CustomerNotActiveException(
                "Customer must be active to create an account");
        }
        
        Account account = new Account();
        account.setCustomerId(request.getCustomerId());
        account.setType(request.getAccountType());
        account.setAccountNumber(generateAccountNumber());
        account.setBalance(BigDecimal.ZERO);
        account.setStatus(AccountStatus.ACTIVE);
        account.setCurrency(request.getCurrency());
        account.setDailyTransactionLimit(
            getDailyLimitForAccountType(request.getAccountType()));
        account.setCreatedAt(LocalDateTime.now());
        
        return accountRepository.save(account);
    }
    
    private String generateAccountNumber() {
        // Generate unique account number logic
        return "ACCT" + System.currentTimeMillis();
    }
    
    private BigDecimal getDailyLimitForAccountType(AccountType type) {
        switch (type) {
            case SAVINGS:
                return new BigDecimal("1000.00");
            case CHECKING:
                return new BigDecimal("2000.00");
            case BUSINESS:
                return new BigDecimal("10000.00");
            default:
                return new BigDecimal("1000.00");
        }
    }
    
    // Other methods
}
```

## Content Management System

### Overview

This case study demonstrates building a content management system (CMS) using Spring Boot, covering content creation, management, versioning, and publishing workflows.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│    Frontend     │      │  API Gateway    │      │ Content Service │
│    (Vue.js)     │◄────►│  (Spring Cloud) │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │ User Service    │      │  Media Service  │
                         │  (Spring Boot)  │◄────►│  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │Workflow Service │      │ Search Service  │
                         │  (Spring Boot)  │      │  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
```

### Key Components

1. **Content Service**: Manages content items and their metadata
2. **Media Service**: Handles file uploads and media asset management
3. **User Service**: Manages users, roles, and permissions
4. **Workflow Service**: Implements content approval workflows
5. **Search Service**: Provides full-text search capabilities

### Implementation Steps

#### 1. Project Setup

```bash
mkdir cms-application
cd cms-application
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=content-service content-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=media-service media-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=user-service user-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=workflow-service workflow-service
spring init --dependencies=web,data-elasticsearch --name=search-service search-service
spring init --dependencies=cloud-gateway,eureka-client --name=api-gateway api-gateway
spring init --dependencies=eureka-server --name=discovery-server discovery-server
```

#### 2. Content Service Implementation

```java
// content-service/src/main/java/com/example/model/Content.java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Content {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String title;
    
    @Lob
    private String body;
    
    @Enumerated(EnumType.STRING)
    private ContentType type;
    
    @Enumerated(EnumType.STRING)
    private ContentStatus status;
    
    private Long authorId;
    
    private String slug;
    
    @OneToMany(cascade = CascadeType.ALL)
    private List<ContentVersion> versions = new ArrayList<>();
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    private LocalDateTime publishedAt;
    
    @ManyToMany
    private Set<Tag> tags = new HashSet<>();
    
    @ManyToOne
    private Category category;
    
    private Boolean featured = false;
}

// content-service/src/main/java/com/example/service/ContentService.java
@Service
@Transactional
public class ContentService {
    private final ContentRepository contentRepository;
    private final UserServiceClient userServiceClient;
    private final WorkflowServiceClient workflowServiceClient;
    private final SearchServiceClient searchServiceClient;
    
    @Autowired
    public ContentService(
            ContentRepository contentRepository,
            UserServiceClient userServiceClient,
            WorkflowServiceClient workflowServiceClient,
            SearchServiceClient searchServiceClient) {
        this.contentRepository = contentRepository;
        this.userServiceClient = userServiceClient;
        this.workflowServiceClient = workflowServiceClient;
        this.searchServiceClient = searchServiceClient;
    }
    
    public Content createDraft(ContentCreationRequest request) {
        // Verify user exists and has permission
        UserResponse userResponse = userServiceClient.getUser(request.getAuthorId());
        
        if (!userServiceClient.hasPermission(
                request.getAuthorId(), "CREATE_CONTENT")) {
            throw new AccessDeniedException("User does not have permission to create content");
        }
        
        Content content = new Content();
        content.setTitle(request.getTitle());
        content.setBody(request.getBody());
        content.setType(request.getContentType());
        content.setStatus(ContentStatus.DRAFT);
        content.setAuthorId(request.getAuthorId());
        content.setSlug(generateSlug(request.getTitle()));
        content.setCreatedAt(LocalDateTime.now());
        
        if (request.getCategoryId() != null) {
            Category category = categoryRepository.findById(request.getCategoryId())
                .orElseThrow(() -> new EntityNotFoundException("Category not found"));
            content.setCategory(category);
        }
        
        if (request.getTagIds() != null && !request.getTagIds().isEmpty()) {
            Set<Tag> tags = tagRepository.findAllById(request.getTagIds())
                .stream()
                .collect(Collectors.toSet());
            content.setTags(tags);
        }
        
        // Create initial version
        ContentVersion version = new ContentVersion();
        version.setVersionNumber(1);
        version.setContent(content);
        version.setTitle(content.getTitle());
        version.setBody(content.getBody());
        version.setCreatedAt(LocalDateTime.now());
        version.setCreatedBy(request.getAuthorId());
        
        content.getVersions().add(version);
        
        Content savedContent = contentRepository.save(content);
        
        // Start workflow if needed
        if (request.isSubmitForApproval()) {
            workflowServiceClient.startApprovalWorkflow(
                new WorkflowRequest(
                    savedContent.getId(), 
                    WorkflowType.CONTENT_APPROVAL,
                    request.getAuthorId()
                )
            );
            savedContent.setStatus(ContentStatus.PENDING_APPROVAL);
            savedContent = contentRepository.save(savedContent);
        }
        
        return savedContent;
    }
    
    private String generateSlug(String title) {
        // Convert to lowercase and replace spaces with hyphens
        String slug = title.toLowerCase()
            .replaceAll("[^a-z0-9\\s-]", "")
            .replaceAll("\\s+", "-");
        
        // Check if slug exists and append number if needed
        String baseSlug = slug;
        int count = 1;
        
        while (contentRepository.existsBySlug(slug)) {
            slug = baseSlug + "-" + count;
            count++;
        }
        
        return slug;
    }
    
    // Other methods
}
```

## Real-time Chat Application

### Overview

This case study demonstrates building a real-time chat application using Spring Boot and WebSockets, covering user authentication, messaging, notifications, and persistence.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│    Frontend     │      │  WebSocket Svc  │      │  User Service   │
│    (React.js)   │◄────►│  (Spring Boot)  │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │ Message Service │      │   Auth Service  │
                         │  (Spring Boot)  │◄────►│  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
                                 ▲                         
                                 │                         
                                 ▼                         
                         ┌─────────────────┐      
                         │Notification Svc │      
                         │  (Spring Boot)  │      
                         └─────────────────┘      
```

### Key Components

1. **WebSocket Service**: Handles real-time communication
2. **User Service**: Manages user profiles and presence status
3. **Message Service**: Stores and retrieves message history
4. **Auth Service**: Handles authentication and authorization
5. **Notification Service**: Manages notifications for offline users

### Implementation Steps

#### 1. Project Setup

```bash
mkdir chat-application
cd chat-application
spring init --dependencies=web,websocket,data-jpa,security,mysql,lombok --name=websocket-service websocket-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=user-service user-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=message-service message-service
spring init --dependencies=web,security,oauth2-resource-server --name=auth-service auth-service
spring init --dependencies=web,data-jpa,mysql,lombok --name=notification-service notification-service
```

#### 2. WebSocket Configuration

```java
// websocket-service/src/main/java/com/example/config/WebSocketConfig.java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");
        config.setApplicationDestinationPrefixes("/app");
        config.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }
    
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new UserInterceptor());
    }
}

// websocket-service/src/main/java/com/example/controller/ChatController.java
@Controller
public class ChatController {
    private final MessageService messageService;
    private final UserService userService;
    private final SimpMessagingTemplate messagingTemplate;
    
    @Autowired
    public ChatController(
            MessageService messageService,
            UserService userService,
            SimpMessagingTemplate messagingTemplate) {
        this.messageService = messageService;
        this.userService = userService;
        this.messagingTemplate = messagingTemplate;
    }
    
    @MessageMapping("/chat.sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage) {
        // Save the message
        MessageEntity savedMessage = messageService.saveMessage(
            chatMessage.getSenderId(),
            chatMessage.getRecipientId(),
            chatMessage.getContent(),
            chatMessage.getType()
        );
        
        // Convert to DTO
        ChatMessage message = new ChatMessage(
            savedMessage.getId(),
            savedMessage.getSenderId(),
            savedMessage.getRecipientId(),
            savedMessage.getContent(),
            savedMessage.getType(),
            savedMessage.getTimestamp()
        );
        
        // Check if recipient is online
        boolean isRecipientOnline = userService.isUserOnline(message.getRecipientId());
        
        if (isRecipientOnline) {
            // Send to specific user
            messagingTemplate.convertAndSendToUser(
                message.getRecipientId().toString(),
                "/queue/messages",
                message
            );
        } else {
            // Create notification for offline user
            notificationService.createMessageNotification(
                message.getRecipientId(),
                message.getSenderId(),
                message.getContent()
            );
        }
        
        // Send back to sender
        messagingTemplate.convertAndSendToUser(
            message.getSenderId().toString(),
            "/queue/messages",
            message
        );
    }
    
    @MessageMapping("/chat.addUser")
    public void addUser(@Payload UserStatusMessage userStatus,
                      SimpMessageHeaderAccessor headerAccessor) {
        // Add username in web socket session
        headerAccessor.getSessionAttributes().put("userId", userStatus.getUserId());
        
        // Update user status to online
        userService.updateUserStatus(userStatus.getUserId(), UserStatus.ONLINE);
        
        // Broadcast user joined message
        UserStatusMessage statusMessage = new UserStatusMessage(
            userStatus.getUserId(),
            UserStatus.ONLINE
        );
        
        messagingTemplate.convertAndSend("/topic/public", statusMessage);
    }
    
    // Other methods
}
```

## Task Management System

### Overview

This case study explores building a task management system using Spring Boot, covering project management, task assignment, tracking, and reporting.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│    Frontend     │      │  API Gateway    │      │ Project Service │
│   (React+Redux) │◄────►│  (Spring Cloud) │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │  Task Service   │      │ User Service    │
                         │  (Spring Boot)  │◄────►│  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
                                 ▲                         ▲
                                 │                         │
                                 ▼                         ▼
                         ┌─────────────────┐      ┌─────────────────┐
                         │ Comment Service │      │ Report Service  │
                         │  (Spring Boot)  │      │  (Spring Boot)  │
                         └─────────────────┘      └─────────────────┘
```

### Key Components

1. **Project Service**: Manages projects and team assignments
2. **Task Service**: Handles task creation, assignment, and tracking
3. **User Service**: Manages user accounts and permissions
4. **Comment Service**: Handles comments and discussions
5. **Report Service**: Generates reports and analytics

### Implementation Steps

#### 1. Project Setup

```bash
mkdir task-manager
cd task-manager
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=project-service project-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=task-service task-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=user-service user-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=comment-service comment-service
spring init --dependencies=web,data-jpa,security,mysql,lombok --name=report-service report-service
spring init --dependencies=cloud-gateway,eureka-client --name=api-gateway api-gateway
spring init --dependencies=eureka-server --name=discovery-server discovery-server
```

#### 2. Task Service Implementation

```java
// task-service/src/main/java/com/example/model/Task.java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String title;
    
    @Lob
    private String description;
    
    private Long projectId;
    
    private Long assigneeId;
    
    private Long reporterId;
    
    @Enumerated(EnumType.STRING)
    private TaskStatus status;
    
    @Enumerated(EnumType.STRING)
    private TaskPriority priority;
    
    private LocalDate dueDate;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    private Long estimatedHours;
    
    private Long actualHours;
    
    @OneToMany(cascade = CascadeType.ALL)
    private List<TaskHistory> history = new ArrayList<>();
}

// task-service/src/main/java/com/example/service/TaskService.java
@Service
@Transactional
public class TaskService {
    private final TaskRepository taskRepository;
    private final ProjectServiceClient projectServiceClient;
    private final UserServiceClient userServiceClient;
    
    @Autowired
    public TaskService(
            TaskRepository taskRepository,
            ProjectServiceClient projectServiceClient,
            UserServiceClient userServiceClient) {
        this.taskRepository = taskRepository;
        this.projectServiceClient = projectServiceClient;
        this.userServiceClient = userServiceClient;
    }
    
    public Task createTask(TaskCreationRequest request) {
        // Verify project exists
        ProjectResponse project = 
            projectServiceClient.getProject(request.getProjectId());
        
        // Verify assignee exists and is project member
        if (request.getAssigneeId() != null) {
            UserResponse assignee = 
                userServiceClient.getUser(request.getAssigneeId());
            
            boolean isMember = projectServiceClient.isUserProjectMember(
                request.getProjectId(), request.getAssigneeId());
                
            if (!isMember) {
                throw new InvalidAssigneeException(
                    "Assignee must be a member of the project");
            }
        }
        
        Task task = new Task();
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setProjectId(request.getProjectId());
        task.setAssigneeId(request.getAssigneeId());
        task.setReporterId(request.getReporterId());
        task.setStatus(TaskStatus.TO_DO);
        task.setPriority(request.getPriority());
        task.setDueDate(request.getDueDate());
        task.setEstimatedHours(request.getEstimatedHours());
        task.setCreatedAt(LocalDateTime.now());
        
        // Record initial history
        TaskHistory history = new TaskHistory();
        history.setField("status");
        history.setOldValue(null);
        history.setNewValue(TaskStatus.TO_DO.name());
        history.setChangedBy(request.getReporterId());
        history.setChangedAt(LocalDateTime.now());
        
        task.getHistory().add(history);
        
        return taskRepository.save(task);
    }
    
    public Task updateTaskStatus(Long taskId, TaskStatusUpdateRequest request) {
        Task task = taskRepository.findById(taskId)
            .orElseThrow(() -> new TaskNotFoundException("Task not found"));
        
        // Record history
        TaskHistory history = new TaskHistory();
        history.setField("status");
        history.setOldValue(task.getStatus().name());
        history.setNewValue(request.getStatus().name());
        history.setChangedBy(request.getUserId());
        history.setChangedAt(LocalDateTime.now());
        
        task.getHistory().add(history);
        
        // Update task
        task.setStatus(request.getStatus());
        task.setUpdatedAt(LocalDateTime.now());
        
        // If moved to IN_PROGRESS, record start time
        if (request.getStatus() == TaskStatus.IN_PROGRESS && 
            history.getOldValue() != TaskStatus.IN_PROGRESS.name()) {
            task.setStartDate(LocalDate.now());
        }
        
        // If moved to DONE, update actual hours
        if (request.getStatus() == TaskStatus.DONE) {
            task.setCompletionDate(LocalDate.now());
            task.setActualHours(request.getActualHours());
        }
        
        return taskRepository.save(task);
    }
    
    // Other methods
}
```

## Microservices Architecture

### Overview

This case study illustrates implementing a microservices architecture using Spring Boot and Spring Cloud, covering service discovery, API gateway, circuit breaker, and distributed tracing.

### Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  API Gateway    │      │Service Discovery│      │ Config Server   │
│  (Spring Cloud) │◄────►│  (Eureka)       │◄────►│  (Spring Cloud) │
└─────────────────┘      └─────────────────┘      └─────────────────┘
        ▲                         ▲                         ▲
        │                         │                         │
        ▼                         ▼                         ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  Service A      │      │  Service B      │      │  Service C      │
│  (Spring Boot)  │◄────►│  (Spring Boot)  │◄────►│  (Spring Boot)  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
        ▲                         ▲                         ▲
        │                         │                         │
        ▼                         ▼                         ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ Circuit Breaker │      │Distributed Trace│      │  Metrics Server │
│  (Resilience4j) │      │  (Zipkin)       │      │  (Prometheus)   │
└─────────────────┘      └─────────────────┘      └─────────────────┘
```

### Key Components

1. **API Gateway**: Routes requests to appropriate microservices
2. **Service Discovery**: Manages service registration and discovery
3. **Config Server**: Centralizes configuration management
4. **Circuit Breaker**: Prevents cascading failures
5. **Distributed Tracing**: Monitors request flow across services

### Implementation Steps

#### 1. Project Setup

```bash
mkdir microservices-demo
cd microservices-demo
spring init --dependencies=cloud-config-server --name=config-server config-server
spring init --dependencies=eureka-server --name=discovery-server discovery-server
spring init --dependencies=cloud-gateway,eureka-client --name=api-gateway api-gateway
spring init --dependencies=web,actuator,eureka-client --name=service-a service-a
spring init --dependencies=web,actuator,eureka-client --name=service-b service-b
spring init --dependencies=web,actuator,eureka-client --name=service-c service-c
```

#### 2. Service Discovery Configuration

```java
// discovery-server/src/main/java/com/example/DiscoveryServerApplication.java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}

// discovery-server/src/main/resources/application.yml
server:
  port: 8761

spring:
  application:
    name: discovery-server

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    wait-time-in-ms-when-sync-empty: 0
```

#### 3. API Gateway Configuration

```java
// api-gateway/src/main/java/com/example/ApiGatewayApplication.java
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}

// api-gateway/src/main/resources/application.yml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: service-a
          uri: lb://service-a
          predicates:
            - Path=/service-a/**
          filters:
            - StripPrefix=1
        - id: service-b
          uri: lb://service-b
          predicates:
            - Path=/service-b/**
          filters:
            - StripPrefix=1
        - id: service-c
          uri: lb://service-c
          predicates:
            - Path=/service-c/**
          filters:
            - StripPrefix=1

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

#### 4. Circuit Breaker Implementation

```java
// service-a/src/main/java/com/example/service/ServiceBClient.java
@Service
public class ServiceBClient {
    private final RestTemplate restTemplate;
    private final String serviceBUrl;
    
    @Autowired
    public ServiceBClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
        this.serviceBUrl = "http://service-b";
    }
    
    @CircuitBreaker(name = "serviceBCircuitBreaker", fallbackMethod = "serviceBFallback")
    public ResponseEntity<String> callServiceB() {
        return restTemplate.getForEntity(serviceBUrl + "/api/data", String.class);
    }
    
    public ResponseEntity<String> serviceBFallback(Exception e) {
        return ResponseEntity.ok("Fallback response from Service B");
    }
}

// service-a/src/main/java/com/example/config/ResilienceConfig.java
@Configuration
public class ResilienceConfig {
    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofMillis(1000))
            .permittedNumberOfCallsInHalfOpenState(2)
            .slidingWindowSize(10)
            .slidingWindowType(CircuitBreakerConfig.SlidingWindowType.COUNT_BASED)
            .build();
        
        return CircuitBreakerRegistry.of(circuitBreakerConfig);
    }
    
    @Bean
    public TimeLimiterRegistry timeLimiterRegistry() {
        TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
            .timeoutDuration(Duration.ofSeconds(4))
            .build();
        
        return TimeLimiterRegistry.of(timeLimiterConfig);
    }
}
```

#### 5. Distributed Tracing Configuration

```java
// service-a/build.gradle (or pom.xml)
// Add dependencies
implementation 'org.springframework.cloud:spring-cloud-starter-sleuth'
implementation 'org.springframework.cloud:spring-cloud-sleuth-zipkin'

// service-a/src/main/resources/application.yml
spring:
  application:
    name: service-a
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    base-url: http://localhost:9411/
```

---

This guide provides a comprehensive overview of various Spring Boot application types with detailed case studies and implementation instructions. Each case study includes architecture diagrams, code examples, and step-by-step guidance to help you build robust and scalable applications.

For more detailed information on specific topics, refer to the official Spring Boot documentation and related Spring projects documentation.