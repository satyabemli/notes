# Complete JUnit 5 & Mockito Guide for REST API Testing 🚀

## Project: Employee Management System

---

## 📁 Project Structure

```
employee-management/
├── src/
│   ├── main/
│   │   ├── java/com/example/employee/
│   │   │   ├── controller/
│   │   │   │   └── EmployeeController.java
│   │   │   ├── service/
│   │   │   │   ├── EmployeeService.java
│   │   │   │   └── EmailService.java
│   │   │   ├── repository/
│   │   │   │   └── EmployeeRepository.java
│   │   │   ├── model/
│   │   │   │   └── Employee.java
│   │   │   ├── dto/
│   │   │   │   ├── EmployeeDTO.java
│   │   │   │   └── EmployeeResponseDTO.java
│   │   │   ├── exception/
│   │   │   │   ├── EmployeeNotFoundException.java
│   │   │   │   └── GlobalExceptionHandler.java
│   │   │   └── EmployeeManagementApplication.java
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       └── java/com/example/employee/
│           ├── controller/
│           │   └── EmployeeControllerTest.java
│           └── service/
│               └── EmployeeServiceTest.java
└── pom.xml
```

---

## 1️⃣ **pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version> <!-- 3.5.0 not released yet, using latest stable -->
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>employee-management</artifactId>
    <version>1.0.0</version>
    <name>Employee Management System</name>
    
    <properties>
        <java.version>17</java.version>
        <lombok.version>1.18.30</lombok.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- Testing Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Mockito (included in spring-boot-starter-test) -->
        <!-- JUnit 5 (included in spring-boot-starter-test) -->
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 2️⃣ **Application Configuration**

### `application.yml`

```yaml
spring:
  application:
    name: employee-management
  
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  
  h2:
    console:
      enabled: true

server:
  port: 8080

logging:
  level:
    root: INFO
    com.hcltech.leave: DEBUG
    org.springframework.web: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
```

---

## 3️⃣ **Model Layer**

### `Employee.java`

```java
package com.example.employee.model;

import jakarta.persistence.*;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "employees")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String department;
    
    @Column(nullable = false)
    private BigDecimal salary;
    
    @Column(nullable = false)
    private LocalDate joiningDate;
    
    @Column(nullable = false)
    private Boolean isActive;
}
```

---

## 4️⃣ **DTO Layer**

### `EmployeeDTO.java`

```java
package com.example.employee.dto;

import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class EmployeeDTO {
    
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
    private String firstName;
    
    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50, message = "Last name must be between 2 and 50 characters")
    private String lastName;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    @NotBlank(message = "Department is required")
    private String department;
    
    @NotNull(message = "Salary is required")
    @DecimalMin(value = "0.0", inclusive = false, message = "Salary must be greater than 0")
    private BigDecimal salary;
    
    @NotNull(message = "Joining date is required")
    @PastOrPresent(message = "Joining date cannot be in the future")
    private LocalDate joiningDate;
}
```

### `EmployeeResponseDTO.java`

```java
package com.example.employee.dto;

import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class EmployeeResponseDTO {
    
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String department;
    private BigDecimal salary;
    private LocalDate joiningDate;
    private Boolean isActive;
    private String fullName;
}
```

---

## 5️⃣ **Repository Layer**

### `EmployeeRepository.java`

```java
package com.example.employee.repository;

import com.example.employee.model.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    Optional<Employee> findByEmail(String email);
    
    List<Employee> findByDepartment(String department);
    
    List<Employee> findByIsActive(Boolean isActive);
    
    @Query("SELECT e FROM Employee e WHERE e.salary >= :minSalary AND e.salary <= :maxSalary")
    List<Employee> findBySalaryRange(BigDecimal minSalary, BigDecimal maxSalary);
    
    boolean existsByEmail(String email);
}
```

---

## 6️⃣ **Exception Handling**

### `EmployeeNotFoundException.java`

```java
package com.example.employee.exception;

public class EmployeeNotFoundException extends RuntimeException {
    
    public EmployeeNotFoundException(String message) {
        super(message);
    }
    
    public EmployeeNotFoundException(Long id) {
        super("Employee not found with id: " + id);
    }
}
```

### `GlobalExceptionHandler.java`

```java
package com.example.employee.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFoundException(EmployeeNotFoundException ex) {
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.NOT_FOUND.value())
                .error("Not Found")
                .message(ex.getMessage())
                .build();
        
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Bad Request")
                .message(ex.getMessage())
                .build();
        
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    @lombok.Data
    @lombok.Builder
    static class ErrorResponse {
        private LocalDateTime timestamp;
        private int status;
        private String error;
        private String message;
    }
}
```

---

## 7️⃣ **Service Layer**

### `EmailService.java`

```java
package com.example.employee.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class EmailService {
    
    /**
     * Simulates sending welcome email to new employee
     */
    public void sendWelcomeEmail(String email, String fullName) {
        log.info("Sending welcome email to {} ({})", fullName, email);
        // Actual email sending logic would go here
    }
    
    /**
     * Simulates sending deactivation email
     */
    public void sendDeactivationEmail(String email, String fullName) {
        log.info("Sending deactivation email to {} ({})", fullName, email);
        // Actual email sending logic would go here
    }
}
```

### `EmployeeService.java`

