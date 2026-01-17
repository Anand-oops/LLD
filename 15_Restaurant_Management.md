# Restaurant Management System - Low Level Design

## Core Classes

```java
// 1. Restaurant
public class Restaurant {
    private String name;
    private Location location;
    private Menu menu;
    private List<Table> tables;
    private KitchenService kitchen;
    private ReservationManager reservationManager;
}

// 2. Table
public class Table {
    private int tableNumber;
    private int capacity;
    private TableStatus status;
    private List<Order> currentOrders;
    
    public boolean isAvailable() {
        return status == TableStatus.AVAILABLE;
    }
}

public enum TableStatus {
    AVAILABLE, OCCUPIED, RESERVED
}

// 3. Menu & MenuItem
public class Menu {
    private Map<String, MenuItem> items;
    private List<MenuCategory> categories;
    
    public List<MenuItem> getItemsByCategory(MenuCategory category) {
        return items.values().stream()
            .filter(item -> item.getCategory() == category)
            .collect(Collectors.toList());
    }
}

public class MenuItem {
    private String itemId;
    private String name;
    private String description;
    private double price;
    private MenuCategory category;
    private boolean isAvailable;
    private List<String> ingredients;
}

public enum MenuCategory {
    APPETIZER, MAIN_COURSE, DESSERT, BEVERAGE
}

// 4. Order
public class Order {
    private String orderId;
    private Table table;
    private List<OrderItem> items;
    private OrderStatus status;
    private LocalDateTime createdAt;
    private Staff server;
    
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(OrderItem::getSubtotal)
            .sum();
    }
    
    public void addItem(MenuItem item, int quantity) {
        items.add(new OrderItem(item, quantity));
        notifyKitchen();
    }
    
    private void notifyKitchen() {
        // Send order to kitchen
    }
}

public class OrderItem {
    private MenuItem item;
    private int quantity;
    private String specialInstructions;
    private OrderItemStatus status;
    
    public double getSubtotal() {
        return item.getPrice() * quantity;
    }
}

public enum OrderStatus {
    PLACED, PREPARING, READY, SERVED, PAID
}

public enum OrderItemStatus {
    PENDING, PREPARING, READY, SERVED
}

// 5. Reservation
public class Reservation {
    private String reservationId;
    private Customer customer;
    private LocalDateTime reservationTime;
    private int partySize;
    private Table assignedTable;
    private ReservationStatus status;
}

public class ReservationManager {
    private Map<String, Reservation> reservations;
    
    public Reservation makeReservation(Customer customer, 
                                      LocalDateTime time, 
                                      int partySize) {
        Table available = findAvailableTable(time, partySize);
        if (available == null) {
            throw new NoTableAvailableException();
        }
        
        Reservation reservation = new Reservation(customer, time, partySize);
        reservation.setAssignedTable(available);
        reservations.put(reservation.getId(), reservation);
        return reservation;
    }
    
    private Table findAvailableTable(LocalDateTime time, int partySize) {
        // Find suitable table
        return null;
    }
}

// 6. Bill
public class Bill {
    private String billId;
    private Order order;
    private double subtotal;
    private double tax;
    private double tip;
    private double total;
    private BillStatus status;
    
    public void calculate() {
        subtotal = order.calculateTotal();
        tax = subtotal * 0.08; // 8% tax
        total = subtotal + tax + tip;
    }
    
    public void addTip(double tipAmount) {
        this.tip = tipAmount;
        calculate();
    }
}

// 7. Kitchen
public class KitchenService {
    private Queue<Order> orderQueue;
    private List<Chef> chefs;
    
    public void receiveOrder(Order order) {
        orderQueue.offer(order);
        assignToChef();
    }
    
    private void assignToChef() {
        Chef available = findAvailableChef();
        if (available != null) {
            Order order = orderQueue.poll();
            available.prepareOrder(order);
        }
    }
}

public class Chef extends Staff {
    private List<Order> currentOrders;
    private Specialization specialization;
    
    public void prepareOrder(Order order) {
        currentOrders.add(order);
        order.setStatus(OrderStatus.PREPARING);
        // Prepare order
        order.setStatus(OrderStatus.READY);
        currentOrders.remove(order);
    }
}

// 8. Staff
public abstract class Staff {
    private String staffId;
    private String name;
    private StaffRole role;
    private double salary;
}

public enum StaffRole {
    MANAGER, CHEF, WAITER, BARTENDER
}
```

