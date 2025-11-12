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

# 🔓 Open/Closed Principle - Real-Time Deep Dive
---

## 🎯 OCP Core Concept

### Definition
> **"Software entities should be OPEN for extension but CLOSED for modification"**

### What It Really Means 💭

```java
/**
 * 🔓 OPEN for Extension:
 * - You can ADD new functionality
 * - You can create new classes/implementations
 * - System can grow and evolve
 * 
 * 🔒 CLOSED for Modification:
 * - DON'T change existing, tested code
 * - Existing classes remain untouched
 * - Reduces risk of breaking working features
 * 
 * HOW TO ACHIEVE:
 * ✅ Use Abstractions (Interfaces/Abstract Classes)
 * ✅ Use Polymorphism
 * ✅ Use Design Patterns (Strategy, Factory, etc.)
 * ✅ Dependency Injection
 * 
 * ❌ AVOID:
 * ✗ Long if-else or switch statements
 * ✗ Modifying existing classes for new features
 * ✗ Hard-coded logic
 */
```

### The "New Feature Test" 🧪

**Ask yourself:** *"Can I add a new feature without modifying existing code?"*

- If **YES** → ✅ **OCP Followed**
- If **NO** (need to modify existing classes) → ❌ **OCP Violated**

---

## 💰 Scenario 1: Discount & Promotion System

### ❌ BAD Example - Violating OCP

```java
/**
 * 🚫 VIOLATION: Need to modify this class for every new discount type!
 * This is a classic OCP violation - the switch/if-else anti-pattern
 */

@Service
public class DiscountCalculator {
    
    /**
     * 🔥 PROBLEM: Adding new discount types requires MODIFYING this method
     * - What if we want to add:
     *   • Buy 2 Get 1 Free?
     *   • Student discount?
     *   • Military discount?
     *   • Flash sale discount?
     *   • Bundle discount?
     *   • Referral discount?
     * 
     * Each new type = MODIFY this class = RISK breaking existing discounts!
     */
    public double calculateDiscount(Order order, String discountType, String code) {
        
        double discount = 0;
        
        // 🔥 Long if-else chain that grows forever
        if (discountType.equals("PERCENTAGE")) {
            // Percentage discount logic
            if (code.equals("SAVE10")) {
                discount = order.getSubtotal() * 0.10;
            } else if (code.equals("SAVE20")) {
                discount = order.getSubtotal() * 0.20;
            } else if (code.equals("VIP30")) {
                discount = order.getSubtotal() * 0.30;
            }
            
        } else if (discountType.equals("FIXED_AMOUNT")) {
            // Fixed amount discount logic
            if (code.equals("FLAT5")) {
                discount = 5.0;
            } else if (code.equals("FLAT10")) {
                discount = 10.0;
            } else if (code.equals("FLAT25")) {
                discount = 25.0;
            }
            
        } else if (discountType.equals("FIRST_TIME_USER")) {
            // First time user discount
            if (order.isFirstOrder()) {
                discount = order.getSubtotal() * 0.15;
            }
            
        } else if (discountType.equals("SEASONAL")) {
            // Seasonal discount logic
            LocalDate now = LocalDate.now();
            int month = now.getMonthValue();
            
            if (month == 11 || month == 12) { // Holiday season
                discount = order.getSubtotal() * 0.25;
            } else if (month == 7) { // Summer sale
                discount = order.getSubtotal() * 0.15;
            }
            
        } else if (discountType.equals("LOYALTY_POINTS")) {
            // Loyalty points discount
            int points = order.getCustomer().getLoyaltyPoints();
            if (points > 1000) {
                discount = 50.0;
            } else if (points > 500) {
                discount = 25.0;
            } else if (points > 100) {
                discount = 10.0;
            }
            
        } else if (discountType.equals("BULK_PURCHASE")) {
            // Bulk purchase discount
            int totalItems = order.getItems().stream()
                .mapToInt(OrderItem::getQuantity)
                .sum();
            
            if (totalItems >= 10) {
                discount = order.getSubtotal() * 0.20;
            } else if (totalItems >= 5) {
                discount = order.getSubtotal() * 0.10;
            }
            
        } else if (discountType.equals("CATEGORY_SPECIFIC")) {
            // Category specific discount
            double categoryTotal = order.getItems().stream()
                .filter(item -> item.getCategory().equals("ELECTRONICS"))
                .mapToDouble(item -> item.getPrice() * item.getQuantity())
                .sum();
            
            discount = categoryTotal * 0.15;
            
        } else if (discountType.equals("MINIMUM_PURCHASE")) {
            // Minimum purchase discount
            if (order.getSubtotal() >= 100) {
                discount = 15.0;
            } else if (order.getSubtotal() >= 50) {
                discount = 5.0;
            }
            
        } else if (discountType.equals("COMBO_DEAL")) {
            // Combo deal logic
            boolean hasLaptop = order.getItems().stream()
                .anyMatch(item -> item.getCategory().equals("LAPTOP"));
            boolean hasMouse = order.getItems().stream()
                .anyMatch(item -> item.getCategory().equals("MOUSE"));
            
            if (hasLaptop && hasMouse) {
                discount = 30.0;
            }
            
        } else if (discountType.equals("CLEARANCE")) {
            // Clearance discount
            discount = order.getItems().stream()
                .filter(OrderItem::isClearanceItem)
                .mapToDouble(item -> item.getPrice() * item.getQuantity() * 0.50)
                .sum();
        }
        // 🔥 MORE ELSE-IFS WILL BE ADDED HERE...
        // 🔥 THIS METHOD GROWS TO 500+ LINES!
        
        return Math.min(discount, order.getSubtotal()); // Can't discount more than order
    }
}

/**
 * 🔥 CLIENT CODE - Also needs modification for new discount types
 */
@RestController
@RequestMapping("/api/discounts")
public class DiscountController {
    
    @Autowired
    private DiscountCalculator discountCalculator;
    
    @PostMapping("/apply")
    public ResponseEntity<DiscountResponse> applyDiscount(@RequestBody DiscountRequest request) {
        
        // 🔥 Client needs to know all discount types
        String discountType = request.getDiscountType();
        
        // 🔥 Validation for each type
        if (!isValidDiscountType(discountType)) {
            return ResponseEntity.badRequest()
                .body(new DiscountResponse("Invalid discount type"));
        }
        
        double discount = discountCalculator.calculateDiscount(
            request.getOrder(), 
            discountType, 
            request.getCode()
        );
        
        return ResponseEntity.ok(new DiscountResponse(discount));
    }
    
    // 🔥 Need to update this for every new discount type
    private boolean isValidDiscountType(String type) {
        return type.equals("PERCENTAGE") ||
               type.equals("FIXED_AMOUNT") ||
               type.equals("FIRST_TIME_USER") ||
               type.equals("SEASONAL") ||
               type.equals("LOYALTY_POINTS") ||
               type.equals("BULK_PURCHASE") ||
               type.equals("CATEGORY_SPECIFIC") ||
               type.equals("MINIMUM_PURCHASE") ||
               type.equals("COMBO_DEAL") ||
               type.equals("CLEARANCE");
        // 🔥 GROWS WITH EVERY NEW DISCOUNT TYPE!
    }
}
```

### 🔥 Problems with Above Code

| Problem | Impact | Why It's Bad |
|---------|--------|--------------|
| **Modify for every new discount** | High risk | 💥 Might break existing discounts |
| **500+ line method** | Unmaintainable | 😱 Impossible to understand |
| **No separation of concerns** | Tight coupling | 🔗 Everything in one place |
| **Hard to test** | Low coverage | 🧪 Need to test all paths together |
| **Merge conflicts** | Team friction | 👥 Everyone modifying same file |
| **Can't add discounts at runtime** | Inflexible | 🚫 Need code deployment |
| **Duplicate logic** | Code smell | 📝 Similar code repeated |

### Real Production Horror Story 😱

```java
/**
 * ACTUAL INCIDENT FROM PRODUCTION:
 * 
 * Timeline:
 * - 2 PM: Marketing requests "Black Friday 40% off" discount
 * - 2:30 PM: Developer adds new else-if to calculateDiscount()
 * - 3 PM: QA tests Black Friday discount ✅ Works!
 * - 3:30 PM: Deployed to production
 * - 4 PM: 💥 ALL PERCENTAGE DISCOUNTS STOP WORKING!
 * 
 * What happened?
 * Developer accidentally changed this line:
 * 
 * BEFORE:
 * if (code.equals("SAVE10")) {
 *     discount = order.getSubtotal() * 0.10;
 * }
 * 
 * AFTER (typo):
 * if (code.equals("SAVE10")) {
 *     discount = order.getSubtotal() * 0.01;  // ← TYPO! 10% became 1%
 * }
 * 
 * Impact:
 * - Lost $100,000 in sales (customers saw wrong discounts)
 * - 2,000 customer complaints
 * - Emergency rollback
 * - Team worked till midnight fixing
 * 
 * Root Cause: VIOLATED OPEN/CLOSED PRINCIPLE
 * - Adding new feature (Black Friday) required modifying existing code
 * - No protection against breaking existing features
 * - All discounts in one giant method
 */
```

---

### ✅ GOOD Example - Following OCP

