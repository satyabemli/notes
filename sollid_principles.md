# 📝 Single Responsibility Principle - Real-Time Deep Dive

---

## 🎯 SRP Core Concept

### Definition
> **"A class should have only ONE reason to change"**

### What It Really Means 💭

```java
/**
 * ONE REASON TO CHANGE = ONE RESPONSIBILITY
 * 
 * ❌ Multiple Reasons to Change:
 * - Business logic changes
 * - Database schema changes
 * - Email template changes
 * - Report format changes
 * - All in ONE class = VIOLATION!
 * 
 * ✅ Single Reason to Change:
 * - Only business logic changes affect this class
 * - Other concerns are in separate classes
 */
```

### The "Boss Test" 👨‍💼

**Ask yourself:** *"Who would request changes to this class?"*

- If **multiple stakeholders** (CTO, Marketing Manager, DBA, etc.) → ❌ **SRP Violated**
- If **one stakeholder** → ✅ **SRP Followed**

---

## 🛒 Scenario 1: User Registration System

### ❌ BAD Example - God Class Anti-Pattern

```java
/**
 * 🚫 VIOLATION: This class has TOO MANY responsibilities!
 * Changes from multiple sources would require modifying this class
 */

@RestController
public class UserRegistrationController {
    
    /**
     * RESPONSIBILITIES IN ONE CLASS:
     * 1. HTTP Request Handling
     * 2. Input Validation
     * 3. Business Logic
     * 4. Password Encryption
     * 5. Database Operations
     * 6. Email Sending
     * 7. SMS Sending
     * 8. Logging
     * 9. Analytics Tracking
     * 10. Promotional Code Validation
     */
    
    @PostMapping("/register")
    public ResponseEntity<String> registerUser(
            @RequestBody Map<String, String> userData) {
        
        try {
            // 🔥 RESPONSIBILITY 1: Input Validation
            String email = userData.get("email");
            String password = userData.get("password");
            String phone = userData.get("phone");
            String promoCode = userData.get("promoCode");
            
            if (email == null || email.isEmpty()) {
                return ResponseEntity.badRequest().body("Email required");
            }
            
            if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
                return ResponseEntity.badRequest().body("Invalid email format");
            }
            
            if (password == null || password.length() < 8) {
                return ResponseEntity.badRequest().body("Password must be 8+ chars");
            }
            
            if (phone == null || !phone.matches("^\\d{10}$")) {
                return ResponseEntity.badRequest().body("Invalid phone number");
            }
            
            // 🔥 RESPONSIBILITY 2: Database - Check existing user
            Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/ecommerce", "root", "password");
            
            PreparedStatement checkStmt = conn.prepareStatement(
                "SELECT * FROM users WHERE email = ?");
            checkStmt.setString(1, email);
            ResultSet rs = checkStmt.executeQuery();
            
            if (rs.next()) {
                conn.close();
                return ResponseEntity.badRequest().body("User already exists");
            }
            
            // 🔥 RESPONSIBILITY 3: Password Encryption
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] hashedPassword = md.digest(password.getBytes());
            String encryptedPassword = Base64.getEncoder().encodeToString(hashedPassword);
            
            // 🔥 RESPONSIBILITY 4: Promo Code Validation
            double discount = 0;
            if (promoCode != null && !promoCode.isEmpty()) {
                PreparedStatement promoStmt = conn.prepareStatement(
                    "SELECT discount FROM promo_codes WHERE code = ? AND active = true");
                promoStmt.setString(1, promoCode);
                ResultSet promoRs = promoStmt.executeQuery();
                
                if (promoRs.next()) {
                    discount = promoRs.getDouble("discount");
                } else {
                    return ResponseEntity.badRequest().body("Invalid promo code");
                }
            }
            
            // 🔥 RESPONSIBILITY 5: Business Logic - Create User
            String userId = UUID.randomUUID().toString();
            String accountType = discount > 0 ? "PREMIUM" : "REGULAR";
            
            // 🔥 RESPONSIBILITY 6: Database - Insert User
            PreparedStatement insertStmt = conn.prepareStatement(
                "INSERT INTO users (id, email, password, phone, account_type, discount, created_at) " +
                "VALUES (?, ?, ?, ?, ?, ?, ?)");
            insertStmt.setString(1, userId);
            insertStmt.setString(2, email);
            insertStmt.setString(3, encryptedPassword);
            insertStmt.setString(4, phone);
            insertStmt.setString(5, accountType);
            insertStmt.setDouble(6, discount);
            insertStmt.setTimestamp(7, new Timestamp(System.currentTimeMillis()));
            insertStmt.executeUpdate();
            
            // 🔥 RESPONSIBILITY 7: Email Sending
            Properties props = new Properties();
            props.put("mail.smtp.host", "smtp.gmail.com");
            props.put("mail.smtp.port", "587");
            props.put("mail.smtp.auth", "true");
            props.put("mail.smtp.starttls.enable", "true");
            
            Session session = Session.getInstance(props, new Authenticator() {
                protected PasswordAuthentication getPasswordAuthentication() {
                    return new PasswordAuthentication("noreply@ecommerce.com", "emailpassword");
                }
            });
            
            Message message = new MimeMessage(session);
            message.setFrom(new InternetAddress("noreply@ecommerce.com"));
            message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(email));
            message.setSubject("Welcome to E-Commerce!");
            
            String emailBody = "<html><body>" +
                "<h1>Welcome!</h1>" +
                "<p>Thank you for registering. Your account type: " + accountType + "</p>" +
                "<p>Start shopping now!</p>" +
                "</body></html>";
            message.setContent(emailBody, "text/html");
            
            Transport.send(message);
            
            // 🔥 RESPONSIBILITY 8: SMS Sending
            String smsUrl = "https://sms-api.com/send";
            HttpClient httpClient = HttpClient.newHttpClient();
            String smsBody = String.format(
                "{\"to\":\"%s\", \"message\":\"Welcome to E-Commerce! Your account is ready.\"}", 
                phone);
            
            HttpRequest smsRequest = HttpRequest.newBuilder()
                .uri(URI.create(smsUrl))
                .header("Content-Type", "application/json")
                .header("Authorization", "Bearer SMS_API_KEY")
                .POST(HttpRequest.BodyPublishers.ofString(smsBody))
                .build();
            
            httpClient.send(smsRequest, HttpResponse.BodyHandlers.ofString());
            
            // 🔥 RESPONSIBILITY 9: Logging
            FileWriter logWriter = new FileWriter("user_registrations.log", true);
            logWriter.write(String.format(
                "[%s] User registered: %s, Account: %s\n",
                LocalDateTime.now(), email, accountType));
            logWriter.close();
            
            // 🔥 RESPONSIBILITY 10: Analytics Tracking
            String analyticsUrl = "https://analytics.com/track";
            String analyticsBody = String.format(
                "{\"event\":\"user_registration\", \"userId\":\"%s\", \"accountType\":\"%s\", \"discount\":%.2f}",
                userId, accountType, discount);
            
            HttpRequest analyticsRequest = HttpRequest.newBuilder()
                .uri(URI.create(analyticsUrl))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(analyticsBody))
                .build();
            
            httpClient.send(analyticsRequest, HttpResponse.BodyHandlers.ofString());
            
            // 🔥 RESPONSIBILITY 11: Welcome Bonus Logic
            if (discount > 0) {
                PreparedStatement bonusStmt = conn.prepareStatement(
                    "INSERT INTO user_credits (user_id, amount, reason) VALUES (?, ?, ?)");
                bonusStmt.setString(1, userId);
                bonusStmt.setDouble(2, 10.0);
                bonusStmt.setString(3, "Welcome bonus");
                bonusStmt.executeUpdate();
            }
            
            conn.close();
            
            return ResponseEntity.ok("User registered successfully!");
            
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("Registration failed: " + e.getMessage());
        }
    }
}
```

### 🔥 Problems with Above Code

| Problem | Impact | Who Wants Change |
|---------|--------|------------------|
| **Email template changes** | Need to modify registration class | 📧 Marketing Team |
| **SMS provider changes** | Need to modify registration class | 📱 Operations Team |
| **Database schema changes** | Need to modify registration class | 🗄️ Database Admin |
| **Validation rules change** | Need to modify registration class | 👨‍💼 Business Analyst |
| **Encryption algorithm changes** | Need to modify registration class | 🔐 Security Team |
| **Analytics platform changes** | Need to modify registration class | 📊 Data Team |
| **Promo code logic changes** | Need to modify registration class | 💰 Finance Team |

**Result:** 
- 🔥 **7+ different teams** need to touch the same file!
- 💥 **Merge conflicts** constantly
- 🐛 **High bug risk** - changing email breaks password logic
- 🧪 **Impossible to test** - too many dependencies
- 📚 **300+ lines** in one method
- 😱 **Maintenance nightmare**

---

### ✅ GOOD Example - Proper SRP

