# Remaining Systems - Quick Reference Guide

## 1. Coffee Machine

### Core Classes
```java
public class CoffeeMachine {
    private Inventory inventory;
    private List<Recipe> recipes;
    
    public Coffee makeCoffee(String coffeetype) {
        Recipe recipe = findRecipe(coffeeType);
        if (inventory.hasIngredients(recipe)) {
            inventory.consume(recipe);
            return new Coffee(coffeeType);
        }
        throw new InsufficientIngredientsException();
    }
}

public class Recipe {
    private String name;
    private Map<Ingredient, Integer> ingredients;
    private double price;
}

public class Inventory {
    private Map<Ingredient, Integer> stock;
    
    public boolean hasIngredients(Recipe recipe) {
        // Check if all ingredients available
    }
    
    public void consume(Recipe recipe) {
        // Reduce inventory
    }
}

public enum Ingredient {
    COFFEE_BEANS, MILK, WATER, SUGAR
}
```

### Key Pattern: **Builder Pattern** for Coffee customization

---

## 2. Hotel Management System

### Core Classes
```java
public class Room {
    private String roomNumber;
    private RoomType type;
    private double basePrice;
    private RoomStatus status;
}

public enum RoomType { SINGLE, DOUBLE, SUITE }
public enum RoomStatus { AVAILABLE, OCCUPIED, MAINTENANCE }

public class Booking {
    private String bookingId;
    private Guest guest;
    private Room room;
    private LocalDate checkIn;
    private LocalDate checkOut;
    private BookingStatus status;
    
    public double calculateTotal() {
        long nights = ChronoUnit.DAYS.between(checkIn, checkOut);
        return nights * room.getBasePrice();
    }
}

public class Hotel {
    private String name;
    private List<Room> rooms;
    private Map<String, Booking> bookings;
    
    public Booking makeBooking(Guest guest, RoomType type, 
                               LocalDate checkIn, LocalDate checkOut) {
        Room available = findAvailableRoom(type, checkIn, checkOut);
        return new Booking(guest, available, checkIn, checkOut);
    }
}
```

---

## 3. Movie Ticket Booking System

### Core Classes
```java
public class Movie {
    private String movieId;
    private String title;
    private int duration;
    private Genre genre;
}

public class Show {
    private String showId;
    private Movie movie;
    private Theater theater;
    private LocalDateTime startTime;
    private Map<String, Seat> seats;
}

public class Seat {
    private String seatNumber;
    private SeatType type;
    private SeatStatus status;
    private double price;
}

public class Booking {
    private String bookingId;
    private Show show;
    private List<Seat> seats;
    private User user;
    private BookingStatus status;
    private Payment payment;
    
    public void confirmBooking(Payment payment) {
        if (payment.isSuccessful()) {
            status = BookingStatus.CONFIRMED;
            seats.forEach(s -> s.setStatus(SeatStatus.BOOKED));
        }
    }
}

public class Theater {
    private String theaterId;
    private String name;
    private Map<String, Show> shows;
    
    public List<Show> getShowsForMovie(String movieId) {
        return shows.values().stream()
            .filter(s -> s.getMovie().getMovieId().equals(movieId))
            .collect(Collectors.toList());
    }
}
```

---

## 4. Logging Framework

### Core Classes
```java
public enum LogLevel {
    DEBUG, INFO, WARN, ERROR, FATAL
}

public class LogMessage {
    private LogLevel level;
    private String message;
    private LocalDateTime timestamp;
    private String className;
    private String methodName;
}

public interface LogAppender {
    void append(LogMessage message);
}

public class ConsoleAppender implements LogAppender {
    @Override
    public void append(LogMessage message) {
        System.out.println(format(message));
    }
}

public class FileAppender implements LogAppender {
    private String filePath;
    
    @Override
    public void append(LogMessage message) {
        // Write to file
    }
}

public class Logger {
    private String name;
    private LogLevel level;
    private List<LogAppender> appenders;
    
    public void log(LogLevel level, String message) {
        if (level.ordinal() >= this.level.ordinal()) {
            LogMessage logMsg = new LogMessage(level, message);
            appenders.forEach(a -> a.append(logMsg));
        }
    }
    
    public void debug(String msg) { log(LogLevel.DEBUG, msg); }
    public void info(String msg) { log(LogLevel.INFO, msg); }
    public void error(String msg) { log(LogLevel.ERROR, msg); }
}

public class LoggerFactory {
    private static Map<String, Logger> loggers = new HashMap<>();
    
    public static Logger getLogger(String name) {
        return loggers.computeIfAbsent(name, Logger::new);
    }
}
```

