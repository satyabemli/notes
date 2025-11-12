# 🚀 Complete Java Developer Guide with Real-Time Examples

---

# 📚 JAVA FUNDAMENTALS

## 🧠 JVM Memory Model

### 💡 Explanation
The JVM divides memory into different areas to manage application data efficiently:

- **Heap Memory** 🏗️: Stores objects and instance variables (shared across threads)
- **Stack Memory** 📚: Stores local variables and method calls (thread-specific)
- **Method Area** 📖: Stores class structures, static variables, constants
- **PC Registers** 🎯: Tracks current instruction execution
- **Native Method Stack** 🔧: For native (C/C++) method execution

### 🎯 Real-Time Example: E-Commerce Shopping Cart

```java
public class ShoppingCartMemoryDemo {
    
    // Stored in METHOD AREA (class level - static)
    private static int totalCustomers = 0; 
    private static final String CURRENCY = "USD"; // Constant Pool
    
    // Stored in HEAP (instance variables)
    private String customerId;
    private List<Product> cartItems;
    
    public ShoppingCartMemoryDemo(String customerId) {
        this.customerId = customerId; // Reference in Stack, Object in Heap
        this.cartItems = new ArrayList<>(); // ArrayList object in Heap
        totalCustomers++; // Modified in Method Area
    }
    
    public double calculateTotal() {
        // Local variables stored in STACK
        double total = 0.0; 
        int itemCount = 0;
        
        // Loop variables also in STACK
        for (Product product : cartItems) { 
            total += product.getPrice(); // product reference in Stack, object in Heap
            itemCount++;
        }
        
        return total; // Removed from stack after method returns
    }
    
    public static void main(String[] args) {
        // 'cart1' reference in STACK, actual object in HEAP
        ShoppingCartMemoryDemo cart1 = new ShoppingCartMemoryDemo("CUST001");
        
        // 'cart2' reference in STACK, another object in HEAP
        ShoppingCartMemoryDemo cart2 = new ShoppingCartMemoryDemo("CUST002");
        
        System.out.println("Total Customers: " + totalCustomers); // Accessed from Method Area
    }
}

class Product {
    private String name;   // Heap
    private double price;  // Heap
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public double getPrice() { return price; }
}
```

**🧠 Memory Breakdown:**
- ✅ `totalCustomers`, `CURRENCY` → Method Area
- ✅ `cart1`, `cart2` objects → Heap Memory
- ✅ Local variables (`total`, `itemCount`) → Stack Memory
- ✅ Method execution frames → Stack Memory

---

## 🎭 OOPs Concepts

### 💡 The Four Pillars

1. **Encapsulation** 🔒: Hiding internal details
2. **Inheritance** 👨‍👦: Reusing parent class properties
3. **Polymorphism** 🎪: Same interface, different implementations
4. **Abstraction** 🎨: Showing only essential features

### 🎯 Real-Time Example: Payment Gateway System

```java
// 🎨 ABSTRACTION - Defining contract without implementation
abstract class PaymentGateway {
    protected String merchantId;
    
    // Abstract method - must be implemented by children
    public abstract PaymentResponse processPayment(double amount);
    
    // Concrete method - shared functionality
    public void logTransaction(String message) {
        System.out.println("📝 LOG: " + message);
    }
}

// 👨‍👦 INHERITANCE - Reusing parent class code
class StripePayment extends PaymentGateway {
    
    // 🔒 ENCAPSULATION - Private fields with controlled access
    private String apiKey;
    private String secretKey;
    
    public StripePayment(String merchantId, String apiKey) {
        this.merchantId = merchantId; // Inherited from parent
        this.apiKey = apiKey;
    }
    
    // Getter provides controlled access (Encapsulation)
    public String getApiKey() {
        return "****" + apiKey.substring(apiKey.length() - 4); // Masked
    }
    
    // 🎪 POLYMORPHISM - Overriding parent method
    @Override
    public PaymentResponse processPayment(double amount) {
        logTransaction("Processing Stripe payment of $" + amount);
        // Stripe-specific logic
        return new PaymentResponse(true, "STRIPE_" + System.currentTimeMillis());
    }
}

class PayPalPayment extends PaymentGateway {
    private String email;
    
    public PayPalPayment(String merchantId, String email) {
        this.merchantId = merchantId;
        this.email = email;
    }
    
    // 🎪 POLYMORPHISM - Different implementation for same method
    @Override
    public PaymentResponse processPayment(double amount) {
        logTransaction("Processing PayPal payment of $" + amount);
        // PayPal-specific logic
        return new PaymentResponse(true, "PAYPAL_" + System.currentTimeMillis());
    }
}

// 🎪 POLYMORPHISM in action
class PaymentProcessor {
    
    // Same method handles different payment types
    public void charge(PaymentGateway gateway, double amount) {
        PaymentResponse response = gateway.processPayment(amount);
        System.out.println("💳 Payment Status: " + response.isSuccess());
    }
    
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        
        // Polymorphism - same interface, different behavior
        PaymentGateway stripe = new StripePayment("MERCH123", "sk_test_12345");
        PaymentGateway paypal = new PayPalPayment("MERCH123", "merchant@shop.com");
        
        processor.charge(stripe, 99.99); // Calls Stripe implementation
        processor.charge(paypal, 149.99); // Calls PayPal implementation
    }
}

class PaymentResponse {
    private boolean success;
    private String transactionId;
    
    public PaymentResponse(boolean success, String transactionId) {
        this.success = success;
        this.transactionId = transactionId;
    }
    
    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
}
```

**🎯 Key Takeaways:**
- 🔒 **Encapsulation**: Private fields with getters (masked API key)
- 👨‍👦 **Inheritance**: `extends PaymentGateway` reuses code
- 🎪 **Polymorphism**: Same `processPayment()` method, different behavior
- 🎨 **Abstraction**: `abstract class` defines contract

---

## ☕ Java 8 Concepts

### 🌊 Stream API

### 💡 Explanation
Streams provide a **declarative** way to process collections - you say **WHAT** you want, not **HOW** to do it.

**Key Operations:**
- **Intermediate** 🔄: `filter()`, `map()`, `sorted()` (lazy, return Stream)
- **Terminal** 🏁: `collect()`, `forEach()`, `reduce()` (eager, return result)

### 🎯 Real-Time Example: E-Commerce Order Processing

```java
import java.util.*;
import java.util.stream.*;
import java.math.BigDecimal;

public class OrderProcessingSystem {
    
    public static void main(String[] args) {
        List<Order> orders = Arrays.asList(
            new Order("ORD001", "Electronics", 1200.00, "COMPLETED", "PREMIUM"),
            new Order("ORD002", "Books", 45.00, "PENDING", "STANDARD"),
            new Order("ORD003", "Electronics", 800.00, "COMPLETED", "PREMIUM"),
            new Order("ORD004", "Clothing", 150.00, "CANCELLED", "STANDARD"),
            new Order("ORD005", "Electronics", 2500.00, "COMPLETED", "PREMIUM"),
            new Order("ORD006", "Books", 30.00, "COMPLETED", "STANDARD")
        );
        
        // 🎯 REAL-TIME USE CASE: Calculate revenue from completed premium electronics orders
        double premiumElectronicsRevenue = orders.stream()
            .filter(order -> order.getStatus().equals("COMPLETED"))     // Only completed
            .filter(order -> order.getCategory().equals("Electronics")) // Only electronics
            .filter(order -> order.getCustomerTier().equals("PREMIUM")) // Only premium customers
            .mapToDouble(Order::getAmount)                              // Extract amounts
            .sum();                                                     // Sum them up
        
        System.out.println("💰 Premium Electronics Revenue: $" + premiumElectronicsRevenue);
        
        
        // 🎯 REAL-TIME USE CASE: Get top 3 highest value completed orders
        List<Order> top3Orders = orders.stream()
            .filter(order -> order.getStatus().equals("COMPLETED"))
            .sorted(Comparator.comparing(Order::getAmount).reversed())
            .limit(3)
            .collect(Collectors.toList());
        
        System.out.println("\n🏆 Top 3 Orders:");
        top3Orders.forEach(order -> 
            System.out.println("  " + order.getOrderId() + ": $" + order.getAmount()));
        
        
        // 🎯 REAL-TIME USE CASE: Group orders by category and count them
        Map<String, Long> ordersByCategory = orders.stream()
            .collect(Collectors.groupingBy(
                Order::getCategory, 
                Collectors.counting()
            ));
        
        System.out.println("\n📊 Orders by Category:");
        ordersByCategory.forEach((category, count) -> 
            System.out.println("  " + category + ": " + count + " orders"));
        
        
        // 🎯 REAL-TIME USE CASE: Average order value by customer tier
        Map<String, Double> avgOrderByTier = orders.stream()
            .filter(order -> order.getStatus().equals("COMPLETED"))
            .collect(Collectors.groupingBy(
                Order::getCustomerTier,
                Collectors.averagingDouble(Order::getAmount)
            ));
        
        System.out.println("\n📈 Average Order Value by Tier:");
        avgOrderByTier.forEach((tier, avg) -> 
            System.out.println("  " + tier + ": $" + String.format("%.2f", avg)));
    }
}

class Order {
    private String orderId;
    private String category;
    private double amount;
    private String status;
    private String customerTier;
    
    public Order(String orderId, String category, double amount, String status, String customerTier) {
        this.orderId = orderId;
        this.category = category;
        this.amount = amount;
        this.status = status;
        this.customerTier = customerTier;
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public String getCategory() { return category; }
    public double getAmount() { return amount; }
    public String getStatus() { return status; }
    public String getCustomerTier() { return customerTier; }
}
```

**📝 Output:**
```
💰 Premium Electronics Revenue: $4500.0
🏆 Top 3 Orders:
  ORD005: $2500.0
  ORD001: $1200.0
  ORD003: $800.0
📊 Orders by Category:
  Electronics: 3 orders
  Clothing: 1 orders
  Books: 2 orders
📈 Average Order Value by Tier:
  STANDARD: $37.5
  PREMIUM: $1500.0
```

---

### 🎭 Functional Interface

### 💡 Explanation
Interface with **exactly ONE abstract method** (can have multiple default/static methods). Enables functional programming and lambda expressions.

**Common Built-in Functional Interfaces:**
- `Predicate<T>` ✅: `T → boolean` (test condition)
- `Function<T, R>` 🔄: `T → R` (transform input)
- `Consumer<T>` 📥: `T → void` (consume input)
- `Supplier<T>` 📤: `() → T` (supply output)

### 🎯 Real-Time Example: Notification System

