# Designing a Chess Game

## Question
Design a Chess Game

## Requirements
1. The chess game should follow the standard rules of chess.
2. The game should support two players, each controlling their own set of pieces.
3. The game board should be represented as an 8x8 grid, with alternating black and white squares.
4. Each player should have 16 pieces: 1 king, 1 queen, 2 rooks, 2 bishops, 2 knights, and 8 pawns.
5. The game should validate legal moves for each piece and prevent illegal moves.
6. The game should detect checkmate and stalemate conditions.
7. The game should handle player turns and allow players to make moves alternately.
8. The game should provide a user interface for players to interact with the game.

## Solution
### Design overview
The model is organized to mirror how a chess game executes: the domain pieces and board first, then
player interaction, then the game loop.

- Keep movement logic local to each piece (SRP).
- Centralize board validation (in-bounds, destination occupancy, check/stalemate detection).
- Have the `Game` orchestrate turns and end-conditions, not low-level movement.

### Core model
- `Piece` (abstract) with concrete `King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn`
- `Board` (8x8 matrix, placement, move validation, end-state queries)
- `Move` (value object capturing intent)
- `Player` (selects and applies a `Move`)
- `Game` (turns, input/output, termination)

### Key flows
1. Input → `Move` creation
2. Validation → `Board.is_valid_move` delegates the pattern-specific rule to the concrete `Piece`
3. Apply move → `Player.make_move` commits source/destination changes on the `Board`
4. End check → `Game` queries board for checkmate/stalemate

### Recommended code reading order
1) `color.py`, 2) `piece.py`, 3) concrete pieces (`king.py`, `queen.py`, `rook.py`, `bishop.py`, `knight.py`, `pawn.py`),  
4) `move.py`, 5) `board.py`, 6) `player.py`, 7) `game.py`, 8) demo.

### Design Details
1. The **Piece** class is an abstract base class representing a chess piece. It contains common attributes such as color, row, and column, and declares an abstract method canMove to be implemented by each specific piece class.
2. The **King**, **Queen**, **Rook**, **Bishop**, **Knight**, and **Pawn** classes extend the Piece class and implement their respective movement logic in the canMove method.
3. The **Board** class represents the chess board and manages the placement of pieces. It provides methods to get and set pieces on the board, check the validity of moves, and determine checkmate and stalemate conditions.
4. The **Player** class represents a player in the game and has a method to make a move on the board.
5. The Move class represents a move made by a player, containing the piece being moved and the destination coordinates.
6. The **Game** class orchestrates the overall game flow. It initializes the board, handles player turns, and determines the game result.
7. The **ChessGame** class is the entry point of the application and starts the game.

### Implementation (Python)
#### bishop.py

```python
from piece import Piece

class Bishop(Piece):
    def can_move(self, board, dest_row, dest_col):
        row_diff = abs(dest_row - self.row)
        col_diff = abs(dest_col - self.col)
        return row_diff == col_diff
```

#### board.py

```python
from rook import Rook
from knight import Knight
from bishop import Bishop
from queen import Queen
from king import King
from pawn import Pawn
from color import Color

class Board:
    def __init__(self):
        self.board = [[None] * 8 for _ in range(8)]
        self._initialize_board()

    def _initialize_board(self):
        # Initialize white pieces
        self.board[0][0] = Rook(Color.WHITE, 0, 0)
        self.board[0][1] = Knight(Color.WHITE, 0, 1)
        self.board[0][2] = Bishop(Color.WHITE, 0, 2)
        self.board[0][3] = Queen(Color.WHITE, 0, 3)
        self.board[0][4] = King(Color.WHITE, 0, 4)
        self.board[0][5] = Bishop(Color.WHITE, 0, 5)
        self.board[0][6] = Knight(Color.WHITE, 0, 6)
        self.board[0][7] = Rook(Color.WHITE, 0, 7)
        for i in range(8):
            self.board[1][i] = Pawn(Color.WHITE, 1, i)

        # Initialize black pieces
        self.board[7][0] = Rook(Color.BLACK, 7, 0)
        self.board[7][1] = Knight(Color.BLACK, 7, 1)
        self.board[7][2] = Bishop(Color.BLACK, 7, 2)
        self.board[7][3] = Queen(Color.BLACK, 7, 3)
        self.board[7][4] = King(Color.BLACK, 7, 4)
        self.board[7][5] = Bishop(Color.BLACK, 7, 5)
        self.board[7][6] = Knight(Color.BLACK, 7, 6)
        self.board[7][7] = Rook(Color.BLACK, 7, 7)
        for i in range(8):
            self.board[6][i] = Pawn(Color.BLACK, 6, i)

    def get_piece(self, row, col):
        return self.board[row][col]

    def set_piece(self, row, col, piece):
        self.board[row][col] = piece

    def is_valid_move(self, piece, dest_row, dest_col):
        if piece is None or dest_row < 0 or dest_row > 7 or dest_col < 0 or dest_col > 7:
            return False
        dest_piece = self.board[dest_row][dest_col]
        return (dest_piece is None or dest_piece.color != piece.color) and \
               piece.can_move(self, dest_row, dest_col)

    def is_checkmate(self, color):
        # TODO: Implement checkmate logic
        return False

    def is_stalemate(self, color):
        # TODO: Implement stalemate logic
        return False
```