```java
package com.example.employee.service;

import com.example.employee.dto.EmployeeDTO;
import com.example.employee.dto.EmployeeResponseDTO;
import com.example.employee.exception.EmployeeNotFoundException;
import com.example.employee.model.Employee;
import com.example.employee.repository.EmployeeRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional
public class EmployeeService {
    
    private final EmployeeRepository employeeRepository;
    private final EmailService emailService;
    
    /**
     * Creates a new employee and sends welcome email
     */
    public EmployeeResponseDTO createEmployee(EmployeeDTO employeeDTO) {
        log.info("Creating employee with email: {}", employeeDTO.getEmail());
        
        // Business validation
        if (employeeRepository.existsByEmail(employeeDTO.getEmail())) {
            throw new IllegalArgumentException("Employee with email " + employeeDTO.getEmail() + " already exists");
        }
        
        Employee employee = mapToEntity(employeeDTO);
        employee.setIsActive(true);
        
        Employee savedEmployee = employeeRepository.save(employee);
        log.info("Employee created with ID: {}", savedEmployee.getId());
        
        // Send welcome email
        emailService.sendWelcomeEmail(
            savedEmployee.getEmail(),
            savedEmployee.getFirstName() + " " + savedEmployee.getLastName()
        );
        
        return mapToResponseDTO(savedEmployee);
    }
    
    /**
     * Retrieves employee by ID
     */
    @Transactional(readOnly = true)
    public EmployeeResponseDTO getEmployeeById(Long id) {
        log.info("Fetching employee with ID: {}", id);
        
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() -> new EmployeeNotFoundException(id));
        
        return mapToResponseDTO(employee);
    }
    
    /**
     * Retrieves all active employees
     */
    @Transactional(readOnly = true)
    public List<EmployeeResponseDTO> getAllActiveEmployees() {
        log.info("Fetching all active employees");
        
        return employeeRepository.findByIsActive(true)
                .stream()
                .map(this::mapToResponseDTO)
                .collect(Collectors.toList());
    }
    
    /**
     * Updates employee information
     */
    public EmployeeResponseDTO updateEmployee(Long id, EmployeeDTO employeeDTO) {
        log.info("Updating employee with ID: {}", id);
        
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() -> new EmployeeNotFoundException(id));
        
        // Check if email is being changed and if new email already exists
        if (!employee.getEmail().equals(employeeDTO.getEmail()) &&
            employeeRepository.existsByEmail(employeeDTO.getEmail())) {
            throw new IllegalArgumentException("Email " + employeeDTO.getEmail() + " is already in use");
        }
        
        updateEntityFromDTO(employee, employeeDTO);
        Employee updatedEmployee = employeeRepository.save(employee);
        
        log.info("Employee updated successfully with ID: {}", id);
        return mapToResponseDTO(updatedEmployee);
    }
    
    /**
     * Deactivates an employee (soft delete)
     */
    public void deactivateEmployee(Long id) {
        log.info("Deactivating employee with ID: {}", id);
        
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() -> new EmployeeNotFoundException(id));
        
        employee.setIsActive(false);
        employeeRepository.save(employee);
        
        // Send deactivation email
        emailService.sendDeactivationEmail(
            employee.getEmail(),
            employee.getFirstName() + " " + employee.getLastName()
        );
        
        log.info("Employee deactivated successfully with ID: {}", id);
    }
    
    /**
     * Gives raise to employee
     */
    public EmployeeResponseDTO giveRaise(Long id, BigDecimal raisePercentage) {
        log.info("Giving {}% raise to employee ID: {}", raisePercentage, id);
        
        if (raisePercentage.compareTo(BigDecimal.ZERO) <= 0 || raisePercentage.compareTo(new BigDecimal("100")) > 0) {
            throw new IllegalArgumentException("Raise percentage must be between 0 and 100");
        }
        
        Employee employee = employeeRepository.findById(id)
                .orElseThrow(() -> new EmployeeNotFoundException(id));
        
        if (!employee.getIsActive()) {
            throw new IllegalArgumentException("Cannot give raise to inactive employee");
        }
        
        BigDecimal currentSalary = employee.getSalary();
        BigDecimal raiseAmount = currentSalary.multiply(raisePercentage).divide(new BigDecimal("100"));
        BigDecimal newSalary = currentSalary.add(raiseAmount);
        
        employee.setSalary(newSalary);
        Employee updatedEmployee = employeeRepository.save(employee);
        
        log.info("Raise applied. New salary: {}", newSalary);
        return mapToResponseDTO(updatedEmployee);
    }
    
    /**
     * Gets employees by department
     */
    @Transactional(readOnly = true)
    public List<EmployeeResponseDTO> getEmployeesByDepartment(String department) {
        log.info("Fetching employees from department: {}", department);
        
        return employeeRepository.findByDepartment(department)
                .stream()
                .map(this::mapToResponseDTO)
                .collect(Collectors.toList());
    }
    
    // Mapping methods
    private Employee mapToEntity(EmployeeDTO dto) {
        return Employee.builder()
                .firstName(dto.getFirstName())
                .lastName(dto.getLastName())
                .email(dto.getEmail())
                .department(dto.getDepartment())
                .salary(dto.getSalary())
                .joiningDate(dto.getJoiningDate())
                .build();
    }
    
    private void updateEntityFromDTO(Employee employee, EmployeeDTO dto) {
        employee.setFirstName(dto.getFirstName());
        employee.setLastName(dto.getLastName());
        employee.setEmail(dto.getEmail());
        employee.setDepartment(dto.getDepartment());
        employee.setSalary(dto.getSalary());
        employee.setJoiningDate(dto.getJoiningDate());
    }
    
    private EmployeeResponseDTO mapToResponseDTO(Employee employee) {
        return EmployeeResponseDTO.builder()
                .id(employee.getId())
                .firstName(employee.getFirstName())
                .lastName(employee.getLastName())
                .email(employee.getEmail())
                .department(employee.getDepartment())
                .salary(employee.getSalary())
                .joiningDate(employee.getJoiningDate())
                .isActive(employee.getIsActive())
                .fullName(employee.getFirstName() + " " + employee.getLastName())
                .build();
    }
}
```

