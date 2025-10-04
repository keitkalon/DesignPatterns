# SOLID Principles Cheat Sheet (C#)
- Single-responsibility principle 
- Open-closed principle 
- Liskov substitution principle 
- Interface segregation principle 
- Dependency inversion principle

---

## S — Single Responsibility Principle (SRP)

**One class = one reason to change.**

**Bad**: mixes validation and persistence.

```csharp
class UserService {
  public bool IsValidEmail(string email) => email.Contains("@");
  public void Save(string email) { /* write to DB */ }
}
```

**Good**: split responsibilities.

```csharp
class EmailValidator { public bool IsValid(string email) => email.Contains("@"); }
class UserRepository { public void Save(string email) { /* write to DB */ } }
class UserService {
  EmailValidator v = new EmailValidator();
  UserRepository r = new UserRepository();
  public void Register(string email) { if (!v.IsValid(email)) return; r.Save(email); }
}
```

---

## O — Open/Closed Principle (OCP)

**Open for extension, closed for modification.**

**Bad**: type checks + `if`/`else`.

```csharp
class AreaCalculatorBad {
  public double Sum(object[] shapes){ double s=0; foreach(var sh in shapes){
    if (sh is RectangleBad rc) s += rc.W*rc.H; else if (sh is CircleBad c) s += c.R*c.R*System.Math.PI; }
    return s; }
}
class RectangleBad { public double W,H; }
class CircleBad { public double R; }
```

**Good**: polymorphic `Area()`.

```csharp
abstract class Shape { public abstract double Area(); }
class Rectangle : Shape { public double W,H; public override double Area()=> W*H; }
class Circle : Shape { public double R; public override double Area()=> R*R*System.Math.PI; }
class AreaCalculator { public double Sum(Shape[] shapes){ double s=0; foreach(var sh in shapes) s+=sh.Area(); return s; } }
```

---

## L — Liskov Substitution Principle (LSP)

**Subtypes must be usable anywhere their base type is expected.**

**Bad**: subtype breaks base expectations.

```csharp
abstract class Bird { public abstract void Fly(); }
class Ostrich : Bird { public override void Fly(){ throw new System.NotSupportedException(); } }
```

**Good**: refine the hierarchy so all subclasses honor the contract.

```csharp
abstract class Bird { }
abstract class FlyingBird : Bird { public abstract void Fly(); }
class Sparrow : FlyingBird { public override void Fly(){ /* fly */ } }
class Ostrich2 : Bird { /* cannot fly, but OK now */ }
```

---

## I — Interface Segregation Principle (ISP)

**Prefer many small interfaces to fat ones.**

**Bad**: forces unneeded members.

```csharp
interface IPrinter { void Print(); void Scan(); void Fax(); }
class SimplePrinter : IPrinter { public void Print(){} public void Scan(){ } public void Fax(){ } }
```

**Good**: segregate capabilities.

```csharp
interface IPrint { void Print(); }
interface IScan { void Scan(); }
interface IFax { void Fax(); }
class BasicPrinter : IPrint { public void Print(){} }
class MultiFunction : IPrint, IScan, IFax { public void Print(){} public void Scan(){} public void Fax(){} }
```

---

## D — Dependency Inversion Principle (DIP)

**Depend on abstractions, not concretions.**

**Bad**: high-level class `new`s a concrete dependency.

```csharp
class SqlLogger { public void Log(string m){ /* write to SQL */ } }
class OrderServiceBad { SqlLogger l = new SqlLogger(); public void Place(){ l.Log("placed"); }
}
```

**Good**: inject an abstraction.

```csharp
interface ILogger { void Log(string m); }
class SqlLogger2 : ILogger { public void Log(string m){} }
class FileLogger : ILogger { public void Log(string m){} }
class OrderService { ILogger l; public OrderService(ILogger logger){ l=logger; } public void Place(){ l.Log("placed"); } }
```

---

## Extra Patterns You’ll Use With SOLID

* **Strategy** (OCP/DIP): swap algorithms at runtime via an interface.
* **Decorator** (OCP): add behavior without modifying existing code.
* **Factory** (DIP): create instances without binding to concrete types.

---

## Quick Smell → Principle Map

* Many unrelated methods in one class → **SRP**
* Long `switch`/`if` on type → **OCP** (prefer polymorphism)
* Subclass throws for valid base ops → **LSP** (fix hierarchy)
* Interface with unused members → **ISP**
* `new`ing dependencies everywhere → **DIP** (inject abstractions)
