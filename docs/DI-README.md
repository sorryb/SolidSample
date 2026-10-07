# DI - Dependency Inversion Principle

> High-level modules(business or domain modules) should not depend on low-level modules(logging or file system modules). Both should depend on abstractions.

Abstraction means:

1. Interface

2. Abstract base clases.

## How to implement DI

1.First, to make sure that your higher-level classes depend on abstractions, not implementation details. 

2.Then be sure that these abstractions don't leak details. They shouldn't care about how they're implemented. 

3.Be sure to write your classes so they are explicit about what they need, and clients should inject these classes' dependencies when they create them, perhaps using a container to help with this. 

4.Finally, use folders in an appropriately structured solution to leverage dependency inversion and produce code that's loosely coupled and easy to maintain and test. 

### Key Concepts

* **High-Level Module**: Business logic or policy decision code (e.g., `OrderProcessor`, `NotificationService`).
* **Low-Level Module**: Infrastructure or implementation detail code (e.g., `SqlDatabaseRepository`, `SmtpEmailSender`, `FileLogger`).
* **Abstraction**: Interfaces or abstract classes (`IMessageSender`, `IRepository`).

---

### The Problem: Tightly Coupled Code (Violating DIP)

In traditional layered architecture, high-level business logic instantiates low-level infrastructure directly using the `new` keyword.

```csharp
// Low-Level Module (Infrastructure detail)
public class SqlDataStore
{
    public void SaveOrder(string orderId)
    {
        Console.WriteLine($"Saved order {orderId} to SQL Database.");
    }
}

// Low-Level Module (Infrastructure detail)
public class SmtpEmailSender
{
    public void SendEmail(string message)
    {
        Console.WriteLine($"Sent SMTP email: {message}");
    }
}

// High-Level Module (Business logic)
public class OrderProcessor
{
    private readonly SqlDataStore _dataStore;
    private readonly SmtpEmailSender _emailSender;

    public OrderProcessor()
    {
        // Direct instantiation couples high-level logic to concrete implementations!
        _dataStore = new SqlDataStore();
        _emailSender = new SmtpEmailSender();
    }

    public void Process(string orderId)
    {
        _dataStore.SaveOrder(orderId);
        _emailSender.SendEmail($"Order {orderId} processed successfully.");
    }
}
```

#### Why This Violates DIP

1. **Tight Coupling**: `OrderProcessor` cannot exist without `SqlDataStore` and `SmtpEmailSender`.
2. **Untestable**: Unit testing `OrderProcessor` in isolation is impossible because it forces actual database writes and SMTP calls.
3. **Fragile**: Switching from SQL to MongoDB, or from SMTP to SendGrid/Twilio SMS, requires modifying `OrderProcessor`.

---

### The Refactored Solution (Adhering to DIP)

We invert the dependencies by introducing interfaces owned by the high-level domain layer. Both high-level and low-level modules now depend on these abstractions.

```csharp
// 1. Define Abstractions
public interface IOrderRepository
{
    void Save(string orderId);
}

public interface INotificationService
{
    void Notify(string message);
}

// 2. Low-Level Implementation: Database
public class SqlOrderRepository : IOrderRepository
{
    public void Save(string orderId) =>
        Console.WriteLine($"Saved order {orderId} to SQL Database.");
}

public class MongoOrderRepository : IOrderRepository
{
    public void Save(string orderId) =>
        Console.WriteLine($"Saved order {orderId} to MongoDB.");
}

// 3. Low-Level Implementation: Notifications
public class EmailNotificationService : INotificationService
{
    public void Notify(string message) =>
        Console.WriteLine($"Email notification: {message}");
}

public class SmsNotificationService : INotificationService
{
    public void Notify(string message) =>
        Console.WriteLine($"SMS notification: {message}");
}

// 4. High-Level Module: Depends ONLY on Abstractions via Constructor Injection
public class OrderProcessor
{
    private readonly IOrderRepository _repository;
    private readonly INotificationService _notifier;

    public OrderProcessor(IOrderRepository repository, INotificationService notifier)
    {
        _repository = repository;
        _notifier = notifier;
    }

    public void Process(string orderId)
    {
        _repository.Save(orderId);
        _notifier.Notify($"Order {orderId} processed successfully.");
    }
}
```

---

### Usage & Composition Root

Dependencies are wired together at application startup (Composition Root / .NET Dependency Injection Container):

```csharp
// Program.cs / Application Startup
IOrderRepository repository = new MongoOrderRepository(); // Swap implementations easily
INotificationService notifier = new SmsNotificationService();

var processor = new OrderProcessor(repository, notifier);
processor.Process("ORD-1001");
```

---

### Summary of Benefits

| Benefit | How DIP Achieves It |
| :--- | :--- |
| **Testability** | You can inject mock implementations (`Mock<IOrderRepository>`) during unit testing. |
| **Flexibility** | Infrastructure details can be swapped out with zero changes to business logic. |
| **Parallel Development** | Teams can build business logic against an interface while the infrastructure team implements the concrete persistence mechanism. |

## Resources

- [Check with fiddle](https://dotnetfiddle.net/)
- [Strategy Design Pattern](https://www.tutorialspoint.com/design_pattern/strategy_pattern.htm)

### References

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com.br/dp/0201633612/?coliid=I3BZ6YLWVQOODS&colid=33HSVS6YEB9GQ&psc=1&ref_=lv_ov_lig_dp_it_im)
- [Clean Architecture](https://www.amazon.com.br/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=pd_bxgy_img_1/146-6852552-2489063?pd_rd_w=zjy9c&pf_rd_p=4a943320-02ab-4775-ad7a-eaf57d00a244&pf_rd_r=ZKKP8CPB3JEAT1YGKPZE&pd_rd_r=6bf3a408-31a9-4080-9645-7b48a056ffa4&pd_rd_wg=NbIDx&pd_rd_i=0134494164&psc=1)
- [The Clean Architecture (The Clean Code Blog)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)