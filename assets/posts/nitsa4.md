# Building a Chess Engine pt.4 - From Android to a Java Desktop App

*This is the final part of [Building my Chess Engine Series](/blog?tag=chess-engine-series)*

When I started building **Nitsa**, the project lived inside an Android application. That was a useful place to begin: I could create the chessboard, connect it to the engine, and test the ai search algorithms easily.

But chess engines are CPU-hungry applications because the search tree grows exponentially. A typical chess position has roughly 30 to 35 legal moves. Without pruning, searching only one move (ply) deeper does not add another 35 positions; it can multiply the existing work by about 35. At an average branching factor of 35, a depth of four plies can theoretically produce around 1.5 million positions, while six plies can exceed 1.8 billion. 

Brute-force computing power alone is not enough, so we found smart ways to solve this via different algorithms.
Alpha-Beta pruning, good move ordering, transposition caching, and other search techniques reduce that number dramatically, but they do not make the problem cheap. Quiescence search may also continue beyond the requested depth when the final position is tactically unstable. A phone is optimized for battery life and short interactive tasks, not for keeping every available CPU core busy while searching a game tree that appears to have a personal problem with exponential growth.

For that reason, I finally migrated Nitsa from Android to a **Java desktop application using JavaFX**.

This was not simply a change of UI. Since the third part of the series, Nitsa gained a new search pipeline, better evaluation, time management, engine statistics, proper analysis tools, PGN support, and Stockfish integration. Somewhere along the way, it stopped feeling like an Android app stretched across a larger screen and became a desktop application in its own right.


## The Desktop Architecture