---

## 8️⃣ **Controller Layer**

### `EmployeeController.java`

```java
package com.example.employee.controller;

import com.example.employee.dto.EmployeeDTO;
import com.example.employee.dto.EmployeeResponseDTO;
import com.example.employee.service.EmployeeService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;
import java.util.List;

@RestController
@RequestMapping("/api/v1/employees")
@RequiredArgsConstructor
@Slf4j
public class EmployeeController {
    
    private final EmployeeService employeeService;
    
    @PostMapping
    public ResponseEntity<EmployeeResponseDTO> createEmployee(@Valid @RequestBody EmployeeDTO employeeDTO) {
        log.info("REST request to create employee: {}", employeeDTO.getEmail());
        EmployeeResponseDTO response = employeeService.createEmployee(employeeDTO);
        return new ResponseEntity<>(response, HttpStatus.CREATED);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<EmployeeResponseDTO> getEmployeeById(@PathVariable Long id) {
        log.info("REST request to get employee by ID: {}", id);
        EmployeeResponseDTO response = employeeService.getEmployeeById(id);
        return ResponseEntity.ok(response);
    }
    
    @GetMapping
    public ResponseEntity<List<EmployeeResponseDTO>> getAllActiveEmployees() {
        log.info("REST request to get all active employees");
        List<EmployeeResponseDTO> response = employeeService.getAllActiveEmployees();
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/department/{department}")
    public ResponseEntity<List<EmployeeResponseDTO>> getEmployeesByDepartment(@PathVariable String department) {
        log.info("REST request to get employees by department: {}", department);
        List<EmployeeResponseDTO> response = employeeService.getEmployeesByDepartment(department);
        return ResponseEntity.ok(response);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<EmployeeResponseDTO> updateEmployee(
            @PathVariable Long id,
            @Valid @RequestBody EmployeeDTO employeeDTO) {
        log.info("REST request to update employee with ID: {}", id);
        EmployeeResponseDTO response = employeeService.updateEmployee(id, employeeDTO);
        return ResponseEntity.ok(response);
    }
    
    @PatchMapping("/{id}/raise")
    public ResponseEntity<EmployeeResponseDTO> giveRaise(
            @PathVariable Long id,
            @RequestParam BigDecimal percentage) {
        log.info("REST request to give raise to employee ID: {} with percentage: {}", id, percentage);
        EmployeeResponseDTO response = employeeService.giveRaise(id, percentage);
        return ResponseEntity.ok(response);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deactivateEmployee(@PathVariable Long id) {
        log.info("REST request to deactivate employee with ID: {}", id);
        employeeService.deactivateEmployee(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 9️⃣ **SERVICE LAYER TEST CASES** 🧪

### `EmployeeServiceTest.java`

```java
package com.example.employee.service;

import com.example.employee.dto.EmployeeDTO;
import com.example.employee.dto.EmployeeResponseDTO;
import com.example.employee.exception.EmployeeNotFoundException;
import com.example.employee.model.Employee;
import com.example.employee.repository.EmployeeRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

/**
 * ✅ @ExtendWith(MockitoExtension.class) - Enables Mockito annotations
 * ✅ @Mock - Creates mock object
 * ✅ @InjectMocks - Injects mock dependencies into the service
 */
@ExtendWith(MockitoExtension.class)
@DisplayName("Employee Service Tests")
class EmployeeServiceTest {
    
    @Mock
    private EmployeeRepository employeeRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private EmployeeService employeeService;
    
    private EmployeeDTO employeeDTO;
    private Employee employee;
    
