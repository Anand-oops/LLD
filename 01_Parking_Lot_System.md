# Parking Lot System - Low Level Design

## Problem Statement
Design a parking lot system that can handle multiple floors, different vehicle types, and payment processing.

## UML Class Diagram

```
┌─────────────────────────┐
│     <<abstract>>        │
│       Vehicle           │
├─────────────────────────┤
│ - licenseNumber: String │
│ - type: VehicleType     │
├─────────────────────────┤
│ + getLicenseNumber()    │
│ + getType()             │
└─────────────────────────┘
           △
           │
    ┌──────┴──────┬──────────┐
    │             │          │
┌───────┐    ┌────────┐  ┌───────┐
│  Car  │    │  Bike  │  │ Truck │
└───────┘    └────────┘  └───────┘


┌──────────────────────────┐
│     <<abstract>>         │
│      ParkingSpot         │
├──────────────────────────┤
│ - spotId: String         │
│ - type: SpotType         │
│ - isAvailable: boolean   │
│ - vehicle: Vehicle       │
├──────────────────────────┤
│ + canFitVehicle()        │
│ + assignVehicle()        │
│ + removeVehicle()        │
└──────────────────────────┘
           △
           │
    ┌──────┴────────┬──────────────┐
    │               │              │
┌─────────────┐ ┌─────────┐ ┌────────────────┐
│ CompactSpot │ │LargeSpot│ │HandicappedSpot │
└─────────────┘ └─────────┘ └────────────────┘


┌──────────────────────────────┐
│      ParkingFloor            │
├──────────────────────────────┤
│ - floorNumber: int           │
│ - spots: List<ParkingSpot>   │
│ - spotMap: Map<String,Spot>  │
├──────────────────────────────┤
│ + addSpot()                  │
│ + findAvailableSpot()        │
│ + getAvailableCount()        │
└──────────────────────────────┘
           │
           │ 1..*
           ▼
┌──────────────────────────────┐
│     ParkingSpot              │
└──────────────────────────────┘


┌──────────────────────────────┐
│      ParkingTicket           │
├──────────────────────────────┤
│ - ticketId: String           │
│ - licenseNumber: String      │
│ - spotId: String             │
│ - entryTime: LocalDateTime   │
│ - exitTime: LocalDateTime    │
│ - fee: double                │
│ - status: TicketStatus       │
├──────────────────────────────┤
│ + markExit()                 │
│ + getDurationInMinutes()     │
└──────────────────────────────┘


┌──────────────────────────────────┐
│   <<Singleton>>                  │
│      ParkingLot                  │
├──────────────────────────────────┤
│ - instance: ParkingLot           │
│ - floors: List<ParkingFloor>     │
│ - activeTickets: Map             │
│ - pricingStrategy: Strategy      │
├──────────────────────────────────┤
│ + getInstance()                  │
│ + parkVehicle()                  │
│ + unparkVehicle()                │
│ + displayAvailability()          │
└──────────────────────────────────┘
           │
           │ uses
           ▼
┌──────────────────────────────┐
│   <<interface>>              │
│    PricingStrategy           │
├──────────────────────────────┤
│ + calculateFee()             │
└──────────────────────────────┘
           △
           │
    ┌──────┴──────┐
    │             │
┌─────────────┐ ┌──────────────────┐
│HourlyPricing│ │FlatRatePricing   │
└─────────────┘ └──────────────────┘


<<enumeration>>
VehicleType
─────────────
CAR
BIKE
TRUCK

<<enumeration>>
SpotType
─────────────
COMPACT
LARGE
HANDICAPPED

<<enumeration>>
TicketStatus
─────────────
ACTIVE
PAID
LOST
```

## Requirements
1. Multiple floors with multiple parking spots
2. Different vehicle types (Car, Bike, Truck)
3. Different spot types (Compact, Large, Handicapped)
4. Entry/Exit gates
5. Parking fee calculation based on duration
6. Display available spots

## Core Classes