```java
/**
 * ✨ SOLUTION: Separate Responsibilities into Focused Classes
 * Each class has ONE reason to change
 */

// ========================================
// 1️⃣ VALIDATION LAYER - One responsibility: Validate input
// ========================================

@Component
public class UserRegistrationValidator {
    
    private static final String EMAIL_REGEX = "^[A-Za-z0-9+_.-]+@(.+)$";
    private static final String PHONE_REGEX = "^\\d{10}$";
    private static final int MIN_PASSWORD_LENGTH = 8;
    
    /**
     * Single Responsibility: Validate user registration data
     * Changes only when validation rules change
     */
    public ValidationResult validate(UserRegistrationRequest request) {
        List<String> errors = new ArrayList<>();
        
        if (request.getEmail() == null || request.getEmail().isEmpty()) {
            errors.add("Email is required");
        } else if (!request.getEmail().matches(EMAIL_REGEX)) {
            errors.add("Invalid email format");
        }
        
        if (request.getPassword() == null || 
            request.getPassword().length() < MIN_PASSWORD_LENGTH) {
            errors.add("Password must be at least " + MIN_PASSWORD_LENGTH + " characters");
        }
        
        if (request.getPhone() == null || !request.getPhone().matches(PHONE_REGEX)) {
            errors.add("Phone number must be 10 digits");
        }
        
        return new ValidationResult(errors.isEmpty(), errors);
    }
}

// Validation result object
public class ValidationResult {
    private final boolean valid;
    private final List<String> errors;
    
    public ValidationResult(boolean valid, List<String> errors) {
        this.valid = valid;
        this.errors = errors;
    }
    
    public boolean isValid() { return valid; }
    public List<String> getErrors() { return errors; }
}

// ========================================
// 2️⃣ SECURITY LAYER - One responsibility: Password encryption
// ========================================

@Component
public class PasswordEncryptionService {
    
    /**
     * Single Responsibility: Encrypt passwords
     * Changes only when encryption algorithm changes
     */
    public String encrypt(String plainPassword) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] hashedPassword = md.digest(plainPassword.getBytes());
            return Base64.getEncoder().encodeToString(hashedPassword);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("Encryption failed", e);
        }
    }
    
    public boolean verify(String plainPassword, String encryptedPassword) {
        String encrypted = encrypt(plainPassword);
        return encrypted.equals(encryptedPassword);
    }
}

// ========================================
// 3️⃣ DATA ACCESS LAYER - One responsibility: Database operations
// ========================================

@Repository
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * Single Responsibility: User data persistence
     * Changes only when database schema changes
     */
    public Optional<User> findByEmail(String email) {
        String sql = "SELECT * FROM users WHERE email = ?";
        
        try {
            User user = jdbcTemplate.queryForObject(sql, 
                new Object[]{email}, 
                new UserRowMapper());
            return Optional.ofNullable(user);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
    
    public User save(User user) {
        String sql = "INSERT INTO users (id, email, password, phone, account_type, " +
                    "discount, created_at) VALUES (?, ?, ?, ?, ?, ?, ?)";
        
        jdbcTemplate.update(sql,
            user.getId(),
            user.getEmail(),
            user.getPassword(),
            user.getPhone(),
            user.getAccountType(),
            user.getDiscount(),
            user.getCreatedAt()
        );
        
        return user;
    }
    
    public boolean existsByEmail(String email) {
        String sql = "SELECT COUNT(*) FROM users WHERE email = ?";
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class, email);
        return count != null && count > 0;
    }
}

// ========================================
// 4️⃣ PROMO CODE SERVICE - One responsibility: Promo validation
// ========================================

@Service
public class PromoCodeService {
    
    @Autowired
    private PromoCodeRepository promoCodeRepository;
    
    /**
     * Single Responsibility: Validate and apply promo codes
     * Changes only when promo code business rules change
     */
    public PromoCodeResult validateAndApply(String code) {
        if (code == null || code.isEmpty()) {
            return PromoCodeResult.empty();
        }
        
        Optional<PromoCode> promoCode = promoCodeRepository.findByCodeAndActive(code, true);
        
        if (promoCode.isPresent()) {
            PromoCode promo = promoCode.get();
            return new PromoCodeResult(true, promo.getDiscount(), promo.getAccountType());
        }
        
        return new PromoCodeResult(false, 0, null);
    }
}

@Repository
public interface PromoCodeRepository extends JpaRepository<PromoCode, Long> {
    Optional<PromoCode> findByCodeAndActive(String code, boolean active);
}

// ========================================
// 5️⃣ EMAIL SERVICE - One responsibility: Send emails
// ========================================

@Service
public class EmailService {
    
    @Value("${email.from}")
    private String fromEmail;
    
    @Autowired
    private JavaMailSender mailSender;
    
    /**
     * Single Responsibility: Send emails
     * Changes only when email provider/template changes
     */
    public void sendWelcomeEmail(User user) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
            
            helper.setFrom(fromEmail);
            helper.setTo(user.getEmail());
            helper.setSubject("Welcome to E-Commerce!");
            
            String emailBody = buildWelcomeEmailBody(user);
            helper.setText(emailBody, true);
            
            mailSender.send(message);
        } catch (MessagingException e) {
            throw new RuntimeException("Failed to send email", e);
        }
    }
    
    private String buildWelcomeEmailBody(User user) {
        return "<html><body>" +
               "<h1>Welcome, " + user.getEmail() + "!</h1>" +
               "<p>Your account type: " + user.getAccountType() + "</p>" +
               "<p>Start shopping now!</p>" +
               "</body></html>";
    }
}

// ========================================
// 6️⃣ SMS SERVICE - One responsibility: Send SMS
// ========================================

@Service
public class SMSService {
    
    @Value("${sms.api.url}")
    private String smsApiUrl;
    
    @Value("${sms.api.key}")
    private String smsApiKey;
    
    private final HttpClient httpClient = HttpClient.newHttpClient();
    
    /**
     * Single Responsibility: Send SMS messages
     * Changes only when SMS provider changes
     */
    public void sendWelcomeSMS(String phoneNumber) {
        try {
            String message = "Welcome to E-Commerce! Your account is ready.";
            
            String requestBody = String.format(
                "{\"to\":\"%s\", \"message\":\"%s\"}", 
                phoneNumber, message);
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(smsApiUrl))
                .header("Content-Type", "application/json")
                .header("Authorization", "Bearer " + smsApiKey)
                .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
            
            httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            
        } catch (Exception e) {
            throw new RuntimeException("Failed to send SMS", e);
        }
    }
}

// ========================================
// 7️⃣ ANALYTICS SERVICE - One responsibility: Track events
// ========================================

@Service
public class AnalyticsService {
    
    @Value("${analytics.api.url}")
    private String analyticsUrl;
    
    private final HttpClient httpClient = HttpClient.newHttpClient();
    
    /**
     * Single Responsibility: Track analytics events
     * Changes only when analytics platform changes
     */
    public void trackUserRegistration(User user) {
        try {
            String eventData = String.format(
                "{\"event\":\"user_registration\", \"userId\":\"%s\", " +
                "\"accountType\":\"%s\", \"discount\":%.2f}",
                user.getId(), user.getAccountType(), user.getDiscount());
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(analyticsUrl))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(eventData))
                .build();
            
            httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString());
            
        } catch (Exception e) {
            // Don't fail registration if analytics fails
            System.err.println("Analytics tracking failed: " + e.getMessage());
        }
    }
}

// ========================================
// 8️⃣ LOGGING SERVICE - One responsibility: Application logging
// ========================================

@Service
public class AuditLogService {
    
    private static final Logger logger = LoggerFactory.getLogger(AuditLogService.class);
    
    /**
     * Single Responsibility: Log important business events
     * Changes only when logging requirements change
     */
    public void logUserRegistration(User user) {
        logger.info("User registered - Email: {}, Account Type: {}, Timestamp: {}",
            user.getEmail(),
            user.getAccountType(),
            user.getCreatedAt()
        );
    }
}

// ========================================
// 9️⃣ BUSINESS LOGIC LAYER - One responsibility: User registration orchestration
// ========================================

@Service
@Transactional
public class UserRegistrationService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncryptionService passwordService;
    
    @Autowired
    private PromoCodeService promoCodeService;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private SMSService smsService;
    
    @Autowired
    private AnalyticsService analyticsService;
    
    @Autowired
    private AuditLogService auditLogService;
    
    /**
     * Single Responsibility: Orchestrate user registration process
     * Changes only when registration business flow changes
     */
    public User registerUser(UserRegistrationRequest request) {
        
        // Check if user already exists
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new UserAlreadyExistsException("User with email already exists");
        }
        
        // Encrypt password
        String encryptedPassword = passwordService.encrypt(request.getPassword());
        
        // Validate and apply promo code
        PromoCodeResult promoResult = promoCodeService.validateAndApply(request.getPromoCode());
        
        // Create user entity
        User user = User.builder()
            .id(UUID.randomUUID().toString())
            .email(request.getEmail())
            .password(encryptedPassword)
            .phone(request.getPhone())
            .accountType(promoResult.getAccountType() != null ? 
                        promoResult.getAccountType() : "REGULAR")
            .discount(promoResult.getDiscount())
            .createdAt(LocalDateTime.now())
            .build();
        
        // Save to database
        User savedUser = userRepository.save(user);
        
        // Send notifications asynchronously (non-blocking)
        CompletableFuture.runAsync(() -> emailService.sendWelcomeEmail(savedUser));
        CompletableFuture.runAsync(() -> smsService.sendWelcomeSMS(savedUser.getPhone()));
        
        // Track analytics
        analyticsService.trackUserRegistration(savedUser);
        
        // Audit log
        auditLogService.logUserRegistration(savedUser);
        
        return savedUser;
    }
}

// ========================================
// 🔟 CONTROLLER LAYER - One responsibility: HTTP request handling
// ========================================

@RestController
@RequestMapping("/api/users")
public class UserRegistrationController {
    
    @Autowired
    private UserRegistrationValidator validator;
    
    @Autowired
    private UserRegistrationService registrationService;
    
    /**
     * Single Responsibility: Handle HTTP requests for user registration
     * Changes only when API contract changes
     */
    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody UserRegistrationRequest request) {
        
        // Validate input
        ValidationResult validationResult = validator.validate(request);
        if (!validationResult.isValid()) {
            return ResponseEntity
                .badRequest()
                .body(new ErrorResponse(validationResult.getErrors()));
        }
        
        try {
            // Delegate to service
            User user = registrationService.registerUser(request);
            
            // Return success response
            return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(new UserRegistrationResponse(user));
                
        } catch (UserAlreadyExistsException e) {
            return ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(new ErrorResponse(e.getMessage()));
                
        } catch (Exception e) {
            return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ErrorResponse("Registration failed"));
        }
    }
}

// ========================================
// 📋 DTOs - Data Transfer Objects
// ========================================

@Data
@Builder
public class UserRegistrationRequest {
    private String email;
    private String password;
    private String phone;
    private String promoCode;
}

@Data
public class UserRegistrationResponse {
    private String userId;
    private String email;
    private String accountType;
    private LocalDateTime registeredAt;
    
    public UserRegistrationResponse(User user) {
        this.userId = user.getId();
        this.email = user.getEmail();
        this.accountType = user.getAccountType();
        this.registeredAt = user.getCreatedAt();
    }
}

@Data
@AllArgsConstructor
public class ErrorResponse {
    private List<String> errors;
    
    public ErrorResponse(String error) {
        this.errors = Collections.singletonList(error);
    }
}
```

---

### ✨ Benefits of SRP Design

| Aspect | Before (God Class) | After (SRP) |
|--------|-------------------|-------------|
| **Lines of Code per Class** | 300+ lines | 20-50 lines per class |
| **Teams Touching Code** | 7+ teams | 1 team per class |
| **Testability** | Nearly impossible | Easy to unit test |
| **Reusability** | Cannot reuse | EmailService reusable |
| **Maintainability** | Nightmare | Simple to maintain |
| **Bug Risk** | High (cascading failures) | Low (isolated changes) |
| **Parallel Development** | Impossible (merge conflicts) | Easy (separate files) |

---

### 📊 Responsibility Matrix

```java
/**
 * 🎯 Who Changes What?
 * 
 * ┌─────────────────────────┬──────────────────────┐
 * │ Class                   │ Changes When...      │
 * ├─────────────────────────┼──────────────────────┤
 * │ UserRegistrationValidator│ Validation rules     │
 * │ PasswordEncryptionService│ Encryption algorithm │
 * │ UserRepository          │ Database schema      │
 * │ PromoCodeService        │ Promo logic          │
 * │ EmailService            │ Email provider       │
 * │ SMSService              │ SMS provider         │
 * │ AnalyticsService        │ Analytics platform   │
 * │ AuditLogService         │ Logging requirements │
 * │ UserRegistrationService │ Registration flow    │
 * │ UserRegistrationController│ API contract       │
 * └─────────────────────────┴──────────────────────┘
 * 
 * ✅ Each class has EXACTLY ONE reason to change!
 */
```

---

## 📦 Scenario 2: Order Processing System

### ❌ BAD Example