    /**
     * 🔹 @BeforeEach - Runs before each test method
     * 🔹 Sets up test data to avoid repetition
     */
    @BeforeEach
    void setUp() {
        employeeDTO = EmployeeDTO.builder()
                .firstName("John")
                .lastName("Doe")
                .email("john.doe@example.com")
                .department("IT")
                .salary(new BigDecimal("75000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        employee = Employee.builder()
                .id(1L)
                .firstName("John")
                .lastName("Doe")
                .email("john.doe@example.com")
                .department("IT")
                .salary(new BigDecimal("75000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .isActive(true)
                .build();
    }
    
    // ==================== CREATE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: Happy path - Successfully create employee
     * 
     * EXPLANATION:
     * - when() - Defines mock behavior
     * - thenReturn() - Returns predefined value
     * - verify() - Verifies method was called
     * - times(1) - Verifies method was called exactly once
     * - assertThat() - AssertJ assertion (more readable than JUnit assertions)
     */
    @Test
    @DisplayName("Should create employee successfully and send welcome email")
    void shouldCreateEmployeeSuccessfully() {
        // GIVEN (Arrange) - Setup test data and mock behavior
        when(employeeRepository.existsByEmail(anyString())).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(employee);
        doNothing().when(emailService).sendWelcomeEmail(anyString(), anyString());
        
        // WHEN (Act) - Execute the method under test
        EmployeeResponseDTO result = employeeService.createEmployee(employeeDTO);
        
        // THEN (Assert) - Verify the results
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getEmail()).isEqualTo("john.doe@example.com");
        assertThat(result.getFullName()).isEqualTo("John Doe");
        assertThat(result.getIsActive()).isTrue();
        
        // Verify interactions
        verify(employeeRepository, times(1)).existsByEmail("john.doe@example.com");
        verify(employeeRepository, times(1)).save(any(Employee.class));
        verify(emailService, times(1)).sendWelcomeEmail(anyString(), anyString());
    }
    
    /**
     * 📝 TEST SCENARIO: Negative case - Duplicate email
     * 
     * EXPLANATION:
     * - assertThatThrownBy() - Verifies that exception is thrown
     * - isInstanceOf() - Checks exception type
     * - hasMessage() - Checks exception message
     * - verify(never()) - Verifies method was never called
     */
    @Test
    @DisplayName("Should throw exception when employee with same email exists")
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        // GIVEN
        when(employeeRepository.existsByEmail(anyString())).thenReturn(true);
        
        // WHEN & THEN
        assertThatThrownBy(() -> employeeService.createEmployee(employeeDTO))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("already exists");
        
        // Verify save was never called
        verify(employeeRepository, never()).save(any(Employee.class));
        verify(emailService, never()).sendWelcomeEmail(anyString(), anyString());
    }
    
    // ==================== GET EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: Find employee by ID - Success
     * 
     * EXPLANATION:
     * - Optional.of() - Mocks repository returning optional with value
     */
    @Test
    @DisplayName("Should return employee when valid ID is provided")
    void shouldReturnEmployeeWhenValidIdProvided() {
        // GIVEN
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        
        // WHEN
        EmployeeResponseDTO result = employeeService.getEmployeeById(1L);
        
        // THEN
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getEmail()).isEqualTo("john.doe@example.com");
        
        verify(employeeRepository, times(1)).findById(1L);
    }
    
    /**
     * 📝 TEST SCENARIO: Find employee by ID - Not found
     * 
     * EXPLANATION:
     * - Optional.empty() - Mocks repository returning empty optional
     * - Tests custom exception handling
     */
    @Test
    @DisplayName("Should throw EmployeeNotFoundException when ID does not exist")
    void shouldThrowExceptionWhenEmployeeNotFound() {
        // GIVEN
        when(employeeRepository.findById(999L)).thenReturn(Optional.empty());
        
        // WHEN & THEN
        assertThatThrownBy(() -> employeeService.getEmployeeById(999L))
                .isInstanceOf(EmployeeNotFoundException.class)
                .hasMessageContaining("999");
        
        verify(employeeRepository, times(1)).findById(999L);
    }
    
    /**
     * 📝 TEST SCENARIO: Get all active employees
     * 
     * EXPLANATION:
     * - Tests list return type
     * - hasSize() - Verifies collection size
     * - extracting() - Extracts property from collection for assertion
     */
    @Test
    @DisplayName("Should return list of all active employees")
    void shouldReturnAllActiveEmployees() {
        // GIVEN
        Employee employee2 = Employee.builder()
                .id(2L)
                .firstName("Jane")
                .lastName("Smith")
                .email("jane.smith@example.com")
                .department("HR")
                .salary(new BigDecimal("80000"))
                .joiningDate(LocalDate.of(2024, 2, 1))
                .isActive(true)
                .build();
        
        when(employeeRepository.findByIsActive(true))
                .thenReturn(Arrays.asList(employee, employee2));
        
        // WHEN
        List<EmployeeResponseDTO> result = employeeService.getAllActiveEmployees();
        
        // THEN
        assertThat(result).isNotNull();
        assertThat(result).hasSize(2);
        assertThat(result)
                .extracting(EmployeeResponseDTO::getEmail)
                .containsExactlyInAnyOrder("john.doe@example.com", "jane.smith@example.com");
        
        verify(employeeRepository, times(1)).findByIsActive(true);
    }
    
    /**
     * 📝 TEST SCENARIO: Get employees by department
     * 
     * EXPLANATION:
     * - Tests filtering functionality
     */
    @Test
    @DisplayName("Should return employees filtered by department")
    void shouldReturnEmployeesByDepartment() {
        // GIVEN
        when(employeeRepository.findByDepartment("IT"))
                .thenReturn(Arrays.asList(employee));
        
        // WHEN
        List<EmployeeResponseDTO> result = employeeService.getEmployeesByDepartment("IT");
        
        // THEN
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getDepartment()).isEqualTo("IT");
        
        verify(employeeRepository, times(1)).findByDepartment("IT");
    }
    
