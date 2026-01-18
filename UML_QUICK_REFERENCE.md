# UML Quick Reference Guide

## Basic Class Notation

### Simple Class
```
┌─────────────────────────┐
│      ClassName          │
├─────────────────────────┤
│ - attribute: Type       │
├─────────────────────────┤
│ + method()              │
└─────────────────────────┘
```

### Abstract Class
```
┌─────────────────────────┐
│    <<abstract>>         │
│      ClassName          │
├─────────────────────────┤
│ # protectedField: Type  │
├─────────────────────────┤
│ + abstractMethod()      │
└─────────────────────────┘
```

### Interface
```
┌─────────────────────────┐
│   <<interface>>         │
│     InterfaceName       │
├─────────────────────────┤
│ + method1()             │
│ + method2()             │
└─────────────────────────┘
```

### Enumeration
```
<<enumeration>>
EnumName
─────────────
VALUE1
VALUE2
VALUE3
```

---

## Visibility Modifiers

| Symbol | Visibility | Description |
|--------|-----------|-------------|
| `+` | public | Accessible from anywhere |
| `-` | private | Accessible only within class |
| `#` | protected | Accessible within class and subclasses |
| `~` | package | Accessible within same package |

---

## Relationships

### 1. Inheritance (Generalization)
**Symbol:** `△` or `▲`

**Meaning:** "is-a" relationship

```
      Parent
        △
        │
        │
      Child
```

**Example:**
```
    Vehicle
      △
      │
   ┌──┴──┐
   │     │
  Car  Bike
```

---

### 2. Realization (Interface Implementation)
**Symbol:** `△` with dashed line

**Meaning:** Class implements interface

```
   <<interface>>
    Drawable
        △
        │
   ┌────┘
   │
Circle
```

---

### 3. Association
**Symbol:** `────` or `────▶`

**Meaning:** "has-a" or "uses" relationship

```
Student ────▶ Course
   1       enrolls in    *
```

**With multiplicity:**
```
Teacher ────▶ Student
   1     teaches    1..*
```

---

### 4. Aggregation
**Symbol:** `◇────`

**Meaning:** "has-a" (whole-part, part can exist independently)

```
Department ◇──── Professor
```

**Example:** Department has Professors, but Professors can exist without Department

---

### 5. Composition
**Symbol:** `◆────`

**Meaning:** "owns-a" (strong whole-part, part cannot exist independently)

```
House ◆──── Room
```

**Example:** House owns Rooms, Rooms cannot exist without House

---

### 6. Dependency
**Symbol:** `----▶` (dashed arrow)

**Meaning:** "uses" or "depends on"

```
Client - - - ▶ Service
```

**Example:** Client uses Service but doesn't store a reference

---

## Multiplicity (Cardinality)

| Notation | Meaning |
|----------|---------|
| `1` | Exactly one |
| `0..1` | Zero or one |
| `*` or `0..*` | Zero or more |
| `1..*` | One or more |
| `m..n` | Between m and n |
| `n` | Exactly n |

### Examples:
```
┌─────────┐        ┌─────────┐
│  Order  │1     * │OrderItem│
│         │────────│         │
└─────────┘        └─────────┘
  An order has many items


┌─────────┐        ┌─────────┐
│ Person  │1    0..1│Passport │
│         │────────│         │
└─────────┘        └─────────┘
  A person may have zero or one passport


┌─────────┐        ┌─────────┐
│University│1   1..*│ Student │
│         │────────│         │
└─────────┘        └─────────┘
  A university must have at least one student
```

---

## Stereotypes

Common stereotypes used in UML:

| Stereotype | Usage |
|------------|-------|
| `<<interface>>` | Interface definition |
| `<<abstract>>` | Abstract class |
| `<<enumeration>>` | Enumeration type |
| `<<Singleton>>` | Singleton pattern |
| `<<utility>>` | Utility/helper class |
| `<<entity>>` | Domain entity |
| `<<boundary>>` | System boundary |
| `<<control>>` | Controller/service |

---

## Method Notation

### Parameters and Return Types
```
+ methodName(param1: Type1, param2: Type2): ReturnType
```

### Static Methods
```
+ {static} staticMethod(): void
```

### Abstract Methods
```
+ {abstract} abstractMethod(): Type
```

### Examples:
```
┌─────────────────────────┐
│       Calculator        │
├─────────────────────────┤
│ + add(a: int, b: int): int
│ + subtract(a: int, b: int): int
│ + {static} getInstance(): Calculator
└─────────────────────────┘
```

---

## Design Pattern Notations

### 1. Singleton Pattern
```
┌─────────────────────────┐
│   <<Singleton>>         │
│      Database           │
├─────────────────────────┤
│ - instance: Database    │
├─────────────────────────┤
│ + {static} getInstance()│
│ - Database()            │  ← private constructor
└─────────────────────────┘
```

