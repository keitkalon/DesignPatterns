# Optimization Design Patterns Comparison Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[Flyweight](#1-flyweight-pattern)** | 1 | 0 | 2+ | Shares common state among multiple objects to reduce memory |
| **[Prototype](#2-prototype-pattern)** | 1 | 0 | 2+ | Creates new objects by cloning existing ones |
| **[Singleton](#3-singleton-pattern)** | 0 | 0 | 1 | Ensures only one instance of a class exists |

---

## 1. FLYWEIGHT PATTERN

### Purpose
Minimizes memory usage by sharing as much data as possible with similar objects. Separates intrinsic (shared) state from extrinsic (unique) state.

### Components
- **Interfaces:** `ICharacter`
- **Abstract classes:** None
- **Concrete classes:** `Character` (flyweight), `CharacterFactory`

### C# Implementation

```csharp
// Flyweight interface
interface ICharacter
{
    void Display(int fontSize, string color);
}

// Concrete Flyweight - shares intrinsic state (the character value)
class Character : ICharacter
{
    private char symbol; // Intrinsic state (shared)

    public Character(char symbol)
    {
        this.symbol = symbol;
    }

    // Extrinsic state (fontSize, color) passed as parameters
    public void Display(int fontSize, string color)
    {
        Console.WriteLine($"Character '{symbol}' - Font: {fontSize}pt, Color: {color}");
    }
}

// Flyweight Factory - manages shared flyweights
class CharacterFactory
{
    private Dictionary<char, ICharacter> characters = new Dictionary<char, ICharacter>();

    public ICharacter GetCharacter(char symbol)
    {
        if (!characters.ContainsKey(symbol))
        {
            characters[symbol] = new Character(symbol);
            Console.WriteLine($"Creating new flyweight for '{symbol}'");
        }
        return characters[symbol];
    }

    public int GetTotalCharacters()
    {
        return characters.Count;
    }
}

// Usage
var factory = new CharacterFactory();

// Create text - many characters reused
var text = "HELLO WORLD";
foreach (char c in text)
{
    if (c != ' ')
    {
        var character = factory.GetCharacter(c);
        character.Display(12, "Black");
    }
}

Console.WriteLine($"\nTotal unique character objects created: {factory.GetTotalCharacters()}");
Console.WriteLine($"Total characters displayed: {text.Replace(" ", "").Length}");
Console.WriteLine("Memory saved by sharing common characters!");
```

### When to Use
- Application uses a large number of objects
- Storage costs are high because of the quantity of objects
- Most object state can be made extrinsic
- Many groups of objects can be replaced by relatively few shared objects
- Application doesn't depend on object identity

---

## 2. PROTOTYPE PATTERN

### Purpose
Creates new objects by copying (cloning) existing objects (prototypes) instead of creating them from scratch. Useful when object creation is expensive.

### Components
- **Interfaces:** `IPrototype`
- **Abstract classes:** None
- **Concrete classes:** `Circle`, `Rectangle`

### C# Implementation

```csharp
// Prototype interface
interface IPrototype
{
    IPrototype Clone();
    void Display();
}

// Concrete Prototype 1
class Circle : IPrototype
{
    public int Radius { get; set; }
    public string Color { get; set; }

    public Circle(int radius, string color)
    {
        Radius = radius;
        Color = color;
        Console.WriteLine($"Creating Circle - expensive operation!");
    }

    // Clone creates a copy without expensive initialization
    public IPrototype Clone()
    {
        Console.WriteLine("Cloning Circle - fast operation!");
        return (IPrototype)this.MemberwiseClone();
    }

    public void Display()
    {
        Console.WriteLine($"Circle: Radius={Radius}, Color={Color}");
    }
}

// Concrete Prototype 2
class Rectangle : IPrototype
{
    public int Width { get; set; }
    public int Height { get; set; }
    public string Color { get; set; }

    public Rectangle(int width, int height, string color)
    {
        Width = width;
        Height = height;
        Color = color;
        Console.WriteLine($"Creating Rectangle - expensive operation!");
    }

    public IPrototype Clone()
    {
        Console.WriteLine("Cloning Rectangle - fast operation!");
        return (IPrototype)this.MemberwiseClone();
    }

    public void Display()
    {
        Console.WriteLine($"Rectangle: Width={Width}, Height={Height}, Color={Color}");
    }
}

// Usage
var originalCircle = new Circle(10, "Red");
originalCircle.Display();

// Clone instead of creating new
var clonedCircle1 = (Circle)originalCircle.Clone();
clonedCircle1.Display();

var clonedCircle2 = (Circle)originalCircle.Clone();
clonedCircle2.Radius = 20; // Modify clone
clonedCircle2.Color = "Blue";
clonedCircle2.Display();

Console.WriteLine("\nOriginal circle unchanged:");
originalCircle.Display();

var originalRectangle = new Rectangle(5, 10, "Green");
var clonedRectangle = (Rectangle)originalRectangle.Clone();
clonedRectangle.Width = 15;
clonedRectangle.Display();
```

### When to Use
- Object creation is expensive (complex initialization, database access, etc.)
- You need to avoid subclasses of an object creator
- Classes to instantiate are specified at runtime
- You want to keep the number of classes in a system to a minimum
- Creating an object is more convenient using cloning than using new

---

## 3. SINGLETON PATTERN

### Purpose
Ensures a class has only one instance and provides a global point of access to it. Useful for managing shared resources.

### Components
- **Interfaces:** None
- **Abstract classes:** None
- **Concrete classes:** `Logger` (singleton)

### C# Implementation

```csharp
// Singleton class
class Logger
{
    // Static instance (the single instance)
    private static Logger instance = null;
    private static readonly object lockObject = new object();

    // Private constructor prevents external instantiation
    private Logger()
    {
        Console.WriteLine("Logger instance created");
    }

    // Public static method to get the single instance
    public static Logger GetInstance()
    {
        if (instance == null)
        {
            lock (lockObject) // Thread-safe
            {
                if (instance == null)
                {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }

    public void Log(string message)
    {
        Console.WriteLine($"[LOG] {DateTime.Now:HH:mm:ss} - {message}");
    }
}

// Usage
var logger1 = Logger.GetInstance();
logger1.Log("Application started");

var logger2 = Logger.GetInstance();
logger2.Log("User logged in");

// Verify both are the same instance
Console.WriteLine($"\nAre logger1 and logger2 the same instance? {object.ReferenceEquals(logger1, logger2)}");
```

### Alternative Implementation - Eager Initialization

```csharp
class DatabaseConnection
{
    // Eager initialization - created immediately
    private static readonly DatabaseConnection instance = new DatabaseConnection();

    private DatabaseConnection()
    {
        Console.WriteLine("Database connection established");
    }

    public static DatabaseConnection GetInstance()
    {
        return instance;
    }

    public void Query(string sql)
    {
        Console.WriteLine($"Executing: {sql}");
    }
}

// Usage
var db1 = DatabaseConnection.GetInstance();
db1.Query("SELECT * FROM Users");

var db2 = DatabaseConnection.GetInstance();
db2.Query("INSERT INTO Logs VALUES (...)");

Console.WriteLine($"Same connection? {object.ReferenceEquals(db1, db2)}");
```

### Alternative Implementation - Lazy<T> (Modern C#)

```csharp
class Configuration
{
    private static readonly Lazy<Configuration> instance = 
        new Lazy<Configuration>(() => new Configuration());

    private Configuration()
    {
        Console.WriteLine("Loading configuration...");
    }

    public static Configuration GetInstance()
    {
        return instance.Value;
    }

    public void ShowConfig()
    {
        Console.WriteLine("App Name: MyApp v1.0");
    }
}

// Usage
var config = Configuration.GetInstance();
config.ShowConfig();
```

### When to Use
- There must be exactly one instance of a class
- The instance must be accessible from a well-known access point
- The sole instance should be extensible by subclassing
- You need to control access to a shared resource (database, file, etc.)
- You want to reduce global variable usage

---

## Key Differences Summary

### Memory Optimization Approaches

**Flyweight:**
- **What:** Shares common data across many objects
- **How:** Separates intrinsic (shared) from extrinsic (unique) state
- **Saves:** Memory by reusing objects
- **Example:** 1000 'A' characters share one Character('A') object

**Prototype:**
- **What:** Clones existing objects instead of creating new ones
- **How:** Copies object state using clone operation
- **Saves:** Time by avoiding expensive initialization
- **Example:** Clone a complex object instead of reconstructing it

**Singleton:**
- **What:** Ensures only one instance exists
- **How:** Private constructor with static instance
- **Saves:** Memory by having single instance
- **Example:** Only one Logger or DatabaseConnection for entire app

### Creation Pattern Comparison

| Aspect | Flyweight | Prototype | Singleton |
|--------|-----------|-----------|-----------|
| **Number of instances** | Many (shared) | Many (cloned) | One |
| **Purpose** | Share state | Clone objects | Single instance |
| **Memory goal** | Reduce total memory | N/A | Single instance memory |
| **Creation cost** | Low (reuse) | Low (clone) | Once only |
| **Mutability** | Typically immutable | Mutable clones | Single mutable instance |

### When to Choose

**Choose Flyweight when:**
- You have many similar objects
- Most object state can be made extrinsic
- Memory is a primary concern

**Choose Prototype when:**
- Object creation is expensive
- You need variations of similar objects
- You want to avoid complex initialization

**Choose Singleton when:**
- Only one instance should exist
- You need global access to that instance
- You're managing a shared resource

### Common Misconceptions

**Flyweight is NOT:**
- Object pooling (though related concept)
- Just caching objects
- About reducing creation time (it's about memory)

**Prototype is NOT:**
- Just using copy constructors
- The same as shallow copying (should be deep copy usually)
- A replacement for Singleton

**Singleton is NOT:**
- Always the best solution (can make testing difficult)
- Just a global variable (provides controlled access)
- Thread-safe by default (needs proper implementation)

### Optimization Summary

All three patterns optimize resource usage but in different ways:

- **Flyweight:** Optimizes memory through sharing
- **Prototype:** Optimizes creation time through cloning
- **Singleton:** Optimizes existence through single instance

These patterns follow different principles but all aim to make applications more efficient in terms of memory, performance, or both.
