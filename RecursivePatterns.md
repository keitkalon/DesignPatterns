# Recursive Design Patterns Comparison Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[Decorator](#1-decorator-pattern)** | 1 | 0-1 | 3+ | Wraps objects recursively to add responsibilities dynamically |
| **[Composite](#2-composite-pattern)** | 1 | 0-1 | 2+ | Tree structure where nodes contain other nodes recursively |

---

## 1. DECORATOR PATTERN

### Purpose
Attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality. Wraps objects recursively, each layer adding new behavior.

### Components
- **Interfaces:** `ICoffee`
- **Abstract classes:** `CoffeeDecorator` (optional, provides base for decorators)
- **Concrete classes:** `SimpleCoffee`, `MilkDecorator`, `SugarDecorator`, `WhipDecorator`

### C# Implementation

```csharp
// Component interface
interface ICoffee
{
    string GetDescription();
    double GetCost();
}

// Concrete Component - the base object
class SimpleCoffee : ICoffee
{
    public string GetDescription()
    {
        return "Simple coffee";
    }

    public double GetCost()
    {
        return 2.00;
    }
}

// Base Decorator (optional but recommended)
abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee coffee;

    public CoffeeDecorator(ICoffee coffee)
    {
        this.coffee = coffee;
    }

    public virtual string GetDescription()
    {
        return coffee.GetDescription();
    }

    public virtual double GetCost()
    {
        return coffee.GetCost();
    }
}

// Concrete Decorator 1
class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", milk";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.50;
    }
}

// Concrete Decorator 2
class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", sugar";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.25;
    }
}

// Concrete Decorator 3
class WhipDecorator : CoffeeDecorator
{
    public WhipDecorator(ICoffee coffee) : base(coffee)
    {
    }

    public override string GetDescription()
    {
        return coffee.GetDescription() + ", whipped cream";
    }

    public override double GetCost()
    {
        return coffee.GetCost() + 0.75;
    }
}

// Usage - demonstrating recursive wrapping
ICoffee coffee = new SimpleCoffee();
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

// Wrap with milk
coffee = new MilkDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

// Wrap with sugar
coffee = new SugarDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

// Wrap with whip
coffee = new WhipDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");

// Or all at once - showing recursive nature
ICoffee fancyCoffee = new WhipDecorator(
    new SugarDecorator(
        new MilkDecorator(
            new SimpleCoffee()
        )
    )
);
Console.WriteLine($"{fancyCoffee.GetDescription()} = ${fancyCoffee.GetCost()}");
```

### Alternative Implementation Without Base Decorator

```csharp
// Component interface
interface INotifier
{
    void Send(string message);
}

// Concrete Component
class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

// Concrete Decorator - directly implements interface
class SMSDecorator : INotifier
{
    private INotifier notifier;

    public SMSDecorator(INotifier notifier)
    {
        this.notifier = notifier;
    }

    public void Send(string message)
    {
        notifier.Send(message);
        Console.WriteLine($"SMS: {message}");
    }
}

// Another Concrete Decorator
class SlackDecorator : INotifier
{
    private INotifier notifier;

    public SlackDecorator(INotifier notifier)
    {
        this.notifier = notifier;
    }

    public void Send(string message)
    {
        notifier.Send(message);
        Console.WriteLine($"Slack: {message}");
    }
}

// Usage
INotifier notifier = new SlackDecorator(
    new SMSDecorator(
        new EmailNotifier()
    )
);
notifier.Send("Server is down!");
```

### When to Use
- You want to add responsibilities to individual objects dynamically and transparently
- Responsibilities can be withdrawn
- Extension by subclassing is impractical (too many combinations)
- You want to add functionality without modifying existing code
- You need to add multiple independent features in various combinations

---

## 2. COMPOSITE PATTERN

### Purpose
Composes objects into tree structures to represent part-whole hierarchies. Lets clients treat individual objects and compositions of objects uniformly. Uses recursive composition where containers hold both leaves and other containers.

### Components
- **Interfaces:** `IFileSystemComponent`
- **Abstract classes:** `FileSystemComponent` (optional, can use interface only)
- **Concrete classes:** `File` (leaf), `Directory` (composite)

### C# Implementation

```csharp
// Component interface
interface IFileSystemComponent
{
    string GetName();
    int GetSize();
    void Display(int depth = 0);
}

// Leaf - represents individual objects (files)
class File : IFileSystemComponent
{
    private string name;
    private int size;

    public File(string name, int size)
    {
        this.name = name;
        this.size = size;
    }

    public string GetName()
    {
        return name;
    }

    public int GetSize()
    {
        return size;
    }

    public void Display(int depth = 0)
    {
        Console.WriteLine(new string('-', depth) + " " + name + " (" + size + " KB)");
    }
}

// Composite - represents containers (directories)
class Directory : IFileSystemComponent
{
    private string name;
    private List<IFileSystemComponent> children = new List<IFileSystemComponent>();

    public Directory(string name)
    {
        this.name = name;
    }

    public void Add(IFileSystemComponent component)
    {
        children.Add(component);
    }

    public void Remove(IFileSystemComponent component)
    {
        children.Remove(component);
    }

    public string GetName()
    {
        return name;
    }

    // Recursive method - calculates size of all children
    public int GetSize()
    {
        int totalSize = 0;
        foreach (var child in children)
        {
            totalSize += child.GetSize();
        }
        return totalSize;
    }

    // Recursive method - displays tree structure
    public void Display(int depth = 0)
    {
        Console.WriteLine(new string('-', depth) + "+ " + name);
        foreach (var child in children)
        {
            child.Display(depth + 2);
        }
    }
}

// Usage - building a tree structure
var root = new Directory("root");

var home = new Directory("home");
var user = new Directory("user");
user.Add(new File("photo1.jpg", 250));
user.Add(new File("photo2.jpg", 180));
user.Add(new File("document.pdf", 420));

var documents = new Directory("documents");
documents.Add(new File("resume.docx", 35));
documents.Add(new File("cover_letter.pdf", 22));

user.Add(documents);
home.Add(user);

var system = new Directory("system");
system.Add(new File("config.sys", 5));
system.Add(new File("boot.ini", 3));

root.Add(home);
root.Add(system);

// Display the entire tree
root.Display();

// Calculate total size recursively
Console.WriteLine($"\nTotal size: {root.GetSize()} KB");
```

### Alternative Implementation with Abstract Base Class

```csharp
// Abstract Component
abstract class MenuComponent
{
    protected string name;

    public MenuComponent(string name)
    {
        this.name = name;
    }

    public virtual void Add(MenuComponent component)
    {
        throw new NotSupportedException();
    }

    public virtual void Remove(MenuComponent component)
    {
        throw new NotSupportedException();
    }

    public abstract void Display(int depth = 0);
}

// Leaf
class MenuItem : MenuComponent
{
    private double price;

    public MenuItem(string name, double price) : base(name)
    {
        this.price = price;
    }

    public override void Display(int depth = 0)
    {
        Console.WriteLine(new string(' ', depth) + name + " - $" + price);
    }
}

// Composite
class Menu : MenuComponent
{
    private List<MenuComponent> items = new List<MenuComponent>();

    public Menu(string name) : base(name)
    {
    }

    public override void Add(MenuComponent component)
    {
        items.Add(component);
    }

    public override void Remove(MenuComponent component)
    {
        items.Remove(component);
    }

    public override void Display(int depth = 0)
    {
        Console.WriteLine(new string(' ', depth) + name);
        foreach (var item in items)
        {
            item.Display(depth + 2);
        }
    }
}

// Usage
var mainMenu = new Menu("Main Menu");

var breakfast = new Menu("Breakfast");
breakfast.Add(new MenuItem("Pancakes", 7.99));
breakfast.Add(new MenuItem("Omelette", 8.99));

var lunch = new Menu("Lunch");
lunch.Add(new MenuItem("Burger", 12.99));
lunch.Add(new MenuItem("Salad", 9.99));

mainMenu.Add(breakfast);
mainMenu.Add(lunch);
mainMenu.Add(new MenuItem("Coffee", 2.99));

mainMenu.Display();
```

### When to Use
- You want to represent part-whole hierarchies of objects
- You want clients to ignore the difference between compositions and individual objects
- The structure can be represented as a tree
- You need to perform operations recursively on a tree structure
- You want a uniform interface for both simple and complex elements

---

## Key Differences Summary

### Decorator vs Composite

**Purpose:**
- **Decorator:** Adds responsibilities/features to individual objects dynamically
- **Composite:** Organizes objects into tree structures to represent hierarchies

**Recursion Type:**
- **Decorator:** Linear recursion (chain/wrapper) - decorators wrap other decorators
- **Composite:** Tree recursion (hierarchical) - composites contain leaves and other composites

**Structure:**
- **Decorator:** One-to-one wrapping relationship
- **Composite:** One-to-many containment relationship

**Intent:**
- **Decorator:** Enhance or modify behavior
- **Composite:** Group objects and treat them uniformly

**Typical Use Cases:**
- **Decorator:** UI components with borders/scrollbars, I/O streams, text formatting
- **Composite:** File systems, organization charts, GUI containers, menu systems

### Recursive Nature

Both patterns use recursion but differently:

**Decorator:**
```csharp
// Linear chain of wrappers
result = Decorator3(Decorator2(Decorator1(Component)))
// Each decorator calls the wrapped object's method
```

**Composite:**
```csharp
// Tree structure with branches
Composite
├── Leaf
├── Leaf
└── Composite
    ├── Leaf
    └── Leaf
// Each composite calls methods on all children recursively
```

### When to Choose

- Choose **Decorator** when you need to add optional features/responsibilities to objects
- Choose **Composite** when you need to represent hierarchical structures and treat parts and wholes uniformly

Both patterns follow the Open/Closed Principle and make extensive use of composition over inheritance.
