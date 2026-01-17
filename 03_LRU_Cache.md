# LRU Cache - Low Level Design

## Core Classes

```java
// 1. LRU Cache Implementation
public class LRUCache<K, V> {
    private class Node {
        K key;
        V value;
        Node prev, next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private final int capacity;
    private final Map<K, Node> cache;
    private final Node head, tail;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        this.head = new Node(null, null);
        this.tail = new Node(null, null);
        head.next = tail;
        tail.prev = head;
    }
    
    public V get(K key) {
        Node node = cache.get(key);
        if (node == null) return null;
        
        moveToHead(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        Node node = cache.get(key);
        
        if (node != null) {
            node.value = value;
            moveToHead(node);
        } else {
            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            addToHead(newNode);
            
            if (cache.size() > capacity) {
                Node lru = removeTail();
                cache.remove(lru.key);
            }
        }
    }
    
    private void addToHead(Node node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }
    
    private Node removeTail() {
        Node lru = tail.prev;
        removeNode(lru);
        return lru;
    }
}
```

## Time Complexity
- Get: O(1)
- Put: O(1)
- Space: O(capacity)

---

# HashMap Implementation - Low Level Design

## Core Classes

```java
public class MyHashMap<K, V> {
    private class Entry<K, V> {
        K key;
        V value;
        Entry<K, V> next;
        
        Entry(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private static final int INITIAL_CAPACITY = 16;
    private static final float LOAD_FACTOR = 0.75f;
    
    private Entry<K, V>[] buckets;
    private int size;
    private int capacity;
    
    @SuppressWarnings("unchecked")
    public MyHashMap() {
        this.capacity = INITIAL_CAPACITY;
        this.buckets = new Entry[capacity];
        this.size = 0;
    }
    
    public void put(K key, V value) {
        if (key == null) throw new IllegalArgumentException("Key cannot be null");
        
        if (size >= capacity * LOAD_FACTOR) {
            resize();
        }
        
        int index = getIndex(key);
        Entry<K, V> entry = buckets[index];
        
        // Update if key exists
        while (entry != null) {
            if (entry.key.equals(key)) {
                entry.value = value;
                return;
            }
            entry = entry.next;
        }
        
        // Add new entry
        Entry<K, V> newEntry = new Entry<>(key, value);
        newEntry.next = buckets[index];
        buckets[index] = newEntry;
        size++;
    }
    
    public V get(K key) {
        if (key == null) return null;
        
        int index = getIndex(key);
        Entry<K, V> entry = buckets[index];
        
        while (entry != null) {
            if (entry.key.equals(key)) {
                return entry.value;
            }
            entry = entry.next;
        }
        
        return null;
    }
    
    public void remove(K key) {
        if (key == null) return;
        
        int index = getIndex(key);
        Entry<K, V> entry = buckets[index];
        Entry<K, V> prev = null;
        
        while (entry != null) {
            if (entry.key.equals(key)) {
                if (prev == null) {
                    buckets[index] = entry.next;
                } else {
                    prev.next = entry.next;
                }
                size--;
                return;
            }
            prev = entry;
            entry = entry.next;
        }
    }
    
    private int getIndex(K key) {
        return Math.abs(key.hashCode() % capacity);
    }
    
    @SuppressWarnings("unchecked")
    private void resize() {
        capacity *= 2;
        Entry<K, V>[] oldBuckets = buckets;
        buckets = new Entry[capacity];
        size = 0;
        
        for (Entry<K, V> entry : oldBuckets) {
            while (entry != null) {
                put(entry.key, entry.value);
                entry = entry.next;
            }
        }
    }
    
    public int size() { return size; }
    public boolean isEmpty() { return size == 0; }
}
```

---

# Trie Implementation - Low Level Design

## Core Classes

```java
public class Trie {
    private class TrieNode {
        private Map<Character, TrieNode> children;
        private boolean isEndOfWord;
        private int wordCount; // For prefix count
        
        TrieNode() {
            children = new HashMap<>();
            isEndOfWord = false;
            wordCount = 0;
        }
    }
    
    private TrieNode root;
    
    public Trie() {
        root = new TrieNode();
    }
    
    public void insert(String word) {
        TrieNode current = root;
        
        for (char ch : word.toCharArray()) {
            current = current.children.computeIfAbsent(ch, c -> new TrieNode());
            current.wordCount++;
        }
        
        current.isEndOfWord = true;
    }
    
    public boolean search(String word) {
        TrieNode node = searchNode(word);
        return node != null && node.isEndOfWord;
    }
    
    public boolean startsWith(String prefix) {
        return searchNode(prefix) != null;
    }
    
    private TrieNode searchNode(String str) {
        TrieNode current = root;
        
        for (char ch : str.toCharArray()) {
            current = current.children.get(ch);
            if (current == null) {
                return null;
            }
        }
        
        return current;
    }
    
    public void delete(String word) {
        delete(root, word, 0);
    }
    
    private boolean delete(TrieNode current, String word, int index) {
        if (index == word.length()) {
            if (!current.isEndOfWord) {
                return false;
            }
            current.isEndOfWord = false;
            return current.children.isEmpty();
        }
        
        char ch = word.charAt(index);
        TrieNode node = current.children.get(ch);
        if (node == null) {
            return false;
        }
        
        boolean shouldDeleteCurrentNode = delete(node, word, index + 1);
        
        if (shouldDeleteCurrentNode) {
            current.children.remove(ch);
            return current.children.isEmpty() && !current.isEndOfWord;
        }
        
        return false;
    }
    
    public List<String> autoComplete(String prefix) {
        List<String> results = new ArrayList<>();
        TrieNode node = searchNode(prefix);
        
        if (node != null) {
            dfs(node, prefix, results);
        }
        
        return results;
    }
    
    private void dfs(TrieNode node, String prefix, List<String> results) {
        if (node.isEndOfWord) {
            results.add(prefix);
        }
        
        for (Map.Entry<Character, TrieNode> entry : node.children.entrySet()) {
            dfs(entry.getValue(), prefix + entry.getKey(), results);
        }
    }
    
    public int countWordsWithPrefix(String prefix) {
        TrieNode node = searchNode(prefix);
        return node != null ? node.wordCount : 0;
    }
}

// Usage for autocomplete
public class AutocompleteSystem {
    private Trie trie;
    
    public AutocompleteSystem(String[] words) {
        trie = new Trie();
        for (String word : words) {
            trie.insert(word);
        }
    }
    
    public List<String> getSuggestions(String prefix) {
        return trie.autoComplete(prefix);
    }
}
```

