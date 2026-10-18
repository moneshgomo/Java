
# Java OOP, Spring AOP & Transaction Management
### Learning and Revision Notes

---

# 1. Abstract Methods

An abstract method declares WHAT needs to be done but does not specify HOW to do it.

An abstract method has no method body.

```java
abstract class Payment {
    abstract void pay();
}
```

A concrete child class must implement all remaining abstract methods.

```java
class Gpay extends Payment {

    @Override
    void pay() {
        System.out.println("Payment via GPay");
    }
}
```

### Important Rules

- A class containing an abstract method must be declared abstract.
- An abstract class can contain both abstract and concrete methods.
- An abstract class cannot be instantiated directly.
- An abstract class does not necessarily need abstract methods.
- An abstract child class can leave inherited abstract methods unimplemented.

---

# 2. Abstract Method vs Method Overriding

## Abstract Method

Parent declares a method without implementation.

```java
abstract class Payment {
    abstract void pay();
}
```

## Method Overriding

A child class provides its own implementation of an inherited method.

```java
class Payment {
    void pay() {
        System.out.println("Default Payment");
    }
}

class Gpay extends Payment {

    @Override
    void pay() {
        System.out.println("GPay Payment");
    }
}
```

### Difference

| Abstract Method | Method Overriding |
|---|---|
| Declares a method without implementation | Provides an implementation of an inherited method |
| Defines a contract | Customizes inherited behaviour |
| Concrete child must implement it if no implementation is inherited | Overriding an existing concrete method is generally optional |

Important: Implementing an inherited abstract method is also a form of method overriding.

---

# 3. Interface in Java

An interface defines a contract that implementing classes must follow.

```java
interface Payment {
    void pay();
    void refund();
}
```

Implementation:

```java
class Gpay implements Payment {

    @Override
    public void pay() {
        System.out.println("Payment successful");
    }

    @Override
    public void refund() {
        System.out.println("Refund successful");
    }
}
```

### Important Rules

- A class uses `implements` to implement an interface.
- A concrete class must implement or inherit implementations for all abstract interface methods.
- Interfaces cannot be instantiated directly.
- Interfaces are implicitly abstract.
- Interface methods without bodies are implicitly public abstract (unless declared otherwise, such as private methods with bodies).
- Interfaces can contain default, static and private methods.

Both declarations are valid:

```java
interface Payment {
    void pay();
}
```

```java
abstract interface Payment {
    public abstract void pay();
}
```

Explicitly writing `abstract` is redundant.

---

# 4. Class vs Interface

| Feature | Class | Interface |
|---|---|---|
| Direct instantiation | Yes, if concrete | No |
| Constructors | Yes | No |
| Instance variables | Yes | No |
| Abstract methods | Only in abstract classes | Yes |
| Implemented methods | Yes | Yes, with applicable method types |
| Multiple inheritance | Cannot extend multiple classes | Can extend multiple interfaces |

### When to Use an Interface

When different classes must follow the same contract.

Example:

```java
interface Payment {
    void pay();
}

class Gpay implements Payment {
    public void pay() {
        System.out.println("GPay");
    }
}

class CardPayment implements Payment {
    public void pay() {
        System.out.println("Card");
    }
}
```

### When to Use an Abstract Class

When related classes share common state or implementation but also require customized behaviour.

```java
abstract class Delivery {

    void validateAddress() {
        System.out.println("Address Validated");
    }

    abstract double calculateCharge();
}
```

```java
class ExpressDelivery extends Delivery {

    @Override
    double calculateCharge() {
        return 100;
    }
}
```

---

# 5. Extends vs Implements

| Scenario | Keyword |
|---|---|
| Class inherits another class | extends |
| Class implements an interface | implements |
| Interface inherits another interface | extends |

A class can extend one class and implement multiple interfaces.

```java
class Gpay extends Transaction implements Payment, Refund {
    // Implement required methods
}
```

---

# 6. Java Access Modifiers

Java has four access levels.

