# State-Behavior Design Patterns Comparison Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[State](#1-state-pattern)** | 1 | 0 | 3+ | Object changes behavior when internal state changes |
| **[Observer](#2-observer-pattern)** | 2 | 0 | 2+ | One-to-many dependency, observers notified of changes |
| **[Iterator](#3-iterator-pattern)** | 1-2 | 0 | 2+ | Sequential access to collection elements without exposing structure |
| **[Visitor](#4-visitor-pattern)** | 2 | 0 | 3+ | Add operations to objects without modifying their classes |
| **[Memento](#5-memento-pattern)** | 0 | 0 | 3 | Capture and restore object state without violating encapsulation |

---

## 1. STATE PATTERN

### Purpose
Allows an object to change its behavior when its internal state changes. The object appears to change its class.

### Components
- **Interfaces:** `IState`
- **Abstract classes:** None
- **Concrete classes:** `RedState`, `YellowState`, `GreenState`, `TrafficLight` (context)

### C# Implementation

```csharp
// State interface
interface IState
{
    void Handle(TrafficLight light);
}

// Concrete State 1
class RedState : IState
{
    public void Handle(TrafficLight light)
    {
        Console.WriteLine("Red Light - STOP");
        light.SetState(new GreenState());
    }
}

// Concrete State 2
class YellowState : IState
{
    public void Handle(TrafficLight light)
    {
        Console.WriteLine("Yellow Light - CAUTION");
        light.SetState(new RedState());
    }
}

// Concrete State 3
class GreenState : IState
{
    public void Handle(TrafficLight light)
    {
        Console.WriteLine("Green Light - GO");
        light.SetState(new YellowState());
    }
}

// Context - maintains current state
class TrafficLight
{
    private IState currentState;

    public TrafficLight()
    {
        currentState = new RedState();
    }

    public void SetState(IState state)
    {
        currentState = state;
    }

    public void Change()
    {
        currentState.Handle(this);
    }
}

// Usage
var trafficLight = new TrafficLight();
trafficLight.Change(); // Red -> Green
trafficLight.Change(); // Green -> Yellow
trafficLight.Change(); // Yellow -> Red
trafficLight.Change(); // Red -> Green
```

### When to Use
- Object behavior depends on its state and must change at runtime
- Operations have large conditional statements that depend on object state
- State-specific behavior should be defined independently

---

## 2. OBSERVER PATTERN

### Purpose
Defines a one-to-many dependency between objects so when one object changes state, all its dependents are notified automatically.

### Components
- **Interfaces:** `ISubject`, `IObserver`
- **Abstract classes:** None
- **Concrete classes:** `NewsAgency` (subject), `NewsChannel` (observer)

### C# Implementation

```csharp
// Observer interface
interface IObserver
{
    void Update(string news);
}

// Subject interface
interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}

// Concrete Subject
class NewsAgency : ISubject
{
    private List<IObserver> observers = new List<IObserver>();
    private string news;

    public void Attach(IObserver observer)
    {
        observers.Add(observer);
        Console.WriteLine("Observer attached");
    }

    public void Detach(IObserver observer)
    {
        observers.Remove(observer);
        Console.WriteLine("Observer detached");
    }

    public void Notify()
    {
        foreach (var observer in observers)
        {
            observer.Update(news);
        }
    }

    public void SetNews(string news)
    {
        this.news = news;
        Console.WriteLine($"\nNewsAgency: Breaking news - {news}");
        Notify();
    }
}

// Concrete Observer
class NewsChannel : IObserver
{
    private string name;

    public NewsChannel(string name)
    {
        this.name = name;
    }

    public void Update(string news)
    {
        Console.WriteLine($"{name} received: {news}");
    }
}

// Usage
var newsAgency = new NewsAgency();

var channel1 = new NewsChannel("CNN");
var channel2 = new NewsChannel("BBC");
var channel3 = new NewsChannel("FOX");

newsAgency.Attach(channel1);
newsAgency.Attach(channel2);
newsAgency.Attach(channel3);

newsAgency.SetNews("Major earthquake detected!");

newsAgency.Detach(channel2);

newsAgency.SetNews("Stock market hits record high!");
```

### When to Use
- A change to one object requires changing others, and you don't know how many
- An object should notify other objects without knowing who they are
- You need loose coupling between interacting objects

---

## 3. ITERATOR PATTERN

### Purpose
Provides a way to access elements of a collection sequentially without exposing its underlying representation.

### Components
- **Interfaces:** `IIterator<T>`, `ICollection<T>`
- **Abstract classes:** None
- **Concrete classes:** `BookIterator`, `BookCollection`

### C# Implementation

```csharp
// Iterator interface
interface IIterator<T>
{
    bool HasNext();
    T Next();
}

// Collection interface
interface ICollection<T>
{
    IIterator<T> CreateIterator();
}

// Simple item class
class Book
{
    public string Title { get; set; }

    public Book(string title)
    {
        Title = title;
    }
}

// Concrete Iterator
class BookIterator : IIterator<Book>
{
    private List<Book> books;
    private int position = 0;

    public BookIterator(List<Book> books)
    {
        this.books = books;
    }

    public bool HasNext()
    {
        return position < books.Count;
    }

    public Book Next()
    {
        return books[position++];
    }
}

// Concrete Collection
class BookCollection : ICollection<Book>
{
    private List<Book> books = new List<Book>();

    public void AddBook(Book book)
    {
        books.Add(book);
    }

    public IIterator<Book> CreateIterator()
    {
        return new BookIterator(books);
    }
}

// Usage
var collection = new BookCollection();
collection.AddBook(new Book("Design Patterns"));
collection.AddBook(new Book("Clean Code"));
collection.AddBook(new Book("The Pragmatic Programmer"));

var iterator = collection.CreateIterator();

while (iterator.HasNext())
{
    var book = iterator.Next();
    Console.WriteLine($"Book: {book.Title}");
}
```

### When to Use
- You need to access a collection's contents without exposing its internal structure
- You need to support multiple traversals of collections
- You want a uniform interface for traversing different collection structures

---

## 4. VISITOR PATTERN

### Purpose
Lets you define new operations on objects without changing the classes of the objects. Separates algorithms from the objects they operate on.

### Components
- **Interfaces:** `IVisitor`, `IElement`
- **Abstract classes:** None
- **Concrete classes:** `AreaVisitor`, `Circle`, `Rectangle`, `Triangle`

### C# Implementation

```csharp
// Visitor interface
interface IVisitor
{
    void Visit(Circle circle);
    void Visit(Rectangle rectangle);
    void Visit(Triangle triangle);
}

// Element interface
interface IElement
{
    void Accept(IVisitor visitor);
}

// Concrete Element 1
class Circle : IElement
{
    public double Radius { get; set; }

    public Circle(double radius)
    {
        Radius = radius;
    }

    public void Accept(IVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Concrete Element 2
class Rectangle : IElement
{
    public double Width { get; set; }
    public double Height { get; set; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public void Accept(IVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Concrete Element 3
class Triangle : IElement
{
    public double Base { get; set; }
    public double Height { get; set; }

    public Triangle(double baseLength, double height)
    {
        Base = baseLength;
        Height = height;
    }

    public void Accept(IVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Concrete Visitor - calculates area
class AreaVisitor : IVisitor
{
    public void Visit(Circle circle)
    {
        double area = Math.PI * circle.Radius * circle.Radius;
        Console.WriteLine($"Circle area: {area:F2}");
    }

    public void Visit(Rectangle rectangle)
    {
        double area = rectangle.Width * rectangle.Height;
        Console.WriteLine($"Rectangle area: {area:F2}");
    }

    public void Visit(Triangle triangle)
    {
        double area = 0.5 * triangle.Base * triangle.Height;
        Console.WriteLine($"Triangle area: {area:F2}");
    }
}

// Usage
var shapes = new List<IElement>
{
    new Circle(5),
    new Rectangle(4, 6),
    new Triangle(3, 8)
};

var areaVisitor = new AreaVisitor();

foreach (var shape in shapes)
{
    shape.Accept(areaVisitor);
}
```

### When to Use
- You need to perform operations on all elements of a complex object structure
- Many distinct operations need to be performed on objects in a structure
- Object structure classes rarely change, but you want to define new operations
- You want to avoid polluting element classes with many operations

---

## 5. MEMENTO PATTERN

### Purpose
Captures and externalizes an object's internal state without violating encapsulation, so the object can be restored to this state later.

### Components
- **Interfaces:** None
- **Abstract classes:** None
- **Concrete classes:** `TextEditor` (originator), `TextMemento` (memento), `History` (caretaker)

### C# Implementation

```csharp
// Memento - stores state
class TextMemento
{
    public string Text { get; private set; }

    public TextMemento(string text)
    {
        Text = text;
    }
}

// Originator - creates and restores from memento
class TextEditor
{
    private string text = "";

    public void Write(string newText)
    {
        text += newText;
        Console.WriteLine($"Current text: {text}");
    }

    public TextMemento Save()
    {
        Console.WriteLine("Saving state...");
        return new TextMemento(text);
    }

    public void Restore(TextMemento memento)
    {
        text = memento.Text;
        Console.WriteLine($"Restored text: {text}");
    }

    public void ShowText()
    {
        Console.WriteLine($"Text: {text}");
    }
}

// Caretaker - manages mementos
class History
{
    private Stack<TextMemento> mementos = new Stack<TextMemento>();

    public void Save(TextEditor editor)
    {
        mementos.Push(editor.Save());
    }

    public void Undo(TextEditor editor)
    {
        if (mementos.Count > 0)
        {
            var memento = mementos.Pop();
            editor.Restore(memento);
        }
        else
        {
            Console.WriteLine("No more undo steps!");
        }
    }
}

// Usage
var editor = new TextEditor();
var history = new History();

editor.Write("Hello ");
history.Save(editor);

editor.Write("World!");
history.Save(editor);

editor.Write(" How are you?");
editor.ShowText();

Console.WriteLine("\nUndo:");
history.Undo(editor);

Console.WriteLine("\nUndo again:");
history.Undo(editor);
```

### When to Use
- You need to save and restore an object's state
- A direct interface to obtain the state would expose implementation details
- You need undo/redo functionality
- You want to maintain snapshots of an object's state

---

## Key Differences Summary

### State vs Strategy
- **State:** Object changes behavior based on its internal state (state transitions)
- **Strategy:** Client chooses behavior from a family of algorithms (no state transitions)

### Observer vs Mediator
- **Observer:** One-to-many, subject notifies all observers (broadcast)
- **Mediator:** Many-to-many through central hub, controls complex interactions

### Iterator vs Visitor
- **Iterator:** Traverses collection elements sequentially
- **Visitor:** Performs operations on elements without modifying their classes

### Memento vs Command
- **Memento:** Saves/restores object state (snapshot of data)
- **Command:** Encapsulates operations (snapshot of behavior)

### All Five Patterns

| Pattern | Main Purpose | Key Behavior |
|---------|-------------|--------------|
| **State** | Manage state transitions | Behavior changes with state |
| **Observer** | Notify dependents of changes | One-to-many notification |
| **Iterator** | Traverse collections | Sequential access |
| **Visitor** | Add operations to objects | Separate algorithm from structure |
| **Memento** | Save/restore state | Undo/snapshot capability |

### Common Themes

**State Management:**
- **State:** Current behavior state
- **Memento:** Historical state snapshots

**Object Interaction:**
- **Observer:** Reactive notification
- **Visitor:** Active operation dispatch

**Collection Handling:**
- **Iterator:** Access pattern
- **Visitor:** Operation pattern on collection elements

All these patterns help manage complex behaviors and state changes while maintaining clean, maintainable code that follows the Open/Closed Principle.
