# File System - Low Level Design

## UML Class Diagram

```
┌─────────────────────────────────┐
│     <<abstract>>                │
│   FileSystemEntity              │
├─────────────────────────────────┤
│ # name: String                  │
│ # path: String                  │
│ # createdAt: LocalDateTime      │
│ # modifiedAt: LocalDateTime     │
│ # size: long                    │
│ # owner: User                   │
│ # permissions: Permissions      │
├─────────────────────────────────┤
│ + getSize(): long               │
│ + display(indent: int)          │
│ + getName(): String             │
│ + getPath(): String             │
└─────────────────────────────────┘
           △
           │
    ┌──────┴──────┐
    │             │
┌─────────┐  ┌──────────┐
│  File   │  │Directory │
└─────────┘  └──────────┘


┌─────────────────────────────────┐
│           File                  │
├─────────────────────────────────┤
│ - content: String               │
│ - extension: String             │
├─────────────────────────────────┤
│ + write(data: String)           │
│ + read(): String                │
│ + getSize(): long               │
│ + display(indent: int)          │
└─────────────────────────────────┘


┌─────────────────────────────────┐
│        Directory                │
├─────────────────────────────────┤
│ - children: Map<String,Entity>  │
├─────────────────────────────────┤
│ + addChild(entity)              │
│ + removeChild(name)             │
│ + getChild(name): Entity        │
│ + listContents(): List          │
│ + getSize(): long               │
│ + display(indent: int)          │
└─────────────────────────────────┘
           │ *
           │
           │ contains
           ▼
    ┌──────────────────┐
    │FileSystemEntity  │
    └──────────────────┘


┌─────────────────────────────────┐
│   <<Singleton>>                 │
│       FileSystem                │
├─────────────────────────────────┤
│ - instance: FileSystem          │
│ - root: Directory               │
│ - pathCache: Map                │
├─────────────────────────────────┤
│ + getInstance()                 │
│ + createFile(path): File        │
│ + createDirectory(path): Dir    │
│ + delete(path): boolean         │
│ + getEntity(path): Entity       │
│ + listDirectory(path): List     │
│ + displayTree()                 │
│ + search(keyword): List         │
│ - parsePath(path): String[]     │
│ - searchHelper()                │
└─────────────────────────────────┘
           │ 1
           │
           │ 1
           ▼
    ┌──────────┐
    │Directory │
    │ (root)   │
    └──────────┘


┌─────────────────────────────────┐
│           User                  │
├─────────────────────────────────┤
│ - username: String              │
│ - userId: String                │
├─────────────────────────────────┤
│ + getUsername(): String         │
│ + getUserId(): String           │
└─────────────────────────────────┘


┌─────────────────────────────────┐
│       Permissions               │
├─────────────────────────────────┤
│ - canRead: boolean              │
│ - canWrite: boolean             │
│ - canExecute: boolean           │
└─────────────────────────────────┘
```

## Core Classes

