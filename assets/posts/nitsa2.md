# Building My Chess Engine pt.2 - Building the Chess Engine

In the second part of my journey to build Nitsa, I dive deeper into the algorithms and optimizations that power 
the engine's decision-making. From core techniques like the Negamax algorithm to more advanced optimizations 
such as Alpha-Beta pruning and move sorting. Each step plays a very important role in how Nitsa evaluates 
and plays chess. The goal was to make the engine not just intelligent but also efficient, 
able to process complex positions quickly while maintaining a strong tactical and positional understanding.

Let’s explore how these techniques come together to form Nitsa’s core.


## Negamax Algorithm: The Backbone of Move Search

At the core of Nitsa’s decision-making is the **Negamax algorithm**, a variant of the minimax algorithm that's more suited for two-player zero-sum games like chess. The basic idea behind Negamax is to evaluate all possible moves from the current position, apply a recursive depth-first search to simulate future moves, and then to return the best move along with it's score. 
Negamax algorithm eliminates the need to write separate code for maximizing (or not) players by using a cool mathematical trick: negating the scores when switching between players.

Here’s how it works in Nitsa:

1. **Search Depth**: The engine explores the game tree to a specified depth, where each node represents a chess position.
2. **Score Evaluation**: After evaluating all possible moves, it assigns scores based on how favorable the position is for the player.
3. **Negation**: When switching turns between White and Black, the engine negates the score of the position, turning the maximization problem into a minimization one.
4. **Recursive Search**: It recursively evaluates future moves until the search depth reaches zero or a game-ending condition like checkmate occurs.

This is the core algorithm but it is not enough. After I implemented negamax the engine was realy slow to play a move and thus it required optimization.
One core optimization technique when we are using decision algorithms like MiniMax and Negamax is to use pruning in order to reduce the engines effort and make computation more optimized.
I did this by using the Alpha-Beta pruning technique.


## Alpha-Beta Pruning: Efficient Search Optimization

Negamax is computationally expensive, as it needs to evaluate a large number of possible positions (nodes). By using **Alpha-Beta pruning**, I improved the search efficiency by pruning away branches that won’t impact the final decision. This means fewer nodes to evaluate, saving both time and computational resources.

- **Alpha** represents the best score achievable for the maximizing player.
- **Beta** represents the best score achievable for the minimizing player.

When Nitsa explores the tree, if it finds a move that is worse than a previously examined move, it **prunes** (cuts off) that entire branch from further consideration. This dramatically reduces the number of positions the engine needs to evaluate, making the search faster without sacrificing accuracy.

There was already a huge increase in performance but we are not even close to how the engine should behave. To further optimized the search algorithm one common approach is to sort the grpah tree with the most important moves. If the best moves are searched first, pruning becomes much more efficient, as inferior moves can be discarded early. 

To do this a created a sorting algorithm based on several heuristics and forced the engine to sort the legal moves in each recursion.


## Move Ordering and Sorting: Prioritizing the Best Moves

To speed up the engine's search further, I optimized the way moves are ordered and evaluated. **Move ordering** is crucial for Alpha-Beta pruning to be effective. 

The move-sorting technique I developed includes the following:

- **Checkmates**: Moves that checkmate the opponent are evaluated first. If a checkmate is found the following operations stop.
- **Captures**: Prioritize capturing moves, especially those where a valuable piece is taken. I did this by using a well known heuristic called **MVV-LVA (Most Valuable Victim – Least Valuable Attacker)**

This heuristic orders captures based on the value of the captured piece versus the piece making the capture, aiming to maximize gain while minimizing loss.

- **Promotions**: Moves that result in the promotion of a pawn are also prioritized.
- **King Checks**: Moves that check the opponent’s king are evaluated early because they result in a gain of tempo.
 
By sorting moves in this way, I ensured that the engine searches the most promising moves first, making Alpha-Beta pruning much more effective.



## MVV-LVA Heuristic: Maximizing Gains in Captures

