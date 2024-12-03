# Building My Chess Engine pt.1 - Designing a Dynamic Chessboard

*This is part of [Building my Chess Engine Series]()*

Chess has always fascinated me because of its complexity and the depth of strategy it offers.
 Inspired by the likes of famous chess engines, I set out to build my own, 'Nitsa', 
 combining AI techniques and a passion for coding. 

## Overview

Chess Nitsa is an Android application featuring its own chess engine, aptly named "Nitsa." 
The goal of this project is to build an engine capable of outplaying human players below the 
Grandmaster (GM) level, leveraging AI algorithms for decision-making, evaluation, pruning,
 and optimization techniques.

For now, the focus is on developing the core engine, keeping things simple by using a high-level 
language to prioritize ease of development, 
even if that comes at the cost of some performance. 
I will also be utilizing the [chesslib](https://github.com/bhlangonijr/chesslib) open source module
 to handle core game mechanics, which will allow me to concentrate on higher-level engine 
 functionality like AI decision-making and strategy, rather than user interface or experience.
 
In the future, once the engine is more refined, I plan to optimize performance-critical components 
by possibly moving them to C++ or even rewriting the entire engine from scratch if necessary.

# Designing a Dynamic Chessboard in Android

In this first part of the series, I’ve implemented the chessboard rendering and interaction using a GridLayout.

This component will be reused in different activities such as playing against an engine, analysis, 
and loading saved positions. The design focus was on clean architecture with a clear separation of concerns. 


## Architecture Overview

I followed a **Model-View-ViewModel (MVVM)** architecture pattern, which is android’s recommended practice, 
in order to separate the core components of the app into: 

- **View (ChessBoardView)**: Responsible for rendering the chessboard and to handle user interactions 
with the chessboard. 
- **ViewModel (ChessBoardViewModel)**: To manage the game state, including piece movements and rules, 
and to provide data to the view. 
- **Model (Board,Piece, Square, Move)**: To represent the game objects and logic, encapsulated in classes. 

This approach ensures scalability and separation of concerns, keeping the engine logic separate from the UI. 


Additionally some utility classes were created:

**Utility Classes**

- **ChessPositionMapper**: This class further decouples the chess logic from the UI by handling 
conversions between grid positions and chess notations. It contains various helper methods 
in order to bridge the gap between the chessboard’s UI representation (2D grid coordinates) 
and the internal chess engine’s representation. 
- **ChessUtils**: Contains various helper methods to handle the conversion between board 
indices and chess notation (algebraic chess notation). 


## Functionality Breakdown 

### ChessBoardView 
In the Chessboard view component, I have created a reusable chessboard for layout files. The chessboard is implemented using a custom `GridLayout` with 8 rows and 8 columns. Each grid square contains an `ImageView` representing a chess square. The squares alternate color based on their position to create a proper chessboard. 

The grid-based layout is created dynamically during the view's initialization. Each square (`ImageView`) has an `onClickListener` to handle click events, allowing players to select and move pieces interactively. Every time a piece is moved, both the beginning square and the target square are highlighted. When the next player starts, the existing highlight is cleared. 

**Methods**

- **`init`**: Creates the chessboard grid and adds `setOnClickListener` to each square (onSquareClick).
- **`setSquareSize`**: Sets the square size, which is used during initialization. This allows the chessboard to be created dynamically based on the activity’s layout, supporting rotation and resizing. It also enables reusability in multiple activities and sizes.
- **`updateBoardLayout`**: Updates the board layout when the square size changes without recreating the squares. Supports dynamic resizing.
- **`updateBoard`**: Updates the board state with new positions (per square), moving pieces, clearing squares, and removing captured pieces.
- **`onSquareClick`**: Calls `chessPieceViewModel.tryMovePiece` to move the piece and calls `highlightSelectedSquare()` on both the beginning and end squares. It also clears the highlighted squares when the next player starts.

### ChessPieceViewModel 
The ViewModel acts as a middle layer between the UI (view) and the game logic (model). It initializes the chessboard with pieces in their starting positions and exposes the current state via `LiveData`, observable by the view. The board state is encapsulated in a `LiveData` object that holds a 2D array of `ChessPiece` objects. It contains logic for validating and executing chess moves, though actual functionality is decoupled into helper methods.

**Methods**
 
- **`initializeBoard`**: Sets up the initial positions of all chess pieces.
- **`tryMovePiece`**: Handles the logic for moving a piece. It checks if a move is legal using `isMoveInLegalMoves()`, and if valid, updates the board state by physically moving the piece in the 2D array and the chess engine (`Board` object).
- **`isMoveInLegalMoves`**: Checks if the move is legal by validating against the position, move squares, and the board using `ChessHelper`.
- **`updateBoard`, `updateBoardForCastling`**: Updates the chessboard state and provides feedback (move notation) for the UI.

### ChessPositionMapper 
The `ChessPositionMapper` is a utility class responsible for mapping between user-friendly chess notation (e.g., "e4", "d5") and internal data structures such as row-column indexing on the 8x8 chessboard grid. It ensures consistency and accuracy in move validation, display, and execution.

**Methods**

- **`convertMoveToNotation`**: Converts a `Move` object to algebraic notation for recording games in standard formats like PGN and displaying move history.
- **`getCoordinatesFromMovesString`**: Converts the engine's move string to array coordinates, e.g., `"b1c3"` => `[fromRow: 7, fromColumn: 1, toRow: 5, toColumn: 2]`.
- **`getSquareByMove`**: Creates a `Square` object from row and column values.
- **`getMoveBySquare`**: Converts a `Square` object back into row-column coordinates.
- **`chessPiecePositionFromNotation`**: Finds the `Piece` object related to the notation and updates its position (Rank, File), then returns the updated piece.

### ChessUtils 
The `ChessUtils` class serves as a collection of utility functions for various chess-related operations. It handles reusable tasks such as move validation, determining check/checkmate conditions, and managing board states. This modular approach simplifies future extensions and feature additions.

## Video Demo 
Below is a video showcasing the dynamic chessboard implemented in the first part of the Nitsa chess engine development. It demonstrates the chessboard setup, piece movements, and interaction handling.


<iframe 
  width="800" 
  height="450" 
  src="https://www.youtube.com/embed/IeO9PEmbFuA" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
  allowfullscreen>
</iframe>


## Design Principles Followed 

- **Separation of Concerns (SoC)**: Divides functionality across different components.
- **Single Responsibility Principle (SRP)**: Each class has a single responsibility, except for the slightly overloaded `ChessBoardView`. Plans exist to move interaction logic to a service.
- **DRY (Don’t Repeat Yourself)**: Encapsulates chess logic in reusable methods (`tryMovePiece`, `convertMoveToNotation`, `getSquareByMove`).
- **KISS (Keep It Simple, Stupid)**: Keeps the architecture simple, avoiding complex patterns, especially for basic chess logic.
- **Modularity**: Logic is divided across different classes, making the system modular and allowing components to evolve independently.

In this first part, I laid the foundation for the chessboard and interaction. In the next post, I’ll dive into implementing the basic structure for the chess engine and algorithms to make it functional. Stay tuned as Nitsa continues to evolve!

## Attribution 

This project uses `chesslib`, an open-source chess library licensed under the Apache License 2.0. 
`chesslib` provides essential functionalities for generating legal chess moves and game mechanics, 
integrated into the Nitsa chess engine for core chess logic.

## References

- [chesslib](https://github.com/bhlangonijr/chesslib)