    // ==================== UPDATE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: Update employee - Success
     * 
     * EXPLANATION:
     * - Tests update operation
     * - Verifies that repository methods are called in correct order
     */
    @Test
    @DisplayName("Should update employee successfully")
    void shouldUpdateEmployeeSuccessfully() {
        // GIVEN
        EmployeeDTO updateDTO = EmployeeDTO.builder()
                .firstName("John")
                .lastName("Doe Updated")
                .email("john.doe@example.com")
                .department("Engineering")
                .salary(new BigDecimal("85000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        when(employeeRepository.existsByEmail("john.doe@example.com")).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(employee);
        
        // WHEN
        EmployeeResponseDTO result = employeeService.updateEmployee(1L, updateDTO);
        
        // THEN
        assertThat(result).isNotNull();
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }
    
    /**
     * 📝 TEST SCENARIO: Update with duplicate email
     * 
     * EXPLANATION:
     * - Tests business rule validation during update
     */
    @Test
    @DisplayName("Should throw exception when updating to existing email")
    void shouldThrowExceptionWhenUpdatingToExistingEmail() {
        // GIVEN
        EmployeeDTO updateDTO = EmployeeDTO.builder()
                .firstName("John")
                .lastName("Doe")
                .email("different@example.com")
                .department("IT")
                .salary(new BigDecimal("75000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        when(employeeRepository.existsByEmail("different@example.com")).thenReturn(true);
        
        // WHEN & THEN
        assertThatThrownBy(() -> employeeService.updateEmployee(1L, updateDTO))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("already in use");
        
        verify(employeeRepository, never()).save(any(Employee.class));
    }
    
    // ==================== DEACTIVATE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: Deactivate employee (soft delete)
     * 
     * EXPLANATION:
     * - doNothing() - For void methods
     * - Tests soft delete functionality
     */
    @Test
    @DisplayName("Should deactivate employee and send email")
    void shouldDeactivateEmployeeSuccessfully() {
        // GIVEN
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        when(employeeRepository.save(any(Employee.class))).thenReturn(employee);
        doNothing().when(emailService).sendDeactivationEmail(anyString(), anyString());
        
        // WHEN
        employeeService.deactivateEmployee(1L);
        
        // THEN
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).save(any(Employee.class));
        verify(emailService, times(1)).sendDeactivationEmail(anyString(), anyString());
    }
    
    // ==================== GIVE RAISE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: Give raise - Complex business logic
     * 
     * EXPLANATION:
     * - Tests calculation logic
     * - isBetween() - Verifies numeric range
     * - Tests multiple conditions
     */
    @Test
    @DisplayName("Should calculate and apply raise correctly")
    void shouldGiveRaiseSuccessfully() {
        // GIVEN
        BigDecimal raisePercentage = new BigDecimal("10");
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        when(employeeRepository.save(any(Employee.class))).thenAnswer(invocation -> invocation.getArgument(0));
        
        // WHEN
        EmployeeResponseDTO result = employeeService.giveRaise(1L, raisePercentage);
        
        // THEN
        BigDecimal expectedSalary = new BigDecimal("82500"); // 75000 + 10%
        assertThat(result.getSalary()).isEqualByComparingTo(expectedSalary);
        
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }
    
    /**
     * 📝 TEST SCENARIO: Invalid raise percentage
     * 
     * EXPLANATION:
     * - Tests input validation
     * - Tests boundary conditions
     */
    @Test
    @DisplayName("Should throw exception for invalid raise percentage")
    void shouldThrowExceptionForInvalidRaisePercentage() {
        // GIVEN
        BigDecimal invalidPercentage = new BigDecimal("-5");
        
        // WHEN & THEN
        assertThatThrownBy(() -> employeeService.giveRaise(1L, invalidPercentage))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("between 0 and 100");
        
        verify(employeeRepository, never()).save(any(Employee.class));
    }
    
    /**
     * 📝 TEST SCENARIO: Cannot give raise to inactive employee
     * 
     * EXPLANATION:
     * - Tests business rule enforcement
     */
    @Test
    @DisplayName("Should throw exception when giving raise to inactive employee")
    void shouldThrowExceptionWhenGivingRaiseToInactiveEmployee() {
        // GIVEN
        employee.setIsActive(false);
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee));
        
        // WHEN & THEN
        assertThatThrownBy(() -> employeeService.giveRaise(1L, new BigDecimal("10")))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("inactive employee");
        
        verify(employeeRepository, never()).save(any(Employee.class));
    }
    
    // ==================== ARGUMENT CAPTOR TEST ====================
    
    /**
     * 📝 ADVANCED: Using ArgumentCaptor
     * 
     * EXPLANATION:
     * - ArgumentCaptor - Captures arguments passed to mock methods
     * - Useful when you need to verify exact values passed to dependencies
     */
    @Test
    @DisplayName("Should capture and verify employee details sent in email")
    void shouldCaptureEmailArgumentsWhenCreatingEmployee() {
        // GIVEN
        when(employeeRepository.existsByEmail(anyString())).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(employee);
        
        org.mockito.ArgumentCaptor<String> emailCaptor = 
            org.mockito.ArgumentCaptor.forClass(String.class);
        org.mockito.ArgumentCaptor<String> nameCaptor = 
            org.mockito.ArgumentCaptor.forClass(String.class);
        
        // WHEN
        employeeService.createEmployee(employeeDTO);
        
        // THEN
        verify(emailService).sendWelcomeEmail(emailCaptor.capture(), nameCaptor.capture());
        
        assertThat(emailCaptor.getValue()).isEqualTo("john.doe@example.com");
        assertThat(nameCaptor.getValue()).isEqualTo("John Doe");
    }
}
```

---

## 🔟 **CONTROLLER LAYER TEST CASES** 🧪

### `EmployeeControllerTest.java`

```java
package com.example.employee.controller;

import com.example.employee.dto.EmployeeDTO;
import com.example.employee.dto.EmployeeResponseDTO;
import com.example.employee.exception.EmployeeNotFoundException;
import com.example.employee.service.EmployeeService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;

import static org.hamcrest.Matchers.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * ✅ @WebMvcTest - Loads only web layer components (Controllers)
 * ✅ @MockBean - Creates mock bean in Spring context
 * ✅ MockMvc - Simulates HTTP requests without starting server
 * ✅ ObjectMapper - Converts objects to JSON and vice versa
 */
@WebMvcTest(EmployeeController.class)
@DisplayName("Employee Controller Tests")
class EmployeeControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @MockBean
    private EmployeeService employeeService;
    
