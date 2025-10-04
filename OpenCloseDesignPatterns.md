# Open/Close Design Patterns Comparison Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[Template Method](#1-template-method-pattern)** | 0 | 1 | 2+ | Abstract class defines algorithm skeleton, subclasses override steps |
| **[Strategy](#2-strategy-pattern)** | 1 | 0 | 3+ | Encapsulates algorithms, makes them interchangeable at runtime |
| **[Bridge](#3-bridge-pattern)** | 1-2 | 0-1 | 4+ | Separates abstraction from implementation, both can vary independently |
| **[Chain of Responsibility](#4-chain-of-responsibility-pattern)** | 0-1 | 0-1 | 2+ | Chain of handlers, each decides to process or pass to next |

---

## 1. TEMPLATE METHOD PATTERN

### Purpose
Defines the skeleton of an algorithm in an abstract class, letting subclasses override specific steps without changing the algorithm's structure.

### Components
- **Interfaces:** None
- **Abstract classes:** `DataProcessor` (defines template method)
- **Concrete classes:** `CSVProcessor`, `JSONProcessor`

### C# Implementation

```csharp
// Abstract class with template method
abstract class DataProcessor
{
    // Template method - defines the algorithm skeleton
    public void ProcessData()
    {
        ReadData();
        ProcessContent();
        ValidateData();
        SaveData();
    }

    // Steps that subclasses must implement
    protected abstract void ReadData();
    protected abstract void ProcessContent();
    
    // Steps with default implementation (can be overridden)
    protected virtual void ValidateData()
    {
        Console.WriteLine("Performing default validation...");
    }
    
    // Common step (cannot be overridden)
    private void SaveData()
    {
        Console.WriteLine("Saving processed data...");
    }
}

// Concrete implementation for CSV
class CSVProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Reading CSV file...");
    }

    protected override void ProcessContent()
    {
        Console.WriteLine("Parsing CSV content...");
    }

    protected override void ValidateData()
    {
        Console.WriteLine("Validating CSV format...");
    }
}

// Concrete implementation for JSON
class JSONProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Reading JSON file...");
    }

    protected override void ProcessContent()
    {
        Console.WriteLine("Parsing JSON content...");
    }
}

// Usage
DataProcessor csvProcessor = new CSVProcessor();
csvProcessor.ProcessData();

DataProcessor jsonProcessor = new JSONProcessor();
jsonProcessor.ProcessData();
```

### When to Use
- You want to let subclasses redefine certain steps of an algorithm without changing its structure
- Common behavior should be factored and localized in a common class
- You want to control which parts of an algorithm can be extended

---

## 2. STRATEGY PATTERN

### Purpose
Encapsulates a family of algorithms, makes them interchangeable, and lets the algorithm vary independently from clients that use it. Enables runtime selection of algorithms.

### Components
- **Interfaces:** `IPaymentStrategy`
- **Abstract classes:** None
- **Concrete classes:** `CreditCardPayment`, `PayPalPayment`, `CryptoPayment`, `ShoppingCart`

### C# Implementation

```csharp
// Strategy interface
interface IPaymentStrategy
{
    void Pay(decimal amount);
}

// Concrete Strategy 1
class CreditCardPayment : IPaymentStrategy
{
    private string cardNumber;
    private string cvv;

    public CreditCardPayment(string cardNumber, string cvv)
    {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using Credit Card ending in {cardNumber.Substring(cardNumber.Length - 4)}");
    }
}

// Concrete Strategy 2
class PayPalPayment : IPaymentStrategy
{
    private string email;

    public PayPalPayment(string email)
    {
        this.email = email;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using PayPal account {email}");
    }
}

// Concrete Strategy 3
class CryptoPayment : IPaymentStrategy
{
    private string walletAddress;

    public CryptoPayment(string walletAddress)
    {
        this.walletAddress = walletAddress;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using Crypto wallet {walletAddress}");
    }
}

// Context - uses a strategy
class ShoppingCart
{
    private IPaymentStrategy paymentStrategy;

    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        this.paymentStrategy = strategy;
    }

    public void Checkout(decimal amount)
    {
        if (paymentStrategy == null)
        {
            Console.WriteLine("Please select a payment method");
            return;
        }
        paymentStrategy.Pay(amount);
    }
}

// Usage
var cart = new ShoppingCart();

cart.SetPaymentStrategy(new CreditCardPayment("1234567890123456", "123"));
cart.Checkout(100.00m);

cart.SetPaymentStrategy(new PayPalPayment("user@example.com"));
cart.Checkout(50.00m);

cart.SetPaymentStrategy(new CryptoPayment("0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"));
cart.Checkout(75.00m);
```

### When to Use
- You have many related classes that differ only in their behavior
- You need different variants of an algorithm
- Algorithm uses data that clients shouldn't know about
- A class defines many behaviors with multiple conditional statements

---

## 3. BRIDGE PATTERN

### Purpose
Separates abstraction from implementation so both can vary independently. Prevents a Cartesian product complexity explosion when you have multiple dimensions of variation.

### Components
- **Interfaces:** `IRenderer` (implementation interface)
- **Abstract classes:** `Shape` (abstraction)
- **Concrete classes:** `VectorRenderer`, `RasterRenderer`, `Circle`, `Square`

### C# Implementation

```csharp
// Implementation interface
interface IRenderer
{
    void RenderCircle(float radius);
    void RenderSquare(float side);
}

// Concrete Implementation 1
class VectorRenderer : IRenderer
{
    public void RenderCircle(float radius)
    {
        Console.WriteLine($"Drawing circle with radius {radius} as vectors");
    }

    public void RenderSquare(float side)
    {
        Console.WriteLine($"Drawing square with side {side} as vectors");
    }
}

// Concrete Implementation 2
class RasterRenderer : IRenderer
{
    public void RenderCircle(float radius)
    {
        Console.WriteLine($"Drawing circle with radius {radius} as pixels");
    }

    public void RenderSquare(float side)
    {
        Console.WriteLine($"Drawing square with side {side} as pixels");
    }
}

// Abstraction
abstract class Shape
{
    protected IRenderer renderer;

    protected Shape(IRenderer renderer)
    {
        this.renderer = renderer;
    }

    public abstract void Draw();
    public abstract void Resize(float factor);
}

// Refined Abstraction 1
class Circle : Shape
{
    private float radius;

    public Circle(IRenderer renderer, float radius) : base(renderer)
    {
        this.radius = radius;
    }

    public override void Draw()
    {
        renderer.RenderCircle(radius);
    }

    public override void Resize(float factor)
    {
        radius *= factor;
    }
}

// Refined Abstraction 2
class Square : Shape
{
    private float side;

    public Square(IRenderer renderer, float side) : base(renderer)
    {
        this.side = side;
    }

    public override void Draw()
    {
        renderer.RenderSquare(side);
    }

    public override void Resize(float factor)
    {
        side *= factor;
    }
}

// Usage
var vectorRenderer = new VectorRenderer();
var rasterRenderer = new RasterRenderer();

var circle = new Circle(vectorRenderer, 5);
circle.Draw();
circle.Resize(2);
circle.Draw();

var square = new Square(rasterRenderer, 10);
square.Draw();
```

### When to Use
- You want to avoid permanent binding between abstraction and implementation
- Both abstraction and implementation should be extensible through subclassing
- Changes in implementation shouldn't impact clients
- You want to hide implementation from clients completely
- You have a proliferation of classes (Cartesian product problem)

---

## 4. CHAIN OF RESPONSIBILITY PATTERN

### Purpose
Passes a request along a chain of handlers. Each handler decides either to process the request or to pass it to the next handler in the chain.

### Components
- **Interfaces:** None (can use interface instead of abstract class)
- **Abstract classes:** `SupportHandler`
- **Concrete classes:** `Level1Support`, `Level2Support`, `Level3Support`

### C# Implementation

```csharp
// Abstract handler
abstract class SupportHandler
{
    protected SupportHandler nextHandler;

    public void SetNext(SupportHandler handler)
    {
        nextHandler = handler;
    }

    public abstract void HandleRequest(string request, int priority);
}

// Concrete Handler 1 - handles low priority
class Level1Support : SupportHandler
{
    public override void HandleRequest(string request, int priority)
    {
        if (priority <= 1)
        {
            Console.WriteLine($"Level 1 Support handled: {request}");
        }
        else if (nextHandler != null)
        {
            Console.WriteLine("Level 1 Support: Escalating to Level 2...");
            nextHandler.HandleRequest(request, priority);
        }
    }
}

// Concrete Handler 2 - handles medium priority
class Level2Support : SupportHandler
{
    public override void HandleRequest(string request, int priority)
    {
        if (priority <= 2)
        {
            Console.WriteLine($"Level 2 Support handled: {request}");
        }
        else if (nextHandler != null)
        {
            Console.WriteLine("Level 2 Support: Escalating to Level 3...");
            nextHandler.HandleRequest(request, priority);
        }
    }
}

// Concrete Handler 3 - handles high priority
class Level3Support : SupportHandler
{
    public override void HandleRequest(string request, int priority)
    {
        Console.WriteLine($"Level 3 Support handled: {request} (Priority {priority})");
    }
}

// Usage
var level1 = new Level1Support();
var level2 = new Level2Support();
var level3 = new Level3Support();

// Build the chain
level1.SetNext(level2);
level2.SetNext(level3);

// Send requests through the chain
level1.HandleRequest("Password reset", 1);
level1.HandleRequest("Software installation issue", 2);
level1.HandleRequest("Server down", 3);
```

### Alternative Implementation with Interface

```csharp
interface IHandler
{
    IHandler SetNext(IHandler handler);
    void Handle(string request);
}

abstract class AbstractHandler : IHandler
{
    private IHandler nextHandler;

    public IHandler SetNext(IHandler handler)
    {
        nextHandler = handler;
        return handler;
    }

    public virtual void Handle(string request)
    {
        if (nextHandler != null)
        {
            nextHandler.Handle(request);
        }
    }
}

class ConcreteHandler1 : AbstractHandler
{
    public override void Handle(string request)
    {
        if (request == "Type1")
        {
            Console.WriteLine("Handler1 processed the request");
        }
        else
        {
            base.Handle(request);
        }
    }
}
```

### When to Use
- More than one object may handle a request, and the handler isn't known a priori
- You want to issue a request to one of several objects without specifying the receiver explicitly
- The set of objects that can handle a request should be specified dynamically
- You want to avoid coupling the sender of a request to its receiver

---

## Key Differences Summary

### Template Method vs Strategy
- **Template Method:** Uses inheritance, algorithm structure fixed in abstract class, subclasses override steps
- **Strategy:** Uses composition, entire algorithm is replaceable, runtime switching possible

### Bridge vs Strategy
- **Bridge:** Separates two independent hierarchies (abstraction and implementation)
- **Strategy:** Focuses on making algorithms interchangeable within single hierarchy

### Chain of Responsibility vs Strategy
- **Chain of Responsibility:** Multiple handlers in sequence, request may be handled by any or none
- **Strategy:** Single algorithm selected and executed, always exactly one strategy

### All Four Patterns
- **Template Method:** Algorithm skeleton with customizable steps (inheritance-based)
- **Strategy:** Interchangeable algorithms (composition-based)
- **Bridge:** Decouples abstraction from implementation (two hierarchies)
- **Chain of Responsibility:** Sequential processing with dynamic handler selection

### Open/Closed Principle Connection
All these patterns follow the Open/Closed Principle (open for extension, closed for modification):
- **Template Method:** Extend by creating new subclasses
- **Strategy:** Extend by adding new strategy implementations
- **Bridge:** Extend either abstraction or implementation independently
- **Chain of Responsibility:** Extend by adding new handlers to the chain