| Modifier | Accessibility |
|---|---|
| public | Everywhere |
| protected | Same package + eligible subclasses in other packages |
| default | Same package only |
| private | Declaring class only |

Note: Default (package-private) access is applied by omitting the modifier.

Example:

```java
public class Employee {

    public String name;

    protected int salary;

    String department; // Default

    private String password;
}
```

### Protected vs Default

Default:
- Accessible within the same package.
- Not accessible from another package, even through inheritance.

Protected:
- Accessible within the same package.
- Also accessible through inheritance from another package, subject to Java's protected-access rules.

Example:

```java
public class Manager extends Employee {

    void display() {
        System.out.println(salary); // Allowed
    }
}
```

---

# 7. What is Spring AOP?

AOP stands for Aspect-Oriented Programming.

Spring AOP allows additional functionality to be applied around selected method executions without manually mixing that functionality into the core business logic.

Common examples:

- Logging
- Transaction management
- Method-level security
- Caching
- Performance monitoring

These are called Cross-Cutting Concerns.

---

# 8. What is a Cross-Cutting Concern?

A cross-cutting concern is common supporting functionality required across different parts of an application.

Example:

EmployeeService:
- Create Employee
- Update Employee

DepartmentService:
- Create Department
- Delete Department

Both services may require:
- Transaction management
- Logging
- Security

Instead of repeatedly writing this supporting functionality inside business methods, we can separate it using AOP-based mechanisms.

---

# 9. Important Spring AOP Terminology

| Term | Meaning |
|---|---|
| Aspect | Encapsulates cross-cutting functionality |
| Advice | Additional behaviour executed at a join point |
| Join Point | A point where advice can execute |
| Pointcut | Expression/rule selecting join points |
| Target | Actual object containing business logic |
| Proxy | Object intercepting calls to the target |
| Weaving | Associating aspects with application code |

Spring AOP uses runtime proxies and supports method-execution join points.

### Advice Types

- @Before
- @After
- @AfterReturning
- @AfterThrowing
- @Around

Example:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore() {
        System.out.println("Method executing");
    }
}
```

---

# 10. How Transactions Come Under AOP

Transaction management is a cross-cutting concern because it involves additional behaviour around business methods.

Without declarative transactions:

```java
beginTransaction();

try {
    saveEmployee();
    assignDepartment();

    commitTransaction();

} catch (Exception e) {
    rollbackTransaction();
    throw e;
}
```

With Spring:

```java
@Transactional
public void createEmployee() {

    employeeRepository.save(employee);

    departmentRepository.assignDepartment(employee);
}
```

Spring handles transaction management using its proxy-based interception infrastructure.

### Internal Flow

Controller
    |
    v
Spring AOP Proxy
    |
    v
TransactionInterceptor
    |
    v
TransactionManager
    |
    v
Begin / Join Transaction
    |
    v
Execute Business Method
    |
    +---- Success ---> Commit
    |
    +---- Failure ---> Rollback (according to rollback rules)

Important:

@Transactional is not itself an Aspect.

It supplies transaction metadata that Spring's transaction infrastructure uses.

---

# 11. @Transactional and Lazy Loading

Lazy loading means Hibernate delays fetching related data until it is actually accessed.

Example:

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Employee> employees;
```

Without an appropriate open Hibernate Session:

```java
Team team = teamRepository.findById(id).get();

team.getEmployees().size();
```

This may produce:

LazyInitializationException: could not initialize proxy - no Session

### Solution

```java
@Transactional(readOnly = true)
public void getTeam(Long id) {

    Team team = teamRepository.findById(id).get();

    team.getEmployees().size();
}
```

The transaction-scoped persistence context remains available throughout the service method.

Important:

Hibernate Session and JDBC Connection are not the same thing.

@Transactional does not automatically fetch every lazy relationship.

Lazy loading must occur while an appropriate Session is available.

---

# 12. @Transactional vs readOnly = true

```java
@Transactional
```

Default readOnly value is false.

Used for read-write transactional operations.

```java
@Transactional(readOnly = true)
```

