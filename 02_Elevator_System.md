# Elevator System - Low Level Design

## Problem Statement
Design an elevator control system for a building with multiple elevators and floors.

## UML Class Diagram

```
┌──────────────────────────────────┐
│         Elevator                 │
├──────────────────────────────────┤
│ - id: int                        │
│ - currentFloor: int              │
│ - direction: Direction           │
│ - state: ElevatorState           │
│ - capacity: int                  │
│ - currentLoad: int               │
│ - upStops: TreeSet<Integer>      │
│ - downStops: TreeSet<Integer>    │
├──────────────────────────────────┤
│ + addStop(floor: int)            │
│ + move()                         │
│ + emergencyStop()                │
│ + getTotalPendingStops()         │
└──────────────────────────────────┘
           △
           │ manages
           │
┌──────────────────────────────────┐
│     ElevatorController           │
├──────────────────────────────────┤
│ - elevators: List<Elevator>      │
│ - pendingRequests: Queue         │
│ - strategy: SelectionStrategy    │
│ - numberOfFloors: int            │
├──────────────────────────────────┤
│ + requestElevator()              │
│ + selectDestination()            │
│ + run()                          │
│ + step()                         │
│ + displayStatus()                │
└──────────────────────────────────┘
           │
           │ uses
           ▼
┌──────────────────────────────────┐
│   <<interface>>                  │
│ ElevatorSelectionStrategy        │
├──────────────────────────────────┤
│ + selectElevator()               │
└──────────────────────────────────┘
           △
           │
    ┌──────┴─────────┬────────────────┐
    │                │                │
┌─────────────┐ ┌──────────┐ ┌────────────────┐
│ Optimal     │ │ Nearest  │ │LoadBalancing   │
│ Strategy    │ │Strategy  │ │   Strategy     │
└─────────────┘ └──────────┘ └────────────────┘


┌──────────────────────────────────┐
│           Request                │
├──────────────────────────────────┤
│ - sourceFloor: int               │
│ - destinationFloor: int          │
│ - direction: Direction           │
│ - timestamp: long                │
├──────────────────────────────────┤
│ + getSourceFloor()               │
│ + getDestinationFloor()          │
│ + getDirection()                 │
└──────────────────────────────────┘


┌──────────────────────────────────┐
│          Building                │
├──────────────────────────────────┤
│ - numberOfFloors: int            │
│ - controller: ElevatorController │
│ - name: String                   │
├──────────────────────────────────┤
│ + requestElevator()              │
│ + startSimulation()              │
└──────────────────────────────────┘
           │ 1
           │
           │ 1
           ▼
┌──────────────────────────────────┐
│     ElevatorController           │
└──────────────────────────────────┘


┌──────────────────────────────────┐
│       DisplayPanel               │
├──────────────────────────────────┤
│ - elevatorId: int                │
│ - externalDisplay: Display       │
│ - internalDisplay: Display       │
├──────────────────────────────────┤
│ + updateDisplay()                │
└──────────────────────────────────┘
           │
           │ uses
           ▼
┌──────────────────────────────────┐
│   <<interface>>                  │
│         Display                  │
├──────────────────────────────────┤
│ + show(floor, direction)         │
└──────────────────────────────────┘
           △
           │
    ┌──────┴──────┐
    │             │
┌─────────────┐ ┌──────────────┐
│ External    │ │  Internal    │
│ Display     │ │  Display     │
└─────────────┘ └──────────────┘


<<enumeration>>          <<enumeration>>
Direction                ElevatorState
─────────────           ─────────────
UP                      IDLE
DOWN                    MOVING
IDLE                    STOPPED
                        MAINTENANCE
```

## Requirements
1. Multiple elevators in a building
2. Users can request elevator from any floor
3. Users can select destination floor from inside elevator
4. Optimize for minimal wait time
5. Handle emergency stops
6. Display current floor and direction

## Core Classes

