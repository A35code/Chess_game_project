# 🧩 Full-Featured Python Tkinter Chess Game

This is a complete chess game built from scratch in **Python** using **Tkinter** for the graphical user interface.  
It includes all the standard chess rules such as **en-passant**, **castling**, **promotion**, **check/checkmate/stalemate detection**, and **undo / restart / save / load** functionalities.  
An optional **AI opponent** is also integrated with selectable difficulty levels.

---

## 🎯 Overview

The program is structured into three main layers:

1. **Game Logic (`GameState`)** — Handles board state, rules, move validation, and chess-specific mechanics.  
2. **GUI Layer (`ChessApp`)** — Manages the Tkinter interface, user interaction, drawing, and event handling.  
3. **Persistence & AI** — Responsible for saving/loading games, undo history, and generating AI moves.

---

## 🧠 Game Logic: The `GameState` Class

This class represents the **core engine** of the chess game. It defines the board, rules, and movement logic.

### **Constructor: `__init__(self)`**

- Initializes the 8×8 chessboard as a nested list.  
- Places all standard chess pieces using single-character codes:
  - Uppercase → White pieces (`R`, `N`, `B`, `Q`, `K`, `P`)
  - Lowercase → Black pieces (`r`, `n`, `b`, `q`, `k`, `p`)
- Sets initial turn (`white_to_move = True`)
- Initializes game history, castling rights, en-passant target square, and flags for checkmate/stalemate.

---

### **Method: `make_move(self, r, c, nr, nc)`**

This method performs a **complete move operation**. It:
1. Moves a piece from `(r, c)` → `(nr, nc)` on the board.
2. Handles **pawn promotion** automatically (usually promotes to Queen).
3. Executes **castling** (moves rook accordingly).
4. Performs **en-passant capture** if applicable.
5. Updates the **game history** for undo functionality.
6. Switches the player turn.
7. Checks for check, checkmate, or stalemate after the move.

---

### **Method: `undo_move(self)`**

- Reverts the game to the **previous state** by popping the last move from `self.history`.
- Restores the board, current player, castling rights, and en-passant target.
- This allows players to take back moves freely.

---

### **Method: `generate_legal_moves(self, for_color=None)`**

This is one of the most important functions in the engine.  
It returns a list of **all valid moves** for the current (or specified) player.

Internally it:
1. Iterates over all pieces of the given color.
2. Calls `generate_moves_for_square()` to get all candidate moves.
3. Simulates each move on a **temporary copy of the board**.
4. Filters out moves that would leave the king in check.
5. Returns a list of legal moves in `(r, c, nr, nc)` format.

This method also correctly simulates **castling** and **en-passant** moves during validation.

---

### **Method: `generate_moves_for_square(self, r, c)`**

This function generates **raw movement patterns** for a piece located at `(r, c)`  
without considering checks or king safety.

It defines how each piece moves:
- **Pawn:** Forward moves, diagonal captures, and en-passant possibilities.
- **Knight:** L-shaped jumps.
- **Bishop:** Diagonal sliding moves.
- **Rook:** Horizontal and vertical sliding moves.
- **Queen:** Combination of rook and bishop movements.
- **King:** One-square moves in all directions, plus castling.

---

### **Method: `is_in_check(self, color, board=None)`**

Determines if the given color’s **king is under attack**.

- Scans the board to find the king’s coordinates.
- Checks if any enemy piece has a move that targets the king’s square.
- Returns `True` if the king is in check; `False` otherwise.

This method is crucial for verifying move legality, detecting checkmate, and evaluating game termination conditions.

---

### **Method: `get_king_position(self, color, board)`**

A helper function that locates the coordinates `(row, col)` of a specific color’s king on the board.

Used internally by `is_in_check()` and `generate_legal_moves()`.

---

### **Method: `in_bounds(self, r, c)`**

Returns `True` if `(r, c)` lies within the 8×8 board, and `False` otherwise.  
Used as a safety check during move generation.

---

### **Method: `is_square_attacked(self, r, c, by_color)`**

Checks whether the square `(r, c)` is attacked by any piece of `by_color`.

It iterates through all pieces of that color and examines if any of them can legally move to `(r, c)`.

---

### **Method: `is_checkmate(self, color)`**

Returns `True` if:
1. The player of `color` is in check, **and**
2. Has **no legal moves**.

Used after every move to determine whether the game should end with a “Checkmate!” message.

---

