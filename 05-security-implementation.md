# Security Implementation with Spring

This guide provides a comprehensive overview of implementing security in Spring applications, covering everything from basic authentication to advanced security patterns and best practices.

## Table of Contents

1. [Introduction to Spring Security](#introduction-to-spring-security)
2. [Spring Security Architecture](#spring-security-architecture)
3. [Basic Authentication](#basic-authentication)
4. [Form-based Authentication](#form-based-authentication)
5. [JWT Authentication](#jwt-authentication)
6. [OAuth2 and OpenID Connect](#oauth2-and-openid-connect)
7. [Authorization](#authorization)
8. [Method Security](#method-security)
9. [CSRF Protection](#csrf-protection)
10. [CORS Configuration](#cors-configuration)
11. [Password Management](#password-management)
12. [Remember Me Authentication](#remember-me-authentication)
13. [Session Management](#session-management)
14. [Security Headers](#security-headers)
15. [Securing REST APIs](#securing-rest-apis)
16. [Testing Security](#testing-security)
17. [Practical Case Study](#practical-case-study)

## Introduction to Spring Security

Spring Security is a powerful and highly customizable framework for authentication and access control. It's the de-facto standard for securing Spring-based applications.

### Key Features

- **Comprehensive Security Model**: Authentication, authorization, and protection against common attacks
- **Flexible Configuration**: XML or Java-based configuration options
- **Integration**: Seamless integration with Spring MVC, Spring Boot, and other Spring projects
- **Multi-tenancy Support**: Can handle multiple authentication sources
- **Protection Against Attacks**: CSRF, session fixation, clickjacking, etc.

### Spring Security Core Concepts

1. **Authentication**: Verifying the identity of a user or system
2. **Authorization**: Determining if an authenticated entity has access to a resource
3. **Principal**: Currently authenticated user
4. **Granted Authority**: Permission granted to the principal
5. **Security Context**: Holds authentication information
6. **Security Filter Chain**: Series of filters that process the security request

## Spring Security Architecture

Spring Security is built around a chain of filters that process HTTP requests.

### Core Components

1. **SecurityContextHolder**: Stores the security context
2. **SecurityContext**: Holds the Authentication object
3. **Authentication**: Represents the authenticated principal
4. **GrantedAuthority**: Permissions granted to the principal
5. **UserDetails**: Core user information
6. **UserDetailsService**: Service to retrieve UserDetails

### Filter Chain

![Spring Security Filter Chain](https://docs.spring.io/spring-security/site/docs/current/reference/html5/images/servlet/architecture/filterchain.png)

Key filters in the chain:
- **SecurityContextPersistenceFilter**: Restores/persists SecurityContext
- **UsernamePasswordAuthenticationFilter**: Processes form-based authentication
- **BasicAuthenticationFilter**: Processes HTTP Basic authentication
- **ExceptionTranslationFilter**: Handles security exceptions
- **FilterSecurityInterceptor**: Makes access control decisions

## Basic Authentication

HTTP Basic Authentication is a simple authentication mechanism.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Basic Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user")
            .password("{noop}password") // {noop} for plain text passwords
            .roles("USER")
            .and()
            .withUser("admin")
            .password("{noop}admin")
            .roles("USER", "ADMIN");
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/", "/home").permitAll() // Public access
            .antMatchers("/admin/**").hasRole("ADMIN") // Admin only
            .anyRequest().authenticated() // Everything else requires authentication
            .and()
            .httpBasic(); // Enable HTTP Basic
    }
}
```

### Custom User Details Service

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword()) // Assuming password is already encoded
            .roles(user.getRoles().toArray(new String[0]))
            .build();
    }
}
```

### Password Encoding

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
    
    // Other configuration
}
```

## Form-based Authentication

Form-based authentication provides a customizable login form.

### Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home", "/css/**", "/js/**", "/images/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login") // Custom login page
                .permitAll()
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .and()
            .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .permitAll();
    }
}
```

### Custom Login Page

```html
<!-- login.html (Thymeleaf template) -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login Page</title>
</head>
<body>
    <h1>Login</h1>
    <div th:if="${param.error}">
        <div class="alert alert-danger">Invalid username or password.</div>
    </div>
    <div th:if="${param.logout}">
        <div class="alert alert-info">You have been logged out.</div>
    </div>
    <form th:action="@{/login}" method="post">
        <div>
            <label>Username:</label>
            <input type="text" name="username"/>
        </div>
        <div>
            <label>Password:</label>
            <input type="password" name="password"/>
        </div>
        <div>
            <input type="submit" value="Log in"/>
        </div>
    </form>
</body>
</html>
```

### Login Controller

```java
@Controller
public class LoginController {
    
    @GetMapping("/login")
    public String login() {
        return "login";
    }
    
    @GetMapping("/dashboard")
    public String dashboard() {
        return "dashboard";
    }
}
```

## JWT Authentication

JSON Web Tokens (JWT) are a popular mechanism for secure, stateless authentication.

### Dependencies

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
```

### JWT Utility Class

```java
@Component
public class JwtTokenUtil {
    
    private final String secret;
    private final long expirationTime;
    
    public JwtTokenUtil(@Value("${jwt.secret}") String secret,
                        @Value("${jwt.expiration}") long expirationTime) {
        this.secret = secret;
        this.expirationTime = expirationTime;
    }
    
    // Generate token for user
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        
        // Add custom claims if needed
        // claims.put("roles", userDetails.getAuthorities().stream()
        //     .map(GrantedAuthority::getAuthority)
        //     .collect(Collectors.toList()));
        
        return createToken(claims, userDetails.getUsername());
    }
    
    private String createToken(Map<String, Object> claims, String subject) {
        Date now = new Date();
        Date validity = new Date(now.getTime() + expirationTime);
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(now)
            .setExpiration(validity)
            .signWith(Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8)))
            .compact();
    }
    
    // Validate token
    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    // Extract username from token
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    // Extract expiration date from token
    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8)))
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
}
```

### JWT Authentication Filter

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtTokenUtil jwtTokenUtil;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(JwtTokenUtil jwtTokenUtil, UserDetailsService userDetailsService) {
        this.jwtTokenUtil = jwtTokenUtil;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        
        final String authorizationHeader = request.getHeader("Authorization");
        
        String username = null;
        String jwt = null;
        
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            jwt = authorizationHeader.substring(7);
            try {
                username = jwtTokenUtil.extractUsername(jwt);
            } catch (Exception e) {
                // Token is invalid, do nothing here
            }
        }
        
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
            
            if (jwtTokenUtil.validateToken(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

### Authentication Controller

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final UserDetailsService userDetailsService;
    private final JwtTokenUtil jwtTokenUtil;
    
    public AuthController(AuthenticationManager authenticationManager,
                          UserDetailsService userDetailsService,
                          JwtTokenUtil jwtTokenUtil) {
        this.authenticationManager = authenticationManager;
        this.userDetailsService = userDetailsService;
        this.jwtTokenUtil = jwtTokenUtil;
    }
    
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest loginRequest) {
        try {
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    loginRequest.getUsername(), loginRequest.getPassword()
                )
            );
        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(new ErrorResponse("Invalid username or password"));
        }
        
        final UserDetails userDetails = userDetailsService
            .loadUserByUsername(loginRequest.getUsername());
        
        final String token = jwtTokenUtil.generateToken(userDetails);
        
        return ResponseEntity.ok(new JwtResponse(token));
    }
    
    // Request and response classes
    @Data
    public static class LoginRequest {
        private String username;
        private String password;
    }
    
    @Data
    @AllArgsConstructor
    public static class JwtResponse {
        private String token;
    }
    
    @Data
    @AllArgsConstructor
    public static class ErrorResponse {
        private String message;
    }
}
```

### Security Configuration for JWT

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final UserDetailsService userDetailsService;
    private final JwtTokenUtil jwtTokenUtil;
    
    @Autowired
    public SecurityConfig(UserDetailsService userDetailsService, JwtTokenUtil jwtTokenUtil) {
        this.userDetailsService = userDetailsService;
        this.jwtTokenUtil = jwtTokenUtil;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // Disable CSRF for REST APIs
            .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll() // Public auth endpoints
                .antMatchers("/api/admin/**").hasRole("ADMIN") // Admin only
                .anyRequest().authenticated()
                .and()
            // No form login for JWT authentication
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS); // No sessions
        
        // Add JWT filter
        http.addFilterBefore(
            new JwtAuthenticationFilter(jwtTokenUtil, userDetailsService),
            UsernamePasswordAuthenticationFilter.class
        );
    }
}
```

### Application Properties

```properties
# JWT Configuration
jwt.secret=your-256-bit-secret-key-here-should-be-very-long-and-secure
jwt.expiration=86400000  # 24 hours in milliseconds
```

## OAuth2 and OpenID Connect

OAuth2 is an authorization framework that enables third-party applications to access resources on behalf of users. OpenID Connect is an identity layer on top of OAuth2.

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

### OAuth2 Login Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/login**", "/error**").permitAll()
                .anyRequest().authenticated()
                .and()
            .oauth2Login() // Enable OAuth2 login
                .loginPage("/login") // Custom login page
                .defaultSuccessUrl("/dashboard")
                .and()
            .logout()
                .logoutSuccessUrl("/")
                .permitAll();
    }
}
```

### Application Properties for OAuth2 Login

```properties
# Google OAuth2 configuration
spring.security.oauth2.client.registration.google.client-id=your-client-id
spring.security.oauth2.client.registration.google.client-secret=your-client-secret
spring.security.oauth2.client.registration.google.scope=openid,profile,email

# GitHub OAuth2 configuration
spring.security.oauth2.client.registration.github.client-id=your-client-id
spring.security.oauth2.client.registration.github.client-secret=your-client-secret
spring.security.oauth2.client.registration.github.scope=user:email,read:user
```

### Custom OAuth2 User Service

```java
@Service
public class CustomOAuth2UserService extends DefaultOAuth2UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public CustomOAuth2UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(userRequest);
        
        // Extract provider (google, github, etc.)
        String provider = userRequest.getClientRegistration().getRegistrationId();
        
        // Extract user info based on provider
        Map<String, Object> attributes = oAuth2User.getAttributes();
        String email = extractEmail(attributes, provider);
        
        // Check if user exists in DB, if not create new user
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> createUser(email, attributes, provider));
        
        // Update user information on each login if needed
        updateUser(user, attributes, provider);
        
        // Return OAuth2User with additional custom authorities if needed
        return new DefaultOAuth2User(
            AuthorityUtils.createAuthorityList("ROLE_" + user.getRole()),
            attributes,
            userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName()
        );
    }
    
    private String extractEmail(Map<String, Object> attributes, String provider) {
        switch (provider) {
            case "google":
                return (String) attributes.get("email");
            case "github":
                return (String) attributes.get("email");
            // Handle other providers
            default:
                throw new OAuth2AuthenticationException(
                    new OAuth2Error("provider_not_supported", "Provider not supported", null)
                );
        }
    }
    
    private User createUser(String email, Map<String, Object> attributes, String provider) {
        User user = new User();
        user.setEmail(email);
        user.setProvider(provider);
        user.setProviderId(extractProviderId(attributes, provider));
        user.setName(extractName(attributes, provider));
        user.setRole("USER"); // Default role
        return userRepository.save(user);
    }
    
    private void updateUser(User user, Map<String, Object> attributes, String provider) {
        // Update information if needed
        userRepository.save(user);
    }
    
    private String extractProviderId(Map<String, Object> attributes, String provider) {
        switch (provider) {
            case "google":
                return (String) attributes.get("sub");
            case "github":
                return String.valueOf(attributes.get("id"));
            default:
                return null;
        }
    }
    
    private String extractName(Map<String, Object> attributes, String provider) {
        switch (provider) {
            case "google":
                return (String) attributes.get("name");
            case "github":
                return (String) attributes.get("name");
            default:
                return null;
        }
    }
}
```

### OAuth2 Resource Server Configuration

```java
@Configuration
@EnableWebSecurity
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .cors().and()
            .csrf().disable()
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/api/user/**").hasRole("USER")
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .oauth2ResourceServer()
                .jwt();
    }
    
    @Bean
    JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("https://your-auth-server/.well-known/jwks.json").build();
    }
}
```

### Application Properties for Resource Server

```properties
# Resource server configuration
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://your-auth-server/issuer
```

## Authorization

Authorization determines whether an authenticated user has access to specific resources or operations.

### Role-Based Access Control

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .antMatchers("/user/**").hasRole("USER")
                .antMatchers("/manager/**").hasRole("MANAGER")
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/api/users/**").hasRole("ADMIN")
                .antMatchers(HttpMethod.POST, "/api/products").hasRole("ADMIN")
                .antMatchers(HttpMethod.PUT, "/api/products/**").hasRole("ADMIN")
                .antMatchers(HttpMethod.DELETE, "/api/products/**").hasRole("ADMIN")
                .antMatchers(HttpMethod.GET, "/api/products/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login").permitAll();
    }
}
```

### Method-Level Security with @PreAuthorize

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    // Method security configuration
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        // Admin only
        return userRepository.findAll();
    }
    
    @PreAuthorize("hasRole('ADMIN') or #username == authentication.principal.username")
    public User getUserByUsername(String username) {
        // Admin or the user themselves
        return userRepository.findByUsername(username);
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // Admin only
        userRepository.deleteById(userId);
    }
    
    @PreAuthorize("hasPermission(#user, 'edit')")
    public User updateUser(User user) {
        // Custom permission check
        return userRepository.save(user);
    }
}
```

### Custom Permission Evaluator

```java
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    
    private final UserRepository userRepository;
    
    @Autowired
    public CustomPermissionEvaluator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null || targetDomainObject == null || !(permission instanceof String)) {
            return false;
        }
        
        String username = authentication.getName();
        
        if (targetDomainObject instanceof User) {
            User user = (User) targetDomainObject;
            
            switch ((String) permission) {
                case "edit":
                    // Allow if user is editing themselves or is an admin
                    return user.getUsername().equals(username) || 
                           authentication.getAuthorities().stream()
                               .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
                case "view":
                    // Allow if user is viewing themselves, or is a manager/admin
                    return user.getUsername().equals(username) || 
                           authentication.getAuthorities().stream()
                               .anyMatch(a -> a.getAuthority().equals("ROLE_MANAGER") || 
                                              a.getAuthority().equals("ROLE_ADMIN"));
                default:
                    return false;
            }
        }
        
        return false;
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        if (authentication == null || targetId == null || targetType == null || !(permission instanceof String)) {
            return false;
        }
        
        if (targetType.equals("User")) {
            User user = userRepository.findById((Long) targetId).orElse(null);
            if (user == null) {
                return false;
            }
            return hasPermission(authentication, user, permission);
        }
        
        return false;
    }
}

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    
    @Autowired
    private CustomPermissionEvaluator customPermissionEvaluator;
    
    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(customPermissionEvaluator);
        return expressionHandler;
    }
}
```

## Method Security

Method security allows you to apply security rules directly to service or controller methods.

### Enabling Method Security

```java
@Configuration
@EnableGlobalMethodSecurity(
    prePostEnabled = true,  // For @PreAuthorize, @PostAuthorize
    securedEnabled = true,  // For @Secured
    jsr250Enabled = true    // For @RolesAllowed
)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    // Method security configuration
}
```

### Using @Secured Annotation

```java
@Service
public class ProductService {
    
    @Secured({"ROLE_ADMIN", "ROLE_MANAGER"})
    public List<Product> getAllProducts() {
        // Only accessible to admin and manager roles
        return productRepository.findAll();
    }
}
```

### Using @RolesAllowed Annotation

```java
@Service
public class OrderService {
    
    @RolesAllowed({"ROLE_ADMIN", "ROLE_SALES"})
    public List<Order> getAllOrders() {
        // Only accessible to admin and sales roles
        return orderRepository.findAll();
    }
}
```

### Using @PreAuthorize and @PostAuthorize

```java
@Service
public class DocumentService {
    
    @PreAuthorize("hasRole('ADMIN') or #document.ownerId == authentication.principal.id")
    public void updateDocument(Document document) {
        // Only accessible to admin or the document owner
        documentRepository.save(document);
    }
    
    @PostAuthorize("returnObject.ownerId == authentication.principal.id or hasRole('ADMIN')")
    public Document getDocumentById(Long id) {
        // Document only returned if user is owner or admin
        return documentRepository.findById(id)
            .orElseThrow(() -> new DocumentNotFoundException("Document not found"));
    }
    
    @PreAuthorize("hasRole('ADMIN') or " +
                  "hasRole('MANAGER') or " +
                  "(hasRole('USER') and @securityService.isDocumentOwner(#documentId))")
    public void deleteDocument(Long documentId) {
        // Only admin, manager, or document owner can delete
        documentRepository.deleteById(documentId);
    }
}

@Service
public class SecurityService {
    
    private final DocumentRepository documentRepository;
    
    @Autowired
    public SecurityService(DocumentRepository documentRepository) {
        this.documentRepository = documentRepository;
    }
    
    public boolean isDocumentOwner(Long documentId) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication == null) {
            return false;
        }
        
        String username = authentication.getName();
        Document document = documentRepository.findById(documentId).orElse(null);
        
        if (document == null) {
            return false;
        }
        
        return document.getOwnerUsername().equals(username);
    }
}
```

### Using SpEL with Method Security

```java
@PreAuthorize("authentication.principal.username == #username and hasRole('USER')")
public UserDetails getUser(String username) {
    // Only the user themselves can access their details
    return userDetailsService.loadUserByUsername(username);
}

@PreAuthorize("#user.id == authentication.principal.id or hasRole('ADMIN')")
public void updateUser(User user) {
    // Only the user themselves or an admin can update the user
    userRepository.save(user);
}

@PreAuthorize("hasRole('ADMIN') and hasIpAddress('192.168.1.0/24')")
public void adminFunction() {
    // Only admin from the office IP range
    // Administrative logic
}
```

## CSRF Protection

Cross-Site Request Forgery (CSRF) is an attack that forces users to execute unwanted actions on a web application they're authenticated with.

### Default CSRF Protection

Spring Security enables CSRF protection by default for any non-GET, HEAD, TRACE, OPTIONS requests.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf() // CSRF protection enabled by default
            .and()
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin();
    }
}
```

### Customizing CSRF Protection

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()) // Store token in a cookie
                .ignoringAntMatchers("/api/webhooks/**") // Ignore CSRF for webhooks
                .and()
            // Other configuration
    }
}
```

### Custom CSRF Token Repository

```java
public class CustomCsrfTokenRepository implements CsrfTokenRepository {
    
    private final String headerName = "X-CSRF-TOKEN";
    private final String parameterName = "_csrf";
    private final String cookieName = "XSRF-TOKEN";
    
    @Override
    public CsrfToken generateToken(HttpServletRequest request) {
        return new DefaultCsrfToken(headerName, parameterName, UUID.randomUUID().toString());
    }
    
    @Override
    public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
        if (token == null) {
            // Remove cookie if token is null
            Cookie cookie = new Cookie(cookieName, null);
            cookie.setMaxAge(0);
            cookie.setPath(getCookiePath(request));
            cookie.setSecure(request.isSecure());
            response.addCookie(cookie);
        } else {
            // Set CSRF token in cookie
            Cookie cookie = new Cookie(cookieName, token.getToken());
            cookie.setPath(getCookiePath(request));
            cookie.setSecure(request.isSecure());
            cookie.setHttpOnly(false); // Allow JavaScript access
            cookie.setMaxAge(-1); // Session cookie
            response.addCookie(cookie);
        }
    }
    
    @Override
    public CsrfToken loadToken(HttpServletRequest request) {
        // Load token from cookie
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals(cookieName)) {
                    return new DefaultCsrfToken(headerName, parameterName, cookie.getValue());
                }
            }
        }
        return null;
    }
    
    private String getCookiePath(HttpServletRequest request) {
        String contextPath = request.getContextPath();
        return contextPath.length() > 0 ? contextPath : "/";
    }
}
```

### CSRF in Thymeleaf Templates

```html
<!-- Form with CSRF token (automatically added by Thymeleaf) -->
<form th:action="@{/process}" method="post">
    <!-- Form fields -->
    <button type="submit">Submit</button>
</form>

<!-- Manual CSRF token addition -->
<form action="/process" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <!-- Form fields -->
    <button type="submit">Submit</button>
</form>
```

### CSRF in JavaScript Applications

```javascript
// Extract CSRF token from cookie
function getCsrfToken() {
    const name = 'XSRF-TOKEN=';
    const decodedCookie = decodeURIComponent(document.cookie);
    const cookieArray = decodedCookie.split(';');
    for (let i = 0; i < cookieArray.length; i++) {
        let cookie = cookieArray[i].trim();
        if (cookie.indexOf(name) === 0) {
            return cookie.substring(name.length, cookie.length);
        }
    }
    return '';
}

// Add CSRF token to headers
async function submitForm(data) {
    const csrfToken = getCsrfToken();
    
    try {
        const response = await fetch('/api/submit', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-CSRF-TOKEN': csrfToken
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`Error: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Error submitting form:', error);
        throw error;
    }
}
```

## CORS Configuration

Cross-Origin Resource Sharing (CORS) is a mechanism that uses HTTP headers to allow a web application running at one origin to access resources from a server at a different origin.

### Global CORS Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .cors() // Enable CORS
            .and()
            // Other configuration
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com", "https://app.example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type", "X-CSRF-TOKEN"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

### Controller-Level CORS

```java
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = {"https://example.com", "https://app.example.com"})
public class ProductController {
    
    @GetMapping("/products")
    public List<Product> getProducts() {
        return productService.getAllProducts();
    }
    
    @CrossOrigin(origins = "https://admin.example.com") // Override class-level annotation
    @PostMapping("/products")
    public Product createProduct(@RequestBody Product product) {
        return productService.saveProduct(product);
    }
}
```

### WebMvc Configuration for CORS

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://example.com", "https://app.example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("Authorization", "Content-Type", "X-CSRF-TOKEN")
            .allowCredentials(true)
            .maxAge(3600);
            
        registry.addMapping("/admin/**")
            .allowedOrigins("https://admin.example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

## Password Management

Proper password management is essential for application security.

### Password Encoding

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // Strength factor 12
    }
    
    // Alternative: Use DelegatingPasswordEncoder for multiple encoding types
    @Bean
    public PasswordEncoder delegatingPasswordEncoder() {
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put("bcrypt", new BCryptPasswordEncoder());
        encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        
        return new DelegatingPasswordEncoder("bcrypt", encoders);
    }
}
```

### Password Validation Service

```java
@Service
public class PasswordValidationService {
    
    private static final int MIN_LENGTH = 8;
    private static final int MAX_LENGTH = 64;
    private static final String SPECIAL_CHARS = "!@#$%^&*()_-+=<>?/[]{}|";
    
    // Validate password strength
    public PasswordValidationResult validatePassword(String password) {
        PasswordValidationResult result = new PasswordValidationResult();
        
        // Check length
        if (password.length() < MIN_LENGTH) {
            result.addError("Password must be at least " + MIN_LENGTH + " characters long");
        }
        
        if (password.length() > MAX_LENGTH) {
            result.addError("Password cannot exceed " + MAX_LENGTH + " characters");
        }
        
        // Check for uppercase letter
        if (!containsUppercaseLetter(password)) {
            result.addError("Password must contain at least one uppercase letter");
        }
        
        // Check for lowercase letter
        if (!containsLowercaseLetter(password)) {
            result.addError("Password must contain at least one lowercase letter");
        }
        
        // Check for digit
        if (!containsDigit(password)) {
            result.addError("Password must contain at least one digit");
        }
        
        // Check for special character
        if (!containsSpecialCharacter(password)) {
            result.addError("Password must contain at least one special character");
        }
        
        return result;
    }
    
    // Helper methods
    private boolean containsUppercaseLetter(String password) {
        return password.chars().anyMatch(Character::isUpperCase);
    }
    
    private boolean containsLowercaseLetter(String password) {
        return password.chars().anyMatch(Character::isLowerCase);
    }
    
    private boolean containsDigit(String password) {
        return password.chars().anyMatch(Character::isDigit);
    }
    
    private boolean containsSpecialCharacter(String password) {
        return password.chars().anyMatch(c -> SPECIAL_CHARS.indexOf(c) >= 0);
    }
    
    // Result class
    public static class PasswordValidationResult {
        private final List<String> errors = new ArrayList<>();
        
        public void addError(String error) {
            errors.add(error);
        }
        
        public List<String> getErrors() {
            return errors;
        }
        
        public boolean isValid() {
            return errors.isEmpty();
        }
    }
}
```

### Password Reset Service

```java
@Service
public class PasswordResetService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final PasswordResetTokenRepository tokenRepository;
    private final EmailService emailService;
    
    // Constructor with dependency injection
    
    public void initiatePasswordReset(String email) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + email));
        
        String token = generateToken();
        PasswordResetToken resetToken = new PasswordResetToken();
        resetToken.setUser(user);
        resetToken.setToken(token);
        resetToken.setExpiryDate(calculateExpiryDate());
        
        tokenRepository.save(resetToken);
        
        emailService.sendPasswordResetEmail(user.getEmail(), token);
    }
    
    public boolean validatePasswordResetToken(String token) {
        PasswordResetToken resetToken = tokenRepository.findByToken(token);
        
        if (resetToken == null) {
            return false;
        }
        
        if (isTokenExpired(resetToken)) {
            tokenRepository.delete(resetToken);
            return false;
        }
        
        return true;
    }
    
    public void resetPassword(String token, String newPassword) {
        PasswordResetToken resetToken = tokenRepository.findByToken(token);
        
        if (resetToken == null || isTokenExpired(resetToken)) {
            throw new InvalidTokenException("Invalid or expired token");
        }
        
        User user = resetToken.getUser();
        user.setPassword(passwordEncoder.encode(newPassword));
        userRepository.save(user);
        
        tokenRepository.delete(resetToken);
    }
    
    private String generateToken() {
        return UUID.randomUUID().toString();
    }
    
    private Date calculateExpiryDate() {
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.HOUR, 24); // Token valid for 24 hours
        return calendar.getTime();
    }
    
    private boolean isTokenExpired(PasswordResetToken token) {
        return token.getExpiryDate().before(new Date());
    }
}

@Entity
public class PasswordResetToken {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String token;
    
    @OneToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    private Date expiryDate;
    
    // Getters and setters
}
```

## Remember Me Authentication

Remember Me authentication allows users to remain authenticated between browser sessions.

### Basic Remember Me Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login").permitAll()
                .and()
            .rememberMe() // Enable remember me
                .key("uniqueAndSecretKey") // Secret key for token signature
                .tokenValiditySeconds(2592000) // 30 days
                .userDetailsService(userDetailsService);
    }
}
```

### Remember Me Checkbox in Login Form

```html
<form th:action="@{/login}" method="post">
    <div>
        <label>Username:</label>
        <input type="text" name="username"/>
    </div>
    <div>
        <label>Password:</label>
        <input type="password" name="password"/>
    </div>
    <div>
        <label>
            <input type="checkbox" name="remember-me"/> Remember me
        </label>
    </div>
    <div>
        <input type="submit" value="Log in"/>
    </div>
</form>
```

### Persistent Remember Me with Database

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login").permitAll()
                .and()
            .rememberMe()
                .tokenRepository(persistentTokenRepository()) // Use database store
                .tokenValiditySeconds(2592000) // 30 days
                .userDetailsService(userDetailsService);
    }
    
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        // Uncomment the following line on first run to create the table automatically
        // tokenRepository.setCreateTableOnStartup(true);
        return tokenRepository;
    }
}
```

### Custom Remember Me Implementation

```java
public class CustomRememberMeServices extends AbstractRememberMeServices {
    
    private final UserDetailsService userDetailsService;
    
    public CustomRememberMeServices(String key, UserDetailsService userDetailsService) {
        super(key, userDetailsService);
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request, HttpServletResponse response) throws RememberMeAuthenticationException {
        if (cookieTokens.length != 2) {
            throw new InvalidCookieException("Cookie token did not contain 2 tokens, but contained '" + Arrays.asList(cookieTokens) + "'");
        }
        
        String username = cookieTokens[0];
        String token = cookieTokens[1];
        
        // Validate token (e.g., check in database)
        
        try {
            return userDetailsService.loadUserByUsername(username);
        } catch (UsernameNotFoundException e) {
            throw new RememberMeAuthenticationException("User not found: " + e.getMessage());
        }
    }
    
    @Override
    protected void onLoginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        String username = authentication.getName();
        String token = generateToken();
        
        // Save token to database
        
        // Set cookie
        setCookie(new String[]{username, token}, getTokenValiditySeconds(), request, response);
    }
}

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login").permitAll()
                .and()
            .rememberMe()
                .rememberMeServices(rememberMeServices())
                .key("uniqueAndSecretKey");
    }
    
    @Bean
    public RememberMeServices rememberMeServices() {
        CustomRememberMeServices rememberMeServices = new CustomRememberMeServices("uniqueAndSecretKey", userDetailsService);
        rememberMeServices.setParameter("remember-me");
        rememberMeServices.setCookieName("REMEMBER_ME");
        rememberMeServices.setTokenValiditySeconds(2592000); // 30 days
        return rememberMeServices;
    }
}
```

## Session Management

Session management controls how user sessions are tracked and protected.

### Basic Session Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED) // Default
                .invalidSessionUrl("/login?invalid") // Redirect on invalid session
                .maximumSessions(1) // Limit to one session per user
                .maxSessionsPreventsLogin(false) // Allow new login, invalidate older session
                .expiredUrl("/login?expired"); // Redirect on session expiration
    }
}
```

### Session Fixation Protection

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .sessionManagement()
                .sessionFixation().changeSessionId() // Default protection
                // Other options: none(), migrateSession(), newSession()
    }
}
```

### Custom Session Management

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private FindByIndexNameSessionRepository<?> sessionRepository;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .sessionManagement()
                .maximumSessions(1)
                .sessionRegistry(sessionRegistry())
                .expiredUrl("/login?expired")
                .and()
            .sessionFixation().changeSessionId()
            .invalidSessionUrl("/login?invalid")
            .sessionAuthenticationErrorUrl("/login?auth-error");
    }
    
    @Bean
    public SpringSessionBackedSessionRegistry<?> sessionRegistry() {
        return new SpringSessionBackedSessionRegistry<>(sessionRepository);
    }
}
```

### Concurrent Session Control with Database

```java
@Configuration
public class SessionConfig {
    
    @Bean
    public HttpSessionEventPublisher httpSessionEventPublisher() {
        return new HttpSessionEventPublisher();
    }
}

@Service
public class SessionManagementService {
    
    private final FindByIndexNameSessionRepository<? extends Session> sessionRepository;
    
    @Autowired
    public SessionManagementService(FindByIndexNameSessionRepository<? extends Session> sessionRepository) {
        this.sessionRepository = sessionRepository;
    }
    
    public List<Session> getUserSessions(String username) {
        Map<String, ? extends Session> userSessions = sessionRepository.findByPrincipalName(username);
        return new ArrayList<>(userSessions.values());
    }
    
    public void terminateUserSessions(String username) {
        Map<String, ? extends Session> userSessions = sessionRepository.findByPrincipalName(username);
        userSessions.keySet().forEach(sessionId -> sessionRepository.deleteById(sessionId));
    }
    
    public void terminateSession(String sessionId) {
        sessionRepository.deleteById(sessionId);
    }
}
```

## Security Headers

Security headers help protect against various browser-based attacks.

### Default Security Headers

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .headers() // Default headers enabled
            .and()
            // Other configuration
    }
}
```

### Custom Security Headers

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .headers()
                .contentSecurityPolicy("default-src 'self'; script-src 'self' https://trusted.cdn.com; img-src 'self' data:; style-src 'self' https://trusted.cdn.com;")
                .and()
                .referrerPolicy(ReferrerPolicyHeaderWriter.ReferrerPolicy.SAME_ORIGIN)
                .and()
                .frameOptions().deny() // X-Frame-Options: DENY
                .and()
                .xssProtection().block(true) // X-XSS-Protection: 1; mode=block
                .and()
                .contentTypeOptions() // X-Content-Type-Options: nosniff
                .and()
            // Other configuration
    }
}
```

### Content Security Policy Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .headers()
                .contentSecurityPolicy(cspConfig -> cspConfig
                    .policyDirectives("default-src 'self'; " +
                                     "script-src 'self' https://trusted.cdn.com; " +
                                     "style-src 'self' https://trusted.cdn.com; " +
                                     "img-src 'self' data: https://images.cdn.com; " +
                                     "connect-src 'self' https://api.example.com; " +
                                     "font-src 'self' https://fonts.googleapis.com; " +
                                     "frame-src 'self' https://video.example.com; " +
                                     "object-src 'none'; " +
                                     "report-uri /csp-report-endpoint"))
                .and()
            // Other configuration
    }
}
```

### Custom Header Writer

```java
public class CustomSecurityHeaderWriter implements HeaderWriter {
    
    @Override
    public void writeHeaders(HttpServletRequest request, HttpServletResponse response) {
        response.setHeader("X-Custom-Security-Header", "CustomValue");
        
        // Add dynamic header based on request
        if (request.isSecure()) {
            response.setHeader("X-Secure-Request", "true");
        }
        
        // Add business-specific headers
        String userAgent = request.getHeader("User-Agent");
        if (userAgent != null && userAgent.contains("Mobile")) {
            response.setHeader("X-Optimized-For", "mobile");
        }
    }
}

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .headers()
                .addHeaderWriter(new CustomSecurityHeaderWriter())
                .and()
            // Other configuration
    }
}
```

## Securing REST APIs

Securing REST APIs requires a combination of authentication, authorization, and additional security measures.

### JWT-Based API Security

```java
@Configuration
@EnableWebSecurity
public class ApiSecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final JwtTokenUtil jwtTokenUtil;
    private final UserDetailsService userDetailsService;
    
    @Autowired
    public ApiSecurityConfig(JwtTokenUtil jwtTokenUtil, UserDetailsService userDetailsService) {
        this.jwtTokenUtil = jwtTokenUtil;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // Typically disabled for APIs
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // No sessions
                .and()
            .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll() // Public auth endpoints
                .antMatchers("/api/public/**").permitAll() // Public endpoints
                .antMatchers("/api/admin/**").hasRole("ADMIN") // Admin endpoints
                .antMatchers("/api/users/**").hasAnyRole("ADMIN", "USER") // User endpoints
                .anyRequest().authenticated()
                .and()
            .exceptionHandling()
                .authenticationEntryPoint(new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED))
                .and()
            .addFilterBefore(
                new JwtAuthenticationFilter(jwtTokenUtil, userDetailsService),
                UsernamePasswordAuthenticationFilter.class
            );
    }
}
```

### API Key Authentication

```java
public class ApiKeyAuthFilter extends OncePerRequestFilter {
    
    private final String headerName;
    private final ApiKeyService apiKeyService;
    
    public ApiKeyAuthFilter(String headerName, ApiKeyService apiKeyService) {
        this.headerName = headerName;
        this.apiKeyService = apiKeyService;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        
        String apiKey = request.getHeader(headerName);
        
        if (apiKey == null) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.getWriter().write("Missing API key");
            return;
        }
        
        ApiKeyDetails apiKeyDetails = apiKeyService.getApiKeyDetails(apiKey);
        
        if (apiKeyDetails == null) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.getWriter().write("Invalid API key");
            return;
        }
        
        // Create authentication token
        List<GrantedAuthority> authorities = apiKeyDetails.getRoles().stream()
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
            
        UsernamePasswordAuthenticationToken authentication = 
            new UsernamePasswordAuthenticationToken(
                apiKeyDetails.getUsername(), 
                null, 
                authorities
            );
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        
        filterChain.doFilter(request, response);
    }
}

@Service
public class ApiKeyService {
    
    private final ApiKeyRepository apiKeyRepository;
    
    @Autowired
    public ApiKeyService(ApiKeyRepository apiKeyRepository) {
        this.apiKeyRepository = apiKeyRepository;
    }
    
    public ApiKeyDetails getApiKeyDetails(String apiKey) {
        ApiKey key = apiKeyRepository.findByKey(apiKey);
        
        if (key == null || !key.isActive()) {
            return null;
        }
        
        return new ApiKeyDetails(key.getUsername(), key.getRoles());
    }
    
    public static class ApiKeyDetails {
        private final String username;
        private final List<String> roles;
        
        public ApiKeyDetails(String username, List<String> roles) {
            this.username = username;
            this.roles = roles;
        }
        
        public String getUsername() {
            return username;
        }
        
        public List<String> getRoles() {
            return roles;
        }
    }
}

@Configuration
@EnableWebSecurity
public class ApiSecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final ApiKeyService apiKeyService;
    
    @Autowired
    public ApiSecurityConfig(ApiKeyService apiKeyService) {
        this.apiKeyService = apiKeyService;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
                .and()
            .addFilterBefore(
                new ApiKeyAuthFilter("X-API-KEY", apiKeyService),
                UsernamePasswordAuthenticationFilter.class
            );
    }
}
```

### Rate Limiting

```java
public class RateLimitingFilter extends OncePerRequestFilter {
    
    private final RateLimiter rateLimiter;
    
    public RateLimitingFilter(RateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        
        String clientId = getClientIdentifier(request);
        
        if (!rateLimiter.tryAcquire(clientId)) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded. Please try again later.");
            return;
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String getClientIdentifier(HttpServletRequest request) {
        // Get client identifier (API key, user ID, or IP address)
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        
        if (authentication != null && authentication.isAuthenticated()) {
            return authentication.getName();
        }
        
        // Fall back to IP address
        return request.getRemoteAddr();
    }
}

@Service
public class RateLimiter {
    
    // Cache with expiry
    private final LoadingCache<String, AtomicInteger> requestCounts;
    
    // Rate limits per client
    private final Map<String, Integer> clientRateLimits = new HashMap<>();
    
    public RateLimiter() {
        // Default rate limit: 100 requests per minute
        requestCounts = CacheBuilder.newBuilder()
            .expireAfterWrite(1, TimeUnit.MINUTES)
            .build(new CacheLoader<String, AtomicInteger>() {
                @Override
                public AtomicInteger load(String key) {
                    return new AtomicInteger(0);
                }
            });
            
        // Set custom rate limits for specific clients
        clientRateLimits.put("premium-client", 1000);
        clientRateLimits.put("admin", 5000);
    }
    
    public boolean tryAcquire(String clientId) {
        AtomicInteger counter = requestCounts.getUnchecked(clientId);
        int limit = clientRateLimits.getOrDefault(clientId, 100); // Default limit
        
        return counter.incrementAndGet() <= limit;
    }
}

@Configuration
@EnableWebSecurity
public class ApiSecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final RateLimiter rateLimiter;
    
    @Autowired
    public ApiSecurityConfig(RateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // Other security configuration
            .addFilterAfter(
                new RateLimitingFilter(rateLimiter),
                UsernamePasswordAuthenticationFilter.class
            );
    }
}
```

## Testing Security

Testing security configuration is essential to ensure that your application is properly protected.

### Unit Testing Authentication

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserDetailsServiceTest {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Test
    public void testLoadUserByUsername_ExistingUser() {
        // Given a username for an existing user
        String username = "testuser";
        
        // When loadUserByUsername is called
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        
        // Then the user details should be correctly loaded
        assertNotNull(userDetails);
        assertEquals(username, userDetails.getUsername());
        assertTrue(userDetails.isEnabled());
        assertTrue(userDetails.getAuthorities().stream()
            .anyMatch(a -> a.getAuthority().equals("ROLE_USER")));
    }
    
    @Test(expected = UsernameNotFoundException.class)
    public void testLoadUserByUsername_NonExistingUser() {
        // Given a username for a non-existing user
        String username = "nonexistinguser";
        
        // When loadUserByUsername is called, then it should throw UsernameNotFoundException
        userDetailsService.loadUserByUsername(username);
    }
}
```

### Testing Security Configuration

```java
@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class SecurityConfigTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @MockBean
    private UserDetailsService userDetailsService;
    
    @Test
    public void testPublicEndpointAccessible() throws Exception {
        mockMvc.perform(get("/public"))
            .andExpect(status().isOk());
    }
    
    @Test
    public void testProtectedEndpointReturnsUnauthorized() throws Exception {
        mockMvc.perform(get("/api/users"))
            .andExpect(status().isUnauthorized());
    }
    
    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    public void testProtectedEndpointAccessibleToAuthenticatedUser() throws Exception {
        mockMvc.perform(get("/api/users"))
            .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    public void testAdminEndpointForbiddenToRegularUser() throws Exception {
        mockMvc.perform(get("/api/admin"))
            .andExpect(status().isForbidden());
    }
    
    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    public void testAdminEndpointAccessibleToAdmin() throws Exception {
        mockMvc.perform(get("/api/admin"))
            .andExpect(status().isOk());
    }
}
```

### Testing Method Security

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceSecurityTest {
    
    @Autowired
    private UserService userService;
    
    @Test(expected = AccessDeniedException.class)
    @WithMockUser(username = "user", roles = {"USER"})
    public void testGetAllUsers_AsUser_ShouldBeDenied() {
        userService.getAllUsers();
    }
    
    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    public void testGetAllUsers_AsAdmin_ShouldBeAllowed() {
        List<User> users = userService.getAllUsers();
        assertNotNull(users);
    }
    
    @Test
    @WithMockUser(username = "john", roles = {"USER"})
    public void testGetUserByUsername_SameUser_ShouldBeAllowed() {
        User user = userService.getUserByUsername("john");
        assertNotNull(user);
        assertEquals("john", user.getUsername());
    }
    
    @Test(expected = AccessDeniedException.class)
    @WithMockUser(username = "john", roles = {"USER"})
    public void testGetUserByUsername_DifferentUser_ShouldBeDenied() {
        userService.getUserByUsername("jane");
    }
}
```

### Testing Authentication

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class AuthenticationIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    public void testLogin_ValidCredentials() {
        // Given valid credentials
        HttpEntity<LoginRequest> request = new HttpEntity<>(new LoginRequest("user", "password"));
        
        // When login endpoint is called
        ResponseEntity<JwtResponse> response = restTemplate.postForEntity(
            "/api/auth/login", request, JwtResponse.class);
        
        // Then it should return OK with a token
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertNotNull(response.getBody());
        assertNotNull(response.getBody().getToken());
    }
    
    @Test
    public void testLogin_InvalidCredentials() {
        // Given invalid credentials
        HttpEntity<LoginRequest> request = new HttpEntity<>(new LoginRequest("user", "wrongpassword"));
        
        // When login endpoint is called
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/auth/login", request, ErrorResponse.class);
        
        // Then it should return UNAUTHORIZED
        assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
    }
    
    @Test
    public void testSecuredEndpoint_WithToken() {
        // First, get a valid token
        HttpEntity<LoginRequest> loginRequest = new HttpEntity<>(new LoginRequest("user", "password"));
        ResponseEntity<JwtResponse> loginResponse = restTemplate.postForEntity(
            "/api/auth/login", loginRequest, JwtResponse.class);
        String token = loginResponse.getBody().getToken();
        
        // When calling secured endpoint with token
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + token);
        HttpEntity<?> requestEntity = new HttpEntity<>(headers);
        
        ResponseEntity<String> response = restTemplate.exchange(
            "/api/users/profile", HttpMethod.GET, requestEntity, String.class);
        
        // Then it should return OK
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
    
    @Data
    public static class LoginRequest {
        private String username;
        private String password;
        
        public LoginRequest(String username, String password) {
            this.username = username;
            this.password = password;
        }
    }
    
    @Data
    public static class JwtResponse {
        private String token;
    }
    
    @Data
    public static class ErrorResponse {
        private String message;
    }
}
```

## Practical Case Study

Let's build a comprehensive security implementation for an e-commerce application.

### Requirements

1. User authentication with JWT for API clients
2. Form-based authentication for web interface
3. OAuth2 login with Google and GitHub
4. Role-based access control (Customer, Employee, Admin)
5. Method-level security
6. Password management with reset functionality
7. CSRF protection for web interface
8. Rate limiting for APIs
9. Remember Me functionality
10. Session management
11. Security headers
12. API key authentication for third-party integrations

### Implementation Plan

1. **Domain Model**
   - User, Role, Permission entities
   - JWT token management
   - Password reset token

2. **Security Configuration**
   - Web security config
   - Method security config
   - JWT authentication
   - OAuth2 login

3. **Controllers**
   - Authentication controller
   - User management controller
   - Product controller
   - Order controller

4. **Security Services**
   - User details service
   - JWT token service
   - Password service
   - Rate limiting service

5. **Integration**
   - Third-party API security
   - API gateway security

By following this guide and the case study, you'll be able to implement a robust security solution for your Spring applications, protecting them against common security threats and ensuring that only authorized users can access sensitive functionality.

## Conclusion

Spring Security is a powerful framework that provides comprehensive protection for your applications. By understanding its core concepts and implementing the techniques described in this guide, you can create a secure application that protects user data and restricts access to authorized users.

Key takeaways:
1. Spring Security offers multiple authentication mechanisms to suit different needs
2. Authorization can be implemented at both the URL and method levels
3. Protection against common attacks (CSRF, XSS, etc.) is built-in
4. Security should be applied in layers for defense in depth
5. Always follow security best practices, especially for password management
6. Testing security configurations is essential

Remember that security is an ongoing process, not a one-time implementation. Stay updated on the latest security threats and Spring Security features to maintain a robust security posture for your applications.