---

# Snake & Ladder - Low Level Design

## Core Classes

```java
// 1. Board
public class Board {
    private int size;
    private Map<Integer, Integer> snakes;   // head -> tail
    private Map<Integer, Integer> ladders;  // bottom -> top
    
    public Board(int size) {
        this.size = size;
        this.snakes = new HashMap<>();
        this.ladders = new HashMap<>();
    }
    
    public void addSnake(int head, int tail) {
        if (head <= tail || head > size || tail < 1) {
            throw new IllegalArgumentException("Invalid snake");
        }
        snakes.put(head, tail);
    }
    
    public void addLadder(int bottom, int top) {
        if (bottom >= top || bottom < 1 || top > size) {
            throw new IllegalArgumentException("Invalid ladder");
        }
        ladders.put(bottom, top);
    }
    
    public int getNewPosition(int position) {
        if (snakes.containsKey(position)) {
            System.out.println("Snake! Down from " + position + " to " + snakes.get(position));
            return snakes.get(position);
        }
        
        if (ladders.containsKey(position)) {
            System.out.println("Ladder! Up from " + position + " to " + ladders.get(position));
            return ladders.get(position);
        }
        
        return position;
    }
    
    public int getSize() { return size; }
}

// 2. Dice
public class Dice {
    private int faces;
    private Random random;
    
    public Dice(int faces) {
        this.faces = faces;
        this.random = new Random();
    }
    
    public int roll() {
        return random.nextInt(faces) + 1;
    }
}

// 3. Player
public class Player {
    private String name;
    private int position;
    
    public Player(String name) {
        this.name = name;
        this.position = 0;
    }
    
    public void move(int steps) {
        position += steps;
    }
    
    public String getName() { return name; }
    public int getPosition() { return position; }
    public void setPosition(int position) { this.position = position; }
}

// 4. Game
public class SnakeAndLadderGame {
    private Board board;
    private Dice dice;
    private Queue<Player> players;
    private Player winner;
    
    public SnakeAndLadderGame(int boardSize, int diceFaces) {
        this.board = new Board(boardSize);
        this.dice = new Dice(diceFaces);
        this.players = new LinkedList<>();
    }
    
    public void addPlayer(Player player) {
        players.offer(player);
    }
    
    public void setupSnakesAndLadders() {
        // Add snakes
        board.addSnake(99, 5);
        board.addSnake(95, 13);
        board.addSnake(88, 24);
        
        // Add ladders
        board.addLadder(4, 25);
        board.addLadder(13, 46);
        board.addLadder(33, 49);
    }
    
    public void start() {
        setupSnakesAndLadders();
        
        while (winner == null) {
            Player currentPlayer = players.poll();
            int diceValue = dice.roll();
            System.out.println(currentPlayer.getName() + " rolled " + diceValue);
            
            int newPosition = currentPlayer.getPosition() + diceValue;
            
            if (newPosition > board.getSize()) {
                System.out.println("Can't move! Exceeds board size.");
                players.offer(currentPlayer);
                continue;
            }
            
            if (newPosition == board.getSize()) {
                winner = currentPlayer;
                System.out.println(currentPlayer.getName() + " wins!");
                break;
            }
            
            currentPlayer.setPosition(board.getNewPosition(newPosition));
            System.out.println(currentPlayer.getName() + " at position " + 
                             currentPlayer.getPosition());
            
            players.offer(currentPlayer);
        }
    }
}
```

## Usage Example
```java
public class SnakeAndLadderDemo {
    public static void main(String[] args) {
        SnakeAndLadderGame game = new SnakeAndLadderGame(100, 6);
        
        game.addPlayer(new Player("Alice"));
        game.addPlayer(new Player("Bob"));
        game.addPlayer(new Player("Charlie"));
        
        game.start();
    }
}
```

---