```java
// 1. FileSystemEntity (Abstract)
public abstract class FileSystemEntity {
    protected String name;
    protected String path;
    protected LocalDateTime createdAt;
    protected LocalDateTime modifiedAt;
    protected long size;
    protected User owner;
    protected Permissions permissions;
    
    public FileSystemEntity(String name, String path, User owner) {
        this.name = name;
        this.path = path;
        this.createdAt = LocalDateTime.now();
        this.modifiedAt = LocalDateTime.now();
        this.owner = owner;
        this.permissions = new Permissions();
    }
    
    public abstract long getSize();
    public abstract void display(int indent);
    
    public String getName() { return name; }
    public String getPath() { return path; }
}

// 2. File
public class File extends FileSystemEntity {
    private String content;
    private String extension;
    
    public File(String name, String path, User owner) {
        super(name, path, owner);
        this.content = "";
        this.extension = extractExtension(name);
        this.size = 0;
    }
    
    public void write(String data) {
        this.content = data;
        this.size = data.length();
        this.modifiedAt = LocalDateTime.now();
    }
    
    public String read() {
        return content;
    }
    
    @Override
    public long getSize() {
        return size;
    }
    
    @Override
    public void display(int indent) {
        System.out.println("  ".repeat(indent) + name + " (" + size + " bytes)");
    }
    
    private String extractExtension(String fileName) {
        int lastDot = fileName.lastIndexOf('.');
        return lastDot > 0 ? fileName.substring(lastDot + 1) : "";
    }
}

// 3. Directory
public class Directory extends FileSystemEntity {
    private Map<String, FileSystemEntity> children;
    
    public Directory(String name, String path, User owner) {
        super(name, path, owner);
        this.children = new HashMap<>();
    }
    
    public void addChild(FileSystemEntity entity) {
        children.put(entity.getName(), entity);
        updateModifiedTime();
    }
    
    public void removeChild(String name) {
        children.remove(name);
        updateModifiedTime();
    }
    
    public FileSystemEntity getChild(String name) {
        return children.get(name);
    }
    
    public List<FileSystemEntity> listContents() {
        return new ArrayList<>(children.values());
    }
    
    @Override
    public long getSize() {
        return children.values().stream()
                      .mapToLong(FileSystemEntity::getSize)
                      .sum();
    }
    
    @Override
    public void display(int indent) {
        System.out.println("  ".repeat(indent) + name + "/");
        for (FileSystemEntity child : children.values()) {
            child.display(indent + 1);
        }
    }
    
    private void updateModifiedTime() {
        this.modifiedAt = LocalDateTime.now();
    }
}

// 4. FileSystem
public class FileSystem {
    private static FileSystem instance;
    private Directory root;
    private Map<String, FileSystemEntity> pathCache;
    
    private FileSystem() {
        this.root = new Directory("/", "/", new User("root"));
        this.pathCache = new HashMap<>();
        pathCache.put("/", root);
    }
    
    public static synchronized FileSystem getInstance() {
        if (instance == null) {
            instance = new FileSystem();
        }
        return instance;
    }
    
    public File createFile(String path, User user) {
        String[] parts = parsePath(path);
        String dirPath = parts[0];
        String fileName = parts[1];
        
        Directory parent = (Directory) getEntity(dirPath);
        if (parent == null) {
            throw new IllegalArgumentException("Directory not found: " + dirPath);
        }
        
        File file = new File(fileName, path, user);
        parent.addChild(file);
        pathCache.put(path, file);
        return file;
    }
    
    public Directory createDirectory(String path, User user) {
        String[] parts = parsePath(path);
        String parentPath = parts[0];
        String dirName = parts[1];
        
        Directory parent = (Directory) getEntity(parentPath);
        if (parent == null) {
            throw new IllegalArgumentException("Parent directory not found");
        }
        
        Directory dir = new Directory(dirName, path, user);
        parent.addChild(dir);
        pathCache.put(path, dir);
        return dir;
    }
    
    public boolean delete(String path) {
        FileSystemEntity entity = getEntity(path);
        if (entity == null || path.equals("/")) {
            return false;
        }
        
        String[] parts = parsePath(path);
        String parentPath = parts[0];
        Directory parent = (Directory) getEntity(parentPath);
        
        if (parent != null) {
            parent.removeChild(entity.getName());
            pathCache.remove(path);
            return true;
        }
        
        return false;
    }
    
    public FileSystemEntity getEntity(String path) {
        return pathCache.get(path);
    }
    
    public List<FileSystemEntity> listDirectory(String path) {
        FileSystemEntity entity = getEntity(path);
        if (entity instanceof Directory) {
            return ((Directory) entity).listContents();
        }
        return new ArrayList<>();
    }
    
    public void displayTree() {
        root.display(0);
    }
    
    private String[] parsePath(String path) {
        int lastSlash = path.lastIndexOf('/');
        if (lastSlash == 0) {
            return new String[]{"/", path.substring(1)};
        }
        return new String[]{path.substring(0, lastSlash), path.substring(lastSlash + 1)};
    }
    
    // Search functionality
    public List<FileSystemEntity> search(String keyword) {
        List<FileSystemEntity> results = new ArrayList<>();
        searchHelper(root, keyword, results);
        return results;
    }
    
    private void searchHelper(FileSystemEntity entity, String keyword, List<FileSystemEntity> results) {
        if (entity.getName().contains(keyword)) {
            results.add(entity);
        }
        
        if (entity instanceof Directory) {
            for (FileSystemEntity child : ((Directory) entity).listContents()) {
                searchHelper(child, keyword, results);
            }
        }
    }
}

// 5. User & Permissions
public class User {
    private String username;
    private String userId;
    
    public User(String username) {
        this.username = username;
        this.userId = UUID.randomUUID().toString();
    }
    
    public String getUsername() { return username; }
    public String getUserId() { return userId; }
}

public class Permissions {
    private boolean canRead;
    private boolean canWrite;
    private boolean canExecute;
    
    public Permissions() {
        this.canRead = true;
        this.canWrite = true;
        this.canExecute = false;
    }
    
    public Permissions(boolean read, boolean write, boolean execute) {
        this.canRead = read;
        this.canWrite = write;
        this.canExecute = execute;
    }
    
    // Getters and setters
}
```

