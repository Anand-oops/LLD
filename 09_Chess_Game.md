# Chess Game - Low Level Design

## Core Classes

```java
// 1. Piece (Abstract)
public abstract class Piece {
    private Color color;
    private Position position;
    private boolean isAlive;
    
    public Piece(Color color, Position position) {
        this.color = color;
        this.position = position;
        this.isAlive = true;
    }
    
    public abstract boolean canMove(Board board, Position to);
    public abstract List<Position> getPossibleMoves(Board board);
    
    public void move(Position newPosition) {
        this.position = newPosition;
    }
    
    public void kill() { this.isAlive = false; }
}

// 2. Specific Pieces
public class King extends Piece {
    private boolean hasMoved = false;
    
    @Override
    public boolean canMove(Board board, Position to) {
        int dx = Math.abs(to.getX() - getPosition().getX());
        int dy = Math.abs(to.getY() - getPosition().getY());
        
        // King moves one square
        if (dx <= 1 && dy <= 1 && (dx + dy) > 0) {
            return !board.isPositionUnderAttack(to, getColor());
        }
        
        // Castling
        if (!hasMoved && dx == 2 && dy == 0) {
            return canCastle(board, to);
        }
        
        return false;
    }
    
    private boolean canCastle(Board board, Position to) {
        // Check castling conditions
        return false; // Simplified
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        List<Position> moves = new ArrayList<>();
        int[] dx = {-1, -1, -1, 0, 0, 1, 1, 1};
        int[] dy = {-1, 0, 1, -1, 1, -1, 0, 1};
        
        for (int i = 0; i < 8; i++) {
            Position newPos = new Position(
                getPosition().getX() + dx[i],
                getPosition().getY() + dy[i]
            );
            if (canMove(board, newPos)) {
                moves.add(newPos);
            }
        }
        return moves;
    }
}

public class Queen extends Piece {
    @Override
    public boolean canMove(Board board, Position to) {
        // Moves like rook + bishop
        Rook rook = new Rook(getColor(), getPosition());
        Bishop bishop = new Bishop(getColor(), getPosition());
        return rook.canMove(board, to) || bishop.canMove(board, to);
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        // Combine rook and bishop moves
        return new ArrayList<>();
    }
}

public class Rook extends Piece {
    @Override
    public boolean canMove(Board board, Position to) {
        // Horizontal or vertical
        return (getPosition().getX() == to.getX() || 
                getPosition().getY() == to.getY()) &&
               board.isPathClear(getPosition(), to);
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        List<Position> moves = new ArrayList<>();
        // Add all horizontal and vertical moves
        return moves;
    }
}

public class Bishop extends Piece {
    @Override
    public boolean canMove(Board board, Position to) {
        // Diagonal
        int dx = Math.abs(to.getX() - getPosition().getX());
        int dy = Math.abs(to.getY() - getPosition().getY());
        return dx == dy && board.isPathClear(getPosition(), to);
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        return new ArrayList<>();
    }
}

public class Knight extends Piece {
    @Override
    public boolean canMove(Board board, Position to) {
        int dx = Math.abs(to.getX() - getPosition().getX());
        int dy = Math.abs(to.getY() - getPosition().getY());
        return (dx == 2 && dy == 1) || (dx == 1 && dy == 2);
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        List<Position> moves = new ArrayList<>();
        int[] dx = {-2, -2, -1, -1, 1, 1, 2, 2};
        int[] dy = {-1, 1, -2, 2, -2, 2, -1, 1};
        
        for (int i = 0; i < 8; i++) {
            Position newPos = new Position(
                getPosition().getX() + dx[i],
                getPosition().getY() + dy[i]
            );
            if (board.isValidPosition(newPos) && canMove(board, newPos)) {
                moves.add(newPos);
            }
        }
        return moves;
    }
}

public class Pawn extends Piece {
    private boolean hasMoved = false;
    
    @Override
    public boolean canMove(Board board, Position to) {
        int direction = getColor() == Color.WHITE ? 1 : -1;
        int dx = to.getX() - getPosition().getX();
        int dy = to.getY() - getPosition().getY();
        
        // Move forward
        if (dx == direction && dy == 0 && board.isEmpty(to)) {
            return true;
        }
        
        // First move two squares
        if (!hasMoved && dx == 2 * direction && dy == 0 && 
            board.isEmpty(to) && board.isEmpty(
                new Position(getPosition().getX() + direction, getPosition().getY())
            )) {
            return true;
        }
        
        // Capture diagonally
        if (dx == direction && Math.abs(dy) == 1 && 
            board.hasOpponentPiece(to, getColor())) {
            return true;
        }
        
        return false;
    }
    
    @Override
    public List<Position> getPossibleMoves(Board board) {
        return new ArrayList<>();
    }
}

// 3. Board
public class Board {
    private Piece[][] board;
    private static final int SIZE = 8;
    
    public Board() {
        board = new Piece[SIZE][SIZE];
        initializeBoard();
    }
    
    private void initializeBoard() {
        // Place white pieces
        board[0][0] = new Rook(Color.WHITE, new Position(0, 0));
        board[0][1] = new Knight(Color.WHITE, new Position(0, 1));
        // ... place all pieces
        
        // Place pawns
        for (int i = 0; i < SIZE; i++) {
            board[1][i] = new Pawn(Color.WHITE, new Position(1, i));
            board[6][i] = new Pawn(Color.BLACK, new Position(6, i));
        }
    }
    
    public Piece getPiece(Position pos) {
        return board[pos.getX()][pos.getY()];
    }
    
    public boolean isEmpty(Position pos) {
        return getPiece(pos) == null;
    }
    
    public boolean isPathClear(Position from, Position to) {
        int dx = Integer.compare(to.getX() - from.getX(), 0);
        int dy = Integer.compare(to.getY() - from.getY(), 0);
        
        int x = from.getX() + dx;
        int y = from.getY() + dy;
        
        while (x != to.getX() || y != to.getY()) {
            if (!isEmpty(new Position(x, y))) {
                return false;
            }
            x += dx;
            y += dy;
        }
        return true;
    }
    
    public boolean movePiece(Position from, Position to) {
        Piece piece = getPiece(from);
        if (piece == null || !piece.canMove(this, to)) {
            return false;
        }
        
        // Capture piece if present
        Piece captured = getPiece(to);
        if (captured != null) {
            captured.kill();
        }
        
        // Move piece
        board[to.getX()][to.getY()] = piece;
        board[from.getX()][from.getY()] = null;
        piece.move(to);
        
        return true;
    }
    
    public boolean isValidPosition(Position pos) {
        return pos.getX() >= 0 && pos.getX() < SIZE &&
               pos.getY() >= 0 && pos.getY() < SIZE;
    }
}

// 4. Game
public class ChessGame {
    private Board board;
    private Player white;
    private Player black;
    private Player currentPlayer;
    private GameStatus status;
    private List<Move> moveHistory;
    
    public ChessGame(Player white, Player black) {
        this.board = new Board();
        this.white = white;
        this.black = black;
        this.currentPlayer = white;
        this.status = GameStatus.ACTIVE;
        this.moveHistory = new ArrayList<>();
    }
    
    public boolean makeMove(Position from, Position to) {
        if (status != GameStatus.ACTIVE) {
            return false;
        }
        
        Piece piece = board.getPiece(from);
        if (piece == null || piece.getColor() != currentPlayer.getColor()) {
            return false;
        }
        
        if (board.movePiece(from, to)) {
            moveHistory.add(new Move(from, to, piece));
            
            // Check game end conditions
            if (isCheckmate(getOpponent())) {
                status = GameStatus.CHECKMATE;
            } else if (isStalemate(getOpponent())) {
                status = GameStatus.STALEMATE;
            }
            
            // Switch player
            currentPlayer = getOpponent();
            return true;
        }
        
        return false;
    }
    
    private boolean isCheckmate(Player player) {
        return isKingInCheck(player) && !hasLegalMoves(player);
    }
    
    private boolean isStalemate(Player player) {
        return !isKingInCheck(player) && !hasLegalMoves(player);
    }
    
    private boolean isKingInCheck(Player player) {
        // Check if king is under attack
        return false; // Simplified
    }
    
    private boolean hasLegalMoves(Player player) {
        // Check if player has any legal moves
        return true; // Simplified
    }
    
    private Player getOpponent() {
        return currentPlayer == white ? black : white;
    }
}

// 5. Supporting Classes
public class Position {
    private int x, y;
    
    public Position(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int getX() { return x; }
    public int getY() { return y; }
}

public class Player {
    private String name;
    private Color color;
    
    public Player(String name, Color color) {
        this.name = name;
        this.color = color;
    }
    
    public Color getColor() { return color; }
}

public class Move {
    private Position from;
    private Position to;
    private Piece piece;
    private Piece capturedPiece;
    private LocalDateTime timestamp;
    
    public Move(Position from, Position to, Piece piece) {
        this.from = from;
        this.to = to;
        this.piece = piece;
        this.timestamp = LocalDateTime.now();
    }
}

public enum Color { WHITE, BLACK }
public enum GameStatus { ACTIVE, CHECKMATE, STALEMATE, RESIGNATION, DRAW }
```

## Key Design Patterns
- **Template Method**: Piece movement validation
- **Factory Pattern**: Piece creation
- **Command Pattern**: Move execution and undo
- **Strategy Pattern**: Different AI strategies

## Key Points
1. Piece movement validation
2. Check and checkmate detection
3. Special moves (castling, en passant)
4. Move history for undo
5. Game state management

---