```java
/**
 * ✨ SOLUTION: Use Strategy Pattern + OCP
 * Adding new discount types = Create new class (EXTEND)
 * NO modification to existing code (CLOSED)
 */

// ========================================
// 1️⃣ ABSTRACTION - The contract all discounts follow
// ========================================

/**
 * ✨ This interface is CLOSED for modification
 * But system is OPEN for extension via new implementations
 */
public interface DiscountStrategy {
    
    /**
     * Calculate discount amount for an order
     * @param order The order to calculate discount for
     * @return The discount amount
     */
    double calculateDiscount(Order order);
    
    /**
     * Check if this discount is applicable
     * @param order The order to check
     * @return true if discount can be applied
     */
    boolean isApplicable(Order order);
    
    /**
     * Get discount description
     * @return Human-readable description
     */
    String getDescription();
    
    /**
     * Get discount type identifier
     * @return Unique identifier for this discount
     */
    String getType();
}

// ========================================
// 2️⃣ CONCRETE IMPLEMENTATIONS - Each discount type
// ========================================

/**
 * ✨ Percentage Discount - CLOSED (never needs modification)
 */
@Component
public class PercentageDiscountStrategy implements DiscountStrategy {
    
    private final double percentage;
    private final String code;
    
    public PercentageDiscountStrategy(String code, double percentage) {
        this.code = code;
        this.percentage = percentage;
    }
    
    @Override
    public double calculateDiscount(Order order) {
        return order.getSubtotal() * (percentage / 100);
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return code.equalsIgnoreCase(order.getDiscountCode());
    }
    
    @Override
    public String getDescription() {
        return String.format("%.0f%% off your order", percentage);
    }
    
    @Override
    public String getType() {
        return "PERCENTAGE_DISCOUNT";
    }
}

/**
 * ✨ Fixed Amount Discount - CLOSED
 */
@Component
public class FixedAmountDiscountStrategy implements DiscountStrategy {
    
    private final double amount;
    private final String code;
    
    public FixedAmountDiscountStrategy(String code, double amount) {
        this.code = code;
        this.amount = amount;
    }
    
    @Override
    public double calculateDiscount(Order order) {
        return Math.min(amount, order.getSubtotal());
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return code.equalsIgnoreCase(order.getDiscountCode());
    }
    
    @Override
    public String getDescription() {
        return String.format("$%.2f off your order", amount);
    }
    
    @Override
    public String getType() {
        return "FIXED_AMOUNT_DISCOUNT";
    }
}

/**
 * ✨ First Time User Discount - CLOSED
 */
@Component
public class FirstTimeUserDiscountStrategy implements DiscountStrategy {
    
    private final double percentage = 15.0;
    
    @Override
    public double calculateDiscount(Order order) {
        if (order.getCustomer().isFirstTimeCustomer()) {
            return order.getSubtotal() * (percentage / 100);
        }
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getCustomer().isFirstTimeCustomer();
    }
    
    @Override
    public String getDescription() {
        return "First-time customer discount: 15% off";
    }
    
    @Override
    public String getType() {
        return "FIRST_TIME_USER_DISCOUNT";
    }
}

/**
 * ✨ Seasonal Discount - CLOSED
 */
@Component
public class SeasonalDiscountStrategy implements DiscountStrategy {
    
    @Override
    public double calculateDiscount(Order order) {
        int month = LocalDate.now().getMonthValue();
        
        // Holiday season (Nov-Dec): 25% off
        if (month == 11 || month == 12) {
            return order.getSubtotal() * 0.25;
        }
        
        // Summer sale (July): 15% off
        if (month == 7) {
            return order.getSubtotal() * 0.15;
        }
        
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        int month = LocalDate.now().getMonthValue();
        return month == 11 || month == 12 || month == 7;
    }
    
    @Override
    public String getDescription() {
        int month = LocalDate.now().getMonthValue();
        if (month == 11 || month == 12) {
            return "Holiday Season Sale: 25% off";
        } else if (month == 7) {
            return "Summer Sale: 15% off";
        }
        return "Seasonal discount";
    }
    
    @Override
    public String getType() {
        return "SEASONAL_DISCOUNT";
    }
}

/**
 * ✨ Loyalty Points Discount - CLOSED
 */
@Component
public class LoyaltyPointsDiscountStrategy implements DiscountStrategy {
    
    @Override
    public double calculateDiscount(Order order) {
        int points = order.getCustomer().getLoyaltyPoints();
        
        if (points >= 1000) {
            return 50.0;
        } else if (points >= 500) {
            return 25.0;
        } else if (points >= 100) {
            return 10.0;
        }
        
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getCustomer().getLoyaltyPoints() >= 100;
    }
    
    @Override
    public String getDescription() {
        return "Loyalty rewards discount";
    }
    
    @Override
    public String getType() {
        return "LOYALTY_POINTS_DISCOUNT";
    }
}

/**
 * ✨ Bulk Purchase Discount - CLOSED
 */
@Component
public class BulkPurchaseDiscountStrategy implements DiscountStrategy {
    
    @Override
    public double calculateDiscount(Order order) {
        int totalItems = order.getItems().stream()
            .mapToInt(OrderItem::getQuantity)
            .sum();
        
        if (totalItems >= 10) {
            return order.getSubtotal() * 0.20; // 20% off
        } else if (totalItems >= 5) {
            return order.getSubtotal() * 0.10; // 10% off
        }
        
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        int totalItems = order.getItems().stream()
            .mapToInt(OrderItem::getQuantity)
            .sum();
        return totalItems >= 5;
    }
    
    @Override
    public String getDescription() {
        return "Bulk purchase discount: Buy more, save more!";
    }
    
    @Override
    public String getType() {
        return "BULK_PURCHASE_DISCOUNT";
    }
}

/**
 * ✨ Category-Specific Discount - CLOSED
 */
@Component
public class CategoryDiscountStrategy implements DiscountStrategy {
    
    private final String category;
    private final double percentage;
    
    public CategoryDiscountStrategy(String category, double percentage) {
        this.category = category;
        this.percentage = percentage;
    }
    
    @Override
    public double calculateDiscount(Order order) {
        double categoryTotal = order.getItems().stream()
            .filter(item -> category.equalsIgnoreCase(item.getCategory()))
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        
        return categoryTotal * (percentage / 100);
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getItems().stream()
            .anyMatch(item -> category.equalsIgnoreCase(item.getCategory()));
    }
    
    @Override
    public String getDescription() {
        return String.format("%.0f%% off on %s items", percentage, category);
    }
    
    @Override
    public String getType() {
        return "CATEGORY_DISCOUNT";
    }
}

/**
 * ✨ Minimum Purchase Discount - CLOSED
 */
@Component
public class MinimumPurchaseDiscountStrategy implements DiscountStrategy {
    
    private final double minimumAmount;
    private final double discountAmount;
    
    public MinimumPurchaseDiscountStrategy(double minimumAmount, double discountAmount) {
        this.minimumAmount = minimumAmount;
        this.discountAmount = discountAmount;
    }
    
    @Override
    public double calculateDiscount(Order order) {
        if (order.getSubtotal() >= minimumAmount) {
            return discountAmount;
        }
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getSubtotal() >= minimumAmount;
    }
    
    @Override
    public String getDescription() {
        return String.format("Spend $%.2f, get $%.2f off", minimumAmount, discountAmount);
    }
    
    @Override
    public String getType() {
        return "MINIMUM_PURCHASE_DISCOUNT";
    }
}

/**
 * ✨ Buy X Get Y Free Discount - CLOSED
 */
@Component
public class BuyXGetYDiscountStrategy implements DiscountStrategy {
    
    private final int buyQuantity;
    private final int freeQuantity;
    
    public BuyXGetYDiscountStrategy(int buyQuantity, int freeQuantity) {
        this.buyQuantity = buyQuantity;
        this.freeQuantity = freeQuantity;
    }
    
    @Override
    public double calculateDiscount(Order order) {
        double totalDiscount = 0;
        
        for (OrderItem item : order.getItems()) {
            int quantity = item.getQuantity();
            int sets = quantity / (buyQuantity + freeQuantity);
            int freeItems = sets * freeQuantity;
            
            totalDiscount += freeItems * item.getPrice();
        }
        
        return totalDiscount;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getItems().stream()
            .anyMatch(item -> item.getQuantity() >= (buyQuantity + freeQuantity));
    }
    
    @Override
    public String getDescription() {
        return String.format("Buy %d Get %d Free!", buyQuantity, freeQuantity);
    }
    
    @Override
    public String getType() {
        return "BUY_X_GET_Y_DISCOUNT";
    }
}

// ========================================
// 3️⃣ WANT TO ADD NEW DISCOUNT TYPE? JUST CREATE NEW CLASS!
// ========================================

/**
 * ✨ NEW: Student Discount - Added without modifying existing code!
 * This class was added 6 months after initial release
 * NO existing code was touched!
 */
@Component
public class StudentDiscountStrategy implements DiscountStrategy {
    
    private final double percentage = 20.0;
    
    @Autowired
    private StudentVerificationService verificationService;
    
    @Override
    public double calculateDiscount(Order order) {
        if (verificationService.isVerifiedStudent(order.getCustomer())) {
            return order.getSubtotal() * (percentage / 100);
        }
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return verificationService.isVerifiedStudent(order.getCustomer());
    }
    
    @Override
    public String getDescription() {
        return "Student discount: 20% off with valid student ID";
    }
    
    @Override
    public String getType() {
        return "STUDENT_DISCOUNT";
    }
}

/**
 * ✨ NEW: Referral Discount - Also added later without touching existing code!
 */
@Component
public class ReferralDiscountStrategy implements DiscountStrategy {
    
    private final double referrerBonus = 10.0;
    private final double referredBonus = 15.0;
    
    @Override
    public double calculateDiscount(Order order) {
        String referralCode = order.getReferralCode();
        
        if (referralCode != null && !referralCode.isEmpty()) {
            // New customer gets bigger discount
            if (order.getCustomer().isFirstTimeCustomer()) {
                return referredBonus;
            }
            // Referring customer gets smaller discount
            return referrerBonus;
        }
        
        return 0;
    }
    
    @Override
    public boolean isApplicable(Order order) {
        return order.getReferralCode() != null && !order.getReferralCode().isEmpty();
    }
    
    @Override
    public String getDescription() {
        return "Referral discount: Refer a friend and both save!";
    }
    
    @Override
    public String getType() {
        return "REFERRAL_DISCOUNT";
    }
}

// ========================================
// 4️⃣ DISCOUNT SERVICE - Manages all discounts
// ========================================

/**
 * ✨ This service is CLOSED for modification
 * New discount types automatically picked up via dependency injection!
 */
@Service
public class DiscountService {
    
    private final List<DiscountStrategy> allStrategies;
    
    /**
     * ✨ Spring auto-injects ALL implementations of DiscountStrategy
     * When new discount class is created, it's automatically available!
     */
    @Autowired
    public DiscountService(List<DiscountStrategy> allStrategies) {
        this.allStrategies = allStrategies;
    }
    
    /**
     * Calculate best discount for an order
     * NO modification needed when new discount types are added!
     */
    public DiscountResult calculateBestDiscount(Order order) {
        
        double maxDiscount = 0;
        DiscountStrategy bestStrategy = null;
        
        // Check all available discounts
        for (DiscountStrategy strategy : allStrategies) {
            if (strategy.isApplicable(order)) {
                double discount = strategy.calculateDiscount(order);
                
                if (discount > maxDiscount) {
                    maxDiscount = discount;
                    bestStrategy = strategy;
                }
            }
        }
        
        if (bestStrategy != null) {
            return DiscountResult.success(
                maxDiscount, 
                bestStrategy.getDescription(),
                bestStrategy.getType()
            );
        }
        
        return DiscountResult.noDiscount();
    }
    
    /**
     * Get all applicable discounts for an order
     */
    public List<DiscountResult> getAllApplicableDiscounts(Order order) {
        return allStrategies.stream()
            .filter(strategy -> strategy.isApplicable(order))
            .map(strategy -> DiscountResult.success(
                strategy.calculateDiscount(order),
                strategy.getDescription(),
                strategy.getType()
            ))
            .sorted(Comparator.comparing(DiscountResult::getAmount).reversed())
            .collect(Collectors.toList());
    }
    
    /**
     * Apply specific discount type
     */
    public DiscountResult applyDiscount(Order order, String discountType) {
        return allStrategies.stream()
            .filter(strategy -> strategy.getType().equals(discountType))
            .filter(strategy -> strategy.isApplicable(order))
            .findFirst()
            .map(strategy -> DiscountResult.success(
                strategy.calculateDiscount(order),
                strategy.getDescription(),
                strategy.getType()
            ))
            .orElse(DiscountResult.notApplicable("Discount not applicable"));
    }
}

// ========================================
// 5️⃣ DISCOUNT RESULT - Value object
// ========================================

@Data
@Builder
@AllArgsConstructor
public class DiscountResult {
    private boolean applied;
    private double amount;
    private String description;
    private String type;
    private String message;
    
    public static DiscountResult success(double amount, String description, String type) {
        return DiscountResult.builder()
            .applied(true)
            .amount(amount)
            .description(description)
            .type(type)
            .build();
    }
    
    public static DiscountResult noDiscount() {
        return DiscountResult.builder()
            .applied(false)
            .amount(0)
            .message("No applicable discounts")
            .build();
    }
    
    public static DiscountResult notApplicable(String message) {
        return DiscountResult.builder()
            .applied(false)
            .amount(0)
            .message(message)
            .build();
    }
}

// ========================================
// 6️⃣ CONTROLLER - Also CLOSED for modification
// ========================================

@RestController
@RequestMapping("/api/discounts")
public class DiscountController {
    
    @Autowired
    private DiscountService discountService;
    
    /**
     * ✨ NO modification needed when new discount types are added!
     */
    @PostMapping("/calculate")
    public ResponseEntity<DiscountResponse> calculateDiscount(@RequestBody Order order) {
        
        DiscountResult result = discountService.calculateBestDiscount(order);
        
        return ResponseEntity.ok(new DiscountResponse(
            result.isApplied(),
            result.getAmount(),
            result.getDescription()
        ));
    }
    
    /**
     * Get all available discounts for customer
     */
    @PostMapping("/available")
    public ResponseEntity<List<DiscountResult>> getAvailableDiscounts(@RequestBody Order order) {
        
        List<DiscountResult> discounts = discountService.getAllApplicableDiscounts(order);
        
        return ResponseEntity.ok(discounts);
    }
}

// ========================================
// 7️⃣ CONFIGURATION - Runtime discount management
// ========================================

/**
 * ✨ BONUS: Configure discounts without code changes!
 */
@Configuration
public class DiscountConfiguration {
    
    /**
     * Define discount codes via configuration
     * Can be moved to database for runtime management!
     */
    @Bean
    public DiscountStrategy save10Discount() {
        return new PercentageDiscountStrategy("SAVE10", 10.0);
    }
    
    @Bean
    public DiscountStrategy save20Discount() {
        return new PercentageDiscountStrategy("SAVE20", 20.0);
    }
    
    @Bean
    public DiscountStrategy vip30Discount() {
        return new PercentageDiscountStrategy("VIP30", 30.0);
    }
    
    @Bean
    public DiscountStrategy flat5Discount() {
        return new FixedAmountDiscountStrategy("FLAT5", 5.0);
    }
    
    @Bean
    public DiscountStrategy electronicsDiscount() {
        return new CategoryDiscountStrategy("ELECTRONICS", 15.0);
    }
    
    @Bean
    public DiscountStrategy spend100Get15() {
        return new MinimumPurchaseDiscountStrategy(100.0, 15.0);
    }
    
    @Bean
    public DiscountStrategy buy2Get1Free() {
        return new BuyXGetYDiscountStrategy(2, 1);
    }
    
    // ✨ Adding new discount = Just add new @Bean method
    // NO modification to existing beans!
}
```

---

### ✨ Benefits of OCP Design

| Aspect | ❌ Before (Violation) | ✅ After (OCP) |
|--------|---------------------|---------------|
| **Adding New Discount** | Modify DiscountCalculator | Create new class |
| **Risk of Breaking** | High (touch existing code) | Zero (no existing code changed) |
| **Code Length** | 500+ lines in one method | 20-30 lines per class |
| **Testing** | Test everything together | Test each discount independently |
| **Deployment** | Full redeployment | Can add via configuration |
| **Parallel Development** | Merge conflicts | Independent work |
| **Runtime Changes** | Impossible | Possible with database config |

### 📊 Real Impact Metrics

```java
/**
 * 📈 BEFORE OCP (6 months):
 * - 12 new discount types added
 * - DiscountCalculator grew to 847 lines
 * - 23 bugs introduced
 * - 8 emergency hotfixes
 * - Average fix time: 4 hours
 * - Team morale: 😢
 * 
 * 📈 AFTER OCP (6 months):
 * - 15 new discount types added
 * - No existing code modified
 * - 2 bugs (only in new discounts)
 * - 0 emergency hotfixes
 * - Average fix time: 15 minutes
 * - Team morale: 😄
 * 
 * 💰 BUSINESS IMPACT:
 * - 90% reduction in bugs
 * - 3x faster feature development
 * - 95% reduction in production incidents
 * - Marketing can launch promotions same day
 */
```

---

## 💳 Scenario 2: Payment Processing System

### ❌ BAD Example

```java
/**
 * 🚫 VIOLATION: Adding new payment method requires code changes
 */
@Service
public class PaymentProcessor {
    
    public PaymentResult processPayment(PaymentRequest request) {
        
        String paymentMethod = request.getPaymentMethod();
        
        // 🔥 Need to modify this for every new payment method
        if ("CREDIT_CARD".equals(paymentMethod)) {
            return processCreditCard(request);
            
        } else if ("DEBIT_CARD".equals(paymentMethod)) {
            return processDebitCard(request);
            
        } else if ("PAYPAL".equals(paymentMethod)) {
            return processPayPal(request);
            
        } else if ("STRIPE".equals(paymentMethod)) {
            return processStripe(request);
            
        } else if ("VENMO".equals(paymentMethod)) {
            return processVenmo(request);
            
        } else if ("APPLE_PAY".equals(paymentMethod)) {
            return processApplePay(request);
            
        } else if ("GOOGLE_PAY".equals(paymentMethod)) {
            return processGooglePay(request);
            
        } else if ("BITCOIN".equals(paymentMethod)) {
            return processBitcoin(request);
            
        } else if ("BANK_TRANSFER".equals(paymentMethod)) {
            return processBankTransfer(request);
        }
        // 🔥 What about UPI, AliPay, WeChat Pay, etc.?
        
        throw new UnsupportedPaymentMethodException("Unknown payment method: " + paymentMethod);
    }
    
    // 🔥 Multiple private methods doing similar things
    private PaymentResult processCreditCard(PaymentRequest request) { /* ... */ }
    private PaymentResult processDebitCard(PaymentRequest request) { /* ... */ }
    private PaymentResult processPayPal(PaymentRequest request) { /* ... */ }
    // ... more methods
}
```

### ✅ GOOD Example

```java
/**
 * ✨ OCP-COMPLIANT: Payment processing using Strategy Pattern
 */

// Payment interface
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
    boolean supports(String paymentMethod);
    String getPaymentMethod();
}

// Credit Card implementation
@Service
public class CreditCardPaymentProcessor implements PaymentProcessor {
    
    @Value("${payment.gateway.creditcard.url}")
    private String gatewayUrl;
    
    @Override
    public PaymentResult process(PaymentRequest request) {
        // Credit card specific logic
        return PaymentResult.success("CC-" + UUID.randomUUID());
    }
    
    @Override
    public boolean supports(String paymentMethod) {
        return "CREDIT_CARD".equals(paymentMethod);
    }
    
    @Override
    public String getPaymentMethod() {
        return "CREDIT_CARD";
    }
}

// PayPal implementation
@Service
public class PayPalPaymentProcessor implements PaymentProcessor {
    
    @Autowired
    private PayPalClient payPalClient;
    
    @Override
    public PaymentResult process(PaymentRequest request) {
        // PayPal specific logic
        String transactionId = payPalClient.charge(
            request.getAmount(), 
            request.getPayPalToken()
        );
        return PaymentResult.success(transactionId);
    }
    
    @Override
    public boolean supports(String paymentMethod) {
        return "PAYPAL".equals(paymentMethod);
    }
    
    @Override
    public String getPaymentMethod() {
        return "PAYPAL";
    }
}

// ✨ NEW: Cryptocurrency payment (added 1 year later, no existing code touched!)
@Service
public class CryptoPaymentProcessor implements PaymentProcessor {
    
    @Autowired
    private BlockchainService blockchainService;
    
    @Override
    public PaymentResult process(PaymentRequest request) {
        String walletAddress = request.getWalletAddress();
        String txHash = blockchainService.sendTransaction(
            walletAddress,
            request.getAmount()
        );
        return PaymentResult.success(txHash);
    }
    
    @Override
    public boolean supports(String paymentMethod) {
        return "CRYPTOCURRENCY".equals(paymentMethod);
    }
    
    @Override
    public String getPaymentMethod() {
        return "CRYPTOCURRENCY";
    }
}

// Payment service - CLOSED for modification
@Service
public class PaymentService {
    
    private final Map<String, PaymentProcessor> processors;
    
    @Autowired
    public PaymentService(List<PaymentProcessor> processorList) {
        // Auto-discover all payment processors
        this.processors = processorList.stream()
            .collect(Collectors.toMap(
                PaymentProcessor::getPaymentMethod,
                Function.identity()
            ));
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        PaymentProcessor processor = processors.get(request.getPaymentMethod());
        
        if (processor == null) {
            return PaymentResult.failure("Unsupported payment method");
        }
        
        return processor.process(request);
    }
    
    public List<String> getSupportedPaymentMethods() {
        return new ArrayList<>(processors.keySet());
    }
}
```

---

## 🚚 Scenario 3: Shipping Calculation

### ✅ GOOD Example - Extensible Shipping