```java
import java.util.*;
import java.util.function.*;

// 🎭 Custom Functional Interface
@FunctionalInterface
interface NotificationSender {
    void send(String message, String recipient);
    
    // Default methods are allowed
    default void sendBulk(List<String> messages, String recipient) {
        messages.forEach(msg -> send(msg, recipient));
    }
}

public class NotificationSystem {
    
    public static void main(String[] args) {
        
        // ✅ PREDICATE - Test if user is eligible for notification
        Predicate<User> isActiveUser = user -> user.isActive() && user.isEmailVerified();
        
        // 🔄 FUNCTION - Transform user to notification message
        Function<User, String> createWelcomeMessage = user -> 
            "Welcome " + user.getName() + "! 🎉 Your account is ready.";
        
        // 📥 CONSUMER - Process notification (side effect)
        Consumer<String> logger = message -> 
            System.out.println("📝 Logged: " + message);
        
        // 📤 SUPPLIER - Generate timestamp
        Supplier<String> timestamp = () -> 
            java.time.LocalDateTime.now().toString();
        
        
        // 🎭 Using Custom Functional Interface
        NotificationSender emailSender = (message, recipient) -> {
            System.out.println("📧 Email sent to: " + recipient);
            System.out.println("   Message: " + message);
            System.out.println("   Time: " + timestamp.get());
        };
        
        NotificationSender smsSender = (message, recipient) -> {
            System.out.println("📱 SMS sent to: " + recipient);
            System.out.println("   Message: " + message);
        };
        
        
        // 🎯 REAL-TIME SCENARIO: Send notifications to eligible users
        List<User> users = Arrays.asList(
            new User("John", "john@email.com", true, true),
            new User("Jane", "jane@email.com", false, true),  // Not active
            new User("Bob", "bob@email.com", true, false)     // Email not verified
        );
        
        System.out.println("🚀 Sending Welcome Notifications...\n");
        
        users.stream()
            .filter(isActiveUser)  // Using Predicate
            .forEach(user -> {
                String message = createWelcomeMessage.apply(user); // Using Function
                emailSender.send(message, user.getEmail());        // Using Custom FI
                logger.accept("Notification sent to " + user.getName()); // Using Consumer
                System.out.println();
            });
        
        
        // 🎯 Method Reference Example
        NotificationSender slackSender = NotificationSystem::sendSlackMessage;
        slackSender.send("Deployment successful! ✅", "#dev-team");
    }
    
    // Method that matches NotificationSender signature
    public static void sendSlackMessage(String message, String channel) {
        System.out.println("💬 Slack message to " + channel + ": " + message);
    }
}

class User {
    private String name;
    private String email;
    private boolean active;
    private boolean emailVerified;
    
    public User(String name, String email, boolean active, boolean emailVerified) {
        this.name = name;
        this.email = email;
        this.active = active;
        this.emailVerified = emailVerified;
    }
    
    public String getName() { return name; }
    public String getEmail() { return email; }
    public boolean isActive() { return active; }
    public boolean isEmailVerified() { return emailVerified; }
}
```

---

### 🧬 Functional Programming

### 💡 Explanation
Programming paradigm treating computation as evaluation of **pure functions**, avoiding **changing state** and **mutable data**.

**Core Principles:**
- 🎯 **Pure Functions**: Same input → Same output (no side effects)
- 🔒 **Immutability**: Data cannot be changed after creation
- 🎪 **First-Class Functions**: Functions as values
- 🔄 **Higher-Order Functions**: Functions that take/return functions

### 🎯 Real-Time Example: Price Calculation Engine

```java
import java.util.*;
import java.util.function.*;

public class PriceCalculator {
    
    // 🎯 PURE FUNCTION - No side effects, same input = same output
    public static double calculateBasePrice(int quantity, double unitPrice) {
        return quantity * unitPrice;
    }
    
    // 🔄 HIGHER-ORDER FUNCTION - Takes function as parameter
    public static double applyDiscount(double price, Function<Double, Double> discountStrategy) {
        return discountStrategy.apply(price);
    }
    
    // 🔄 HIGHER-ORDER FUNCTION - Returns function
    public static Function<Double, Double> createSeasonalDiscount(String season) {
        return switch (season) {
            case "BLACK_FRIDAY" -> price -> price * 0.5;  // 50% off
            case "CHRISTMAS" -> price -> price * 0.7;     // 30% off
            case "REGULAR" -> price -> price * 0.9;       // 10% off
            default -> price -> price;                     // No discount
        };
    }
    
    // 🔗 FUNCTION COMPOSITION - Combining multiple functions
    public static Function<Double, Double> createPricingPipeline(
            Function<Double, Double> discount,
            Function<Double, Double> tax,
            Function<Double, Double> shippingFee) {
        
        // Compose functions: price -> discount -> tax -> shipping
        return discount.andThen(tax).andThen(shippingFee);
    }
    
    public static void main(String[] args) {
        
        // 📦 Product details
        int quantity = 5;
        double unitPrice = 100.0;
        
        // 🎯 Pure function call
        double basePrice = calculateBasePrice(quantity, unitPrice);
        System.out.println("💵 Base Price: $" + basePrice);
        
        
        // 🔄 Higher-order function usage
        Function<Double, Double> blackFridayDiscount = createSeasonalDiscount("BLACK_FRIDAY");
        double discountedPrice = applyDiscount(basePrice, blackFridayDiscount);
        System.out.println("🎉 Black Friday Price: $" + discountedPrice);
        
        
        // 🔗 FUNCTION COMPOSITION - Real-time pricing pipeline
        Function<Double, Double> memberDiscount = price -> price * 0.85;  // 15% member discount
        Function<Double, Double> taxCalculation = price -> price * 1.08;  // 8% tax
        Function<Double, Double> shipping = price -> price + 10.0;        // $10 shipping
        
        Function<Double, Double> completePricingPipeline = 
            createPricingPipeline(memberDiscount, taxCalculation, shipping);
        
        double finalPrice = completePricingPipeline.apply(basePrice);
        System.out.println("💳 Final Price (Member + Tax + Shipping): $" + 
            String.format("%.2f", finalPrice));
        
        
        // 🔒 IMMUTABILITY Example
        List<Double> prices = List.of(100.0, 200.0, 300.0); // Immutable list
        
        // Instead of modifying list, create new list with transformed values
        List<Double> discountedPrices = prices.stream()
            .map(blackFridayDiscount)  // Transform each element
            .toList();                 // Create new list
        
        System.out.println("\n📊 Original Prices: " + prices);
        System.out.println("🎁 Discounted Prices: " + discountedPrices);
        
        
        // 🎪 FIRST-CLASS FUNCTIONS - Storing functions in data structures
        Map<String, Function<Double, Double>> discountStrategies = Map.of(
            "VIP", price -> price * 0.75,
            "PREMIUM", price -> price * 0.85,
            "REGULAR", price -> price * 0.95
        );
        
        String customerTier = "VIP";
        Function<Double, Double> selectedDiscount = discountStrategies.get(customerTier);
        double vipPrice = selectedDiscount.apply(basePrice);
        
        System.out.println("\n👑 " + customerTier + " Customer Price: $" + vipPrice);
    }
}
```

**📝 Output:**
```
💵 Base Price: $500.0
🎉 Black Friday Price: $250.0
💳 Final Price (Member + Tax + Shipping): $469.00

📊 Original Prices: [100.0, 200.0, 300.0]
🎁 Discounted Prices: [50.0, 100.0, 150.0]

👑 VIP Customer Price: $375.0
```

---

### ⚡ Lambda Expressions

### 💡 Explanation
Anonymous functions providing **cleaner syntax** for implementing functional interfaces. Format: `(parameters) -> expression` or `(parameters) -> { statements; }`

### 🎯 Real-Time Example: Event Handler System

```java
import java.util.*;
import java.util.function.*;

@FunctionalInterface
interface EventHandler {
    void handle(Event event);
}

public class EventProcessingSystem {
    
    private Map<String, List<EventHandler>> listeners = new HashMap<>();
    
    // Register event listener
    public void addEventListener(String eventType, EventHandler handler) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(handler);
    }
    
    // Trigger event
    public void fireEvent(Event event) {
        List<EventHandler> handlers = listeners.get(event.getType());
        if (handlers != null) {
            handlers.forEach(handler -> handler.handle(event));
        }
    }
    
    public static void main(String[] args) {
        EventProcessingSystem eventSystem = new EventProcessingSystem();
        
        // ⚡ LAMBDA - Single line expression
        eventSystem.addEventListener("USER_LOGIN", 
            event -> System.out.println("✅ User logged in: " + event.getData()));
        
        // ⚡ LAMBDA - Multiple statements block
        eventSystem.addEventListener("USER_LOGIN", event -> {
            System.out.println("📊 Updating analytics...");
            System.out.println("🕒 Login time: " + event.getTimestamp());
            System.out.println("👤 User: " + event.getData());
        });
        
        // ⚡ LAMBDA - With type declaration
        eventSystem.addEventListener("ORDER_PLACED", (Event event) -> {
            System.out.println("🛒 New order received!");
            System.out.println("📦 Order details: " + event.getData());
            System.out.println("💳 Processing payment...");
        });
        
        // ⚡ LAMBDA - Stored in variable
        EventHandler emailNotifier = event -> 
            System.out.println("📧 Email sent for: " + event.getType());
        
        eventSystem.addEventListener("ORDER_PLACED", emailNotifier);
        eventSystem.addEventListener("USER_REGISTRATION", emailNotifier);
        
        
        // 🔥 Different Lambda Styles
        
        // No parameters
        Runnable task1 = () -> System.out.println("🔄 Background task running");
        
        // One parameter (parentheses optional)
        Consumer<String> logger = msg -> System.out.println("📝 LOG: " + msg);
        
        // Multiple parameters
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        
        // Returning value
        Supplier<Double> randomPrice = () -> Math.random() * 100;
        
        // Complex lambda
        Comparator<Product> byPriceThenName = (p1, p2) -> {
            int priceComparison = Double.compare(p1.getPrice(), p2.getPrice());
            return priceComparison != 0 ? priceComparison : p1.getName().compareTo(p2.getName());
        };
        
        
        // 🎯 TRIGGERING EVENTS
        System.out.println("🚀 Simulating Real-Time Events\n");
        
        eventSystem.fireEvent(new Event("USER_LOGIN", "john@example.com"));
        System.out.println();
        
        eventSystem.fireEvent(new Event("ORDER_PLACED", "Order #12345 - $299.99"));
        System.out.println();
        
        eventSystem.fireEvent(new Event("USER_REGISTRATION", "jane@example.com"));
        
        
        // 🎯 Using other lambdas
        System.out.println("\n🔧 Other Lambda Examples:");
        task1.run();
        logger.accept("System initialized");
        System.out.println("➕ 5 + 3 = " + add.apply(5, 3));
        System.out.println("💰 Random price: $" + String.format("%.2f", randomPrice.get()));
    }
}

class Event {
    private String type;
    private String data;
    private long timestamp;
    
    public Event(String type, String data) {
        this.type = type;
        this.data = data;
        this.timestamp = System.currentTimeMillis();
    }
    
    public String getType() { return type; }
    public String getData() { return data; }
    public long getTimestamp() { return timestamp; }
}

class Product {
    private String name;
    private double price;
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

---

### 🎁 Default Methods

### 💡 Explanation
Methods in interfaces with **default implementation**. Allows adding new methods to interfaces **without breaking** existing implementations. Introduced in Java 8 to enable **interface evolution**.

### 🎯 Real-Time Example: Payment Processing Evolution

```java
// 🎯 Original interface (Java 7 style)
interface PaymentProcessor_V1 {
    boolean processPayment(double amount);
}