### 1. Vehicle (Abstract)
```java
public abstract class Vehicle {
    private String licenseNumber;
    private VehicleType type;
    
    public Vehicle(String licenseNumber, VehicleType type) {
        this.licenseNumber = licenseNumber;
        this.type = type;
    }
    
    public String getLicenseNumber() { return licenseNumber; }
    public VehicleType getType() { return type; }
}

public class Car extends Vehicle {
    public Car(String licenseNumber) {
        super(licenseNumber, VehicleType.CAR);
    }
}

public class Bike extends Vehicle {
    public Bike(String licenseNumber) {
        super(licenseNumber, VehicleType.BIKE);
    }
}

public class Truck extends Vehicle {
    public Truck(String licenseNumber) {
        super(licenseNumber, VehicleType.TRUCK);
    }
}

public enum VehicleType {
    CAR, BIKE, TRUCK
}
```

### 2. ParkingSpot (Abstract)
```java
public abstract class ParkingSpot {
    private String spotId;
    private SpotType type;
    private boolean isAvailable;
    private Vehicle vehicle;
    
    public ParkingSpot(String spotId, SpotType type) {
        this.spotId = spotId;
        this.type = type;
        this.isAvailable = true;
    }
    
    public boolean isAvailable() { return isAvailable; }
    
    public boolean canFitVehicle(Vehicle vehicle) {
        // Override in subclasses
        return false;
    }
    
    public void assignVehicle(Vehicle vehicle) {
        this.vehicle = vehicle;
        this.isAvailable = false;
    }
    
    public void removeVehicle() {
        this.vehicle = null;
        this.isAvailable = true;
    }
    
    public String getSpotId() { return spotId; }
    public Vehicle getVehicle() { return vehicle; }
}

public class CompactSpot extends ParkingSpot {
    public CompactSpot(String spotId) {
        super(spotId, SpotType.COMPACT);
    }
    
    @Override
    public boolean canFitVehicle(Vehicle vehicle) {
        return vehicle.getType() == VehicleType.BIKE || 
               vehicle.getType() == VehicleType.CAR;
    }
}

public class LargeSpot extends ParkingSpot {
    public LargeSpot(String spotId) {
        super(spotId, SpotType.LARGE);
    }
    
    @Override
    public boolean canFitVehicle(Vehicle vehicle) {
        return true; // Can fit all vehicles
    }
}

public class HandicappedSpot extends ParkingSpot {
    public HandicappedSpot(String spotId) {
        super(spotId, SpotType.HANDICAPPED);
    }
    
    @Override
    public boolean canFitVehicle(Vehicle vehicle) {
        return vehicle.getType() == VehicleType.CAR;
    }
}

public enum SpotType {
    COMPACT, LARGE, HANDICAPPED
}
```

### 3. ParkingFloor
```java
public class ParkingFloor {
    private int floorNumber;
    private List<ParkingSpot> spots;
    private Map<String, ParkingSpot> spotMap; // spotId -> ParkingSpot
    
    public ParkingFloor(int floorNumber) {
        this.floorNumber = floorNumber;
        this.spots = new ArrayList<>();
        this.spotMap = new HashMap<>();
    }
    
    public void addSpot(ParkingSpot spot) {
        spots.add(spot);
        spotMap.put(spot.getSpotId(), spot);
    }
    
    public ParkingSpot findAvailableSpot(Vehicle vehicle) {
        for (ParkingSpot spot : spots) {
            if (spot.isAvailable() && spot.canFitVehicle(vehicle)) {
                return spot;
            }
        }
        return null;
    }
    
    public int getAvailableCount(SpotType type) {
        return (int) spots.stream()
            .filter(s -> s.isAvailable() && s.getType() == type)
            .count();
    }
    
    public int getFloorNumber() { return floorNumber; }
}
```

### 4. ParkingTicket
```java
public class ParkingTicket {
    private String ticketId;
    private String licenseNumber;
    private String spotId;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private double fee;
    private TicketStatus status;
    
    public ParkingTicket(String licenseNumber, String spotId) {
        this.ticketId = UUID.randomUUID().toString();
        this.licenseNumber = licenseNumber;
        this.spotId = spotId;
        this.entryTime = LocalDateTime.now();
        this.status = TicketStatus.ACTIVE;
    }
    
    public void markExit() {
        this.exitTime = LocalDateTime.now();
        this.status = TicketStatus.PAID;
    }
    
    public long getDurationInMinutes() {
        LocalDateTime end = exitTime != null ? exitTime : LocalDateTime.now();
        return Duration.between(entryTime, end).toMinutes();
    }
    
    // Getters and setters
    public String getTicketId() { return ticketId; }
    public String getSpotId() { return spotId; }
    public void setFee(double fee) { this.fee = fee; }
    public double getFee() { return fee; }
}

public enum TicketStatus {
    ACTIVE, PAID, LOST
}
```

