The majority of testing is done to verify that a functional change(patch) gains elo in a chess engine. Sometimes, a test is done to verify a change doesn't lose elo.
Common beginner traps include
- Targeting poor measures of playing strength, such as ttd/ntd
- Testing with any kind of test suite
- Not using a large enough sample size with self-play games
- Using a very long time control that makes it difficult ro get sufficient sample sizes
- Playing a fixed number of games rather than using SPRT
- A poor choice of an opening book(or no opening book)

The standard way to test changes is to run self-play SPRTs. This involves taking the engine with the change and the engine without the change and playing them against each other for hundreds or thousands of games, until a statistical conclusion can be made about the elo difference.
Games are typically run at STC(8+0.08 or 10+0.1). The games are played out from specific openings obtained from an opening book. The engines have no say in the opening selection for testing, the match runner selects a random opening and the engines are to play out from there.

# Why Elo
A natural consideration is why elo is used as a metric rather than something like time to depth.

# Time control
It's important to understand why testing is done at STC.
### Why not fixed depth
The argument against fixed depth is similar to the argument against using ttd/ntd as the metric. Depth is really not a strong way to control how much time engines use, and you can easily manipulate the depth to gain elo at fixed depth, regardless of if it would gain at an actual time control
With fixed depth testing, almost any form of reductions/pruning is sure to lose elo, as it just searches less nodes with no benefit. At a real time control, the less nodes spent in one depth are possibly made up for by searching to higher depths overall, and thus spending more time on useful parts of the search tree.
Similarly, almost any form of extensions is sure to gain elo, as it just searches more nodes with no downsides. At a real time control, the extensions could take away from time spent in other parts of the search tree.
### Why not fixed nodes
### Why not fixed time

### Why such a short time control
A longer time control would actually be more ideal, as you almost always want to optimize for the longest time control possible.
The issue is that, testing takes a lot of games, and that takes a lot of time. If uou use a very long time control, you will never be able to reach statistically significant results in a reasonable amount of time.
The specific choice of STC is arbitrary and the effective time control also depends on hardware. Sometimes LTC is used to optimize for longer time controls.

### Why increment
Testing at sudden death(0 increment) generally makes no sense for chess engines. If an engine is losing on time the test result is considered to be useless.
The specific choice of base time / 100 is something which I believed the stockfish team measured to maximize elo difference(which makes tests pass faster)

# Why self-play

# What is SPRT
(Note that this is simplifying things a bit)
SPRT is a statistical test that will automatically stop when reaching statistical significance. It is provably optimal in that it takes the least number of samples on average to prove a hypothesis with statistical significance.

# Why opening books