#### chess_game_demo.py

```python
from game import Game

class ChessGameDemo:
    @staticmethod
    def run():
        game = Game()
        game.start()

if __name__ == "__main__":
    ChessGameDemo.run()
```

#### color.py

```python
from enum import Enum

class Color(Enum):
    WHITE = 1
    BLACK = 2
```

#### game.py

```python
from board import Board
from player import Player
from color import Color
from move import Move

class Game:
    def __init__(self):
        self.board = Board()
        self.players = [Player(Color.WHITE), Player(Color.BLACK)]
        self.current_player = 0

    def start(self):
        # Game loop
        while not self._is_game_over():
            player = self.players[self.current_player]
            print(f"{player.color.name}'s turn.")

            # Get move from the player
            move = self._get_player_move(player)

            # Make the move on the board
            player.make_move(self.board, move)

            # Switch to the next player
            self.current_player = (self.current_player + 1) % 2

        # Display game result
        self._display_result()

    def _is_game_over(self):
        return self.board.is_checkmate(Color.WHITE) or self.board.is_checkmate(Color.BLACK) or \
               self.board.is_stalemate(Color.WHITE) or self.board.is_stalemate(Color.BLACK)

    def _get_player_move(self, player):
        # TODO: Implement logic to get a valid move from the player
        # For simplicity, let's assume the player enters the move via console input
        source_row = int(input("Enter source row: "))
        source_col = int(input("Enter source column: "))
        dest_row = int(input("Enter destination row: "))
        dest_col = int(input("Enter destination column: "))

        piece = self.board.get_piece(source_row, source_col)
        if piece is None or piece.color != player.color:
            raise ValueError("Invalid piece selection!")

        return Move(piece, dest_row, dest_col)

    def _display_result(self):
        if self.board.is_checkmate(Color.WHITE):
            print("Black wins by checkmate!")
        elif self.board.is_checkmate(Color.BLACK):
            print("White wins by checkmate!")
        elif self.board.is_stalemate(Color.WHITE) or self.board.is_stalemate(Color.BLACK):
            print("The game ends in a stalemate!")
```

#### king.py

```python
from piece import Piece

class King(Piece):
    def can_move(self, board, dest_row, dest_col):
        row_diff = abs(dest_row - self.row)
        col_diff = abs(dest_col - self.col)
        return row_diff <= 1 and col_diff <= 1
```

#### knight.py

```python
from piece import Piece

class Knight(Piece):
    def can_move(self, board, dest_row, dest_col):
        row_diff = abs(dest_row - self.row)
        col_diff = abs(dest_col - self.col)
        return (row_diff == 2 and col_diff == 1) or (row_diff == 1 and col_diff == 2)
```

#### move.py

```python
class Move:
    def __init__(self, piece, dest_row, dest_col):
        self.piece = piece
        self.dest_row = dest_row
        self.dest_col = dest_col
```

#### pawn.py

```python
from piece import Piece
from color import Color

class Pawn(Piece):
    def can_move(self, board, dest_row, dest_col):
        row_diff = dest_row - self.row
        col_diff = abs(dest_col - self.col)

        if self.color == Color.WHITE:
            return (row_diff == 1 and col_diff == 0) or \
                   (self.row == 1 and row_diff == 2 and col_diff == 0) or \
                   (row_diff == 1 and col_diff == 1 and board.get_piece(dest_row, dest_col) is not None)
        else:
            return (row_diff == -1 and col_diff == 0) or \
                   (self.row == 6 and row_diff == -2 and col_diff == 0) or \
                   (row_diff == -1 and col_diff == 1 and board.get_piece(dest_row, dest_col) is not None)
```

#### piece.py

```python
from abc import ABC, abstractmethod
from color import Color

class Piece(ABC):
    def __init__(self, color, row, col):
        self.color = color
        self.row = row
        self.col = col

    @abstractmethod
    def can_move(self, board, dest_row, dest_col):
        pass
```

#### player.py

```python
class Player:
    def __init__(self, color):
        self.color = color

    def make_move(self, board, move):
        piece = move.piece
        dest_row = move.dest_row
        dest_col = move.dest_col

        if board.is_valid_move(piece, dest_row, dest_col):
            source_row = piece.row
            source_col = piece.col
            board.set_piece(source_row, source_col, None)
            board.set_piece(dest_row, dest_col, piece)
            piece.row = dest_row
            piece.col = dest_col
        else:
            raise ValueError("Invalid move!")
```

#### queen.py

```python
from piece import Piece

class Queen(Piece):
    def can_move(self, board, dest_row, dest_col):
        row_diff = abs(dest_row - self.row)
        col_diff = abs(dest_col - self.col)
        return (row_diff == col_diff) or (self.row == dest_row or self.col == dest_col)
```

#### rook.py

```python
from piece import Piece

class Rook(Piece):
    def can_move(self, board, dest_row, dest_col):
        return self.row == dest_row or self.col == dest_col
```