```java
/**
 * ✨ OCP-COMPLIANT: Shipping calculation system
 */

public interface ShippingCalculator {
    double calculateCost(ShippingRequest request);
    int getEstimatedDays(ShippingRequest request);
    boolean supports(String shippingMethod);
    String getShippingMethod();
}

@Service
public class StandardShippingCalculator implements ShippingCalculator {
    
    private static final double BASE_RATE = 5.99;
    private static final double PER_KG_RATE = 2.50;
    
    @Override
    public double calculateCost(ShippingRequest request) {
        double weight = request.getTotalWeight();
        double distance = calculateDistance(request.getOrigin(), request.getDestination());
        
        double cost = BASE_RATE + (weight * PER_KG_RATE);
        
        // Free shipping for orders over $100
        if (request.getOrderTotal() >= 100) {
            return 0;
        }
        
        return cost;
    }
    
    @Override
    public int getEstimatedDays(ShippingRequest request) {
        return 5; // 5 business days
    }
    
    @Override
    public boolean supports(String shippingMethod) {
        return "STANDARD".equals(shippingMethod);
    }
    
    @Override
    public String getShippingMethod() {
        return "STANDARD";
    }
    
    private double calculateDistance(Address origin, Address destination) {
        // Distance calculation logic
        return 100.0; // Simplified
    }
}

@Service
public class ExpressShippingCalculator implements ShippingCalculator {
    
    private static final double BASE_RATE = 15.99;
    private static final double PER_KG_RATE = 5.00;
    
    @Override
    public double calculateCost(ShippingRequest request) {
        double weight = request.getTotalWeight();
        return BASE_RATE + (weight * PER_KG_RATE);
    }
    
    @Override
    public int getEstimatedDays(ShippingRequest request) {
        return 2; // 2 business days
    }
    
    @Override
    public boolean supports(String shippingMethod) {
        return "EXPRESS".equals(shippingMethod);
    }
    
    @Override
    public String getShippingMethod() {
        return "EXPRESS";
    }
}

@Service
public class OvernightShippingCalculator implements ShippingCalculator {
    
    private static final double BASE_RATE = 25.99;
    private static final double PER_KG_RATE = 10.00;
    
    @Override
    public double calculateCost(ShippingRequest request) {
        double weight = request.getTotalWeight();
        double urgencySurcharge = 5.00;
        
        return BASE_RATE + (weight * PER_KG_RATE) + urgencySurcharge;
    }
    
    @Override
    public int getEstimatedDays(ShippingRequest request) {
        return 1; // Next business day
    }
    
    @Override
    public boolean supports(String shippingMethod) {
        return "OVERNIGHT".equals(shippingMethod);
    }
    
    @Override
    public String getShippingMethod() {
        return "OVERNIGHT";
    }
}

// ✨ NEW: International shipping (added later, no modification to existing code!)
@Service
public class InternationalShippingCalculator implements ShippingCalculator {
    
    @Autowired
    private CustomsDutyCalculator customsCalculator;
    
    @Override
    public double calculateCost(ShippingRequest request) {
        double baseCost = 50.00;
        double weight = request.getTotalWeight();
        double perKgRate = 15.00;
        
        double shippingCost = baseCost + (weight * perKgRate);
        double customsDuty = customsCalculator.calculate(request);
        
        return shippingCost + customsDuty;
    }
    
    @Override
    public int getEstimatedDays(ShippingRequest request) {
        return 10; // 10-14 business days
    }
    
    @Override
    public boolean supports(String shippingMethod) {
        return "INTERNATIONAL".equals(shippingMethod);
    }
    
    @Override
    public String getShippingMethod() {
        return "INTERNATIONAL";
    }
}

// Shipping service
@Service
public class ShippingService {
    
    private final Map<String, ShippingCalculator> calculators;
    
    @Autowired
    public ShippingService(List<ShippingCalculator> calculatorList) {
        this.calculators = calculatorList.stream()
            .collect(Collectors.toMap(
                ShippingCalculator::getShippingMethod,
                Function.identity()
            ));
    }
    
    public ShippingQuote getQuote(ShippingRequest request) {
        ShippingCalculator calculator = calculators.get(request.getShippingMethod());
        
        if (calculator == null) {
            throw new UnsupportedShippingMethodException(
                "Shipping method not supported: " + request.getShippingMethod()
            );
        }
        
        double cost = calculator.calculateCost(request);
        int estimatedDays = calculator.getEstimatedDays(request);
        
        return new ShippingQuote(cost, estimatedDays, request.getShippingMethod());
    }
    
    public List<ShippingQuote> getAllQuotes(ShippingRequest request) {
        return calculators.values().stream()
            .map(calculator -> new ShippingQuote(
                calculator.calculateCost(request),
                calculator.getEstimatedDays(request),
                calculator.getShippingMethod()
            ))
            .sorted(Comparator.comparing(ShippingQuote::getCost))
            .collect(Collectors.toList());
    }
}
```

---

## 🔔 Scenario 4: Notification System

### ✅ GOOD Example - Extensible Notifications

```java
/**
 * ✨ OCP-COMPLIANT: Multi-channel notification system
 */

public interface NotificationChannel {
    void send(Notification notification);
    boolean supports(String channelType);
    String getChannelType();
}

@Service
public class EmailNotificationChannel implements NotificationChannel {
    
    @Autowired
    private JavaMailSender mailSender;
    
    @Override
    public void send(Notification notification) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            
            helper.setTo(notification.getRecipient());
            helper.setSubject(notification.getSubject());
            helper.setText(notification.getBody(), true);
            
            mailSender.send(message);
            
            System.out.println("�� Email sent to: " + notification.getRecipient());
            
        } catch (Exception e) {
            throw new NotificationException("Failed to send email", e);
        }
    }
    
    @Override
    public boolean supports(String channelType) {
        return "EMAIL".equals(channelType);
    }
    
    @Override
    public String getChannelType() {
        return "EMAIL";
    }
}

@Service
public class SMSNotificationChannel implements NotificationChannel {
    
    @Value("${sms.provider.url}")
    private String smsProviderUrl;
    
    private final HttpClient httpClient = HttpClient.newHttpClient();
    
    @Override
    public void send(Notification notification) {
        try {
            String requestBody = String.format(
                "{\"to\":\"%s\", \"message\":\"%s\"}",
                notification.getRecipient(),
                notification.getBody()
            );
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(smsProviderUrl))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
            
            httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            
            System.out.println("📱 SMS sent to: " + notification.getRecipient());
            
        } catch (Exception e) {
            throw new NotificationException("Failed to send SMS", e);
        }
    }
    
    @Override
    public boolean supports(String channelType) {
        return "SMS".equals(channelType);
    }
    
    @Override
    public String getChannelType() {
        return "SMS";
    }
}

@Service
public class PushNotificationChannel implements NotificationChannel {
    
    @Autowired
    private FirebaseMessaging firebaseMessaging;
    
    @Override
    public void send(Notification notification) {
        try {
            Message message = Message.builder()
                .setToken(notification.getDeviceToken())
                .setNotification(com.google.firebase.messaging.Notification.builder()
                    .setTitle(notification.getSubject())
                    .setBody(notification.getBody())
                    .build())
                .build();
            
            firebaseMessaging.send(message);
            
            System.out.println("🔔 Push notification sent to device: " + notification.getDeviceToken());
            
        } catch (Exception e) {
            throw new NotificationException("Failed to send push notification", e);
        }
    }
    
    @Override
    public boolean supports(String channelType) {
        return "PUSH".equals(channelType);
    }
    
    @Override
    public String getChannelType() {
        return "PUSH";
    }
}

// ✨ NEW: Slack notifications (added 6 months later)
@Service
public class SlackNotificationChannel implements NotificationChannel {
    
    @Value("${slack.webhook.url}")
    private String webhookUrl;
    
    private final HttpClient httpClient = HttpClient.newHttpClient();
    
    @Override
    public void send(Notification notification) {
        try {
            String slackMessage = String.format(
                "{\"text\":\"%s\\n%s\"}",
                notification.getSubject(),
                notification.getBody()
            );
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(webhookUrl))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(slackMessage))
                .build();
            
            httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            
            System.out.println("💬 Slack notification sent to channel");
            
        } catch (Exception e) {
            throw new NotificationException("Failed to send Slack notification", e);
        }
    }
    
    @Override
    public boolean supports(String channelType) {
        return "SLACK".equals(channelType);
    }
    
    @Override
    public String getChannelType() {
        return "SLACK";
    }
}

// ✨ NEW: WhatsApp notifications (added 1 year later)
@Service
public class WhatsAppNotificationChannel implements NotificationChannel {
    
    @Autowired
    private WhatsAppBusinessClient whatsAppClient;
    
    @Override
    public void send(Notification notification) {
        whatsAppClient.sendMessage(
            notification.getRecipient(),
            notification.getBody()
        );
        
        System.out.println("💚 WhatsApp message sent to: " + notification.getRecipient());
    }
    
    @Override
    public boolean supports(String channelType) {
        return "WHATSAPP".equals(channelType);
    }
    
    @Override
    public String getChannelType() {
        return "WHATSAPP";
    }
}

// Notification service - CLOSED for modification
@Service
public class NotificationService {
    
    private final Map<String, NotificationChannel> channels;
    
    @Autowired
    public NotificationService(List<NotificationChannel> channelList) {
        this.channels = channelList.stream()
            .collect(Collectors.toMap(
                NotificationChannel::getChannelType,
                Function.identity()
            ));
    }
    
    /**
     * Send notification via specified channel
     */
    public void send(Notification notification, String channelType) {
        NotificationChannel channel = channels.get(channelType);
        
        if (channel == null) {
            throw new UnsupportedChannelException("Channel not supported: " + channelType);
        }
        
        channel.send(notification);
    }
    
    /**
     * Send notification via multiple channels
     */
    public void sendMultiChannel(Notification notification, List<String> channelTypes) {
        for (String channelType : channelTypes) {
            CompletableFuture.runAsync(() -> send(notification, channelType));
        }
    }
    
    /**
     * Broadcast to all available channels
     */
    public void broadcast(Notification notification) {
        channels.values().forEach(channel -> 
            CompletableFuture.runAsync(() -> channel.send(notification))
        );
    }
    
    public List<String> getAvailableChannels() {
        return new ArrayList<>(channels.keySet());
    }
}

// Usage example
@Service
public class OrderNotificationService {
    
    @Autowired
    private NotificationService notificationService;
    
    public void notifyOrderConfirmation(Order order, User user) {
        Notification notification = Notification.builder()
            .recipient(user.getEmail())
            .subject("Order Confirmation #" + order.getId())
            .body("Your order has been confirmed. Total: $" + order.getTotal())
            .build();
        
        // Send via multiple channels
        notificationService.sendMultiChannel(
            notification,
            Arrays.asList("EMAIL", "SMS", "PUSH")
        );
    }
}
```

---

## 📊 Scenario 5: Report Generation

### ✅ GOOD Example - Extensible Reports

```java
/**
 * ✨ OCP-COMPLIANT: Report generation system
 */

public interface ReportGenerator {
    byte[] generate(ReportData data);
    String getFormat();
    String getContentType();
}

@Service
public class PDFReportGenerator implements ReportGenerator {
    
    @Override
    public byte[] generate(ReportData data) {
        // Using iText or similar library
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            // PDF generation logic
            Document document = new Document();
            PdfWriter.getInstance(document, baos);
            document.open();
            
            document.add(new Paragraph("Report Title: " + data.getTitle()));
            document.add(new Paragraph("Generated: " + LocalDateTime.now()));
            
            // Add data
            for (ReportRow row : data.getRows()) {
                document.add(new Paragraph(row.toString()));
            }
            
            document.close();
            
            return baos.toByteArray();
            
        } catch (Exception e) {
            throw new ReportGenerationException("Failed to generate PDF", e);
        }
    }
    
    @Override
    public String getFormat() {
        return "PDF";
    }
    
    @Override
    public String getContentType() {
        return "application/pdf";
    }
}

@Service
public class ExcelReportGenerator implements ReportGenerator {
    
    @Override
    public byte[] generate(ReportData data) {
        try (Workbook workbook = new XSSFWorkbook();
             ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            
            Sheet sheet = workbook.createSheet("Report");
            
            // Header row
            Row headerRow = sheet.createRow(0);
            List<String> headers = data.getHeaders();
            for (int i = 0; i < headers.size(); i++) {
                headerRow.createCell(i).setCellValue(headers.get(i));
            }
            
            // Data rows
            List<ReportRow> rows = data.getRows();
            for (int i = 0; i < rows.size(); i++) {
                Row row = sheet.createRow(i + 1);
                ReportRow reportRow = rows.get(i);
                
                for (int j = 0; j < reportRow.getValues().size(); j++) {
                    row.createCell(j).setCellValue(reportRow.getValues().get(j));
                }
            }
            
            workbook.write(baos);
            return baos.toByteArray();
            
        } catch (Exception e) {
            throw new ReportGenerationException("Failed to generate Excel", e);
        }
    }
    
    @Override
    public String getFormat() {
        return "EXCEL";
    }
    
    @Override
    public String getContentType() {
        return "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
    }
}

@Service
public class CSVReportGenerator implements ReportGenerator {
    
    @Override
    public byte[] generate(ReportData data) {
        StringBuilder csv = new StringBuilder();
        
        // Headers
        csv.append(String.join(",", data.getHeaders())).append("\n");
        
        // Rows
        for (ReportRow row : data.getRows()) {
            csv.append(String.join(",", row.getValues())).append("\n");
        }
        
        return csv.toString().getBytes(StandardCharsets.UTF_8);
    }
    
    @Override
    public String getFormat() {
        return "CSV";
    }
    
    @Override
    public String getContentType() {
        return "text/csv";
    }
}

// ✨ NEW: JSON report format (added later)
@Service
public class JSONReportGenerator implements ReportGenerator {
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Override
    public byte[] generate(ReportData data) {
        try {
            Map<String, Object> report = new HashMap<>();
            report.put("title", data.getTitle());
            report.put("generatedAt", LocalDateTime.now());
            report.put("headers", data.getHeaders());
            report.put("rows", data.getRows());
            
            return objectMapper.writeValueAsBytes(report);
            
        } catch (Exception e) {
            throw new ReportGenerationException("Failed to generate JSON", e);
        }
    }
    
    @Override
    public String getFormat() {
        return "JSON";
    }
    
    @Override
    public String getContentType() {
        return "application/json";
    }
}

// Report service
@Service
public class ReportService {
    
    private final Map<String, ReportGenerator> generators;
    
    @Autowired
    public ReportService(List<ReportGenerator> generatorList) {
        this.generators = generatorList.stream()
            .collect(Collectors.toMap(
                ReportGenerator::getFormat,
                Function.identity()
            ));
    }
    
    public ReportResponse generateReport(ReportRequest request) {
        ReportGenerator generator = generators.get(request.getFormat().toUpperCase());
        
        if (generator == null) {
            throw new UnsupportedFormatException("Format not supported: " + request.getFormat());
        }
        
        ReportData data = prepareReportData(request);
        byte[] content = generator.generate(data);
        
        return ReportResponse.builder()
            .content(content)
            .contentType(generator.getContentType())
            .filename("report_" + LocalDateTime.now() + "." + request.getFormat().toLowerCase())
            .build();
    }
    
    private ReportData prepareReportData(ReportRequest request) {
        // Fetch and prepare data based on request
        return new ReportData();
    }
    
    public List<String> getSupportedFormats() {
        return new ArrayList<>(generators.keySet());
    }
}

// Controller
@RestController
@RequestMapping("/api/reports")
public class ReportController {
    
    @Autowired
    private ReportService reportService;
    
    @GetMapping("/sales")
    public ResponseEntity<byte[]> generateSalesReport(
            @RequestParam(defaultValue = "PDF") String format) {
        
        ReportRequest request = new ReportRequest("SALES", format);
        ReportResponse response = reportService.generateReport(request);
        
        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, 
                   "attachment; filename=\"" + response.getFilename() + "\"")
            .contentType(MediaType.parseMediaType(response.getContentType()))
            .body(response.getContent());
    }
    
    @GetMapping("/formats")
    public ResponseEntity<List<String>> getSupportedFormats() {
        return ResponseEntity.ok(reportService.getSupportedFormats());
    }
}
```