```java
/**
 * 🚫 VIOLATION: Order processing doing everything!
 */

@Service
public class OrderProcessor {
    
    public String processOrder(OrderRequest orderRequest) {
        
        // 🔥 Responsibility 1: Validate order
        if (orderRequest.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have items");
        }
        
        // 🔥 Responsibility 2: Check inventory
        for (OrderItem item : orderRequest.getItems()) {
            Connection conn = getConnection();
            PreparedStatement stmt = conn.prepareStatement(
                "SELECT quantity FROM inventory WHERE product_id = ?");
            stmt.setString(1, item.getProductId());
            ResultSet rs = stmt.executeQuery();
            
            if (rs.next()) {
                int available = rs.getInt("quantity");
                if (available < item.getQuantity()) {
                    throw new OutOfStockException("Product out of stock");
                }
            }
        }
        
        // 🔥 Responsibility 3: Calculate pricing
        double subtotal = 0;
        for (OrderItem item : orderRequest.getItems()) {
            PreparedStatement stmt = conn.prepareStatement(
                "SELECT price FROM products WHERE id = ?");
            stmt.setString(1, item.getProductId());
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                double price = rs.getDouble("price");
                subtotal += price * item.getQuantity();
            }
        }
        
        // Apply tax
        double tax = subtotal * 0.08;
        
        // Apply shipping
        double shipping = subtotal > 100 ? 0 : 9.99;
        
        // Apply discount
        double discount = 0;
        if (orderRequest.getDiscountCode() != null) {
            PreparedStatement stmt = conn.prepareStatement(
                "SELECT discount_percent FROM discount_codes WHERE code = ?");
            stmt.setString(1, orderRequest.getDiscountCode());
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                discount = subtotal * rs.getDouble("discount_percent") / 100;
            }
        }
        
        double total = subtotal + tax + shipping - discount;
        
        // 🔥 Responsibility 4: Process payment
        String paymentUrl = "https://payment-gateway.com/charge";
        HttpClient client = HttpClient.newHttpClient();
        String paymentBody = String.format(
            "{\"amount\":%.2f, \"cardNumber\":\"%s\"}", 
            total, orderRequest.getCardNumber());
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(paymentUrl))
            .POST(HttpRequest.BodyPublishers.ofString(paymentBody))
            .build();
        
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
        
        if (response.statusCode() != 200) {
            throw new PaymentFailedException("Payment failed");
        }
        
        // 🔥 Responsibility 5: Update inventory
        for (OrderItem item : orderRequest.getItems()) {
            PreparedStatement stmt = conn.prepareStatement(
                "UPDATE inventory SET quantity = quantity - ? WHERE product_id = ?");
            stmt.setInt(1, item.getQuantity());
            stmt.setString(2, item.getProductId());
            stmt.executeUpdate();
        }
        
        // 🔥 Responsibility 6: Save order to database
        String orderId = UUID.randomUUID().toString();
        PreparedStatement stmt = conn.prepareStatement(
            "INSERT INTO orders (id, user_id, total, status, created_at) " +
            "VALUES (?, ?, ?, ?, ?)");
        stmt.setString(1, orderId);
        stmt.setString(2, orderRequest.getUserId());
        stmt.setDouble(3, total);
        stmt.setString(4, "CONFIRMED");
        stmt.setTimestamp(5, new Timestamp(System.currentTimeMillis()));
        stmt.executeUpdate();
        
        // 🔥 Responsibility 7: Send confirmation email
        sendEmail(orderRequest.getUserEmail(), 
            "Order Confirmation", 
            "Your order " + orderId + " has been placed!");
        
        // 🔥 Responsibility 8: Send SMS
        sendSMS(orderRequest.getUserPhone(), 
            "Order confirmed! Total: $" + total);
        
        // 🔥 Responsibility 9: Create shipping label
        String trackingNumber = createShippingLabel(orderRequest.getAddress());
        
        // 🔥 Responsibility 10: Update loyalty points
        int points = (int) (total / 10);
        PreparedStatement loyaltyStmt = conn.prepareStatement(
            "UPDATE users SET loyalty_points = loyalty_points + ? WHERE id = ?");
        loyaltyStmt.setInt(1, points);
        loyaltyStmt.setString(2, orderRequest.getUserId());
        loyaltyStmt.executeUpdate();
        
        return orderId;
    }
}
```

---

### ✅ GOOD Example - Properly Separated

```java
/**
 * ✨ CLEAN SRP-COMPLIANT ORDER PROCESSING
 */

// ========================================
// 1️⃣ ORDER VALIDATOR - Validates order data
// ========================================

@Component
public class OrderValidator {
    
    public ValidationResult validate(OrderRequest request) {
        List<String> errors = new ArrayList<>();
        
        if (request.getItems() == null || request.getItems().isEmpty()) {
            errors.add("Order must contain at least one item");
        }
        
        if (request.getUserId() == null) {
            errors.add("User ID is required");
        }
        
        if (request.getShippingAddress() == null) {
            errors.add("Shipping address is required");
        }
        
        return new ValidationResult(errors.isEmpty(), errors);
    }
}

// ========================================
// 2️⃣ INVENTORY SERVICE - Manages stock
// ========================================

@Service
public class InventoryService {
    
    @Autowired
    private InventoryRepository inventoryRepository;
    
    /**
     * Single Responsibility: Check and reserve inventory
     */
    public InventoryCheckResult checkAvailability(List<OrderItem> items) {
        List<String> unavailableItems = new ArrayList<>();
        
        for (OrderItem item : items) {
            Optional<Inventory> inventory = inventoryRepository
                .findByProductId(item.getProductId());
            
            if (inventory.isEmpty() || 
                inventory.get().getQuantity() < item.getQuantity()) {
                unavailableItems.add(item.getProductId());
            }
        }
        
        return new InventoryCheckResult(unavailableItems.isEmpty(), unavailableItems);
    }
    
    @Transactional
    public void reserveInventory(List<OrderItem> items) {
        for (OrderItem item : items) {
            inventoryRepository.decrementQuantity(
                item.getProductId(), 
                item.getQuantity()
            );
        }
    }
    
    @Transactional
    public void releaseInventory(List<OrderItem> items) {
        for (OrderItem item : items) {
            inventoryRepository.incrementQuantity(
                item.getProductId(), 
                item.getQuantity()
            );
        }
    }
}

@Repository
public interface InventoryRepository extends JpaRepository<Inventory, Long> {
    Optional<Inventory> findByProductId(String productId);
    
    @Modifying
    @Query("UPDATE Inventory i SET i.quantity = i.quantity - :qty WHERE i.productId = :productId")
    void decrementQuantity(@Param("productId") String productId, @Param("qty") int quantity);
    
    @Modifying
    @Query("UPDATE Inventory i SET i.quantity = i.quantity + :qty WHERE i.productId = :productId")
    void incrementQuantity(@Param("productId") String productId, @Param("qty") int quantity);
}

// ========================================
// 3️⃣ PRICING SERVICE - Calculates prices
// ========================================

@Service
public class PricingService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private TaxService taxService;
    
    @Autowired
    private ShippingCalculator shippingCalculator;
    
    @Autowired
    private DiscountService discountService;
    
    /**
     * Single Responsibility: Calculate order pricing
     */
    public PricingResult calculateOrderPrice(OrderRequest request) {
        
        // Calculate subtotal
        double subtotal = 0;
        for (OrderItem item : request.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
            subtotal += product.getPrice() * item.getQuantity();
        }
        
        // Calculate tax
        double tax = taxService.calculateTax(subtotal, request.getShippingAddress());
        
        // Calculate shipping
        double shipping = shippingCalculator.calculate(
            request.getItems(), 
            request.getShippingAddress()
        );
        
        // Apply discount
        double discount = discountService.calculateDiscount(
            request.getDiscountCode(), 
            subtotal
        );
        
        double total = subtotal + tax + shipping - discount;
        
        return PricingResult.builder()
            .subtotal(subtotal)
            .tax(tax)
            .shipping(shipping)
            .discount(discount)
            .total(total)
            .build();
    }
}

// ========================================
// 4️⃣ TAX SERVICE - Calculates taxes
// ========================================

@Service
public class TaxService {
    
    /**
     * Single Responsibility: Calculate applicable taxes
     */
    public double calculateTax(double amount, Address address) {
        // Get tax rate based on state/country
        double taxRate = getTaxRate(address.getState(), address.getCountry());
        return amount * taxRate;
    }
    
    private double getTaxRate(String state, String country) {
        // Simplified - in reality, would query tax database
        if ("US".equals(country)) {
            return switch (state) {
                case "CA" -> 0.0725;
                case "NY" -> 0.08;
                case "TX" -> 0.0625;
                default -> 0.05;
            };
        }
        return 0;
    }
}

// ========================================
// 5️⃣ SHIPPING CALCULATOR - Calculates shipping
// ========================================

@Service
public class ShippingCalculator {
    
    /**
     * Single Responsibility: Calculate shipping costs
     */
    public double calculate(List<OrderItem> items, Address address) {
        
        // Calculate total weight
        double totalWeight = items.stream()
            .mapToDouble(item -> item.getWeight() * item.getQuantity())
            .sum();
        
        // Base shipping rate
        double baseRate = 5.99;
        
        // Weight surcharge
        double weightSurcharge = totalWeight > 10 ? (totalWeight - 10) * 0.5 : 0;
        
        // Free shipping for orders over $100
        double subtotal = items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        
        if (subtotal >= 100) {
            return 0;
        }
        
        return baseRate + weightSurcharge;
    }
}

// ========================================
// 6️⃣ DISCOUNT SERVICE - Applies discounts
// ========================================

@Service
public class DiscountService {
    
    @Autowired
    private DiscountCodeRepository discountCodeRepository;
    
    /**
     * Single Responsibility: Calculate and validate discounts
     */
    public double calculateDiscount(String code, double subtotal) {
        if (code == null || code.isEmpty()) {
            return 0;
        }
        
        Optional<DiscountCode> discountCode = discountCodeRepository
            .findByCodeAndActiveTrue(code);
        
        if (discountCode.isEmpty()) {
            return 0;
        }
        
        DiscountCode discount = discountCode.get();
        
        // Check if minimum purchase requirement met
        if (subtotal < discount.getMinimumPurchase()) {
            return 0;
        }
        
        // Calculate discount
        if (discount.getType() == DiscountType.PERCENTAGE) {
            return subtotal * discount.getValue() / 100;
        } else {
            return Math.min(discount.getValue(), subtotal);
        }
    }
}

// ========================================
// 7️⃣ PAYMENT SERVICE - Processes payments
// ========================================

@Service
public class PaymentService {
    
    @Value("${payment.gateway.url}")
    private String paymentGatewayUrl;
    
    private final HttpClient httpClient = HttpClient.newHttpClient();
    
    /**
     * Single Responsibility: Process payments
     */
    public PaymentResult processPayment(PaymentRequest paymentRequest) {
        try {
            String requestBody = buildPaymentRequest(paymentRequest);
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(paymentGatewayUrl + "/charge"))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
            
            HttpResponse<String> response = httpClient.send(
                request, 
                HttpResponse.BodyHandlers.ofString()
            );
            
            if (response.statusCode() == 200) {
                String transactionId = parseTransactionId(response.body());
                return PaymentResult.success(transactionId);
            } else {
                return PaymentResult.failure("Payment declined");
            }
            
        } catch (Exception e) {
            return PaymentResult.failure("Payment processing error: " + e.getMessage());
        }
    }
    
    private String buildPaymentRequest(PaymentRequest request) {
        return String.format(
            "{\"amount\":%.2f, \"currency\":\"%s\", \"cardToken\":\"%s\"}",
            request.getAmount(),
            request.getCurrency(),
            request.getCardToken()
        );
    }
    
    private String parseTransactionId(String responseBody) {
        // Parse JSON response
        return "TXN-" + System.currentTimeMillis();
    }
}

// ========================================
// 8️⃣ ORDER REPOSITORY - Data persistence
// ========================================

@Repository
public interface OrderRepository extends JpaRepository<Order, String> {
    List<Order> findByUserId(String userId);
    List<Order> findByStatus(OrderStatus status);
}

// ========================================
// 9️⃣ SHIPPING SERVICE - Creates shipping labels
// ========================================

@Service
public class ShippingService {
    
    @Value("${shipping.carrier.api.url}")
    private String carrierApiUrl;
    
    /**
     * Single Responsibility: Create and manage shipping
     */
    public ShippingLabel createShippingLabel(Order order, Address address) {
        try {
            // Call shipping carrier API
            String trackingNumber = "TRACK-" + UUID.randomUUID().toString();
            
            return ShippingLabel.builder()
                .trackingNumber(trackingNumber)
                .carrier("FedEx")
                .estimatedDelivery(LocalDate.now().plusDays(5))
                .build();
                
        } catch (Exception e) {
            throw new ShippingException("Failed to create shipping label", e);
        }
    }
}

// ========================================
// 🔟 LOYALTY SERVICE - Manages loyalty points
// ========================================

@Service
public class LoyaltyPointsService {
    
    @Autowired
    private UserRepository userRepository;
    
    /**
     * Single Responsibility: Award and manage loyalty points
     */
    @Transactional
    public void awardPoints(String userId, double orderTotal) {
        
        // Award 1 point per $10 spent
        int points = (int) (orderTotal / 10);
        
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        
        user.setLoyaltyPoints(user.getLoyaltyPoints() + points);
        userRepository.save(user);
    }
}

// ========================================
// 1️⃣1️⃣ NOTIFICATION SERVICE - Sends notifications
// ========================================

@Service
public class OrderNotificationService {
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private SMSService smsService;
    
    /**
     * Single Responsibility: Send order-related notifications
     */
    public void sendOrderConfirmation(Order order, User user) {
        
        // Send email
        CompletableFuture.runAsync(() -> 
            emailService.sendOrderConfirmation(order, user)
        );
        
        // Send SMS
        CompletableFuture.runAsync(() -> 
            smsService.sendOrderConfirmation(order, user.getPhone())
        );
    }
}

// ========================================
// 1️⃣2️⃣ ORDER SERVICE - Orchestrates the process
// ========================================

@Service
@Transactional
public class OrderService {
    
    @Autowired private OrderValidator orderValidator;
    @Autowired private InventoryService inventoryService;
    @Autowired private PricingService pricingService;
    @Autowired private PaymentService paymentService;
    @Autowired private OrderRepository orderRepository;
    @Autowired private ShippingService shippingService;
    @Autowired private LoyaltyPointsService loyaltyPointsService;
    @Autowired private OrderNotificationService notificationService;
    @Autowired private UserRepository userRepository;
    
    /**
     * Single Responsibility: Orchestrate order processing workflow
     * This is the ONLY place where the entire order flow is defined
     */
    public OrderResult processOrder(OrderRequest request) {
        
        // Step 1: Validate
        ValidationResult validation = orderValidator.validate(request);
        if (!validation.isValid()) {
            return OrderResult.validationError(validation.getErrors());
        }
        
        // Step 2: Check inventory
        InventoryCheckResult inventoryCheck = inventoryService
            .checkAvailability(request.getItems());
        
        if (!inventoryCheck.isAvailable()) {
            return OrderResult.outOfStock(inventoryCheck.getUnavailableItems());
        }
        
        // Step 3: Calculate pricing
        PricingResult pricing = pricingService.calculateOrderPrice(request);
        
        // Step 4: Process payment
        PaymentRequest paymentRequest = PaymentRequest.builder()
            .amount(pricing.getTotal())
            .currency("USD")
            .cardToken(request.getPaymentToken())
            .build();
        
        PaymentResult payment = paymentService.processPayment(paymentRequest);
        
        if (!payment.isSuccessful()) {
            return OrderResult.paymentFailed(payment.getErrorMessage());
        }
        
        try {
            // Step 5: Reserve inventory
            inventoryService.reserveInventory(request.getItems());
            
            // Step 6: Create order
            Order order = Order.builder()
                .id(UUID.randomUUID().toString())
                .userId(request.getUserId())
                .items(request.getItems())
                .subtotal(pricing.getSubtotal())
                .tax(pricing.getTax())
                .shipping(pricing.getShipping())
                .discount(pricing.getDiscount())
                .total(pricing.getTotal())
                .paymentTransactionId(payment.getTransactionId())
                .status(OrderStatus.CONFIRMED)
                .createdAt(LocalDateTime.now())
                .build();
            
            Order savedOrder = orderRepository.save(order);
            
            // Step 7: Create shipping label
            ShippingLabel shippingLabel = shippingService.createShippingLabel(
                savedOrder, 
                request.getShippingAddress()
            );
            savedOrder.setTrackingNumber(shippingLabel.getTrackingNumber());
            orderRepository.save(savedOrder);
            
            // Step 8: Award loyalty points
            loyaltyPointsService.awardPoints(request.getUserId(), pricing.getTotal());
            
            // Step 9: Send notifications (async)
            User user = userRepository.findById(request.getUserId())
                .orElseThrow(() -> new UserNotFoundException(request.getUserId()));
            notificationService.sendOrderConfirmation(savedOrder, user);
            
            return OrderResult.success(savedOrder);
            
        } catch (Exception e) {
            // Rollback inventory if something fails
            inventoryService.releaseInventory(request.getItems());
            throw new OrderProcessingException("Order processing failed", e);
        }
    }
}
```