### 2. Factory Pattern
```
┌──────────────────┐
│  <<interface>>   │
│     Product      │
└──────────────────┘
        △
        │
   ┌────┴────┐
   │         │
ProductA  ProductB

┌──────────────────┐
│ ProductFactory   │
├──────────────────┤
│ + create(): Prod │
└──────────────────┘
```

### 3. Strategy Pattern
```
┌──────────────────┐
│  <<interface>>   │
│    Strategy      │
├──────────────────┤
│ + execute()      │
└──────────────────┘
        △
        │
   ┌────┴────┐
   │         │
StrategyA StrategyB

┌──────────────────┐
│    Context       │
├──────────────────┤
│ - strategy       │
├──────────────────┤
│ + setStrategy()  │
│ + executeStrategy()
└──────────────────┘
```

### 4. Observer Pattern
```
┌──────────────┐         ┌──────────────┐
│   Subject    │1      * │   Observer   │
│              │◇────────│              │
│ + attach()   │         │ + update()   │
│ + detach()   │         └──────────────┘
│ + notify()   │
└──────────────┘
```

---

## Common Relationships Summary

### Quick Reference Table

| Relationship | Symbol | Strength | Description |
|--------------|--------|----------|-------------|
| Inheritance | `──△` | Strongest | IS-A relationship |
| Composition | `──◆` | Very Strong | OWNS-A (lifecycle dependency) |
| Aggregation | `──◇` | Strong | HAS-A (no lifecycle dependency) |
| Association | `──▶` | Medium | USES or KNOWS-ABOUT |
| Dependency | `- - -▶` | Weakest | DEPENDS-ON temporarily |

---

## Complete Example

```
┌──────────────────────┐
│   <<interface>>      │
│      Animal          │
├──────────────────────┤
│ + makeSound()        │
│ + eat()              │
└──────────────────────┘
          △
          │
    ┌─────┴─────┐
    │           │
┌─────────┐ ┌─────────┐
│   Dog   │ │   Cat   │
├─────────┤ ├─────────┤
│ - name  │ │ - name  │
├─────────┤ ├─────────┤
│ + bark()│ │ + meow()│
└─────────┘ └─────────┘
    │            │
    │  owned by  │  owned by
    │            │
    └────┬───────┘
         │ *
         ▼ 1
┌─────────────────┐
│     Person      │
├─────────────────┤
│ - name: String  │
│ - age: int      │
│ - pets: List    │
├─────────────────┤
│ + adoptPet()    │
│ + feedPets()    │
└─────────────────┘
         │ 1
         │ lives in
         ▼ 1
┌─────────────────┐
│     House       │
├─────────────────┤
│ - address       │
├─────────────────┤
│ + getAddress()  │
└─────────────────┘
```

---

## Notes Drawing Tips

### For Interviews:

1. **Start Simple** - Draw main boxes first
2. **Add Relationships** - Connect boxes with appropriate lines
3. **Show Key Attributes** - Don't need to list everything
4. **Highlight Patterns** - Use stereotypes to show design patterns
5. **Use Clear Labels** - Name relationships clearly
6. **Keep It Clean** - Avoid crossing lines when possible
7. **Explain As You Draw** - Talk through your design decisions

### Common Mistakes to Avoid:

❌ Too much detail in one diagram
❌ Missing relationship labels
❌ Wrong arrow directions
❌ Confusing composition with aggregation
❌ Forgetting multiplicity
❌ Not showing visibility modifiers

✅ Focus on important classes
✅ Label all relationships
✅ Use correct arrow types
✅ Understand ownership semantics
✅ Show cardinality clearly
✅ Use + and - for public/private

---

## Practice Exercise

Draw UML for: **Library Management System**

Classes to include:
- Book
- Member
- Library
- Loan
- Librarian

Relationships:
- Member borrows Book (creates Loan)
- Library contains Books
- Librarian manages Library
- Loan tracks Book and Member

Try drawing this before checking the solution in the repository!

---

## Additional Resources

### Tools for Drawing UML:
- **Online:** draw.io, Lucidchart, PlantUML
- **Desktop:** StarUML, Visual Paradigm, ArgoUML
- **Code-based:** PlantUML, Mermaid
- **Whiteboard:** Physical whiteboard for interviews!

### Learn More:
- UML 2.5 Specification
- "UML Distilled" by Martin Fowler
- "Applying UML and Patterns" by Craig Larman

---

**Remember:** In interviews, correctness > prettiness. Focus on showing you understand:
- Class responsibilities
- Relationships between classes
- Design patterns
- SOLID principles

Good luck! 🎯
