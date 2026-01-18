# URL Shortener Service - Low Level Design

## Problem Statement
Design a URL shortening service like bit.ly or tinyurl that converts long URLs into short, shareable links.

## UML Class Diagram

```
┌────────────────────────────────────┐
│           URL                      │
├────────────────────────────────────┤
│ - id: String                       │
│ - shortUrl: String                 │
│ - longUrl: String                  │
│ - userId: String                   │
│ - createdAt: LocalDateTime         │
│ - expiresAt: LocalDateTime         │
│ - clickCount: int                  │
│ - status: URLStatus                │
├────────────────────────────────────┤
│ + isExpired(): boolean             │
│ + incrementClickCount()            │
│ + deactivate()                     │
└────────────────────────────────────┘


┌────────────────────────────────────┐
│       URLShortener                 │
├────────────────────────────────────┤
│ - urlDatabase: Map<String,URL>     │
│ - reverseMap: Map<String,String>   │
│ - encodingStrategy: Strategy       │
│ - counter: Counter                 │
├────────────────────────────────────┤
│ + shortenURL(longUrl): String      │
│ + expandURL(shortUrl): String      │
│ + getAnalytics(): URLAnalytics     │
│ + deleteURL(): boolean             │
│ - generateShortCode(): String      │
└────────────────────────────────────┘
           │
           │ uses
           ▼
┌────────────────────────────────────┐
│   <<interface>>                    │
│    EncodingStrategy                │
├────────────────────────────────────┤
│ + encode(id: long): String         │
│ + decode(shortCode: String): long  │
└────────────────────────────────────┘
           △
           │
    ┌──────┴──────┐
    │             │
┌─────────────┐ ┌──────────────┐
│Base62       │ │ MD5          │
│Encoding     │ │ Encoding     │
└─────────────┘ └──────────────┘


┌────────────────────────────────────┐
│          Counter                   │
├────────────────────────────────────┤
│ - counter: AtomicLong              │
├────────────────────────────────────┤
│ + getNextId(): long                │
└────────────────────────────────────┘


┌────────────────────────────────────┐
│  DistributedIdGenerator            │
├────────────────────────────────────┤
│ - workerId: long                   │
│ - sequence: long                   │
│ - lastTimestamp: long              │
├────────────────────────────────────┤
│ + nextId(): long                   │
│ - waitNextMillis(): long           │
└────────────────────────────────────┘


┌────────────────────────────────────┐
│   <<Singleton>>                    │
│    AnalyticsService                │
├────────────────────────────────────┤
│ - instance: AnalyticsService       │
│ - clickHistory: Map                │
├────────────────────────────────────┤
│ + getInstance()                    │
│ + recordClick(url: URL)            │
│ + getAnalytics(): URLAnalytics     │
└────────────────────────────────────┘
           │
           │ creates
           ▼
┌────────────────────────────────────┐
│       ClickEvent                   │
├────────────────────────────────────┤
│ - urlId: String                    │
│ - timestamp: LocalDateTime         │
│ - clientInfo: ClientInfo           │
└────────────────────────────────────┘
           │
           │ contains
           ▼
┌────────────────────────────────────┐
│       ClientInfo                   │
├────────────────────────────────────┤
│ - ipAddress: String                │
│ - userAgent: String                │
│ - location: String                 │
└────────────────────────────────────┘


┌────────────────────────────────────┐
│       URLAnalytics                 │
├────────────────────────────────────┤
│ - urlId: String                    │
│ - totalClicks: int                 │
│ - createdAt: LocalDateTime         │
│ - clicksByCountry: Map             │
│ - clicksByDate: Map                │
├────────────────────────────────────┤
│ + displayAnalytics()               │
└────────────────────────────────────┘


┌────────────────────────────────────┐
│         URLCache                   │
├────────────────────────────────────┤
│ - cache: Cache<String,String>      │
│ - MAX_SIZE: int                    │
│ - TTL_MINUTES: int                 │
├────────────────────────────────────┤
│ + get(shortCode): String           │
│ + put(shortCode, longUrl)          │
│ + invalidate(shortCode)            │
└────────────────────────────────────┘


<<enumeration>>
URLStatus
─────────────
ACTIVE
INACTIVE
EXPIRED
```

## Requirements
1. Generate unique short URLs
2. Redirect short URL to original URL
3. Track analytics (click count, timestamps)
4. Custom aliases (optional)
5. Expiration time for URLs
6. High availability and low latency

## Core Classes

