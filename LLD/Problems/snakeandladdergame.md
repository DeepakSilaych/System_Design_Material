# Designing Snake and Ladder Game

## Question
Design Snake and Ladder Game

## Requirements
1. The game should be played on a board with numbered cells, typically with 100 cells.
2. The board should have a predefined set of snakes and ladders, connecting certain cells.
3. The game should support multiple players, each represented by a unique game piece.
4. Players should take turns rolling a dice to determine the number of cells to move forward.
5. If a player lands on a cell with the head of a snake, they should slide down to the cell with the tail of the snake.
6. If a player lands on a cell with the base of a ladder, they should climb up to the cell at the top of the ladder.
7. The game should continue until one of the players reaches the final cell on the board.
8. The game should handle multiple game sessions concurrently, allowing different groups of players to play independently.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Key components include: Board, Player, Snake, Ladder, Dice, SnakeAndLadderGame, GameManager,
SnakeAndLadderDemo.

### Design Details
1. The **Board** class represents the game board with a fixed size (e.g., 100 cells). It contains the positions of snakes and ladders and provides methods to initialize them and retrieve the new position after encountering a snake or ladder.
2. The **Player** class represents a player in the game, with properties such as name and current position on the board.
3. The **Snake** class represents a snake on the board, with properties for the start and end positions.
4. The **Ladder** class represents a ladder on the board, with properties for the start and end positions.
5. The **Dice** class represents a dice used in the game, with a method to roll the dice and return a random value between 1 and 6.
6. The **SnakeAndLadderGame** class represents a single game session. It initializes the game with a board, a list of players, and a dice. The play method handles the game loop, where players take turns rolling the dice and moving their positions on the board. It checks for snakes and ladders and updates the player's position accordingly. The game continues until a player reaches the final position on the board.
7. The **GameManager** class is a singleton that manages multiple game sessions. It maintains a list of active games and provides a method to start a new game with a list of player names. Each game is started in a separate thread to allow concurrent game sessions.
8. The **SnakeAndLadderDemo** class demonstrates the usage of the game by creating an instance of the GameManager and starting two separate game sessions with different sets of players.

### Implementation (Python)
#### board.py

```python
from board_entity import BoardEntity
from typing import List

class Board:
    def __init__(self, size: int, entities: List[BoardEntity]):
        self.size = size
        self.snakes_and_ladders = {}
        
        for entity in entities:
            self.snakes_and_ladders[entity.get_start()] = entity.get_end()
    
    def get_size(self) -> int:
        return self.size
    
    def get_final_position(self, position: int) -> int:
        return self.snakes_and_ladders.get(position, position)
```

#### board_entity.py

```python
from abc import ABC

class BoardEntity(ABC):
    def __init__(self, start: int, end: int):
        self.start = start
        self.end = end
    
    def get_start(self) -> int:
        return self.start
    
    def get_end(self) -> int:
        return self.end
```

#### dice.py

```python
import random

class Dice:
    def __init__(self, min_value: int, max_value: int):
        self.min_value = min_value
        self.max_value = max_value

    def roll(self) -> int:
        return int(random.random() * (self.max_value - self.min_value + 1) + self.min_value)
```

#### game.py

