# 🚀 QUICK START GUIDE - Amazon SDE 2 LLD Interview

## ⚡ 5-Minute Quick Start

### If you have 2 weeks:
1. Open `INTERVIEW_GUIDE.md` - Read completely (30 mins)
2. Study detailed systems in order (1 per day)
3. Implement 5 systems from scratch
4. Mock interview in Week 2

### If you have 1 week:
1. Read `INTERVIEW_GUIDE.md` summary (15 mins)
2. Study these 5 systems:
   - `01_Parking_Lot_System.md`
   - `03_LRU_Cache.md`
   - `04_URL_Shortener.md`
   - `12_Shopping_Cart.md`
   - `05_Rate_Limiter.md`
3. Skim `REMAINING_SYSTEMS.md` for patterns
4. Mock interview on Day 6

### If you have 2-3 days:
1. Read `INTERVIEW_GUIDE.md` (1 hour)
2. Study these 3 systems deeply:
   - `01_Parking_Lot_System.md` (Complex state)
   - `03_LRU_Cache.md` (Data structure)
   - `12_Shopping_Cart.md` (Business logic)
3. Quick scan of `REMAINING_SYSTEMS.md` (30 mins)
4. Review design patterns (30 mins)
5. Practice explaining 1 system aloud

### If you have TODAY only:
**Morning (2 hours):**
- Read this file completely
- Read `INTERVIEW_GUIDE.md` - Interview Strategy section
- Study `01_Parking_Lot_System.md` deeply

**Afternoon (2 hours):**
- Study `03_LRU_Cache.md`
- Quick scan of `REMAINING_SYSTEMS.md`

**Evening (1 hour):**
- Review design patterns in `INTERVIEW_GUIDE.md`
- Review SOLID principles
- Practice explaining Parking Lot aloud

---

## 🎯 The 5-Step Interview Framework

### Step 1: Clarify (5 mins)
**Questions to ask:**
- "What are the core features needed?"
- "What's the expected scale?"
- "Any specific constraints?"
- "Should I consider concurrency?"

### Step 2: Entities (5 mins)
**What to do:**
- List main actors (User, Admin, etc.)
- Identify core objects
- Define relationships
- Quick sketch

### Step 3: Design (20 mins)
**What to code:**
- Core classes with attributes
- Key methods
- Relationships (inheritance/composition)
- Apply 2-3 design patterns
- Explain each decision

### Step 4: Edge Cases (10 mins)
**What to discuss:**
- Thread safety
- Error handling
- Validation
- Scalability

### Step 5: Extensions (5 mins)
**What to mention:**
- Possible improvements
- Trade-offs
- Alternative approaches

---

## 📋 Must-Know Design Patterns (Top 10)

| Pattern | When | Example |
|---------|------|---------|
| **Singleton** | One instance | Database, Logger |
| **Factory** | Create objects | VehicleFactory |
| **Builder** | Complex construction | PizzaBuilder |
| **Strategy** | Swap algorithms | PaymentMethod |
| **Observer** | Event notification | Stock updates |
| **State** | State transitions | OrderStatus |
| **Command** | Encapsulate action | Undo/Redo |
| **Composite** | Tree structure | FileSystem |
| **Decorator** | Add features | Stream wrappers |
| **Template** | Algorithm skeleton | DataProcessor |

---

## 🎨 SOLID Principles - One Line Each

- **S**: One class, one responsibility
- **O**: Open for extension, closed for modification
- **L**: Subclass can replace parent
- **I**: Small, specific interfaces
- **D**: Depend on abstractions, not concrete classes

---

## 💡 Common Interview Questions - Quick Answers

**Q: "Make it thread-safe"**
→ Use `synchronized`, `ConcurrentHashMap`, `AtomicInteger`, or `ReadWriteLock`

**Q: "How to scale to millions?"**
→ Database sharding, caching (Redis), load balancing, async queues, CDN

**Q: "Trade-offs in your design?"**
→ Always mention: Time vs Space, Simplicity vs Flexibility, Consistency vs Availability

**Q: "How to test this?"**
→ Unit tests, integration tests, mock dependencies, edge cases

**Q: "Add feature X?"**
→ Show extensibility: "We can extend interface Y without modifying existing code"

---

## 🎓 System Priority List

### Must Study (Essential) ⭐⭐⭐
1. **Parking Lot** - State management, multiple patterns
2. **LRU Cache** - Data structure design, O(1) operations
3. **URL Shortener** - Scalability, encoding algorithms
4. **Shopping Cart** - Business logic, payment flow
5. **Elevator** - Algorithm design, optimization

### Should Study (Important) ⭐⭐
6. Rate Limiter - Multiple algorithms
7. Chess Game - Complex validation logic
8. File System - Composite pattern
9. ATM System - State pattern
10. Restaurant Management - End-to-end flow

### Good to Know (Bonus) ⭐
11-15. Systems in `REMAINING_SYSTEMS.md` - Pattern practice

---

## 🔥 Last-Hour Cramming (If Interview is in 1 hour!)

**60-45 mins before:**
- Read this file completely
- Skim `INTERVIEW_GUIDE.md` interview strategy
- Review SOLID principles above

**45-30 mins before:**
- Read `01_Parking_Lot_System.md` quickly
- Focus on class structure and patterns used
- Don't worry about memorizing code

**30-15 mins before:**
- Review the 5-step framework above
- Review top 10 design patterns
- Practice clarifying questions

**15-0 mins before:**
- Take deep breaths
- Remember: They want to see your thinking process
- Be ready to ask questions
- Stay calm and confident

**During interview:**
- Think aloud
- Ask clarifying questions
- Start simple, then extend
- Be open to feedback
- Manage time wisely

---

## 🎯 Example Opening (First 2 minutes)

**Interviewer:** "Design a parking lot system"

**You:** 
"Great! Let me clarify a few requirements:
1. Is this for a single building or multiple locations?
2. What types of vehicles should we support?
3. Do we need pricing/payment functionality?
4. Should we track available spots in real-time?
5. Any specific capacity constraints?

Based on your answers, I'll design a system with these core entities:
- Vehicle (abstract): Car, Bike, Truck
- ParkingSpot (abstract): Compact, Large, Handicapped
- ParkingLot: Main controller
- Ticket: For entry/exit tracking
- PricingStrategy: Flexible pricing

I'll use Singleton for ParkingLot, Strategy for pricing, and Factory for creating vehicles. Does this sound good?"

---

## ✅ Pre-Interview Checklist (2 mins)

- [ ] I know the 5-step framework
- [ ] I can name 5 design patterns
- [ ] I understand SOLID principles
- [ ] I can explain 1 system end-to-end
- [ ] I'm calm and confident

**If all checked → You're ready! Go ace that interview! 💪**

---

## 📞 File Reference

| What you need | Open this file |
|---------------|----------------|
| Interview strategy | `INTERVIEW_GUIDE.md` |
| First system to study | `01_Parking_Lot_System.md` |
| Data structures | `03_LRU_Cache.md` |
| Business logic example | `12_Shopping_Cart.md` |
| Quick patterns | `REMAINING_SYSTEMS.md` |
| Complete guide | `00_README.md` |

---

## 🚀 You've Got This!

Remember:
- **Be calm** - Take your time
- **Ask questions** - Show you care about requirements
- **Think aloud** - Let them see your process
- **Be flexible** - Adapt based on feedback
- **Have fun** - Enjoy the problem-solving!

**Good luck! 🎉**
