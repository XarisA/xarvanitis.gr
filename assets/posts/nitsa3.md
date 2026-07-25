# Building a Chess Engine pt.3 - Adding Book Support and a Tiny Bit of Randomness

*This is part of [Building my Chess Engine Series](/blog?tag=chess-engine-series)*


Alright, let’s get into it! In this part of the series, I made some big improvements to **Nitsa**.  After the decision-making algorithms and their performance in earlier parts of the project, it was time to make Nitsa more “human” and unpredictable. Also, Nitsa should be able to play different openings, and the deterministic evaluation does not help with this.
To do this I added book support for a better opening play and a little bit of randomness to spice things up.

So let’s take a deeper look.

## Opening Book Support

If you’ve ever played a chess engine in the opening phase, you’ll know that it can sometimes feel like the engine plays the same moves over and over. The truth is, that most engines do not rely on hard calculations to figure out what to do in the initial positions regarding opening phase. This would be slow and not the most efficient way to go. 

It was time to add an opening book.

**What’s an opening book?** Well, it’s basically a pre-loaded database of strong opening moves that have been played by chess players for years. Instead of Nitsa reinventing the wheel every time it faces a common opening, I’ve hooked it up to an opening book that will let her play known opening moves without having to do extensive calculations.

There are generally two types of opening book formats that the engines usually go with, the binary and the PGN (Portable Game Notation). PGN books are simple and human-readable and it's basically text based format. Reading a PGN would require parsing the file and interpreting each move during runtime. On the other hand, binary books are most of the times faster to load and access the data (if the implementation has the correct approach).
I have chosen to go with the binary book and after some search I have used the [komodo polyglot book](https://komodochess.com/downloads.htm).

Here are the points of how I made it work:

### Zobrist Hashing: Fire in the hole

I used **Zobrist Hashing** to efficiently convert board positions into unique hash values that can be quickly searched in the opening book. Zobrist hashing is a standard practice in chess engines for representing positions compactly. Instead of storing the entire board object, each piece on the board contributes to a **64-bit hash** by XOR-ing a precomputed random number associated with its piece type and square. The result is a "unique"  identifier for every possible board position.

Once I had the hash for the current position, I scanned the entire book, looking for a match. The book entries are stored as key-value pairs where:
- The **key** is the Zobrist hash of a position.  
- The **value** includes the recommended move and its **weight** (which is the move frequency in the games of the database).


### Implement Move Variety: Add some Randomness

Initially, I simply selected the move with the highest weight, but this led to repetitive games. To introduce variety, I have added the moves with the top weights in a set and then I picked a random move from the **set of best moves**. This ensures that the engine still plays strong openings but doesn’t always repeat the same sequence of moves, making it feel more human-like and less deterministic. After all, there are thousands of opening variations and it would be a pity to play the same each time 😉.

### Binary Search Algorithm: Optimizing Search Speed

At first, scanning the entire book linearly was **too slow**, especially for a large book. To speed up the process, I implemented a **binary search** algorithm on the sorted book entries.

Binary search is an efficient way to find an element in a sorted array. Instead of checking every entry, it repeatedly divides the search space in half, checking the middle element and discarding half the remaining possibilities. This reduces lookup time from **O(n) (linear search)** to **O(log n)**, which is way much faster and is considered a great optimization.

The binary search was great but a potential improvement would have been to use a **parallel binary search**, splitting the book into multiple chunks and searching them simultaneously using multithreading. However, given that binary search already performed well, I decided it wasn’t necessary. The lookup time was already reduced significantly, and adding concurrency would introduce complexity without major benefits add the current stage.


## Less Predictable Nitsa

By that time Nitsa had been playing well and using the book moves with the **random factor** created a great game experience. So this made me think that randomness is probably something that I also need to use in my evaluation function. Till now **Nitsa** would peak the best possible move according to her evaluation function, but there might be also other moves with the same score. Till now it would go with the first one.
Well, I thought to use this randomness in the best moves array inside the evaluation function. 

- When Nitsa is at positions where multiple moves have the same strength, I included into the decision process a degree of **randomness** so it doesn't just make the same choice every time. The engine picks at random between those equally good moves, though all might have the same evaluation score.
- A very **slight** randomness. It won't be like: the engine suddenly starts making blunders or bad moves. This is just to make play a little less "robotic" and more dynamic. 


## Conclusion

Now that there's added book support and some move variety, the engine is finally over 🎉🎉. I know that the engine would also need major enhancements regarding the strategic understanding of the position but I don't fill that I need to go that deep. It's not that I will have Nitsa compete in a tournament or something.
I developed this project for fun and in order to implement, thus have a better understanding, of the AI algorithms.

So, I think it's time for some Engine matches! 

### Next Up: Analytics - Engine Strength

The next step will be having Nitsa play against another engine **Stockfish**, and some of the chess.com bots. I've already done more than a few playtests during the different development stages. I already trust her performance, and I know how good it performs, but now is the time to gather some data. After all, I suppose we all like plots!

## References

- [komodochess](https://komodochess.com/downloads.htm)
- [chessprogramming.org - Zobrist_Hashing](https://www.chessprogramming.org/Zobrist_Hashing)
- [stockfish on github](https://github.com/peterosterlund2/droidfish/blob/master/DroidFishApp/src/main/java/org/petero/droidfish/book/PolyglotBook.java)