---

## 💵 Scenario 6: Tax Calculation Engine

### ✅ GOOD Example

```java
/**
 * ✨ OCP-COMPLIANT: Flexible tax calculation
 */

public interface TaxCalculator {
    double calculate(TaxableOrder order);
    boolean appliesTo(Address address);
    String getTaxRegion();
}

@Service
public class USTaxCalculator implements TaxCalculator {
    
    private static final Map<String, Double> STATE_TAX_RATES = Map.of(
        "CA", 0.0725,
        "NY", 0.08,
        "TX", 0.0625,
        "FL", 0.06,
        "WA", 0.065
    );
    
    @Override
    public double calculate(TaxableOrder order) {
        String state = order.getShippingAddress().getState();
        double taxRate = STATE_TAX_RATES.getOrDefault(state, 0.05);
        
        return order.getSubtotal() * taxRate;
    }
    
    @Override
    public boolean appliesTo(Address address) {
        return "US".equals(address.getCountry());
    }
    
    @Override
    public String getTaxRegion() {
        return "US";
    }
}

@Service
public class EUVATCalculator implements TaxCalculator {
    
    private static final Map<String, Double> VAT_RATES = Map.of(
        "DE", 0.19,  // Germany
        "FR", 0.20,  // France
        "UK", 0.20,  // United Kingdom
        "IT", 0.22,  // Italy
        "ES", 0.21   // Spain
    );
    
    @Override
    public double calculate(TaxableOrder order) {
        String country = order.getShippingAddress().getCountry();
        double vatRate = VAT_RATES.getOrDefault(country, 0.20);
        
        return order.getSubtotal() * vatRate;
    }
    
    @Override
    public boolean appliesTo(Address address) {
        return VAT_RATES.containsKey(address.getCountry());
    }
    
    @Override
    public String getTaxRegion() {
        return "EU";
    }
}

// ✨ NEW: Canada GST/PST calculator (added later)
@Service
public class CanadaTaxCalculator implements TaxCalculator {
    
    @Override
    public double calculate(TaxableOrder order) {
        double gst = 0.05; // 5% GST (federal)
        String province = order.getShippingAddress().getState();
        
        // Provincial sales tax varies
        double pst = switch (province) {
            case "BC" -> 0.07;
            case "SK" -> 0.06;
            case "MB" -> 0.07;
            case "QC" -> 0.09975;
            default -> 0.0;
        };
        
        return order.getSubtotal() * (gst + pst);
    }
    
    @Override
    public boolean appliesTo(Address address) {
        return "CA".equals(address.getCountry());
    }
    
    @Override
    public String getTaxRegion() {
        return "CANADA";
    }
}

// Tax service
@Service
public class TaxService {
    
    private final List<TaxCalculator> calculators;
    
    @Autowired
    public TaxService(List<TaxCalculator> calculators) {
        this.calculators = calculators;
    }
    
    public double calculateTax(TaxableOrder order) {
        Address address = order.getShippingAddress();
        
        return calculators.stream()
            .filter(calc -> calc.appliesTo(address))
            .findFirst()
            .map(calc -> calc.calculate(order))
            .orElse(0.0);
    }
}
```

---

## 🎤 Interview Questions & Answers

### Q1: "What is the Open/Closed Principle?"

**Perfect Answer:**
```
OCP states that software entities should be OPEN for extension 
but CLOSED for modification.

This means:
✅ You can ADD new functionality (extend)
✅ WITHOUT changing existing code (closed)

Real Example from my project:

PROBLEM:
We had a discount calculator with 500 lines of if-else.
Every new promotion type required:
- Modifying the same method
- Testing EVERYTHING again
- Risk of breaking existing discounts

We had 8 production bugs in 6 months!

SOLUTION (OCP):
- Created DiscountStrategy interface
- Each promotion = separate class
- Added 12 new promotions in next 6 months
- Zero bugs in existing discounts
- Zero modifications to working code

The key insight:
"Add new code, don't change old code"
```

---

### Q2: "How do you identify OCP violations?"

**Perfect Answer:**
```
Red flags that scream OCP violation:

1️⃣ LONG IF-ELSE or SWITCH statements
   if (type.equals("A")) { ... }
   else if (type.equals("B")) { ... }
   else if (type.equals("C")) { ... }
   // Growing forever...

2️⃣ Modifying existing class for new features
   "I need to add email notification"
   → Opens NotificationService.java
   → Modifies existing method
   → Risk breaking SMS notifications

3️⃣ String/enum type checking
   if (paymentType == PaymentType.CREDIT_CARD)
   // What if we add BITCOIN?

4️⃣ Multiple teams modifying same file
   Constant merge conflicts

5️⃣ Fear of adding features
   "Let's not touch that, it might break"

Real example from code review:

❌ VIOLATION:
public void process(String type) {
    if ("EMAIL".equals(type)) { ... }
    else if ("SMS".equals(type)) { ... }
    // Team adds more if-else every sprint
}

✅ FIXED:
interface NotificationChannel { void send(); }

class EmailChannel implements NotificationChannel { ... }
class SMSChannel implements NotificationChannel { ... }
class SlackChannel implements NotificationChannel { ... }
// New channels = new classes, zero modification!
```

---

### Q3: "Give a real example where violating OCP caused problems"

**Perfect Answer:**
```
REAL PRODUCTION INCIDENT:

CONTEXT:
E-commerce platform with payment processing.
All payment methods in one 600-line class with giant switch.

THE REQUEST:
Marketing: "Add Apple Pay before Black Friday"

THE IMPLEMENTATION:
Developer added new case in switch statement:

switch (paymentMethod) {
    case "CREDIT_CARD": ...
    case "PAYPAL": ...
    case "APPLE_PAY": ...  // ← NEW
}

THE MISTAKE:
While adding Apple Pay code, developer accidentally:
- Removed a break; statement in PayPal case
- PayPal logic fell through to Apple Pay logic

DEPLOYED TO PRODUCTION:

IMPACT:
💥 All PayPal payments started FAILING
💥 Lost $250,000 in sales (2 hours before fix)
💥 3,500 customer complaints
💥 Emergency rollback at 2 AM
💥 Team worked 14 hours straight

ROOT CAUSE: OCP Violation
- Adding new feature required modifying existing code
- 600 lines in one method
- One typo broke everything
- No isolation between payment methods

THE FIX (OCP):
interface PaymentProcessor { ... }

class CreditCardProcessor implements PaymentProcessor { ... }
class PayPalProcessor implements PaymentProcessor { ... }
class ApplePayProcessor implements PaymentProcessor { ... }

Result after refactor:
✅ Added 6 new payment methods in next year
✅ Zero bugs in existing methods
✅ Each method independently testable
✅ Zero production incidents

LESSON: OCP isn't academic - it prevents REAL $$ loss!
```

---

### Q4: "What's the difference between OCP and just creating new classes?"

**Perfect Answer:**
```
Great question! OCP is about the DESIGN, not just adding classes.

❌ WRONG (Just adding classes):
class OrderProcessor {
    void process(Order order) {
        if (order.type == "ONLINE") {
            new OnlineOrderProcessor().process(order);
        } else if (order.type == "RETAIL") {
            new RetailOrderProcessor().process(order);
        } else if (order.type == "WHOLESALE") {
            new WholesaleOrderProcessor().process(order);
        }
        // Still need to MODIFY this if-else!
    }
}

✅ CORRECT (OCP Design):
interface OrderProcessor {
    void process(Order order);
}

class OrderService {
    private Map<String, OrderProcessor> processors;
    
    @Autowired  // Auto-discovers ALL implementations
    OrderService(List<OrderProcessor> processorList) {
        processors = processorList.stream()
            .collect(Collectors.toMap(
                OrderProcessor::getType, 
                Function.identity()
            ));
    }
    
    void process(Order order) {
        processors.get(order.type).process(order);
        // NO modification needed for new processor types!
    }
}

Key Difference:
❌ Wrong: Client code knows about all implementations
✅ Right: Client code works through abstraction

OCP means:
1. Design with abstraction (interface/abstract class)
2. Client depends on abstraction, not concrete classes
3. New features = new implementations
4. Zero changes to existing client code

Real Test:
"Can I add feature without modifying ANY existing class?"
- YES → Following OCP ✅
- NO → Violating OCP ❌
```

---

### Q5: "Doesn't OCP lead to too many classes?"

**Perfect Answer:**
```
Yes and no - it's about balance!

ANTI-PATTERN: Over-engineering
class AddTwoNumbersStrategy { }
class MultiplyTwoNumbersStrategy { }
class DivideTwoNumbersStrategy { }
// This is ridiculous over-engineering!

GOOD PATTERN: Meaningful extension points
interface PaymentProcessor { }
class StripeProcessor implements PaymentProcessor { }
class PayPalProcessor implements PaymentProcessor { }
// These are likely to vary independently

RULE OF THUMB:

Create abstraction when:
✅ You have 3+ if-else for same type check
✅ Different implementations likely to be added
✅ Different teams might extend functionality
✅ Runtime configuration needed
✅ Plugin architecture desired

DON'T create abstraction when:
❌ Only 2 simple variations
❌ Logic unlikely to change
❌ Over-complicates simple code
❌ No foreseeable new cases

Real Example:

BAD:
interface NameFormatter {
    String format(String name);
}
class UpperCaseNameFormatter { }
class LowerCaseNameFormatter { }
// Just use name.toUpperCase()!

GOOD:
interface DiscountStrategy {
    double calculate(Order order);
}
class PercentageDiscount { }
class FixedAmountDiscount { }
class BuyXGetYDiscount { }
class SeasonalDiscount { }
// Business adds new promotions monthly!

Ask yourself:
"Is this variation point likely to grow?"
- YES → Use OCP
- NO → Keep it simple

Remember:
OCP is about VARIATION POINTS, not every single decision.
```

---

### Q6: "How does OCP relate to other SOLID principles?"

**Perfect Answer:**
```
OCP works WITH other principles:

OCP + SRP:
- SRP: Each class has one responsibility
- OCP: Extend via new single-responsibility classes
- Together: Add features without modifying existing responsibilities

Example:
interface NotificationChannel { }  // SRP: Just send notifications
class EmailChannel { }              // SRP: Email only
class SMSChannel { }                // SRP: SMS only
// OCP: Add PushChannel without touching existing code

OCP + LSP:
- LSP: Subclass must work like parent
- OCP: Extension works because subclasses are substitutable

Example:
interface PaymentProcessor { PaymentResult process(...); }
// All implementations return PaymentResult (LSP)
// Can add new processor types safely (OCP)

OCP + ISP:
- ISP: Small, focused interfaces
- OCP: Easier to extend when interfaces are focused

Example:
interface Discountable { double getDiscount(); }
interface Trackable { String getTrackingNumber(); }
// Easy to mix and match behaviors

OCP + DIP:
- DIP: Depend on abstractions
- OCP: Extension through abstractions

Example:
class OrderService {
    private PaymentProcessor processor;  // DIP: depend on abstraction
    
    OrderService(PaymentProcessor processor) {
        this.processor = processor;
    }
    // OCP: Can inject any payment processor
}

They all work together:
SOLID = Synergistic principles, not isolated rules
```

---

## 🎯 Quick Reference Card

```java
/**
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 *      OPEN/CLOSED PRINCIPLE (OCP)
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 * 
 * 🎯 DEFINITION:
 * "Open for extension, Closed for modification"
 * 
 * 🔍 IDENTIFY VIOLATIONS:
 * ❌ Long if-else or switch statements
 * ❌ Modifying existing classes for new features
 * ❌ String/enum type checking everywhere
 * ❌ Fear of adding features (might break stuff)
 * ❌ Constant merge conflicts
 * 
 * ✅ SOLUTIONS:
 * ✓ Use interfaces/abstract classes
 * ✓ Strategy Pattern
 * ✓ Template Method Pattern
 * ✓ Factory Pattern
 * ✓ Dependency Injection
 * ✓ Polymorphism
 * 
 * 🔧 IMPLEMENTATION TECHNIQUES:
 * 1. Define abstraction (interface/abstract class)
 * 2. Create concrete implementations
 * 3. Client depends on abstraction
 * 4. Add new features = new implementations
 * 5. Zero changes to existing code
 * 
 * 🎁 BENEFITS:
 * ✓ Add features without breaking existing code
 * ✓ Reduced testing (only test new code)
 * ✓ Parallel development (no conflicts)
 * ✓ Runtime flexibility
 * ✓ Plugin architecture
 * ✓ Easier maintenance
 * 
 * ⚠️ WHEN TO APPLY:
 * ✓ 3+ variations of same concept
 * ✓ Frequent new feature requests
 * ✓ Multiple teams extending system
 * ✓ Plugin architecture needed
 * 
 * 🚫 WHEN NOT TO APPLY:
 * ✗ Only 2 simple cases
 * ✗ Logic never changes
 * ✗ Over-complicates simple code
 * 
 * 💡 THE TEST:
 * "Can I add new feature without modifying existing classes?"
 * • YES → OCP followed ✅
 * • NO → OCP violated ❌
 * 
 * 📝 REMEMBER:
 * "Add new code, don't change old code!"
 * 
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 */
```

---

## 🌟 Final Pro Tips

### 1️⃣ Start with Strategy Pattern
```java
// Most common OCP implementation
interface Strategy { }
class ConcreteStrategyA implements Strategy { }
class ConcreteStrategyB implements Strategy { }
```

### 2️⃣ Use Dependency Injection
```java
// Let framework manage implementations
@Autowired
List<DiscountStrategy> allDiscounts;  // Auto-discovers all!
```

### 3️⃣ Avoid Premature Abstraction
```java
// Don't create abstraction until you have 3+ cases
// YAGNI (You Aren't Gonna Need It) applies
```

### 4️⃣ Make Abstractions Meaningful
```java
✅ interface PaymentProcessor
✅ interface ShippingCalculator
✅ interface TaxCalculator
❌ interface IService
❌ interface BaseProcessor
```

### 5️⃣ Document Extension Points
```java
/**
 * To add new payment method:
 * 1. Implement PaymentProcessor interface
 * 2. Annotate with @Service
 * 3. Done! Auto-discovered by Spring
 */
public interface PaymentProcessor { }
```

---

**Remember:** 🌟

> "The best code is the code you DON'T have to change.
> Design for extension, not modification.
> Your production system will thank you!"

---

*Made with ❤️ for Extensible Systems*
*Zero Bugs Rate: 💯%*

🚀 **Happy Extending!**

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

# ✂️ Interface Segregation Principle - Real-Time E-Commerce Deep Dive
---

## 🎯 ISP Core Concept

### Definition
> **"Clients should not be forced to depend on interfaces they don't use"**

### What It Really Means 💭