The desktop version is built with Java 21, JavaFX, Maven, [`chesslib`](https://github.com/bhlangonijr/chesslib), and Jackson. I kept the architecture layered and close to MVVM because the separation already worked well in the Android version.

The JavaFX views render the board, tables, dialogs, and engine output. `ChessBoardViewModel` translates between the visual `ChessPiece[][]` array and the real `chesslib Board`. Controllers keep track of whether the board is being used for a live game, a replay, or analysis. Services handle engine searches, PGN loading, Stockfish communication, and paired matches. Finally, the algorithm layer contains the search, move ordering, evaluation, bitboard helpers, and transposition table.

The basic direction is:

`JavaFX View -> ViewModel -> ChessEngine -> Search -> Evaluation`

The result then comes back as:

`Best Move + Score + Principal Variation -> ViewModel -> JavaFX View`

This boundary is important. The search code knows nothing about buttons and images, while the board view does not need to understand Alpha-Beta bounds. A mouse click should not know what a killer move is, and Negamax should not care which shade of brown I selected for the board. Everybody has enough problems already.

The UI also has its own background executor, while the engine owns a separate search executor. This avoids a subtle threading problem I had, where the application submits `searchBestMove` to a pool and that same search tries to submit more tasks to the already occupied pool. That is a nice recipe for thread starvation and for staring at a frozen chessboard while questioning all previous life choices 😉.


## The New Search Pipeline

In the second part of the series, Nitsa already had Negamax, Alpha-Beta pruning, quiescence search, move ordering, multithreading, and a transposition table. The desktop version keeps those ideas but makes them work as a more complete pipeline:

`searchBestMove -> Iterative Deepening -> Aspiration Window -> Root Search -> Negamax + Alpha-Beta + Principal Variation Search -> Quiescence Search -> Evaluation`

This order matters. Aspiration windows do not run after Negamax, and Principal Variation Search is not a replacement for Negamax. They wrap and specialize parts of the same search.


## Iterative Deepening

[Iterative deepening](https://www.chessprogramming.org/Iterative_Deepening) searches the same position several times. Nitsa starts at depth one, completes it, then searches depth two, depth three, and continues until it reaches the configured limit or runs out of time.

At first this sounds like deliberately repeating work, which is usually considered a bug. In a chess engine it is extremely useful. A shallow search quickly finds a likely best move. Nitsa places that move first during the next, deeper iteration, which improves Alpha-Beta pruning. The previous result also provides an expected score for the aspiration window.

Iterative deepening gives time management a clean fallback as well. Every search session can have a deadline and can be cancelled. If time expires halfway through depth seven, Nitsa returns the result from the last fully completed depth instead of returning a random half-calculated opinion. The search checks for cancellation throughout the tree, so starting another search, pausing a match, or closing the application does not require waiting for the old one to finish.


## Aspiration Windows

A normal Alpha-Beta search can use a very wide score range, from "this is a disaster" to "forced checkmate." The problem is that a wide range gives pruning less information.

After the first iterative deepening pass, Nitsa already has a score from the previous depth. It therefore begins the next iteration with an [aspiration window](https://www.chessprogramming.org/Aspiration_Windows) of 40 centipawns around that score. If the previous evaluation was +60, the next search initially expects something close to +20 through +100.

When the score remains inside the window, the search usually finishes with more cutoffs and less work. If the score is below alpha, the search fails low. If it is above beta, it fails high. Nitsa then widens the relevant side of the window and searches again. Yes, this can cause a second search, but the successful narrow searches save enough nodes to make it count.

This is another example of a chess-engine optimization that sounds wrong until it starts working: repeat the search so you can search less.


## Negamax, Alpha-Beta, and Principal Variation Search

[Negamax](https://www.chessprogramming.org/Negamax) is still the recursive backbone. As we probably mentioned in the previous articles, chess is a zero-sum game, so the value of a position for one side is the negative value for the other. Instead of maintaining separate maximizing and minimizing functions, Negamax changes the sign when the side to move changes.

[Alpha-Beta pruning](https://www.chessprogramming.org/Alpha-Beta) runs inside that recursion. Alpha represents the best score already found for the current side, while beta is the limit the opponent can force. When alpha reaches or passes beta, the remaining moves cannot change the decision and the branch is cut.

The new addition is [Principal Variation Search](https://www.chessprogramming.org/Principal_Variation_Search), or PVS. Good move ordering means that the first move is expected to be the best one. Nitsa searches that first move with the full Alpha-Beta window. Later moves are initially tested with a tiny null window, only checking whether they can beat the current alpha. Most cannot, so they are rejected cheaply. If one does beat alpha without reaching beta, Nitsa does not trust the cheap test and searches it again with the full window.

PVS depends heavily on move ordering. If the best move is searched first, most later moves fail quickly. If the ordering is terrible, PVS performs re-searches and becomes an expensive way to discover that the engine guessed badly.


## Move Ordering: Helping Alpha-Beta Do Its Job

Move ordering does not change the final answer. It changes how quickly the engine can prove that answer.

The highest priority is the hash move stored in the transposition table, because another search has already found it useful. Captures are scored using the value of the victim, the value of the attacker, and a static exchange estimate. Promotions receive a large bonus. Quiet moves can be promoted by the killer-move and history heuristics, while piece-square-table changes provide a smaller positional hint.

A killer move is a quiet move that caused an Alpha-Beta cutoff at the same search depth somewhere else. It may also work in the current branch, so Nitsa remembers two killers per ply. The history heuristic is broader: whenever a quiet move causes a cutoff, it gains a depth-weighted score. Over the search, moves that repeatedly prove useful slowly move toward the front.

The capture estimate asks a simple question: if this piece captures on that square and can immediately be recaptured, is the material exchange likely to be good? It is not a complete Static Exchange Evaluation, but it is cheap and useful for ordering. The real search still verifies the move, so a heuristic can delay a move but in reality, even if it is wrong, it does not remove a legal tactic.


## The Transposition Table and the Principal Variation

Different move orders can reach the same position. Without caching, the engine calculates that position again and congratulates itself for rediscovering the same answer.

Nitsa identifies positions using the Zobrist key provided by `chesslib`. The desktop engine stores entries in a fixed table. Each entry contains the key, score, searched depth, bound type, and best move. The bound can be exact, a lower bound, or an upper bound, which allows Alpha-Beta to reuse information even when the stored result was produced by a cutoff.

The table uses an [AtomicReferenceArray](https://docs.oracle.com/en/java/javase/21//docs/api/java.base/java/util/concurrent/atomic/AtomicReferenceArray.html), so parallel root searches can read and replace entries safely. It has a fixed size instead of growing forever, and deeper entries are preferred over shallower entries for the same key. The full key is checked before an entry is trusted.

Storing the best move has two benefits. First, that move becomes the hash move for ordering. Second, the engine can reconstruct the [principal variation](https://www.chessprogramming.org/Principal_Variation), the line Nitsa currently considers best. Starting with the chosen root move, it follows the stored best move for each resulting position until the requested depth is reached or a valid entry is missing.

The principal variation is now displayed in analysis mode and in the Stockfish match window. A score such as +0.70 is useful, but seeing the line it is much better. It also makes debugging easier 😎, because a strange evaluation becomes strange for a reason. And that reason helps identifying a bug or a line that I did not think that existed.


## Quiescence Search: Do Not Evaluate in the Middle of an Explosion

When the normal search reaches depth zero, Nitsa does not immediately call the evaluation function. Doing that in a tactical position causes the [horizon effect](https://www.chessprogramming.org/Horizon_Effect). The engine may stop immediately after winning a rook and before noticing that its queen is hanging.

[Quiescence search](https://www.chessprogramming.org/Quiescence_Search) first calculates a stand-pat score, which is the evaluation if the side chooses not to continue a tactical sequence. It then explores captures and promotions until the position becomes quiet or the quiescence limit is reached.

There are two important exceptions. If the king is in check, standing still is not legal, so Nitsa searches every legal evasion, including quiet king moves. 
Quiescence uses the same Alpha-Beta window, move-ordering helpers, cancellation checks, and search statistics as the main search. It is not an unrelated search bolted onto the end. It is the final part of the same decision process.


## Parallel Root Search

Parallelism is applied at the root from depth three onward. Each root move can be searched on a cloned board, so the tasks do not mutate the live JavaFX board or one another's board state. Nitsa first searches the most promising root move synchronously. That establishes a useful alpha value. The remaining root moves can then run in the dedicated search pool. Each one uses Negamax, Alpha-Beta, PVS, quiescence, and the shared transposition table as usual.

I deliberately kept the deeper recursive search single-threaded inside each root task. Parallel tree search becomes complicated quickly because workers need to share bounds and useful work without spending all their time coordinating. Root parallelism is simpler, fits the existing architecture, and uses multiple cores without turning the code into a research paper.


## A More Positional Evaluation

The evaluation function also became more structured. Nitsa uses bitboards, where a 64-bit value represents a set of squares. Counting attacks or finding pieces on open files then becomes a collection of bit operations instead of repeatedly walking visual board objects.

The score combines relative material values, piece-square tables, and mobility. It adds the bishop-pair bonus and evaluates doubled pawns, isolated pawns, and pawn islands. King safety considers missing shield pawns and whether the king has castled during the opening. Other terms reward center control, rooks on open or half-open files, knight outposts, and normal development. The king uses different piece-square tables for the middlegame and endgame, because in the end game it becomes a useful attacker! Hiding in the corner is what we do while queens and rooks are active, but in an endgame the king is a valuable attacker!
All values are combined into a centipawn score from the perspective of the side being searched. A value of +100 is approximately one pawn of advantage. It is still a hand-tuned evaluation, not a neural network, and tuning it remains the part where changing one constant can make Nitsa greater or terrible.


## Making the Search Measurable

A performance improvement is difficult to trust if the only measurement is "the move felt faster." The desktop engine now records the completed depth, main-search nodes, quiescence nodes, cutoffs, transposition-table hits, aspiration re-searches, elapsed time, and nodes per second. The Engine Test window can search the current position, a custom FEN, a benchmark position, or a mate test. It accepts both a target depth and a time limit, then reports the best move, evaluation, and search statistics. This gives me a repeatable way to compare engine changes instead of playing five games and trusting my mood.
This addition helped me actually progress the positional evaluation of the engine because I could use existing positions with known replies. So every test case could re-run after tweaking or even magor changes to understand if anything broke. Git reset was an ally here!


## Desktop Functionality

The JavaFX application now has three main board modes. Live mode lets the user play against Nitsa. Replay mode loads and navigates complete games without triggering an engine response. Analysis mode runs a deterministic search on a snapshot of the current position and displays the evaluation in pawns and centipawns together with the principal variation. Unlike live play, analysis does not replace the first search with a weighted opening choice, because an analysis score is not very useful if the engine quietly skipped the analysis. The search never receives the live UI board, and the result is only displayed if the board still has the same FEN when the calculation finishes. Otherwise a slow result from the previous position could overwrite the analysis of the new one. 

>Computers are fast, but race conditions are faster.

The board supports normal moves, captures, castling, en passant, promotion, notation, move highlighting, and dynamic resizing. Live games now finish properly on checkmate, stalemate, repetition, the fifty-move rule, or insufficient material.

The PGN database view loads files asynchronously and streams parsed games to JavaFX in batches. The UI remains responsive, loading can be cancelled, malformed games are skipped.

Settings are stored locally as JSON through Jackson. They include the engine depth, AI side, worker count, and optional Stockfish executable path. Stockfish can be selected manually or left to auto-detection.

## Stockfish Integration

I also added a `StockfishEngine` service that communicates with Stockfish through the [UCI protocol](https://www.chessprogramming.org/UCI). The Java application starts the Stockfish process, sends positions and time controls, listens for `info` messages, validates the returned `bestmove`, and closes the process when the match ends.

The match window can configure the Stockfish skill level, Nitsa's depth, Stockfish's time per move, and which color Nitsa plays. During the game it shows the board, notation, both engines' evaluations, nodes, nodes per second, depth, and principal variations. Matches can also be paused or stopped without leaving an orphan Stockfish process quietly thinking forever in the background.

A single engine match is entertaining but not very scientific. But it is a actually a way to measure Nitsa's performance and track it over time. White has an advantage, openings have different characteristics, and one tactical blunder can dominate the result. For more repeatable comparisons, I added paired match tests. Each fixed opening is played twice, with the colors reversed. The test records wins, draws, losses, score rate, an estimated Elo difference. Results can be exported to CSV for later comparison.

This does not magically produce an official rating but only a controlled benchmark between a specific Nitsa build and a specific Stockfish configuration. That is still far more useful than announcing that the engine became stronger because it beat me after I blundered a rook.

## How Everything Works Together

The individual algorithms are useful, but their real strength comes from cooperation. Iterative deepening supplies a likely best move and expected score. The aspiration window narrows the next search. The transposition table and ordering heuristics put strong moves first. Good ordering makes PVS and Alpha-Beta cut more branches. Fewer branches allow a deeper search. Quiescence keeps the leaf evaluation tactically stable. Bitboards make that evaluation cheaper. Parallel root search spreads independent candidates across the available cores. Statistics then tell me whether any of this actually helped.

That is the main lesson from building Nitsa. A chess engine is not one brilliant algorithm. It is a stack of small ideas that make the next idea more effective. Remove one layer and the others still work, but usually with much more effort and much less chess.

## Final Thoughts

The third article ended with Nitsa's opening support and some controlled randomness. Since then, the project grew into a complete desktop application with a deeper search pipeline, analysis output, time control, cancellation, engine benchmarks, PGN replay, and automated Stockfish matches.

Nitsa is obviously not Stockfish, and that was never the target. I built it to understand the algorithms by implementing them, breaking them, measuring them, and eventually understanding why the engine had once sacrificed a queen for no reason.

You can see the complete application, video, and project overview on the [Nitsa Chess Engine project page](/projects/nitsa-chess-engine).

## Video Demo

Below is the desktop application running with its analysis and engine tools.

[youtube:gkZk7t6WAqk]

## References

- [Chess Programming Wiki - Iterative Deepening](https://www.chessprogramming.org/Iterative_Deepening)
- [Chess Programming Wiki - Aspiration Windows](https://www.chessprogramming.org/Aspiration_Windows)
- [Chess Programming Wiki - Principal Variation Search](https://www.chessprogramming.org/Principal_Variation_Search)
- [Chess Programming Wiki - Transposition Table](https://www.chessprogramming.org/Transposition_Table)
- [Chess Programming Wiki - Quiescence Search](https://www.chessprogramming.org/Quiescence_Search)
- [Chess Programming Wiki - UCI](https://www.chessprogramming.org/UCI)
- [JavaFX](https://openjfx.io/)
- [chesslib](https://github.com/bhlangonijr/chesslib)