One of the key techniques used to optimize move sorting in my chess engine is the **MVV-LVA** heuristic. This heuristic helps prioritize which capturing moves should be evaluated first, especially in tactical situations.

In chess, not all captures are equal. Some moves capture more valuable pieces (victims) using less valuable pieces (attackers), leading to greater material gain which is one core requirement. 
The MVV-LVA heuristic orders capturing moves by prioritizing those where:

- The **victim** (the piece being captured) is the most valuable.
- The **attacker** (the piece doing the capturing) is the least valuable.

This means, for example, capturing a queen with a pawn  will be evaluated before capturing a knight with a rook. This prioritization helps the engine quickly find the best tactical moves that win the most material with the least risk, which is crucial for both mid-game attacks and endgame scenarios.

This technique ensures that the most favorable trades are considered early in the search process, improving further the efficiency of **Alpha-Beta pruning**. It helps the engine quickly find the moves that lead to material advantage, making it more tactically sharp in positions where captures are critical.

After adding the sorting heuristic the engine performed even better and much more faster than before, but still played some weird moves. After analyzing the plan that engine used I understood that the problem was related to the depth of the search algorithm. The engine could propose a move that could lead to a capture to the next move when the depth was reached. This problem is called the  Horizon Effect and to bypass it I used the Quiescence search.


## Quiescence Search: Avoiding the Horizon Effect

One major issue in traditional Negamax search is the **horizon effect**, where the engine prematurely stops evaluating a position, missing important tactical moves like captures or checks that occur just beyond the search depth. To solve this, I implemented **quiescence search**.

Quiescence search extends the search for a few more moves but focuses only on "noisy" moves (typically captures, promotions, or checks). This avoids the problem where the engine overlooks critical tactics and produces more accurate evaluations. So even if the defined depth is reached if the position results to an important move like the ones mentioned before the engine continues evaluation.


## Evaluation Function: Combining Material, Mobility, and Positional Insights

In the development of Nitsa, I’ve used a custom evaluation function that combines several key aspects of chess like **material**, **mobility**, and **positional** evaluation. This function forms the backbone of the engine’s ability to assess a board state and determine whether it is favorable for one side or the other.

Here’s a breakdown of the components used in this evaluation function:

### 1. **Material Score**

The **material score** evaluates the total value of pieces each player has on the board. Each piece is assigned a standard value (in centipawns) based on its relative strength:
This component ensures that the engine understands the raw material strength of a position—e.g., being up a queen or down a rook is a significant advantage or disadvantage.

The engine accumulates the value of all pieces on the board, adding their centipawn values to the total score.

### 2. **Mobility Score**

The **mobility score** represents how many legal moves a player can make. Greater mobility usually indicates more control over the board and tactical flexibility. It's a standard technique to penalize positions where a player's pieces are too restricted or to reward open positions where a player has many move options because that's how chess works.

- If a player has many active pieces with multiple legal moves, their mobility score would increase.
- If the opponent’s pieces are more restricted, their mobility score would decrease.

### 3. **Positional Score**

The **positional score** is a finer measure that looks beyond material and asks how well-placed the pieces are. Certain positions, such as controlling the center or placing a knight on an outpost, can be more advantageous than simply having material superiority.

In this engine, the **positional score** is calculated based on the coordinates of each piece. More specifically each piece type has a table of positional values that prioritize certain squares. For example, knights are stronger in the center, while rooks excel on seventh rank. For each piece an array with positional scores has been created and the engine checks the score for each move and adds it to the total (but this could be negative also).

This is only a basic positional evaluation and there many more things that the engine should consider to achieve human's Strategy like:
Passed Pawns, Advanced Pawns, Bishop Pair, Doubled Pawns, Isolated Pawns, Pawn Structure, Bad Bishop, Rook on Open or Semi-Open Files, King Safety, Control of the Center, Piece Coordination, Tempo, Outposts and  Space Advantage.

But this is something that I will try to achieve in the next part of the series. 

### 4. **Game Outcome Scoring**