## Key Design Patterns
- **Composite Pattern**: FileSystemEntity hierarchy
- **Singleton**: FileSystem
- **Flyweight**: For repeated file types
- **Iterator**: For traversing directory tree

---

# Library Management System - Low Level Design

## Core Classes

```java
// 1. Book
public class Book {
    private String isbn;
    private String title;
    private List<String> authors;
    private String publisher;
    private int publicationYear;
    private String category;
    private int totalCopies;
    private int availableCopies;
    
    public Book(String isbn, String title, List<String> authors, int copies) {
        this.isbn = isbn;
        this.title = title;
        this.authors = authors;
        this.totalCopies = copies;
        this.availableCopies = copies;
    }
    
    public boolean isAvailable() {
        return availableCopies > 0;
    }
    
    public void borrowCopy() {
        if (availableCopies > 0) {
            availableCopies--;
        }
    }
    
    public void returnCopy() {
        if (availableCopies < totalCopies) {
            availableCopies++;
        }
    }
    
    // Getters
    public String getIsbn() { return isbn; }
    public String getTitle() { return title; }
    public int getAvailableCopies() { return availableCopies; }
}

// 2. Member
public class Member {
    private String memberId;
    private String name;
    private String email;
    private String phone;
    private LocalDate membershipDate;
    private MembershipType type;
    private List<Loan> activeLoans;
    private int maxBooksAllowed;
    
    public Member(String name, String email, MembershipType type) {
        this.memberId = UUID.randomUUID().toString();
        this.name = name;
        this.email = email;
        this.type = type;
        this.membershipDate = LocalDate.now();
        this.activeLoans = new ArrayList<>();
        this.maxBooksAllowed = type == MembershipType.PREMIUM ? 10 : 5;
    }
    
    public boolean canBorrowMore() {
        return activeLoans.size() < maxBooksAllowed;
    }
    
    public void addLoan(Loan loan) {
        activeLoans.add(loan);
    }
    
    public void removeLoan(Loan loan) {
        activeLoans.remove(loan);
    }
    
    public boolean hasOverdueBooks() {
        return activeLoans.stream().anyMatch(Loan::isOverdue);
    }
    
    // Getters
    public String getMemberId() { return memberId; }
    public List<Loan> getActiveLoans() { return activeLoans; }
}

public enum MembershipType {
    REGULAR, PREMIUM, STUDENT
}

// 3. Loan
public class Loan {
    private String loanId;
    private Book book;
    private Member member;
    private LocalDate borrowDate;
    private LocalDate dueDate;
    private LocalDate returnDate;
    private LoanStatus status;
    private double fine;
    
    public Loan(Book book, Member member) {
        this.loanId = UUID.randomUUID().toString();
        this.book = book;
        this.member = member;
        this.borrowDate = LocalDate.now();
        this.dueDate = borrowDate.plusDays(14); // 2 weeks
        this.status = LoanStatus.ACTIVE;
        this.fine = 0.0;
    }
    
    public void returnBook() {
        this.returnDate = LocalDate.now();
        this.status = LoanStatus.RETURNED;
        
        if (isOverdue()) {
            calculateFine();
        }
    }
    
    public boolean isOverdue() {
        LocalDate today = LocalDate.now();
        return today.isAfter(dueDate) && status == LoanStatus.ACTIVE;
    }
    
    private void calculateFine() {
        long daysOverdue = ChronoUnit.DAYS.between(dueDate, returnDate);
        fine = daysOverdue * 1.0; // $1 per day
    }
    
    // Getters
    public String getLoanId() { return loanId; }
    public Book getBook() { return book; }
    public double getFine() { return fine; }
}

public enum LoanStatus {
    ACTIVE, RETURNED, OVERDUE
}

// 4. Library
public class Library {
    private static Library instance;
    private String name;
    private Map<String, Book> bookCatalog; // ISBN -> Book
    private Map<String, Member> members;   // MemberID -> Member
    private Map<String, Loan> loans;       // LoanID -> Loan
    
    private Library(String name) {
        this.name = name;
        this.bookCatalog = new HashMap<>();
        this.members = new HashMap<>();
        this.loans = new HashMap<>();
    }
    
    public static synchronized Library getInstance(String name) {
        if (instance == null) {
            instance = new Library(name);
        }
        return instance;
    }
    
    public void addBook(Book book) {
        bookCatalog.put(book.getIsbn(), book);
    }
    
    public void registerMember(Member member) {
        members.put(member.getMemberId(), member);
    }
    
    public Loan borrowBook(String isbn, String memberId) {
        Book book = bookCatalog.get(isbn);
        Member member = members.get(memberId);
        
        if (book == null || member == null) {
            throw new IllegalArgumentException("Book or Member not found");
        }
        
        if (!book.isAvailable()) {
            throw new IllegalStateException("Book not available");
        }
        
        if (!member.canBorrowMore()) {
            throw new IllegalStateException("Member reached borrow limit");
        }
        
        if (member.hasOverdueBooks()) {
            throw new IllegalStateException("Member has overdue books");
        }
        
        book.borrowCopy();
        Loan loan = new Loan(book, member);
        member.addLoan(loan);
        loans.put(loan.getLoanId(), loan);
        
        return loan;
    }
    
    public void returnBook(String loanId) {
        Loan loan = loans.get(loanId);
        if (loan == null) {
            throw new IllegalArgumentException("Loan not found");
        }
        
        loan.returnBook();
        loan.getBook().returnCopy();
        loan.getMember().removeLoan(loan);
        
        if (loan.getFine() > 0) {
            System.out.println("Fine: $" + loan.getFine());
        }
    }
    
    public List<Book> searchBooks(String keyword) {
        return bookCatalog.values().stream()
            .filter(b -> b.getTitle().toLowerCase().contains(keyword.toLowerCase()))
            .collect(Collectors.toList());
    }
    
    public List<Loan> getOverdueLoans() {
        return loans.values().stream()
            .filter(Loan::isOverdue)
            .collect(Collectors.toList());
    }
}
```

