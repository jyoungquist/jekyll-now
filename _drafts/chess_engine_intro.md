---
layout: page
title: Chess Engine Basics
---

The first time it was thought that computers could one day play chess was in 1945 when Alan Turing used chess as an example of what a computer could be able to do.
Fifty-two years later his idea had advanced so far that the chess engine Deep Blue had defeated Garry Kasparov.

The goal of this series to build an entry-level chess engine.
I will use techniques that have a good balance of conceptual simplicity and speed, the goal being to produce a minimal working chess engine.
Once that is in place, a more sophisticated engine can be built around it, adding in techniques such as transposition tables and iterative deepening.

Every chess engine has three main components that it needs to play a game of chess:

## Board Representation & Move Generation

At the core of every chess engine is the underlying representation of the board.
It is responsible for storing the current game state and should be able to generate every legal move from a given position, answe queries relevant to position evaluation, and be light weight so it can be copied without requiring an excessive amount of memory.

The data structure that I am choosing to use is the [bitboard](https://chessprogramming.wikispaces.com/Bitboards).
For each piece type (white rook, white knight, white bishop, white king, white queen, white pawn, and the corresponding black pieces), the bitboard stores a 64-bit integer.
The 64-bit integer corresponds to the 8x8 positions on a chess board, with a 1 for an occupied square and a 0 for an unoccupied square.
Hence, there are 12 64-bit integers that represent the current position, plus some extra information for castling and en passant, meaning they are very space efficient.

Furthermore, most queries that need to be done can be performed by various [bit twiddling hacks](http://graphics.stanford.edu/~seander/bithacks.html) which take advantage of parallelism present in the processor.
Move generation can be complicated, especially for sliding pieces such as rooks and bishops, so these methods will be covered in a later post, in addition to a more detailed analysis of bitboards vs other representations.

## Evaluation

[Evaluation](https://chessprogramming.wikispaces.com/Evaluation) is the process of looking at a given board position and determining which side is better and by how much.
It is extremely important to have a fast and accurate method of evaluation because this will be called repeatedly throughout searching and will be the basis on which a move is chosen from a set of possibilities.

Material advantage is the easiest aspect to test and is also one of the most significant determiners of which side has the advantage.
After that, the position of the pieces is evaluated easily, typically via a mask of some sort.
A 2x2 array stores values for each square with higher values corresponding to better locations for the pieces.
For example a knight is typically better in the center of the board than on the edges, and a 7th rank white rook is advantageous.
This lookup table is then used to tune the score comparison.

The remaining aspects are more complicated.
Pinned pieces, pawn structure, control of a file or diagonal, king mobility, etc are all important aspects of the balance of the game, yet are chellenging to implement in an efficient and robust manner.
A good resource for this is the code for Stockfish, described [here](https://chess.stackexchange.com/a/19792).

## Search

The final stage is searching for the best move to make.
The board is queried for each possible move, and then the board is evaluated after making each one to determine which one is best.
To have a really good searching algorithm, this must be done to a high depth: after making a move for white, say, you have to then see what black's best response is, and then determine how you would respond, and so on...
This algorithm is called a minimax search and is the standard search method for most chess engines.
To help keep the depth down some, it is typically paired with alpha-beta pruning to avoid searching down paths that will not result in an advantage for the side which is playing, allowing it to search deeper than if it did not prune the search tree.


## Some Resources

Here are some resources that I have found to be useful:

1. [Chess Programming Wiki](https://chessprogramming.wikispaces.com/)
2. [Chess Programming series on gamedev.net](https://www.gamedev.net/articles/programming/artificial-intelligence/chess-programming-part-i-getting-started-r1014)
3. [More technical introduction](http://www.frayn.net/beowulf/theory.html)
4. [Common operations on a bitboard](http://pages.cs.wisc.edu/~psilord/blog/data/chess-pages/physical.html)
5. [Bit twiddling hacks reference](http://graphics.stanford.edu/~seander/bithacks.html)
6. [Perft debugging technique](http://mediocrechess.blogspot.com/2007/01/guide-perft-scores.html)
7. [Intuitive chess engine tutorial in javascript](https://medium.freecodecamp.org/simple-chess-ai-step-by-step-1d55a9266977)
8. [Minimax with Alpha-Beta pruning](https://www.hackerearth.com/blog/artificial-intelligence/minimax-algorithm-alpha-beta-pruning/)
9. [A brief history of computer chess](https://thebestschools.org/magazine/brief-history-of-computer-chess/)