### 1. URL
```java
public class URL {
    private String id;
    private String shortUrl;
    private String longUrl;
    private String userId;
    private LocalDateTime createdAt;
    private LocalDateTime expiresAt;
    private int clickCount;
    private URLStatus status;
    
    public URL(String longUrl, String userId) {
        this.id = UUID.randomUUID().toString();
        this.longUrl = longUrl;
        this.userId = userId;
        this.createdAt = LocalDateTime.now();
        this.expiresAt = createdAt.plusYears(1); // Default 1 year
        this.clickCount = 0;
        this.status = URLStatus.ACTIVE;
    }
    
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expiresAt);
    }
    
    public void incrementClickCount() {
        this.clickCount++;
    }
    
    public void deactivate() {
        this.status = URLStatus.INACTIVE;
    }
    
    // Getters and setters
    public String getShortUrl() { return shortUrl; }
    public void setShortUrl(String shortUrl) { this.shortUrl = shortUrl; }
    public String getLongUrl() { return longUrl; }
    public int getClickCount() { return clickCount; }
    public String getId() { return id; }
}

public enum URLStatus {
    ACTIVE, INACTIVE, EXPIRED
}
```

### 2. URLShortener (Core Service)
```java
public class URLShortener {
    private static final String BASE_URL = "https://short.url/";
    private static final int SHORT_URL_LENGTH = 7;
    
    private Map<String, URL> urlDatabase;          // shortCode -> URL
    private Map<String, String> reverseMap;        // longUrl -> shortCode
    private EncodingStrategy encodingStrategy;
    private Counter counter;
    
    public URLShortener() {
        this.urlDatabase = new ConcurrentHashMap<>();
        this.reverseMap = new ConcurrentHashMap<>();
        this.encodingStrategy = new Base62Encoding();
        this.counter = new Counter();
    }
    
    public String shortenURL(String longUrl, String userId) {
        return shortenURL(longUrl, userId, null);
    }
    
    public String shortenURL(String longUrl, String userId, String customAlias) {
        // Validate URL
        if (!isValidURL(longUrl)) {
            throw new IllegalArgumentException("Invalid URL");
        }
        
        // Check if already exists
        if (reverseMap.containsKey(longUrl)) {
            String shortCode = reverseMap.get(longUrl);
            return BASE_URL + shortCode;
        }
        
        // Generate short code
        String shortCode;
        if (customAlias != null && !customAlias.isEmpty()) {
            if (urlDatabase.containsKey(customAlias)) {
                throw new IllegalArgumentException("Alias already exists");
            }
            shortCode = customAlias;
        } else {
            shortCode = generateShortCode();
        }
        
        // Create URL object
        URL url = new URL(longUrl, userId);
        url.setShortUrl(BASE_URL + shortCode);
        
        // Store in database
        urlDatabase.put(shortCode, url);
        reverseMap.put(longUrl, shortCode);
        
        return url.getShortUrl();
    }
    
    public String expandURL(String shortUrl) {
        String shortCode = extractShortCode(shortUrl);
        URL url = urlDatabase.get(shortCode);
        
        if (url == null) {
            throw new IllegalArgumentException("URL not found");
        }
        
        if (url.isExpired()) {
            url.deactivate();
            throw new IllegalArgumentException("URL has expired");
        }
        
        // Track analytics
        url.incrementClickCount();
        trackClick(url);
        
        return url.getLongUrl();
    }
    
    public URLAnalytics getAnalytics(String shortUrl) {
        String shortCode = extractShortCode(shortUrl);
        URL url = urlDatabase.get(shortCode);
        
        if (url == null) {
            throw new IllegalArgumentException("URL not found");
        }
        
        return new URLAnalytics(url);
    }
    
    public boolean deleteURL(String shortUrl, String userId) {
        String shortCode = extractShortCode(shortUrl);
        URL url = urlDatabase.get(shortCode);
        
        if (url == null || !url.getUserId().equals(userId)) {
            return false;
        }
        
        urlDatabase.remove(shortCode);
        reverseMap.remove(url.getLongUrl());
        return true;
    }
    
    private String generateShortCode() {
        long id = counter.getNextId();
        return encodingStrategy.encode(id);
    }
    
    private String extractShortCode(String shortUrl) {
        return shortUrl.replace(BASE_URL, "");
    }
    
    private boolean isValidURL(String url) {
        try {
            new java.net.URL(url);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    
    private void trackClick(URL url) {
        // Store click event for analytics
        AnalyticsService.getInstance().recordClick(url);
    }
}
```