```java
/**
 * ❌ FAT INTERFACE (Bad):
 * interface Worker {
 *     void work();
 *     void eat();
 *     void sleep();
 *     void getSalary();
 *     void takeVacation();
 * }
 * 
 * class Robot implements Worker {
 *     void work() { ... }
 *     void eat() { throw new UnsupportedOperationException(); } // 🔥 Forced!
 *     void sleep() { throw new UnsupportedOperationException(); } // 🔥 Forced!
 *     void getSalary() { throw new UnsupportedOperationException(); } // 🔥 Forced!
 *     void takeVacation() { throw new UnsupportedOperationException(); } // 🔥 Forced!
 * }
 * 
 * ✅ SEGREGATED INTERFACES (Good):
 * interface Workable { void work(); }
 * interface Eatable { void eat(); }
 * interface Sleepable { void sleep(); }
 * interface Payable { void getSalary(); }
 * 
 * class Human implements Workable, Eatable, Sleepable, Payable { ... }
 * class Robot implements Workable { ... } // ✨ Only what it needs!
 * 
 * KEY PRINCIPLE:
 * ✅ Many small, specific interfaces
 * ✅ Clients implement only what they need
 * ✅ No empty/throwing methods
 * ✅ High cohesion, low coupling
 * 
 * ❌ One large, general interface
 * ❌ Forced to implement unused methods
 * ❌ Polluted implementations
 * ❌ Tight coupling
 */
```

### The "Fat Interface" Smell Test 👃

**Ask yourself:** *"Does this class have empty methods or methods that throw UnsupportedOperationException?"*

- If **YES** → ❌ **ISP Violated** (Fat Interface)
- If **NO** → ✅ **ISP Followed** (Segregated Interfaces)

---

## 📦 Scenario 1: Product Management System

### ❌ BAD Example - Fat Interface Violation

```java
/**
 * 🚫 VIOLATION: One massive interface forcing all products to implement everything!
 * This is the classic ISP violation in e-commerce
 */

public interface Product {
    
    // Basic product info - ALL products need this ✅
    String getId();
    String getName();
    String getDescription();
    double getPrice();
    String getCategory();
    
    // Physical product methods - Only physical products need these! ⚠️
    double getWeight();
    double getLength();
    double getWidth();
    double getHeight();
    String getShippingClass();
    int getInventoryCount();
    void updateInventory(int quantity);
    
    // Digital product methods - Only digital products need these! ⚠️
    String getDownloadUrl();
    long getFileSizeInBytes();
    String getFileFormat();
    int getDownloadLimit();
    LocalDateTime getExpirationDate();
    
    // Service/Subscription methods - Only services need these! ⚠️
    int getDurationInDays();
    boolean isRecurring();
    double getRenewalPrice();
    LocalDateTime getNextBillingDate();
    
    // Customizable product methods - Only some products need these! ⚠️
    List<CustomizationOption> getCustomizationOptions();
    void addCustomization(String option, String value);
    boolean isCustomizable();
    
    // Rental product methods - Only rental products need these! ⚠️
    double getDailyRentalRate();
    double getSecurityDeposit();
    int getMaxRentalDays();
    
    // Perishable product methods - Only food items need these! ⚠️
    LocalDate getExpiryDate();
    String getStorageInstructions();
    boolean requiresRefrigeration();
}

/**
 * 🔥 PROBLEM: Physical Book is forced to implement irrelevant methods
 */
public class PhysicalBook implements Product {
    
    private String id;
    private String name;
    private double price;
    private double weight;
    private int inventoryCount;
    
    // ✅ Implements what it needs
    @Override
    public String getId() { return id; }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public double getPrice() { return price; }
    
    @Override
    public double getWeight() { return weight; }
    
    @Override
    public int getInventoryCount() { return inventoryCount; }
    
    @Override
    public void updateInventory(int quantity) {
        this.inventoryCount += quantity;
    }
    
    // 🔥 FORCED to implement digital product methods (DOESN'T MAKE SENSE!)
    @Override
    public String getDownloadUrl() {
        throw new UnsupportedOperationException("Physical books can't be downloaded!");
    }
    
    @Override
    public long getFileSizeInBytes() {
        throw new UnsupportedOperationException("Physical books don't have file size!");
    }
    
    @Override
    public String getFileFormat() {
        throw new UnsupportedOperationException("Physical books don't have file format!");
    }
    
    @Override
    public int getDownloadLimit() {
        throw new UnsupportedOperationException("Physical books can't be downloaded!");
    }
    
    @Override
    public LocalDateTime getExpirationDate() {
        throw new UnsupportedOperationException("Physical books don't expire!");
    }
    
    // 🔥 FORCED to implement service/subscription methods
    @Override
    public int getDurationInDays() {
        throw new UnsupportedOperationException("Books are not services!");
    }
    
    @Override
    public boolean isRecurring() {
        return false; // Meaningless for books
    }
    
    @Override
    public double getRenewalPrice() {
        throw new UnsupportedOperationException("Books don't renew!");
    }
    
    @Override
    public LocalDateTime getNextBillingDate() {
        throw new UnsupportedOperationException("Books don't have billing dates!");
    }
    
    // 🔥 FORCED to implement customization methods
    @Override
    public List<CustomizationOption> getCustomizationOptions() {
        return Collections.emptyList(); // Empty - wasteful!
    }
    
    @Override
    public void addCustomization(String option, String value) {
        throw new UnsupportedOperationException("Books can't be customized!");
    }
    
    @Override
    public boolean isCustomizable() {
        return false;
    }
    
    // 🔥 FORCED to implement rental methods
    @Override
    public double getDailyRentalRate() {
        throw new UnsupportedOperationException("Books are not for rent!");
    }
    
    @Override
    public double getSecurityDeposit() {
        throw new UnsupportedOperationException("No security deposit for books!");
    }
    
    @Override
    public int getMaxRentalDays() {
        throw new UnsupportedOperationException("Books are not rentals!");
    }
    
    // 🔥 FORCED to implement perishable methods
    @Override
    public LocalDate getExpiryDate() {
        throw new UnsupportedOperationException("Books don't expire!");
    }
    
    @Override
    public String getStorageInstructions() {
        throw new UnsupportedOperationException("Books don't need special storage!");
    }
    
    @Override
    public boolean requiresRefrigeration() {
        return false; // Meaningless
    }
}

/**
 * 🔥 PROBLEM: Digital Course has the same issues
 */
public class DigitalCourse implements Product {
    
    private String id;
    private String name;
    private double price;
    private String downloadUrl;
    private long fileSizeInBytes;
    
    // ✅ Implements what it needs
    @Override
    public String getId() { return id; }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public double getPrice() { return price; }
    
    @Override
    public String getDownloadUrl() { return downloadUrl; }
    
    @Override
    public long getFileSizeInBytes() { return fileSizeInBytes; }
    
    // 🔥 FORCED to implement physical product methods (DOESN'T MAKE SENSE!)
    @Override
    public double getWeight() {
        return 0; // Digital products have no weight - meaningless!
    }
    
    @Override
    public double getLength() {
        throw new UnsupportedOperationException("Digital products have no dimensions!");
    }
    
    @Override
    public double getWidth() {
        throw new UnsupportedOperationException("Digital products have no dimensions!");
    }
    
    @Override
    public double getHeight() {
        throw new UnsupportedOperationException("Digital products have no dimensions!");
    }
    
    @Override
    public String getShippingClass() {
        throw new UnsupportedOperationException("Digital products don't ship!");
    }
    
    @Override
    public int getInventoryCount() {
        return Integer.MAX_VALUE; // Infinite digital copies - meaningless!
    }
    
    @Override
    public void updateInventory(int quantity) {
        // Does nothing - digital products don't have inventory
    }
    
    // 🔥 More forced implementations...
    @Override
    public boolean isRecurring() { return false; }
    
    @Override
    public double getDailyRentalRate() {
        throw new UnsupportedOperationException("Courses are not rentals!");
    }
    
    // ... 20+ more meaningless methods!
}

/**
 * 🔥 CLIENT CODE PROBLEMS
 */
@Service
public class ProductService {
    
    /**
     * 🔥 PROBLEM: Can't trust the interface!
     * Need to check types and handle exceptions
     */
    public void displayProductDetails(Product product) {
        System.out.println("Name: " + product.getName());
        System.out.println("Price: $" + product.getPrice());
        
        // 🔥 Need to check what kind of product it is!
        if (product instanceof PhysicalBook) {
            try {
                System.out.println("Weight: " + product.getWeight() + " kg");
                System.out.println("Stock: " + product.getInventoryCount());
            } catch (UnsupportedOperationException e) {
                // Ignore - shouldn't happen for physical products
            }
        }
        
        if (product instanceof DigitalCourse) {
            try {
                System.out.println("Download: " + product.getDownloadUrl());
                System.out.println("Size: " + product.getFileSizeInBytes() + " bytes");
            } catch (UnsupportedOperationException e) {
                // Ignore - shouldn't happen for digital products
            }
        }
        
        // 🔥 This defeats the purpose of polymorphism!
    }
    
    /**
     * 🔥 PROBLEM: Methods that should work on all products can fail
     */
    public void calculateShippingCost(Product product) {
        try {
            double weight = product.getWeight(); // 💥 Might throw exception!
            double cost = weight * 5.0;
            System.out.println("Shipping cost: $" + cost);
        } catch (UnsupportedOperationException e) {
            System.out.println("This product cannot be shipped");
        }
    }
}
```

### 🔥 Problems with Fat Interface

| Problem | Impact | Why It's Bad |
|---------|--------|--------------|
| **20+ forced implementations** | Code pollution | 😱 Cluttered with exceptions |
| **Meaningless methods** | Confusion | 🤔 What's real, what's fake? |
| **Runtime exceptions** | Crashes | 💥 UnsupportedOperationException everywhere |
| **Can't trust interface** | Type checking needed | 🔍 instanceof all over |
| **Hard to test** | Low coverage | 🧪 Testing exceptions instead of logic |
| **Violates LSP** | Broken polymorphism | 🔄 Can't substitute freely |
| **Maintenance nightmare** | High cost | 💸 Changes affect everyone |

---

### ✅ GOOD Example - Segregated Interfaces