### 1. Elevator
```java
public class Elevator {
    private int id;
    private int currentFloor;
    private Direction direction;
    private ElevatorState state;
    private int capacity;
    private int currentLoad;
    private TreeSet<Integer> upStops;    // Floors to stop going up
    private TreeSet<Integer> downStops;  // Floors to stop going down
    
    public Elevator(int id, int capacity) {
        this.id = id;
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
        this.state = ElevatorState.IDLE;
        this.capacity = capacity;
        this.currentLoad = 0;
        this.upStops = new TreeSet<>();
        this.downStops = new TreeSet<>();
    }
    
    public void addStop(int floor) {
        if (floor > currentFloor) {
            upStops.add(floor);
        } else if (floor < currentFloor) {
            downStops.add(floor);
        }
        
        if (state == ElevatorState.IDLE) {
            state = ElevatorState.MOVING;
            direction = floor > currentFloor ? Direction.UP : Direction.DOWN;
        }
    }
    
    public void move() {
        if (state != ElevatorState.MOVING) {
            return;
        }
        
        if (direction == Direction.UP) {
            currentFloor++;
            if (upStops.contains(currentFloor)) {
                arriveAtFloor();
            }
            
            if (upStops.isEmpty()) {
                if (!downStops.isEmpty()) {
                    direction = Direction.DOWN;
                } else {
                    state = ElevatorState.IDLE;
                    direction = Direction.IDLE;
                }
            }
        } else if (direction == Direction.DOWN) {
            currentFloor--;
            if (downStops.contains(currentFloor)) {
                arriveAtFloor();
            }
            
            if (downStops.isEmpty()) {
                if (!upStops.isEmpty()) {
                    direction = Direction.UP;
                } else {
                    state = ElevatorState.IDLE;
                    direction = Direction.IDLE;
                }
            }
        }
    }
    
    private void arriveAtFloor() {
        state = ElevatorState.STOPPED;
        if (direction == Direction.UP) {
            upStops.remove(currentFloor);
        } else {
            downStops.remove(currentFloor);
        }
        
        openDoors();
        closeDoors();
        state = ElevatorState.MOVING;
    }
    
    private void openDoors() {
        System.out.println("Elevator " + id + " opening doors at floor " + currentFloor);
    }
    
    private void closeDoors() {
        System.out.println("Elevator " + id + " closing doors at floor " + currentFloor);
    }
    
    public void emergencyStop() {
        state = ElevatorState.MAINTENANCE;
        upStops.clear();
        downStops.clear();
        direction = Direction.IDLE;
    }
    
    public int getTotalPendingStops() {
        return upStops.size() + downStops.size();
    }
    
    // Getters
    public int getId() { return id; }
    public int getCurrentFloor() { return currentFloor; }
    public Direction getDirection() { return direction; }
    public ElevatorState getState() { return state; }
    public boolean isIdle() { return state == ElevatorState.IDLE; }
}

public enum Direction {
    UP, DOWN, IDLE
}

public enum ElevatorState {
    IDLE, MOVING, STOPPED, MAINTENANCE
}
```

### 2. Request
```java
public class Request {
    private int sourceFloor;
    private int destinationFloor;
    private Direction direction;
    private long timestamp;
    
    public Request(int sourceFloor, int destinationFloor) {
        this.sourceFloor = sourceFloor;
        this.destinationFloor = destinationFloor;
        this.direction = destinationFloor > sourceFloor ? 
            Direction.UP : Direction.DOWN;
        this.timestamp = System.currentTimeMillis();
    }
    
    // Getters
    public int getSourceFloor() { return sourceFloor; }
    public int getDestinationFloor() { return destinationFloor; }
    public Direction getDirection() { return direction; }
}
```