---

### 📊 Comparison: Before vs After

| Metric | ❌ Before (God Class) | ✅ After (SRP) |
|--------|---------------------|---------------|
| **Total Lines** | 250+ in one class | 20-40 per class |
| **Number of Classes** | 1 | 12 specialized classes |
| **Testability** | 10% code coverage | 90%+ code coverage |
| **Reusability** | 0% (everything coupled) | 100% (each service reusable) |
| **Change Impact** | 1 change affects entire class | 1 change affects 1 class |
| **Parallel Development** | Impossible (one file) | Easy (12 separate files) |
| **Bug Isolation** | Hard (everything tangled) | Easy (isolated responsibilities) |
| **Code Review Time** | Hours (too complex) | Minutes (simple, focused) |

---

## 🎤 Interview Questions & Answers

### Q1: "What is Single Responsibility Principle?"

**Perfect Answer:**
```
SRP states that a class should have only ONE reason to change, 
meaning it should have only ONE responsibility or job.

Example from my experience:
We had an OrderService doing:
- Validation
- Pricing calculation
- Payment processing
- Inventory management
- Email sending

This violated SRP because:
✗ 5+ teams needed to modify this one class
✗ Changes to email broke payment logic
✗ Impossible to test in isolation
✗ Constant merge conflicts

We refactored into:
✓ OrderValidator
✓ PricingService  
✓ PaymentService
✓ InventoryService
✓ NotificationService

Result:
✓ 80% reduction in bugs
✓ Testing became easy
✓ Teams could work in parallel
```

---

### Q2: "How do you identify SRP violations?"

**Perfect Answer:**
```
I use the "Who wants this changed?" test:

Signs of SRP Violation:
1. Class has multiple "and" in description
   ❌ "This class validates AND saves AND sends email"
   
2. Multiple teams need to modify the same class
   ❌ Marketing (emails), DBA (queries), DevOps (APIs)
   
3. Hard to name the class clearly
   ❌ "UserManagerHelperServiceUtil"
   
4. More than 200-300 lines of code
   ❌ Probably doing too much
   
5. Class has many private methods with different concerns
   ❌ validateEmail(), encryptPassword(), sendSMS()
   
6. Tests are hard to write
   ❌ Need to mock 10+ dependencies

Real Example:
Our UserService had 800 lines and was being modified by:
- Backend team (business logic)
- DBA team (queries)
- Security team (authentication)
- Email team (templates)
- Analytics team (tracking)

→ Split into 5 focused services
```

---

### Q3: "What's the difference between SRP and having too many classes?"

**Perfect Answer:**
```
Great question! SRP doesn't mean one method per class.

✅ CORRECT SRP:
- Each class has COHESIVE responsibility
- Methods work together toward ONE goal

Example - PricingService:
✓ calculateSubtotal()
✓ calculateTax()
✓ calculateShipping()
✓ calculateDiscount()
✓ calculateTotal()

All methods work together for ONE responsibility: PRICING

❌ OVER-ENGINEERING:
Creating classes for every tiny operation:

class AddTwoNumbersService
class MultiplyByTaxRateService
class FormatCurrencyService

Balance Guidelines:
- If methods share the same data/goal → Same class
- If methods have different reasons to change → Different classes
- If class > 300 lines → Probably needs splitting
- If class has 1-2 lines → Probably over-engineered

Real metric I use:
"Can I describe this class's purpose in one simple sentence?"
✓ "Handles user authentication" → Good
✗ "Manages users and sends emails and logs events" → Bad
```

---

### Q4: "Give me a real example where violating SRP caused problems"

**Perfect Answer:**
```
Real incident from my project:

PROBLEM:
We had a ProductService that:
- Managed product data
- Calculated pricing
- Managed inventory
- Generated product reports
- Sent low-stock alerts

INCIDENT:
Marketing wanted to change report format.
Developer modified ProductService.
Accidentally changed a variable name used in pricing calculation.
Deployed to production.

IMPACT:
💥 All products showed $0 price
💥 Customers placed orders for free
💥 Lost $50,000 in one hour
💥 Took 3 hours to find the bug (800 line class)

WHY IT HAPPENED:
- Report logic and pricing logic in same class
- No clear separation
- One change affected unrelated feature
- Hard to test (mocked everything)

SOLUTION:
Split into:
✓ ProductService (data only)
✓ PricingService (pricing only)
✓ InventoryService (stock only)
✓ ReportService (reports only)
✓ AlertService (notifications only)

RESULT:
✓ Each class < 100 lines
✓ Easy to test
✓ Changes isolated
✓ No more cross-contamination

Lesson: SRP isn't academic theory - it prevents REAL production bugs!
```

---

### Q5: "How does SRP relate to microservices?"

**Perfect Answer:**
```
SRP at class level → Foundation for microservices

CLASS LEVEL (SRP):
UserService - handles user operations
PaymentService - handles payments
OrderService - handles orders

MICROSERVICE LEVEL (Same principle):
User Microservice - user operations
Payment Microservice - payment operations  
Order Microservice - order operations

The principle scales:

METHOD → Single responsibility
↓
CLASS → Single responsibility (SRP)
↓
MODULE → Single responsibility
↓
MICROSERVICE → Single responsibility

Example from our e-commerce:

Monolith (Violating SRP):
┌──────────────────────────────┐
│    One Giant Application     │
│  - Users                     │
│  - Products                  │
│  - Orders                    │
│  - Payments                  │
│  - Inventory                 │
│  - Notifications             │
└──────────────────────────────┘
Changes to notification affected everything

Microservices (Following SRP):
┌─────────┐ ┌─────────┐ ┌─────────┐
│  User   │ │ Product │ │  Order  │
│ Service │ │ Service │ │ Service │
└─────────┘ └─────────┘ └─────────┘
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Payment │ │Inventory│ │  Email  │
│ Service │ │ Service │ │ Service │
└─────────┘ └─────────┘ └─────────┘

Benefits:
✓ Each service can be deployed independently
✓ Different teams own different services
✓ Technology can vary per service
✓ Scales independently
```

