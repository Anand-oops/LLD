# Rate Limiter - Low Level Design

## Problem Statement
Design a rate limiter to control the rate of requests sent or received by a system.

## Requirements
1. Limit requests per user/API key
2. Different rate limits for different users
3. Handle distributed systems
4. Low latency
5. Memory efficient

## Core Classes

### 1. RateLimiter Interface
```java
public interface RateLimiter {
    boolean allowRequest(String userId);
    void reset(String userId);
}
```

### 2. Token Bucket Algorithm
```java
public class TokenBucketRateLimiter implements RateLimiter {
    private class Bucket {
        private int capacity;
        private int tokens;
        private long lastRefillTime;
        private int refillRate; // tokens per second
        
        public Bucket(int capacity, int refillRate) {
            this.capacity = capacity;
            this.tokens = capacity;
            this.refillRate = refillRate;
            this.lastRefillTime = System.currentTimeMillis();
        }
        
        public synchronized boolean tryConsume() {
            refill();
            if (tokens > 0) {
                tokens--;
                return true;
            }
            return false;
        }
        
        private void refill() {
            long now = System.currentTimeMillis();
            long timePassed = now - lastRefillTime;
            int tokensToAdd = (int) ((timePassed / 1000.0) * refillRate);
            
            if (tokensToAdd > 0) {
                tokens = Math.min(capacity, tokens + tokensToAdd);
                lastRefillTime = now;
            }
        }
    }
    
    private Map<String, Bucket> buckets;
    private int capacity;
    private int refillRate;
    
    public TokenBucketRateLimiter(int capacity, int refillRate) {
        this.buckets = new ConcurrentHashMap<>();
        this.capacity = capacity;
        this.refillRate = refillRate;
    }
    
    @Override
    public boolean allowRequest(String userId) {
        Bucket bucket = buckets.computeIfAbsent(
            userId, 
            k -> new Bucket(capacity, refillRate)
        );
        return bucket.tryConsume();
    }
    
    @Override
    public void reset(String userId) {
        buckets.remove(userId);
    }
}
```

### 3. Sliding Window Log
```java
public class SlidingWindowLogRateLimiter implements RateLimiter {
    private Map<String, Queue<Long>> requestLogs;
    private int maxRequests;
    private long windowSizeMs;
    
    public SlidingWindowLogRateLimiter(int maxRequests, long windowSizeMs) {
        this.requestLogs = new ConcurrentHashMap<>();
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
    }
    
    @Override
    public synchronized boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        Queue<Long> log = requestLogs.computeIfAbsent(
            userId,
            k -> new LinkedList<>()
        );
        
        // Remove old requests
        while (!log.isEmpty() && now - log.peek() > windowSizeMs) {
            log.poll();
        }
        
        if (log.size() < maxRequests) {
            log.offer(now);
            return true;
        }
        
        return false;
    }
    
    @Override
    public void reset(String userId) {
        requestLogs.remove(userId);
    }
}
```

### 4. Sliding Window Counter
```java
public class SlidingWindowCounterRateLimiter implements RateLimiter {
    private class WindowCounter {
        private long currentWindowStart;
        private int currentCount;
        private int previousCount;
        private long windowSizeMs;
        
        public WindowCounter(long windowSizeMs) {
            this.windowSizeMs = windowSizeMs;
            this.currentWindowStart = System.currentTimeMillis();
            this.currentCount = 0;
            this.previousCount = 0;
        }
        
        public synchronized boolean tryAcquire(int maxRequests) {
            long now = System.currentTimeMillis();
            long timeSinceCurrent = now - currentWindowStart;
            
            if (timeSinceCurrent > windowSizeMs) {
                // Move to next window
                previousCount = currentCount;
                currentCount = 0;
                currentWindowStart = now;
            }
            
            // Calculate weighted count
            double previousWeight = 1.0 - (timeSinceCurrent / (double) windowSizeMs);
            double estimatedCount = previousCount * previousWeight + currentCount;
            
            if (estimatedCount < maxRequests) {
                currentCount++;
                return true;
            }
            
            return false;
        }
    }
    
    private Map<String, WindowCounter> counters;
    private int maxRequests;
    private long windowSizeMs;
    
    public SlidingWindowCounterRateLimiter(int maxRequests, long windowSizeMs) {
        this.counters = new ConcurrentHashMap<>();
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
    }
    
    @Override
    public boolean allowRequest(String userId) {
        WindowCounter counter = counters.computeIfAbsent(
            userId,
            k -> new WindowCounter(windowSizeMs)
        );
        return counter.tryAcquire(maxRequests);
    }
    
    @Override
    public void reset(String userId) {
        counters.remove(userId);
    }
}
```

### 5. Rate Limiter Factory
```java
public class RateLimiterFactory {
    public enum Algorithm {
        TOKEN_BUCKET,
        SLIDING_WINDOW_LOG,
        SLIDING_WINDOW_COUNTER,
        FIXED_WINDOW
    }
    
    public static RateLimiter createRateLimiter(
        Algorithm algorithm,
        int maxRequests,
        long windowSizeMs
    ) {
        switch (algorithm) {
            case TOKEN_BUCKET:
                return new TokenBucketRateLimiter(maxRequests, maxRequests);
            case SLIDING_WINDOW_LOG:
                return new SlidingWindowLogRateLimiter(maxRequests, windowSizeMs);
            case SLIDING_WINDOW_COUNTER:
                return new SlidingWindowCounterRateLimiter(maxRequests, windowSizeMs);
            default:
                throw new IllegalArgumentException("Unknown algorithm");
        }
    }
}
```