### Key Patterns: **Chain of Responsibility, Strategy, Singleton**

---

## 5. Meeting Room Scheduler

### Core Classes
```java
public class MeetingRoom {
    private String roomId;
    private String name;
    private int capacity;
    private List<String> amenities;
    private boolean isAvailable;
}

public class Meeting {
    private String meetingId;
    private String title;
    private MeetingRoom room;
    private LocalDateTime startTime;
    private LocalDateTime endTime;
    private User organizer;
    private List<User> participants;
    private MeetingStatus status;
}

public class Calendar {
    private Map<String, List<Meeting>> roomSchedule; // roomId -> meetings
    
    public boolean isRoomAvailable(String roomId, LocalDateTime start, LocalDateTime end) {
        List<Meeting> meetings = roomSchedule.get(roomId);
        return meetings.stream().noneMatch(m -> 
            m.overlaps(start, end) && m.getStatus() != MeetingStatus.CANCELLED
        );
    }
    
    public Meeting scheduleMeeting(MeetingRoom room, LocalDateTime start, 
                                   LocalDateTime end, User organizer) {
        if (!isRoomAvailable(room.getRoomId(), start, end)) {
            throw new RoomNotAvailableException();
        }
        
        Meeting meeting = new Meeting(room, start, end, organizer);
        roomSchedule.computeIfAbsent(room.getRoomId(), k -> new ArrayList<>())
                   .add(meeting);
        return meeting;
    }
    
    public List<MeetingRoom> findAvailableRooms(LocalDateTime start, LocalDateTime end, 
                                                 int minCapacity) {
        // Find rooms that are available in the time slot
    }
}
```

---

## 6. Notification Service

### Core Classes
```java
public interface NotificationChannel {
    void send(Notification notification);
}

public class EmailChannel implements NotificationChannel {
    @Override
    public void send(Notification notification) {
        // Send email via SMTP
    }
}

public class SMSChannel implements NotificationChannel {
    @Override
    public void send(Notification notification) {
        // Send SMS via Twilio
    }
}

public class PushNotificationChannel implements NotificationChannel {
    @Override
    public void send(Notification notification) {
        // Send push notification via FCM
    }
}

public class Notification {
    private String id;
    private String recipient;
    private String subject;
    private String message;
    private NotificationType type;
    private Priority priority;
    private LocalDateTime scheduledTime;
}

public class NotificationService {
    private Map<NotificationType, NotificationChannel> channels;
    private Queue<Notification> queue;
    private ExecutorService executor;
    
    public void sendNotification(Notification notification) {
        if (notification.getScheduledTime() != null) {
            scheduleNotification(notification);
        } else {
            queue.offer(notification);
            processNotifications();
        }
    }
    
    private void processNotifications() {
        executor.submit(() -> {
            Notification notif = queue.poll();
            if (notif != null) {
                NotificationChannel channel = channels.get(notif.getType());
                channel.send(notif);
            }
        });
    }
}
```

### Key Patterns: **Observer, Strategy, Queue**

---

## 7. Task Scheduler