---

## 🎯 Quick Reference Card

```java
/**
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 *  SINGLE RESPONSIBILITY PRINCIPLE (SRP)
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 * 
 * 🎯 DEFINITION:
 * "A class should have only ONE reason to change"
 * 
 * 🔍 IDENTIFY VIOLATIONS:
 * ❌ Class name has "Manager", "Helper", "Util"
 * ❌ Class has multiple "and" in description
 * ❌ Class has 300+ lines
 * ❌ Multiple teams modify same class
 * ❌ Hard to write unit tests
 * ❌ Private methods serve different purposes
 * 
 * ✅ GOOD SIGNS:
 * ✓ Class has clear, single purpose
 * ✓ Can describe purpose in one sentence
 * ✓ Easy to name
 * ✓ Easy to test
 * ✓ 50-200 lines typically
 * ✓ One team/person owns it
 * 
 * 🔧 REFACTORING TECHNIQUES:
 * 1. Extract Method
 * 2. Extract Class
 * 3. Move Method
 * 4. Introduce Service Layer
 * 5. Use Dependency Injection
 * 
 * 🎁 BENEFITS:
 * ✓ Easier to understand
 * ✓ Easier to test
 * ✓ Easier to maintain
 * ✓ Reduces coupling
 * ✓ Increases cohesion
 * ✓ Prevents bugs
 * ✓ Enables parallel development
 * 
 * ⚠️ ANTI-PATTERNS:
 * • God Class
 * • Swiss Army Knife Class
 * • Kitchen Sink Class
 * 
 * 📏 THE "BOSS" TEST:
 * Ask: "Who would request changes to this class?"
 * • One person/role = ✅ Good (SRP followed)
 * • Multiple people/roles = ❌ Bad (SRP violated)
 * 
 * 💡 REMEMBER:
 * "One class, one job, one boss!"
 * 
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 */
```

---

## 🎓 SRP Mastery Checklist

Before you ship code, ask:

- [ ] Can I describe this class's purpose in ONE sentence?
- [ ] Does this class have only ONE reason to change?
- [ ] Would only ONE team/person need to modify this?
- [ ] Is this class easy to test?
- [ ] Is the class name clear and specific?
- [ ] Are all methods related to the same concern?
- [ ] Is the class under 300 lines?
- [ ] Can I reuse this class in other contexts?
- [ ] Would changing business logic here affect unrelated features?
- [ ] Does this class do only ONE thing well?

**If you answered NO to any → Refactor! ⚠️**

---

## 🌟 Final Pro Tips

### 1️⃣ Start Small
```java
// Don't try to refactor entire codebase at once
// Pick ONE God class and split it
// Show the team the benefits
// Then gradually improve other classes
```

### 2️⃣ Use Layers
```java
Controller → Service → Repository → Database
Each layer has ONE responsibility
```

### 3️⃣ Extract Early
```java
// When you see method > 20 lines doing multiple things
// Extract immediately before it becomes a monster
```

### 4️⃣ Name Matters
```java
✅ UserAuthenticationService    // Clear responsibility
✅ OrderPricingCalculator       // Clear responsibility
❌ UserManager                  // Too vague
❌ OrderProcessor               // What processing?
```

### 5️⃣ The 5-Second Rule
```
If you can't explain what a class does in 5 seconds,
it probably violates SRP!
```

---

**Remember:** 🌟

> "Code is read 10x more than it's written.
> Make it OBVIOUS what each class does.
> Your future self will thank you!"

---

*Made with ❤️ for Clean Code Enthusiasts*
*Interview Success Rate: 💯%*

🚀 **Happy Coding!**

# 🛒 Liskov Substitution Principle - E-Commerce Real-Time Examples
---

## 🎯 LSP Quick 

### Definition
> **"Objects of a superclass should be replaceable with objects of subclasses without breaking the application"**

### E-Commerce Context 🛍️
**If your code expects a `Product`, it should work correctly with any `PhysicalProduct`, `DigitalProduct`, or `ServiceProduct` without any surprises!**

---

## 🏷️ Scenario 1: Product Discount System

### ❌ BAD Example - Violating LSP

```java
/**
 * 🚫 VIOLATION: SpecialProduct changes the behavior unexpectedly
 */

public class Product {
    protected String name;
    protected double price;
    protected String sku;
    
    public Product(String name, double price, String sku) {
        this.name = name;
        this.price = price;
        this.sku = sku;
    }
    
    public double getPrice() {
        return this.price;
    }
    
    // Apply discount to product
    public void applyDiscount(double percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Invalid discount percentage");
        }
        this.price = this.price - (this.price * percentage / 100);
        System.out.println("✅ Discount applied: " + percentage + "%");
    }
    
    public String getName() {
        return name;
    }
}

/**
 * 🔥 PROBLEM: GiftCard violates LSP
 * Gift cards should not be discounted in this e-commerce system
 */
public class GiftCard extends Product {
    
    public GiftCard(String name, double price, String sku) {
        super(name, price, sku);
    }
    
    @Override
    public void applyDiscount(double percentage) {
        // 🚫 VIOLATION: Throwing exception breaks LSP
        throw new UnsupportedOperationException(
            "Gift cards cannot be discounted!"
        );
    }
}

/**
 * 🔥 PROBLEM: ClearanceProduct changes preconditions
 */
public class ClearanceProduct extends Product {
    
    public ClearanceProduct(String name, double price, String sku) {
        super(name, price, sku);
    }
    
    @Override
    public void applyDiscount(double percentage) {
        // 🚫 VIOLATION: Strengthening preconditions (only 50-90% allowed)
        if (percentage < 50 || percentage > 90) {
            throw new IllegalArgumentException(
                "Clearance items only accept 50-90% discount"
            );
        }
        super.applyDiscount(percentage);
    }
}

/**
 * 🔥 PROBLEM: SubscriptionProduct changes behavior silently
 */
public class SubscriptionProduct extends Product {
    
    public SubscriptionProduct(String name, double monthlyPrice, String sku) {
        super(name, monthlyPrice, sku);
    }
    
    @Override
    public void applyDiscount(double percentage) {
        // 🚫 VIOLATION: Discount applied to first month only
        // Parent expects permanent discount, child applies temporary
        System.out.println("⚠️ Discount applied to first month only!");
        super.applyDiscount(percentage);
    }
}

// Shopping Cart Service
public class ShoppingCartService {
    
    public void applySeasonalDiscount(List<Product> products, double discount) {
        System.out.println("🎉 Applying " + discount + "% seasonal discount...\n");
        
        for (Product product : products) {
            try {
                System.out.println("Processing: " + product.getName());
                double oldPrice = product.getPrice();
                
                product.applyDiscount(discount); // 🔥 May throw exception!
                
                double newPrice = product.getPrice();
                System.out.println("💰 Old Price: $" + oldPrice);
                System.out.println("💰 New Price: $" + newPrice);
                System.out.println("---");
                
            } catch (Exception e) {
                // 🚫 Shouldn't need exception handling for valid operations!
                System.out.println("❌ Error: " + e.getMessage());
                System.out.println("---");
            }
        }
    }
}

// Testing the violation
public class LSPViolationDemo {
    public static void main(String[] args) {
        ShoppingCartService cartService = new ShoppingCartService();
        
        List<Product> products = Arrays.asList(
            new Product("Laptop", 1000.0, "LAP001"),
            new GiftCard("$100 Gift Card", 100.0, "GC001"),      // 💥 Will throw exception
            new ClearanceProduct("Old TV", 500.0, "TV001"),       // 💥 Will throw exception for 20%
            new SubscriptionProduct("Netflix", 15.99, "SUB001")   // ⚠️ Behaves differently
        );
        
        // Applying 20% discount
        cartService.applySeasonalDiscount(products, 20.0);
        
        /**
         * 🔥 OUTPUT PROBLEMS:
         * 
         * Processing: Laptop
         * ✅ Discount applied: 20.0%
         * 💰 Old Price: $1000.0
         * 💰 New Price: $800.0
         * ---
         * Processing: $100 Gift Card
         * ❌ Error: Gift cards cannot be discounted!  ← BREAKS!
         * ---
         * Processing: Old TV
         * ❌ Error: Clearance items only accept 50-90% discount  ← BREAKS!
         * ---
         * Processing: Netflix
         * ⚠️ Discount applied to first month only!  ← UNEXPECTED BEHAVIOR!
         * ✅ Discount applied: 20.0%
         * 💰 Old Price: $15.99
         * 💰 New Price: $12.79
         * ---
         */
    }
}
```

### 🔥 Problems with Above Code:

| Issue | Description | Impact |
|-------|-------------|--------|
| **Breaking Contracts** | GiftCard throws exception for valid parent operation | Code crashes 💥 |
| **Strengthening Preconditions** | ClearanceProduct restricts valid discount range | Unexpected errors 🚫 |
| **Changing Behavior** | SubscriptionProduct applies discount differently | Silent bugs 🐛 |
| **Client Code Suffers** | ShoppingCart needs try-catch and type checking | Complexity increases 📈 |

---

### ✅ GOOD Example - Following LSP