    private EmployeeDTO employeeDTO;
    private EmployeeResponseDTO employeeResponseDTO;
    
    @BeforeEach
    void setUp() {
        employeeDTO = EmployeeDTO.builder()
                .firstName("John")
                .lastName("Doe")
                .email("john.doe@example.com")
                .department("IT")
                .salary(new BigDecimal("75000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        employeeResponseDTO = EmployeeResponseDTO.builder()
                .id(1L)
                .firstName("John")
                .lastName("Doe")
                .email("john.doe@example.com")
                .department("IT")
                .salary(new BigDecimal("75000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .isActive(true)
                .fullName("John Doe")
                .build();
    }
    
    // ==================== CREATE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: POST request - Create employee successfully
     * 
     * EXPLANATION:
     * - mockMvc.perform() - Executes HTTP request
     * - post() - HTTP POST method
     * - content() - Request body
     * - contentType() - Request content type
     * - andExpect() - Verifies response
     * - status().isCreated() - Checks HTTP 201
     * - jsonPath() - Extracts and verifies JSON fields
     */
    @Test
    @DisplayName("POST /api/v1/employees - Should create employee successfully")
    void shouldCreateEmployeeSuccessfully() throws Exception {
        // GIVEN
        when(employeeService.createEmployee(any(EmployeeDTO.class)))
                .thenReturn(employeeResponseDTO);
        
        // WHEN
        ResultActions response = mockMvc.perform(post("/api/v1/employees")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(employeeDTO)));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.email", is("john.doe@example.com")))
                .andExpect(jsonPath("$.fullName", is("John Doe")))
                .andExpect(jsonPath("$.isActive", is(true)))
                .andExpect(jsonPath("$.salary", is(75000)));
        
        verify(employeeService, times(1)).createEmployee(any(EmployeeDTO.class));
    }
    
    /**
     * 📝 TEST SCENARIO: POST request - Validation failure
     * 
     * EXPLANATION:
     * - Tests @Valid annotation
     * - status().isBadRequest() - Checks HTTP 400
     * - Tests field validation messages
     */
    @Test
    @DisplayName("POST /api/v1/employees - Should return 400 for invalid data")
    void shouldReturn400WhenEmployeeDataIsInvalid() throws Exception {
        // GIVEN - Invalid DTO (missing required fields)
        EmployeeDTO invalidDTO = EmployeeDTO.builder()
                .firstName("") // Empty first name
                .lastName("Doe")
                .email("invalid-email") // Invalid email format
                .department("IT")
                .salary(new BigDecimal("-1000")) // Negative salary
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        // WHEN
        ResultActions response = mockMvc.perform(post("/api/v1/employees")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidDTO)));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.firstName").exists())
                .andExpect(jsonPath("$.email").exists())
                .andExpect(jsonPath("$.salary").exists());
        
        verify(employeeService, never()).createEmployee(any(EmployeeDTO.class));
    }
    
    /**
     * 📝 TEST SCENARIO: POST request - Business logic exception
     * 
     * EXPLANATION:
     * - Tests exception handling
     * - willThrow() - Makes mock throw exception
     */
    @Test
    @DisplayName("POST /api/v1/employees - Should return 400 for duplicate email")
    void shouldReturn400WhenEmailAlreadyExists() throws Exception {
        // GIVEN
        when(employeeService.createEmployee(any(EmployeeDTO.class)))
                .thenThrow(new IllegalArgumentException("Employee with email john.doe@example.com already exists"));
        
        // WHEN
        ResultActions response = mockMvc.perform(post("/api/v1/employees")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(employeeDTO)));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.message", containsString("already exists")));
    }
    
    // ==================== GET EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: GET request - Get employee by ID
     * 
     * EXPLANATION:
     * - get() - HTTP GET method
     * - pathVariable passed in URL
     */
    @Test
    @DisplayName("GET /api/v1/employees/{id} - Should return employee")
    void shouldReturnEmployeeById() throws Exception {
        // GIVEN
        when(employeeService.getEmployeeById(1L)).thenReturn(employeeResponseDTO);
        
        // WHEN
        ResultActions response = mockMvc.perform(get("/api/v1/employees/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.email", is("john.doe@example.com")))
                .andExpect(jsonPath("$.department", is("IT")));
        
        verify(employeeService, times(1)).getEmployeeById(1L);
    }
    