// 🎁 Evolved interface with default methods (Java 8+)
interface PaymentProcessor_V2 {
    // Original abstract method
    boolean processPayment(double amount);
    
    // ✨ NEW: Default method - doesn't break existing implementations
    default boolean processRefund(double amount) {
        System.out.println("💰 Processing standard refund: $" + amount);
        return true;
    }
    
    // ✨ NEW: Default method with logic
    default PaymentResult processPaymentWithRetry(double amount, int maxRetries) {
        System.out.println("🔄 Attempting payment with " + maxRetries + " retries");
        
        for (int i = 1; i <= maxRetries; i++) {
            System.out.println("   Attempt " + i + "...");
            if (processPayment(amount)) {
                return new PaymentResult(true, "Success on attempt " + i);
            }
            
            try {
                Thread.sleep(1000); // Wait before retry
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        return new PaymentResult(false, "Failed after " + maxRetries + " attempts");
    }
    
    // ✨ Default method calling other default method
    default void processFullTransaction(double amount) {
        System.out.println("📋 Starting full transaction for $" + amount);
        
        if (processPayment(amount)) {
            System.out.println("✅ Payment successful");
            generateReceipt(amount);
        } else {
            System.out.println("❌ Payment failed");
        }
    }
    
    default void generateReceipt(double amount) {
        System.out.println("🧾 Receipt generated for $" + amount);
        System.out.println("   Date: " + java.time.LocalDateTime.now());
    }
    
    // 🎯 Static method in interface (Java 8+)
    static boolean validateAmount(double amount) {
        return amount > 0 && amount <= 10000;
    }
}

// 🏦 Old implementation - Still works without changes!
class StripePayment implements PaymentProcessor_V2 {
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("💳 Stripe processing payment: $" + amount);
        return amount < 5000; // Simulate: fails for large amounts
    }
    
    // Can optionally override default methods
    @Override
    public boolean processRefund(double amount) {
        System.out.println("💳 Stripe-specific refund logic: $" + amount);
        return true;
    }
}

// 🏦 New implementation - Can use all default methods
class PayPalPayment implements PaymentProcessor_V2 {
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("🅿️ PayPal processing payment: $" + amount);
        return true;
    }
    
    // Using default refund method (no need to implement)
}

// 🔥 Diamond Problem Solution
interface FraudDetection {
    default boolean checkFraud(double amount) {
        System.out.println("🔍 Basic fraud check");
        return amount < 10000;
    }
}

interface AdvancedSecurity {
    default boolean checkFraud(double amount) {
        System.out.println("🛡️ Advanced fraud check");
        return amount < 5000;
    }
}

class SecurePayment implements PaymentProcessor_V2, FraudDetection, AdvancedSecurity {
    
    @Override
    public boolean processPayment(double amount) {
        // Must explicitly resolve the diamond problem
        boolean isSafe = checkFraud(amount);
        
        if (isSafe) {
            System.out.println("🔒 Secure payment processed: $" + amount);
            return true;
        }
        return false;
    }
    
    // 🎯 MUST override to resolve conflict between FraudDetection and AdvancedSecurity
    @Override
    public boolean checkFraud(double amount) {
        System.out.println("🎯 Custom fraud check combining both");
        // Can call specific parent implementation
        return FraudDetection.super.checkFraud(amount) && 
               AdvancedSecurity.super.checkFraud(amount);
    }
}

public class DefaultMethodDemo {
    public static void main(String[] args) {
        
        System.out.println("🎯 Example 1: Using Default Methods\n");
        
        PaymentProcessor_V2 stripe = new StripePayment();
        
        // Original method
        stripe.processPayment(100.0);
        
        // Default method (overridden)
        stripe.processRefund(50.0);
        
        // Default method (inherited)
        stripe.generateReceipt(100.0);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 2: Default Method with Logic\n");
        
        PaymentProcessor_V2 paypal = new PayPalPayment();
        
        // Using default refund (not overridden)
        paypal.processRefund(75.0);
        
        // Using default method with complex logic
        paypal.processFullTransaction(200.0);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 3: Retry Logic with Default Method\n");
        
        PaymentResult result = stripe.processPaymentWithRetry(6000.0, 3);
        System.out.println("Result: " + result);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 4: Static Method in Interface\n");
        
        double amount = 15000.0;
        if (PaymentProcessor_V2.validateAmount(amount)) {
            System.out.println("✅ Amount is valid");
        } else {
            System.out.println("❌ Amount exceeds limit");
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 5: Resolving Diamond Problem\n");
        
        SecurePayment securePayment = new SecurePayment();
        securePayment.processPayment(3000.0);
    }
}

class PaymentResult {
    private boolean success;
    private String message;
    
    public PaymentResult(boolean success, String message) {
        this.success = success;
        this.message = message;
    }
    
