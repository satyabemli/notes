

# 1️⃣ What is Spring Framework? 🌱

**Simple line:**  
👉 Spring is a Java framework that helps you build flexible, testable, and loosely-coupled applications.

**Details:**

- It manages **objects (beans)** for you instead of you creating them manually.
- Uses **Inversion of Control (IoC)** and **Dependency Injection (DI)**.
- Provides support for:
  - Web (Spring MVC)
  - Data access (Spring Data, JDBC)
  - Security (Spring Security)
  - AOP, Transactions, etc.

---

# 2️⃣ What is Spring Boot & how is it different from Spring? 🚀

**Simple line:**  
👉 Spring Boot is a way to build Spring apps quickly with **almost zero configuration** and an **embedded server**.

**Difference:**

- **Spring Framework**: Core infrastructure + lots of configuration (XML/Java config).
- **Spring Boot**: Opinionated, auto-configured Spring with:
  - Embedded server (Tomcat/Jetty/Undertow)
  - Starters (predefined dependencies)
  - `application.properties`/`application.yml` config

**Code example (main class):**

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

# 3️⃣ Advantages of Spring Boot ✅

**Simple line:**  
👉 Spring Boot makes Spring apps **faster to start**, **easier to configure**, and **simpler to deploy.**

**Key benefits:**

- ⭐ Auto-configuration (no boilerplate setup)
- ⭐ Embedded server (`java -jar app.jar`)
- ⭐ Starter dependencies (`spring-boot-starter-web`, etc.)
- ⭐ Production-ready features (actuator, metrics, health checks)
- ⭐ Properties-based configuration

---

# 4️⃣ What is Dependency Injection (DI) in Spring? 💉

**Simple line:**  
👉 DI means **Spring creates and injects objects for you** instead of you doing `new` everywhere.

**Example without DI:**

```java
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // Tightly coupled
}
```

**With DI:**

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) { // Spring injects it
        this.paymentService = paymentService;
    }
}
```

- You just declare **dependencies as fields/constructor parameters**
- Spring **wires** them automatically.

---

# 5️⃣ @Component vs @Service vs @Repository vs @Controller 🧩

**Simple line:**  
👉 All are **stereotypes** and create **beans**, but indicate **different roles**.

- `@Component` 🧱  
  - Generic component; any Spring-managed bean.
- `@Service` 🧠  
  - Business logic services (service layer).
- `@Repository` 🗄️  
  - DAO layer; data access logic; adds exception translation.
- `@Controller` 🧭  
  - Web MVC controller; handles HTTP requests (returns views).

**Quick example:**

```java
@Component
public class UtilityHelper {}

@Service
public class UserService {}

@Repository
public class UserRepository {}

@Controller
public class UserController {}
```

---

# 6️⃣ What is ApplicationContext? 🧠

**Simple line:**  
👉 `ApplicationContext` is the **Spring IoC container** that manages beans and their dependencies.

**It:**

- Creates, configures, and manages beans.
- Handles DI, lifecycle, events, message sources, etc.

**Example:**

```java
AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

MyService myService = context.getBean(MyService.class);
```

---

# 7️⃣ What is @SpringBootApplication? 🎯

**Simple line:**  
👉 `@SpringBootApplication` is a convenience annotation that combines **3 annotations** for a Spring Boot app.

It is basically:

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {}
```

- `@SpringBootConfiguration` → like `@Configuration`
- `@EnableAutoConfiguration` → enables auto config
- `@ComponentScan` → scans for components in the package

---

# 8️⃣ What is @EnableAutoConfiguration? 🤖

**Simple line:**  
👉 Tells Spring Boot: “**Configure beans automatically based on the classpath and properties.**”

- Looks at:
  - What dependencies are on the **classpath**.
  - Your **configuration properties**.
- Then picks suitable auto-configurations from `spring-boot-autoconfigure`.

You rarely use it directly; it’s inside `@SpringBootApplication`.

---

