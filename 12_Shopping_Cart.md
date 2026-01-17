# Online Shopping Cart - Low Level Design

## Core Classes

```java
// 1. Product
public class Product {
    private String id;
    private String name;
    private String description;
    private double price;
    private String category;
    private int availableQuantity;
    
    public Product(String id, String name, double price, int quantity) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.availableQuantity = quantity;
    }
    
    public boolean isAvailable(int quantity) {
        return availableQuantity >= quantity;
    }
    
    public void reduceQuantity(int quantity) {
        if (availableQuantity >= quantity) {
            availableQuantity -= quantity;
        }
    }
    
    // Getters
    public String getId() { return id; }
    public double getPrice() { return price; }
    public String getName() { return name; }
}

// 2. CartItem
public class CartItem {
    private Product product;
    private int quantity;
    
    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
    
    public void incrementQuantity(int amount) {
        this.quantity += amount;
    }
    
    public void decrementQuantity(int amount) {
        this.quantity = Math.max(0, this.quantity - amount);
    }
    
    public double getSubtotal() {
        return product.getPrice() * quantity;
    }
    
    // Getters
    public Product getProduct() { return product; }
    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }
}

// 3. ShoppingCart
public class ShoppingCart {
    private String cartId;
    private String userId;
    private Map<String, CartItem> items; // productId -> CartItem
    private CartStatus status;
    
    public ShoppingCart(String userId) {
        this.cartId = UUID.randomUUID().toString();
        this.userId = userId;
        this.items = new HashMap<>();
        this.status = CartStatus.ACTIVE;
    }
    
    public void addItem(Product product, int quantity) {
        if (!product.isAvailable(quantity)) {
            throw new IllegalStateException("Product not available");
        }
        
        String productId = product.getId();
        if (items.containsKey(productId)) {
            items.get(productId).incrementQuantity(quantity);
        } else {
            items.put(productId, new CartItem(product, quantity));
        }
    }
    
    public void removeItem(String productId) {
        items.remove(productId);
    }
    
    public void updateQuantity(String productId, int newQuantity) {
        if (items.containsKey(productId)) {
            if (newQuantity <= 0) {
                removeItem(productId);
            } else {
                CartItem item = items.get(productId);
                if (item.getProduct().isAvailable(newQuantity)) {
                    item.setQuantity(newQuantity);
                }
            }
        }
    }
    
    public double getTotal() {
        return items.values().stream()
                   .mapToDouble(CartItem::getSubtotal)
                   .sum();
    }
    
    public int getItemCount() {
        return items.values().stream()
                   .mapToInt(CartItem::getQuantity)
                   .sum();
    }
    
    public void clear() {
        items.clear();
    }
    
    public List<CartItem> getItems() {
        return new ArrayList<>(items.values());
    }
    
    public void checkout() {
        status = CartStatus.CHECKED_OUT;
    }
    
    // Getters
    public String getCartId() { return cartId; }
    public String getUserId() { return userId; }
    public CartStatus getStatus() { return status; }
}

public enum CartStatus {
    ACTIVE, CHECKED_OUT, ABANDONED
}

// 4. Order
public class Order {
    private String orderId;
    private String userId;
    private List<OrderItem> items;
    private double totalAmount;
    private OrderStatus status;
    private Address shippingAddress;
    private Payment payment;
    private LocalDateTime createdAt;
    
    public Order(String userId, ShoppingCart cart, Address address) {
        this.orderId = UUID.randomUUID().toString();
        this.userId = userId;
        this.items = new ArrayList<>();
        this.shippingAddress = address;
        this.createdAt = LocalDateTime.now();
        this.status = OrderStatus.PENDING;
        
        // Convert cart items to order items
        for (CartItem cartItem : cart.getItems()) {
            items.add(new OrderItem(
                cartItem.getProduct(),
                cartItem.getQuantity(),
                cartItem.getProduct().getPrice()
            ));
        }
        
        this.totalAmount = cart.getTotal();
    }
    
    public void confirmPayment(Payment payment) {
        this.payment = payment;
        if (payment.getStatus() == PaymentStatus.SUCCESS) {
            this.status = OrderStatus.CONFIRMED;
        }
    }
    
    public void ship() {
        if (status == OrderStatus.CONFIRMED) {
            status = OrderStatus.SHIPPED;
        }
    }
    
    public void deliver() {
        if (status == OrderStatus.SHIPPED) {
            status = OrderStatus.DELIVERED;
        }
    }
    
    public void cancel() {
        if (status == OrderStatus.PENDING || status == OrderStatus.CONFIRMED) {
            status = OrderStatus.CANCELLED;
        }
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public OrderStatus getStatus() { return status; }
    public double getTotalAmount() { return totalAmount; }
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED, RETURNED
}

// 5. OrderItem
public class OrderItem {
    private Product product;
    private int quantity;
    private double priceAtPurchase;
    
    public OrderItem(Product product, int quantity, double price) {
        this.product = product;
        this.quantity = quantity;
        this.priceAtPurchase = price;
    }
    
    public double getSubtotal() {
        return priceAtPurchase * quantity;
    }
    
    // Getters
}

// 6. Payment
public class Payment {
    private String paymentId;
    private String orderId;
    private double amount;
    private PaymentMethod method;
    private PaymentStatus status;
    private LocalDateTime timestamp;
    
    public Payment(String orderId, double amount, PaymentMethod method) {
        this.paymentId = UUID.randomUUID().toString();
        this.orderId = orderId;
        this.amount = amount;
        this.method = method;
        this.status = PaymentStatus.PENDING;
        this.timestamp = LocalDateTime.now();
    }
    
    public void process() {
        // Process payment through payment gateway
        boolean success = method.processPayment(amount);
        this.status = success ? PaymentStatus.SUCCESS : PaymentStatus.FAILED;
    }
    
    // Getters
    public PaymentStatus getStatus() { return status; }
}

public enum PaymentStatus {
    PENDING, SUCCESS, FAILED, REFUNDED
}

// 7. PaymentMethod (Strategy Pattern)
public interface PaymentMethod {
    boolean processPayment(double amount);
    String getPaymentDetails();
}

public class CreditCardPayment implements PaymentMethod {
    private String cardNumber;
    private String cvv;
    private String expiryDate;
    
    public CreditCardPayment(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public boolean processPayment(double amount) {
        // Integrate with payment gateway
        System.out.println("Processing credit card payment: $" + amount);
        return true; // Simplified
    }
    
    @Override
    public String getPaymentDetails() {
        return "Credit Card ending in " + cardNumber.substring(cardNumber.length() - 4);
    }
}

public class PayPalPayment implements PaymentMethod {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
        return true;
    }
    
    @Override
    public String getPaymentDetails() {
        return "PayPal: " + email;
    }
}

// 8. Address
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    private String country;
    
    public Address(String street, String city, String state, String zipCode, String country) {
        this.street = street;
        this.city = city;
        this.state = state;
        this.zipCode = zipCode;
        this.country = country;
    }
    
    @Override
    public String toString() {
        return street + ", " + city + ", " + state + " " + zipCode + ", " + country;
    }
}

// 9. User
public class User {
    private String userId;
    private String name;
    private String email;
    private List<Address> addresses;
    private ShoppingCart cart;
    private List<Order> orderHistory;
    
    public User(String name, String email) {
        this.userId = UUID.randomUUID().toString();
        this.name = name;
        this.email = email;
        this.addresses = new ArrayList<>();
        this.cart = new ShoppingCart(userId);
        this.orderHistory = new ArrayList<>();
    }
    
    public void addAddress(Address address) {
        addresses.add(address);
    }
    
    public Order checkout(Address shippingAddress, PaymentMethod paymentMethod) {
        if (cart.getItems().isEmpty()) {
            throw new IllegalStateException("Cart is empty");
        }
        
        // Create order
        Order order = new Order(userId, cart, shippingAddress);
        
        // Process payment
        Payment payment = new Payment(order.getOrderId(), order.getTotalAmount(), paymentMethod);
        payment.process();
        order.confirmPayment(payment);
        
        if (payment.getStatus() == PaymentStatus.SUCCESS) {
            // Reduce product quantities
            for (CartItem item : cart.getItems()) {
                item.getProduct().reduceQuantity(item.getQuantity());
            }
            
            // Clear cart
            cart.clear();
            cart.checkout();
            
            // Add to order history
            orderHistory.add(order);
            
            // Create new cart
            cart = new ShoppingCart(userId);
        }
        
        return order;
    }
    
    // Getters
    public ShoppingCart getCart() { return cart; }
    public List<Order> getOrderHistory() { return orderHistory; }
}

// 10. Shopping System
public class ShoppingSystem {
    private static ShoppingSystem instance;
    private Map<String, Product> productCatalog;
    private Map<String, User> users;
    
    private ShoppingSystem() {
        this.productCatalog = new HashMap<>();
        this.users = new HashMap<>();
    }
    
    public static synchronized ShoppingSystem getInstance() {
        if (instance == null) {
            instance = new ShoppingSystem();
        }
        return instance;
    }
    
    public void addProduct(Product product) {
        productCatalog.put(product.getId(), product);
    }
    
    public Product getProduct(String productId) {
        return productCatalog.get(productId);
    }
    
    public void registerUser(User user) {
        users.put(user.getUserId(), user);
    }
    
    public User getUser(String userId) {
        return users.get(userId);
    }
    
    public List<Product> searchProducts(String keyword) {
        return productCatalog.values().stream()
            .filter(p -> p.getName().toLowerCase().contains(keyword.toLowerCase()))
            .collect(Collectors.toList());
    }
}
```