```java
/**
 * ✨ SOLUTION: Split fat interface into multiple role-specific interfaces
 * Each product type implements ONLY what it needs!
 */

// ========================================
// 1️⃣ BASE INTERFACE - All products need this
// ========================================

/**
 * ✨ Core product information - EVERY product has these
 */
public interface Product {
    String getId();
    String getName();
    String getDescription();
    double getPrice();
    String getCategory();
    ProductType getType();
}

public enum ProductType {
    PHYSICAL, DIGITAL, SERVICE, RENTAL, PERISHABLE
}

// ========================================
// 2️⃣ ROLE INTERFACES - Specific capabilities
// ========================================

/**
 * ✨ For products that have physical dimensions and weight
 * Only implemented by physical products
 */
public interface PhysicalProduct extends Product {
    double getWeight();
    Dimensions getDimensions();
    String getShippingClass();
}

@Data
@AllArgsConstructor
public class Dimensions {
    private double length;
    private double width;
    private double height;
    
    public double getVolume() {
        return length * width * height;
    }
}

/**
 * ✨ For products that have inventory/stock
 * Physical products need this, digital don't
 */
public interface Stockable {
    int getInventoryCount();
    void updateInventory(int quantity);
    boolean isInStock();
    int getReorderLevel();
}

/**
 * ✨ For products that can be downloaded
 * Only digital products need this
 */
public interface Downloadable extends Product {
    String getDownloadUrl();
    long getFileSizeInBytes();
    String getFileFormat();
    int getDownloadLimit();
}

/**
 * ✨ For products that expire
 * Digital downloads, subscriptions, food items
 */
public interface Expirable {
    LocalDateTime getExpirationDate();
    boolean isExpired();
    int getDaysUntilExpiration();
}

/**
 * ✨ For subscription-based products
 * Services, memberships, subscriptions
 */
public interface Subscribable extends Product {
    int getDurationInDays();
    boolean isRecurring();
    double getRenewalPrice();
    LocalDateTime getNextBillingDate();
    SubscriptionStatus getStatus();
}

public enum SubscriptionStatus {
    ACTIVE, PAUSED, CANCELLED, EXPIRED
}

/**
 * ✨ For products that can be customized
 * T-shirts, mugs, jewelry, etc.
 */
public interface Customizable {
    List<CustomizationOption> getCustomizationOptions();
    void addCustomization(String optionId, String value);
    boolean isCustomizable();
    double getCustomizationUpcharge();
}

@Data
@AllArgsConstructor
public class CustomizationOption {
    private String id;
    private String name;
    private List<String> availableValues;
    private double additionalCost;
}

/**
 * ✨ For products that can be rented
 * Equipment, vehicles, tools
 */
public interface Rentable extends Product {
    double getDailyRentalRate();
    double getSecurityDeposit();
    int getMaxRentalDays();
    boolean isAvailableForRent(LocalDate startDate, LocalDate endDate);
}

/**
 * ✨ For perishable items
 * Food, flowers, medicine
 */
public interface Perishable {
    LocalDate getExpiryDate();
    String getStorageInstructions();
    boolean requiresRefrigeration();
    int getShelfLifeDays();
}

// ========================================
// 3️⃣ CONCRETE IMPLEMENTATIONS - Clean and focused!
// ========================================

/**
 * ✨ Physical Book - Only implements what it needs!
 */
@Data
public class PhysicalBook implements PhysicalProduct, Stockable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    private String isbn;
    private String author;
    
    // Physical attributes
    private double weight;
    private Dimensions dimensions;
    private String shippingClass;
    
    // Stock management
    private int inventoryCount;
    private int reorderLevel;
    
    @Override
    public ProductType getType() {
        return ProductType.PHYSICAL;
    }
    
    @Override
    public boolean isInStock() {
        return inventoryCount > 0;
    }
    
    @Override
    public void updateInventory(int quantity) {
        this.inventoryCount += quantity;
        
        if (inventoryCount < reorderLevel) {
            System.out.println("⚠️ Low stock alert for: " + name);
        }
    }
    
    // ✨ NO empty methods!
    // ✨ NO exceptions!
    // ✨ Everything makes sense for a physical book!
}

/**
 * ✨ eBook - Different interface combination!
 */
@Data
public class EBook implements Downloadable, Expirable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    private String isbn;
    private String author;
    
    // Digital attributes
    private String downloadUrl;
    private long fileSizeInBytes;
    private String fileFormat; // PDF, EPUB, MOBI
    private int downloadLimit;
    
    // Expiration (rental or time-limited access)
    private LocalDateTime expirationDate;
    
    @Override
    public ProductType getType() {
        return ProductType.DIGITAL;
    }
    
    @Override
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expirationDate);
    }
    
    @Override
    public int getDaysUntilExpiration() {
        return (int) ChronoUnit.DAYS.between(LocalDateTime.now(), expirationDate);
    }
    
    // ✨ NO inventory methods (doesn't need them)!
    // ✨ NO physical dimension methods!
    // ✨ Only implements what makes sense!
}

/**
 * ✨ Online Course - Combination of digital + subscription
 */
@Data
public class OnlineCourse implements Downloadable, Subscribable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    private String instructor;
    
    // Digital content
    private String downloadUrl;
    private long fileSizeInBytes;
    private String fileFormat;
    private int downloadLimit;
    
    // Subscription details
    private int durationInDays;
    private boolean recurring;
    private double renewalPrice;
    private LocalDateTime nextBillingDate;
    private SubscriptionStatus status;
    
    @Override
    public ProductType getType() {
        return ProductType.SERVICE;
    }
    
    // ✨ Implements BOTH Downloadable AND Subscribable perfectly!
    // ✨ NO forced methods!
}

/**
 * ✨ Custom T-Shirt - Physical + Stockable + Customizable
 */
@Data
public class CustomTShirt implements PhysicalProduct, Stockable, Customizable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    
    // Physical attributes
    private double weight;
    private Dimensions dimensions;
    private String shippingClass;
    
    // Stock
    private int inventoryCount;
    private int reorderLevel;
    
    // Customization
    private List<CustomizationOption> customizationOptions;
    private Map<String, String> appliedCustomizations;
    
    public CustomTShirt() {
        this.customizationOptions = Arrays.asList(
            new CustomizationOption("size", "Size", 
                Arrays.asList("S", "M", "L", "XL", "XXL"), 0),
            new CustomizationOption("color", "Color",
                Arrays.asList("Red", "Blue", "Green", "Black", "White"), 0),
            new CustomizationOption("text", "Custom Text",
                Collections.emptyList(), 5.0)
        );
        this.appliedCustomizations = new HashMap<>();
    }
    
    @Override
    public ProductType getType() {
        return ProductType.PHYSICAL;
    }
    
    @Override
    public boolean isInStock() {
        return inventoryCount > 0;
    }
    
    @Override
    public void updateInventory(int quantity) {
        this.inventoryCount += quantity;
    }
    
    @Override
    public void addCustomization(String optionId, String value) {
        appliedCustomizations.put(optionId, value);
    }
    
    @Override
    public boolean isCustomizable() {
        return true;
    }
    
    @Override
    public double getCustomizationUpcharge() {
        return customizationOptions.stream()
            .filter(opt -> appliedCustomizations.containsKey(opt.getId()))
            .mapToDouble(CustomizationOption::getAdditionalCost)
            .sum();
    }
    
    // ✨ Implements THREE interfaces naturally!
    // ✨ All methods make perfect sense!
}

/**
 * ✨ Camera Rental - Physical + Rentable
 */
@Data
public class CameraRental implements PhysicalProduct, Rentable {
    
    private String id;
    private String name;
    private String description;
    private double price; // Purchase price
    private String category;
    
    // Physical attributes
    private double weight;
    private Dimensions dimensions;
    private String shippingClass;
    
    // Rental details
    private double dailyRentalRate;
    private double securityDeposit;
    private int maxRentalDays;
    private List<RentalPeriod> rentedPeriods;
    
    public CameraRental() {
        this.rentedPeriods = new ArrayList<>();
    }
    
    @Override
    public ProductType getType() {
        return ProductType.RENTAL;
    }
    
    @Override
    public boolean isAvailableForRent(LocalDate startDate, LocalDate endDate) {
        return rentedPeriods.stream()
            .noneMatch(period -> period.overlaps(startDate, endDate));
    }
    
    // ✨ Perfect for rental equipment!
}

@Data
@AllArgsConstructor
class RentalPeriod {
    private LocalDate startDate;
    private LocalDate endDate;
    
    public boolean overlaps(LocalDate start, LocalDate end) {
        return !endDate.isBefore(start) && !startDate.isAfter(end);
    }
}

/**
 * ✨ Fresh Produce - Physical + Stockable + Perishable
 */
@Data
public class FreshProduce implements PhysicalProduct, Stockable, Perishable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    
    // Physical attributes
    private double weight;
    private Dimensions dimensions;
    private String shippingClass;
    
    // Stock
    private int inventoryCount;
    private int reorderLevel;
    
    // Perishable attributes
    private LocalDate expiryDate;
    private String storageInstructions;
    private boolean requiresRefrigeration;
    private int shelfLifeDays;
    
    @Override
    public ProductType getType() {
        return ProductType.PERISHABLE;
    }
    
    @Override
    public boolean isInStock() {
        // Not in stock if expired OR quantity is zero
        return inventoryCount > 0 && !isExpired();
    }
    
    @Override
    public void updateInventory(int quantity) {
        this.inventoryCount += quantity;
    }
    
    private boolean isExpired() {
        return LocalDate.now().isAfter(expiryDate);
    }
    
    // ✨ Perfect combination for perishable goods!
}

/**
 * ✨ Netflix-like Subscription - Pure subscription
 */
@Data
public class StreamingSubscription implements Subscribable {
    
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    
    // Subscription details
    private int durationInDays;
    private boolean recurring;
    private double renewalPrice;
    private LocalDateTime nextBillingDate;
    private SubscriptionStatus status;
    private String planType; // Basic, Standard, Premium
    
    @Override
    public ProductType getType() {
        return ProductType.SERVICE;
    }
    
    // ✨ NO download methods!
    // ✨ NO physical attributes!
    // ✨ NO inventory!
    // ✨ Only subscription logic - PERFECT!
}

// ========================================
// 4️⃣ SERVICE LAYER - Type-safe and clean!
// ========================================

/**
 * ✨ Product Service - No more type checking nightmares!
 */
@Service
public class ProductService {
    
    /**
     * ✨ Works with ANY product - uses only base interface
     */
    public void displayBasicInfo(Product product) {
        System.out.println("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━");
        System.out.println("📦 Product: " + product.getName());
        System.out.println("💰 Price: $" + product.getPrice());
        System.out.println("🏷️ Category: " + product.getCategory());
        System.out.println("📋 Type: " + product.getType());
        System.out.println("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━");
    }
    
    /**
     * ✨ Works ONLY with physical products - type safe!
     */
    public double calculateShippingCost(PhysicalProduct product) {
        double weight = product.getWeight();
        Dimensions dims = product.getDimensions();
        double volume = dims.getVolume();
        
        double baseCost = 5.0;
        double weightCost = weight * 2.0;
        double volumeCost = volume * 0.1;
        
        return baseCost + weightCost + volumeCost;
    }
    
    /**
     * ✨ Works ONLY with stockable products - type safe!
     */
    public void checkAndReorderStock(Stockable product) {
        if (!product.isInStock()) {
            System.out.println("⚠️ Out of stock! Reordering...");
            // Trigger reorder logic
        } else if (product.getInventoryCount() < product.getReorderLevel()) {
            System.out.println("⚠️ Low stock warning!");
        }
    }
    
    /**
     * ✨ Works ONLY with downloadable products - type safe!
     */
    public DownloadInfo getDownloadInfo(Downloadable product) {
        return DownloadInfo.builder()
            .url(product.getDownloadUrl())
            .fileSize(product.getFileSizeInBytes())
            .format(product.getFileFormat())
            .remainingDownloads(product.getDownloadLimit())
            .build();
    }
    
    /**
     * ✨ Works ONLY with expirable products - type safe!
     */
    public void checkExpiration(Expirable product) {
        if (product.isExpired()) {
            System.out.println("❌ Product has expired!");
        } else {
            int daysLeft = product.getDaysUntilExpiration();
            if (daysLeft <= 7) {
                System.out.println("⚠️ Expiring soon: " + daysLeft + " days left");
            }
        }
    }
    
    /**
     * ✨ Works ONLY with subscribable products - type safe!
     */
    public void processSubscriptionRenewal(Subscribable product) {
        if (product.isRecurring() && 
            LocalDateTime.now().isAfter(product.getNextBillingDate())) {
            
            System.out.println("💳 Processing renewal: $" + product.getRenewalPrice());
            // Process payment
        }
    }
    
    /**
     * ✨ Works ONLY with customizable products - type safe!
     */
    public double calculateCustomizationCost(Customizable product) {
        return product.getCustomizationUpcharge();
    }
    
    /**
     * ✨ Works ONLY with rentable products - type safe!
     */
    public double calculateRentalCost(Rentable product, int days) {
        if (days > product.getMaxRentalDays()) {
            throw new IllegalArgumentException(
                "Rental period exceeds maximum: " + product.getMaxRentalDays() + " days"
            );
        }
        
        return (product.getDailyRentalRate() * days) + product.getSecurityDeposit();
    }
}

// ========================================
// 5️⃣ SHOPPING CART - Handles all product types elegantly!
// ========================================

@Service
public class ShoppingCartService {
    
    @Autowired
    private ProductService productService;
    
    /**
     * ✨ Calculate total - works with ANY product type!
     */
    public double calculateCartTotal(List<Product> products) {
        double subtotal = products.stream()
            .mapToDouble(Product::getPrice)
            .sum();
        
        // Add customization costs
        double customizationCost = products.stream()
            .filter(p -> p instanceof Customizable)
            .mapToDouble(p -> ((Customizable) p).getCustomizationUpcharge())
            .sum();
        
        // Calculate shipping only for physical products
        double shippingCost = products.stream()
            .filter(p -> p instanceof PhysicalProduct)
            .mapToDouble(p -> productService.calculateShippingCost((PhysicalProduct) p))
            .sum();
        
        return subtotal + customizationCost + shippingCost;
    }
    
    /**
     * ✨ Check availability - handles different product types correctly
     */
    public boolean areAllItemsAvailable(List<Product> products) {
        
        // Check stock for physical products
        boolean stockAvailable = products.stream()
            .filter(p -> p instanceof Stockable)
            .allMatch(p -> ((Stockable) p).isInStock());
        
        // Check expiration for expirable products
        boolean notExpired = products.stream()
            .filter(p -> p instanceof Expirable)
            .noneMatch(p -> ((Expirable) p).isExpired());
        
        return stockAvailable && notExpired;
    }
    
    /**
     * ✨ Generate order summary with product-specific details
     */
    public OrderSummary generateOrderSummary(List<Product> products) {
        OrderSummary summary = new OrderSummary();
        
        for (Product product : products) {
            ProductSummary prodSummary = new ProductSummary();
            prodSummary.setName(product.getName());
            prodSummary.setPrice(product.getPrice());
            
            // Add type-specific information
            if (product instanceof PhysicalProduct) {
                PhysicalProduct pp = (PhysicalProduct) product;
                prodSummary.setShipping(productService.calculateShippingCost(pp));
                prodSummary.setWeight(pp.getWeight());
            }
            
            if (product instanceof Downloadable) {
                prodSummary.setDownloadable(true);
            }
            
            if (product instanceof Customizable) {
                Customizable cp = (Customizable) product;
                prodSummary.setCustomizationCost(cp.getCustomizationUpcharge());
            }
            
            if (product instanceof Expirable) {
                Expirable ep = (Expirable) product;
                prodSummary.setExpiresIn(ep.getDaysUntilExpiration());
            }
            
            summary.addProduct(prodSummary);
        }
        
        return summary;
    }
}

@Data
class OrderSummary {
    private List<ProductSummary> products = new ArrayList<>();
    private double total;
    
    public void addProduct(ProductSummary product) {
        products.add(product);
        total += product.getPrice() + product.getShipping() + product.getCustomizationCost();
    }
}

@Data
class ProductSummary {
    private String name;
    private double price;
    private double shipping;
    private double customizationCost;
    private double weight;
    private boolean downloadable;
    private int expiresIn;
}

@Data
@Builder
class DownloadInfo {
    private String url;
    private long fileSize;
    private String format;
    private int remainingDownloads;
}
```

---

### 📊 Comparison: Before vs After

| Aspect | ❌ Fat Interface | ✅ Segregated Interfaces |
|--------|-----------------|-------------------------|
| **Interface methods** | 30+ methods | 3-7 methods per interface |
| **Empty implementations** | 20+ per class | 0 (zero!) |
| **Exception throwing** | Everywhere | None |
| **Type checking needed** | Constant instanceof | Minimal, type-safe |
| **LSP compliance** | Violated | Perfect |
| **Code clarity** | Confusing | Crystal clear |
| **Maintainability** | Nightmare | Easy |
| **Testing** | Hard | Simple |

---

### 🎯 Real Impact Story

```java
/**
 * 📈 BEFORE ISP (Fat Interface):
 * 
 * - Product interface: 32 methods
 * - PhysicalBook: 18 empty/throwing methods
 * - EBook: 15 empty/throwing methods
 * - 47 production bugs in 6 months
 * - 12 UnsupportedOperationException crashes
 * - Team morale: 😢 "I hate this codebase"
 * - Time to add new product type: 3 days
 * 
 * 📈 AFTER ISP (Segregated):
 * 
 * - Base Product interface: 6 methods
 * - 7 role interfaces: 3-7 methods each
 * - Zero empty/throwing methods
 * - 3 bugs in 6 months (95% reduction!)
 * - Zero runtime exceptions
 * - Team morale: 😄 "Clean and elegant!"
 * - Time to add new product type: 2 hours
 * 
 * 💰 BUSINESS IMPACT:
 * - 95% reduction in product-related bugs
 * - 90% faster feature development
 * - $50,000 saved in development time
 * - Zero customer-facing crashes
 */
```

---

## 👥 Scenario 2: User Role Management

### ❌ BAD Example - Fat User Interface

```java
/**
 * 🚫 VIOLATION: One User interface for all user types
 */
public interface User {
    
    // Basic user info - Everyone needs ✅
    String getUserId();
    String getEmail();
    String getFullName();
    
    // Customer-specific - Only customers need! ⚠️
    List<Order> getOrderHistory();
    ShoppingCart getCart();
    List<Address> getSavedAddresses();
    PaymentMethod getDefaultPaymentMethod();
    int getLoyaltyPoints();
    WishList getWishList();
    
    // Vendor-specific - Only vendors need! ⚠️
    String getStoreName();
    List<Product> getProductCatalog();
    double getCommissionRate();
    BankAccount getPayoutAccount();
    List<VendorOrder> getPendingOrders();
    
    // Admin-specific - Only admins need! ⚠️
    List<Permission> getPermissions();
    boolean canAccessAdminPanel();
    boolean canManageUsers();
    boolean canManageProducts();
    boolean canViewReports();
    void assignRole(String userId, Role role);
    
    // Guest-specific - Only guests need! ⚠️
    String getSessionId();
    LocalDateTime getSessionExpiry();
    void convertToRegisteredUser(String password);
}

/**
 * 🔥 PROBLEM: Customer forced to implement vendor/admin methods
 */
public class Customer implements User {
    
    private String userId;
    private String email;
    private String fullName;
    private List<Order> orderHistory;
    private ShoppingCart cart;
    
    // ✅ Customer methods implemented properly
    @Override
    public List<Order> getOrderHistory() {
        return orderHistory;
    }
    
    @Override
    public ShoppingCart getCart() {
        return cart;
    }
    
    // 🔥 Forced to implement vendor methods!
    @Override
    public String getStoreName() {
        throw new UnsupportedOperationException("Customers don't have stores!");
    }
    
    @Override
    public List<Product> getProductCatalog() {
        throw new UnsupportedOperationException("Customers don't have product catalogs!");
    }
    
    @Override
    public double getCommissionRate() {
        throw new UnsupportedOperationException("Customers don't have commission rates!");
    }
    
    // 🔥 Forced to implement admin methods!
    @Override
    public boolean canAccessAdminPanel() {
        return false; // Meaningless
    }
    
    @Override
    public void assignRole(String userId, Role role) {
        throw new UnsupportedOperationException("Customers can't assign roles!");
    }
    
    // ... 10+ more forced methods
}
```

---

### ✅ GOOD Example - Segregated User Interfaces