# 9️⃣ How does Spring Boot decide what to auto-configure? 🧩

**Simple line:**  
👉 Spring Boot auto-configures based on **what’s on the classpath + existing beans + properties + conditions.**

Internally:

- Auto-config classes are listed in:
  - `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
- Each has `@Conditional` annotations like:
  - `@ConditionalOnClass`
  - `@ConditionalOnMissingBean`
  - `@ConditionalOnProperty`

**Example (simplified idea):**

```java
@Configuration
@ConditionalOnClass(DataSource.class)
public class DataSourceAutoConfiguration { ... }
```

If `DataSource` is on your classpath → this config is applied.

---

# 🔟 Role of application.properties / application.yml ⚙️

**Simple line:**  
👉 These files store **configuration values** for your app (ports, DB URLs, etc.).

**Examples:**

`application.properties`:

```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/testdb
app.message=Hello from properties!
```

`application.yml`:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb

app:
  message: Hello from YAML!
```

You can read them with `@Value` or by binding to `@ConfigurationProperties`.

---

# 1️⃣1️⃣ What is a Bean in Spring? 🫘

**Simple line:**  
👉 A **bean** is an object that Spring’s container **creates, manages, and injects**.

- Normal Java object
- Lifecycle is controlled by Spring
- Defined via:
  - Annotations (`@Component`, `@Service`, `@Bean`)
  - XML (older style)

---

# 1️⃣2️⃣ How do you define a Bean? 🧪

**Simple line:**  
👉 Define beans using annotations like `@Component` or with a `@Bean` method in a `@Configuration` class.

**Using stereotype annotations:**

```java
@Component
public class EmailService {}
```

**Using @Bean in @Configuration:**

```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        return new EmailService();
    }
}
```

---

# 1️⃣3️⃣ Lifecycle of a Spring Bean 🔁

**Simple line:**  
👉 A bean goes from **creation → dependency injection → initialization → ready → destruction.**

**Typical lifecycle steps:**

1. Instantiate bean (constructor)
2. Populate properties (DI)
3. `BeanNameAware`, `BeanFactoryAware`, etc. (optional)
4. `BeanPostProcessor` (before init)
5. Initialization:
   - `afterPropertiesSet()` (if implements `InitializingBean`)
   - `@PostConstruct` method
6. `BeanPostProcessor` (after init)
7. Bean is **ready to use**
8. On context shutdown:
   - `@PreDestroy` method
   - `destroy()` if `DisposableBean`

**Example:**

```java
@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Bean initialized");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean about to be destroyed");
    }
}
```

---

# 1️⃣4️⃣ What is @Qualifier used for? 🎯

**Simple line:**  
👉 `@Qualifier` is used to choose **which bean** to inject when there are **multiple beans of the same type.**

**Example:**

```java
public interface PaymentService {}

@Service
@Qualifier("creditCard")
class CreditCardPaymentService implements PaymentService {}

@Service
@Qualifier("paypal")
class PaypalPaymentService implements PaymentService {}

@Service
class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(@Qualifier("paypal") PaymentService paymentService) {
        this.paymentService = paymentService; // will inject PaypalPaymentService
    }
}
```

---

# 1️⃣5️⃣ Singleton vs Prototype scope in Spring 🧬

**Simple line:**  
👉 **Singleton** = one bean instance per container.  
👉 **Prototype** = a new bean instance each time it’s requested.

**Default scope:** `singleton`.

```java
@Component
@Scope("singleton")
public class SingletonBean {}

@Component
@Scope("prototype")
public class PrototypeBean {}
```

- Singleton:
  - Same object returned every time from the context.
- Prototype:
  - New object created whenever you call `context.getBean()`.

---

# 1️⃣6️⃣ What is @Value used for? 💵

**Simple line:**  
👉 `@Value` injects **literal values, properties, or SpEL expressions** into fields or parameters.

**Examples:**