### 6. Distributed Rate Limiter (Redis-based)
```java
public class DistributedRateLimiter implements RateLimiter {
    private RedisClient redis;
    private int maxRequests;
    private long windowSizeMs;
    
    public DistributedRateLimiter(RedisClient redis, int maxRequests, long windowSizeMs) {
        this.redis = redis;
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
    }
    
    @Override
    public boolean allowRequest(String userId) {
        String key = "rate_limit:" + userId;
        long now = System.currentTimeMillis();
        
        // Lua script for atomicity
        String script = 
            "local key = KEYS[1] " +
            "local now = tonumber(ARGV[1]) " +
            "local window = tonumber(ARGV[2]) " +
            "local limit = tonumber(ARGV[3]) " +
            "redis.call('zremrangebyscore', key, 0, now - window) " +
            "local count = redis.call('zcard', key) " +
            "if count < limit then " +
            "  redis.call('zadd', key, now, now) " +
            "  redis.call('expire', key, window / 1000) " +
            "  return 1 " +
            "end " +
            "return 0";
        
        Object result = redis.eval(
            script,
            Arrays.asList(key),
            Arrays.asList(String.valueOf(now), 
                         String.valueOf(windowSizeMs),
                         String.valueOf(maxRequests))
        );
        
        return "1".equals(result.toString());
    }
    
    @Override
    public void reset(String userId) {
        redis.del("rate_limit:" + userId);
    }
}

// Mock Redis client interface
interface RedisClient {
    Object eval(String script, List<String> keys, List<String> args);
    void del(String key);
}
```

### 7. Rate Limiter with Multiple Rules
```java
public class MultiRuleRateLimiter implements RateLimiter {
    private Map<String, List<RateLimitRule>> userRules;
    private RateLimitRule defaultRule;
    
    public MultiRuleRateLimiter(RateLimitRule defaultRule) {
        this.userRules = new ConcurrentHashMap<>();
        this.defaultRule = defaultRule;
    }
    
    public void addRule(String userId, RateLimitRule rule) {
        userRules.computeIfAbsent(userId, k -> new ArrayList<>()).add(rule);
    }
    
    @Override
    public boolean allowRequest(String userId) {
        List<RateLimitRule> rules = userRules.getOrDefault(
            userId,
            Collections.singletonList(defaultRule)
        );
        
        for (RateLimitRule rule : rules) {
            if (!rule.allowRequest(userId)) {
                return false;
            }
        }
        
        return true;
    }
    
    @Override
    public void reset(String userId) {
        List<RateLimitRule> rules = userRules.get(userId);
        if (rules != null) {
            rules.forEach(r -> r.reset(userId));
        }
    }
}

public class RateLimitRule {
    private RateLimiter rateLimiter;
    private String name;
    
    public RateLimitRule(String name, RateLimiter rateLimiter) {
        this.name = name;
        this.rateLimiter = rateLimiter;
    }
    
    public boolean allowRequest(String userId) {
        return rateLimiter.allowRequest(userId);
    }
    
    public void reset(String userId) {
        rateLimiter.reset(userId);
    }
}
```

## Usage Example
```java
public class RateLimiterDemo {
    public static void main(String[] args) {
        // Token bucket: 10 requests per second
        RateLimiter tokenBucket = new TokenBucketRateLimiter(10, 10);
        
        // Sliding window: 100 requests per minute
        RateLimiter slidingWindow = new SlidingWindowLogRateLimiter(
            100,
            60000
        );
        
        // Multi-rule rate limiter
        MultiRuleRateLimiter multiRule = new MultiRuleRateLimiter(
            new RateLimitRule("default", tokenBucket)
        );
        
        // Add premium user rule
        multiRule.addRule(
            "premium_user",
            new RateLimitRule(
                "premium",
                new TokenBucketRateLimiter(100, 100)
            )
        );
        
        // Test requests
        String userId = "user123";
        for (int i = 0; i < 15; i++) {
            boolean allowed = tokenBucket.allowRequest(userId);
            System.out.println("Request " + i + ": " + 
                (allowed ? "ALLOWED" : "DENIED"));
        }
    }
}
```

## Algorithm Comparison

| Algorithm | Pros | Cons | Memory | Accuracy |
|-----------|------|------|--------|----------|
| Token Bucket | Simple, burst traffic | Memory per user | O(n) | Good |
| Sliding Window Log | Accurate | High memory | O(n*m) | Excellent |
| Sliding Window Counter | Balance | Complex | O(n) | Very Good |
| Fixed Window | Simple | Burst at edges | O(n) | Fair |

## Key Design Patterns Used
1. **Strategy Pattern**: Different rate limiting algorithms
2. **Factory Pattern**: Creating rate limiters
3. **Singleton Pattern**: For shared rate limiter instance
4. **Decorator Pattern**: Adding rules on top of base limiter

## Interview Talking Points

1. **Algorithm Selection**:
   - Token Bucket: Best for most cases
   - Sliding Window: Most accurate
   - Trade-off: Accuracy vs Memory

2. **Distributed Systems**:
   - Use Redis for shared state
   - Lua scripts for atomicity
   - Eventual consistency acceptable

3. **Performance**:
   - In-memory for single server
   - Redis for distributed
   - Cache user limits locally

4. **Edge Cases**:
   - Clock skew in distributed systems
   - Burst traffic handling
   - User tier changes

## Possible Extensions
1. Add tier-based limits (free, premium, enterprise)
2. Add geographic rate limiting
3. Add IP-based rate limiting
4. Add dynamic rate adjustment
5. Add rate limit headers in response
6. Add monitoring and alerting
7. Add graceful degradation
8. Add whitelist/blacklist support