Indicates that the transaction is intended for reading.

With Hibernate, Spring can apply optimizations such as reduced dirty-checking overhead and adjusted flushing behaviour.

| Feature | Transactional | readOnly = true |
|---|---|---|
| Transaction boundary | Yes | Yes |
| Lazy loading support | Yes | Yes |
| Intended for writes | Yes | No |
| Read optimization | Normal | Possible |

Important:

readOnly = true is not a universal guarantee that SQL writes are prohibited.

---

# 13. Trade-offs of @Transactional

Adding many transactional annotations is not inherently bad.

The important factors are transaction scope and duration.

Potential problems:

1. Long-running transactions.
2. Holding database resources unnecessarily.
3. Lock contention.
4. External API calls inside transactions.
5. Unexpected rollback behaviour.
6. Unnecessary transactions around non-database operations.

### Bad Example

```java
@Transactional
public void generateReport() {

    List<Employee> employees = employeeRepository.findAll();

    generateLargePDF(employees);

    reportRepository.save(report);
}
```

Heavy PDF generation occurs inside the transaction.

### Better Approach

1. Fetch required data in a short transaction.
2. Convert entities into DTOs.
3. Generate the PDF outside the transaction.
4. Save the report result in another short transaction.

For long-running jobs, asynchronous processing can be considered.

@Async and @Transactional solve different problems:

- @Async: Executes work asynchronously.
- @Transactional: Manages database transaction boundaries.

---

# 14. Transaction Propagation

Default propagation:

```java
@Transactional(propagation = Propagation.REQUIRED)
```

REQUIRED means:

- Join an existing transaction if one exists.
- Otherwise, start a new transaction.

Multiple @Transactional annotations do not necessarily create multiple independent transactions.

---

# 15. @Transactional and Access Modifiers

In Spring Framework 6+, with default proxy-based transaction management:

| Modifier | Support |
|---|---|
| public | Yes |
| protected | Yes, with class-based proxies |
| default | Yes, with class-based proxies |
| private | No |

JDK interface-based proxies require public methods exposed through the proxied interface.

Recommended convention:

```java
@Service
public class EmployeeService {

    @Transactional
    public void createEmployee() {
        // Business logic
    }
}
```

Use public transactional service entry-point methods for predictable behaviour.

---

# 16. Self-Invocation Problem

Consider:

```java
@Service
public class EmployeeService {

    public void processEmployee() {
        saveEmployee();
    }

    @Transactional
    public void saveEmployee() {
        employeeRepository.save(employee);
    }
}
```

When processEmployee() calls saveEmployee() directly, the internal call bypasses Spring's AOP proxy.

Therefore, the @Transactional annotation on saveEmployee() is not intercepted through that internal call.

This is called Self-Invocation.

If processEmployee() already has an active transaction, the internal operation can still participate in it.

---

# 17. Final Revision Points

1. Abstract method = WHAT to do, without HOW.
2. Interface = Common contract.
3. Abstract class = Common base with optional shared implementation and abstract methods.
4. implements = Class implements an interface.
5. extends = Class inheritance or interface inheritance.
6. protected = Same package + eligible subclass access.
7. default = Same package only.
8. AOP = Separates cross-cutting concerns from core business logic.
9. Proxy = Intercepts calls and applies additional behaviour.
10. @Transactional = Declarative transaction management using Spring's transaction infrastructure.
11. TransactionInterceptor = Applies transaction behaviour around method execution.
12. Lazy loading requires an appropriate Hibernate Session.
13. readOnly = true indicates a read-oriented transaction.
14. Long-running transactions should generally be avoided.
15. Private methods cannot be transactionally intercepted by Spring AOP proxies.
16. Self-invocation bypasses proxy-based transactional advice.

## Key Interview Statement

Spring AOP enables separation of cross-cutting concerns from business logic through proxy-based method interception. Spring's declarative transaction management uses this mechanism to apply transaction boundaries around selected service methods, handling transaction creation, participation, commit and rollback according to the configured transaction rules.