```java
/**
 * ✨ SOLUTION: Role-based interface segregation
 */

// ========================================
// 1️⃣ BASE USER - Everyone has this
// ========================================

public interface User {
    String getUserId();
    String getEmail();
    String getFullName();
    String getPhoneNumber();
    UserType getUserType();
    LocalDateTime getCreatedAt();
    boolean isActive();
}

public enum UserType {
    CUSTOMER, VENDOR, ADMIN, GUEST, SUPPORT_AGENT
}

// ========================================
// 2️⃣ ROLE-SPECIFIC INTERFACES
// ========================================

/**
 * ✨ Shopping capabilities - Customers and Guests
 */
public interface Shopper extends User {
    ShoppingCart getCart();
    void addToCart(Product product, int quantity);
    void removeFromCart(String productId);
    double getCartTotal();
}

/**
 * ✨ Order management - Customers who can place orders
 */
public interface OrderPlacer extends Shopper {
    List<Order> getOrderHistory();
    Order placeOrder(OrderRequest request);
    void cancelOrder(String orderId);
    Order trackOrder(String orderId);
}

/**
 * ✨ Customer loyalty features
 */
public interface LoyaltyMember {
    int getLoyaltyPoints();
    void addLoyaltyPoints(int points);
    void redeemPoints(int points);
    LoyaltyTier getTier();
    double getDiscountPercentage();
}

public enum LoyaltyTier {
    BRONZE(0, 0.05),
    SILVER(100, 0.10),
    GOLD(500, 0.15),
    PLATINUM(1000, 0.20);
    
    private final int minPoints;
    private final double discountPercentage;
    
    LoyaltyTier(int minPoints, double discountPercentage) {
        this.minPoints = minPoints;
        this.discountPercentage = discountPercentage;
    }
    
    public double getDiscountPercentage() { return discountPercentage; }
}

/**
 * ✨ Profile management - Registered users
 */
public interface ProfileOwner extends User {
    List<Address> getSavedAddresses();
    void addAddress(Address address);
    void removeAddress(String addressId);
    List<PaymentMethod> getSavedPaymentMethods();
    void addPaymentMethod(PaymentMethod method);
    PaymentMethod getDefaultPaymentMethod();
}

/**
 * ✨ Wishlist capabilities
 */
public interface WishlistOwner {
    WishList getWishList();
    void addToWishList(Product product);
    void removeFromWishList(String productId);
    void moveWishListToCart(String productId);
}

/**
 * ✨ Vendor capabilities - Sellers
 */
public interface Seller extends User {
    String getStoreName();
    String getStoreDescription();
    List<Product> getProductCatalog();
    void addProduct(Product product);
    void updateProduct(String productId, Product product);
    void removeProduct(String productId);
    List<VendorOrder> getPendingOrders();
    void fulfillOrder(String orderId);
}

/**
 * ✨ Financial tracking - Vendors
 */
public interface Earner {
    double getCommissionRate();
    BankAccount getPayoutAccount();
    List<Transaction> getTransactionHistory();
    double getTotalEarnings();
    double getPendingPayouts();
}

/**
 * ✨ Admin capabilities
 */
public interface Administrator extends User {
    Set<Permission> getPermissions();
    boolean hasPermission(Permission permission);
    void assignRole(String userId, Role role);
    void revokeRole(String userId, Role role);
}

public enum Permission {
    MANAGE_USERS,
    MANAGE_PRODUCTS,
    MANAGE_ORDERS,
    VIEW_REPORTS,
    MANAGE_SETTINGS,
    MODERATE_CONTENT,
    MANAGE_PROMOTIONS
}

/**
 * ✨ Content moderation - Admins and Moderators
 */
public interface ContentModerator {
    List<Review> getPendingReviews();
    void approveReview(String reviewId);
    void rejectReview(String reviewId, String reason);
    List<ReportedContent> getReportedContent();
    void takeAction(String contentId, ModerationAction action);
}

public enum ModerationAction {
    APPROVE, REJECT, DELETE, FLAG, WARNING
}

/**
 * ✨ Guest session management
 */
public interface GuestSession {
    String getSessionId();
    LocalDateTime getSessionExpiry();
    boolean isExpired();
    void extendSession(int minutes);
    RegisteredCustomer convertToRegistered(RegistrationData data);
}

// ========================================
// 3️⃣ CONCRETE IMPLEMENTATIONS - Clean!
// ========================================

/**
 * ✨ Regular Customer - Implements only what needed!
 */
@Data
public class RegisteredCustomer implements User, OrderPlacer, LoyaltyMember, 
                                           ProfileOwner, WishlistOwner {
    
    // Base user info
    private String userId;
    private String email;
    private String fullName;
    private String phoneNumber;
    private LocalDateTime createdAt;
    private boolean active;
    
    // Shopping
    private ShoppingCart cart;
    private List<Order> orderHistory;
    
    // Loyalty
    private int loyaltyPoints;
    
    // Profile
    private List<Address> savedAddresses;
    private List<PaymentMethod> savedPaymentMethods;
    private String defaultPaymentMethodId;
    
    // Wishlist
    private WishList wishList;
    
    @Override
    public UserType getUserType() {
        return UserType.CUSTOMER;
    }
    
    @Override
    public void addToCart(Product product, int quantity) {
        cart.addItem(product, quantity);
    }
    
    @Override
    public void removeFromCart(String productId) {
        cart.removeItem(productId);
    }
    
    @Override
    public double getCartTotal() {
        return cart.calculateTotal();
    }
    
    @Override
    public Order placeOrder(OrderRequest request) {
        // Order placement logic
        Order order = new Order(userId, cart, request);
        orderHistory.add(order);
        
        // Award loyalty points
        int points = (int) (order.getTotal() / 10); // 1 point per $10
        addLoyaltyPoints(points);
        
        // Clear cart
        cart.clear();
        
        return order;
    }
    
    @Override
    public void cancelOrder(String orderId) {
        orderHistory.stream()
            .filter(o -> o.getId().equals(orderId))
            .findFirst()
            .ifPresent(Order::cancel);
    }
    
    @Override
    public Order trackOrder(String orderId) {
        return orderHistory.stream()
            .filter(o -> o.getId().equals(orderId))
            .findFirst()
            .orElseThrow(() -> new OrderNotFoundException(orderId));
    }
    
    @Override
    public void addLoyaltyPoints(int points) {
        this.loyaltyPoints += points;
    }
    
    @Override
    public void redeemPoints(int points) {
        if (points > loyaltyPoints) {
            throw new InsufficientPointsException();
        }
        this.loyaltyPoints -= points;
    }
    
    @Override
    public LoyaltyTier getTier() {
        return Arrays.stream(LoyaltyTier.values())
            .filter(tier -> loyaltyPoints >= tier.minPoints)
            .reduce((first, second) -> second) // Get highest tier
            .orElse(LoyaltyTier.BRONZE);
    }
    
    @Override
    public double getDiscountPercentage() {
        return getTier().getDiscountPercentage();
    }
    
    @Override
    public void addAddress(Address address) {
        savedAddresses.add(address);
    }
    
    @Override
    public void removeAddress(String addressId) {
        savedAddresses.removeIf(a -> a.getId().equals(addressId));
    }
    
    @Override
    public void addPaymentMethod(PaymentMethod method) {
        savedPaymentMethods.add(method);
    }
    
    @Override
    public PaymentMethod getDefaultPaymentMethod() {
        return savedPaymentMethods.stream()
            .filter(pm -> pm.getId().equals(defaultPaymentMethodId))
            .findFirst()
            .orElse(null);
    }
    
    @Override
    public void addToWishList(Product product) {
        wishList.addItem(product);
    }
    
    @Override
    public void removeFromWishList(String productId) {
        wishList.removeItem(productId);
    }
    
    @Override
    public void moveWishListToCart(String productId) {
        Product product = wishList.getItem(productId);
        wishList.removeItem(productId);
        cart.addItem(product, 1);
    }
    
    // ✨ NO vendor methods!
    // ✨ NO admin methods!
    // ✨ NO guest session methods!
    // ✨ Everything makes sense for a customer!
}

/**
 * ✨ Vendor Account - Different capabilities!
 */
@Data
public class VendorAccount implements User, Seller, Earner, ProfileOwner {
    
    // Base user info
    private String userId;
    private String email;
    private String fullName;
    private String phoneNumber;
    private LocalDateTime createdAt;
    private boolean active;
    
    // Vendor info
    private String storeName;
    private String storeDescription;
    private List<Product> productCatalog;
    private List<VendorOrder> pendingOrders;
    
    // Financial
    private double commissionRate;
    private BankAccount payoutAccount;
    private List<Transaction> transactionHistory;
    
    // Profile
    private List<Address> savedAddresses;
    private List<PaymentMethod> savedPaymentMethods;
    private String defaultPaymentMethodId;
    
    @Override
    public UserType getUserType() {
        return UserType.VENDOR;
    }
    
    @Override
    public void addProduct(Product product) {
        productCatalog.add(product);
    }
    
    @Override
    public void updateProduct(String productId, Product product) {
        productCatalog.removeIf(p -> p.getId().equals(productId));
        productCatalog.add(product);
    }
    
    @Override
    public void removeProduct(String productId) {
        productCatalog.removeIf(p -> p.getId().equals(productId));
    }
    
    @Override
    public void fulfillOrder(String orderId) {
        pendingOrders.stream()
            .filter(o -> o.getId().equals(orderId))
            .findFirst()
            .ifPresent(order -> {
                order.markAsFulfilled();
                pendingOrders.remove(order);
                
                // Record transaction
                double earnings = order.getTotal() * (1 - commissionRate);
                Transaction transaction = new Transaction(orderId, earnings);
                transactionHistory.add(transaction);
            });
    }
    
    @Override
    public double getTotalEarnings() {
        return transactionHistory.stream()
            .mapToDouble(Transaction::getAmount)
            .sum();
    }
    
    @Override
    public double getPendingPayouts() {
        return transactionHistory.stream()
            .filter(t -> !t.isPaidOut())
            .mapToDouble(Transaction::getAmount)
            .sum();
    }
    
    // ✨ NO shopping cart methods!
    // ✨ NO loyalty points!
    // ✨ NO admin methods!
    // ✨ Perfect for vendors!
}

/**
 * ✨ Admin User - Administrative capabilities
 */
@Data
public class AdminUser implements User, Administrator, ContentModerator {
    
    // Base user info
    private String userId;
    private String email;
    private String fullName;
    private String phoneNumber;
    private LocalDateTime createdAt;
    private boolean active;
    
    // Admin
    private Set<Permission> permissions;
    
    // Moderation
    private List<Review> pendingReviews;
    private List<ReportedContent> reportedContent;
    
    @Override
    public UserType getUserType() {
        return UserType.ADMIN;
    }
    
    @Override
    public boolean hasPermission(Permission permission) {
        return permissions.contains(permission);
    }
    
    @Override
    public void assignRole(String userId, Role role) {
        if (!hasPermission(Permission.MANAGE_USERS)) {
            throw new UnauthorizedException("No permission to manage users");
        }
        // Assignment logic
    }
    
    @Override
    public void revokeRole(String userId, Role role) {
        if (!hasPermission(Permission.MANAGE_USERS)) {
            throw new UnauthorizedException("No permission to manage users");
        }
        // Revocation logic
    }
    
    @Override
    public void approveReview(String reviewId) {
        pendingReviews.stream()
            .filter(r -> r.getId().equals(reviewId))
            .findFirst()
            .ifPresent(review -> {
                review.setApproved(true);
                pendingReviews.remove(review);
            });
    }
    
    @Override
    public void rejectReview(String reviewId, String reason) {
        pendingReviews.stream()
            .filter(r -> r.getId().equals(reviewId))
            .findFirst()
            .ifPresent(review -> {
                review.setRejected(true);
                review.setRejectionReason(reason);
                pendingReviews.remove(review);
            });
    }
    
    @Override
    public void takeAction(String contentId, ModerationAction action) {
        // Moderation action logic
    }
    
    // ✨ NO shopping cart!
    // ✨ NO vendor methods!
    // ✨ NO loyalty points!
    // ✨ Perfect for admins!
}

/**
 * ✨ Guest User - Minimal capabilities
 */
@Data
public class GuestUser implements User, Shopper, GuestSession {
    
    // Base user info
    private String userId;
    private String email; // Optional for guests
    private String fullName;
    private String phoneNumber;
    private LocalDateTime createdAt;
    private boolean active;
    
    // Guest session
    private String sessionId;
    private LocalDateTime sessionExpiry;
    
    // Shopping
    private ShoppingCart cart;
    
    @Override
    public UserType getUserType() {
        return UserType.GUEST;
    }
    
    @Override
    public void addToCart(Product product, int quantity) {
        if (isExpired()) {
            throw new SessionExpiredException();
        }
        cart.addItem(product, quantity);
    }
    
    @Override
    public void removeFromCart(String productId) {
        cart.removeItem(productId);
    }
    
    @Override
    public double getCartTotal() {
        return cart.calculateTotal();
    }
    
    @Override
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(sessionExpiry);
    }
    
    @Override
    public void extendSession(int minutes) {
        sessionExpiry = sessionExpiry.plusMinutes(minutes);
    }
    
    @Override
    public RegisteredCustomer convertToRegistered(RegistrationData data) {
        RegisteredCustomer customer = new RegisteredCustomer();
        customer.setUserId(UUID.randomUUID().toString());
        customer.setEmail(data.getEmail());
        customer.setFullName(data.getFullName());
        customer.setPhoneNumber(data.getPhoneNumber());
        customer.setCart(this.cart); // Transfer cart
        customer.setCreatedAt(LocalDateTime.now());
        customer.setActive(true);
        
        return customer;
    }
    
    // ✨ NO order history (guests can't place orders)!
    // ✨ NO saved addresses!
    // ✨ NO loyalty points!
    // ✨ Perfect for guest users!
}

// ========================================
// 4️⃣ SERVICE LAYER - Type-safe operations
// ========================================

@Service
public class UserService {
    
    /**
     * ✨ Works with ANY user type
     */
    public void displayUserInfo(User user) {
        System.out.println("User: " + user.getFullName());
        System.out.println("Type: " + user.getUserType());
        System.out.println("Email: " + user.getEmail());
    }
    
    /**
     * ✨ Works ONLY with order placers - type safe!
     */
    public void processCheckout(OrderPlacer customer, OrderRequest request) {
        if (customer.getCartTotal() == 0) {
            throw new EmptyCartException();
        }
        
        Order order = customer.placeOrder(request);
        System.out.println("✅ Order placed: " + order.getId());
    }
    
    /**
     * ✨ Works ONLY with loyalty members - type safe!
     */
    public void applyLoyaltyDiscount(LoyaltyMember member, double orderAmount) {
        double discount = orderAmount * member.getDiscountPercentage();
        System.out.println("💎 Loyalty discount: $" + discount);
        System.out.println("Tier: " + member.getTier());
    }
    
    /**
     * ✨ Works ONLY with sellers - type safe!
     */
    public void manageInventory(Seller vendor, String productId, int quantity) {
        Product product = vendor.getProductCatalog().stream()
            .filter(p -> p.getId().equals(productId))
            .findFirst()
            .orElseThrow(() -> new ProductNotFoundException(productId));
        
        if (product instanceof Stockable) {
            ((Stockable) product).updateInventory(quantity);
        }
    }
    
    /**
     * ✨ Works ONLY with administrators - type safe!
     */
    public void performAdminAction(Administrator admin, String action) {
        if (action.equals("DELETE_USER") && !admin.hasPermission(Permission.MANAGE_USERS)) {
            throw new UnauthorizedException("Insufficient permissions");
        }
        
        // Perform admin action
    }
    
    /**
     * ✨ Convert guest to customer - type safe!
     */
    public RegisteredCustomer convertGuestToCustomer(GuestSession guest, RegistrationData data) {
        if (guest.isExpired()) {
            throw new SessionExpiredException();
        }
        
        return guest.convertToRegistered(data);
    }
}
```

---

## 🎤 Interview Questions & Answers

### Q1: "What is Interface Segregation Principle?"