```java
@Value("${app.message}")
private String message; // from properties

@Value("Hello")
private String fixed;   // literal

@Value("#{2 + 3}")
private int sum;        // SpEL
```

---

# 1️⃣7️⃣ What is Spring AOP? 🧿

**Simple line:**  
👉 AOP (Aspect-Oriented Programming) lets you apply **cross-cutting concerns** (logging, security, transactions) **around** your methods.

**Key terms:**

- Aspect → class with cross-cutting logic
- Advice → action taken (before/after/around method)
- Join point → point during execution (method call)
- Pointcut → expression to select join points

**Example:**

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint jp) {
        System.out.println("Calling: " + jp.getSignature());
    }
}
```

---

# 1️⃣8️⃣ Key Modules in Spring 🧱

**Simple line:**  
👉 Spring is made of many modules; you use only what you need.

Important ones:

- 🌱 **Core, Beans, Context** → IoC & DI
- 🌐 **Spring Web / Spring MVC** → web apps & REST
- 🗄️ **Spring JDBC / ORM / Data JPA** → data access
- 🔐 **Spring Security** → authentication & authorization
- 🔁 **Spring AOP** → aspect-oriented programming
- 💳 **Spring Transaction** → transaction management
- 🔌 **Spring Integration / Cloud** → integration & cloud

---

# 1️⃣9️⃣ What is Circular Dependency & how does Spring handle it? 🔄

**Simple line:**  
👉 Circular dependency is when **Bean A depends on Bean B and Bean B depends on Bean A**.

**Example:**

```java
@Service
class A {
    @Autowired
    B b;
}

@Service
class B {
    @Autowired
    A a;
}
```

- With **constructor injection**, this causes a **circular reference error** in modern Spring Boot (circular references are disabled by default from Boot 2.6).
- With **setter/field injection**, Spring can sometimes resolve it using proxies, but it’s still a design smell.

Best practice: **avoid circular dependencies** (refactor to a third bean or interface).

---

# 2️⃣0️⃣ How do you run a Spring Boot application? ▶️

**Simple line:**  
👉 Run using the `main` method, or `mvn spring-boot:run`, or `java -jar`.

**Options:**

1. From IDE:
   ```java
   SpringApplication.run(DemoApplication.class, args);
   ```
2. Maven:
   ```bash
   mvn spring-boot:run
   ```
3. Fat JAR:
   ```bash
   mvn clean package
   java -jar target/demo-0.0.1-SNAPSHOT.jar
   ```

---

# 2️⃣1️⃣ BeanFactory vs ApplicationContext ⚖️

**Simple line:**  
👉 `BeanFactory` is the basic container; `ApplicationContext` is a **superset** with more features, used in almost all real apps.

- `BeanFactory`:
  - Basic DI container.
  - Lazy loading of beans.
- `ApplicationContext`:
  - Extends `BeanFactory`.
  - Adds:
    - Internationalization (messages)
    - Application events
    - AOP integration
    - Environment & property support

Spring Boot uses `ApplicationContext` by default.

---

# 2️⃣2️⃣ What is @Primary? 🥇

**Simple line:**  
👉 `@Primary` marks a bean as the **default** when multiple beans of the same type exist.

**Example:**

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {}

@Service
public class PaypalPaymentService implements PaymentService {}

@Service
class OrderService {
    @Autowired
    private PaymentService paymentService; 
    // -> CreditCardPaymentService will be injected
}
```

Use `@Primary` or `@Qualifier` to resolve ambiguity.

---

# 2️⃣3️⃣ What is @Lazy? 🐢

**Simple line:**  
👉 `@Lazy` delays bean creation until it is **needed**.

**At bean definition:**

```java
@Component
@Lazy
public class HeavyBean {}
```

**At injection point:**

```java
@Service
public class MyService {

    @Autowired
    @Lazy
    private HeavyBean heavyBean;
}
```

- Improves startup time.
- The bean is created only when first requested.

---

# 2️⃣4️⃣ @Inject vs @Autowired vs @Resource ⚔️