    /**
     * 📝 TEST SCENARIO: GET request - Employee not found
     * 
     * EXPLANATION:
     * - Tests 404 response
     * - Tests custom exception handling
     */
    @Test
    @DisplayName("GET /api/v1/employees/{id} - Should return 404 when not found")
    void shouldReturn404WhenEmployeeNotFound() throws Exception {
        // GIVEN
        when(employeeService.getEmployeeById(999L))
                .thenThrow(new EmployeeNotFoundException(999L));
        
        // WHEN
        ResultActions response = mockMvc.perform(get("/api/v1/employees/{id}", 999L)
                .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.message", containsString("999")));
    }
    
    /**
     * 📝 TEST SCENARIO: GET request - Get all employees
     * 
     * EXPLANATION:
     * - Tests list response
     * - jsonPath("$") - Root element
     * - hasSize() - Verifies array size
     */
    @Test
    @DisplayName("GET /api/v1/employees - Should return all active employees")
    void shouldReturnAllActiveEmployees() throws Exception {
        // GIVEN
        EmployeeResponseDTO employee2 = EmployeeResponseDTO.builder()
                .id(2L)
                .firstName("Jane")
                .lastName("Smith")
                .email("jane.smith@example.com")
                .department("HR")
                .salary(new BigDecimal("80000"))
                .joiningDate(LocalDate.of(2024, 2, 1))
                .isActive(true)
                .fullName("Jane Smith")
                .build();
        
        List<EmployeeResponseDTO> employees = Arrays.asList(employeeResponseDTO, employee2);
        when(employeeService.getAllActiveEmployees()).thenReturn(employees);
        
        // WHEN
        ResultActions response = mockMvc.perform(get("/api/v1/employees")
                .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].email", is("john.doe@example.com")))
                .andExpect(jsonPath("$[1].email", is("jane.smith@example.com")));
    }
    
    /**
     * 📝 TEST SCENARIO: GET request with path variable
     * 
     * EXPLANATION:
     * - Tests filtering by department
     * - Multiple path variables
     */
    @Test
    @DisplayName("GET /api/v1/employees/department/{department} - Should return employees by department")
    void shouldReturnEmployeesByDepartment() throws Exception {
        // GIVEN
        when(employeeService.getEmployeesByDepartment("IT"))
                .thenReturn(Arrays.asList(employeeResponseDTO));
        
        // WHEN
        ResultActions response = mockMvc.perform(
                get("/api/v1/employees/department/{department}", "IT")
                        .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(1)))
                .andExpect(jsonPath("$[0].department", is("IT")));
    }
    
