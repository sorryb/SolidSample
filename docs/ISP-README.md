# ISP - Interface Segregation Principle

> A big interface should be *segregatted* in small ones. A class that implements an interface should not be force to implement methods it don't use.


# Interface Segregation Principle (ISP)

The **Interface Segregation Principle** is the **"I"** in the **SOLID** software design principles. It states:

> **"Clients should not be forced to depend upon interfaces that they do not use."**
> **A big interface should be *segregatted* in small ones. A class that implements an interface should not be force to implement methods it don't use.
---

### Key Concepts

* **Fat / Bloated Interfaces**: Interfaces that define too many methods, forcing implementing classes to write boilerplate or dummy implementations for behaviors they don't actually support.
* **Role Interfaces**: Small, focused interfaces designed around specific behaviors (e.g., `IPrinter`, `IScanner`, `IFaxService`) rather than large multipurpose interfaces.

---

### The Problem: Fat Interfaces (Violating ISP)

Consider a multi-function machine interface designed to handle printing, scanning, faxing, and stapling.

```csharp
// Fat Interface: Forces all implementations to handle every feature
public interface IMultiFunctionDevice
{
    void Print(string document);
    void Scan(string document);
    void Fax(string document);
    void Staple(string document);
}

// Basic Printer: Only capable of printing!
public class BasicPrinter : IMultiFunctionDevice
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }

    // Forced to implement methods it cannot perform:
    public void Scan(string document)
    {
        throw new NotImplementedException("BasicPrinter cannot scan!");
    }

    public void Fax(string document)
    {
        throw new NotImplementedException("BasicPrinter cannot fax!");
    }

    public void Staple(string document)
    {
        throw new NotImplementedException("BasicPrinter cannot staple!");
    }
}
```

#### Why This Violates ISP

1. **Forced `NotImplementedException`**: Clients using `IMultiFunctionDevice` might call `Scan()` on a `BasicPrinter`, causing unexpected runtime crashes.
2. **Fragile Code**: Adding a new method to `IMultiFunctionDevice` forces **every** implementing class across the application to be modified, even if they don't use the feature.
3. **Leaky Abstractions**: High-level modules calling `IMultiFunctionDevice` carry unnecessary dependencies on features they don't need.

---

### The Refactored Solution (Adhering to ISP)

Break down the large interface into smaller, focused interfaces. Classes only implement what they actually support, and multi-function devices can compose multiple interfaces together.

```csharp
// 1. Segregated, Role-Based Interfaces
public interface IPrinter
{
    void Print(string document);
}

public interface IScanner
{
    void Scan(string document);
}

public interface IFaxMachine
{
    void Fax(string document);
}

public interface IStapler
{
    void Staple(string document);
}

// 2. Simple Device: Only implements what it supports
public class SimplePrinter : IPrinter
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing document: {document}");
    }
}

// 3. High-End Device: Implements multiple specific interfaces
public class SuperPrinter : IPrinter, IScanner, IFaxMachine
{
    public void Print(string document) => Console.WriteLine($"SuperPrinter printing: {document}");
    public void Scan(string document) => Console.WriteLine($"SuperPrinter scanning: {document}");
    public void Fax(string document) => Console.WriteLine($"SuperPrinter faxing: {document}");
}

// 4. Combined Interface (Optional): Useful when a client explicitly needs all behaviors
public interface IMultiFunctionMachine : IPrinter, IScanner, IFaxMachine { }
```

---

### Usage & Client Separation

Clients depend only on the specific interface they need, ensuring safe and predictable execution:

```csharp
public class PrintJobManager
{
    private readonly IPrinter _printer;

    // Only requires printing capabilities; accepts SimplePrinter or SuperPrinter safely
    public PrintJobManager(IPrinter printer)
    {
        _printer = printer;
    }

    public void Process(string doc)
    {
        _printer.Print(doc);
    }
}
```

---

### Summary of Benefits

| Benefit | How ISP Achieves It |
| :--- | :--- |
| **No Runtime Surprises** | Eliminates forced `NotImplementedException` calls in client code. |
| **High Cohesion** | Interfaces remain single-purpose, focused, and lean. |
| **Low Coupling** | Changing or adding methods to one interface doesn't force unrelated classes to recompile or update. |



## Resources

- [Check with fiddle ](https://dotnetfiddle.net/)
- [Simplifying the Liskov Substitution Principle of SOLID in C#](https://www.infragistics.com/community/blogs/b/dhananjay_kumar/posts/simplifying-the-liskov-substitution-principle-of-solid-in-c)
- [The Liskov Substitution Principle](https://cleancoders.com/episode/clean-code-episode-11-p2)