### Core Classes
```java
public class Task implements Comparable<Task> {
    private String taskId;
    private Runnable action;
    private long scheduledTime;
    private long delay;
    private long period; // For repeating tasks
    private TaskType type;
    
    @Override
    public int compareTo(Task other) {
        return Long.compare(this.scheduledTime, other.scheduledTime);
    }
}

public enum TaskType { ONE_TIME, REPEATING, CRON }

public class TaskScheduler {
    private PriorityQueue<Task> taskQueue;
    private ScheduledExecutorService executor;
    private volatile boolean running;
    
    public TaskScheduler(int poolSize) {
        this.taskQueue = new PriorityQueue<>();
        this.executor = Executors.newScheduledThreadPool(poolSize);
        this.running = false;
    }
    
    public String schedule(Runnable task, long delay, TimeUnit unit) {
        long scheduledTime = System.currentTimeMillis() + unit.toMillis(delay);
        Task t = new Task(task, scheduledTime, TaskType.ONE_TIME);
        taskQueue.offer(t);
        return t.getTaskId();
    }
    
    public String scheduleAtFixedRate(Runnable task, long initialDelay, 
                                     long period, TimeUnit unit) {
        Task t = new Task(task, period, TaskType.REPEATING);
        executor.scheduleAtFixedRate(task, initialDelay, period, unit);
        return t.getTaskId();
    }
    
    public void start() {
        running = true;
        executor.submit(this::run);
    }
    
    private void run() {
        while (running) {
            Task task = taskQueue.peek();
            if (task != null && System.currentTimeMillis() >= task.getScheduledTime()) {
                task = taskQueue.poll();
                executor.submit(task.getAction());
                
                if (task.getType() == TaskType.REPEATING) {
                    task.setScheduledTime(System.currentTimeMillis() + task.getPeriod());
                    taskQueue.offer(task);
                }
            }
            Thread.sleep(100);
        }
    }
    
    public void cancel(String taskId) {
        taskQueue.removeIf(t -> t.getTaskId().equals(taskId));
    }
}
```

---

## 8. Text Editor with Undo/Redo

### Core Classes
```java
public class TextEditor {
    private StringBuilder content;
    private Stack<Command> undoStack;
    private Stack<Command> redoStack;
    private int cursorPosition;
    
    public void executeCommand(Command command) {
        command.execute();
        undoStack.push(command);
        redoStack.clear(); // Clear redo stack on new command
    }
    
    public void undo() {
        if (!undoStack.isEmpty()) {
            Command command = undoStack.pop();
            command.undo();
            redoStack.push(command);
        }
    }
    
    public void redo() {
        if (!redoStack.isEmpty()) {
            Command command = redoStack.pop();
            command.execute();
            undoStack.push(command);
        }
    }
    
    public void insertText(String text, int position) {
        executeCommand(new InsertCommand(this, text, position));
    }
    
    public void deleteText(int start, int end) {
        executeCommand(new DeleteCommand(this, start, end));
    }
    
    // Package-private for commands
    void insert(String text, int position) {
        content.insert(position, text);
    }
    
    void delete(int start, int end) {
        content.delete(start, end);
    }
}

public interface Command {
    void execute();
    void undo();
}

public class InsertCommand implements Command {
    private TextEditor editor;
    private String text;
    private int position;
    
    @Override
    public void execute() {
        editor.insert(text, position);
    }
    
    @Override
    public void undo() {
        editor.delete(position, position + text.length());
    }
}

public class DeleteCommand implements Command {
    private TextEditor editor;
    private String deletedText;
    private int start, end;
    
    @Override
    public void execute() {
        deletedText = editor.getContent().substring(start, end);
        editor.delete(start, end);
    }
    
    @Override
    public void undo() {
        editor.insert(deletedText, start);
    }
}
```

### Key Patterns: **Command, Memento**

---

## 9. Expense Sharing App (Splitwise)