---

# Vending Machine - Low Level Design

## Core Classes

```java
// 1. Product
public class Product {
    private String id;
    private String name;
    private double price;
    private ProductType type;
    
    public Product(String id, String name, double price, ProductType type) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.type = type;
    }
    
    // Getters
    public String getId() { return id; }
    public double getPrice() { return price; }
    public String getName() { return name; }
}

public enum ProductType {
    BEVERAGE, SNACK, CANDY
}

// 2. Inventory
public class Inventory {
    private Map<String, Integer> stock; // productId -> quantity
    private Map<String, Product> products;
    
    public Inventory() {
        this.stock = new HashMap<>();
        this.products = new HashMap<>();
    }
    
    public void addProduct(Product product, int quantity) {
        products.put(product.getId(), product);
        stock.put(product.getId(), stock.getOrDefault(product.getId(), 0) + quantity);
    }
    
    public boolean isAvailable(String productId) {
        return stock.getOrDefault(productId, 0) > 0;
    }
    
    public void reduceStock(String productId) {
        if (isAvailable(productId)) {
            stock.put(productId, stock.get(productId) - 1);
        }
    }
    
    public Product getProduct(String productId) {
        return products.get(productId);
    }
    
    public Map<String, Integer> getStock() {
        return new HashMap<>(stock);
    }
}

// 3. VendingMachine (State Pattern)
public class VendingMachine {
    private Inventory inventory;
    private VendingMachineState state;
    private double currentAmount;
    private String selectedProductId;
    
    public VendingMachine() {
        this.inventory = new Inventory();
        this.state = new IdleState(this);
        this.currentAmount = 0.0;
    }
    
    public void insertMoney(double amount) {
        state.insertMoney(amount);
    }
    
    public void selectProduct(String productId) {
        state.selectProduct(productId);
    }
    
    public void dispense() {
        state.dispense();
    }
    
    public void returnMoney() {
        state.returnMoney();
    }
    
    public void addProduct(Product product, int quantity) {
        inventory.addProduct(product, quantity);
    }
    
    // Package-private methods for state transitions
    void setState(VendingMachineState state) {
        this.state = state;
    }
    
    void addAmount(double amount) {
        this.currentAmount += amount;
    }
    
    double getCurrentAmount() {
        return currentAmount;
    }
    
    void setSelectedProduct(String productId) {
        this.selectedProductId = productId;
    }
    
    String getSelectedProduct() {
        return selectedProductId;
    }
    
    Inventory getInventory() {
        return inventory;
    }
    
    void resetTransaction() {
        currentAmount = 0.0;
        selectedProductId = null;
    }
}

// 4. State Pattern
public interface VendingMachineState {
    void insertMoney(double amount);
    void selectProduct(String productId);
    void dispense();
    void returnMoney();
}

public class IdleState implements VendingMachineState {
    private VendingMachine machine;
    
    public IdleState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertMoney(double amount) {
        machine.addAmount(amount);
        System.out.println("Inserted: $" + amount);
        machine.setState(new HasMoneyState(machine));
    }
    
    @Override
    public void selectProduct(String productId) {
        System.out.println("Please insert money first");
    }
    
    @Override
    public void dispense() {
        System.out.println("Please insert money and select product");
    }
    
    @Override
    public void returnMoney() {
        System.out.println("No money to return");
    }
}

public class HasMoneyState implements VendingMachineState {
    private VendingMachine machine;
    
    public HasMoneyState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertMoney(double amount) {
        machine.addAmount(amount);
        System.out.println("Inserted: $" + amount + ", Total: $" + machine.getCurrentAmount());
    }
    
    @Override
    public void selectProduct(String productId) {
        Inventory inventory = machine.getInventory();
        Product product = inventory.getProduct(productId);
        
        if (product == null) {
            System.out.println("Invalid product");
            return;
        }
        
        if (!inventory.isAvailable(productId)) {
            System.out.println("Product out of stock");
            return;
        }
        
        if (machine.getCurrentAmount() < product.getPrice()) {
            System.out.println("Insufficient funds. Need $" + product.getPrice());
            return;
        }
        
        machine.setSelectedProduct(productId);
        machine.setState(new DispenseState(machine));
        dispense();
    }
    
    @Override
    public void dispense() {
        System.out.println("Please select a product first");
    }
    
    @Override
    public void returnMoney() {
        System.out.println("Returning $" + machine.getCurrentAmount());
        machine.resetTransaction();
        machine.setState(new IdleState(machine));
    }
}

public class DispenseState implements VendingMachineState {
    private VendingMachine machine;
    
    public DispenseState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertMoney(double amount) {
        System.out.println("Please wait, dispensing product");
    }
    
    @Override
    public void selectProduct(String productId) {
        System.out.println("Already dispensing");
    }
    
    @Override
    public void dispense() {
        Inventory inventory = machine.getInventory();
        String productId = machine.getSelectedProduct();
        Product product = inventory.getProduct(productId);
        
        // Dispense product
        inventory.reduceStock(productId);
        System.out.println("Dispensing: " + product.getName());
        
        // Return change
        double change = machine.getCurrentAmount() - product.getPrice();
        if (change > 0) {
            System.out.println("Change: $" + change);
        }
        
        machine.resetTransaction();
        machine.setState(new IdleState(machine));
    }
    
    @Override
    public void returnMoney() {
        System.out.println("Cannot return money during dispensing");
    }
}
```

## Usage Example
```java
public class VendingMachineDemo {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine();
        
        // Stock products
        machine.addProduct(new Product("P1", "Coke", 1.50, ProductType.BEVERAGE), 10);
        machine.addProduct(new Product("P2", "Chips", 2.00, ProductType.SNACK), 5);
        
        // Transaction
        machine.insertMoney(2.00);
        machine.selectProduct("P1"); // Dispenses Coke, returns $0.50 change
    }
}
```

---

