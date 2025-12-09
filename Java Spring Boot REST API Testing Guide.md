# 🚀 Java Spring Boot REST API Testing Guide
## Complete Testing with JUnit & Mockito 📚

---

# 📋 Table of Contents

1. [Testing Fundamentals](#-testing-fundamentals)
2. [Controller Testing](#-controller-testing)
3. [Service Testing](#-service-testing)
4. [Repository Testing](#-repository-testing)
5. [Integration Testing](#-integration-testing)
6. [Interview Questions & Answers](#-interview-questions--answers)
7. [Complete Example Project](#-complete-example-project)

---

# 🎯 Testing Fundamentals

## 📦 Required Dependencies

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Boot Starter Test (includes JUnit 5, Mockito, AssertJ) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- H2 Database for Integration Testing -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 🏗️ Testing Pyramid

```
                    🔺
                   /  \
                  / E2E \        ⬅️ Few (Slow, Expensive)
                 /--------\
                /Integration\    ⬅️ Some (Medium)
               /--------------\
              /   Unit Tests   \ ⬅️ Many (Fast, Cheap)
             /------------------\
```

---

# 🎮 Controller Testing

> 💡 **Purpose**: Test HTTP request/response handling, validation, and proper delegation to services

## 🟢 Basic Level - Simple GET Endpoint

### 📁 Controller Class

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.findAll());
    }
}
```

### 🧪 Basic Controller Test

```java
@WebMvcTest(UserController.class)  // 🎯 Loads only web layer
class UserControllerBasicTest {

    @Autowired
    private MockMvc mockMvc;  // 🔧 Simulates HTTP requests
    
    @MockBean  // 🎭 Creates mock of UserService
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;  // 📝 JSON serialization
    
    // ✅ Test 1: GET single user - Success scenario
    @Test
    @DisplayName("GET /api/users/{id} - Should return user when exists")
    void getUserById_WhenUserExists_ReturnsUser() throws Exception {
        // 📋 ARRANGE - Setup test data and mock behavior
        Long userId = 1L;
        User mockUser = new User(userId, "John Doe", "john@email.com");
        
        when(userService.findById(userId)).thenReturn(mockUser);
        
        // 🎬 ACT & ASSERT - Perform request and verify response
        mockMvc.perform(get("/api/users/{id}", userId)
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())  // ✅ HTTP 200
                .andExpect(jsonPath("$.id").value(userId))
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@email.com"));
        
        // 🔍 Verify service was called
        verify(userService, times(1)).findById(userId);
    }
    
    // ✅ Test 2: GET all users
    @Test
    @DisplayName("GET /api/users - Should return list of all users")
    void getAllUsers_ReturnsUserList() throws Exception {
        // 📋 ARRANGE
        List<User> users = Arrays.asList(
            new User(1L, "John", "john@email.com"),
            new User(2L, "Jane", "jane@email.com")
        );
        
        when(userService.findAll()).thenReturn(users);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(get("/api/users"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.length()").value(2))
                .andExpect(jsonPath("$[0].name").value("John"))
                .andExpect(jsonPath("$[1].name").value("Jane"));
    }
}
```

---

## 🟡 Intermediate Level - POST, PUT, DELETE with Validation

### 📁 Enhanced Controller

```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserDTO userDTO) {
        User createdUser = userService.create(userDTO);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(createdUser.getId())
                .toUri();
        return ResponseEntity.created(location).body(createdUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserDTO userDTO) {
        User updatedUser = userService.update(id, userDTO);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<User>> searchUsers(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String email) {
        return ResponseEntity.ok(userService.search(name, email));
    }
}
```

### 🧪 Intermediate Controller Tests

```java
@WebMvcTest(UserController.class)
@ExtendWith(MockitoExtension.class)
class UserControllerIntermediateTest {

    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    // ✅ Test 3: POST - Create user successfully
    @Test
    @DisplayName("POST /api/users - Should create user and return 201")
    void createUser_WithValidData_ReturnsCreated() throws Exception {
        // 📋 ARRANGE
        UserDTO inputDTO = new UserDTO("John Doe", "john@email.com", "password123");
        User createdUser = new User(1L, "John Doe", "john@email.com");
        
        when(userService.create(any(UserDTO.class))).thenReturn(createdUser);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(inputDTO)))
                .andExpect(status().isCreated())  // ✅ HTTP 201
                .andExpect(header().exists("Location"))
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("John Doe"));
        
        // 🔍 Verify with argument capture
        ArgumentCaptor<UserDTO> captor = ArgumentCaptor.forClass(UserDTO.class);
        verify(userService).create(captor.capture());
        assertEquals("john@email.com", captor.getValue().getEmail());
    }
    
    // ❌ Test 4: POST - Validation failure
    @Test
    @DisplayName("POST /api/users - Should return 400 for invalid data")
    void createUser_WithInvalidData_ReturnsBadRequest() throws Exception {
        // 📋 ARRANGE - Invalid email format
        UserDTO invalidDTO = new UserDTO("", "invalid-email", "");
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidDTO)))
                .andExpect(status().isBadRequest())  // ❌ HTTP 400
                .andExpect(jsonPath("$.errors").isArray());
    }
    
    // ✅ Test 5: PUT - Update user
    @Test
    @DisplayName("PUT /api/users/{id} - Should update existing user")
    void updateUser_WhenUserExists_ReturnsUpdatedUser() throws Exception {
        // 📋 ARRANGE
        Long userId = 1L;
        UserDTO updateDTO = new UserDTO("John Updated", "john.updated@email.com", null);
        User updatedUser = new User(userId, "John Updated", "john.updated@email.com");
        
        when(userService.update(eq(userId), any(UserDTO.class))).thenReturn(updatedUser);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(put("/api/users/{id}", userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updateDTO)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Updated"));
    }
    
    // ✅ Test 6: DELETE - Remove user
    @Test
    @DisplayName("DELETE /api/users/{id} - Should delete user and return 204")
    void deleteUser_WhenUserExists_ReturnsNoContent() throws Exception {
        // 📋 ARRANGE
        Long userId = 1L;
        doNothing().when(userService).delete(userId);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(delete("/api/users/{id}", userId))
                .andExpect(status().isNoContent());  // ✅ HTTP 204
        
        verify(userService, times(1)).delete(userId);
    }
    
    // ✅ Test 7: GET with query parameters
    @Test
    @DisplayName("GET /api/users/search - Should search with query params")
    void searchUsers_WithQueryParams_ReturnsFilteredList() throws Exception {
        // 📋 ARRANGE
        List<User> searchResults = Collections.singletonList(
            new User(1L, "John", "john@email.com")
        );
        
        when(userService.search("John", null)).thenReturn(searchResults);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(get("/api/users/search")
                .param("name", "John"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.length()").value(1));
    }
}
```

---

## 🔴 Advanced Level - Exception Handling, Security, Pagination

### 📁 Advanced Controller with Exception Handling

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.findById(id)
                .map(ResponseEntity::ok)
                .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
    
    @GetMapping("/paginated")
    public ResponseEntity<Page<User>> getUsersPaginated(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return ResponseEntity.ok(userService.findAllPaginated(pageable));
    }
    
    @PostMapping("/batch")
    public ResponseEntity<List<User>> createUsersBatch(
            @Valid @RequestBody List<UserDTO> userDTOs) {
        return ResponseEntity.ok(userService.createBatch(userDTOs));
    }
}

// 🛡️ Global Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        ErrorResponse errorResponse = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(errorResponse);
    }
}
```

### 🧪 Advanced Controller Tests

```java
@WebMvcTest(UserController.class)
@Import(GlobalExceptionHandler.class)  // 📦 Include exception handler
class UserControllerAdvancedTest {

    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    // ❌ Test 8: Exception handling - User not found
    @Test
    @DisplayName("GET /api/users/{id} - Should return 404 when user not found")
    void getUserById_WhenNotExists_ReturnsNotFound() throws Exception {
        // 📋 ARRANGE
        Long userId = 999L;
        when(userService.findById(userId)).thenReturn(Optional.empty());
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(get("/api/users/{id}", userId))
                .andExpect(status().isNotFound())  // ❌ HTTP 404
                .andExpect(jsonPath("$.status").value(404))
                .andExpect(jsonPath("$.message").value("User not found: 999"));
    }
    
    // ✅ Test 9: Pagination
    @Test
    @DisplayName("GET /api/users/paginated - Should return paginated results")
    void getUsersPaginated_ReturnsPagedResponse() throws Exception {
        // 📋 ARRANGE
        List<User> users = Arrays.asList(
            new User(1L, "John", "john@email.com"),
            new User(2L, "Jane", "jane@email.com")
        );
        Page<User> userPage = new PageImpl<>(users, PageRequest.of(0, 10), 50);
        
        when(userService.findAllPaginated(any(Pageable.class))).thenReturn(userPage);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(get("/api/users/paginated")
                .param("page", "0")
                .param("size", "10")
                .param("sortBy", "name"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content.length()").value(2))
                .andExpect(jsonPath("$.totalElements").value(50))
                .andExpect(jsonPath("$.totalPages").value(5))
                .andExpect(jsonPath("$.number").value(0));
    }
    
    // ✅ Test 10: Batch operations
    @Test
    @DisplayName("POST /api/users/batch - Should create multiple users")
    void createUsersBatch_WithValidData_ReturnsCreatedUsers() throws Exception {
        // 📋 ARRANGE
        List<UserDTO> inputDTOs = Arrays.asList(
            new UserDTO("User 1", "user1@email.com", "pass1"),
            new UserDTO("User 2", "user2@email.com", "pass2")
        );
        
        List<User> createdUsers = Arrays.asList(
            new User(1L, "User 1", "user1@email.com"),
            new User(2L, "User 2", "user2@email.com")
        );
        
        when(userService.createBatch(anyList())).thenReturn(createdUsers);
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(post("/api/users/batch")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(inputDTOs)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.length()").value(2));
    }
    
    // ✅ Test 11: Custom headers and content negotiation
    @Test
    @DisplayName("Should handle custom headers and content type")
    void getUserById_WithCustomHeaders_ReturnsCorrectResponse() throws Exception {
        // 📋 ARRANGE
        User user = new User(1L, "John", "john@email.com");
        when(userService.findById(1L)).thenReturn(Optional.of(user));
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(get("/api/users/1")
                .header("X-Custom-Header", "CustomValue")
                .header("Accept-Language", "en-US")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON));
    }
    
    // ✅ Test 12: Testing with ResultActions for complex assertions
    @Test
    @DisplayName("Should verify response using ResultActions chain")
    void getUserById_VerifyCompleteResponse() throws Exception {
        // 📋 ARRANGE
        User user = new User(1L, "John Doe", "john@email.com");
        user.setCreatedAt(LocalDateTime.of(2024, 1, 1, 10, 0));
        when(userService.findById(1L)).thenReturn(Optional.of(user));
        
        // 🎬 ACT
        MvcResult result = mockMvc.perform(get("/api/users/1"))
                .andExpect(status().isOk())
                .andDo(print())  // 🖨️ Print request/response details
                .andReturn();
        
        // 🔍 ASSERT - Parse and verify response
        String content = result.getResponse().getContentAsString();
        User responseUser = objectMapper.readValue(content, User.class);
        
        assertAll("User verification",
            () -> assertNotNull(responseUser),
            () -> assertEquals(1L, responseUser.getId()),
            () -> assertEquals("John Doe", responseUser.getName()),
            () -> assertTrue(responseUser.getEmail().contains("@"))
        );
    }
}
```

---

# 🔧 Service Testing

> 💡 **Purpose**: Test business logic in isolation from web and database layers

## 🟢 Basic Level - Simple Service Methods

### 📁 Service Class

```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    public User findById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public User create(UserDTO userDTO) {
        // Check for duplicate email
        if (userRepository.existsByEmail(userDTO.getEmail())) {
            throw new DuplicateResourceException("Email already exists");
        }
        
        User user = new User();
        user.setName(userDTO.getName());
        user.setEmail(userDTO.getEmail());
        user.setPassword(passwordEncoder.encode(userDTO.getPassword()));
        user.setCreatedAt(LocalDateTime.now());
        
        return userRepository.save(user);
    }
}
```

### 🧪 Basic Service Tests

```java
@ExtendWith(MockitoExtension.class)  // 🎯 Enable Mockito annotations
class UserServiceBasicTest {

    @Mock  // 🎭 Create mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks  // 📥 Inject mocks into service
    private UserService userService;
    
    // ✅ Test 1: Find by ID - Success
    @Test
    @DisplayName("findById - Should return user when exists")
    void findById_WhenUserExists_ReturnsUser() {
        // 📋 ARRANGE
        Long userId = 1L;
        User expectedUser = new User(userId, "John", "john@email.com");
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(expectedUser));
        
        // 🎬 ACT
        User result = userService.findById(userId);
        
        // ✅ ASSERT
        assertNotNull(result);
        assertEquals(userId, result.getId());
        assertEquals("John", result.getName());
        
        // 🔍 Verify interaction
        verify(userRepository, times(1)).findById(userId);
    }
    
    // ❌ Test 2: Find by ID - Not found
    @Test
    @DisplayName("findById - Should throw exception when not exists")
    void findById_WhenUserNotExists_ThrowsException() {
        // 📋 ARRANGE
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // 🎬 ACT & ASSERT
        ResourceNotFoundException exception = assertThrows(
            ResourceNotFoundException.class,
            () -> userService.findById(userId)
        );
        
        assertEquals("User not found", exception.getMessage());
    }
    
    // ✅ Test 3: Find all users
    @Test
    @DisplayName("findAll - Should return all users")
    void findAll_ReturnsAllUsers() {
        // 📋 ARRANGE
        List<User> expectedUsers = Arrays.asList(
            new User(1L, "John", "john@email.com"),
            new User(2L, "Jane", "jane@email.com")
        );
        
        when(userRepository.findAll()).thenReturn(expectedUsers);
        
        // 🎬 ACT
        List<User> result = userService.findAll();
        
        // ✅ ASSERT
        assertEquals(2, result.size());
        assertEquals("John", result.get(0).getName());
    }
    
    // ✅ Test 4: Create user - Success
    @Test
    @DisplayName("create - Should create and return new user")
    void create_WithValidData_ReturnsCreatedUser() {
        // 📋 ARRANGE
        UserDTO userDTO = new UserDTO("John", "john@email.com", "password123");
        User savedUser = new User(1L, "John", "john@email.com");
        
        when(userRepository.existsByEmail(userDTO.getEmail())).thenReturn(false);
        when(passwordEncoder.encode("password123")).thenReturn("encodedPassword");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // 🎬 ACT
        User result = userService.create(userDTO);
        
        // ✅ ASSERT
        assertNotNull(result);
        assertEquals(1L, result.getId());
        
        // 🔍 Verify all interactions
        verify(userRepository).existsByEmail(userDTO.getEmail());
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }
}
```

---

## 🟡 Intermediate Level - Complex Business Logic

### 📁 Enhanced Service

```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private AuditService auditService;
    
    public User update(Long id, UserDTO userDTO) {
        User existingUser = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        
        // Check if email is being changed and if new email exists
        if (!existingUser.getEmail().equals(userDTO.getEmail())) {
            if (userRepository.existsByEmail(userDTO.getEmail())) {
                throw new DuplicateResourceException("Email already in use");
            }
            existingUser.setEmail(userDTO.getEmail());
            emailService.sendEmailChangeNotification(existingUser);
        }
        
        existingUser.setName(userDTO.getName());
        existingUser.setUpdatedAt(LocalDateTime.now());
        
        User updatedUser = userRepository.save(existingUser);
        auditService.logUpdate("USER", id, userDTO);
        
        return updatedUser;
    }
    
    public void delete(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        
        if (user.hasActiveOrders()) {
            throw new BusinessException("Cannot delete user with active orders");
        }
        
        userRepository.delete(user);
        auditService.logDeletion("USER", id);
    }
    
    public UserStatistics calculateStatistics(Long userId) {
        User user = findById(userId);
        
        int totalOrders = user.getOrders().size();
        BigDecimal totalSpent = user.getOrders().stream()
                .map(Order::getTotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        return new UserStatistics(userId, totalOrders, totalSpent);
    }
}
```

### 🧪 Intermediate Service Tests

```java
@ExtendWith(MockitoExtension.class)
class UserServiceIntermediateTest {

    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @Mock
    private AuditService auditService;
    
    @InjectMocks
    private UserService userService;
    
    @Captor  // 📸 Argument captor for complex verification
    private ArgumentCaptor<User> userCaptor;
    
    // ✅ Test 5: Update with email change
    @Test
    @DisplayName("update - Should update user and send notification on email change")
    void update_WhenEmailChanged_SendsNotification() {
        // 📋 ARRANGE
        Long userId = 1L;
        User existingUser = new User(userId, "John", "old@email.com");
        UserDTO updateDTO = new UserDTO("John Updated", "new@email.com", null);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(existingUser));
        when(userRepository.existsByEmail("new@email.com")).thenReturn(false);
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> invocation.getArgument(0));
        
        // 🎬 ACT
        User result = userService.update(userId, updateDTO);
        
        // ✅ ASSERT
        assertEquals("new@email.com", result.getEmail());
        
        // 🔍 Verify email notification was sent
        verify(emailService, times(1)).sendEmailChangeNotification(any(User.class));
        verify(auditService).logUpdate(eq("USER"), eq(userId), any(UserDTO.class));
    }
    
    // ✅ Test 6: Update without email change
    @Test
    @DisplayName("update - Should not send notification when email unchanged")
    void update_WhenEmailUnchanged_NoNotification() {
        // 📋 ARRANGE
        Long userId = 1L;
        User existingUser = new User(userId, "John", "same@email.com");
        UserDTO updateDTO = new UserDTO("John Updated", "same@email.com", null);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(existingUser));
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> invocation.getArgument(0));
        
        // 🎬 ACT
        userService.update(userId, updateDTO);
        
        // ✅ ASSERT - No email notification
        verify(emailService, never()).sendEmailChangeNotification(any());
    }
    
    // ❌ Test 7: Delete user with active orders
    @Test
    @DisplayName("delete - Should throw exception when user has active orders")
    void delete_WhenHasActiveOrders_ThrowsException() {
        // 📋 ARRANGE
        Long userId = 1L;
        User userWithOrders = mock(User.class);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(userWithOrders));
        when(userWithOrders.hasActiveOrders()).thenReturn(true);
        
        // 🎬 ACT & ASSERT
        BusinessException exception = assertThrows(
            BusinessException.class,
            () -> userService.delete(userId)
        );
        
        assertEquals("Cannot delete user with active orders", exception.getMessage());
        verify(userRepository, never()).delete(any());
    }
    
    // ✅ Test 8: Using ArgumentCaptor
    @Test
    @DisplayName("create - Should capture and verify saved user details")
    void create_VerifySavedUserDetails() {
        // 📋 ARRANGE
        UserDTO userDTO = new UserDTO("John", "john@email.com", "pass123");
        
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(userCaptor.capture())).thenAnswer(inv -> {
            User savedUser = inv.getArgument(0);
            savedUser.setId(1L);
            return savedUser;
        });
        
        // 🎬 ACT
        userService.create(userDTO);
        
        // ✅ ASSERT - Verify captured argument
        User capturedUser = userCaptor.getValue();
        assertAll("Captured user verification",
            () -> assertEquals("John", capturedUser.getName()),
            () -> assertEquals("john@email.com", capturedUser.getEmail()),
            () -> assertNotNull(capturedUser.getCreatedAt())
        );
    }
    
    // ✅ Test 9: Calculate statistics
    @Test
    @DisplayName("calculateStatistics - Should compute correct statistics")
    void calculateStatistics_ReturnsCorrectValues() {
        // 📋 ARRANGE
        Long userId = 1L;
        User user = new User(userId, "John", "john@email.com");
        
        List<Order> orders = Arrays.asList(
            createOrder(new BigDecimal("100.00")),
            createOrder(new BigDecimal("250.50")),
            createOrder(new BigDecimal("49.50"))
        );
        user.setOrders(orders);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        
        // 🎬 ACT
        UserStatistics stats = userService.calculateStatistics(userId);
        
        // ✅ ASSERT
        assertEquals(3, stats.getTotalOrders());
        assertEquals(new BigDecimal("400.00"), stats.getTotalSpent());
    }
    
    private Order createOrder(BigDecimal total) {
        Order order = new Order();
        order.setTotal(total);
        return order;
    }
}
```

---

## 🔴 Advanced Level - BDD Style, Spy, Answer

### 🧪 Advanced Service Tests

```java
@ExtendWith(MockitoExtension.class)
class UserServiceAdvancedTest {

    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @Spy  // 🕵️ Partial mock - real methods unless stubbed
    private PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    
    @InjectMocks
    private UserService userService;
    
    // ✅ Test 10: BDD Style with given-when-then
    @Test
    @DisplayName("BDD Style - Should find user by email")
    void shouldFindUserByEmail_BDDStyle() {
        // 📋 GIVEN
        String email = "john@email.com";
        User expectedUser = new User(1L, "John", email);
        
        given(userRepository.findByEmail(email)).willReturn(Optional.of(expectedUser));
        
        // 🎬 WHEN
        Optional<User> result = userService.findByEmail(email);
        
        // ✅ THEN
        then(userRepository).should(times(1)).findByEmail(email);
        assertThat(result)
            .isPresent()
            .hasValueSatisfying(user -> {
                assertThat(user.getName()).isEqualTo("John");
                assertThat(user.getEmail()).isEqualTo(email);
            });
    }
    
    // ✅ Test 11: Using Answer for dynamic responses
    @Test
    @DisplayName("Should use Answer for dynamic mock behavior")
    void create_WithDynamicId_UsingAnswer() {
        // 📋 ARRANGE
        AtomicLong idGenerator = new AtomicLong(1);
        
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> {
            User userToSave = invocation.getArgument(0);
            userToSave.setId(idGenerator.getAndIncrement());
            return userToSave;
        });
        
        // 🎬 ACT
        User user1 = userService.create(new UserDTO("User1", "user1@email.com", "pass"));
        User user2 = userService.create(new UserDTO("User2", "user2@email.com", "pass"));
        
        // ✅ ASSERT
        assertEquals(1L, user1.getId());
        assertEquals(2L, user2.getId());
    }
    
    // ✅ Test 12: Testing with Spy (partial mock)
    @Test
    @DisplayName("Should use real password encoder with Spy")
    void create_UsesRealPasswordEncoder() {
        // 📋 ARRANGE
        UserDTO userDTO = new UserDTO("John", "john@email.com", "password123");
        
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenAnswer(inv -> inv.getArgument(0));
        
        // 🎬 ACT
        User result = userService.create(userDTO);
        
        // ✅ ASSERT - Real BCrypt encoding was used
        assertTrue(result.getPassword().startsWith("$2a$"));
        assertTrue(passwordEncoder.matches("password123", result.getPassword()));
        
        // 🔍 Verify spy was called
        verify(passwordEncoder).encode("password123");
    }
    
    // ✅ Test 13: Verify order of method calls
    @Test
    @DisplayName("Should verify correct order of operations")
    void update_VerifyMethodCallOrder() {
        // 📋 ARRANGE
        Long userId = 1L;
        User existingUser = new User(userId, "John", "old@email.com");
        UserDTO updateDTO = new UserDTO("John", "new@email.com", null);
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(existingUser));
        when(userRepository.existsByEmail("new@email.com")).thenReturn(false);
        when(userRepository.save(any())).thenReturn(existingUser);
        
        // 🎬 ACT
        userService.update(userId, updateDTO);
        
        // ✅ ASSERT - Verify order
        InOrder inOrder = inOrder(userRepository, emailService, auditService);
        inOrder.verify(userRepository).findById(userId);
        inOrder.verify(userRepository).existsByEmail("new@email.com");
        inOrder.verify(emailService).sendEmailChangeNotification(any());
        inOrder.verify(userRepository).save(any());
        inOrder.verify(auditService).logUpdate(anyString(), anyLong(), any());
    }
    
    // ✅ Test 14: Timeout verification
    @Test
    @DisplayName("Should complete within timeout")
    @Timeout(value = 1, unit = TimeUnit.SECONDS)
    void findAll_CompletesWithinTimeout() {
        // 📋 ARRANGE
        when(userRepository.findAll()).thenReturn(Collections.emptyList());
        
        // 🎬 ACT
        List<User> result = userService.findAll();
        
        // ✅ ASSERT
        assertNotNull(result);
        verify(userRepository, timeout(500)).findAll();
    }
    
    // ✅ Test 15: Nested/Grouped tests
    @Nested
    @DisplayName("User Creation Tests")
    class UserCreationTests {
        
        @Test
        @DisplayName("Should create user with valid data")
        void shouldCreateUser() {
            // Test implementation
        }
        
        @Test
        @DisplayName("Should reject duplicate email")
        void shouldRejectDuplicateEmail() {
            // Test implementation
        }
        
        @Nested
        @DisplayName("Validation Tests")
        class ValidationTests {
            
            @Test
            @DisplayName("Should reject empty name")
            void shouldRejectEmptyName() {
                // Test implementation
            }
        }
    }
    
    // ✅ Test 16: Parameterized tests
    @ParameterizedTest
    @ValueSource(strings = {"", " ", "   "})
    @DisplayName("Should reject blank names")
    void create_WithBlankName_ThrowsException(String blankName) {
        // 📋 ARRANGE
        UserDTO userDTO = new UserDTO(blankName, "john@email.com", "password");
        
        // 🎬 ACT & ASSERT
        assertThrows(ValidationException.class, () -> userService.create(userDTO));
    }
    
    @ParameterizedTest
    @CsvSource({
        "john@email.com, true",
        "jane@email.com, true",
        "invalid-email, false"
    })
    @DisplayName("Should validate email format")
    void validateEmail_ReturnsExpectedResult(String email, boolean expected) {
        // 🎬 ACT
        boolean result = userService.isValidEmail(email);
        
        // ✅ ASSERT
        assertEquals(expected, result);
    }
}
```

---

# 💾 Repository Testing

> 💡 **Purpose**: Test database queries and data access logic with actual database operations

## 🟢 Basic Level - CRUD Operations

### 📁 Repository Interface

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByEmail(String email);
    
    boolean existsByEmail(String email);
    
    List<User> findByNameContainingIgnoreCase(String name);
    
    @Query("SELECT u FROM User u WHERE u.createdAt >= :startDate")
    List<User> findUsersCreatedAfter(@Param("startDate") LocalDateTime startDate);
    
    @Query("SELECT u FROM User u WHERE u.status = :status ORDER BY u.name")
    Page<User> findByStatus(@Param("status") UserStatus status, Pageable pageable);
    
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
}
```

### 🧪 Basic Repository Tests

```java
@DataJpaTest  // 🎯 Loads only JPA components, uses H2 by default
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)  // Use real DB (optional)
class UserRepositoryBasicTest {

    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;  // 🔧 For test data setup
    
    // ✅ Test 1: Save and retrieve
    @Test
    @DisplayName("save - Should persist user and generate ID")
    void save_PersistsUserWithGeneratedId() {
        // 📋 ARRANGE
        User user = new User();
        user.setName("John Doe");
        user.setEmail("john@email.com");
        user.setPassword("encoded_password");
        user.setCreatedAt(LocalDateTime.now());
        
        // 🎬 ACT
        User savedUser = userRepository.save(user);
        
        // ✅ ASSERT
        assertNotNull(savedUser.getId());
        assertEquals("John Doe", savedUser.getName());
        
        // 🔍 Verify in database
        User foundUser = entityManager.find(User.class, savedUser.getId());
        assertEquals("john@email.com", foundUser.getEmail());
    }
    
    // ✅ Test 2: Find by email
    @Test
    @DisplayName("findByEmail - Should return user when email exists")
    void findByEmail_WhenExists_ReturnsUser() {
        // 📋 ARRANGE - Using TestEntityManager for isolation
        User user = new User();
        user.setName("John");
        user.setEmail("john@email.com");
        user.setPassword("pass");
        user.setCreatedAt(LocalDateTime.now());
        entityManager.persistAndFlush(user);
        entityManager.clear();  // Clear cache to force DB read
        
        // 🎬 ACT
        Optional<User> result = userRepository.findByEmail("john@email.com");
        
        // ✅ ASSERT
        assertTrue(result.isPresent());
        assertEquals("John", result.get().getName());
    }
    
    // ✅ Test 3: Find by email - Not found
    @Test
    @DisplayName("findByEmail - Should return empty when email not exists")
    void findByEmail_WhenNotExists_ReturnsEmpty() {
        // 🎬 ACT
        Optional<User> result = userRepository.findByEmail("nonexistent@email.com");
        
        // ✅ ASSERT
        assertTrue(result.isEmpty());
    }
    
    // ✅ Test 4: Exists by email
    @Test
    @DisplayName("existsByEmail - Should return true when email exists")
    void existsByEmail_WhenExists_ReturnsTrue() {
        // 📋 ARRANGE
        User user = createAndPersistUser("test@email.com");
        
        // 🎬 ACT & ASSERT
        assertTrue(userRepository.existsByEmail("test@email.com"));
        assertFalse(userRepository.existsByEmail("other@email.com"));
    }
    
    // ✅ Test 5: Find all
    @Test
    @DisplayName("findAll - Should return all users")
    void findAll_ReturnsAllUsers() {
        // 📋 ARRANGE
        createAndPersistUser("user1@email.com");
        createAndPersistUser("user2@email.com");
        createAndPersistUser("user3@email.com");
        
        // 🎬 ACT
        List<User> users = userRepository.findAll();
        
        // ✅ ASSERT
        assertEquals(3, users.size());
    }
    
    // 🔧 Helper method
    private User createAndPersistUser(String email) {
        User user = new User();
        user.setName("Test User");
        user.setEmail(email);
        user.setPassword("password");
        user.setCreatedAt(LocalDateTime.now());
        return entityManager.persistAndFlush(user);
    }
}
```

---

## 🟡 Intermediate Level - Custom Queries

### 🧪 Intermediate Repository Tests

```java
@DataJpaTest
class UserRepositoryIntermediateTest {

    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    private User user1, user2, user3;
    
    @BeforeEach
    void setUp() {
        // 📋 Setup test data
        user1 = createUser("John Doe", "john@email.com", 
            LocalDateTime.of(2024, 1, 15, 10, 0), UserStatus.ACTIVE);
        user2 = createUser("Jane Smith", "jane@email.com", 
            LocalDateTime.of(2024, 2, 20, 10, 0), UserStatus.ACTIVE);
        user3 = createUser("Bob Wilson", "bob@email.com", 
            LocalDateTime.of(2024, 3, 10, 10, 0), UserStatus.INACTIVE);
        
        entityManager.persist(user1);
        entityManager.persist(user2);
        entityManager.persist(user3);
        entityManager.flush();
        entityManager.clear();
    }
    
    // ✅ Test 6: Search by name containing
    @Test
    @DisplayName("findByNameContaining - Should find users with matching name")
    void findByNameContaining_ReturnsMatchingUsers() {
        // 🎬 ACT
        List<User> results = userRepository.findByNameContainingIgnoreCase("john");
        
        // ✅ ASSERT
        assertEquals(1, results.size());
        assertEquals("John Doe", results.get(0).getName());
    }
    
    // ✅ Test 7: Custom query - Find users created after date
    @Test
    @DisplayName("findUsersCreatedAfter - Should return users created after date")
    void findUsersCreatedAfter_ReturnsRecentUsers() {
        // 📋 ARRANGE
        LocalDateTime cutoffDate = LocalDateTime.of(2024, 2, 1, 0, 0);
        
        // 🎬 ACT
        List<User> results = userRepository.findUsersCreatedAfter(cutoffDate);
        
        // ✅ ASSERT
        assertEquals(2, results.size());
        assertTrue(results.stream().allMatch(u -> u.getCreatedAt().isAfter(cutoffDate)));
    }
    
    // ✅ Test 8: Pagination
    @Test
    @DisplayName("findByStatus - Should return paginated results")
    void findByStatus_ReturnsPaginatedResults() {
        // 📋 ARRANGE
        Pageable pageable = PageRequest.of(0, 1, Sort.by("name"));
        
        // 🎬 ACT
        Page<User> result = userRepository.findByStatus(UserStatus.ACTIVE, pageable);
        
        // ✅ ASSERT
        assertEquals(2, result.getTotalElements());  // Total active users
        assertEquals(2, result.getTotalPages());     // 2 pages with size 1
        assertEquals(1, result.getContent().size()); // 1 user per page
        assertEquals("Jane Smith", result.getContent().get(0).getName());  // Sorted by name
    }
    
    // ✅ Test 9: Modifying query
    @Test
    @DisplayName("updateStatus - Should update user status")
    void updateStatus_UpdatesUserStatus() {
        // 📋 ARRANGE
        Long userId = user1.getId();
        
        // 🎬 ACT
        int updatedCount = userRepository.updateStatus(userId, UserStatus.INACTIVE);
        entityManager.clear();  // Clear cache to get fresh data
        
        // ✅ ASSERT
        assertEquals(1, updatedCount);
        User updatedUser = entityManager.find(User.class, userId);
        assertEquals(UserStatus.INACTIVE, updatedUser.getStatus());
    }
    
    // ✅ Test 10: Delete operations
    @Test
    @DisplayName("delete - Should remove user from database")
    void delete_RemovesUser() {
        // 📋 ARRANGE
        Long userId = user1.getId();
        
        // 🎬 ACT
        userRepository.deleteById(userId);
        entityManager.flush();
        
        // ✅ ASSERT
        User deletedUser = entityManager.find(User.class, userId);
        assertNull(deletedUser);
        assertEquals(2, userRepository.count());
    }
    
    private User createUser(String name, String email, LocalDateTime createdAt, UserStatus status) {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        user.setPassword("password");
        user.setCreatedAt(createdAt);
        user.setStatus(status);
        return user;
    }
}
```

---

## 🔴 Advanced Level - Complex Queries, Relationships

### 📁 Entity with Relationships

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
    
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
}
```

### 🧪 Advanced Repository Tests

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Sql(scripts = "/test-data.sql", executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
@Sql(scripts = "/cleanup.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
class UserRepositoryAdvancedTest {

    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    // ✅ Test 11: Fetch with relationships
    @Test
    @DisplayName("Should fetch user with orders using entity graph")
    void findUserWithOrders_UsingEntityGraph() {
        // 📋 ARRANGE
        User user = createUserWithOrders();
        
        // 🎬 ACT
        User fetchedUser = userRepository.findByIdWithOrders(user.getId())
                .orElseThrow();
        
        // ✅ ASSERT - Orders should be loaded
        assertFalse(Hibernate.isInitialized(fetchedUser.getOrders())); // Lazy by default
        assertEquals(3, fetchedUser.getOrders().size());
    }
    
    // ✅ Test 12: Complex aggregation query
    @Test
    @DisplayName("Should calculate user order statistics")
    void findUserOrderStatistics() {
        // 📋 ARRANGE
        User user = createUserWithOrders();
        
        // 🎬 ACT
        UserOrderStats stats = userRepository.findOrderStatsByUserId(user.getId());
        
        // ✅ ASSERT
        assertNotNull(stats);
        assertEquals(3, stats.getOrderCount());
        assertTrue(stats.getTotalAmount().compareTo(BigDecimal.ZERO) > 0);
    }
    
    // ✅ Test 13: Native query test
    @Test
    @DisplayName("Should execute native query correctly")
    void nativeQuery_ReturnsCorrectResults() {
        // 📋 ARRANGE
        createAndPersistMultipleUsers(5);
        
        // 🎬 ACT
        List<Object[]> results = userRepository.findUserSummaryNative();
        
        // ✅ ASSERT
        assertFalse(results.isEmpty());
        Object[] firstRow = results.get(0);
        assertNotNull(firstRow[0]); // ID
        assertNotNull(firstRow[1]); // Name
    }
    
    // ✅ Test 14: Specification test
    @Test
    @DisplayName("Should filter users using Specification")
    void findAll_WithSpecification_FiltersCorrectly() {
        // 📋 ARRANGE
        createAndPersistUsersWithDifferentStatuses();
        
        Specification<User> spec = (root, query, cb) -> {
            List<Predicate> predicates = new ArrayList<>();
            predicates.add(cb.equal(root.get("status"), UserStatus.ACTIVE));
            predicates.add(cb.like(root.get("name"), "%John%"));
            return cb.and(predicates.toArray(new Predicate[0]));
        };
        
        // 🎬 ACT
        List<User> results = userRepository.findAll(spec);
        
        // ✅ ASSERT
        assertTrue(results.stream()
            .allMatch(u -> u.getStatus() == UserStatus.ACTIVE && u.getName().contains("John")));
    }
    
    // ✅ Test 15: Auditing test
    @Test
    @DisplayName("Should auto-populate audit fields")
    void save_PopulatesAuditFields() {
        // 📋 ARRANGE
        User user = new User();
        user.setName("Test User");
        user.setEmail("test@email.com");
        
        // 🎬 ACT
        User savedUser = userRepository.save(user);
        entityManager.flush();
        
        // ✅ ASSERT
        assertNotNull(savedUser.getCreatedAt());
        assertNotNull(savedUser.getCreatedBy());
    }
    
    // ✅ Test 16: Transaction rollback test
    @Test
    @DisplayName("Should rollback on exception")
    @Transactional
    void transaction_RollsBackOnException() {
        // 📋 ARRANGE
        User user = createAndPersistUser("rollback@test.com");
        long initialCount = userRepository.count();
        
        // 🎬 ACT & ASSERT
        assertThrows(DataIntegrityViolationException.class, () -> {
            User duplicateEmailUser = new User();
            duplicateEmailUser.setEmail("rollback@test.com"); // Duplicate email
            userRepository.saveAndFlush(duplicateEmailUser);
        });
        
        // Verify rollback
        assertEquals(initialCount, userRepository.count());
    }
    
    // ✅ Test 17: Query performance test
    @Test
    @DisplayName("Should execute query within acceptable time")
    void query_CompletesWithinTimeout() {
        // 📋 ARRANGE
        createAndPersistMultipleUsers(1000);
        
        // 🎬 ACT & ASSERT
        assertTimeout(Duration.ofMillis(500), () -> {
            Page<User> results = userRepository.findAll(PageRequest.of(0, 10));
            assertNotNull(results);
        });
    }
    
    private User createUserWithOrders() {
        User user = new User();
        user.setName("Test User");
        user.setEmail("test@email.com");
        user.setPassword("password");
        
        for (int i = 0; i < 3; i++) {
            Order order = new Order();
            order.setTotal(new BigDecimal("100.00"));
            order.setUser(user);
            user.getOrders().add(order);
        }
        
        return entityManager.persistAndFlush(user);
    }
}
```

---

# 🔗 Integration Testing

> 💡 **Purpose**: Test complete request flow from controller to database

## 🟢 Basic Level - Full Stack Test

### 🧪 Basic Integration Tests

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY)
@TestPropertySource(locations = "classpath:application-test.properties")
class UserIntegrationBasicTest {

    @Autowired
    private TestRestTemplate restTemplate;  // 🔧 Real HTTP client
    
    @Autowired
    private UserRepository userRepository;
    
    @LocalServerPort
    private int port;
    
    private String baseUrl;
    
    @BeforeEach
    void setUp() {
        baseUrl = "http://localhost:" + port + "/api/users";
        userRepository.deleteAll();  // Clean state
    }
    
    // ✅ Test 1: Full create and retrieve flow
    @Test
    @DisplayName("Should create user and retrieve it")
    void createAndRetrieve_FullFlow() {
        // 📋 ARRANGE
        UserDTO createRequest = new UserDTO("John Doe", "john@email.com", "password123");
        
        // 🎬 ACT - Create user
        ResponseEntity<User> createResponse = restTemplate.postForEntity(
            baseUrl,
            createRequest,
            User.class
        );
        
        // ✅ ASSERT - Creation
        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        assertNotNull(createResponse.getBody());
        Long userId = createResponse.getBody().getId();
        
        // 🎬 ACT - Retrieve user
        ResponseEntity<User> getResponse = restTemplate.getForEntity(
            baseUrl + "/" + userId,
            User.class
        );
        
        // ✅ ASSERT - Retrieval
        assertEquals(HttpStatus.OK, getResponse.getStatusCode());
        assertEquals("John Doe", getResponse.getBody().getName());
        
        // 🔍 Verify in database
        assertTrue(userRepository.existsById(userId));
    }
    
    // ✅ Test 2: List all users
    @Test
    @DisplayName("Should return list of all users")
    void getAllUsers_ReturnsAllUsersFromDatabase() {
        // 📋 ARRANGE - Create test data directly in DB
        userRepository.save(new User(null, "User 1", "user1@email.com", "pass", LocalDateTime.now()));
        userRepository.save(new User(null, "User 2", "user2@email.com", "pass", LocalDateTime.now()));
        
        // 🎬 ACT
        ResponseEntity<User[]> response = restTemplate.getForEntity(
            baseUrl,
            User[].class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals(2, response.getBody().length);
    }
    
    // ❌ Test 3: Get non-existent user
    @Test
    @DisplayName("Should return 404 for non-existent user")
    void getUser_WhenNotExists_Returns404() {
        // 🎬 ACT
        ResponseEntity<ErrorResponse> response = restTemplate.getForEntity(
            baseUrl + "/999",
            ErrorResponse.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
        assertNotNull(response.getBody());
        assertTrue(response.getBody().getMessage().contains("not found"));
    }
}
```

---

## 🟡 Intermediate Level - Using MockMvc with Full Context

### 🧪 Intermediate Integration Tests

```java
@SpringBootTest
@AutoConfigureMockMvc  // 🎯 MockMvc with full Spring context
@Transactional  // 🔄 Rollback after each test
class UserIntegrationIntermediateTest {

    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    // ✅ Test 4: Complete CRUD operations
    @Test
    @DisplayName("Should perform complete CRUD lifecycle")
    void crudLifecycle_CompletesSuccessfully() throws Exception {
        // 📝 CREATE
        UserDTO createDTO = new UserDTO("John Doe", "john@email.com", "password123");
        
        MvcResult createResult = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(createDTO)))
                .andExpect(status().isCreated())
                .andReturn();
        
        User createdUser = objectMapper.readValue(
            createResult.getResponse().getContentAsString(), 
            User.class
        );
        Long userId = createdUser.getId();
        
        // 📖 READ
        mockMvc.perform(get("/api/users/{id}", userId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Doe"));
        
        // ✏️ UPDATE
        UserDTO updateDTO = new UserDTO("John Updated", "john.updated@email.com", null);
        
        mockMvc.perform(put("/api/users/{id}", userId)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updateDTO)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Updated"));
        
        // 🗑️ DELETE
        mockMvc.perform(delete("/api/users/{id}", userId))
                .andExpect(status().isNoContent());
        
        // ✅ Verify deletion
        mockMvc.perform(get("/api/users/{id}", userId))
                .andExpect(status().isNotFound());
    }
    
    // ✅ Test 5: Validation in full context
    @Test
    @DisplayName("Should validate request body with all validators")
    void create_WithInvalidData_ReturnsValidationErrors() throws Exception {
        // 📋 ARRANGE - Multiple validation errors
        UserDTO invalidDTO = new UserDTO("", "not-an-email", "123");  // short password
        
        // 🎬 ACT & ASSERT
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidDTO)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errors").isArray())
                .andExpect(jsonPath("$.errors.length()").value(3));
    }
    
    // ✅ Test 6: Test with database state verification
    @Test
    @DisplayName("Should persist correct data to database")
    void create_VerifiesDatabaseState() throws Exception {
        // 📋 ARRANGE
        UserDTO createDTO = new UserDTO("Jane Doe", "jane@email.com", "password123");
        
        // 🎬 ACT
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(createDTO)))
                .andExpect(status().isCreated());
        
        // ✅ ASSERT - Verify database state
        Optional<User> savedUser = userRepository.findByEmail("jane@email.com");
        
        assertTrue(savedUser.isPresent());
        assertEquals("Jane Doe", savedUser.get().getName());
        assertTrue(passwordEncoder.matches("password123", savedUser.get().getPassword()));
        assertNotNull(savedUser.get().getCreatedAt());
    }
    
    // ✅ Test 7: Concurrent request handling
    @Test
    @DisplayName("Should handle concurrent requests correctly")
    void concurrent_HandlesSafely() throws Exception {
        // 📋 ARRANGE
        int threadCount = 10;
        ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(threadCount);
        AtomicInteger successCount = new AtomicInteger(0);
        
        // 🎬 ACT
        for (int i = 0; i < threadCount; i++) {
            final int index = i;
            executorService.submit(() -> {
                try {
                    UserDTO dto = new UserDTO(
                        "User " + index,
                        "user" + index + "@email.com",
                        "password"
                    );
                    
                    mockMvc.perform(post("/api/users")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(dto)))
                            .andExpect(status().isCreated());
                    
                    successCount.incrementAndGet();
                } catch (Exception e) {
                    // Handle exception
                } finally {
                    latch.countDown();
                }
            });
        }
        
        latch.await(10, TimeUnit.SECONDS);
        
        // ✅ ASSERT
        assertEquals(threadCount, successCount.get());
        assertEquals(threadCount, userRepository.count());
    }
}
```

---

## 🔴 Advanced Level - TestContainers, Security, External Services

### 🧪 Advanced Integration Tests

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers  // 🐳 Docker containers for testing
@ActiveProfiles("integration-test")
class UserIntegrationAdvancedTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:14")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6")
            .withExposedPorts(6379);
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", () -> redis.getMappedPort(6379));
    }
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @MockBean
    private ExternalPaymentService paymentService;  // 🎭 Mock external service
    
    @LocalServerPort
    private int port;
    
    // ✅ Test 8: Full integration with real PostgreSQL
    @Test
    @DisplayName("Should work with real PostgreSQL container")
    void testWithRealDatabase() {
        // 📋 ARRANGE
        UserDTO dto = new UserDTO("Docker User", "docker@test.com", "password");
        
        // 🎬 ACT
        ResponseEntity<User> response = restTemplate.postForEntity(
            "/api/users",
            dto,
            User.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        
        // Verify directly with PostgreSQL
        Optional<User> dbUser = userRepository.findByEmail("docker@test.com");
        assertTrue(dbUser.isPresent());
    }
    
    // ✅ Test 9: Test with security
    @Test
    @DisplayName("Should require authentication for protected endpoints")
    void protectedEndpoint_RequiresAuth() {
        // 🎬 ACT - Without auth
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/api/admin/users",
            String.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
    }
    
    @Test
    @DisplayName("Should allow access with valid JWT token")
    @WithMockUser(roles = "ADMIN")
    void protectedEndpoint_WithValidToken_AllowsAccess() throws Exception {
        // 📋 ARRANGE
        String token = generateTestJwtToken("admin", "ROLE_ADMIN");
        
        // 🎬 ACT
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        HttpEntity<Void> request = new HttpEntity<>(headers);
        
        ResponseEntity<User[]> response = restTemplate.exchange(
            "/api/admin/users",
            HttpMethod.GET,
            request,
            User[].class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
    
    // ✅ Test 10: Mock external service in integration test
    @Test
    @DisplayName("Should handle external service in integration")
    void createOrder_WithPaymentService() {
        // 📋 ARRANGE
        when(paymentService.processPayment(any())).thenReturn(new PaymentResult(true, "TXN123"));
        
        // First create a user
        User user = userRepository.save(new User(null, "Test", "test@email.com", "pass", LocalDateTime.now()));
        
        OrderDTO orderDTO = new OrderDTO(user.getId(), new BigDecimal("99.99"));
        
        // 🎬 ACT
        ResponseEntity<Order> response = restTemplate.postForEntity(
            "/api/orders",
            orderDTO,
            Order.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertEquals("TXN123", response.getBody().getTransactionId());
        verify(paymentService).processPayment(any());
    }
    
    // ✅ Test 11: Test caching with Redis
    @Test
    @DisplayName("Should cache user data in Redis")
    void getUser_CachesResult() {
        // 📋 ARRANGE
        User user = userRepository.save(new User(null, "Cached User", "cache@test.com", "pass", LocalDateTime.now()));
        
        // 🎬 ACT - First call (cache miss)
        restTemplate.getForEntity("/api/users/" + user.getId(), User.class);
        
        // Second call (should hit cache)
        restTemplate.getForEntity("/api/users/" + user.getId(), User.class);
        
        // ✅ ASSERT - Verify cache was used (repository called only once)
        // This would require additional instrumentation
    }
    
    // ✅ Test 12: API documentation test
    @Test
    @DisplayName("Should expose OpenAPI documentation")
    void openApiDocs_AreAvailable() {
        // 🎬 ACT
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/v3/api-docs",
            String.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertTrue(response.getBody().contains("openapi"));
    }
    
    // ✅ Test 13: Health check test
    @Test
    @DisplayName("Should report healthy status")
    void healthCheck_ReportsHealthy() {
        // 🎬 ACT
        ResponseEntity<Map> response = restTemplate.getForEntity(
            "/actuator/health",
            Map.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals("UP", response.getBody().get("status"));
    }
    
    // ✅ Test 14: End-to-end workflow test
    @Test
    @DisplayName("Should complete full user registration workflow")
    void userRegistrationWorkflow_CompletesSuccessfully() {
        // 📋 ARRANGE
        UserRegistrationRequest registrationRequest = new UserRegistrationRequest(
            "New User",
            "newuser@email.com",
            "securePassword123",
            "securePassword123"
        );
        
        // 🎬 ACT - Step 1: Register
        ResponseEntity<RegistrationResponse> registerResponse = restTemplate.postForEntity(
            "/api/auth/register",
            registrationRequest,
            RegistrationResponse.class
        );
        
        assertEquals(HttpStatus.OK, registerResponse.getStatusCode());
        String verificationToken = registerResponse.getBody().getVerificationToken();
        
        // 🎬 ACT - Step 2: Verify email
        restTemplate.getForEntity(
            "/api/auth/verify?token=" + verificationToken,
            Void.class
        );
        
        // 🎬 ACT - Step 3: Login
        LoginRequest loginRequest = new LoginRequest("newuser@email.com", "securePassword123");
        ResponseEntity<LoginResponse> loginResponse = restTemplate.postForEntity(
            "/api/auth/login",
            loginRequest,
            LoginResponse.class
        );
        
        assertEquals(HttpStatus.OK, loginResponse.getStatusCode());
        assertNotNull(loginResponse.getBody().getAccessToken());
        
        // 🎬 ACT - Step 4: Access protected resource
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(loginResponse.getBody().getAccessToken());
        
        ResponseEntity<User> profileResponse = restTemplate.exchange(
            "/api/users/profile",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            User.class
        );
        
        // ✅ ASSERT
        assertEquals(HttpStatus.OK, profileResponse.getStatusCode());
        assertEquals("New User", profileResponse.getBody().getName());
    }
}
```

---

# ❓ Interview Questions & Answers

## 🎯 Basic Questions

### ❓ Q1: What is the difference between @Mock and @MockBean?

```java
// 🎭 @Mock - Plain Mockito mock (unit tests)
@ExtendWith(MockitoExtension.class)
class ServiceUnitTest {
    @Mock
    private UserRepository userRepository;  // Mockito-managed mock
    
    @InjectMocks
    private UserService userService;  // Injects mocks
}

// 🌱 @MockBean - Spring-managed mock (integration tests)
@SpringBootTest
class ServiceIntegrationTest {
    @MockBean
    private UserRepository userRepository;  // Replaces bean in context
    
    @Autowired
    private UserService userService;  // Uses mocked repository
}
```

> 💡 **Simple Explanation**: `@Mock` creates a standalone mock for unit tests, while `@MockBean` replaces a Spring bean in the application context for integration tests.

---

### ❓ Q2: What is the difference between @WebMvcTest and @SpringBootTest?

```java
// 🎯 @WebMvcTest - Only web layer (controllers)
@WebMvcTest(UserController.class)
class ControllerSliceTest {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;  // Must mock services
}

// 🌐 @SpringBootTest - Full application context
@SpringBootTest
@AutoConfigureMockMvc
class FullContextTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserService userService;  // Real service
}
```

> 💡 **Simple Explanation**: `@WebMvcTest` loads only web-related beans for fast controller testing, while `@SpringBootTest` loads the entire application for complete integration testing.

---

### ❓ Q3: How do you verify method calls with Mockito?

```java
@Test
void verifyMethodCalls() {
    // 📋 ARRANGE
    when(userRepository.save(any(User.class))).thenReturn(new User());
    
    // 🎬 ACT
    userService.create(new UserDTO("John", "john@email.com", "pass"));
    
    // ✅ VERIFY - Basic
    verify(userRepository).save(any(User.class));           // Called at least once
    verify(userRepository, times(1)).save(any(User.class)); // Called exactly once
    verify(userRepository, never()).delete(any());          // Never called
    verify(userRepository, atLeast(1)).save(any());         // Called 1+ times
    verify(userRepository, atMost(3)).save(any());          // Called max 3 times
    
    // ✅ VERIFY - With argument capture
    ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
    verify(userRepository).save(captor.capture());
    assertEquals("John", captor