### 5. ParkingLot (Singleton)
```java
public class ParkingLot {
    private static ParkingLot instance;
    private List<ParkingFloor> floors;
    private Map<String, ParkingTicket> activeTickets; // ticketId -> Ticket
    private PricingStrategy pricingStrategy;
    
    private ParkingLot() {
        this.floors = new ArrayList<>();
        this.activeTickets = new HashMap<>();
        this.pricingStrategy = new HourlyPricingStrategy();
    }
    
    public static synchronized ParkingLot getInstance() {
        if (instance == null) {
            instance = new ParkingLot();
        }
        return instance;
    }
    
    public void addFloor(ParkingFloor floor) {
        floors.add(floor);
    }
    
    public ParkingTicket parkVehicle(Vehicle vehicle) {
        for (ParkingFloor floor : floors) {
            ParkingSpot spot = floor.findAvailableSpot(vehicle);
            if (spot != null) {
                spot.assignVehicle(vehicle);
                ParkingTicket ticket = new ParkingTicket(
                    vehicle.getLicenseNumber(), 
                    spot.getSpotId()
                );
                activeTickets.put(ticket.getTicketId(), ticket);
                return ticket;
            }
        }
        return null; // No spot available
    }
    
    public double unparkVehicle(String ticketId) {
        ParkingTicket ticket = activeTickets.get(ticketId);
        if (ticket == null) {
            throw new IllegalArgumentException("Invalid ticket");
        }
        
        // Find and free the spot
        for (ParkingFloor floor : floors) {
            for (ParkingSpot spot : floor.getSpots()) {
                if (spot.getSpotId().equals(ticket.getSpotId())) {
                    spot.removeVehicle();
                    break;
                }
            }
        }
        
        // Calculate fee
        double fee = pricingStrategy.calculateFee(ticket);
        ticket.setFee(fee);
        ticket.markExit();
        
        activeTickets.remove(ticketId);
        return fee;
    }
    
    public void displayAvailability() {
        for (ParkingFloor floor : floors) {
            System.out.println("Floor " + floor.getFloorNumber() + ":");
            System.out.println("  Compact: " + 
                floor.getAvailableCount(SpotType.COMPACT));
            System.out.println("  Large: " + 
                floor.getAvailableCount(SpotType.LARGE));
            System.out.println("  Handicapped: " + 
                floor.getAvailableCount(SpotType.HANDICAPPED));
        }
    }
}
```

### 6. PricingStrategy (Strategy Pattern)
```java
public interface PricingStrategy {
    double calculateFee(ParkingTicket ticket);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private static final double RATE_PER_HOUR = 10.0;
    
    @Override
    public double calculateFee(ParkingTicket ticket) {
        long minutes = ticket.getDurationInMinutes();
        long hours = (minutes + 59) / 60; // Round up
        return hours * RATE_PER_HOUR;
    }
}

public class FlatRatePricingStrategy implements PricingStrategy {
    private static final double FLAT_RATE = 50.0;
    
    @Override
    public double calculateFee(ParkingTicket ticket) {
        return FLAT_RATE;
    }
}
```

## Key Design Patterns Used
1. **Singleton Pattern**: ParkingLot class
2. **Factory Pattern**: Can be used for creating vehicles and spots
3. **Strategy Pattern**: PricingStrategy for flexible pricing
4. **Abstract Class**: Vehicle and ParkingSpot for extensibility

## Interview Talking Points
1. **Scalability**: Each floor manages its own spots independently
2. **Extensibility**: Easy to add new vehicle types or spot types
3. **Thread Safety**: Singleton getInstance() is synchronized
4. **SOLID Principles**:
   - Single Responsibility: Each class has one job
   - Open/Closed: Can extend vehicle types without modifying code
   - Liskov Substitution: All spot types can substitute ParkingSpot
   - Interface Segregation: Clean interfaces
   - Dependency Inversion: Depends on abstractions (PricingStrategy)

## Possible Extensions
1. Add payment gateway integration
2. Add reservation system
3. Add multiple entry/exit gates with their own queues
4. Add admin dashboard for monitoring
5. Add electric vehicle charging spots
