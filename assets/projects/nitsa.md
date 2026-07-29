# Nitsa Chess Engine

#### *This page demonstrates the application itself. If you're interested in the technical details, you can find them in [Building my Chess Engine Series](/blog?tag=chess-engine-series).*



Nitsa started as a simple Android project, 2 years ago, because I wanted to understand how chess engines actually work. Reading about [Minimax](https://www.chessprogramming.org/Minimax) and [Alpha-Beta pruning](https://www.chessprogramming.org/Alpha-Beta) is one thing, but writing them and then watching your engine sacrifice a queen for absolutely no reason is a much more educational experience. The first version was mostly a dynamic chessboard with minimax algorithm, connected to `chesslib`, but it slowly grew into a real engine with move ordering, relative evaluation, positional evaluation, quiescence search, opening logic, caching, and parallel execution. I was building it for fun and to understand the [ai algorithms](https://en.wikipedia.org/wiki/List_of_artificial_intelligence_algorithms), which is usually how I end up spending far more time on a project than any reasonable person would.

Eventually Android became the wrong home for it. Chess search is CPU-heavy, and a phone is designed to save battery and remain cool, while Nitsa wants to examine as many positions as possible and turn the processor into a heater. I migrated the project to Java 21 and JavaFX, keeping the engine in Java but giving it a proper desktop environment. The result is now more than an engine with a board attached to it. The app offers a bunch of cool features like live games, position analysis, and the ability to load and replay PGN files. You can check out engine statistics, explore principal variations, and even play matches against Stockfish. Plus, it lets you tweak the search depth, set time limits, and adjust thread usage. Because, you know, just building the engine wasn’t enough, I also needed some tools to understand and tweek when it wasn’t performing as expected.

## Architecture: Keeping it Clean with Separation of Concerns

The application follows an MVVM-based layered architecture. JavaFX views handle the board, windows, tables, and user interaction. The ViewModel translates between the visual `ChessPiece[][]` representation and the real [`chesslib Board`](https://github.com/bhlangonijr/chesslib), while controllers decide whether the application is in live, replay, or analysis mode. Below that, services handle engine searches, PGN loading, background execution, and Stockfish management. The search and evaluation code lives in its own algorithm layer, with separate helpers for move ordering, bitboards, relative evaluation, positional evaluation, constants, and domain models. In simple terms, a mouse click should not know what a knight outpost is, and Negamax should not care which shade of brown I picked for the chessboard. Keeping those concerns separate from the early begining, made the desktop migration easier and allowed me to improve the engine without redesigning the UI every weekend.

## How the Algorithms Work Together

The real search best move flow is the following:

![Search Best Move Flow](assets/images/chess_algorithm.png)

`searchBestMove` starts with the legal moves and searches the position repeatedly, first at depth one, then depth two, and so on. This is [iterative deepening](https://www.chessprogramming.org/Iterative_Deepening). It may sound wasteful to search the same position again, but the previous iteration gives the engine a very good guess about the best move and score. That guess is used to order the next search and to create a narrow [aspiration window](https://www.chessprogramming.org/Aspiration_Windows) around the expected score. A narrow window allows [Alpha-Beta](https://www.chessprogramming.org/Alpha-Beta) to reject useless branches earlier. If the real score falls outside it, Nitsa widens the window and searches again. Yes, sometimes doing extra work saves work. Chess engines are perfectly comfortable with this kind of nonsense.

[Negamax](https://www.chessprogramming.org/Negamax) performs the recursive search and uses Alpha-Beta pruning to stop exploring lines that can no longer improve the result. [Principal Variation Search](https://www.chessprogramming.org/Principal_Variation_Search) makes this more aggressive by fully searching the first and most promising move, then testing later moves with a very small window. If one of them looks really good, it gets a full search. Move ordering is therefore extremely important: [transposition table](https://www.chessprogramming.org/Transposition_Table) moves, strong captures, killer moves, history scores, checks, promotions, and opening heuristics try to put the best candidates first so pruning can do its job. The transposition table also stores positions already examined using their [Zobrist keys](https://www.chessprogramming.org/Zobrist_Hashing), preventing Nitsa from calculating the same thing again through a different move order.

When the normal depth reaches zero, the engine does not immediately trust the position because that can produce the [horizon effect](https://www.chessprogramming.org/Horizon_Effect). A position may look great only because the search stopped one move before a queen was captured 😉. Quiescence search continues through tactical moves such as captures, promotions, and required check evasions until the position becomes quiet enough to evaluate. The final evaluation uses [bitboards](https://www.chessprogramming.org/Bitboards) and combines material, piece-square tables, mobility, pawn structure, passed pawns, king safety, development, open files, and other positional features. None of these techniques is especially magical alone. The strength comes from how they cooperate: good ordering improves pruning, pruning makes deeper searches possible, iterative deepening improves ordering again, caching removes repeated work, and quiescence prevents the final evaluation from being confidently wrong at the worst possible moment.

Nitsa is not Stockfish, and that was never the point. The point was to take algorithms that looked abstract on paper and turn them into a complete application that can play, analyze, measure itself, and lose against Stockfish in a controlled and statistically useful manner 😎. It has been one of my favorite projects because every improvement exposed another problem, which is either the joy of engineering or a very specific form of punishment.

## Video Demo

Below is a short video of Nitsa running as a desktop application.

[youtube:gkZk7t6WAqk]