### 3. EncodingStrategy (Strategy Pattern)
```java
public interface EncodingStrategy {
    String encode(long id);
    long decode(String shortCode);
}

public class Base62Encoding implements EncodingStrategy {
    private static final String BASE62 = 
        "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private static final int BASE = 62;
    
    @Override
    public String encode(long id) {
        if (id == 0) return String.valueOf(BASE62.charAt(0));
        
        StringBuilder sb = new StringBuilder();
        while (id > 0) {
            sb.append(BASE62.charAt((int)(id % BASE)));
            id /= BASE;
        }
        return sb.reverse().toString();
    }
    
    @Override
    public long decode(String shortCode) {
        long id = 0;
        for (char c : shortCode.toCharArray()) {
            id = id * BASE + BASE62.indexOf(c);
        }
        return id;
    }
}

public class MD5Encoding implements EncodingStrategy {
    @Override
    public String encode(long id) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] hash = md.digest(String.valueOf(id).getBytes());
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < 4; i++) {
                sb.append(Integer.toHexString(hash[i] & 0xFF));
            }
            return sb.toString().substring(0, 7);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
    
    @Override
    public long decode(String shortCode) {
        // MD5 is one-way, can't decode
        throw new UnsupportedOperationException();
    }
}
```

### 4. Counter (For generating unique IDs)
```java
public class Counter {
    private AtomicLong counter;
    
    public Counter() {
        this.counter = new AtomicLong(1000000); // Start from a large number
    }
    
    public long getNextId() {
        return counter.incrementAndGet();
    }
}

// Alternative: Distributed ID Generator (Snowflake-like)
public class DistributedIdGenerator {
    private static final long EPOCH = 1640995200000L; // Custom epoch
    private static final long WORKER_ID_BITS = 10L;
    private static final long SEQUENCE_BITS = 12L;
    
    private final long workerId;
    private long sequence = 0L;
    private long lastTimestamp = -1L;
    
    public DistributedIdGenerator(long workerId) {
        this.workerId = workerId;
    }
    
    public synchronized long nextId() {
        long timestamp = System.currentTimeMillis();
        
        if (timestamp < lastTimestamp) {
            throw new RuntimeException("Clock moved backwards");
        }
        
        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & ((1 << SEQUENCE_BITS) - 1);
            if (sequence == 0) {
                timestamp = waitNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0;
        }
        
        lastTimestamp = timestamp;
        
        return ((timestamp - EPOCH) << (WORKER_ID_BITS + SEQUENCE_BITS)) |
               (workerId << SEQUENCE_BITS) |
               sequence;
    }
    
    private long waitNextMillis(long lastTimestamp) {
        long timestamp = System.currentTimeMillis();
        while (timestamp <= lastTimestamp) {
            timestamp = System.currentTimeMillis();
        }
        return timestamp;
    }
}
```

### 5. Analytics Service
```java
public class AnalyticsService {
    private static AnalyticsService instance;
    private Map<String, List<ClickEvent>> clickHistory;
    
    private AnalyticsService() {
        this.clickHistory = new ConcurrentHashMap<>();
    }
    
    public static synchronized AnalyticsService getInstance() {
        if (instance == null) {
            instance = new AnalyticsService();
        }
        return instance;
    }
    
    public void recordClick(URL url) {
        ClickEvent event = new ClickEvent(
            url.getId(),
            LocalDateTime.now(),
            getClientInfo()
        );
        
        clickHistory.computeIfAbsent(url.getId(), k -> new ArrayList<>())
                   .add(event);
    }
    
    public URLAnalytics getAnalytics(String urlId) {
        List<ClickEvent> events = clickHistory.getOrDefault(urlId, new ArrayList<>());
        return new URLAnalytics(events);
    }
    
    private ClientInfo getClientInfo() {
        // In real system, extract from HTTP request
        return new ClientInfo("Unknown", "Unknown", "Unknown");
    }
}

public class ClickEvent {
    private String urlId;
    private LocalDateTime timestamp;
    private ClientInfo clientInfo;
    
    public ClickEvent(String urlId, LocalDateTime timestamp, ClientInfo clientInfo) {
        this.urlId = urlId;
        this.timestamp = timestamp;
        this.clientInfo = clientInfo;
    }
    
    // Getters
    public LocalDateTime getTimestamp() { return timestamp; }
    public ClientInfo getClientInfo() { return clientInfo; }
}

public class ClientInfo {
    private String ipAddress;
    private String userAgent;
    private String location;
    
    public ClientInfo(String ipAddress, String userAgent, String location) {
        this.ipAddress = ipAddress;
        this.userAgent = userAgent;
        this.location = location;
    }
    
    // Getters
}
```