    // ==================== UPDATE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: PUT request - Update employee
     * 
     * EXPLANATION:
     * - put() - HTTP PUT method
     * - Tests update operation
     */
    @Test
    @DisplayName("PUT /api/v1/employees/{id} - Should update employee successfully")
    void shouldUpdateEmployeeSuccessfully() throws Exception {
        // GIVEN
        EmployeeDTO updateDTO = EmployeeDTO.builder()
                .firstName("John")
                .lastName("Doe Updated")
                .email("john.doe@example.com")
                .department("Engineering")
                .salary(new BigDecimal("85000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .build();
        
        EmployeeResponseDTO updatedResponse = EmployeeResponseDTO.builder()
                .id(1L)
                .firstName("John")
                .lastName("Doe Updated")
                .email("john.doe@example.com")
                .department("Engineering")
                .salary(new BigDecimal("85000"))
                .joiningDate(LocalDate.of(2024, 1, 15))
                .isActive(true)
                .fullName("John Doe Updated")
                .build();
        
        when(employeeService.updateEmployee(eq(1L), any(EmployeeDTO.class)))
                .thenReturn(updatedResponse);
        
        // WHEN
        ResultActions response = mockMvc.perform(put("/api/v1/employees/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updateDTO)));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.lastName", is("Doe Updated")))
                .andExpect(jsonPath("$.department", is("Engineering")))
                .andExpect(jsonPath("$.salary", is(85000)));
    }
    
    // ==================== PATCH EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: PATCH request - Give raise
     * 
     * EXPLANATION:
     * - patch() - HTTP PATCH method
     * - param() - Query parameter
     * - Tests partial update
     */
    @Test
    @DisplayName("PATCH /api/v1/employees/{id}/raise - Should give raise successfully")
    void shouldGiveRaiseSuccessfully() throws Exception {
        // GIVEN
        EmployeeResponseDTO raisedEmployee = EmployeeResponseDTO.builder()
                .id(1L)
                .firstName("John")
                .lastName("Doe")
                .email("john.doe@example.com")
                .department("IT")
                .salary(new BigDecimal("82500")) // 10% raise
                .joiningDate(LocalDate.of(2024, 1, 15))
                .isActive(true)
                .fullName("John Doe")
                .build();
        
        when(employeeService.giveRaise(eq(1L), any(BigDecimal.class)))
                .thenReturn(raisedEmployee);
        
        // WHEN
        ResultActions response = mockMvc.perform(
                patch("/api/v1/employees/{id}/raise", 1L)
                        .param("percentage", "10")
                        .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.salary", is(82500)));
    }
    
    // ==================== DELETE EMPLOYEE TESTS ====================
    
    /**
     * 📝 TEST SCENARIO: DELETE request - Deactivate employee
     * 
     * EXPLANATION:
     * - delete() - HTTP DELETE method
     * - doNothing() - For void service methods
     * - status().isNoContent() - Checks HTTP 204
     */
    @Test
    @DisplayName("DELETE /api/v1/employees/{id} - Should deactivate employee")
    void shouldDeactivateEmployeeSuccessfully() throws Exception {
        // GIVEN
        doNothing().when(employeeService).deactivateEmployee(1L);
        
        // WHEN
        ResultActions response = mockMvc.perform(delete("/api/v1/employees/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isNoContent());
        
        verify(employeeService, times(1)).deactivateEmployee(1L);
    }
    
    /**
     * 📝 TEST SCENARIO: DELETE request - Employee not found
     */
    @Test
    @DisplayName("DELETE /api/v1/employees/{id} - Should return 404 when employee not found")
    void shouldReturn404WhenDeactivatingNonExistentEmployee() throws Exception {
        // GIVEN
        doThrow(new EmployeeNotFoundException(999L))
                .when(employeeService).deactivateEmployee(999L);
        
        // WHEN
        ResultActions response = mockMvc.perform(delete("/api/v1/employees/{id}", 999L)
                .contentType(MediaType.APPLICATION_JSON));
        
        // THEN
        response.andDo(print())
                .andExpect(status().isNotFound());
    }
}
```

---

## 1️⃣1️⃣ **KEY TESTING CONCEPTS - CHEAT SHEET** 📝

### **Mockito Annotations**

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@Mock` | Creates mock object | `@Mock EmployeeRepository repo;` |
| `@InjectMocks` | Injects mocks into class | `@InjectMocks EmployeeService service;` |
| `@ExtendWith(MockitoExtension.class)` | Enables Mockito | For JUnit 5 |
| `@MockBean` | Spring context mock | Used in `@WebMvcTest` |

### **Common Mockito Methods**

```java
// Define behavior
when(repository.findById(1L)).thenReturn(Optional.of(employee));
when(service.save(any())).thenThrow(new RuntimeException());

// Void methods
doNothing().when(emailService).send(anyString());
doThrow(new Exception()).when(service).delete(1L);

// Verification
verify(repository, times(1)).save(any());
verify(repository, never()).delete(any());
verify(repository, atLeast(1)).findAll();

// Argument matching
any() // any object
anyString() // any string
eq(value) // exact value
```

### **AssertJ Assertions (Better than JUnit)**

```java
// Basic assertions
assertThat(result).isNotNull();
assertThat(result.getId()).isEqualTo(1L);
assertThat(result.getEmail()).isEqualTo("test@example.com");

// String assertions
assertThat(result.getName()).startsWith("John");
assertThat(result.getName()).contains("Doe");

// Collection assertions
assertThat(list).hasSize(2);
assertThat(list).isEmpty();
assertThat(list).containsExactly(item1, item2);

// Exception assertions
assertThatThrownBy(() -> service.method())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("error message");

// Numeric assertions
assertThat(salary).isGreaterThan(BigDecimal.ZERO);
assertThat(salary).isBetween(min, max);
```

### **MockMvc Methods**

```java
// Perform requests
mockMvc.perform(get("/api/employees"))
mockMvc.perform(post("/api/employees").content(json))
mockMvc.perform(put("/api/employees/1"))
mockMvc.perform(delete("/api/employees/1"))

// Status checks
.andExpect(status().isOk())           // 200
.andExpect(status().isCreated())      // 201
.andExpect(status().isNoContent())    // 204
.andExpect(status().isBadRequest())   // 400
.andExpect(status().isNotFound())     // 404

// JSON Path
.andExpect(jsonPath("$.id").value(1))
.andExpect(jsonPath("$.name").value("John"))
.andExpect(jsonPath("$", hasSize(2)))
.andExpect(jsonPath("$[0].id").exists())
```

---

## 1️⃣2️⃣ **INTERVIEW PREPARATION QUESTIONS** 🎯

### **Q1: What is the difference between @Mock and @MockBean?**
**Answer:** 
- `@Mock` is pure Mockito annotation, creates mock outside Spring context
- `@MockBean` is Spring Boot annotation, creates mock AND adds it to Spring ApplicationContext
- Use `@Mock` in unit tests, `@MockBean` in integration tests with `@WebMvcTest`

### **Q2: What is MockMvc and why use it?**
**Answer:** MockMvc allows testing Spring MVC controllers without starting HTTP server. It simulates HTTP requests and validates responses, perfect for testing REST APIs.

### **Q3: Explain @WebMvcTest vs @SpringBootTest**
**Answer:**
- `@WebMvcTest`: Loads only web layer (controllers), faster, for controller tests
- `@SpringBootTest`: Loads entire application context, slower, for integration tests

### **Q4: What is ArgumentCaptor?**
**Answer:** Captures arguments passed to mock methods for verification. Useful when you need to verify exact values passed to dependencies.

### **Q5: Difference between verify() and when()?**
**Answer:**
- `when()`: Defines mock behavior (stubbing)
- `verify()`: Verifies mock was called with expected arguments

---

## 🎓 **RUN THE TESTS**

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=EmployeeServiceTest

# Run with coverage
mvn test jacoco:report
```

---

## ✅ **BEST PRACTICES SUMMARY**

1. ✅ **Use @DisplayName** for readable test names
2. ✅ **Follow AAA pattern**: Arrange, Act, Assert
3. ✅ **Test both happy and negative scenarios**
4. ✅ **Use AssertJ** over JUnit assertions (more readable)
5. ✅ **Mock external dependencies** (email service, external APIs)
6. ✅ **Verify interactions** with `verify()`
7. ✅ **Use @BeforeEach** for common setup
8. ✅ **Keep tests independent** (no shared state)
9. ✅ **Test one thing per test method**
10. ✅ **Use meaningful test data**

This comprehensive guide covers **90% of interview scenarios**. Practice these patterns and you'll be well-prepared! 🚀