The function handles game-ending scenarios as well. If the side to move is **checkmated**, the engine assigns an extremely high negative score for the losing side and a very high positive score for the winning side:

This ensures that the engine recognizes immediate wins or losses without needing further search.

### 5. **Final Evaluation**

The final score combines all these factors, **material**, **positional**, and **mobility** into a single evaluation. The score is also multiplied by `whoToMove`, meaning:
- A positive score indicates a favorable position for White.
- A negative score indicates a favorable position for Black.

The result is scaled down (divided by 100.0) to make the scores more human regarding pawns


## Why This Matters

The strength of a chess engine is determined by how accurately it can evaluate a position. By combining **material**, **mobility**, and **positional** factors, this evaluation function 
gives Nitsa a well-rounded understanding of the game state, allowing it to make more informed decisions. Over time, this evaluation can be further fine-tuned as the engine grows more sophisticated.

In summary, this multi-dimensional evaluation function provides the engine with a balance between raw material counting and more nuanced aspects of chess, like piece activity and board control.

The engine now seems to play really great chess! but also really slow. So it's time for multithreading!


## Thread Pool: Parallelizing Move Search

To handle the heavy computational load of evaluating many possible moves, I implemented **multithreading** using a **ThreadPoolExecutor**. Each possible move is evaluated in parallel, 
allowing Nitsa to take advantage of modern multi-core processors and significantly speed up the search process.

For every legal move, a separate thread is dispatched to evaluate the resulting position using the Negamax algorithm. 
Once all the threads finish, the engine collects the results and chooses the move with the best evaluation.

By doing this the engines evalation time was reduced by 40-50 percent. This was a great win but was not enough. I had to do something more.. so I continue searching the chessprogramming wiki and found 
Zobrist Hashing! 

## Transposition Table: Caching Results for Repeated Positions

To further improve performance, Nitsa uses a **transposition table**. This is a cache that stores the evaluation of previously examined positions, keyed by a unique **Zobrist hash** for each board state. 
If the engine is about to evaluate the same position again during the search, it can skip recalculating the evaluation and retrieve the result from the cache.

This is especially useful in chess, where certain positions can arise multiple times in different move orders (a phenomenon called transposition).


## Conclusion

In this second part of the series, we’ve covered the critical algorithms and optimization techniques behind the Nitsa chess engine. From **Negamax** and **Alpha-Beta pruning** to **quiescence search**, 
**move sorting**, and **parallelization**, these techniques work together to allow Nitsa to think several moves ahead and play efficiently.

This was an exciting part of the journey in developing Nitsa, and I can’t wait to share the next steps as the engine grows even smarter. There are still so much to do.

In the next step I will try to make Nitsa play more strategicaly by tuning the evaluation function and by incorporating specific chess knowledge.

Closing this post with a quote from Savielly Tartakower:

>Tactics is to know what to do when there is something to do, and strategy is to know what to do when there is nothing to do.


## References

- [chessprogramming.org - Negamax](https://www.chessprogramming.org/Negamax)
- [chessprogramming.org - Zobrist_Hashing](https://www.chessprogramming.org/Zobrist_Hashing)
- [chessprogramming.org - Alpha-Beta](https://www.chessprogramming.org/Alpha-Beta)
- [chessprogramming.org - Quiescence](https://www.chessprogramming.org/Quiescence_Search)
- [chessprogramming.org - MVV-LVA](https://www.chessprogramming.org/MVV-LVA)
- [chessprogramming.org - SEF](https://www.chessprogramming.org/Simplified_Evaluation_Function)
- [chessprogramming.org - LL Chess](https://www.chessprogramming.org/LL_Chess#PointValues)
- [chessprogramming.org - Endgame](https://www.chessprogramming.org/Endgame)
- [cornell university - how stockfish mastered chess](https://blogs.cornell.edu/info2040/2022/09/30/game-theory-how-stockfish-mastered-chess/)
- [frayn.net - chess theory](https://www.frayn.net/beowulf/theory.html)
