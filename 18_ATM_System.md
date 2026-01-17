# ATM System - Low Level Design

## Core Classes

```java
// 1. Card
public class Card {
    private String cardNumber;
    private String pin;
    private String accountNumber;
    private CardType type;
    
    public boolean validatePin(String inputPin) {
        return this.pin.equals(inputPin);
    }
}

public enum CardType { DEBIT, CREDIT }

// 2. Account
public class Account {
    private String accountNumber;
    private double balance;
    private AccountType type;
    
    public synchronized boolean withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }
    
    public synchronized void deposit(double amount) {
        balance += amount;
    }
}

public enum AccountType { SAVINGS, CHECKING }

// 3. ATM
public class ATM {
    private String atmId;
    private String location;
    private ATMState state;
    private Card currentCard;
    private Map<Denomination, Integer> cashInventory;
    
    public void insertCard(Card card) {
        if (state != ATMState.IDLE) {
            throw new IllegalStateException("ATM busy");
        }
        this.currentCard = card;
        state = ATMState.CARD_INSERTED;
    }
    
    public boolean authenticatePin(String pin) {
        if (currentCard.validatePin(pin)) {
            state = ATMState.AUTHENTICATED;
            return true;
        }
        return false;
    }
    
    public boolean withdrawCash(double amount) {
        if (state != ATMState.AUTHENTICATED) return false;
        
        Map<Denomination, Integer> notes = calculateCashDispense(amount);
        if (notes == null) return false;
        
        // Dispense cash
        dispenseCash(notes);
        return true;
    }
    
    private Map<Denomination, Integer> calculateCashDispense(double amount) {
        Map<Denomination, Integer> result = new HashMap<>();
        double remaining = amount;
        
        for (Denomination denom : Denomination.values()) {
            int count = Math.min(
                (int)(remaining / denom.getValue()),
                cashInventory.get(denom)
            );
            if (count > 0) {
                result.put(denom, count);
                remaining -= count * denom.getValue();
            }
        }
        
        return remaining == 0 ? result : null;
    }
    
    private void dispenseCash(Map<Denomination, Integer> notes) {
        notes.forEach((denom, count) -> {
            cashInventory.put(denom, cashInventory.get(denom) - count);
        });
    }
    
    public void ejectCard() {
        currentCard = null;
        state = ATMState.IDLE;
    }
}

public enum ATMState { IDLE, CARD_INSERTED, AUTHENTICATED, PROCESSING }
public enum Denomination { HUNDRED(100), FIFTY(50), TWENTY(20), TEN(10);
    private double value;
    Denomination(double value) { this.value = value; }
    public double getValue() { return value; }
}

// 4. Transaction
public class Transaction {
    private String transactionId;
    private TransactionType type;
    private double amount;
    private LocalDateTime timestamp;
    private TransactionStatus status;
    
    public Transaction(TransactionType type, double amount) {
        this.transactionId = UUID.randomUUID().toString();
        this.type = type;
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
        this.status = TransactionStatus.PENDING;
    }
}

public enum TransactionType { WITHDRAWAL, DEPOSIT, BALANCE_INQUIRY, PIN_CHANGE }
public enum TransactionStatus { PENDING, SUCCESS, FAILED, CANCELLED }

// 5. Bank System
public class BankSystem {
    private static BankSystem instance;
    private Map<String, Account> accounts;
    private Map<String, Card> cards;
    
    public static BankSystem getInstance() {
        if (instance == null) {
            instance = new BankSystem();
        }
        return instance;
    }
    
    public Account getAccount(String cardNumber) {
        Card card = cards.get(cardNumber);
        return card != null ? accounts.get(card.getAccountNumber()) : null;
    }
    
    public boolean processTransaction(Transaction transaction, Account account) {
        switch (transaction.getType()) {
            case WITHDRAWAL:
                return account.withdraw(transaction.getAmount());
            case DEPOSIT:
                account.deposit(transaction.getAmount());
                return true;
            default:
                return false;
        }
    }
}
```

## Key Design Patterns
- **Singleton**: BankSystem
- **State Pattern**: ATMState
- **Strategy Pattern**: Cash dispense algorithms
- **Command Pattern**: Transaction processing

---