### 3. ElevatorController
```java
public class ElevatorController {
    private List<Elevator> elevators;
    private Queue<Request> pendingRequests;
    private ElevatorSelectionStrategy strategy;
    private int numberOfFloors;
    
    public ElevatorController(int numElevators, int capacity, int numberOfFloors) {
        this.elevators = new ArrayList<>();
        for (int i = 0; i < numElevators; i++) {
            elevators.add(new Elevator(i, capacity));
        }
        this.pendingRequests = new LinkedList<>();
        this.strategy = new OptimalElevatorStrategy();
        this.numberOfFloors = numberOfFloors;
    }
    
    public void requestElevator(Request request) {
        Elevator selected = strategy.selectElevator(elevators, request);
        
        if (selected != null) {
            selected.addStop(request.getSourceFloor());
        } else {
            pendingRequests.offer(request);
        }
    }
    
    public void selectDestination(int elevatorId, int floor) {
        elevators.stream()
            .filter(e -> e.getId() == elevatorId)
            .findFirst()
            .ifPresent(e -> e.addStop(floor));
    }
    
    public void run() {
        // Simulation loop
        while (true) {
            step();
            displayStatus();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
    
    private void step() {
        // Move all elevators
        for (Elevator elevator : elevators) {
            elevator.move();
        }
        
        // Process pending requests
        processPendingRequests();
    }
    
    private void processPendingRequests() {
        Iterator<Request> it = pendingRequests.iterator();
        while (it.hasNext()) {
            Request request = it.next();
            Elevator selected = strategy.selectElevator(elevators, request);
            if (selected != null) {
                selected.addStop(request.getSourceFloor());
                it.remove();
            }
        }
    }
    
    public void displayStatus() {
        System.out.println("=== Elevator Status ===");
        for (Elevator e : elevators) {
            System.out.printf("Elevator %d: Floor %d, %s, %s%n",
                e.getId(), e.getCurrentFloor(), 
                e.getDirection(), e.getState());
        }
        System.out.println();
    }
}
```

### 4. ElevatorSelectionStrategy (Strategy Pattern)
```java
public interface ElevatorSelectionStrategy {
    Elevator selectElevator(List<Elevator> elevators, Request request);
}

public class OptimalElevatorStrategy implements ElevatorSelectionStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, Request request) {
        Elevator best = null;
        int minCost = Integer.MAX_VALUE;
        
        for (Elevator elevator : elevators) {
            if (elevator.getState() == ElevatorState.MAINTENANCE) {
                continue;
            }
            
            int cost = calculateCost(elevator, request);
            if (cost < minCost) {
                minCost = cost;
                best = elevator;
            }
        }
        
        return best;
    }
    
    private int calculateCost(Elevator elevator, Request request) {
        int cost = 0;
        int sourceFloor = request.getSourceFloor();
        Direction requestDir = request.getDirection();
        
        if (elevator.isIdle()) {
            // Idle elevator: just distance
            cost = Math.abs(elevator.getCurrentFloor() - sourceFloor);
        } else if (elevator.getDirection() == requestDir) {
            // Moving in same direction
            if (requestDir == Direction.UP && sourceFloor >= elevator.getCurrentFloor()) {
                cost = sourceFloor - elevator.getCurrentFloor();
            } else if (requestDir == Direction.DOWN && sourceFloor <= elevator.getCurrentFloor()) {
                cost = elevator.getCurrentFloor() - sourceFloor;
            } else {
                cost = Integer.MAX_VALUE; // Wrong direction
            }
        } else {
            // Moving in opposite direction
            cost = elevator.getTotalPendingStops() * 2 + 
                   Math.abs(elevator.getCurrentFloor() - sourceFloor);
        }
        
        return cost;
    }
}

public class NearestElevatorStrategy implements ElevatorSelectionStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, Request request) {
        return elevators.stream()
            .filter(e -> e.getState() != ElevatorState.MAINTENANCE)
            .min((e1, e2) -> Integer.compare(
                Math.abs(e1.getCurrentFloor() - request.getSourceFloor()),
                Math.abs(e2.getCurrentFloor() - request.getSourceFloor())
            ))
            .orElse(null);
    }
}

public class LoadBalancingStrategy implements ElevatorSelectionStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, Request request) {
        return elevators.stream()
            .filter(e -> e.getState() != ElevatorState.MAINTENANCE)
            .min((e1, e2) -> Integer.compare(
                e1.getTotalPendingStops(),
                e2.getTotalPendingStops()
            ))
            .orElse(null);
    }
}
```