    @Override
    public String toString() {
        return (success ? "✅" : "❌") + " " + message;
    }
}
```

**🎯 Key Benefits:**
- ✅ **Backward Compatibility**: Old code doesn't break
- ✅ **Interface Evolution**: Add new methods without breaking changes
- ✅ **Code Reuse**: Share common implementations
- ✅ **Multiple Inheritance**: Inherit behavior from multiple interfaces

---

### 🗺️ Map Implementation Changes (Java 8)

### 💡 Explanation
Java 8 enhanced `HashMap` with:
- **Treeification** 🌳: Converts buckets to trees when collision count > 8 (improves worst-case from O(n) to O(log n))
- **New Methods**: `computeIfAbsent`, `merge`, `forEach`, `replaceAll`

### 🎯 Real-Time Example: User Session Cache

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class SessionCacheSystem {
    
    // Session storage
    private Map<String, UserSession> sessionCache = new HashMap<>();
    
    // Login attempts tracking
    private Map<String, Integer> loginAttempts = new HashMap<>();
    
    // User preferences with nested map
    private Map<String, Map<String, String>> userPreferences = new HashMap<>();
    
    
    // 🎯 COMPUTE IF ABSENT - Create session only if doesn't exist
    public UserSession getOrCreateSession(String userId) {
        return sessionCache.computeIfAbsent(userId, key -> {
            System.out.println("🆕 Creating new session for: " + userId);
            return new UserSession(userId, System.currentTimeMillis());
        });
    }
    
    // 🎯 MERGE - Increment login attempts
    public void recordLoginAttempt(String userId) {
        loginAttempts.merge(userId, 1, (oldValue, newValue) -> oldValue + newValue);
        System.out.println("🔐 Login attempts for " + userId + ": " + loginAttempts.get(userId));
    }
    
    // 🎯 COMPUTE IF PRESENT - Update existing session
    public void updateSessionActivity(String userId) {
        sessionCache.computeIfPresent(userId, (key, session) -> {
            session.setLastAccess(System.currentTimeMillis());
            System.out.println("🔄 Updated session for: " + userId);
            return session;
        });
    }
    
    // 🎯 COMPUTE - Complex logic (create or update)
    public void processUserAction(String userId, long timestamp) {
        sessionCache.compute(userId, (key, session) -> {
            if (session == null) {
                System.out.println("🆕 First action - creating session: " + userId);
                return new UserSession(userId, timestamp);
            } else if (timestamp - session.getLastAccess() > 30 * 60 * 1000) {
                System.out.println("⏰ Session expired - creating new session: " + userId);
                return new UserSession(userId, timestamp);
            } else {
                System.out.println("✅ Existing session - updating: " + userId);
                session.setLastAccess(timestamp);
                return session;
            }
        });
    }
    
    // 🎯 GET OR DEFAULT - Safe retrieval
    public int getLoginAttempts(String userId) {
        return loginAttempts.getOrDefault(userId, 0);
    }
    
    // 🎯 PUT IF ABSENT - Old vs New way
    public void setUserPreference(String userId, String key, String value) {
        // Old way (verbose)
        // Map<String, String> prefs = userPreferences.get(userId);
        // if (prefs == null) {
        //     prefs = new HashMap<>();
        //     userPreferences.put(userId, prefs);
        // }
        // prefs.put(key, value);
        
        // ✨ New way (clean)
        userPreferences
            .computeIfAbsent(userId, k -> new HashMap<>())
            .put(key, value);
        
        System.out.println("⚙️ Set preference for " + userId + ": " + key + "=" + value);
    }
    
    // 🎯 FOREACH - Iteration made easy
    public void printAllSessions() {
        System.out.println("\n📊 Active Sessions:");
        sessionCache.forEach((userId, session) -> 
            System.out.println("  👤 " + userId + " | Last access: " + 
                new Date(session.getLastAccess())));
    }
    
    // 🎯 REPLACE ALL - Bulk transformation
    public void extendAllSessions(long additionalTime) {
        sessionCache.replaceAll((userId, session) -> {
            session.setLastAccess(session.getLastAccess() + additionalTime);
            return session;
        });
        System.out.println("⏰ Extended all sessions by " + additionalTime + "ms");
    }
    
    // 🎯 REMOVE IF - Conditional removal
    public void cleanupExpiredSessions(long expiryThreshold) {
        int removed = 0;
        removed = (int) sessionCache.entrySet().removeIf(entry -> {
            boolean expired = System.currentTimeMillis() - entry.getValue().getLastAccess() 
                > expiryThreshold;
            if (expired) {
                System.out.println("🗑️ Removing expired session: " + entry.getKey());
            }
            return expired;
        });
        System.out.println("✅ Cleaned up " + removed + " expired sessions");
    }
    
    
    public static void main(String[] args) throws InterruptedException {
        SessionCacheSystem cache = new SessionCacheSystem();
        
        System.out.println("🚀 Session Cache System Demo\n");
        
        // 🎯 computeIfAbsent - Only creates if absent
        cache.getOrCreateSession("user123");
        cache.getOrCreateSession("user123"); // Won't create again
        cache.getOrCreateSession("user456");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 merge - Increment counter
        cache.recordLoginAttempt("user123");
        cache.recordLoginAttempt("user123");
        cache.recordLoginAttempt("user456");
        
        System.out.println("📊 Total login attempts for user123: " + 
            cache.getLoginAttempts("user123"));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 computeIfPresent - Update existing
        Thread.sleep(100);
        cache.updateSessionActivity("user123");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 compute - Complex logic
        cache.processUserAction("user789", System.currentTimeMillis());
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 Nested map operations
        cache.setUserPreference("user123", "theme", "dark");
        cache.setUserPreference("user123", "language", "en");
        cache.setUserPreference("user456", "theme", "light");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 forEach
        cache.printAllSessions();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 replaceAll
        cache.extendAllSessions(5000);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // 🎯 removeIf
        cache.cleanupExpiredSessions(1000);
    }
}

class UserSession {
    private String userId;
    private long createdAt;
    private long lastAccess;
    
    public UserSession(String userId, long timestamp) {
        this.userId = userId;
        this.createdAt = timestamp;
        this.lastAccess = timestamp;
    }
    
    public String getUserId() { return userId; }
    public long getCreatedAt() { return createdAt; }
    public long getLastAccess() { return lastAccess; }
    public void setLastAccess(long lastAccess) { this.lastAccess = lastAccess; }
}
```

---

### 🎁 Optional

### 💡 Explanation
Container object that **may or may not contain a value**. Eliminates `NullPointerException` and makes intent explicit. **Never use `Optional` as method parameter or field!**

### 🎯 Real-Time Example: User Profile Service

```java
import java.util.*;

public class UserProfileService {
    
    private Map<String, User> userDatabase = new HashMap<>();
    
    public UserProfileService() {
        // Sample data
        userDatabase.put("USR001", new User("USR001", "John Doe", "john@email.com", "Premium"));
        userDatabase.put("USR002", new User("USR002", "Jane Smith", "jane@email.com", null)); // No membership
        userDatabase.put("USR003", new User("USR003", "Bob Wilson", null, "Standard")); // No email
    }
    
    // ❌ OLD WAY - Prone to NullPointerException
    public User findUserOldWay(String userId) {
        return userDatabase.get(userId); // Returns null if not found
    }
    
    // ✅ NEW WAY - Using Optional
    public Optional<User> findUser(String userId) {
        return Optional.ofNullable(userDatabase.get(userId));
    }
    
    // 🎯 GET EMAIL - Handling nested nulls
    public String getUserEmailOldWay(String userId) {
        User user = userDatabase.get(userId);
        if (user != null) {
            String email = user.getEmail();
            if (email != null) {
                return email.toLowerCase();
            }
        }
        return "no-email@default.com";
    }
    
    // ✅ OPTIONAL WAY - Clean and safe
    public String getUserEmail(String userId) {
        return findUser(userId)
            .map(User::getEmail)              // Extract email (returns Optional<String>)
            .map(String::toLowerCase)         // Transform to lowercase
            .orElse("no-email@default.com");  // Default value
    }
    
    // 🎯 GET MEMBERSHIP WITH UPGRADE LOGIC
    public String getMembershipLevel(String userId) {
        return findUser(userId)
            .map(User::getMembership)
            .filter(membership -> !membership.equals("Standard")) // Only non-standard
            .orElse("Free");  // Default to Free if null or Standard
    }
    
    // 🎯 SEND NOTIFICATION - Only if user exists and has email
    public void sendNotification(String userId, String message) {
        findUser(userId)
            .map(User::getEmail)
            .filter(email -> email.contains("@"))  // Validate email
            .ifPresent(email -> 
                System.out.println("📧 Sent to " + email + ": " + message)
            );
    }
    
    // 🎯 GET DISCOUNT - Chain of fallbacks
    public double getDiscount(String userId) {
        return findUser(userId)
            .map(User::getMembership)
            .map(membership -> switch (membership) {
                case "Premium" -> 0.20;
                case "Standard" -> 0.10;
                default -> 0.05;
            })
            .orElse(0.0);  // No discount for non-existent users
    }
    
    // 🎯 OR ELSE GET - Expensive operation executed only if needed
    public User getUserOrCreateDefault(String userId) {
        return findUser(userId)
            .orElseGet(() -> {
                System.out.println("🔧 Creating default user (expensive operation)");
                return new User("DEFAULT", "Guest User", "guest@email.com", "Free");
            });
    }
    
    // 🎯 OR ELSE THROW - Explicit error handling
    public User getUserOrThrow(String userId) throws UserNotFoundException {
        return findUser(userId)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + userId));
    }
    
    // 🎯 FLAT MAP - Avoiding Optional<Optional<T>>
    public Optional<String> getPremiumUserEmail(String userId) {
        return findUser(userId)
            .filter(user -> "Premium".equals(user.getMembership()))
            .flatMap(user -> Optional.ofNullable(user.getEmail())); // Returns Optional<String>
    }
    
    // 🎯 OR - Fallback to another Optional (Java 9+)
    public Optional<User> findUserWithFallback(String primaryId, String secondaryId) {
        return findUser(primaryId)
            .or(() -> {
                System.out.println("🔄 Primary not found, trying secondary...");
                return findUser(secondaryId);
            });
    }
    
    
    public static void main(String[] args) {
        UserProfileService service = new UserProfileService();
        
        System.out.println("🎯 Example 1: Basic Optional Usage\n");
        
        Optional<User> user = service.findUser("USR001");
        
        // ✅ isPresent() + get()
        if (user.isPresent()) {
            System.out.println("Found: " + user.get().getName());
        }
        
        // ✅ Better: ifPresent()
        user.ifPresent(u -> System.out.println("✅ User: " + u.getName()));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 2: Handling Missing Values\n");
        
        // User doesn't exist
        String email1 = service.getUserEmail("USR999");
        System.out.println("Email for USR999: " + email1);
        
        // User exists but no email
        String email2 = service.getUserEmail("USR003");
        System.out.println("Email for USR003: " + email2);
        
        // User exists with email
        String email3 = service.getUserEmail("USR001");
        System.out.println("Email for USR001: " + email3);
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 3: Filter and Transform\n");
        
        System.out.println("USR001 Membership: " + service.getMembershipLevel("USR001"));
        System.out.println("USR002 Membership: " + service.getMembershipLevel("USR002"));
        System.out.println("USR003 Membership: " + service.getMembershipLevel("USR003"));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 4: ifPresent() - Side Effects\n");
        
        service.sendNotification("USR001", "Your order shipped!");
        service.sendNotification("USR003", "Your order shipped!"); // No email
        service.sendNotification("USR999", "Your order shipped!"); // User doesn't exist
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 5: Discounts Based on Membership\n");
        
        System.out.println("USR001 Discount: " + (service.getDiscount("USR001") * 100) + "%");
        System.out.println("USR002 Discount: " + (service.getDiscount("USR002") * 100) + "%");
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 6: orElseGet() - Lazy Evaluation\n");
        
        User user1 = service.getUserOrCreateDefault("USR001"); // Doesn't create
        System.out.println("Got user: " + user1.getName());
        
        User user2 = service.getUserOrCreateDefault("USR999"); // Creates default
        System.out.println("Got user: " + user2.getName());
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 7: orElseThrow() - Explicit Errors\n");
        
        try {
            User user3 = service.getUserOrThrow("USR999");
        } catch (UserNotFoundException e) {
            System.out.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 8: flatMap() - Nested Optionals\n");
        
        Optional<String> premiumEmail = service.getPremiumUserEmail("USR001");
        premiumEmail.ifPresentOrElse(
            email -> System.out.println("Premium user email: " + email),
            () -> System.out.println("Not a premium user or no email")
        );
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        
        System.out.println("🎯 Example 9: or() - Chaining Optionals\n");
        
        Optional<User> foundUser = service.findUserWithFallback("USR999", "USR001");
        foundUser.ifPresent(u -> System.out.println("✅ Found: " + u.getName()));
    }
}

class User {
    private String id;
    private String name;
    private String email;
    private String membership;
    
    public User(String id, String name, String email, String membership) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.membership = membership;
    }
    
    public String getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getMembership() { return membership; }
}

class UserNotFoundException extends Exception {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

**🎯 Optional Best Practices:**
- ✅ **DO**: Use as return type
- ✅ **DO**: Use `map()`, `flatMap()`, `filter()` for transformations
- ✅ **DO**: Use `orElse()`, `orElseGet()`, `orElseThrow()`
- ❌ **DON'T**: Use as method parameter
- ❌ **DON'T**: Use as class field
- ❌ **DON'T**: Call `get()` without checking `isPresent()`

---

## 📦 Collections

### 🗺️ Map

### 💡 Explanation
Key-Value pair storage. Common implementations:
- **HashMap** ⚡: Fast, unordered, allows null (O(1) average)
- **LinkedHashMap** 🔗: Maintains insertion order
- **TreeMap** 🌳: Sorted by keys (O(log n))
- **ConcurrentHashMap** 🔒: Thread-safe without full locking

### 🎯 Real-Time Example: Product Inventory System

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class InventoryManagementSystem {
    
    // Different Map types for different use cases
    private Map<String, Product> productsById = new HashMap<>();              // Fast lookup
    private Map<String, List<Product>> productsByCategory = new HashMap<>();  // Grouped data
    private LinkedHashMap<String, Integer> recentSearches = new LinkedHashMap<>(); // Order matters
    private Map<String, Integer> stock = new ConcurrentHashMap<>();          // Thread-safe
    
    // 🎯 ADD PRODUCTS
    public void addProduct(Product product) {
        // HashMap - O(1) insertion
        productsById.put(product.getId(), product);
        
        // Group by category
        productsByCategory
            .computeIfAbsent(product.getCategory(), k -> new ArrayList<>())
            .add(product);
        
        // Initialize stock
        stock.put(product.getId(), product.getInitialStock());
        
        System.out.println("✅ Added: " + product.getName());
    }
    
    // 🎯 SEARCH PRODUCT - Track recent searches
    public Optional<Product> searchProduct(String productId) {
        // LinkedHashMap - maintains order + updates on access
        recentSearches.merge(productId, 1, Integer::sum);
        
        // Keep only last 5 searches
        if (recentSearches.size() > 5) {
            Iterator<String> iterator = recentSearches.keySet().iterator();
            iterator.next();
            iterator.remove();
        }
        
        return Optional.ofNullable(productsById.get(productId));
    }
    
    // 🎯 GET PRODUCTS BY CATEGORY
    public List<Product> getProductsByCategory(String category) {
        return productsByCategory.getOrDefault(category, Collections.emptyList());
    }
    
    // 🎯 UPDATE STOCK - Thread-safe operation
    public boolean purchaseProduct(String productId, int quantity) {
        return stock.compute(productId, (key, currentStock) -> {
            if (currentStock == null || currentStock < quantity) {
                System.out.println("❌ Insufficient stock for: " + productId);
                return currentStock;
            }
            
            int newStock = currentStock - quantity;
            System.out.println("✅ Purchased " + quantity + " of " + productId + 
                ". Remaining: " + newStock);
            return newStock;
        }) != null;
    }
    
    // 🎯 GET LOW STOCK PRODUCTS
    public Map<String, Integer> getLowStockProducts(int threshold) {
        Map<String, Integer> lowStock = new HashMap<>();
        
        stock.forEach((productId, quantity) -> {
            if (quantity < threshold) {
                lowStock.put(productId, quantity);
            }
        });
        
        return lowStock;
    }
    
    // 🎯 INVENTORY VALUE CALCULATION
    public double calculateTotalValue() {
        return stock.entrySet().stream()
            .mapToDouble(entry -> {
                Product product = productsById.get(entry.getKey());
                return product.getPrice() * entry.getValue();
            })
            .sum();
    }
    
    // 🎯 RECENT SEARCHES REPORT
    public void printRecentSearches() {
        System.out.println("\n🔍 Recent Searches (in order):");
        recentSearches.forEach((productId, count) -> {
            productsById.get(productId);
            System.out.println("  " + productId + " - " + count + " times");
        });
    }
    
    // 🎯 INVENTORY REPORT
    public void printInventoryReport() {
        System.out.println("\n📊 Inventory Report:");
        System.out.println("Total Products: " + productsById.size());
        System.out.println("Total Categories: " + productsByCategory.size());
        System.out.println("Total Value: $" + String.format("%.2f", calculateTotalValue()));
        
        System.out.println("\n📦 Stock by Product:");
        stock.forEach((id, quantity) -> {
            Product product = productsById.get(id);
            System.out.println("  " + product.getName() + ": " + quantity + " units");
        });
    }
    
    
    public static void main(String[] args) {
        InventoryManagementSystem inventory = new InventoryManagementSystem();
        
        // Add products
        inventory.addProduct(new Product("P001", "Laptop", "Electronics", 999.99, 10));
        inventory.addProduct(new Product("P002", "Mouse", "Electronics", 29.99, 50));
        inventory.addProduct(new Product("P003", "Desk", "Furniture", 299.99, 5));
        inventory.addProduct(new Product("P004", "Chair", "Furniture", 199.99, 8));
        inventory.addProduct(new Product("P005", "Monitor", "Electronics", 399.99, 15));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Search products (building recent searches)
        inventory.searchProduct("P001");
        inventory.searchProduct("P002");
        inventory.searchProduct("P001"); // Searched again
        inventory.searchProduct("P003");
        inventory.searchProduct("P004");
        inventory.searchProduct("P005");
        inventory.searchProduct("P001"); // Searched third time
        
        inventory.printRecentSearches();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Get products by category
        System.out.println("🔍 Electronics Products:");
        List<Product> electronics = inventory.getProductsByCategory("Electronics");
        electronics.forEach(p -> System.out.println("  - " + p.getName()));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Purchase products
        inventory.purchaseProduct("P001", 3);
        inventory.purchaseProduct("P002", 10);
        inventory.purchaseProduct("P003", 10); // Should fail
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Low stock alert
        System.out.println("⚠️ Low Stock Products (< 10 units):");
        Map<String, Integer> lowStock = inventory.getLowStockProducts(10);
        lowStock.forEach((id, quantity) -> 
            System.out.println("  " + id + ": " + quantity + " units"));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Full report
        inventory.printInventoryReport();
    }
}

class Product {
    private String id;
    private String name;
    private String category;
    private double price;
    private int initialStock;
    
    public Product(String id, String name, String category, double price, int initialStock) {
        this.id = id;
        this.name = name;
        this.category = category;
        this.price = price;
        this.initialStock = initialStock;
    }
    
    public String getId() { return id; }
    public String getName() { return name; }
    public String getCategory() { return category; }
    public double getPrice() { return price; }
    public int getInitialStock() { return initialStock; }
}
```

---

### 🌳 TreeMap

### 💡 Explanation
**Sorted Map** implementation using Red-Black Tree. Keys always maintained in **sorted order** (natural or custom comparator). Operations: O(log n).

**Use Cases:**
- Range queries (subMap, headMap, tailMap)
- Sorted iteration
- Finding closest elements (floorKey, ceilingKey)

### 🎯 Real-Time Example: Hotel Booking System

```java
import java.time.LocalDate;
import java.util.*;

public class HotelBookingSystem {
    
    // TreeMap automatically sorts by date
    private TreeMap<LocalDate, List<Booking>> bookingsByDate = new TreeMap<>();
    
    // TreeMap for pricing (sorted by room number)
    private TreeMap<Integer, Double> roomPrices = new TreeMap<>();
    
    public HotelBookingSystem() {
        // Initialize room prices
        roomPrices.put(101, 150.0);
        roomPrices.put(102, 150.0);
        roomPrices.put(201, 200.0);
        roomPrices.put(202, 200.0);
        roomPrices.put(301, 300.0); // Premium room
        roomPrices.put(302, 300.0);
    }
    
    // 🎯 ADD BOOKING - Automatically sorted by date
    public void addBooking(Booking booking) {
        bookingsByDate
            .computeIfAbsent(booking.getCheckInDate(), k -> new ArrayList<>())
            .add(booking);
        
        System.out.println("✅ Booking added for room " + booking.getRoomNumber() + 
            " on " + booking.getCheckInDate());
    }
    
    // 🎯 GET BOOKINGS FOR DATE RANGE - Using TreeMap range operations
    public Map<LocalDate, List<Booking>> getBookingsInRange(LocalDate start, LocalDate end) {
        // subMap: returns view of portion of map
        return bookingsByDate.subMap(start, true, end, true);
    }
    
    // 🎯 GET UPCOMING BOOKINGS - Using tailMap
    public Map<LocalDate, List<Booking>> getUpcomingBookings() {
        LocalDate today = LocalDate.now();
        // tailMap: from today onwards
        return bookingsByDate.tailMap(today, true);
    }
    
    // 🎯 GET PAST BOOKINGS - Using headMap
    public Map<LocalDate, List<Booking>> getPastBookings() {
        LocalDate today = LocalDate.now();
        // headMap: before today
        return bookingsByDate.headMap(today, false);
    }
    
    // 🎯 FIND NEXT BOOKING DATE - Using ceilingKey
    public Optional<LocalDate> findNextBookingDate(LocalDate afterDate) {
        LocalDate nextDate = bookingsByDate.ceilingKey(afterDate);
        return Optional.ofNullable(nextDate);
    }
    
    // 🎯 FIND PREVIOUS BOOKING DATE - Using floorKey
    public Optional<LocalDate> findPreviousBookingDate(LocalDate beforeDate) {
        LocalDate prevDate = bookingsByDate.floorKey(beforeDate);
        return Optional.ofNullable(prevDate);
    }
    
    // 🎯 GET ROOMS IN PRICE RANGE - Using subMap on price TreeMap
    public Map<Integer, Double> getRoomsInPriceRange(double minPrice, double maxPrice) {
        Map<Integer, Double> roomsInRange = new TreeMap<>();
        
        roomPrices.forEach((roomNumber, price) -> {
            if (price >= minPrice && price <= maxPrice) {
                roomsInRange.put(roomNumber, price);
            }
        });
        
        return roomsInRange;
    }
    
    // 🎯 GET CHEAPEST AVAILABLE ROOM - Using firstEntry
    public Optional<Map.Entry<Integer, Double>> getCheapestRoom() {
        return Optional.ofNullable(roomPrices.firstEntry());
    }
    
    // 🎯 GET MOST EXPENSIVE ROOM - Using lastEntry
    public Optional<Map.Entry<Integer, Double>> getMostExpensiveRoom() {
        return Optional.ofNullable(roomPrices.lastEntry());
    }
    
    // 🎯 GET OCCUPANCY RATE FOR DATE RANGE
    public void printOccupancyReport(LocalDate start, LocalDate end) {
        System.out.println("\n📊 Occupancy Report: " + start + " to " + end);
        
        Map<LocalDate, List<Booking>> bookings = getBookingsInRange(start, end);
        
        if (bookings.isEmpty()) {
            System.out.println("No bookings in this range");
            return;
        }
        
        // TreeMap ensures dates are printed in order
        bookings.forEach((date, bookingList) -> {
            double occupancy = (bookingList.size() * 100.0) / roomPrices.size();
            System.out.println("  " + date + ": " + bookingList.size() + " rooms (" + 
                String.format("%.1f", occupancy) + "% occupancy)");
        });
    }
    
    // 🎯 NAVIGATION METHODS DEMO
    public void demonstrateNavigationMethods() {
        System.out.println("\n🧭 TreeMap Navigation Methods:");
        
        LocalDate today = LocalDate.now();
        
        System.out.println("First booking date: " + bookingsByDate.firstKey());
        System.out.println("Last booking date: " + bookingsByDate.lastKey());
        
        System.out.println("\nCheapest room: " + roomPrices.firstEntry());
        System.out.println("Most expensive room: " + roomPrices.lastEntry());
        
        System.out.println("\nRooms higher than 101: " + roomPrices.higherKey(101));
        System.out.println("Rooms lower than 301: " + roomPrices.lowerKey(301));
    }
    
    
    public static void main(String[] args) {
        HotelBookingSystem hotel = new HotelBookingSystem();
        
        // Create sample bookings
        LocalDate today = LocalDate.now();
        
        hotel.addBooking(new Booking(101, today.plusDays(1), "John Doe"));
        hotel.addBooking(new Booking(102, today.plusDays(1), "Jane Smith"));
        hotel.addBooking(new Booking(201, today.plusDays(2), "Bob Wilson"));
        hotel.addBooking(new Booking(301, today.plusDays(3), "Alice Brown"));
        hotel.addBooking(new Booking(101, today.plusDays(5), "Charlie Davis"));
        hotel.addBooking(new Booking(102, today.minusDays(1), "Past Guest")); // Past booking
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Get upcoming bookings (automatically sorted)
        System.out.println("📅 Upcoming Bookings:");
        hotel.getUpcomingBookings().forEach((date, bookings) -> {
            System.out.println("  " + date + ":");
            bookings.forEach(b -> System.out.println("    - Room " + b.getRoomNumber() + 
                ": " + b.getGuestName()));
        });
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Find rooms in price range
        System.out.println("💰 Rooms between $150-$250:");
        Map<Integer, Double> affordableRooms = hotel.getRoomsInPriceRange(150, 250);
        affordableRooms.forEach((room, price) -> 
            System.out.println("  Room " + room + ": $" + price));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Find next booking
        LocalDate searchDate = today.plusDays(2);
        hotel.findNextBookingDate(searchDate).ifPresent(date -> 
            System.out.println("🔍 Next booking after " + searchDate + ": " + date));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Occupancy report
        hotel.printOccupancyReport(today.plusDays(1), today.plusDays(5));
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Navigation methods
        hotel.demonstrateNavigationMethods();
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Cheapest and most expensive
        hotel.getCheapestRoom().ifPresent(entry -> 
            System.out.println("💵 Cheapest room: " + entry.getKey() + " - $" + entry.getValue()));
        
        hotel.getMostExpensiveRoom().ifPresent(entry -> 
            System.out.println("💎 Most expensive room: " + entry.getKey() + " - $" + entry.getValue()));
    }
}

class Booking {
    private int roomNumber;
    private LocalDate checkInDate;
    private String guestName;
    
    public Booking(int roomNumber, LocalDate checkInDate, String guestName) {
        this.roomNumber = roomNumber;
        this.checkInDate = checkInDate;
        this.guestName = guestName;
    }
    
    public int getRoomNumber() { return roomNumber; }
    public LocalDate getCheckInDate() { return checkInDate; }
    public String getGuestName() { return guestName; }
}
```

**🎯 TreeMap Key Features:**
- ✅ **Sorted Order**: Always maintains key order
- ✅ **Range Operations**: `subMap()`, `headMap()`, `tailMap()`
- ✅ **Navigation**: `floorKey()`, `ceilingKey()`, `lowerKey()`, `higherKey()`
- ✅ **First/Last**: `firstKey()`, `lastKey()`, `firstEntry()`, `lastEntry()`
- ⚠️ **Performance**: O(log n) vs HashMap's O(1)

---

## 🧵 Multi-Threading

### 💡 Explanation
Running multiple threads **concurrently** to utilize CPU efficiently. Java provides multiple ways:
- **Thread class** extension
- **Runnable interface** implementation
- **ExecutorService** framework (preferred)
- **CompletableFuture** for async programming

**Key Concepts:**
- 🔒 **Synchronization**: Prevent race conditions
- 🎯 **Thread States**: New, Runnable, Running, Blocked, Waiting, Terminated
- ⚡ **Thread Pool**: Reuse threads instead of creating new ones

### 🎯 Real-Time Example: Order Processing System

```java
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class OrderProcessingSystem {
    
    // Thread-safe counter
    private AtomicInteger processedOrders = new AtomicInteger(0);
    
    // Thread-safe collection
    private ConcurrentHashMap<String, OrderStatus> orderStatusMap = new ConcurrentHashMap<>();
    
    // Thread pool for processing orders
    private ExecutorService orderProcessingPool = Executors.newFixedThreadPool(5);
    
    // Thread pool for sending notifications
    private ExecutorService notificationPool = Executors.newFixedThreadPool(3);
    
    
    // 🎯 PROCESS ORDER - Runnable approach
    class OrderProcessor implements Runnable {
        private Order order;
        
        public OrderProcessor(Order order) {
            this.order = order;
        }
        
        @Override
        public void run() {
            try {
                String threadName = Thread.currentThread().getName();
                System.out.println("🔄 [" + threadName + "] Processing order: " + order.getId());
                
                // Update status
                orderStatusMap.put(order.getId(), OrderStatus.PROCESSING);
                
                // Simulate payment processing (1-3 seconds)
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 3000));
                
                boolean paymentSuccess = processPayment(order);
                
                if (paymentSuccess) {
                    orderStatusMap.put(order.getId(), OrderStatus.CONFIRMED);
                    processedOrders.incrementAndGet();
                    System.out.println("✅ [" + threadName + "] Order confirmed: " + order.getId());
                    
                    // Send notification asynchronously
                    sendNotificationAsync(order);
                } else {
                    orderStatusMap.put(order.getId(), OrderStatus.FAILED);
                    System.out.println("❌ [" + threadName + "] Order failed: " + order.getId());
                }
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("⚠️ Order processing interrupted: " + order.getId());
            }
        }
    }
    
    // 🎯 PROCESS PAYMENT - Synchronized method
    private synchronized boolean processPayment(Order order) {
        // Simulate payment gateway call
        return order.getAmount() <= 10000; // Fail for large amounts
    }
    
    // 🎯 SEND NOTIFICATION - Callable approach (returns result)
    private void sendNotificationAsync(Order order) {
        Callable<Boolean> notificationTask = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("📧 [" + threadName + "] Sending notification for: " + order.getId());
            
            Thread.sleep(500); // Simulate email sending
            
            System.out.println("✉️ [" + threadName + "] Notification sent to: " + 
                order.getCustomerEmail());
            return true;
        };
        
        notificationPool.submit(notificationTask);
    }
    
    // 🎯 SUBMIT ORDER - Using ExecutorService
    public Future<OrderStatus> submitOrder(Order order) {
        orderStatusMap.put(order.getId(), OrderStatus.PENDING);
        
        // Submit to thread pool and return Future
        return orderProcessingPool.submit(() -> {
            new OrderProcessor(order).run();
            return orderStatusMap.get(order.getId());
        });
    }
    
    // 🎯 PROCESS BATCH ORDERS - Using CompletableFuture
    public CompletableFuture<BatchResult> processBatchAsync(List<Order> orders) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("\n🚀 Starting batch processing of " + orders.size() + " orders");
            
            List<Future<OrderStatus>> futures = new ArrayList<>();
            
            // Submit all orders
            for (Order order : orders) {
                futures.add(submitOrder(order));
            }
            
            // Wait for all to complete
            int successful = 0;
            int failed = 0;
            
            for (Future<OrderStatus> future : futures) {
                try {
                    OrderStatus status = future.get(5, TimeUnit.SECONDS);
                    if (status == OrderStatus.CONFIRMED) {
                        successful++;
                    } else {
                        failed++;
                    }
                } catch (Exception e) {
                    failed++;
                }
            }
            
            return new BatchResult(successful, failed);
        }, orderProcessingPool);
    }
    
    // 🎯 GET ORDER STATUS - Non-blocking
    public OrderStatus getOrderStatus(String orderId) {
        return orderStatusMap.getOrDefault(orderId, OrderStatus.NOT_FOUND);
    }
    
    // 🎯 WAIT FOR ALL ORDERS - CountDownLatch example
    public void processOrdersWithLatch(List<Order> orders) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(orders.size());
        
        System.out.println("\n⏳ Processing " + orders.size() + " orders with CountDownLatch");
        
        for (Order order : orders) {
            orderProcessingPool.submit(() -> {
                try {
                    new OrderProcessor(order).run();
                } finally {
                    latch.countDown(); // Decrease counter
                }
            });
        }
        
        // Wait for all orders to complete
        latch.await();
        System.out.println("✅ All orders processed!");
    }
    
    // 🎯 SHUTDOWN
    public void shutdown() {
        System.out.println("\n🛑 Shutting down order processing system...");
        
        orderProcessingPool.shutdown();
        notificationPool.shutdown();
        
        try {
            if (!orderProcessingPool.awaitTermination(5, TimeUnit.SECONDS)) {
                orderProcessingPool.shutdownNow();
            }
            if (!notificationPool.awaitTermination(5, TimeUnit.SECONDS)) {
                notificationPool.shutdownNow();
            }
        } catch (InterruptedException e) {
            orderProcessingPool.shutdownNow();
            notificationPool.shutdownNow();
        }
        
        System.out.println("📊 Total orders processed: " + processedOrders.get());
    }
    
    
    public static void main(String[] args) throws Exception {
        OrderProcessingSystem system = new OrderProcessingSystem();
        
        // Create sample orders
        List<Order> orders = Arrays.asList(
            new Order("ORD001", 150.0, "customer1@email.com"),
            new Order("ORD002", 250.0, "customer2@email.com"),
            new Order("ORD003", 15000.0, "customer3@email.com"), // Will fail
            new Order("ORD004", 75.0, "customer4@email.com"),
            new Order("ORD005", 500.0, "customer5@email.com"),
            new Order("ORD006", 120.0, "customer6@email.com"),
            new Order("ORD007", 890.0, "customer7@email.com"),
            new Order("ORD008", 12000.0, "customer8@email.com")  // Will fail
        );
        
        System.out.println("🎯 Example 1: Individual Order Submission\n");
        
        Future<OrderStatus> future1 = system.submitOrder(orders.get(0));
        Future<OrderStatus> future2 = system.submitOrder(orders.get(1));
        
        // Do other work while orders are processing
        System.out.println("💼 Doing other work while orders are processing...\n");
        Thread.sleep(1000);
        
        // Check results
        System.out.println("Order 1 status: " + future1.get());
        System.out.println("Order 2 status: " + future2.get());
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        
        System.out.println("🎯 Example 2: Batch Processing with CompletableFuture\n");
        
        CompletableFuture<BatchResult> batchFuture = system.processBatchAsync(orders);
        
        // Attach callback when batch completes
        batchFuture.thenAccept(result -> {
            System.out.println("\n📊 Batch Processing Complete!");
            System.out.println("   ✅ Successful: " + result.getSuccessful());
            System.out.println("   ❌ Failed: " + result.getFailed());
        });
        
        // Wait for batch to complete
        BatchResult result = batchFuture.get();
        
        Thread.sleep(2000); // Wait for notifications to complete
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        
        System.out.println("🎯 Example 3: CountDownLatch\n");
        
        List<Order> newOrders = Arrays.asList(
            new Order("ORD009", 200.0, "customer9@email.com"),
            new Order("ORD010", 300.0, "customer10@email.com"),
            new Order("ORD011", 400.0, "customer11@email.com")
        );
        
        system.processOrdersWithLatch(newOrders);
        
        Thread.sleep(1000);
        
        // Shutdown
        system.shutdown();
    }
}

class Order {
    private String id;
    private double amount;
    private String customerEmail;
    
    public Order(String id, double amount, String customerEmail) {
        this.id = id;
        this.amount = amount;
        this.customerEmail = customerEmail;
    }
    
    public String getId() { return id; }
    public double getAmount() { return amount; }
    public String getCustomerEmail() { return customerEmail; }
}

enum OrderStatus {
    PENDING, PROCESSING, CONFIRMED, FAILED, NOT_FOUND
}

class BatchResult {
    private int successful;
    private int failed;
    
    public BatchResult(int successful, int failed) {
        this.successful = successful;
        this.failed = failed;
    }
    
    public int getSuccessful() { return successful; }
    public int getFailed() { return failed; }
}
```

**🎯 Thread Safety Best Practices:**
- ✅ Use **AtomicInteger/AtomicReference** for counters
- ✅ Use **ConcurrentHashMap** instead of synchronized HashMap
- ✅ Use **ExecutorService** instead of creating threads manually
- ✅ Always **shutdown** thread pools
- ✅ Use **synchronized** blocks for critical sections
- ✅ Prefer **CompletableFuture** for async operations

---

## 🔤 String

### 💡 Explanation
Strings are **immutable** in Java - once created, cannot be changed. This enables:
- **Thread Safety** 🔒: Can be shared across threads
- **String Pool** 💾: Reuses identical strings to save memory
- **Security** 🛡️: Prevents modification of sensitive data

**Key Classes:**
- `String` - Immutable
- `StringBuilder` - Mutable, not thread-safe (faster)
- `StringBuffer` - Mutable, thread-safe (slower)

### 🎯 Real-Time Example: Log Processing System

```java
public class LogProcessingSystem {
    
    // 🎯 STRING POOL DEMONSTRATION
    public void demonstrateStringPool() {
        System.out.println("🎯 String Pool Behavior:\n");
        
        // Literal - goes to String Pool
        String s1 = "Hello";
        String s2 = "Hello";
        
        // new keyword - creates new object in heap
        String s3 = new String("Hello");
        String s4 = new String("Hello");
        
        System.out.println("s1 == s2 (both literals): " + (s1 == s2)); // true - same pool object
        System.out.println("s1 == s3 (literal vs new): " + (s1 == s3)); // false - different objects
        System.out.println("s3 == s4 (both new): " + (s3 == s4));       // false - different objects
        System.out.println("s1.equals(s3) (content): " + s1.equals(s3)); // true - same content
        
        // intern() - move to pool
        String s5 = s3.intern(); // Returns pool reference
        System.out.println("s1 == s5 (after intern): " + (s1 == s5)); // true - now from pool
    }
    
    // 🎯 STRING IMMUTABILITY
    public void demonstrateImmutability() {
        System.out.println("\n🎯 String Immutability:\n");
        
        String original = "Error";
        System.out.println("Original string: " + original);
        System.out.println("Original hashCode: " + System.identityHashCode(original));
        
        // This creates a NEW string, doesn't modify original
        String modified = original.concat(" occurred");
        
        System.out.println("\nAfter concat:");
        System.out.println("Original: " + original); // Still "Error"
        System.out.println("Original hashCode: " + System.identityHashCode(original)); // Same
        System.out.println("Modified: " + modified); // "Error occurred"
        System.out.println("Modified hashCode: " + System.identityHashCode(modified)); // Different
    }
    
    // ❌ INEFFICIENT - Creates many intermediate String objects
    public String parseLogInefficient(List<String> logLines) {
        String result = "";
        
        for (String line : logLines) {
            result += line + "\n"; // Creates new String object each time!
        }
        
        return result;
    }
    
    // ✅ EFFICIENT - Uses StringBuilder
    public String parseLogEfficient(List<String> logLines) {
        StringBuilder result = new StringBuilder(logLines.size() * 100); // Initial capacity
        
        for (String line : logLines) {
            result.append(line).append("\n"); // Modifies same object
        }
        
        return result.toString();
    }
    
    // 🎯 REAL-TIME: Parse and filter logs
    public List<LogEntry> parseAndFilterLogs(String logData, String levelFilter) {
        List<LogEntry> entries = new ArrayList<>();
        
        String[] lines = logData.split("\n");
        
        for (String line : lines) {
            if (line.trim().isEmpty()) continue;
            
            // Parse log format: [LEVEL] timestamp - message
            int levelEnd = line.indexOf(']');
            if (levelEnd == -1) continue;
            
            String level = line.substring(line.indexOf('[') + 1, levelEnd);
            
            if (levelFilter != null && !level.equalsIgnoreCase(levelFilter)) {
                continue;
            }
            
            int dashIndex = line.indexOf(" - ", levelEnd);
            if (dashIndex == -1) continue;
            
            String timestamp = line.substring(levelEnd + 2, dashIndex).trim();
            String message = line.substring(dashIndex + 3);
            
            entries.add(new LogEntry(level, timestamp, message));
        }
        
        return entries;
    }
    
    // 🎯 REAL-TIME: Sanitize sensitive data
    public String sanitizeSensitiveData(String log) {
        // Use StringBuilder for multiple replacements
        StringBuilder sanitized = new StringBuilder(log);
        
        // Replace credit card (16 digits)
        String ccPattern = "\\d{16}";
        String sanitizedLog = sanitized.toString().replaceAll(ccPattern, "****-****-****-****");
        
        // Replace email
        sanitizedLog = sanitizedLog.replaceAll("\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b", 
            "***@***.com");
        
        // Replace API keys (assuming format: key_xxxxxxxxxx)
        sanitizedLog = sanitizedLog.replaceAll("key_[a-zA-Z0-9]{20}", "key_********************");
        
        return sanitizedLog;
    }
    
    // 🎯 REAL-TIME: Generate log summary
    public String generateLogSummary(List<LogEntry> entries) {
        StringBuilder summary = new StringBuilder();
        
        Map<String, Long> countByLevel = entries.stream()
            .collect(java.util.stream.Collectors.groupingBy(
                LogEntry::getLevel,
                java.util.stream.Collectors.counting()
            ));
        
        summary.append("📊 Log Summary\n");
        summary.append("=".repeat(40)).append("\n");
        summary.append(String.format("Total Entries: %d\n", entries.size()));
        summary.append("\nBreakdown by Level:\n");
        
        countByLevel.forEach((level, count) -> 
            summary.append(String.format("  %s: %d entries\n", level, count))
        );
        
        return summary.toString();
    }
    
    // 🎯 PERFORMANCE COMPARISON
    public void comparePerformance() {
        System.out.println("\n🎯 Performance Comparison:\n");
        
        List<String> logLines = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            logLines.add("[INFO] Log entry number " + i);
        }
        
        // String concatenation
        long start1 = System.currentTimeMillis();
        parseLogInefficient(logLines);
        long end1 = System.currentTimeMillis();
        
        // StringBuilder
        long start2 = System.currentTimeMillis();
        parseLogEfficient(logLines);
        long end2 = System.currentTimeMillis();
        
        System.out.println("❌ String concatenation: " + (end1 - start1) + "ms");
        System.out.println("✅ StringBuilder: " + (end2 - start2) + "ms");
        System.out.println("⚡ Speedup: " + ((end1 - start1) / (double)(end2 - start2)) + "x faster");
    }
    
    // 🎯 STRING METHODS IN ACTION
    public void demonstrateStringMethods() {
        System.out.println("\n🎯 Useful String Methods:\n");
        
        String log = "  [ERROR] 2024-01-15 - Database connection failed  ";
        
        System.out.println("Original: '" + log + "'");
        System.out.println("trim(): '" + log.trim() + "'");
        System.out.println("startsWith('[ERROR]'): " + log.trim().startsWith("[ERROR]"));
        System.out.println("contains('Database'): " + log.contains("Database"));
        System.out.println("replace('ERROR', 'WARNING'): " + log.replace("ERROR", "WARNING"));
        System.out.println("substring(2, 9): '" + log.trim().substring(2, 9) + "'");
        System.out.println("toLowerCase(): " + log.toLowerCase());
        System.out.println("split(' - ').length: " + log.split(" - ").length);
        
        // String formatting
        String formatted = String.format("Log Level: %s, Date: %s", "ERROR", "2024-01-15");
        System.out.println("\nFormatted: " + formatted);
        
        // Text blocks (Java 15+)
        String multiline = """
            {
              "level": "ERROR",
              "message": "Connection failed"
            }
            """;
        System.out.println("\nText Block:\n" + multiline);
    }
    
    
    public static void main(String[] args) {
        LogProcessingSystem system = new LogProcessingSystem();
        
        // Demo 1: String Pool
        system.demonstrateStringPool();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Demo 2: Immutability
        system.demonstrateImmutability();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Demo 3: Real log parsing
        String logData = """
            [INFO] 2024-01-15 10:30:00 - Application started
            [ERROR] 2024-01-15 10:31:15 - Database connection failed
            [WARN] 2024-01-15 10:31:20 - Retrying connection...
            [INFO] 2024-01-15 10:31:25 - Connection established
            [ERROR] 2024-01-15 10:32:00 - API timeout
            [INFO] 2024-01-15 10:33:00 - Request processed successfully
            """;
        
        System.out.println("🎯 Log Parsing:\n");
        List<LogEntry> allLogs = system.parseAndFilterLogs(logData, null);
        System.out.println("Total logs: " + allLogs.size());
        
        List<LogEntry> errorLogs = system.parseAndFilterLogs(logData, "ERROR");
        System.out.println("Error logs: " + errorLogs.size());
        errorLogs.forEach(entry -> System.out.println("  - " + entry.getMessage()));
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Demo 4: Sanitization
        System.out.println("🎯 Data Sanitization:\n");
        String sensitiveLog = "User 1234567812345678 with email john@example.com used key_abc123def456ghi789jk";
        System.out.println("Original: " + sensitiveLog);
        System.out.println("Sanitized: " + system.sanitizeSensitiveData(sensitiveLog));
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Demo 5: Summary
        System.out.println(system.generateLogSummary(allLogs));
        
        System.out.println("=".repeat(60) + "\n");
        
        // Demo 6: Performance
        system.comparePerformance();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Demo 7: String methods
        system.demonstrateStringMethods();
    }
}

class LogEntry {
    private String level;
    private String timestamp;
    private String message;
    
    public LogEntry(String level, String timestamp, String message) {
        this.level = level;
        this.timestamp = timestamp;
        this.message = message;
    }
    
    public String getLevel() { return level; }
    public String getTimestamp() { return timestamp; }
    public String getMessage() { return message; }
}
```

**🎯 String Best Practices:**
- ✅ Use **String literals** for constants (goes to pool)
- ✅ Use **StringBuilder** for string concatenation in loops
- ✅ Use **StringBuffer** only when thread-safety is needed
- ✅ Use `equals()` for content comparison, `==` for reference
- ✅ Consider **Text Blocks** (Java 15+) for multiline strings
- ❌ Avoid `new String()` unless absolutely necessary

---

## ⚠️ Exceptions

### 💡 Explanation
Exceptions handle **abnormal conditions** during program execution. Java has hierarchy:

```
Throwable
├── Error (JVM errors - don't catch)
└── Exception
    ├── RuntimeException (Unchecked - optional handling)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── IOException, SQLException, etc. (Checked - must handle)
```

**Types:**
- **Checked** ✅: Compiler forces handling (IOException, SQLException)
- **Unchecked** ⚡: Runtime exceptions (NullPointerException, IllegalArgumentException)

### 🎯 Real-Time Example: Banking Transaction System

```java
import java.io.*;
import java.sql.*;
import java.time.LocalDateTime;
import java.util.*;

// ��� CUSTOM EXCEPTIONS - Business logic exceptions
class InsufficientFundsException extends Exception {
    private double balance;
    private double withdrawAmount;
    
    public InsufficientFundsException(double balance, double withdrawAmount) {
        super(String.format("Insufficient funds. Balance: $%.2f, Withdrawal: $%.2f", 
            balance, withdrawAmount));
        this.balance = balance;
        this.withdrawAmount = withdrawAmount;
    }
    
    public double getShortfall() {
        return withdrawAmount - balance;
    }
}

class AccountLockedException extends Exception {
    public AccountLockedException(String accountNumber) {
        super("Account is locked: " + accountNumber);
    }
}

class InvalidTransactionException extends RuntimeException {
    public InvalidTransactionException(String message) {
        super(message);
    }
}

public class BankingSystem {
    
    private Map<String, Account> accounts = new HashMap<>();
    private List<String> transactionLog = new ArrayList<>();
    
    public BankingSystem() {
        // Initialize sample accounts
        accounts.put("ACC001", new Account("ACC001", 1000.0, false));
        accounts.put("ACC002", new Account("ACC002", 500.0, false));
        accounts.put("ACC003", new Account("ACC003", 2000.0, true)); // Locked
    }
    
    // 🎯 CHECKED EXCEPTION - Must be declared or handled
    public void withdraw(String accountNumber, double amount) 
            throws InsufficientFundsException, AccountLockedException {
        
        Account account = accounts.get(accountNumber);
        
        if (account == null) {
            // Unchecked exception - validation error
            throw new InvalidTransactionException("Account not found: " + accountNumber);
        }
        
        if (account.isLocked()) {
            // Checked exception - business rule violation
            throw new AccountLockedException(accountNumber);
        }
        
        if (amount <= 0) {
            // Unchecked exception - invalid input
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        
        if (account.getBalance() < amount) {
            // Checked exception - business logic
            throw new InsufficientFundsException(account.getBalance(), amount);
        }
        
        account.setBalance(account.getBalance() - amount);
        logTransaction("WITHDRAW", accountNumber, amount, true);
        System.out.println("✅ Withdrawn $" + amount + " from " + accountNumber);
    }
    
    // 🎯 TRY-CATCH-FINALLY - Resource management
    public void exportTransactionLog(String filename) {
        BufferedWriter writer = null;
        
        try {
            writer = new BufferedWriter(new FileWriter(filename));
            
            System.out.println("📝 Exporting transaction log to: " + filename);
            
            for (String transaction : transactionLog) {
                writer.write(transaction);
                writer.newLine();
            }
            
            System.out.println("✅ Export completed successfully");
            
        } catch (IOException e) {
            System.err.println("❌ Failed to export log: " + e.getMessage());
            
        } finally {
            // Always executes - cleanup resources
            if (writer != null) {
                try {
                    writer.close();
                    System.out.println("🔒 File closed");
                } catch (IOException e) {
                    System.err.println("⚠️ Error closing file: " + e.getMessage());
                }
            }
        }
    }
    
    // 🎯 TRY-WITH-RESOURCES - Automatic resource management (Java 7+)
    public void exportTransactionLogModern(String filename) {
        // Resources automatically closed
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            
            System.out.println("📝 Exporting transaction log (modern way)");
            
            for (String transaction : transactionLog) {
                writer.write(transaction);
                writer.newLine();
            }
            
            System.out.println("✅ Export completed");
            
        } catch (IOException e) {
            System.err.println("❌ Export failed: " + e.getMessage());
        }
        // No finally needed - writer.close() called automatically
    }
    
    // 🎯 MULTI-CATCH (Java 7+)
    public void processTransaction(String type, String accountNumber, double amount) {
        try {
            if ("WITHDRAW".equals(type)) {
                withdraw(accountNumber, amount);
            } else if ("TRANSFER".equals(type)) {
                // Simulate transfer
                throw new SQLException("Database connection failed");
            }
            
        } catch (InsufficientFundsException | AccountLockedException e) {
            // Handle multiple exceptions the same way
            System.err.println("🚫 Transaction failed: " + e.getMessage());
            logTransaction(type, accountNumber, amount, false);
            
        } catch (SQLException e) {
            System.err.println("💾 Database error: " + e.getMessage());
            // Retry logic here
            
        } catch (Exception e) {
            // Catch-all for unexpected errors
            System.err.println("⚠️ Unexpected error: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    // 🎯 RETHROWING with additional context
    public void performCriticalTransaction(String accountNumber, double amount) throws Exception {
        try {
            withdraw(accountNumber, amount);
            
        } catch (InsufficientFundsException e) {
            // Add context and rethrow
            System.err.println("Critical transaction failed!");
            System.err.println("Shortfall: $" + e.getShortfall());
            throw new Exception("Critical transaction failure for " + accountNumber, e);
            
        } catch (AccountLockedException e) {
            // Log and rethrow
            System.err.println("Cannot perform critical transaction on locked account");
            throw e;
        }
    }
    
    // 🎯 EXCEPTION CHAINING
    public void validateAndProcess(String accountNumber, double amount) {
        try {
            validateAccount(accountNumber);
            withdraw(accountNumber, amount);
            
        } catch (AccountLockedException e) {
            // Wrap checked exception in unchecked
            throw new InvalidTransactionException("Validation failed: " + e.getMessage());
        } catch (InsufficientFundsException e) {
            // Chain exceptions to preserve stack trace
            throw new InvalidTransactionException("Processing failed: " + e.getMessage());
        }
    }
    
    private void validateAccount(String accountNumber) throws AccountLockedException {
        Account account = accounts.get(accountNumber);
        if (account != null && account.isLocked()) {
            throw new AccountLockedException(accountNumber);
        }
    }
    
    // 🎯 ASSERT - Development-time checks (enable with -ea flag)
    public void deposit(String accountNumber, double amount) {
        assert amount > 0 : "Amount must be positive";
        assert accountNumber != null : "Account number cannot be null";
        
        Account account = accounts.get(accountNumber);
        if (account != null) {
            account.setBalance(account.getBalance() + amount);
            logTransaction("DEPOSIT", accountNumber, amount, true);
            System.out.println("✅ Deposited $" + amount + " to " + accountNumber);
        }
    }
    
    private void logTransaction(String type, String accountNumber, double amount, boolean success) {
        String log = String.format("%s | %s | %s | $%.2f | %s",
            LocalDateTime.now(),
            type,
            accountNumber,
            amount,
            success ? "SUCCESS" : "FAILED"
        );
        transactionLog.add(log);
    }
    
    
    public static void main(String[] args) {
        BankingSystem bank = new BankingSystem();
        
        System.out.println("🏦 Banking System - Exception Handling Demo\n");
        System.out.println("=".repeat(60) + "\n");
        
        // 🎯 Example 1: Successful withdrawal
        System.out.println("🎯 Example 1: Successful Withdrawal\n");
        try {
            bank.withdraw("ACC001", 100.0);
        } catch (InsufficientFundsException | AccountLockedException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 2: Insufficient funds
        System.out.println("🎯 Example 2: Insufficient Funds\n");
        try {
            bank.withdraw("ACC002", 1000.0);
        } catch (InsufficientFundsException e) {
            System.err.println("❌ " + e.getMessage());
            System.err.println("💡 Shortfall: $" + String.format("%.2f", e.getShortfall()));
        } catch (AccountLockedException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 3: Locked account
        System.out.println("🎯 Example 3: Locked Account\n");
        try {
            bank.withdraw("ACC003", 100.0);
        } catch (InsufficientFundsException | AccountLockedException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 4: Invalid account (unchecked exception)
        System.out.println("🎯 Example 4: Invalid Account (Unchecked Exception)\n");
        try {
            bank.withdraw("ACC999", 100.0);
        } catch (InvalidTransactionException e) {
            System.err.println("❌ Runtime error: " + e.getMessage());
        } catch (InsufficientFundsException | AccountLockedException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 5: Invalid amount (unchecked exception)
        System.out.println("🎯 Example 5: Invalid Amount (IllegalArgumentException)\n");
        try {
            bank.withdraw("ACC001", -50.0);
        } catch (IllegalArgumentException e) {
            System.err.println("❌ Validation error: " + e.getMessage());
        } catch (InsufficientFundsException | AccountLockedException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 6: Multi-catch
        System.out.println("🎯 Example 6: Multi-Catch\n");
        bank.processTransaction("WITHDRAW", "ACC002", 1000.0);
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 7: Try-with-resources
        System.out.println("🎯 Example 7: Exporting Log (Try-With-Resources)\n");
        bank.deposit("ACC001", 500.0);
        bank.exportTransactionLogModern("transactions.log");
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 8: Exception chaining
        System.out.println("🎯 Example 8: Exception Chaining\n");
        try {
            bank.validateAndProcess("ACC003", 100.0);
        } catch (InvalidTransactionException e) {
            System.err.println("❌ " + e.getMessage());
        }
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // 🎯 Example 9: Critical transaction with rethrow
        System.out.println("🎯 Example 9: Critical Transaction\n");
        try {
            bank.performCriticalTransaction("ACC002", 1000.0);
        } catch (Exception e) {
            System.err.println("❌ Critical failure: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("   Caused by: " + e.getCause().getMessage());
            }
        }
    }
}

class Account {
    private String accountNumber;
    private double balance;
    private boolean locked;
    
    public Account(String accountNumber, double balance, boolean locked) {
        this.accountNumber = accountNumber;
        this.balance = balance;
        this.locked = locked;
    }
    
    public String getAccountNumber() { return accountNumber; }
    public double getBalance() { return balance; }
    public boolean isLocked() { return locked; }
    public void setBalance(double balance) { this.balance = balance; }
}
```

**🎯 Exception Best Practices:**
- ✅ Use **specific exceptions** (not generic Exception)
- ✅ **Create custom exceptions** for business logic
- ✅ **Try-with-resources** for automatic cleanup
- ✅ **Log exceptions** with proper context
- ✅ **Handle or declare** checked exceptions
- ✅ **Chain exceptions** to preserve stack trace
- ❌ Never catch `Throwable` or `Error`
- ❌ Don't use exceptions for control flow
- ❌ Don't swallow exceptions (empty catch blocks)

---

Due to character limits, I'll continue with the rest of the topics in the next response. Would you like me to continue with:

1. ⚠️ Exceptions for Collection Frameworks
2. 🧪 Unit Testing and JUnit 5
3. 🌐 REST API Development
4. 🍃 Spring Framework
5. 🎨 Design Thinking & Principles

Let me know and I'll provide the same detailed format! 🚀