## Usage Example
```java
public class RestaurantDemo {
    public static void main(String[] args) {
        Restaurant restaurant = new Restaurant("Delicious Eats");
        
        // Customer makes reservation
        Customer customer = new Customer("John Doe", "555-1234");
        Reservation reservation = restaurant.makeReservation(
            customer, 
            LocalDateTime.now().plusHours(2), 
            4
        );
        
        // Customer arrives and places order
        Table table = reservation.getAssignedTable();
        Order order = new Order(table);
        order.addItem(menu.getItem("PASTA"), 2);
        order.addItem(menu.getItem("WINE"), 1);
        
        // Kitchen prepares
        kitchen.receiveOrder(order);
        
        // Generate bill
        Bill bill = new Bill(order);
        bill.addTip(10.00);
        bill.calculate();
    }
}
```

---

# Food Ordering System (Uber Eats/DoorDash) - Low Level Design

## Core Classes

```java
// 1. Restaurant
public class Restaurant {
    private String restaurantId;
    private String name;
    private Location location;
    private Menu menu;
    private double rating;
    private List<Review> reviews;
    private boolean isOpen;
    private int deliveryRadius; // in km
}

// 2. Customer
public class Customer extends User {
    private List<Address> savedAddresses;
    private PaymentMethod defaultPayment;
    private List<Order> orderHistory;
    private Cart cart;
}

// 3. Cart
public class Cart {
    private String customerId;
    private Restaurant restaurant;
    private List<CartItem> items;
    
    public void addItem(MenuItem item, int quantity) {
        // Can only order from one restaurant at a time
        if (restaurant != null && !restaurant.equals(item.getRestaurant())) {
            throw new IllegalStateException("Can't add items from different restaurant");
        }
        
        CartItem existing = findItem(item.getId());
        if (existing != null) {
            existing.incrementQuantity(quantity);
        } else {
            items.add(new CartItem(item, quantity));
        }
    }
    
    public double getSubtotal() {
        return items.stream()
            .mapToDouble(CartItem::getPrice)
            .sum();
    }
}

// 4. Order
public class Order {
    private String orderId;
    private Customer customer;
    private Restaurant restaurant;
    private List<OrderItem> items;
    private Address deliveryAddress;
    private OrderStatus status;
    private double itemsTotal;
    private double deliveryFee;
    private double tax;
    private double tip;
    private double total;
    private DeliveryAgent assignedAgent;
    private LocalDateTime orderedAt;
    private LocalDateTime estimatedDelivery;
    
    public void calculateTotals() {
        itemsTotal = items.stream()
            .mapToDouble(OrderItem::getSubtotal)
            .sum();
        
        deliveryFee = calculateDeliveryFee();
        tax = itemsTotal * 0.08;
        total = itemsTotal + deliveryFee + tax + tip;
    }
    
    private double calculateDeliveryFee() {
        double distance = customer.getAddress()
            .distanceTo(restaurant.getLocation());
        return Math.max(2.99, distance * 0.5); // $0.50 per km, min $2.99
    }
}

public enum OrderStatus {
    PLACED, CONFIRMED, PREPARING, READY_FOR_PICKUP, 
    PICKED_UP, ON_THE_WAY, DELIVERED, CANCELLED
}

// 5. DeliveryAgent
public class DeliveryAgent {
    private String agentId;
    private String name;
    private Vehicle vehicle;
    private Location currentLocation;
    private AgentStatus status;
    private List<Order> activeDeliveries;
    private double rating;
    
    public void acceptOrder(Order order) {
        if (status != AgentStatus.AVAILABLE) {
            throw new IllegalStateException("Agent not available");
        }
        
        activeDeliveries.add(order);
        status = AgentStatus.BUSY;
        order.setAssignedAgent(this);
    }
    
    public void updateLocation(Location location) {
        this.currentLocation = location;
        // Notify customers of orders
    }
    
    public void completeDelivery(Order order) {
        activeDeliveries.remove(order);
        order.setStatus(OrderStatus.DELIVERED);
        
        if (activeDeliveries.isEmpty()) {
            status = AgentStatus.AVAILABLE;
        }
    }
}

public enum AgentStatus {
    AVAILABLE, BUSY, OFFLINE
}

// 6. Delivery Assignment Service
public class DeliveryAssignmentService {
    private List<DeliveryAgent> agents;
    private Queue<Order> pendingOrders;
    
    public void assignDelivery(Order order) {
        DeliveryAgent bestAgent = findBestAgent(order);
        if (bestAgent != null) {
            bestAgent.acceptOrder(order);
        } else {
            pendingOrders.offer(order);
        }
    }
    
    private DeliveryAgent findBestAgent(Order order) {
        Location restaurantLocation = order.getRestaurant().getLocation();
        
        return agents.stream()
            .filter(a -> a.getStatus() == AgentStatus.AVAILABLE)
            .min((a1, a2) -> Double.compare(
                a1.getCurrentLocation().distanceTo(restaurantLocation),
                a2.getCurrentLocation().distanceTo(restaurantLocation)
            ))
            .orElse(null);
    }
}

// 7. Search Service
public class SearchService {
    private List<Restaurant> restaurants;
    
    public List<Restaurant> searchRestaurants(String query, 
                                             Location userLocation, 
                                             SearchFilters filters) {
        return restaurants.stream()
            .filter(r -> matchesQuery(r, query))
            .filter(r -> r.isOpen())
            .filter(r -> isWithinDeliveryRadius(r, userLocation))
            .filter(filters::matches)
            .sorted((r1, r2) -> compareRestaurants(r1, r2, filters))
            .collect(Collectors.toList());
    }
    
    private boolean matchesQuery(Restaurant r, String query) {
        return r.getName().toLowerCase().contains(query.toLowerCase()) ||
               r.getCuisineType().contains(query);
    }
    
    private boolean isWithinDeliveryRadius(Restaurant r, Location loc) {
        return r.getLocation().distanceTo(loc) <= r.getDeliveryRadius();
    }
}

public class SearchFilters {
    private Double minRating;
    private List<String> cuisines;
    private PriceRange priceRange;
    private boolean vegetarianOnly;
    
    public boolean matches(Restaurant restaurant) {
        // Apply filters
        return true;
    }
}

// 8. Notification Service
public class OrderTrackingService {
    public void updateOrderStatus(Order order, OrderStatus newStatus) {
        order.setStatus(newStatus);
        notifyCustomer(order);
        
        if (newStatus == OrderStatus.READY_FOR_PICKUP) {
            notifyDeliveryAgent(order);
        }
    }
    
    private void notifyCustomer(Order order) {
        String message = getStatusMessage(order.getStatus());
        NotificationService.send(order.getCustomer(), message);
    }
}
```

