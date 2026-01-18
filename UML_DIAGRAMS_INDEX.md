# UML Class Diagrams Index

This document provides an index of all UML class diagrams available in the LLD Interview Prep repository.

## Available UML Diagrams

### 1. Parking Lot System
**File:** `01_Parking_Lot_System.md`

**Key Components:**
- Vehicle hierarchy (Car, Bike, Truck)
- ParkingSpot hierarchy (CompactSpot, LargeSpot, HandicappedSpot)
- ParkingFloor
- ParkingTicket
- ParkingLot (Singleton)
- PricingStrategy (Strategy Pattern)

**Design Patterns:** Singleton, Strategy, Abstract Class

---

### 2. Elevator System
**File:** `02_Elevator_System.md`

**Key Components:**
- Elevator
- ElevatorController
- Request
- Building
- DisplayPanel
- ElevatorSelectionStrategy (Optimal, Nearest, LoadBalancing)

**Design Patterns:** Strategy, State, Observer

---

### 3. LRU Cache
**File:** `03_LRU_Cache.md`

**Key Components:**
- LRUCache with doubly-linked list
- Node (inner class)
- MyHashMap implementation
- Trie implementation
- AutocompleteSystem

**Design Patterns:** Data Structure Design

---

### 4. URL Shortener
**File:** `04_URL_Shortener.md`

**Key Components:**
- URL
- URLShortener
- EncodingStrategy (Base62, MD5)
- Counter / DistributedIdGenerator
- AnalyticsService (Singleton)
- ClickEvent & ClientInfo
- URLAnalytics
- URLCache

**Design Patterns:** Singleton, Strategy, Factory

---

### 5. Rate Limiter
**File:** `05_Rate_Limiter.md`

**Key Components:**
- RateLimiter interface
- TokenBucketRateLimiter (with Bucket)
- SlidingWindowLogRateLimiter
- SlidingWindowCounterRateLimiter (with WindowCounter)
- DistributedRateLimiter
- MultiRuleRateLimiter
- RateLimiterFactory

**Design Patterns:** Strategy, Factory, Singleton, Decorator

---

### 6. Chess Game
**File:** `09_Chess_Game.md`

**Key Components:**
- Piece hierarchy (King, Queen, Rook, Bishop, Knight, Pawn)
- Board
- Position
- Player
- ChessGame
- Move

**Design Patterns:** Template Method, Factory, Command, Strategy

---

### 7. Tic Tac Toe
**File:** `10_Tic_Tac_Toe.md`

**Key Components:**
- Board
- Cell
- Player
- AIPlayer (with minimax algorithm)
- TicTacToeGame
- Move

**Design Patterns:** Strategy, Command, State, Template Method

---

### 8. Shopping Cart
**File:** `12_Shopping_Cart.md`

**Key Components:**
- Product
- CartItem
- ShoppingCart
- Order & OrderItem
- Payment
- PaymentMethod (CreditCard, PayPal)
- Address
- User
- ShoppingSystem (Singleton)

**Design Patterns:** Singleton, Strategy, State, Observer

---

### 9. Restaurant Management
**File:** `15_Restaurant_Management.md`

**Key Components:**
- Restaurant
- Table
- Menu & MenuItem
- Order & OrderItem
- Reservation & ReservationManager
- Bill
- KitchenService
- Staff hierarchy (Chef, Waiter)

**Design Patterns:** Observer, State, Command

---

### 10. ATM System
**File:** `18_ATM_System.md`

**Key Components:**
- Card
- Account
- ATM
- Transaction
- BankSystem (Singleton)
- Denomination enum

**Design Patterns:** Singleton, State, Strategy, Command

---

### 11. File System
**File:** `29_File_System.md`

**Key Components:**
- FileSystemEntity (Abstract)
- File
- Directory
- FileSystem (Singleton)
- User
- Permissions

**Design Patterns:** Composite, Singleton, Flyweight, Iterator

---

## How to Read UML Diagrams

### Class Structure
```
┌─────────────────────────┐
│      ClassName          │  ← Class Name
├─────────────────────────┤
│ - privateField: Type    │  ← Attributes (- private, + public, # protected)
│ + publicField: Type     │
├─────────────────────────┤
│ + publicMethod()        │  ← Methods
│ - privateMethod()       │
└─────────────────────────┘
```

### Relationships

**Inheritance (△):**
```
    Parent
      △
      │
    Child
```

**Composition (filled diamond):**
```
Container ◆────── Component
```

**Association (line with arrow):**
```
ClassA ────▶ ClassB
```

**Uses/Depends (dashed arrow):**
```
Client ----▶ Service
```

**Multiplicity:**
- `1` - exactly one
- `*` - zero or more
- `1..*` - one or more
- `0..1` - zero or one

### Stereotypes

- `<<abstract>>` - Abstract class
- `<<interface>>` - Interface
- `<<enumeration>>` - Enum
- `<<Singleton>>` - Singleton pattern

---

## Design Pattern Summary

| System | Primary Patterns |
|--------|------------------|
| Parking Lot | Singleton, Strategy, Factory |
| Elevator | Strategy, State, Observer |
| LRU Cache | Data Structure Design |
| URL Shortener | Singleton, Strategy, Factory |
| Rate Limiter | Strategy, Factory, Decorator |
| Chess Game | Template Method, Factory, Command |
| Tic Tac Toe | Strategy, State |
| Shopping Cart | Singleton, Strategy, State |
| Restaurant | Observer, State, Command |
| ATM System | Singleton, State, Command |
| File System | Composite, Singleton, Iterator |

---

## Interview Tips

1. **Start with high-level components** - Draw main classes first
2. **Show relationships** - Indicate inheritance, composition, association
3. **Include key methods** - Don't need every method, just important ones
4. **Mark design patterns** - Use stereotypes to indicate patterns
5. **Keep it readable** - Don't overcrowd the diagram
6. **Explain as you draw** - Talk through your thought process
7. **Be ready to zoom in** - Have detailed designs ready for any component

---

## Practice Approach

For each system:
1. ✅ Read the problem statement
2. ✅ Study the UML diagram
3. ✅ Understand class relationships
4. ✅ Review the code implementation
5. ✅ Identify design patterns used
6. ✅ Practice drawing from memory
7. ✅ Explain to someone or out loud

---

## Additional Resources

### Books
- "Head First Design Patterns" by Freeman & Robson
- "Design Patterns: Elements of Reusable Object-Oriented Software" by Gang of Four
- "Clean Code" by Robert C. Martin

### Online
- Refactoring.Guru (Design Patterns)
- SourceMaking.com
- GeeksforGeeks System Design

---

**Last Updated:** January 2026
**Total Systems with UML:** 11