### 6. URL Analytics
```java
public class URLAnalytics {
    private String urlId;
    private int totalClicks;
    private LocalDateTime createdAt;
    private Map<String, Integer> clicksByCountry;
    private Map<LocalDate, Integer> clicksByDate;
    
    public URLAnalytics(URL url) {
        this.urlId = url.getId();
        this.totalClicks = url.getClickCount();
        this.createdAt = url.getCreatedAt();
        this.clicksByCountry = new HashMap<>();
        this.clicksByDate = new HashMap<>();
    }
    
    public URLAnalytics(List<ClickEvent> events) {
        this.totalClicks = events.size();
        this.clicksByCountry = new HashMap<>();
        this.clicksByDate = new HashMap<>();
        
        for (ClickEvent event : events) {
            // Aggregate by date
            LocalDate date = event.getTimestamp().toLocalDate();
            clicksByDate.merge(date, 1, Integer::sum);
            
            // Aggregate by country
            String country = event.getClientInfo().getLocation();
            clicksByCountry.merge(country, 1, Integer::sum);
        }
    }
    
    public void displayAnalytics() {
        System.out.println("Total Clicks: " + totalClicks);
        System.out.println("Clicks by Country: " + clicksByCountry);
        System.out.println("Clicks by Date: " + clicksByDate);
    }
    
    // Getters
}
```

### 7. Cache Layer
```java
public class URLCache {
    private final Cache<String, String> cache; // shortCode -> longUrl
    private final int MAX_SIZE = 10000;
    private final int TTL_MINUTES = 60;
    
    public URLCache() {
        this.cache = CacheBuilder.newBuilder()
            .maximumSize(MAX_SIZE)
            .expireAfterWrite(TTL_MINUTES, TimeUnit.MINUTES)
            .build();
    }
    
    public String get(String shortCode) {
        return cache.getIfPresent(shortCode);
    }
    
    public void put(String shortCode, String longUrl) {
        cache.put(shortCode, longUrl);
    }
    
    public void invalidate(String shortCode) {
        cache.invalidate(shortCode);
    }
}
```

## Usage Example
```java
public class URLShortenerDemo {
    public static void main(String[] args) {
        URLShortener shortener = new URLShortener();
        
        // Shorten URL
        String longUrl = "https://www.example.com/very/long/url/with/parameters?id=123";
        String shortUrl = shortener.shortenURL(longUrl, "user123");
        System.out.println("Short URL: " + shortUrl);
        
        // Custom alias
        String customUrl = shortener.shortenURL(
            "https://www.example.com/another",
            "user123",
            "custom"
        );
        System.out.println("Custom URL: " + customUrl);
        
        // Expand URL
        String original = shortener.expandURL(shortUrl);
        System.out.println("Original URL: " + original);
        
        // Get analytics
        URLAnalytics analytics = shortener.getAnalytics(shortUrl);
        analytics.displayAnalytics();
    }
}
```

## Key Design Patterns Used
1. **Strategy Pattern**: Different encoding strategies (Base62, MD5)
2. **Singleton Pattern**: URLShortener, AnalyticsService
3. **Factory Pattern**: Can be used for creating different URL types
4. **Observer Pattern**: For real-time analytics updates

## Interview Talking Points

### 1. Collision Handling
- **Base62 with counter**: No collisions, sequential
- **MD5 hashing**: Check for collisions, regenerate if needed
- **Trade-off**: Predictability vs randomness

### 2. Scalability
- **Distributed ID generation**: Snowflake algorithm
- **Database sharding**: Shard by hash of short code
- **Caching**: Redis for hot URLs (80-20 rule)
- **Load balancing**: Multiple app servers

### 3. Analytics
- **Asynchronous processing**: Queue-based (Kafka)
- **Time-series database**: For click tracking
- **Aggregation**: Pre-compute daily/weekly stats

### 4. Security
- **Rate limiting**: Prevent abuse
- **URL validation**: Prevent malicious URLs
- **Access control**: User-based permissions

### 5. Performance
- **Read-heavy system**: Focus on read optimization
- **Cache hit ratio**: 90%+ for popular URLs
- **Database indexing**: On short_code field
- **CDN**: Serve redirects from edge locations

## System Design Considerations

### Database Schema
```sql
CREATE TABLE urls (
    id VARCHAR(36) PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    user_id VARCHAR(36),
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    click_count INT DEFAULT 0,
    status VARCHAR(20),
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);

CREATE TABLE clicks (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    url_id VARCHAR(36),
    timestamp TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    country VARCHAR(2),
    FOREIGN KEY (url_id) REFERENCES urls(id),
    INDEX idx_url_timestamp (url_id, timestamp)
);
```

### Capacity Estimation
- **1 billion URLs**: 7 characters @ 62^7 = 3.5 trillion combinations
- **Storage per URL**: ~500 bytes
- **Total storage**: 500 GB for 1B URLs
- **QPS**: 1000 reads/sec, 10 writes/sec (read-heavy)

## Possible Extensions
1. Add QR code generation
2. Add link preview generation
3. Add password protection for URLs
4. Add link expiry notifications
5. Add A/B testing support
6. Add branded domains
7. Add API rate limiting per user
8. Add bulk URL shortening
9. Add link bundling (multiple links in one)
10. Add deep linking for mobile apps