### Core Classes
```java
public class User {
    private String userId;
    private String name;
    private String email;
    private double totalOwed;
    private double totalOwing;
}

public class Expense {
    private String expenseId;
    private double amount;
    private User paidBy;
    private List<Split> splits;
    private String description;
    private LocalDateTime timestamp;
    
    public void validate() {
        double totalSplit = splits.stream()
            .mapToDouble(Split::getAmount)
            .sum();
        if (Math.abs(totalSplit - amount) > 0.01) {
            throw new IllegalStateException("Splits don't add up");
        }
    }
}

public abstract class Split {
    private User user;
    private double amount;
    
    public abstract void calculate(double totalAmount);
}

public class EqualSplit extends Split {
    @Override
    public void calculate(double totalAmount) {
        // Amount set by expense divider
    }
}

public class ExactSplit extends Split {
    public ExactSplit(User user, double amount) {
        super(user);
        this.amount = amount;
    }
    
    @Override
    public void calculate(double totalAmount) {
        // Amount already set
    }
}

public class PercentSplit extends Split {
    private double percent;
    
    @Override
    public void calculate(double totalAmount) {
        this.amount = totalAmount * percent / 100.0;
    }
}

public class ExpenseManager {
    private Map<String, User> users;
    private List<Expense> expenses;
    private Map<String, Map<String, Double>> balances; // user1 -> user2 -> amount
    
    public void addExpense(Expense expense) {
        expense.validate();
        expenses.add(expense);
        updateBalances(expense);
    }
    
    private void updateBalances(Expense expense) {
        User paidBy = expense.getPaidBy();
        for (Split split : expense.getSplits()) {
            User user = split.getUser();
            if (!user.equals(paidBy)) {
                double amount = split.getAmount();
                addBalance(user.getUserId(), paidBy.getUserId(), amount);
            }
        }
    }
    
    private void addBalance(String user1, String user2, double amount) {
        balances.computeIfAbsent(user1, k -> new HashMap<>())
               .merge(user2, amount, Double::sum);
    }
    
    public Map<String, Double> getBalances(String userId) {
        return balances.getOrDefault(userId, new HashMap<>());
    }
    
    public List<Transaction> settleBalances(String userId) {
        // Simplify debts using graph algorithms
        return simplifyDebts();
    }
    
    private List<Transaction> simplifyDebts() {
        // Use min-cash flow algorithm
        List<Transaction> transactions = new ArrayList<>();
        // Implementation here
        return transactions;
    }
}

public class Transaction {
    private User from;
    private User to;
    private double amount;
}
```

---

## 10. Stock Trading Platform

### Core Classes
```java
public class Stock {
    private String symbol;
    private String name;
    private double currentPrice;
    private long volume;
}

public class Order {
    private String orderId;
    private String userId;
    private String symbol;
    private OrderType type;
    private OrderSide side;
    private int quantity;
    private double price; // Limit price
    private OrderStatus status;
    private LocalDateTime createdAt;
}

public enum OrderType { MARKET, LIMIT, STOP_LOSS }
public enum OrderSide { BUY, SELL }
public enum OrderStatus { PENDING, FILLED, PARTIALLY_FILLED, CANCELLED }

public class OrderBook {
    private String symbol;
    private PriorityQueue<Order> buyOrders;   // Max heap by price
    private PriorityQueue<Order> sellOrders;  // Min heap by price
    
    public OrderBook(String symbol) {
        this.symbol = symbol;
        this.buyOrders = new PriorityQueue<>((a, b) -> 
            Double.compare(b.getPrice(), a.getPrice()));
        this.sellOrders = new PriorityQueue<>((a, b) -> 
            Double.compare(a.getPrice(), b.getPrice()));
    }
    
    public void addOrder(Order order) {
        if (order.getSide() == OrderSide.BUY) {
            buyOrders.offer(order);
        } else {
            sellOrders.offer(order);
        }
        matchOrders();
    }
    
    private void matchOrders() {
        while (!buyOrders.isEmpty() && !sellOrders.isEmpty()) {
            Order buy = buyOrders.peek();
            Order sell = sellOrders.peek();
            
            if (buy.getPrice() >= sell.getPrice()) {
                executeTrade(buy, sell);
            } else {
                break;
            }
        }
    }
    
    private void executeTrade(Order buy, Order sell) {
        int quantity = Math.min(buy.getQuantity(), sell.getQuantity());
        double price = sell.getPrice(); // Seller's price
        
        Trade trade = new Trade(buy, sell, quantity, price);
        
        buy.setQuantity(buy.getQuantity() - quantity);
        sell.setQuantity(sell.getQuantity() - quantity);
        
        if (buy.getQuantity() == 0) {
            buyOrders.poll();
            buy.setStatus(OrderStatus.FILLED);
        }
        
        if (sell.getQuantity() == 0) {
            sellOrders.poll();
            sell.setStatus(OrderStatus.FILLED);
        }
        
        notifyTrade(trade);
    }
}

public class Portfolio {
    private String userId;
    private Map<String, Integer> holdings; // symbol -> quantity
    private double cashBalance;
    
    public boolean canBuy(String symbol, int quantity, double price) {
        return cashBalance >= quantity * price;
    }
    
    public void buy(String symbol, int quantity, double price) {
        cashBalance -= quantity * price;
        holdings.merge(symbol, quantity, Integer::sum);
    }
    
    public void sell(String symbol, int quantity, double price) {
        holdings.merge(symbol, -quantity, Integer::sum);
        cashBalance += quantity * price;
    }
}
```

