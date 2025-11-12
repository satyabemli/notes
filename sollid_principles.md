# 🛒 Liskov Substitution Principle - E-Commerce Real-Time Examples

## 📚 Table of Contents
1. [LSP Recap](#-lsp-quick-recap)
2. [E-Commerce Scenario 1: Product Discount System](#-scenario-1-product-discount-system)
3. [E-Commerce Scenario 2: Payment Processing](#-scenario-2-payment-processing)
4. [E-Commerce Scenario 3: Shipping Methods](#-scenario-3-shipping-methods)
5. [E-Commerce Scenario 4: Order Types](#-scenario-4-order-types)
6. [E-Commerce Scenario 5: User Account Types](#-scenario-5-user-account-types)
7. [Interview Questions](#-interview-questions)

---

## 🎯 LSP Quick Recap

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