```java
/**
 * ✨ SOLUTION 1: Use Composition over Inheritance
 * Separate discountable and non-discountable items
 */

// Base abstraction - all products share this
public abstract class Product {
    protected String name;
    protected double basePrice;
    protected String sku;
    
    public Product(String name, double basePrice, String sku) {
        this.name = name;
        this.basePrice = basePrice;
        this.sku = sku;
    }
    
    public abstract double getPrice();
    
    public String getName() {
        return name;
    }
    
    public String getSku() {
        return sku;
    }
}

/**
 * ✨ Interface for products that support discounts
 */
public interface Discountable {
    void applyDiscount(double percentage);
    double getDiscountedPrice();
    double getOriginalPrice();
}

/**
 * ✨ Standard Product - Supports discounts
 */
public class StandardProduct extends Product implements Discountable {
    private double discountPercentage = 0;
    
    public StandardProduct(String name, double price, String sku) {
        super(name, price, sku);
    }
    
    @Override
    public void applyDiscount(double percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Invalid discount: " + percentage);
        }
        this.discountPercentage = percentage;
        System.out.println("✅ " + percentage + "% discount applied to " + name);
    }
    
    @Override
    public double getDiscountedPrice() {
        return basePrice - (basePrice * discountPercentage / 100);
    }
    
    @Override
    public double getOriginalPrice() {
        return basePrice;
    }
    
    @Override
    public double getPrice() {
        return getDiscountedPrice();
    }
}

/**
 * ✨ Gift Card - Does NOT implement Discountable
 * No surprise exceptions!
 */
public class GiftCardProduct extends Product {
    
    public GiftCardProduct(String name, double value, String sku) {
        super(name, value, sku);
    }
    
    @Override
    public double getPrice() {
        return basePrice; // Always return face value
    }
    
    // No applyDiscount method - compiler prevents misuse!
}

/**
 * ✨ Clearance Product - Special discount rules
 * Still honors LSP by accepting same discount range
 */
public class ClearanceProduct extends Product implements Discountable {
    private double discountPercentage = 50; // Minimum 50% off
    
    public ClearanceProduct(String name, double price, String sku) {
        super(name, price, sku);
    }
    
    @Override
    public void applyDiscount(double percentage) {
        // ✅ Still accepts 0-100%, but has a minimum floor
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Invalid discount: " + percentage);
        }
        
        // Apply at least 50% discount (clearance minimum)
        this.discountPercentage = Math.max(percentage, 50);
        
        if (percentage < 50) {
            System.out.println("⚠️ Clearance minimum is 50%, applying 50% for " + name);
        } else {
            System.out.println("✅ " + percentage + "% discount applied to " + name);
        }
    }
    
    @Override
    public double getDiscountedPrice() {
        return basePrice - (basePrice * discountPercentage / 100);
    }
    
    @Override
    public double getOriginalPrice() {
        return basePrice;
    }
    
    @Override
    public double getPrice() {
        return getDiscountedPrice();
    }
}

/**
 * ✨ Subscription Product - Clear behavior
 */
public class SubscriptionProduct extends Product implements Discountable {
    private double firstMonthDiscount = 0;
    private final int billingCycleMonths;
    
    public SubscriptionProduct(String name, double monthlyPrice, String sku, int billingCycleMonths) {
        super(name, monthlyPrice, sku);
        this.billingCycleMonths = billingCycleMonths;
    }
    
    @Override
    public void applyDiscount(double percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Invalid discount: " + percentage);
        }
        this.firstMonthDiscount = percentage;
        System.out.println("✅ " + percentage + "% first-month discount applied to " + name);
    }
    
    @Override
    public double getDiscountedPrice() {
        return basePrice - (basePrice * firstMonthDiscount / 100);
    }
    
    @Override
    public double getOriginalPrice() {
        return basePrice;
    }
    
    @Override
    public double getPrice() {
        return getDiscountedPrice(); // Returns first month price
    }
    
    public double getRecurringPrice() {
        return basePrice; // Regular monthly price
    }
}

/**
 * ✨ Smart Shopping Cart Service
 */
public class ShoppingCartService {
    
    // Method specifically for discountable items
    public void applySeasonalDiscount(List<Discountable> discountableItems, double discount) {
        System.out.println("🎉 Applying " + discount + "% seasonal discount...\n");
        
        for (Discountable item : discountableItems) {
            double oldPrice = item.getOriginalPrice();
            item.applyDiscount(discount);
            double newPrice = item.getDiscountedPrice();
            
            System.out.println("💰 Original: $" + String.format("%.2f", oldPrice));
            System.out.println("💰 Final: $" + String.format("%.2f", newPrice));
            System.out.println("---");
        }
    }
    
    // Method for all products
    public double calculateTotal(List<Product> products) {
        double total = 0;
        System.out.println("\n🧾 Cart Summary:");
        
        for (Product product : products) {
            double price = product.getPrice();
            total += price;
            
            String type = product instanceof Discountable ? "✅ Discountable" : "🎁 Fixed Price";
            System.out.println(product.getName() + ": $" + 
                             String.format("%.2f", price) + " " + type);
        }
        
        System.out.println("---");
        System.out.println("💵 Total: $" + String.format("%.2f", total));
        return total;
    }
    
    // Separate discountable from non-discountable
    public List<Discountable> getDiscountableItems(List<Product> products) {
        return products.stream()
                .filter(p -> p instanceof Discountable)
                .map(p -> (Discountable) p)
                .collect(Collectors.toList());
    }
}

/**
 * ✨ Demo - LSP Compliant Code
 */
public class LSPCompliantDemo {
    public static void main(String[] args) {
        ShoppingCartService cartService = new ShoppingCartService();
        
        // Create diverse product catalog
        List<Product> allProducts = Arrays.asList(
            new StandardProduct("Laptop", 1000.0, "LAP001"),
            new StandardProduct("Mouse", 25.0, "MOU001"),
            new GiftCardProduct("$100 Gift Card", 100.0, "GC001"),
            new ClearanceProduct("Old TV", 500.0, "TV001"),
            new SubscriptionProduct("Netflix Premium", 15.99, "SUB001", 12)
        );
        
        System.out.println("🛒 E-COMMERCE DISCOUNT SYSTEM\n");
        System.out.println("=" .repeat(50));
        
        // Get only discountable items
        List<Discountable> discountableItems = cartService.getDiscountableItems(allProducts);
        
        // Apply discount ONLY to discountable items - No exceptions!
        cartService.applySeasonalDiscount(discountableItems, 20.0);
        
        System.out.println("=" .repeat(50));
        
        // Calculate total for ALL products
        cartService.calculateTotal(allProducts);
        
        /**
         * ✅ OUTPUT - No Errors, Predictable Behavior:
         * 
         * 🛒 E-COMMERCE DISCOUNT SYSTEM
         * 
         * ==================================================
         * 🎉 Applying 20.0% seasonal discount...
         * 
         * ✅ 20.0% discount applied to Laptop
         * 💰 Original: $1000.00
         * 💰 Final: $800.00
         * ---
         * ✅ 20.0% discount applied to Mouse
         * 💰 Original: $25.00
         * 💰 Final: $20.00
         * ---
         * ⚠️ Clearance minimum is 50%, applying 50% for Old TV
         * 💰 Original: $500.00
         * 💰 Final: $250.00
         * ---
         * ✅ 20.0% first-month discount applied to Netflix Premium
         * 💰 Original: $15.99
         * 💰 Final: $12.79
         * ---
         * ==================================================
         * 
         * 🧾 Cart Summary:
         * Laptop: $800.00 ✅ Discountable
         * Mouse: $20.00 ✅ Discountable
         * $100 Gift Card: $100.00 🎁 Fixed Price
         * Old TV: $250.00 ✅ Discountable
         * Netflix Premium: $12.79 ✅ Discountable
         * ---
         * 💵 Total: $1182.79
         */
    }
}
```

### ✨ Benefits of LSP-Compliant Design:

| Benefit | Description | Impact |
|---------|-------------|--------|
| **No Surprises** | Each subclass behaves consistently | Predictable code ✅ |
| **Type Safety** | Compiler prevents misuse | Fewer runtime errors 🛡️ |
| **Clean Separation** | Clear distinction between types | Better organization 📁 |
| **Easy Testing** | Each type can be tested independently | Reliable tests 🧪 |
| **Flexible** | Easy to add new product types | Scalable system 📈 |

---

## 💳 Scenario 2: Payment Processing

### ❌ BAD Example - Violating LSP

```java
/**
 * 🚫 Payment processing that violates LSP
 */

public class PaymentMethod {
    protected String accountNumber;
    
    public PaymentMethod(String accountNumber) {
        this.accountNumber = accountNumber;
    }
    
    public boolean processPayment(double amount) {
        System.out.println("Processing payment of $" + amount);
        return true;
    }
    
    public boolean refund(double amount) {
        System.out.println("Processing refund of $" + amount);
        return true;
    }
    
    public boolean authorizePayment(double amount) {
        System.out.println("Authorizing payment of $" + amount);
        return true;
    }
}

/**
 * 🔥 PROBLEM: Cash payment can't do refunds or authorization
 */
public class CashPayment extends PaymentMethod {
    
    public CashPayment() {
        super("CASH");
    }
    
    @Override
    public boolean refund(double amount) {
        // 🚫 VIOLATION: Cash can't be refunded electronically
        throw new UnsupportedOperationException("Cash refunds not supported!");
    }
    
    @Override
    public boolean authorizePayment(double amount) {
        // 🚫 VIOLATION: Cash doesn't need authorization
        throw new UnsupportedOperationException("Cash doesn't need authorization!");
    }
}

/**
 * 🔥 PROBLEM: Store Credit changes postconditions
 */
public class StoreCreditPayment extends PaymentMethod {
    private double availableCredit;
    
    public StoreCreditPayment(String accountNumber, double availableCredit) {
        super(accountNumber);
        this.availableCredit = availableCredit;
    }
    
    @Override
    public boolean processPayment(double amount) {
        if (amount > availableCredit) {
            // 🚫 VIOLATION: Parent always returns true, child can return false
            System.out.println("Insufficient store credit!");
            return false;
        }
        availableCredit -= amount;
        return true;
    }
}

// Order Service trying to use payments
public class OrderService {
    
    public void processOrder(PaymentMethod payment, double amount) {
        try {
            // Authorize first
            if (payment.authorizePayment(amount)) { // 💥 May throw exception!
                
                // Process payment
                if (payment.processPayment(amount)) {
                    System.out.println("✅ Order completed!");
                } else {
                    System.out.println("❌ Payment failed!");
                }
            }
        } catch (Exception e) {
            System.out.println("💥 Error: " + e.getMessage());
        }
    }
}
```

### ✅ GOOD Example - Following LSP

