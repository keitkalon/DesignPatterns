# Middle Men Design Patterns Comparison Cheat Sheet

## Structural Comparison Table

| Pattern | Interfaces | Abstract Classes | Concrete Classes | Key Characteristic |
|---------|-----------|------------------|------------------|-------------------|
| **[Facade](#1-facade-pattern)** | 0 | 0 | 5+ | One facade wraps multiple subsystem classes |
| **[Mediator](#2-mediator-pattern)** | 2 | 0 | 2+ | Mediator + multiple colleagues that communicate through it |
| **[Proxy](#3-proxy-pattern)** | 1 | 0 | 2 | Proxy and RealSubject share same interface |
| **[Adapter](#4-adapter-pattern)** | 1 | 0 | 2 | Adapter wraps Adaptee to match Target interface |
| **[Command](#5-command-pattern)** | 1 | 0 | 3+ | Commands + Receiver + Invoker (request as object) |

---

## 1. FACADE PATTERN

### Purpose
Provides a simplified interface to a complex subsystem. One-way relationship where facade knows subsystem, but subsystem doesn't know facade.

### Components
- **Interfaces:** None
- **Abstract classes:** None
- **Concrete classes:** `TV`, `SoundSystem`, `DVDPlayer`, `Lights`, `HomeTheaterFacade`

### C# Implementation

```csharp
// Complex subsystem classes
class TV
{
    public void On() => Console.WriteLine("TV is on");
    public void Off() => Console.WriteLine("TV is off");
    public void SetInput(string input) => Console.WriteLine($"TV input: {input}");
}

class SoundSystem
{
    public void On() => Console.WriteLine("Sound system is on");
    public void Off() => Console.WriteLine("Sound system is off");
    public void SetVolume(int level) => Console.WriteLine($"Volume: {level}");
}

class DVDPlayer
{
    public void On() => Console.WriteLine("DVD player is on");
    public void Off() => Console.WriteLine("DVD player is off");
    public void Play(string movie) => Console.WriteLine($"Playing: {movie}");
}

class Lights
{
    public void Dim(int level) => Console.WriteLine($"Lights dimmed to {level}%");
}

// Facade - simplifies the complex subsystem
class HomeTheaterFacade
{
    private TV tv;
    private SoundSystem soundSystem;
    private DVDPlayer dvdPlayer;
    private Lights lights;

    public HomeTheaterFacade(TV tv, SoundSystem sound, DVDPlayer dvd, Lights lights)
    {
        this.tv = tv;
        this.soundSystem = sound;
        this.dvdPlayer = dvd;
        this.lights = lights;
    }

    public void WatchMovie(string movie)
    {
        Console.WriteLine("Get ready to watch a movie...");
        lights.Dim(10);
        tv.On();
        tv.SetInput("DVD");
        soundSystem.On();
        soundSystem.SetVolume(5);
        dvdPlayer.On();
        dvdPlayer.Play(movie);
    }

    public void EndMovie()
    {
        Console.WriteLine("Shutting down theater...");
        dvdPlayer.Off();
        soundSystem.Off();
        tv.Off();
        lights.Dim(100);
    }
}

// Usage
var facade = new HomeTheaterFacade(new TV(), new SoundSystem(), new DVDPlayer(), new Lights());
facade.WatchMovie("Inception");
```

### When to Use
- You need to simplify a complex subsystem
- You want to reduce dependencies on subsystem classes
- You want to layer your subsystems

---

## 2. MEDIATOR PATTERN

### Purpose
Manages communication between multiple objects (colleagues) to reduce coupling. Hub-and-spoke architecture for coordinating interactions.

### Components
- **Interfaces:** `IMediator`, `IColleague`
- **Abstract classes:** None
- **Concrete classes:** `ChatRoomMediator`, `User`

### C# Implementation

```csharp
// Mediator interface
interface IMediator
{
    void SendMessage(string message, IColleague colleague);
    void RegisterColleague(IColleague colleague);
}

// Colleague interface
interface IColleague
{
    void Send(string message);
    void Receive(string message);
}

// Concrete Mediator - coordinates communication
class ChatRoomMediator : IMediator
{
    private List<IColleague> colleagues = new List<IColleague>();

    public void RegisterColleague(IColleague colleague)
    {
        colleagues.Add(colleague);
    }

    public void SendMessage(string message, IColleague sender)
    {
        foreach (var colleague in colleagues)
        {
            if (colleague != sender)
            {
                colleague.Receive(message);
            }
        }
    }
}

// Concrete Colleague - communicates through mediator
class User : IColleague
{
    private IMediator mediator;
    private string name;

    public User(string name, IMediator mediator)
    {
        this.name = name;
        this.mediator = mediator;
        mediator.RegisterColleague(this);
    }

    public void Send(string message)
    {
        Console.WriteLine($"{name} sends: {message}");
        mediator.SendMessage(message, this);
    }

    public void Receive(string message)
    {
        Console.WriteLine($"{name} receives: {message}");
    }
}

// Usage
var chatRoom = new ChatRoomMediator();
var user1 = new User("Alice", chatRoom);
var user2 = new User("Bob", chatRoom);
var user3 = new User("Charlie", chatRoom);

user1.Send("Hello everyone!");
```

### When to Use
- Objects communicate in complex but well-defined ways
- Reusing objects is difficult because they refer to many other objects
- Behavior distributed between classes should be customizable

---

## 3. PROXY PATTERN

### Purpose
Controls access to an object by providing a surrogate. Same interface as the real object. Used for lazy loading, access control, logging, or remote access.

### Components
- **Interfaces:** `IImage`
- **Abstract classes:** None
- **Concrete classes:** `RealImage`, `ProxyImage`

### C# Implementation

```csharp
// Subject interface
interface IImage
{
    void Display();
}

// Real Subject - the actual object
class RealImage : IImage
{
    private string filename;

    public RealImage(string filename)
    {
        this.filename = filename;
        LoadFromDisk();
    }

    private void LoadFromDisk()
    {
        Console.WriteLine($"Loading {filename} from disk...");
    }

    public void Display()
    {
        Console.WriteLine($"Displaying {filename}");
    }
}

// Proxy - controls access to the real subject
class ProxyImage : IImage
{
    private RealImage realImage;
    private string filename;

    public ProxyImage(string filename)
    {
        this.filename = filename;
    }

    public void Display()
    {
        if (realImage == null)
        {
            realImage = new RealImage(filename);
        }
        realImage.Display();
    }
}

// Usage
IImage image = new ProxyImage("photo.jpg");
// Image not loaded yet
image.Display(); // Loads and displays
image.Display(); // Just displays (already loaded)
```

### When to Use
- **Virtual Proxy:** Expensive object creation (lazy initialization)
- **Protection Proxy:** Access control to the real object
- **Remote Proxy:** Represents object in different address space
- **Logging Proxy:** Log requests to the real object

---

## 4. ADAPTER PATTERN

### Purpose
Converts one interface to another that clients expect. It's about compatibility between incompatible interfaces.

### Components
- **Interfaces:** `ITarget`
- **Abstract classes:** None
- **Concrete classes:** `Adaptee`, `Adapter`

### C# Implementation

```csharp
// Target interface - what the client expects
interface ITarget
{
    void Request();
}

// Adaptee - existing class with incompatible interface
class Adaptee
{
    public void SpecificRequest()
    {
        Console.WriteLine("Adaptee's specific request");
    }
}

// Adapter - converts Adaptee's interface to Target interface
class Adapter : ITarget
{
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee)
    {
        this.adaptee = adaptee;
    }

    public void Request()
    {
        // Translates the call
        adaptee.SpecificRequest();
    }
}

// Usage
Adaptee adaptee = new Adaptee();
ITarget adapter = new Adapter(adaptee);
adapter.Request(); // Client uses the expected interface
```

### When to Use
- You want to use an existing class with an incompatible interface
- You need to create a reusable class that cooperates with unrelated classes
- You need to adapt several different subclasses (object adapter)

---

## 5. COMMAND PATTERN

### Purpose
Encapsulates requests as objects, allowing you to parameterize operations, queue them, log them, or support undo/redo.

### Components
- **Interfaces:** `ICommand`
- **Abstract classes:** None
- **Concrete classes:** `Light`, `LightOnCommand`, `LightOffCommand`, `RemoteControl`

### C# Implementation

```csharp
// Command interface
interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver - the object that performs the actual work
class Light
{
    public void On()
    {
        Console.WriteLine("Light is ON");
    }

    public void Off()
    {
        Console.WriteLine("Light is OFF");
    }
}

// Concrete Commands - encapsulate requests as objects
class LightOnCommand : ICommand
{
    private Light light;

    public LightOnCommand(Light light)
    {
        this.light = light;
    }

    public void Execute()
    {
        light.On();
    }

    public void Undo()
    {
        light.Off();
    }
}

class LightOffCommand : ICommand
{
    private Light light;

    public LightOffCommand(Light light)
    {
        this.light = light;
    }

    public void Execute()
    {
        light.Off();
    }

    public void Undo()
    {
        light.On();
    }
}

// Invoker - asks the command to carry out the request
class RemoteControl
{
    private ICommand command;

    public void SetCommand(ICommand command)
    {
        this.command = command;
    }

    public void PressButton()
    {
        command.Execute();
    }

    public void PressUndo()
    {
        command.Undo();
    }
}

// Usage
var light = new Light();
var lightOn = new LightOnCommand(light);
var lightOff = new LightOffCommand(light);

var remote = new RemoteControl();
remote.SetCommand(lightOn);
remote.PressButton(); // Light ON
remote.PressUndo();   // Light OFF

remote.SetCommand(lightOff);
remote.PressButton(); // Light OFF
```

### When to Use
- Parameterize objects with operations
- Queue operations, schedule execution, or execute remotely
- Support undo/redo functionality
- Support logging changes (for crash recovery)
- Structure a system around high-level operations built on primitive operations

---

## Key Differences Summary

### Facade vs Mediator
- **Facade:** Simplifies access (unidirectional) - client → facade → subsystem
- **Mediator:** Coordinates communication (bidirectional) - colleagues ↔ mediator ↔ colleagues

### Proxy vs Adapter
- **Proxy:** Same interface, controls access to single object
- **Adapter:** Different interfaces, makes incompatible interfaces compatible

### Command vs Others
- **Command:** Treats operations as objects, fundamentally different from structural patterns
- Others focus on relationships between objects, Command focuses on encapsulating behavior

### All Five Patterns
- **Facade:** Simplification
- **Mediator:** Coordination
- **Proxy:** Access control
- **Adapter:** Interface translation
- **Command:** Operation encapsulation