## Usage Example
```java
public class ShoppingDemo {
    public static void main(String[] args) {
        ShoppingSystem system = ShoppingSystem.getInstance();
        
        // Add products
        Product laptop = new Product("P1", "Laptop", 999.99, 10);
        Product mouse = new Product("P2", "Mouse", 29.99, 50);
        system.addProduct(laptop);
        system.addProduct(mouse);
        
        // Create user
        User user = new User("John Doe", "john@example.com");
        Address address = new Address("123 Main St", "NYC", "NY", "10001", "USA");
        user.addAddress(address);
        
        // Add to cart
        user.getCart().addItem(laptop, 1);
        user.getCart().addItem(mouse, 2);
        
        System.out.println("Cart Total: $" + user.getCart().getTotal());
        
        // Checkout
        PaymentMethod payment = new CreditCardPayment("1234567890123456", "123", "12/25");
        Order order = user.checkout(address, payment);
        
        System.out.println("Order " + order.getOrderId() + " placed successfully!");
        System.out.println("Status: " + order.getStatus());
    }
}
```

## Key Design Patterns
- **Singleton**: ShoppingSystem
- **Strategy**: PaymentMethod
- **State**: Order status transitions
- **Observer**: For price changes, stock alerts

## Key Points
1. Cart persistence
2. Inventory management
3. Payment processing
4. Order tracking
5. Thread-safe operations

---