```java
/**
 * ✨ LSP-Compliant Payment System
 */

// Base abstraction
public interface PaymentMethod {
    PaymentResult processPayment(double amount);
    String getPaymentType();
}

// Payment result object
public class PaymentResult {
    private boolean success;
    private String message;
    private String transactionId;
    
    public PaymentResult(boolean success, String message, String transactionId) {
        this.success = success;
        this.message = message;
        this.transactionId = transactionId;
    }
    
    public boolean isSuccess() { return success; }
    public String getMessage() { return message; }
    public String getTransactionId() { return transactionId; }
}

// Interface for refundable payments
public interface RefundablePayment extends PaymentMethod {
    PaymentResult refund(double amount, String transactionId);
}

// Interface for payments that need authorization
public interface AuthorizablePayment extends PaymentMethod {
    PaymentResult authorize(double amount);
    PaymentResult capture(String authorizationId);
}

/**
 * ✨ Credit Card - Full featured
 */
public class CreditCardPayment implements AuthorizablePayment, RefundablePayment {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public PaymentResult authorize(double amount) {
        String authId = "AUTH-" + System.currentTimeMillis();
        System.out.println("💳 Credit card authorized: $" + amount);
        return new PaymentResult(true, "Authorized", authId);
    }
    
    @Override
    public PaymentResult capture(String authorizationId) {
        System.out.println("💳 Payment captured: " + authorizationId);
        return new PaymentResult(true, "Captured", authorizationId);
    }
    
    @Override
    public PaymentResult processPayment(double amount) {
        String txnId = "TXN-" + System.currentTimeMillis();
        System.out.println("💳 Credit card charged: $" + amount);
        return new PaymentResult(true, "Payment successful", txnId);
    }
    
    @Override
    public PaymentResult refund(double amount, String transactionId) {
        System.out.println("💳 Refunded to credit card: $" + amount);
        return new PaymentResult(true, "Refund successful", transactionId);
    }
    
    @Override
    public String getPaymentType() {
        return "CREDIT_CARD";
    }
}

/**
 * ✨ Cash Payment - Simple, no refunds/auth
 */
public class CashPayment implements PaymentMethod {
    
    @Override
    public PaymentResult processPayment(double amount) {
        String txnId = "CASH-" + System.currentTimeMillis();
        System.out.println("💵 Cash payment received: $" + amount);
        return new PaymentResult(true, "Cash payment accepted", txnId);
    }
    
    @Override
    public String getPaymentType() {
        return "CASH";
    }
    
    // No authorize() or refund() methods - type safe!
}

/**
 * ✨ Store Credit - Can fail gracefully
 */
public class StoreCreditPayment implements RefundablePayment {
    private double availableCredit;
    private String accountId;
    
    public StoreCreditPayment(String accountId, double availableCredit) {
        this.accountId = accountId;
        this.availableCredit = availableCredit;
    }
    
    @Override
    public PaymentResult processPayment(double amount) {
        if (amount > availableCredit) {
            System.out.println("🎫 Insufficient store credit");
            return new PaymentResult(false, "Insufficient credit", null);
        }
        
        availableCredit -= amount;
        String txnId = "SC-" + System.currentTimeMillis();
        System.out.println("🎫 Store credit used: $" + amount);
        return new PaymentResult(true, "Store credit applied", txnId);
    }
    
    @Override
    public PaymentResult refund(double amount, String transactionId) {
        availableCredit += amount;
        System.out.println("🎫 Store credit refunded: $" + amount);
        return new PaymentResult(true, "Credit restored", transactionId);
    }
    
    @Override
    public String getPaymentType() {
        return "STORE_CREDIT";
    }
}

/**
 * ✨ Smart Order Service
 */
public class OrderService {
    
    // Works with any payment method
    public void processSimpleOrder(PaymentMethod payment, double amount) {
        System.out.println("\n💼 Processing " + payment.getPaymentType() + " payment...");
        
        PaymentResult result = payment.processPayment(amount);
        
        if (result.isSuccess()) {
            System.out.println("✅ Order completed! Transaction: " + result.getTransactionId());
        } else {
            System.out.println("❌ Payment failed: " + result.getMessage());
        }
    }
    
    // Works specifically with authorizable payments
    public void processAuthorizableOrder(AuthorizablePayment payment, double amount) {
        System.out.println("\n💼 Processing authorized payment...");
        
        // Authorize first
        PaymentResult authResult = payment.authorize(amount);
        if (!authResult.isSuccess()) {
            System.out.println("❌ Authorization failed!");
            return;
        }
        
        System.out.println("✅ Payment authorized, capturing...");
        
        // Capture the payment
        PaymentResult captureResult = payment.capture(authResult.getTransactionId());
        if (captureResult.isSuccess()) {
            System.out.println("✅ Order completed!");
        }
    }
    
    // Handle refunds
    public void processRefund(RefundablePayment payment, double amount, String transactionId) {
        System.out.println("\n💼 Processing refund...");
        
        PaymentResult result = payment.refund(amount, transactionId);
        if (result.isSuccess()) {
            System.out.println("✅ Refund completed!");
        } else {
            System.out.println("❌ Refund failed: " + result.getMessage());
        }
    }
}

/**
 * ✨ Demo
 */
public class PaymentDemo {
    public static void main(String[] args) {
        OrderService orderService = new OrderService();
        
        // Different payment methods
        CreditCardPayment creditCard = new CreditCardPayment("**** 1234");
        CashPayment cash = new CashPayment();
        StoreCreditPayment storeCredit = new StoreCreditPayment("SC123", 50.0);
        
        System.out.println("🛒 E-COMMERCE PAYMENT PROCESSING\n");
        System.out.println("=".repeat(50));
        
        // All can be processed through base interface
        orderService.processSimpleOrder(creditCard, 100.0);
        orderService.processSimpleOrder(cash, 50.0);
        orderService.processSimpleOrder(storeCredit, 30.0);
        
        // Credit card specific - authorization
        orderService.processAuthorizableOrder(creditCard, 200.0);
        
        // Refundable payments
        orderService.processRefund(creditCard, 100.0, "TXN-123");
        orderService.processRefund(storeCredit, 30.0, "SC-456");
        
        // cash.refund() - COMPILER ERROR! Type safe! ✅
        
        /**
         * ✅ OUTPUT:
         * 
         * 🛒 E-COMMERCE PAYMENT PROCESSING
         * 
         * ==================================================
         * 
         * 💼 Processing CREDIT_CARD payment...
         * 💳 Credit card charged: $100.0
         * ✅ Order completed! Transaction: TXN-1234567890
         * 
         * 💼 Processing CASH payment...
         * 💵 Cash payment received: $50.0
         * ✅ Order completed! Transaction: CASH-1234567891
         * 
         * 💼 Processing STORE_CREDIT payment...
         * 🎫 Store credit used: $30.0
         * ✅ Order completed! Transaction: SC-1234567892
         * 
         * 💼 Processing authorized payment...
         * 💳 Credit card authorized: $200.0
         * ✅ Payment authorized, capturing...
         * 💳 Payment captured: AUTH-1234567893
         * ✅ Order completed!
         * 
         * 💼 Processing refund...
         * 💳 Refunded to credit card: $100.0
         * ✅ Refund completed!
         * 
         * 💼 Processing refund...
         * 🎫 Store credit refunded: $30.0
         * ✅ Refund completed!
         */
    }
}
```

---

## 🚚 Scenario 3: Shipping Methods

### ❌ BAD Example

```java
public class ShippingMethod {
    public double calculateCost(double weight, String destination) {
        return weight * 5.0; // Base rate
    }
    
    public int getEstimatedDays(String destination) {
        return 5; // Default 5 days
    }
    
    public String getTrackingNumber(String orderId) {
        return "TRACK-" + orderId;
    }
}

/**
 * 🔥 PROBLEM: InStorePickup violates LSP
 */
public class InStorePickup extends ShippingMethod {
    
    @Override
    public double calculateCost(double weight, String destination) {
        return 0; // ✅ This is OK
    }
    
    @Override
    public int getEstimatedDays(String destination) {
        return 0; // Same day pickup - ✅ OK
    }
    
    @Override
    public String getTrackingNumber(String orderId) {
        // 🚫 VIOLATION: No tracking for in-store pickup
        throw new UnsupportedOperationException("No tracking for in-store pickup!");
    }
}
```

### ✅ GOOD Example

```java
/**
 * ✨ LSP-Compliant Shipping System
 */

public abstract class ShippingMethod {
    protected String name;
    
    public ShippingMethod(String name) {
        this.name = name;
    }
    
    public abstract double calculateCost(double weight, String destination);
    public abstract int getEstimatedDays(String destination);
    public abstract String getName();
}

// Interface for trackable shipping
public interface Trackable {
    String getTrackingNumber(String orderId);
    TrackingInfo getTrackingInfo(String trackingNumber);
}

// Tracking information
public class TrackingInfo {
    private String status;
    private String location;
    private LocalDateTime lastUpdate;
    
    public TrackingInfo(String status, String location) {
        this.status = status;
        this.location = location;
        this.lastUpdate = LocalDateTime.now();
    }
    
    // Getters...
    public String getStatus() { return status; }
    public String getLocation() { return location; }
}

/**
 * ✨ Standard Shipping - Trackable
 */
public class StandardShipping extends ShippingMethod implements Trackable {
    
    public StandardShipping() {
        super("Standard Shipping");
    }
    
    @Override
    public double calculateCost(double weight, String destination) {
        return weight * 5.0;
    }
    
    @Override
    public int getEstimatedDays(String destination) {
        return 5;
    }
    
    @Override
    public String getTrackingNumber(String orderId) {
        return "STD-" + orderId;
    }
    
    @Override
    public TrackingInfo getTrackingInfo(String trackingNumber) {
        return new TrackingInfo("In Transit", "Distribution Center");
    }
    
    @Override
    public String getName() {
        return name;
    }
}

/**
 * ✨ Express Shipping - Trackable
 */
public class ExpressShipping extends ShippingMethod implements Trackable {
    
    public ExpressShipping() {
        super("Express Shipping");
    }
    
    @Override
    public double calculateCost(double weight, String destination) {
        return weight * 15.0; // Premium rate
    }
    
    @Override
    public int getEstimatedDays(String destination) {
        return 2;
    }
    
    @Override
    public String getTrackingNumber(String orderId) {
        return "EXP-" + orderId;
    }
    
    @Override
    public TrackingInfo getTrackingInfo(String trackingNumber) {
        return new TrackingInfo("Priority Processing", "Express Hub");
    }
    
    @Override
    public String getName() {
        return name;
    }
}

/**
 * ✨ In-Store Pickup - NOT trackable (no violation!)
 */
public class InStorePickup extends ShippingMethod {
    private String storeAddress;
    
    public InStorePickup(String storeAddress) {
        super("In-Store Pickup");
        this.storeAddress = storeAddress;
    }
    
    @Override
    public double calculateCost(double weight, String destination) {
        return 0; // Free pickup
    }
    
    @Override
    public int getEstimatedDays(String destination) {
        return 0; // Same day
    }
    
    @Override
    public String getName() {
        return name + " - " + storeAddress;
    }
    
    public String getPickupInstructions() {
        return "Visit " + storeAddress + " with your order confirmation";
    }
}

/**
 * ✨ Shipping Service
 */
public class ShippingService {
    
    public void displayShippingOptions(List<ShippingMethod> methods, 
                                      double weight, String destination) {
        System.out.println("\n📦 SHIPPING OPTIONS:");
        System.out.println("Package Weight: " + weight + " lbs");
        System.out.println("Destination: " + destination);
        System.out.println("-".repeat(50));
        
        for (ShippingMethod method : methods) {
            double cost = method.calculateCost(weight, destination);
            int days = method.getEstimatedDays(String destination);
            
            System.out.println("\n🚚 " + method.getName());
            System.out.println("   Cost: $" + String.format("%.2f", cost));
            System.out.println("   Delivery: " + days + " days");
            
            // Only show tracking for trackable methods
            if (method instanceof Trackable) {
                System.out.println("   📍 Tracking: Available");
            } else {
                System.out.println("   📍 Tracking: Not applicable");
            }
        }
    }
    
    public void trackShipment(Trackable trackable, String orderId) {
        String trackingNumber = trackable.getTrackingNumber(orderId);
        TrackingInfo info = trackable.getTrackingInfo(trackingNumber);
        
        System.out.println("\n📍 TRACKING INFO:");
        System.out.println("Tracking #: " + trackingNumber);
        System.out.println("Status: " + info.getStatus());
        System.out.println("Location: " + info.getLocation());
    }
}

/**
 * ✨ Demo
 */
public class ShippingDemo {
    public static void main(String[] args) {
        ShippingService service = new ShippingService();
        
        List<ShippingMethod> options = Arrays.asList(
            new StandardShipping(),
            new ExpressShipping(),
            new InStorePickup("123 Main St, New York")
        );
        
        // Display all options - works for ALL shipping methods
        service.displayShippingOptions(options, 5.0, "Los Angeles");
        
        // Track only trackable shipments
        System.out.println("\n" + "=".repeat(50));
        service.trackShipment(new StandardShipping(), "ORD-12345");
        service.trackShipment(new ExpressShipping(), "ORD-67890");
        
        // inStorePickup.getTrackingNumber() - COMPILER ERROR! ✅
    }
}
```

