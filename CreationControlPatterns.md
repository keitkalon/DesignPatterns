# Creation Control Design Patterns Comparison Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[Factory Method](#1-factory-method-pattern)** | 1 | 1 | 4+ | Subclasses decide which class to instantiate |
| **[Abstract Factory](#2-abstract-factory-pattern)** | 2-3 | 0 | 4+ | Creates families of related objects without specifying concrete classes |
| **[Builder](#3-builder-pattern)** | 1 | 0 | 3+ | Constructs complex objects step by step |

---

## 1. FACTORY METHOD PATTERN

### Purpose
Defines an interface for creating an object, but lets subclasses decide which class to instantiate. Delegates the responsibility of object creation to subclasses.

### Components
- **Interfaces:** `IShape`
- **Abstract classes:** `ShapeFactory`
- **Concrete classes:** `Circle`, `Square`, `CircleFactory`, `SquareFactory`

### C# Implementation

```csharp
// Product interface
interface IShape
{
    void Draw();
}

// Concrete Product 1
class Circle : IShape
{
    public void Draw()
    {
        Console.WriteLine("Drawing a Circle");
    }
}

// Concrete Product 2
class Square : IShape
{
    public void Draw()
    {
        Console.WriteLine("Drawing a Square");
    }
}

// Abstract Creator (Factory)
abstract class ShapeFactory
{
    // Factory Method - subclasses override this
    public abstract IShape CreateShape();

    // Template method using factory method
    public void RenderShape()
    {
        var shape = CreateShape();
        Console.WriteLine("Rendering shape:");
        shape.Draw();
    }
}

// Concrete Creator 1
class CircleFactory : ShapeFactory
{
    public override IShape CreateShape()
    {
        return new Circle();
    }
}

// Concrete Creator 2
class SquareFactory : ShapeFactory
{
    public override IShape CreateShape()
    {
        return new Square();
    }
}

// Usage
ShapeFactory factory1 = new CircleFactory();
factory1.RenderShape();

ShapeFactory factory2 = new SquareFactory();
factory2.RenderShape();

// Or directly create
IShape shape = new CircleFactory().CreateShape();
shape.Draw();
```

### When to Use
- A class can't anticipate the type of objects it needs to create
- A class wants its subclasses to specify the objects it creates
- Classes delegate responsibility to one of several helper subclasses
- You want to localize the knowledge of which class gets created

---

## 2. ABSTRACT FACTORY PATTERN

### Purpose
Provides an interface for creating families of related or dependent objects without specifying their concrete classes. Like a factory of factories.

### Components
- **Interfaces:** `IButton`, `ICheckbox`, `IGUIFactory`
- **Abstract classes:** None
- **Concrete classes:** `WindowsButton`, `MacButton`, `WindowsCheckbox`, `MacCheckbox`, `WindowsFactory`, `MacFactory`

### C# Implementation

```csharp
// Abstract Product A
interface IButton
{
    void Click();
}

// Abstract Product B
interface ICheckbox
{
    void Check();
}

// Concrete Product A1
class WindowsButton : IButton
{
    public void Click()
    {
        Console.WriteLine("Windows button clicked");
    }
}

// Concrete Product A2
class MacButton : IButton
{
    public void Click()
    {
        Console.WriteLine("Mac button clicked");
    }
}

// Concrete Product B1
class WindowsCheckbox : ICheckbox
{
    public void Check()
    {
        Console.WriteLine("Windows checkbox checked");
    }
}

// Concrete Product B2
class MacCheckbox : ICheckbox
{
    public void Check()
    {
        Console.WriteLine("Mac checkbox checked");
    }
}

// Abstract Factory
interface IGUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Concrete Factory 1
class WindowsFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new WindowsButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new WindowsCheckbox();
    }
}

// Concrete Factory 2
class MacFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new MacButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new MacCheckbox();
    }
}

// Client code
class Application
{
    private IButton button;
    private ICheckbox checkbox;

    public Application(IGUIFactory factory)
    {
        button = factory.CreateButton();
        checkbox = factory.CreateCheckbox();
    }

    public void Render()
    {
        button.Click();
        checkbox.Check();
    }
}

// Usage
IGUIFactory factory;

string os = "Windows"; // Could be determined at runtime

if (os == "Windows")
{
    factory = new WindowsFactory();
}
else
{
    factory = new MacFactory();
}

var app = new Application(factory);
app.Render();
```

### When to Use
- System should be independent of how its products are created
- System should be configured with one of multiple families of products
- Family of related product objects is designed to be used together
- You want to provide a class library of products, revealing only interfaces

---

## 3. BUILDER PATTERN

### Purpose
Separates the construction of a complex object from its representation, allowing the same construction process to create different representations. Builds objects step by step.

### Components
- **Interfaces:** `IPizzaBuilder`
- **Abstract classes:** None
- **Concrete classes:** `Pizza`, `MargheritaPizzaBuilder`, `PepperoniPizzaBuilder`, `PizzaDirector`

### C# Implementation

```csharp
// Product
class Pizza
{
    public string Dough { get; set; }
    public string Sauce { get; set; }
    public string Topping { get; set; }

    public void Display()
    {
        Console.WriteLine($"Pizza with {Dough} dough, {Sauce} sauce, and {Topping} topping");
    }
}

// Builder interface
interface IPizzaBuilder
{
    void BuildDough();
    void BuildSauce();
    void BuildTopping();
    Pizza GetPizza();
}

// Concrete Builder 1
class MargheritaPizzaBuilder : IPizzaBuilder
{
    private Pizza pizza = new Pizza();

    public void BuildDough()
    {
        pizza.Dough = "thin";
    }

    public void BuildSauce()
    {
        pizza.Sauce = "tomato";
    }

    public void BuildTopping()
    {
        pizza.Topping = "mozzarella";
    }

    public Pizza GetPizza()
    {
        return pizza;
    }
}

// Concrete Builder 2
class PepperoniPizzaBuilder : IPizzaBuilder
{
    private Pizza pizza = new Pizza();

    public void BuildDough()
    {
        pizza.Dough = "thick";
    }

    public void BuildSauce()
    {
        pizza.Sauce = "spicy tomato";
    }

    public void BuildTopping()
    {
        pizza.Topping = "pepperoni and cheese";
    }

    public Pizza GetPizza()
    {
        return pizza;
    }
}

// Director (optional)
class PizzaDirector
{
    public void ConstructPizza(IPizzaBuilder builder)
    {
        builder.BuildDough();
        builder.BuildSauce();
        builder.BuildTopping();
    }
}

// Usage
var director = new PizzaDirector();

var margheritaBuilder = new MargheritaPizzaBuilder();
director.ConstructPizza(margheritaBuilder);
var margherita = margheritaBuilder.GetPizza();
margherita.Display();

var pepperoniBuilder = new PepperoniPizzaBuilder();
director.ConstructPizza(pepperoniBuilder);
var pepperoni = pepperoniBuilder.GetPizza();
pepperoni.Display();
```

### Alternative Implementation - Fluent Builder

```csharp
// Product
class Burger
{
    public string Bun { get; set; }
    public string Patty { get; set; }
    public bool Cheese { get; set; }
    public bool Lettuce { get; set; }
    public bool Tomato { get; set; }

    public void Display()
    {
        var extras = new List<string>();
        if (Cheese) extras.Add("cheese");
        if (Lettuce) extras.Add("lettuce");
        if (Tomato) extras.Add("tomato");

        var extrasStr = extras.Count > 0 ? " with " + string.Join(", ", extras) : "";
        Console.WriteLine($"{Patty} burger on {Bun} bun{extrasStr}");
    }
}

// Fluent Builder
class BurgerBuilder
{
    private Burger burger = new Burger();

    public BurgerBuilder SetBun(string bun)
    {
        burger.Bun = bun;
        return this;
    }

    public BurgerBuilder SetPatty(string patty)
    {
        burger.Patty = patty;
        return this;
    }

    public BurgerBuilder AddCheese()
    {
        burger.Cheese = true;
        return this;
    }

    public BurgerBuilder AddLettuce()
    {
        burger.Lettuce = true;
        return this;
    }

    public BurgerBuilder AddTomato()
    {
        burger.Tomato = true;
        return this;
    }

    public Burger Build()
    {
        return burger;
    }
}

// Usage - fluent style
var burger = new BurgerBuilder()
    .SetBun("sesame")
    .SetPatty("beef")
    .AddCheese()
    .AddLettuce()
    .AddTomato()
    .Build();

burger.Display();

var simpleBurger = new BurgerBuilder()
    .SetBun("plain")
    .SetPatty("chicken")
    .Build();

simpleBurger.Display();
```

### When to Use
- Algorithm for creating a complex object should be independent of the parts
- Construction process must allow different representations
- You want to construct an object with many optional parameters
- You want to create immutable objects in a clean way
- Object creation requires multiple steps

---

## Key Differences Summary

### Factory Method vs Abstract Factory

**Factory Method:**
- Creates **one** product type
- Uses inheritance (subclasses override factory method)
- One method to create objects
- **Example:** ShapeFactory creates shapes

**Abstract Factory:**
- Creates **families** of related products
- Uses object composition
- Multiple methods to create different products
- **Example:** GUIFactory creates buttons AND checkboxes

### Factory Method vs Builder

**Factory Method:**
- Creates object in **one step**
- Focuses on **which** object to create
- Returns the created object immediately
- **Example:** CreateShape() returns a shape

**Builder:**
- Creates object in **multiple steps**
- Focuses on **how** to create the object
- Constructs object piece by piece
- **Example:** BuildDough(), BuildSauce(), BuildTopping()

### Abstract Factory vs Builder

**Abstract Factory:**
- Emphasizes a **family of products**
- Returns product immediately
- Products are simple or complex
- **Example:** Create Windows UI components

**Builder:**
- Emphasizes **step-by-step construction**
- Returns product at the end
- Products are typically complex
- **Example:** Build a pizza with multiple steps

### All Three Patterns

| Aspect | Factory Method | Abstract Factory | Builder |
|--------|---------------|------------------|---------|
| **Purpose** | Subclass decides what to create | Create families of objects | Construct complex objects |
| **Complexity** | Simple | Medium | Complex |
| **Steps** | One step | One step per product | Multiple steps |
| **Focus** | Which class | Which family | How to build |
| **Returns** | One product | Multiple related products | One complex product |

### When to Choose

**Choose Factory Method when:**
- You have a single product type
- Subclasses should decide the concrete class
- You want to delegate instantiation to subclasses

**Choose Abstract Factory when:**
- You need to create families of related objects
- You want to ensure created objects work together
- You want to switch between different product families

**Choose Builder when:**
- Object creation is complex with many steps
- You need different representations of the same type
- You want a fluent interface for object creation
- Object has many optional parameters

### Real-World Examples

**Factory Method:**
- Document types (PDF, Word, Excel)
- Database connections (MySQL, PostgreSQL, SQL Server)
- Logger implementations (FileLogger, ConsoleLogger)

**Abstract Factory:**
- UI toolkits (Windows, Mac, Linux themes)
- Database access layers (different database providers)
- Game environments (Medieval, SciFi, Fantasy themes)

**Builder:**
- StringBuilder in .NET
- SQL query builders
- HTTP request builders
- Configuration builders

### Common Pattern Combinations

Often used together:
- **Factory Method + Builder:** Factory creates builder, builder constructs product
- **Abstract Factory + Factory Method:** Abstract factory uses factory methods
- **Builder + Singleton:** Director can be a singleton

All three patterns solve the same fundamental problem (object creation) but at different levels of complexity and with different goals in mind.
