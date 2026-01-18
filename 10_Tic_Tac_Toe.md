# Tic Tac Toe - Low Level Design

## UML Class Diagram

```
┌─────────────────────────────────┐
│          Board                  │
├─────────────────────────────────┤
│ - cells: Cell[][]               │
│ - SIZE: int = 3                 │
├─────────────────────────────────┤
│ + makeMove(row, col, symbol)    │
│ + isFull(): boolean             │
│ + checkWin(symbol): boolean     │
│ + display()                     │
│ - checkRow(): boolean           │
│ - checkColumn(): boolean        │
│ - checkDiagonals(): boolean     │
└─────────────────────────────────┘
           │ 3x3
           ▼
    ┌──────────────────────┐
    │       Cell           │
    ├──────────────────────┤
    │ - row: int           │
    │ - col: int           │
    │ - symbol: Symbol     │
    ├──────────────────────┤
    │ + isEmpty(): boolean │
    │ + getSymbol()        │
    │ + setSymbol()        │
    └──────────────────────┘


┌─────────────────────────────────┐
│          Player                 │
├─────────────────────────────────┤
│ - name: String                  │
│ - symbol: Symbol                │
├─────────────────────────────────┤
│ + getName(): String             │
│ + getSymbol(): Symbol           │
└─────────────────────────────────┘
           △
           │
    ┌──────┴──────┐
    │             │
┌─────────┐  ┌──────────┐
│ Player  │  │AIPlayer  │
└─────────┘  └──────────┘


┌─────────────────────────────────┐
│         AIPlayer                │
├─────────────────────────────────┤
│ - difficulty: DifficultyLevel   │
├─────────────────────────────────┤
│ + getMove(board): Move          │
│ - getRandomMove(): Move         │
│ - getSmartMove(): Move          │
│ - getMiniMaxMove(): Move        │
│ - minimax(board, bool): int     │
│ - findWinningMove(): Move       │
│ - getAvailableMoves(): List     │
└─────────────────────────────────┘


┌─────────────────────────────────┐
│      TicTacToeGame              │
├─────────────────────────────────┤
│ - board: Board                  │
│ - player1: Player               │
│ - player2: Player               │
│ - currentPlayer: Player         │
│ - status: GameStatus            │
├─────────────────────────────────┤
│ + makeMove(row, col): boolean   │
│ + start()                       │
└─────────────────────────────────┘
           │ 1
           │
           │ 1
           ▼
    ┌──────────┐
    │  Board   │
    └──────────┘


┌─────────────────────────────────┐
│          Move                   │
├─────────────────────────────────┤
│ + row: int                      │
│ + col: int                      │
└─────────────────────────────────┘


<<enumeration>>        <<enumeration>>
Symbol                 GameStatus
───────────           ──────────────
X                     IN_PROGRESS
O                     WIN
EMPTY                 DRAW


<<enumeration>>
DifficultyLevel
───────────────
EASY
MEDIUM
HARD
```

## Core Classes