**Simple line:**  
👉 All inject dependencies, but from different specs and with slightly different behavior.

- `@Autowired` (Spring specific)
  - From `org.springframework.beans.factory.annotation`
  - Injection by type (default)
  - Has `required` attribute (e.g., `@Autowired(required = false)`)

- `@Inject` (JSR-330 / Jakarta)
  - From `jakarta.inject` (or `javax.inject` in older)
  - Similar to `@Autowired`
  - No `required` attribute

- `@Resource` (JSR-250)
  - From `jakarta.annotation` (`javax.annotation` older)
  - Injection by **name** first, then type
  - You can specify `@Resource(name="beanName")`

**Example:**

```java
@Autowired
private PaymentService paymentService1;

@Inject
private PaymentService paymentService2;

@Resource(name = "paypalPaymentService")
private PaymentService paymentService3;
```

---

# 2️⃣5️⃣ What is Spring Expression Language (SpEL)? 🧮

**Simple line:**  
👉 SpEL is a powerful expression language you can use inside annotations or XML to compute values at runtime.

**Examples:**

```java
@Value("#{2 + 3}")
private int sum; // 5

@Value("#{T(java.lang.Math).random() * 100}")
private double random;

@Value("#{userService.defaultUserName}")
private String defaultUserName;
```

Used in `@Value`, security expressions, conditional configs, etc.

---

# 2️⃣6️⃣ @Controller vs @RestController 🌐

**Simple line:**  
👉 `@RestController = @Controller + @ResponseBody` for every method.

- `@Controller`
  - Returns **views** (e.g., JSP, Thymeleaf)
  - Use `@ResponseBody` on methods to return JSON

- `@RestController`
  - Used for **REST APIs**
  - Every method returns JSON/XML directly

**Example:**

```java
@Controller
public class PageController {
    @GetMapping("/home")
    public String homePage() {
        return "home"; // view name
    }
}

@RestController
public class ApiController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello JSON"; // response body
    }
}
```

---

# 2️⃣7️⃣ What are Spring Boot Starters? ⭐

**Simple line:**  
👉 Starters are **pre-packaged dependency bundles** to quickly add features.

Examples:

- `spring-boot-starter-web` → Spring MVC + embedded Tomcat + JSON + validation
- `spring-boot-starter-data-jpa` → Spring Data JPA + Hibernate
- `spring-boot-starter-security` → Spring Security
- `spring-boot-starter-test` → JUnit, Mockito, etc.

**Maven example:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

# 2️⃣8️⃣ How does Spring Boot handle embedded servers? 🧩

**Simple line:**  
👉 Spring Boot embeds a web server (Tomcat/Jetty/Undertow) inside your JAR, so you can **run the app directly**.

- `spring-boot-starter-web` → embedded Tomcat (default)
- `spring-boot-starter-webflux` → Netty (by default)
- No need to deploy a WAR to external Tomcat.

You just run:

```bash
java -jar app.jar
```

Spring Boot starts the embedded server and hosts your REST endpoints.

---

# 2️⃣9️⃣ @PostConstruct vs @PreDestroy ⏱️

**Simple line:**  
👉 `@PostConstruct` runs **after dependency injection**; `@PreDestroy` runs **before bean destruction**.

**Example:**

```java
@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Init logic here");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Cleanup logic here");
    }
}
```

- `@PostConstruct` → initialization (e.g., open connections, load cache)
- `@PreDestroy` → cleanup (e.g., close connections)

---

# 3️⃣0️⃣ What is the Spring IoC Container? 📦

**Simple line:**  
👉 The IoC (Inversion of Control) container is the part of Spring that **creates and manages beans and their dependencies.**

In Spring, IoC container is mainly:

- `BeanFactory`
- `ApplicationContext` (commonly used)

It:

- Reads configuration (annotations/XML/Java config)
- Creates beans
- Injects dependencies
- Manages lifecycle

---