---

## 📦 Scenario 4: Order Types

### Real-World LSP Challenge

```java
/**
 * ✨ Different order types in e-commerce
 */

public abstract class Order {
    protected String orderId;
    protected List<Product> items;
    protected OrderStatus status;
    
    public abstract double calculateTotal();
    public abstract boolean canBeCancelled();
    public abstract String getOrderType();
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

/**
 * ✨ Regular Order
 */
public class RegularOrder extends Order {
    
    @Override
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(Product::getPrice)
            .sum();
    }
    
    @Override
    public boolean canBeCancelled() {
        return status == OrderStatus.PENDING || 
               status == OrderStatus.CONFIRMED;
    }
    
    @Override
    public String getOrderType() {
        return "Regular Order";
    }
}

/**
 * ✨ Pre-Order - Different rules but still consistent
 */
public class PreOrder extends Order {
    private LocalDate releaseDate;
    
    @Override
    public double calculateTotal() {
        // Pre-orders might have discount
        double subtotal = items.stream()
            .mapToDouble(Product::getPrice)
            .sum();
        return subtotal * 0.9; // 10% pre-order discount
    }
    
    @Override
    public boolean canBeCancelled() {
        // Can cancel until release date
        return LocalDate.now().isBefore(releaseDate);
    }
    
    @Override
    public String getOrderType() {
        return "Pre-Order (Release: " + releaseDate + ")";
    }
}

/**
 * ✨ Subscription Order - Recurring behavior
 */
public class SubscriptionOrder extends Order {
    private LocalDate nextBillingDate;
    
    @Override
    public double calculateTotal() {
        // Subscription monthly fee
        return items.stream()
            .mapToDouble(Product::getPrice)
            .sum();
    }
    
    @Override
    public boolean canBeCancelled() {
        // Subscriptions can always be cancelled
        // (will stop at next billing cycle)
        return true;
    }
    
    @Override
    public String getOrderType() {
        return "Subscription (Next billing: " + nextBillingDate + ")";
    }
}

/**
 * ✨ Order Service - Works with all order types
 */
public class OrderManagementService {
    
    public void displayOrder(Order order) {
        System.out.println("\n📋 " + order.getOrderType());
        System.out.println("Order ID: " + order.orderId);
        System.out.println("Total: $" + String.format("%.2f", order.calculateTotal()));
        System.out.println("Can Cancel: " + (order.canBeCancelled() ? "✅ Yes" : "❌ No"));
    }
    
    public boolean cancelOrder(Order order) {
        if (order.canBeCancelled()) {
            order.status = OrderStatus.CANCELLED;
            System.out.println("✅ Order " + order.orderId + " cancelled");
            return true;
        } else {
            System.out.println("❌ Cannot cancel order " + order.orderId);
            return false;
        }
    }
    
    // Works consistently for ALL order types
    public void processOrders(List<Order> orders) {
        double totalRevenue = 0;
        int cancellableOrders = 0;
        
        for (Order order : orders) {
            displayOrder(order);
            totalRevenue += order.calculateTotal();
            if (order.canBeCancelled()) {
                cancellableOrders++;
            }
        }
        
        System.out.println("\n" + "=".repeat(50));
        System.out.println("💰 Total Revenue: $" + String.format("%.2f", totalRevenue));
        System.out.println("📊 Cancellable Orders: " + cancellableOrders + "/" + orders.size());
    }
}
```

---

## 👤 Scenario 5: User Account Types

```java
/**
 * ✨ Different customer types in e-commerce
 */

public interface Customer {
    String getCustomerId();
    String getName();
    double calculateDiscount(double orderAmount);
}

/**
 * ✨ Guest Customer - No discounts
 */
public class GuestCustomer implements Customer {
    private String sessionId;
    private String email;
    
    @Override
    public String getCustomerId() {
        return "GUEST-" + sessionId;
    }
    
    @Override
    public String getName() {
        return "Guest";
    }
    
    @Override
    public double calculateDiscount(double orderAmount) {
        return 0; // No discount for guests
    }
}

/**
 * ✨ Regular Customer - Tiered discounts
 */
public class RegisteredCustomer implements Customer {
    private String customerId;
    private String name;
    private int loyaltyPoints;
    
    @Override
    public String getCustomerId() {
        return customerId;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public double calculateDiscount(double orderAmount) {
        // Loyalty points give discount
        if (loyaltyPoints > 1000) {
            return orderAmount * 0.15; // 15% discount
        } else if (loyaltyPoints > 500) {
            return orderAmount * 0.10; // 10% discount
        } else if (loyaltyPoints > 100) {
            return orderAmount * 0.05; // 5% discount
        }
        return 0;
    }
}

/**
 * ✨ VIP Customer - Premium benefits
 */
public class VIPCustomer implements Customer {
    private String customerId;
    private String name;
    private double discountRate = 0.20; // 20% discount
    
    @Override
    public String getCustomerId() {
        return "VIP-" + customerId;
    }
    
    @Override
    public String getName() {
        return name + " (VIP)";
    }
    
    @Override
    public double calculateDiscount(double orderAmount) {
        // VIP always gets 20% + additional $10 on orders > $100
        double discount = orderAmount * discountRate;
        if (orderAmount > 100) {
            discount += 10;
        }
        return discount;
    }
}

/**
 * ✨ Employee Customer - Special pricing
 */
public class EmployeeCustomer implements Customer {
    private String employeeId;
    private String name;
    
    @Override
    public String getCustomerId() {
        return "EMP-" + employeeId;
    }
    
    @Override
    public String getName() {
        return name + " (Employee)";
    }
    
    @Override
    public double calculateDiscount(double orderAmount) {
        // Flat 30% employee discount
        return orderAmount * 0.30;
    }
}

/**
 * ✨ Checkout Service - Works with ANY customer type
 */
public class CheckoutService {
    
    public void processCheckout(Customer customer, double orderAmount) {
        System.out.println("\n🛍️ CHECKOUT");
        System.out.println("Customer: " + customer.getName());
        System.out.println("Customer ID: " + customer.getCustomerId());
        System.out.println("Order Amount: $" + String.format("%.2f", orderAmount));
        
        double discount = customer.calculateDiscount(orderAmount);
        double finalAmount = orderAmount - discount;
        
        System.out.println("Discount: -$" + String.format("%.2f", discount));
        System.out.println("Final Amount: $" + String.format("%.2f", finalAmount));
        System.out.println("✅ Payment processed");
    }
}

/**
 * ✨ Demo
 */
public class CustomerDemo {
    public static void main(String[] args) {
        CheckoutService checkout = new CheckoutService();
        
        double orderAmount = 200.0;
        
        // All customer types work consistently!
        checkout.processCheckout(new GuestCustomer(), orderAmount);
        checkout.processCheckout(new RegisteredCustomer("John Doe", 600), orderAmount);
        checkout.processCheckout(new VIPCustomer("Jane Smith"), orderAmount);
        checkout.processCheckout(new EmployeeCustomer("Bob Johnson"), orderAmount);
        
        /**
         * ✅ OUTPUT - All behave consistently:
         * 
         * 🛍️ CHECKOUT
         * Customer: Guest
         * Customer ID: GUEST-abc123
         * Order Amount: $200.00
         * Discount: -$0.00
         * Final Amount: $200.00
         * ✅ Payment processed
         * 
         * 🛍️ CHECKOUT
         * Customer: John Doe
         * Customer ID: CUST-123
         * Order Amount: $200.00
         * Discount: -$20.00
         * Final Amount: $180.00
         * ✅ Payment processed
         * 
         * 🛍️ CHECKOUT
         * Customer: Jane Smith (VIP)
         * Customer ID: VIP-456
         * Order Amount: $200.00
         * Discount: -$50.00
         * Final Amount: $150.00
         * ✅ Payment processed
         * 
         * 🛍️ CHECKOUT
         * Customer: Bob Johnson (Employee)
         * Customer ID: EMP-789
         * Order Amount: $200.00
         * Discount: -$60.00
         * Final Amount: $140.00
         * ✅ Payment processed
         */
    }
}
```

---

## 🎤 Interview Questions

### Q1: "Give me an e-commerce example of LSP violation"

**Answer:**
```
A common violation is the Rectangle-Square problem applied to products:

❌ BAD:
- Product has setPrice() method
- GiftCard extends Product but throws exception on setPrice()
- Violates LSP because GiftCard can't substitute Product

✅ GOOD:
- Separate Priceable interface
- Only products that can change price implement it
- GiftCard doesn't implement Priceable
```

### Q2: "How would you refactor code violating LSP?"

**Answer:**
```
Steps:
1. Identify where child class:
   - Throws unexpected exceptions
   - Strengthens preconditions  
   - Weakens postconditions
   - Changes behavior unexpectedly

2. Refactor using:
   - Composition over inheritance
   - Separate interfaces for different behaviors
   - Abstract base class for common behavior only

3. Example: Payment methods
   - Split into Refundable, Authorizable interfaces
   - Each payment implements only what it supports
```

### Q3: "LSP vs. Open/Closed Principle - How are they related?"

**Answer:**
```
They work together:

OCP: Code should be open for extension (new subclasses)
LSP: Those subclasses must be proper substitutes

If you violate LSP, you break OCP because:
- Client code needs type checking
- Can't extend without modifying client code

Example:
✅ Can add new PaymentMethod implementations
✅ Because all honor the same contract
✅ Client code doesn't need modification
```

---

## 📊 LSP Checklist for E-Commerce

### Before Creating Subclass, Ask:

| Question | Example | LSP Compliant? |
|----------|---------|----------------|
| Can it do everything parent does? | GiftCard can't apply discounts | ❌ No |
| Does it have stricter rules? | ClearanceProduct requires 50-90% discount | ❌ No |
| Does it throw new exceptions? | CashPayment throws on refund() | ❌ No |
| Does it return unexpected results? | StoreCredit can fail payment | ⚠️ Maybe* |
| Does it change behavior silently? | SubscriptionDiscount applies to first month only | ❌ No |

*If failure is part of the contract (returns boolean/Result object), it's OK

---

## 🎯 Key Takeaways

### ✅ DO:
- Design interfaces based on **capabilities**, not types
- Use composition when behaviors vary significantly
- Return result objects instead of throwing exceptions
- Document contracts clearly
- Test substitutability

### ❌ DON'T:
- Force child to implement methods it can't support
- Strengthen preconditions in child classes
- Weaken postconditions in child classes
- Throw new exception types not in parent
- Change behavior silently

---

## 💡 Remember:

```java
/**
 * 🎯 LSP in E-Commerce:
 * 
 * "If your code works with a Product, 
 *  it should work with ANY type of Product
 *  without surprises!"
 * 
 * Translation:
 * - No unexpected exceptions 💥
 * - No type checking needed 🔍
 * - No special cases 🚫
 * - Pure polymorphism ✨
 */
```

**The Golden Test:** 🌟
> Can I replace the parent with the child and not break anything? 
> **If YES → LSP ✅**  
> **If NO → Refactor! ⚠️**

---

*Happy Coding! 🚀*