### **Method: `is_stalemate(self, color)`**

Returns `True` if:
1. The player of `color` is **not in check**, and  
2. Has **no legal moves**.

When both conditions are true, the game ends in a **draw**.

---

### **Method: `save_game(self, filename)`**

Saves the current game state (board, history, turn, etc.) into a **JSON file**.  
This allows resuming games later.

---

### **Method: `load_game(self, filename)`**

Reads a previously saved JSON file and restores all game parameters into memory.

---

### **Method: `get_ai_move(self, difficulty)`**

Implements simple AI behavior for one side.

Depending on difficulty:
- **Easy:** Makes random legal moves.
- **Medium:** Random moves with basic heuristics (avoid obvious losses).
- **Hard:** Searches for moves that maximize positional advantage (simplified minimax).

---

## 🎨 GUI: The `ChessApp` Class

This class is responsible for the **user interface** using Tkinter.  
It draws the board, handles mouse clicks, updates piece graphics, and displays alerts.

### **Constructor: `__init__(self, root)`**

- Initializes the main Tkinter window.
- Creates a canvas grid for the chessboard.
- Loads piece icons.
- Binds mouse click events for selecting and moving pieces.
- Displays side indicators, score counters, and menu buttons (Undo, Save, Load, Restart).

---

### **Method: `draw_board(self)`**

Draws the 8×8 checkerboard pattern on the canvas using alternating light/dark colors.

---

### **Method: `draw_pieces(self)`**

Loops through the board matrix in `GameState` and draws the appropriate image for each piece on its square.

---

### **Method: `on_square_click(self, event)`**

Handles user mouse clicks:
1. On the first click → selects a piece to move.
2. On the second click → attempts to move to the clicked destination.
3. If the move is legal → calls `game.make_move()` and updates the board.
4. After the move → checks for checkmate/stalemate, plays sounds, or triggers AI moves.

---

### **Method: `update_status(self)`**

Updates the GUI labels for:
- Whose turn it is.
- Whether the player is in check.
- Current game outcome (Checkmate, Stalemate, Draw).

---

### **Method: `show_message(self, msg)`**

Displays a modal message box (Tkinter `messagebox`) for notifications like:
- “Checkmate! White wins.”
- “Stalemate — Draw.”
- “Invalid move.”

---

### **Method: `restart_game(self)`**

Resets everything to the starting position using a new `GameState` object.  
Clears history, resets turn, and redraws the board.

---

### **Method: `undo_move(self)`**

Calls `GameState.undo_move()` and refreshes the GUI.

---

### **Method: `save_game(self)` / `load_game(self)`**

Wraps around the respective `GameState` functions for persistence.

---

## 🧩 Additional Features

| Feature | Description |
|----------|-------------|
| **En Passant** | Implemented via `self.en_passant` square tracking. |
| **Castling** | Automatically validated and executed when legal. |
| **Undo** | Fully state-aware; supports multi-step undo. |
| **AI Opponent** | Selectable difficulty; move generator uses random or heuristic logic. |
| **Save / Load** | Saves game to JSON; restores complete state. |
| **Promotion** | Pawns automatically promote to Queens. |
| **Check / Checkmate / Stalemate** | Verified after every move using `is_in_check()` and `generate_legal_moves()`. |

---

## ⚙️ How It All Works Together

1. The user clicks a piece.
2. The GUI calls `generate_legal_moves()` to highlight legal destinations.
3. When a move is chosen, `make_move()` updates the board and triggers `is_in_check()`.
4. If the move results in checkmate/stalemate, `ChessApp` shows the appropriate message.
5. Otherwise, the turn toggles and the game continues.

---

## 💡 Developer Notes

- Board coordinates follow `(row, column)` order with `(0,0)` = top-left (White’s rook).
- Piece notation matches standard chess characters for simplicity.
- All move validation logic is encapsulated within `GameState`; the GUI only visualizes it.
- The separation of logic and GUI ensures the game could later be adapted for CLI, web, or multiplayer versions.

---

## 🏁 Summary

This project demonstrates:
- **Object-Oriented Design** in Python  
- **Event-driven programming** using Tkinter  
- **Game state management**, move generation, and rule enforcement  
- **AI and file persistence integration**  
- **Practical handling of complex rules** like castling and en-passant  

Every part of the game — from move validation to board drawing — was written manually without external chess libraries, making it both a learning project and a fully functional chess application.

## 👨‍💻Authors/Creators
Group 20

---