```python
from game_status import GameStatus
from player import Player
from board import Board
from dice import Dice
from snake import Snake
from ladder import Ladder
from typing import List
from collections import deque
from board_entity import BoardEntity

class Game:
    class Builder:
        def __init__(self):
            self.board = None
            self.players = None
            self.dice = None
        
        def set_board(self, board_size: int, board_entities: List[BoardEntity]):
            self.board = Board(board_size, board_entities)
            return self
        
        def set_players(self, player_names: List[str]):
            self.players = deque()
            for player_name in player_names:
                self.players.append(Player(player_name))
            return self
        
        def set_dice(self, dice: Dice):
            self.dice = dice
            return self
        
        def build(self):
            if self.board is None or self.players is None or self.dice is None:
                raise ValueError("Board, Players, and Dice must be set.")
            return Game(self)
    
    def __init__(self, builder: 'Game.Builder'):
        self.board = builder.board
        self.players = deque(builder.players)
        self.dice = builder.dice
        self.status = GameStatus.NOT_STARTED
        self.winner = None
    
    def play(self):
        if len(self.players) < 2:
            print("Cannot start game. At least 2 players are required.")
            return
        
        self.status = GameStatus.RUNNING
        print("Game started!")
        
        while self.status == GameStatus.RUNNING:
            current_player = self.players.popleft()
            self.take_turn(current_player)
            
            # If the game is not finished and the player didn't roll a 6, add them back to the queue
            if self.status == GameStatus.RUNNING:
                self.players.append(current_player)
        
        print("Game Finished!")
        if self.winner is not None:
            print(f"The winner is {self.winner.get_name()}!")
    
    def take_turn(self, player: Player):
        roll = self.dice.roll()
        print(f"\n{player.get_name()}'s turn. Rolled a {roll}.")
        
        current_position = player.get_position()
        next_position = current_position + roll
        
        if next_position > self.board.get_size():
            print(f"Oops, {player.get_name()} needs to land exactly on {self.board.get_size()}. Turn skipped.")
            return
        
        if next_position == self.board.get_size():
            player.set_position(next_position)
            self.winner = player
            self.status = GameStatus.FINISHED
            print(f"Hooray! {player.get_name()} reached the final square {self.board.get_size()} and won!")
            return
        
        final_position = self.board.get_final_position(next_position)
        
        if final_position > next_position:  # Ladder
            print(f"Wow! {player.get_name()} found a ladder 🪜 at {next_position} and climbed to {final_position}.")
        elif final_position < next_position:  # Snake
            print(f"Oh no! {player.get_name()} was bitten by a snake 🐍 at {next_position} and slid down to {final_position}.")
        else:
            print(f"{player.get_name()} moved from {current_position} to {final_position}.")
        
        player.set_position(final_position)
        
        if roll == 6:
            print(f"{player.get_name()} rolled a 6 and gets another turn!")
            self.take_turn(player)
```

#### game_status.py

```python
from enum import Enum

class GameStatus(Enum):
    NOT_STARTED = "NOT_STARTED"
    RUNNING = "RUNNING"
    FINISHED = "FINISHED"
```

#### ladder.py

```python
from board_entity import BoardEntity

class Ladder(BoardEntity):
    def __init__(self, start: int, end: int):
        super().__init__(start, end)
        if start >= end:
            raise ValueError("Ladder bottom must be at a lower position than its top.")
```

#### player.py

```python
class Player:
    def __init__(self, name: str):
        self.name = name
        self.position = 0

    def get_name(self) -> str:
        return self.name

    def get_position(self) -> int:
        return self.position

    def set_position(self, position: int):
        self.position = position
```

#### snake.py

```python
from board_entity import BoardEntity

class Snake(BoardEntity):
    def __init__(self, start: int, end: int):
        super().__init__(start, end)
        if start <= end:
            raise ValueError("Snake head must be at a higher position than its tail.")
```

#### snake_and_ladder_demo.py

```python
from game import Game
from snake import Snake
from ladder import Ladder
from dice import Dice

class SnakeAndLadderDemo:
    @staticmethod
    def main():
        board_entities = [
            Snake(17, 7), Snake(54, 34),
            Snake(62, 19), Snake(98, 79),
            Ladder(3, 38), Ladder(24, 33),
            Ladder(42, 93), Ladder(72, 84)
        ]
        
        players = ["Alice", "Bob", "Charlie"]
        
        game = Game.Builder() \
            .set_board(100, board_entities) \
            .set_players(players) \
            .set_dice(Dice(1, 6)) \
            .build()
        
        game.play()


if __name__ == "__main__":
    SnakeAndLadderDemo.main()
```