---

## 11. Pizza Pricing System

### Core Classes
```java
public class Pizza {
    private PizzaSize size;
    private Crust crust;
    private List<Topping> toppings;
    
    public double calculatePrice() {
        double basePrice = size.getPrice() + crust.getPrice();
        double toppingPrice = toppings.stream()
            .mapToDouble(Topping::getPrice)
            .sum();
        return basePrice + toppingPrice;
    }
}

public enum PizzaSize {
    SMALL(8.99), MEDIUM(10.99), LARGE(12.99), XLARGE(14.99);
    private double price;
    PizzaSize(double price) { this.price = price; }
    public double getPrice() { return price; }
}

public enum Crust {
    THIN(0), THICK(2), STUFFED(3);
    private double price;
    Crust(double price) { this.price = price; }
    public double getPrice() { return price; }
}

public class Topping {
    private String name;
    private double price;
    private boolean isVeg;
}

// Builder Pattern
public class PizzaBuilder {
    private Pizza pizza;
    
    public PizzaBuilder(PizzaSize size) {
        pizza = new Pizza(size);
    }
    
    public PizzaBuilder withCrust(Crust crust) {
        pizza.setCrust(crust);
        return this;
    }
    
    public PizzaBuilder addTopping(Topping topping) {
        pizza.addTopping(topping);
        return this;
    }
    
    public Pizza build() {
        return pizza;
    }
}

// Usage
Pizza pizza = new PizzaBuilder(PizzaSize.LARGE)
    .withCrust(Crust.STUFFED)
    .addTopping(new Topping("Pepperoni", 2.00))
    .addTopping(new Topping("Mushrooms", 1.50))
    .build();
    
double price = pizza.calculatePrice();
```

---

## 12. Car Rental System

### Core Classes
```java
public class Vehicle {
    private String vehicleId;
    private String make;
    private String model;
    private int year;
    private VehicleType type;
    private double dailyRate;
    private VehicleStatus status;
    private Location currentLocation;
}

public enum VehicleType { SEDAN, SUV, TRUCK, LUXURY }
public enum VehicleStatus { AVAILABLE, RENTED, MAINTENANCE }

public class Reservation {
    private String reservationId;
    private User user;
    private Vehicle vehicle;
    private LocalDate startDate;
    private LocalDate endDate;
    private Location pickupLocation;
    private Location dropoffLocation;
    private ReservationStatus status;
    
    public double calculateCost() {
        long days = ChronoUnit.DAYS.between(startDate, endDate);
        double baseCost = days * vehicle.getDailyRate();
        
        // Add insurance, taxes, etc.
        double insurance = baseCost * 0.1;
        double tax = baseCost * 0.08;
        
        return baseCost + insurance + tax;
    }
}

public class RentalSystem {
    private Map<String, Vehicle> vehicles;
    private Map<String, Reservation> reservations;
    
    public List<Vehicle> searchVehicles(VehicleType type, 
                                       LocalDate startDate, 
                                       LocalDate endDate,
                                       Location location) {
        return vehicles.values().stream()
            .filter(v -> v.getType() == type)
            .filter(v -> v.getStatus() == VehicleStatus.AVAILABLE)
            .filter(v -> !isReserved(v, startDate, endDate))
            .collect(Collectors.toList());
    }
    
    public Reservation makeReservation(User user, Vehicle vehicle,
                                      LocalDate start, LocalDate end) {
        if (isReserved(vehicle, start, end)) {
            throw new VehicleNotAvailableException();
        }
        
        Reservation reservation = new Reservation(user, vehicle, start, end);
        reservations.put(reservation.getId(), reservation);
        return reservation;
    }
}
```

---

This covers the essential 40 systems! Each design follows SOLID principles and uses appropriate design patterns. The key is to understand:

1. **Core entities** and their relationships
2. **Design patterns** applicable to each problem
3. **Extensibility** for future requirements
4. **Trade-offs** in design decisions

