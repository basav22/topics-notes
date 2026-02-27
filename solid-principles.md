# SOLID Principles in Java

---

## 1. S — Single Responsibility Principle (SRP)

> A class should have only **one reason to change**.

**Bad:**

```java
class Employee {
    void calculateSalary() { /* ... */ }
    void saveToDatabase() { /* ... */ }
    void generateReport() { /* ... */ }
}
```

**Good:**

```java
class Employee {
    private String name;
    private double salary;
}

class SalaryCalculator {
    double calculate(Employee emp) { /* ... */ }
}

class EmployeeRepository {
    void save(Employee emp) { /* ... */ }
}

class ReportGenerator {
    void generate(Employee emp) { /* ... */ }
}
```

Each class has **one job**. If report format changes, only `ReportGenerator` changes.

---

## 2. O — Open/Closed Principle (OCP)

> Classes should be **open for extension, closed for modification**.

**Bad:**

```java
class DiscountCalculator {
    double calculate(String customerType, double amount) {
        if (customerType.equals("REGULAR")) return amount * 0.1;
        if (customerType.equals("PREMIUM")) return amount * 0.2;
        // Adding new type = modifying this class every time
        return 0;
    }
}
```

**Good:**

```java
interface DiscountStrategy {
    double calculate(double amount);
}

class RegularDiscount implements DiscountStrategy {
    public double calculate(double amount) { return amount * 0.1; }
}

class PremiumDiscount implements DiscountStrategy {
    public double calculate(double amount) { return amount * 0.2; }
}

// Adding new discount = just add a new class, no existing code changes
class FestiveDiscount implements DiscountStrategy {
    public double calculate(double amount) { return amount * 0.3; }
}
```

---

## 3. L — Liskov Substitution Principle (LSP)

> Subclass should be **substitutable** for its parent without breaking behavior.

**Bad:**

```java
class Bird {
    void fly() { System.out.println("Flying..."); }
}

class Penguin extends Bird {
    void fly() { throw new UnsupportedOperationException("Can't fly!"); }
    // Breaks code that expects all Birds to fly
}
```

**Good:**

```java
class Bird {
    void eat() { System.out.println("Eating..."); }
}

interface Flyable {
    void fly();
}

class Sparrow extends Bird implements Flyable {
    public void fly() { System.out.println("Flying..."); }
}

class Penguin extends Bird {
    void swim() { System.out.println("Swimming..."); }
    // No broken contract — Penguin never promises to fly
}
```

---

## 4. I — Interface Segregation Principle (ISP)

> Don't force classes to implement **interfaces they don't use**.

**Bad:**

```java
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work() { /* ok */ }
    public void eat() { /* Robot doesn't eat! Forced to implement */ }
    public void sleep() { /* Robot doesn't sleep! */ }
}
```

**Good:**

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class Human implements Workable, Eatable, Sleepable {
    public void work() { /* ... */ }
    public void eat() { /* ... */ }
    public void sleep() { /* ... */ }
}

class Robot implements Workable {
    public void work() { /* ... */ }
    // Only implements what it actually does
}
```

---

## 5. D — Dependency Inversion Principle (DIP)

> High-level modules should depend on **abstractions**, not concrete classes.

**Bad:**

```java
class MySQLDatabase {
    void save(String data) { /* save to MySQL */ }
}

class UserService {
    private MySQLDatabase db = new MySQLDatabase(); // tightly coupled
    void saveUser(String user) { db.save(user); }
}
// Switching to PostgreSQL = rewrite UserService
```

**Good:**

```java
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    public void save(String data) { /* MySQL logic */ }
}

class PostgreSQLDatabase implements Database {
    public void save(String data) { /* PostgreSQL logic */ }
}

class UserService {
    private Database db;

    UserService(Database db) { // injected via constructor
        this.db = db;
    }

    void saveUser(String user) { db.save(user); }
}

// Usage — easy to swap
UserService service = new UserService(new PostgreSQLDatabase());
```

---

## Quick Interview Cheat Sheet

| Principle | One-liner |
|-----------|-----------|
| **SRP** | One class = one job |
| **OCP** | Add new features without changing old code |
| **LSP** | Child class must honor parent's contract |
| **ISP** | Many small interfaces > one fat interface |
| **DIP** | Depend on abstractions, not concrete classes |