**Perfect Answer:**
```
ISP states that clients should not be forced to depend on 
interfaces they don't use. In simpler terms:

"Many small, specific interfaces are better than one large, 
general-purpose interface"

Real Example:

❌ VIOLATION:
interface Worker {
    void work();
    void eat();
    void sleep();
    void getSalary();
}

class Robot implements Worker {
    void work() { ... }
    void eat() { throw new UnsupportedOperationException(); } // 🔥
    void sleep() { throw new UnsupportedOperationException(); } // 🔥
    void getSalary() { throw new UnsupportedOperationException(); } // 🔥
}

✅ SOLUTION:
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }
interface Payable { void getSalary(); }

class Human implements Workable, Eatable, Sleepable, Payable { ... }
class Robot implements Workable { ... } // ✨ Only what it needs!

From my e-commerce project:
We had a Product interface with 32 methods.
Physical books had to implement digital download methods!
After ISP refactoring:
- Base Product: 6 methods
- PhysicalProduct, Downloadable, Stockable: 3-7 methods each
- Zero forced implementations
- 95% reduction in bugs!
```

---

### Q2: "How do you identify ISP violations?"

**Perfect Answer:**
```
Red flags that scream ISP violation:

1️⃣ THROWING EXCEPTIONS
   class DigitalProduct implements Product {
       double getWeight() {
           throw new UnsupportedOperationException(); // 🔥 ISP violation!
       }
   }

2️⃣ EMPTY METHOD IMPLEMENTATIONS
   class GuestUser implements User {
       List<Order> getOrderHistory() {
           return Collections.emptyList(); // 🔥 Meaningless!
       }
   }

3️⃣ RETURNING NULL OR DEFAULT VALUES
   class StreamingService implements Subscribable {
       String getDownloadUrl() {
           return null; // 🔥 Doesn't apply!
       }
   }

4️⃣ MULTIPLE "NOT APPLICABLE" COMMENTS
   // This method not applicable for digital products
   // This method not used by guest users
   // 🔥 If writing these comments, ISP violated!

5️⃣ BOOLEAN FLAGS TO SKIP FUNCTIONALITY
   void processOrder(Product product) {
       if (product.isPhysical()) {
           double weight = product.getWeight(); // Risky!
       }
   }

Real Example from Code Review:

❌ FOUND THIS:
interface PaymentMethod {
    void processPayment();
    void processRefund();
    void authorize();
    void capture();
    void setupRecurring();
}

class CashPayment implements PaymentMethod {
    void processRefund() { throw new Exception(); } // 🔥
    void authorize() { throw new Exception(); } // 🔥
    void capture() { throw new Exception(); } // 🔥
    void setupRecurring() { throw new Exception(); } // 🔥
}

✅ REFACTORED TO:
interface PaymentProcessor { void process(); }
interface Refundable { void refund(); }
interface Authorizable { void authorize(); void capture(); }
interface RecurringCapable { void setupRecurring(); }

class CreditCard implements PaymentProcessor, Refundable, 
                            Authorizable, RecurringCapable { }
class Cash implements PaymentProcessor { } // ✨ Clean!
```

---

### Q3: "Give a real production example of ISP violation causing problems"

**Perfect Answer:**
```
REAL INCIDENT - E-Commerce Platform:

CONTEXT:
We had a fat User interface with 25+ methods covering:
- Customers (order history, cart, wishlist)
- Vendors (product catalog, sales)
- Admins (permissions, moderation)
- Guests (session management)

THE REQUEST:
"Add Instagram-style user following feature"

DEVELOPER'S MISTAKE:
Added to fat User interface:
interface User {
    // ... existing 25 methods
    List<User> getFollowers();  // NEW
    List<User> getFollowing();  // NEW
    void followUser(String userId);  // NEW
    void unfollowUser(String userId);  // NEW
}

ALL implementing classes needed changes:
- GuestUser (guests can't follow!)
- AdminUser (admins don't follow!)
- VendorAccount (vendors don't need this!)

WHAT HAPPENED:
Developer implemented:

class AdminUser implements User {
    List<User> getFollowers() {
        return Collections.emptyList(); // Empty list
    }
    
    void followUser(String userId) {
        // Did nothing - no exception thrown!
    }
}

DEPLOYED TO PRODUCTION:

BUG DISCOVERED:
Admin accidentally clicked "Follow" button (shown due to UI bug).
No error occurred (method silently did nothing).
UI showed "Following" but didn't work.
Admins reported "Following feature broken!"

IMPACT:
💥 5 days debugging (hard to find silent failures)
💥 Had to add instanceof checks everywhere
💥 User trust decreased
💥 Team spent 40 hours fixing

THE FIX (ISP):
interface User { ... basic methods ... }
interface SocialUser extends User {
    List<User> getFollowers();
    void followUser(String userId);
}

class Customer implements SocialUser { ... } ✅
class Vendor implements SocialUser { ... } ✅
class AdminUser implements User { ... } ✅ No social methods!
class GuestUser implements User { ... } ✅ No social methods!

RESULT:
✅ Compile-time safety (admins CAN'T follow)
✅ UI correctly hides follow button for admins
✅ Zero silent failures
✅ Clear separation of concerns

LESSON:
ISP violations create SILENT BUGS that are hard to detect!
```

---

### Q4: "How is ISP different from SRP?"

**Perfect Answer:**
```
Great question! They're related but different:

SRP (Single Responsibility Principle):
- About CLASSES having one reason to change
- Focuses on IMPLEMENTATION
- "A class should do ONE thing"

ISP (Interface Segregation Principle):
- About INTERFACES not forcing unused methods
- Focuses on CONTRACT/API
- "Clients shouldn't depend on what they don't use"

THEY WORK TOGETHER:

Example 1 - Violates BOTH:

class ProductManager {  // Violates SRP
    void addProduct() { }
    void calculateShipping() { }
    void sendEmail() { }
    void updateInventory() { }
    void generateReport() { }
}

interface Product {  // Violates ISP
    void ship();
    void download();
    void stream();
    // Digital products forced to implement ship()!
}

Example 2 - Follows BOTH:

// ISP - Segregated interfaces
interface Shippable { void ship(); }
interface Downloadable { void download(); }
interface Streamable { void stream(); }

// SRP - Single responsibility classes
class ShippingService { ... }  // Only shipping
class DownloadService { ... }  // Only downloads
class ReportService { ... }    // Only reports

REAL ANALOGY:

SRP = Swiss Army Knife vs Individual Tools
❌ One class doing everything (Swiss Army Knife)
✅ Separate classes for separate jobs (Individual Tools)

ISP = Restaurant Menu
❌ One huge menu forcing vegetarians to see meat options
✅ Separate menus (Veg, Non-Veg, Drinks) - pick what you need

From my project:

SRP Issue:
OrderService doing: validation, payment, shipping, email
Solution: Separate services for each

ISP Issue:
Product interface with physical + digital + service methods
Solution: Separate interfaces (PhysicalProduct, Downloadable, etc.)

BOTH together = Clean, maintainable code!
```

---

### Q5: "Doesn't ISP lead to too many interfaces?"

**Perfect Answer:**
```
Yes, if done wrong! It's about BALANCE.

❌ OVER-ENGINEERING (Too many interfaces):

interface Nameable { String getName(); }
interface Priceable { double getPrice(); }
interface Identifiable { String getId(); }
interface Describable { String getDescription(); }

class Product implements Nameable, Priceable, 
                        Identifiable, Describable {
    // 4 interfaces for basic properties - OVERKILL!
}

✅ BALANCED APPROACH:

interface Product {
    String getId();
    String getName();
    double getPrice();
    String getDescription();
}

// Segregate only for VARYING CAPABILITIES
interface Shippable { ... }
interface Downloadable { ... }
interface Customizable { ... }

RULES OF THUMB:

1️⃣ GROUP COHESIVE METHODS
✅ interface Shippable {
       double getWeight();
       Dimensions getDimensions();
       String getShippingClass();
   }
   // All related to shipping - GOOD

❌ interface Weightable { double getWeight(); }
   interface Dimensional { Dimensions getDimensions(); }
   // Too granular - BAD

2️⃣ SPLIT WHEN IMPLEMENTATIONS VARY
✅ Split if:
   - Some products are downloadable, some aren't
   - Some users are sellers, some aren't
   - Some payments refundable, some aren't

❌ Don't split if:
   - ALL products have name, price, ID
   - ALL users have email, name
   - ALL orders have date, total

3️⃣ ASK: "WILL CLIENTS USE ALL METHODS?"
✅ Split if client uses only subset
❌ Keep together if client needs all methods

Real Example from my project:

BEFORE (Too many interfaces - 15 interfaces):
interface HasId { String getId(); }
interface HasName { String getName(); }
interface HasPrice { double getPrice(); }
// Too granular!

AFTER (Balanced - 5 interfaces):
interface Product { ... core properties ... }
interface PhysicalProduct extends Product { ... }
interface Downloadable extends Product { ... }
interface Subscribable extends Product { ... }
// Perfect balance!

GUIDELINE:
- 3-7 methods per interface = GOOD
- 1-2 methods = Probably too granular
- 15+ methods = Probably too fat

FINAL TEST:
"Can I describe this interface in one sentence?"
✅ "Products that can be shipped" → Good
✅ "Products that can be downloaded" → Good
❌ "Things that have names" → Too granular
```

---

### Q6: "How does ISP relate to other SOLID principles?"

**Perfect Answer:**
```
ISP works synergistically with all SOLID principles:

ISP + SRP:
- SRP: Class has one responsibility
- ISP: Interface exposes one capability
- Together: Focused, cohesive design

Example:
interface OrderPlacer { Order placeOrder(); }  // ISP
class OrderService implements OrderPlacer { }  // SRP
// One interface, one implementation, one job

ISP + OCP:
- OCP: Open for extension, closed for modification
- ISP: Small interfaces easier to extend

Example:
interface PaymentProcessor { void process(); }  // ISP
interface Refundable { void refund(); }         // ISP

// Easy to add new payment with refund capability (OCP)
class NewPayment implements PaymentProcessor, Refundable { }

ISP + LSP:
- LSP: Subtypes must be substitutable
- ISP: Prevents forced implementations that break LSP

Example WITHOUT ISP (Breaks LSP):
interface Product { void ship(); }
class DigitalProduct implements Product {
    void ship() { throw new Exception(); } // 🔥 Breaks LSP!
}

Example WITH ISP (Preserves LSP):
interface Product { }
interface Shippable { void ship(); }

class PhysicalProduct implements Product, Shippable { ... } ✅
class DigitalProduct implements Product { ... } ✅
// Both are valid Product subtypes (LSP) thanks to ISP!

ISP + DIP:
- DIP: Depend on abstractions
- ISP: Depend on right-sized abstractions

Example:
class OrderService {
    private PaymentProcessor processor;  // DIP
    
    // Thanks to ISP, PaymentProcessor is focused
    // Not forced to depend on unused refund/recurring methods
}

REAL EXAMPLE FROM MY PROJECT:

Problem: Fat interface breaking multiple principles

interface Worker {  // Violates ISP
    void work();
    void eat();
    void getSalary();
}

class Robot implements Worker {
    void eat() { throw ... }      // Violates LSP
    void getSalary() { throw ... } // Violates LSP
}

class WorkerService {
    void feedWorker(Worker w) {
        w.eat(); // Might throw! Violates LSP
    }
}

Solution: ISP enables all principles

interface Workable { void work(); }
interface Eatable { void eat(); }
interface Payable { void getSalary(); }

class Human implements Workable, Eatable, Payable { } // LSP ✅
class Robot implements Workable { }                   // LSP ✅

class FeedingService {
    void feed(Eatable e) { e.eat(); }  // DIP + LSP ✅
}

class WorkManager {
    void assign(Workable w) { w.work(); }  // SRP + DIP ✅
}

// Can add new worker types easily (OCP) ✅

ALL SOLID PRINCIPLES WORK TOGETHER:
S - Focused classes
O - Easy to extend
L - Proper substitution
I - Right-sized interfaces ← Enables the others!
D - Depend on abstractions
```

---

## 🎯 Quick Reference Card

```java
/**
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 *  INTERFACE SEGREGATION PRINCIPLE (ISP)
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 * 
 * 🎯 DEFINITION:
 * "Clients should not be forced to depend on 
 *  interfaces they don't use"
 * 
 * 🔍 IDENTIFY VIOLATIONS:
 * ❌ throw new UnsupportedOperationException()
 * ❌ Empty method implementations
 * ❌ Returning null/default values
 * ❌ Methods that don't apply to class
 * ❌ Many instanceof checks
 * ❌ Comments saying "not applicable"
 * 
 * ✅ SOLUTIONS:
 * ✓ Split large interfaces into smaller ones
 * ✓ Group cohesive methods together
 * ✓ Create role-based interfaces
 * ✓ Use interface composition
 * ✓ Apply only what's needed
 * 
 * 🔧 REFACTORING PATTERN:
 * 
 * BEFORE (Fat Interface):
 * interface Product {
 *     // 30 methods covering all product types
 * }
 * 
 * AFTER (Segregated):
 * interface Product { }              // Base
 * interface PhysicalProduct { }      // Role
 * interface Downloadable { }         // Role
 * interface Customizable { }         // Role
 * 
 * class Book implements Product, PhysicalProduct { }
 * class EBook implements Product, Downloadable { }
 * 
 * 🎁 BENEFITS:
 * ✓ No forced implementations
 * ✓ No UnsupportedOperationException
 * ✓ Clear responsibilities
 * ✓ Easier to understand
 * ✓ Better testability
 * ✓ Maintains LSP
 * ✓ Flexible composition
 * 
 * ⚠️ BALANCE:
 * ✓ 3-7 methods per interface = Good
 * ⚠️ 1-2 methods = Might be too granular
 * ❌ 15+ methods = Probably too fat
 * 
 * 💡 THE TESTS:
 * 1. "Does this class use ALL interface methods?"
 *    NO → ISP violated
 * 
 * 2. "Do I have empty/throwing implementations?"
 *    YES → ISP violated
 * 
 * 3. "Can I describe interface in one sentence?"
 *    NO → Probably too complex
 * 
 * 📝 REMEMBER:
 * "Many small interfaces > One fat interface"
 * "Implement only what you need"
 * 
 * ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 */
```

---

## 🌟 Final Pro Tips

### 1️⃣ Start with Violations
```java
// Look for this smell in your code:
throw new UnsupportedOperationException();
// Every time you see this = ISP violation!
```

### 2️⃣ Use Composition Over Inheritance
```java
// Instead of huge inheritance hierarchy
// Use interface composition
class CustomTShirt implements Physical, Stockable, Customizable { }
```

### 3️⃣ Think "Roles" Not "Types"
```java
// ✅ Role-based
interface Purchasable { }
interface Rentable { }
interface Downloadable { }

// ❌ Type-based (too specific)
interface Book { }
interface Movie { }
interface Song { }
```

### 4️⃣ Group Related Methods
```java
// ✅ Cohesive
interface Shippable {
    double getWeight();
    Dimensions getDimensions();
    String getShippingClass();
}

// ❌ Not cohesive
interface Product {
    double getWeight();
    String getDownloadUrl();
    boolean isRecurring();
}
```

### 5️⃣ Document Interface Purpose
```java
/**
 * Products that can be physically shipped.
 * Implement this for products with weight and dimensions.
 */
public interface Shippable { }
```


---

**Remember:** 🌟

> "Force no one to implement what they don't need.
> Design interfaces like a buffet, not a set menu.
> Take only what you'll actually use!"

---

*Made with ❤️ for Clean Interfaces*
*Zero Forced Methods Rate: 💯%*

🚀 **Happy Segregating!**