### 5. Building
```java
public class Building {
    private int numberOfFloors;
    private ElevatorController controller;
    private String name;
    
    public Building(String name, int numberOfFloors, 
                    int numElevators, int elevatorCapacity) {
        this.name = name;
        this.numberOfFloors = numberOfFloors;
        this.controller = new ElevatorController(
            numElevators, 
            elevatorCapacity, 
            numberOfFloors
        );
    }
    
    public void requestElevator(int sourceFloor, int destinationFloor) {
        validateFloor(sourceFloor);
        validateFloor(destinationFloor);
        
        Request request = new Request(sourceFloor, destinationFloor);
        controller.requestElevator(request);
    }
    
    private void validateFloor(int floor) {
        if (floor < 0 || floor >= numberOfFloors) {
            throw new IllegalArgumentException(
                "Invalid floor: " + floor
            );
        }
    }
    
    public void startSimulation() {
        controller.run();
    }
    
    public String getName() { return name; }
    public int getNumberOfFloors() { return numberOfFloors; }
}
```

### 6. Display Panel
```java
public class DisplayPanel {
    private int elevatorId;
    private Display externalDisplay;
    private Display internalDisplay;
    
    public DisplayPanel(int elevatorId) {
        this.elevatorId = elevatorId;
        this.externalDisplay = new ExternalDisplay();
        this.internalDisplay = new InternalDisplay();
    }
    
    public void updateDisplay(int floor, Direction direction, ElevatorState state) {
        externalDisplay.show(floor, direction);
        internalDisplay.show(floor, direction);
    }
}

public interface Display {
    void show(int floor, Direction direction);
}

public class ExternalDisplay implements Display {
    @Override
    public void show(int floor, Direction direction) {
        String arrow = direction == Direction.UP ? "↑" : 
                      direction == Direction.DOWN ? "↓" : "-";
        System.out.println("External: Floor " + floor + " " + arrow);
    }
}

public class InternalDisplay implements Display {
    @Override
    public void show(int floor, Direction direction) {
        String arrow = direction == Direction.UP ? "↑" : 
                      direction == Direction.DOWN ? "↓" : "-";
        System.out.println("Internal: Current Floor " + floor + " " + arrow);
    }
}
```

## Usage Example
```java
public class ElevatorSystemDemo {
    public static void main(String[] args) {
        // Create a building with 10 floors, 3 elevators, capacity 8
        Building building = new Building("Tech Tower", 10, 3, 8);
        
        // Simulate requests
        building.requestElevator(0, 5);  // Ground to 5th floor
        building.requestElevator(3, 7);  // 3rd to 7th floor
        building.requestElevator(8, 1);  // 8th to 1st floor
        
        // Start simulation
        building.startSimulation();
    }
}
```

## Key Algorithms

### SCAN Algorithm (Elevator Algorithm)
- Elevator continues in current direction until no more requests
- Then reverses direction
- Minimizes direction changes
- Used in disk scheduling too

### LOOK Algorithm
- Similar to SCAN but reverses before reaching end
- More efficient in practice

## Key Design Patterns Used
1. **Strategy Pattern**: Different elevator selection strategies
2. **State Pattern**: Elevator states (IDLE, MOVING, STOPPED, MAINTENANCE)
3. **Observer Pattern**: For displays and notifications
4. **Singleton**: Can be applied to Building/Controller

## Interview Talking Points

1. **Optimization Strategies**:
   - Minimize wait time
   - Balance load across elevators
   - Consider direction and distance

2. **Real-time Constraints**:
   - Time-stepped simulation
   - Priority queue for requests
   
3. **Scalability**:
   - Can handle multiple buildings
   - Easy to add more elevators
   
4. **Thread Safety**:
   - Synchronize elevator state updates
   - Thread-safe request queue

5. **Trade-offs**:
   - Simple nearest vs optimal selection
   - Fairness vs efficiency
   - Individual queues vs central dispatcher

## Possible Extensions
1. Add VIP/Express elevators
2. Add weight sensors for overload detection
3. Add peak hour optimization (morning up, evening down)
4. Add elevator groups for different floor ranges
5. Add predictive algorithms using ML
6. Add emergency evacuation mode
7. Add energy optimization (sleep mode for idle elevators)
8. Add destination dispatch system (Singapore MRT style)

## Complexity Analysis
- **Request Processing**: O(n) where n = number of elevators
- **Move Operation**: O(log k) where k = pending stops
- **Space**: O(n * k) for all elevator queues