```java
// 1. Board
public class Board {
    private Cell[][] cells;
    private static final int SIZE = 3;
    
    public Board() {
        cells = new Cell[SIZE][SIZE];
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                cells[i][j] = new Cell(i, j);
            }
        }
    }
    
    public boolean makeMove(int row, int col, Symbol symbol) {
        if (row < 0 || row >= SIZE || col < 0 || col >= SIZE) {
            return false;
        }
        
        if (cells[row][col].isEmpty()) {
            cells[row][col].setSymbol(symbol);
            return true;
        }
        return false;
    }
    
    public boolean isFull() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (cells[i][j].isEmpty()) {
                    return false;
                }
            }
        }
        return true;
    }
    
    public boolean checkWin(Symbol symbol) {
        // Check rows
        for (int i = 0; i < SIZE; i++) {
            if (checkRow(i, symbol)) return true;
        }
        
        // Check columns
        for (int j = 0; j < SIZE; j++) {
            if (checkColumn(j, symbol)) return true;
        }
        
        // Check diagonals
        return checkDiagonals(symbol);
    }
    
    private boolean checkRow(int row, Symbol symbol) {
        for (int j = 0; j < SIZE; j++) {
            if (cells[row][j].getSymbol() != symbol) {
                return false;
            }
        }
        return true;
    }
    
    private boolean checkColumn(int col, Symbol symbol) {
        for (int i = 0; i < SIZE; i++) {
            if (cells[i][col].getSymbol() != symbol) {
                return false;
            }
        }
        return true;
    }
    
    private boolean checkDiagonals(Symbol symbol) {
        // Main diagonal
        boolean mainDiag = true;
        for (int i = 0; i < SIZE; i++) {
            if (cells[i][i].getSymbol() != symbol) {
                mainDiag = false;
                break;
            }
        }
        
        // Anti diagonal
        boolean antiDiag = true;
        for (int i = 0; i < SIZE; i++) {
            if (cells[i][SIZE - 1 - i].getSymbol() != symbol) {
                antiDiag = false;
                break;
            }
        }
        
        return mainDiag || antiDiag;
    }
    
    public void display() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                System.out.print(cells[i][j].getSymbol() + " ");
            }
            System.out.println();
        }
        System.out.println();
    }
}

// 2. Cell
public class Cell {
    private int row;
    private int col;
    private Symbol symbol;
    
    public Cell(int row, int col) {
        this.row = row;
        this.col = col;
        this.symbol = Symbol.EMPTY;
    }
    
    public boolean isEmpty() {
        return symbol == Symbol.EMPTY;
    }
    
    public Symbol getSymbol() { return symbol; }
    public void setSymbol(Symbol symbol) { this.symbol = symbol; }
}

// 3. Player
public class Player {
    private String name;
    private Symbol symbol;
    
    public Player(String name, Symbol symbol) {
        this.name = name;
        this.symbol = symbol;
    }
    
    public String getName() { return name; }
    public Symbol getSymbol() { return symbol; }
}

// 4. Game
public class TicTacToeGame {
    private Board board;
    private Player player1;
    private Player player2;
    private Player currentPlayer;
    private GameStatus status;
    
    public TicTacToeGame(String player1Name, String player2Name) {
        this.board = new Board();
        this.player1 = new Player(player1Name, Symbol.X);
        this.player2 = new Player(player2Name, Symbol.O);
        this.currentPlayer = player1;
        this.status = GameStatus.IN_PROGRESS;
    }
    
    public boolean makeMove(int row, int col) {
        if (status != GameStatus.IN_PROGRESS) {
            System.out.println("Game is over!");
            return false;
        }
        
        if (board.makeMove(row, col, currentPlayer.getSymbol())) {
            board.display();
            
            if (board.checkWin(currentPlayer.getSymbol())) {
                status = GameStatus.WIN;
                System.out.println(currentPlayer.getName() + " wins!");
                return true;
            }
            
            if (board.isFull()) {
                status = GameStatus.DRAW;
                System.out.println("It's a draw!");
                return true;
            }
            
            // Switch player
            currentPlayer = (currentPlayer == player1) ? player2 : player1;
            return true;
        }
        
        System.out.println("Invalid move! Try again.");
        return false;
    }
    
    public void start() {
        Scanner scanner = new Scanner(System.in);
        board.display();
        
        while (status == GameStatus.IN_PROGRESS) {
            System.out.println(currentPlayer.getName() + "'s turn (" + 
                             currentPlayer.getSymbol() + ")");
            System.out.print("Enter row (0-2): ");
            int row = scanner.nextInt();
            System.out.print("Enter col (0-2): ");
            int col = scanner.nextInt();
            
            makeMove(row, col);
        }
    }
}

// 5. Enums
public enum Symbol {
    X, O, EMPTY;
    
    @Override
    public String toString() {
        return this == EMPTY ? "-" : name();
    }
}

public enum GameStatus {
    IN_PROGRESS, WIN, DRAW
}

// 6. AI Player (Bonus)
public class AIPlayer extends Player {
    private DifficultyLevel difficulty;
    
    public AIPlayer(String name, Symbol symbol, DifficultyLevel difficulty) {
        super(name, symbol);
        this.difficulty = difficulty;
    }
    
    public Move getMove(Board board) {
        switch (difficulty) {
            case EASY:
                return getRandomMove(board);
            case MEDIUM:
                return getSmartMove(board);
            case HARD:
                return getMiniMaxMove(board);
            default:
                return getRandomMove(board);
        }
    }
    
    private Move getRandomMove(Board board) {
        List<Move> availableMoves = getAvailableMoves(board);
        Random random = new Random();
        return availableMoves.get(random.nextInt(availableMoves.size()));
    }
    
    private Move getSmartMove(Board board) {
        // Try to win
        Move winMove = findWinningMove(board, getSymbol());
        if (winMove != null) return winMove;
        
        // Block opponent
        Symbol opponentSymbol = (getSymbol() == Symbol.X) ? Symbol.O : Symbol.X;
        Move blockMove = findWinningMove(board, opponentSymbol);
        if (blockMove != null) return blockMove;
        
        // Random move
        return getRandomMove(board);
    }
    
    private Move getMiniMaxMove(Board board) {
        // Implement Minimax algorithm
        int bestScore = Integer.MIN_VALUE;
        Move bestMove = null;
        
        for (Move move : getAvailableMoves(board)) {
            // Try move
            board.makeMove(move.row, move.col, getSymbol());
            int score = minimax(board, false);
            // Undo move
            board.makeMove(move.row, move.col, Symbol.EMPTY);
            
            if (score > bestScore) {
                bestScore = score;
                bestMove = move;
            }
        }
        
        return bestMove;
    }
    
    private int minimax(Board board, boolean isMaximizing) {
        // Check terminal states
        if (board.checkWin(getSymbol())) return 1;
        Symbol opponent = (getSymbol() == Symbol.X) ? Symbol.O : Symbol.X;
        if (board.checkWin(opponent)) return -1;
        if (board.isFull()) return 0;
        
        if (isMaximizing) {
            int bestScore = Integer.MIN_VALUE;
            for (Move move : getAvailableMoves(board)) {
                board.makeMove(move.row, move.col, getSymbol());
                int score = minimax(board, false);
                board.makeMove(move.row, move.col, Symbol.EMPTY);
                bestScore = Math.max(score, bestScore);
            }
            return bestScore;
        } else {
            int bestScore = Integer.MAX_VALUE;
            Symbol opponent = (getSymbol() == Symbol.X) ? Symbol.O : Symbol.X;
            for (Move move : getAvailableMoves(board)) {
                board.makeMove(move.row, move.col, opponent);
                int score = minimax(board, true);
                board.makeMove(move.row, move.col, Symbol.EMPTY);
                bestScore = Math.min(score, bestScore);
            }
            return bestScore;
        }
    }
    
    private Move findWinningMove(Board board, Symbol symbol) {
        for (Move move : getAvailableMoves(board)) {
            board.makeMove(move.row, move.col, symbol);
            boolean wins = board.checkWin(symbol);
            board.makeMove(move.row, move.col, Symbol.EMPTY);
            if (wins) return move;
        }
        return null;
    }
    
    private List<Move> getAvailableMoves(Board board) {
        // Get all empty cells
        List<Move> moves = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (board.cells[i][j].isEmpty()) {
                    moves.add(new Move(i, j));
                }
            }
        }
        return moves;
    }
}

public class Move {
    public int row;
    public int col;
    
    public Move(int row, int col) {
        this.row = row;
        this.col = col;
    }
}

public enum DifficultyLevel {
    EASY, MEDIUM, HARD
}
```

## Usage Example
```java
public class TicTacToeDemo {
    public static void main(String[] args) {
        // Human vs Human
        TicTacToeGame game = new TicTacToeGame("Alice", "Bob");
        game.start();
        
        // Human vs AI
        TicTacToeGame aiGame = new TicTacToeGame(
            "Alice",
            new AIPlayer("Computer", Symbol.O, DifficultyLevel.HARD)
        );
        aiGame.start();
    }
}
```

## Key Design Patterns
- **Strategy Pattern**: AI difficulty levels
- **Command Pattern**: Move execution
- **State Pattern**: Game status
- **Template Method**: Board checking logic

## Key Points
1. Win condition checking
2. AI with Minimax algorithm
3. Move validation
4. Game state management

---