## Key Features
1. Real-time order tracking
2. Delivery agent assignment
3. Restaurant search with filters
4. Rating and review system
5. Multiple payment methods
6. Order scheduling

## Key Patterns
- **Strategy Pattern**: Delivery assignment strategies
- **Observer Pattern**: Order status notifications
- **Factory Pattern**: Creating different order types
- **State Pattern**: Order status transitions

---

# Music Streaming Service - Low Level Design

## Core Classes

```java
// 1. Song
public class Song {
    private String songId;
    private String title;
    private Artist artist;
    private Album album;
    private int duration; // in seconds
    private Genre genre;
    private String audioFileUrl;
    private int playCount;
}

// 2. Playlist
public class Playlist {
    private String playlistId;
    private String name;
    private User owner;
    private List<Song> songs;
    private boolean isPublic;
    private LocalDateTime createdAt;
    
    public void addSong(Song song) {
        if (!songs.contains(song)) {
            songs.add(song);
        }
    }
    
    public void removeSong(Song song) {
        songs.remove(song);
    }
    
    public int getTotalDuration() {
        return songs.stream()
            .mapToInt(Song::getDuration)
            .sum();
    }
}

// 3. User
public class User {
    private String userId;
    private String username;
    private SubscriptionType subscription;
    private List<Playlist> playlists;
    private List<Artist> followedArtists;
    private List<Song> likedSongs;
    private Queue<Song> recentlyPlayed;
}

public enum SubscriptionType {
    FREE, PREMIUM, FAMILY
}

// 4. MusicPlayer
public class MusicPlayer {
    private User user;
    private Song currentSong;
    private Playlist currentPlaylist;
    private int currentPosition; // in seconds
    private PlayerState state;
    private PlayMode playMode;
    private double volume;
    
    public void play(Song song) {
        if (user.getSubscription() == SubscriptionType.FREE) {
            playAd();
        }
        
        currentSong = song;
        currentPosition = 0;
        state = PlayerState.PLAYING;
        
        // Stream audio
        streamAudio(song.getAudioFileUrl());
        
        // Update play count
        song.incrementPlayCount();
        user.addToRecentlyPlayed(song);
    }
    
    public void pause() {
        state = PlayerState.PAUSED;
    }
    
    public void resume() {
        if (state == PlayerState.PAUSED) {
            state = PlayerState.PLAYING;
        }
    }
    
    public void next() {
        if (currentPlaylist != null) {
            Song nextSong = getNextSong();
            if (nextSong != null) {
                play(nextSong);
            }
        }
    }
    
    private Song getNextSong() {
        int currentIndex = currentPlaylist.getSongs().indexOf(currentSong);
        
        switch (playMode) {
            case SEQUENTIAL:
                return getSequentialNext(currentIndex);
            case SHUFFLE:
                return getRandomSong();
            case REPEAT_ONE:
                return currentSong;
            case REPEAT_ALL:
                return getSequentialNext(currentIndex, true);
            default:
                return null;
        }
    }
    
    private void playAd() {
        // Play advertisement for free users
    }
}

public enum PlayerState {
    PLAYING, PAUSED, STOPPED
}

public enum PlayMode {
    SEQUENTIAL, SHUFFLE, REPEAT_ONE, REPEAT_ALL
}

// 5. Recommendation Engine
public class RecommendationEngine {
    public List<Song> getRecommendations(User user) {
        List<Song> recommendations = new ArrayList<>();
        
        // Based on liked songs
        recommendations.addAll(getSimilarSongs(user.getLikedSongs()));
        
        // Based on followed artists
        recommendations.addAll(getArtistNewReleases(user.getFollowedArtists()));
        
        // Based on listening history
        recommendations.addAll(getPopularInGenre(user.getTopGenres()));
        
        return recommendations.stream()
            .distinct()
            .limit(50)
            .collect(Collectors.toList());
    }
    
    private List<Song> getSimilarSongs(List<Song> songs) {
        // Use collaborative filtering or content-based filtering
        return new ArrayList<>();
    }
}

// 6. Search Service
public class SearchService {
    private Map<String, Song> songIndex;
    private Map<String, Artist> artistIndex;
    private Map<String, Album> albumIndex;
    
    public SearchResult search(String query) {
        SearchResult result = new SearchResult();
        
        result.setSongs(searchSongs(query));
        result.setArtists(searchArtists(query));
        result.setAlbums(searchAlbums(query));
        result.setPlaylists(searchPlaylists(query));
        
        return result;
    }
    
    private List<Song> searchSongs(String query) {
        return songIndex.values().stream()
            .filter(s -> s.getTitle().toLowerCase().contains(query.toLowerCase()))
            .sorted((s1, s2) -> Integer.compare(s2.getPlayCount(), s1.getPlayCount()))
            .limit(20)
            .collect(Collectors.toList());
    }
}
```

## Key Features
1. Streaming with buffering
2. Playlist management
3. Recommendations
4. Offline downloads (Premium)
5. Social features (following, sharing)
6. Queue management

---

