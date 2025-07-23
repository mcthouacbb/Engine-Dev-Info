The majority of testing is done to verify that a functional change(patch) gains elo in a chess engine. Sometimes, a test is done to verify a change doesn't lose elo. The method used to test changes used is very important, and the correct way to test changes has been standardized by engine developers over decades.
Common beginner traps include
- Targeting poor measures of playing strength, such as TTD/NTD
- Testing with any kind of test suite
- Not using a large enough sample size with self-play games
- Using a very long time control that makes it difficult/impossible to get sufficient sample sizes
- Playing a fixed number of games rather than using SPRT
- A poor choice of an opening book(or no opening book)

The standard way to test changes is to run self-play SPRTs. This involves taking the engine with the change and the engine without the change and playing them against each other for hundreds or thousands of games, until a statistical conclusion can be made about the elo difference.
Games are typically run at STC(8+0.08 or 10+0.1). The games are played out from specific openings obtained from an opening book. The engines have no say in the opening selection for testing, the match runner selects a random opening and the engines are to play out from there.

# Why Elo
A natural consideration is why elo is used as a metric rather than something like TTD.

# Time control
It's important to understand why testing is done at STC.
### Why not fixed depth
The argument against fixed depth is similar to the argument against using ttd/ntd as the metric. Depth is really not a strong way to control how much time engines use, and you can easily manipulate the depth to gain elo at fixed depth, regardless of if it would gain at an actual time control
With fixed depth testing, almost any form of reductions/pruning is sure to lose elo, as it just searches less nodes with no benefit. At a real time control, the less nodes spent in one depth are possibly made up for by searching to higher depths overall, and thus spending more time on useful parts of the search tree.
Similarly, almost any form of extensions is sure to gain elo, as it just searches more nodes with no downsides. At a real time control, the extensions could take away from time spent in other parts of the search tree.
### Why not fixed nodes
Testing at fixed nodes is not entirely unreasonable. In fact, in some cases it may make sense to test at fixed nodes(e.g. measuring elo without considering speed).
However, fixed nodes does not allow for any sort of time management to come into play, and fixed nodes time controls are usually much shorter than STC. For this reason, it is considered a bad idea to only test at fixed nodes in most cases
### Why not fixed time
The argument is the same against fixed nodes here.

### Why such a short time control
A longer time control would actually be more ideal, as you almost always want to optimize for the longest time control possible.
The issue is that, testing takes a lot of games, and that takes a lot of time. If uou use a very long time control, you will never be able to reach statistically significant results in a reasonable amount of time.
The specific choice of STC is arbitrary and the effective time control also depends on hardware. Sometimes LTC is used to optimize for longer time controls.

### Why increment
Testing at sudden death(0 increment) generally makes no sense for chess engines. If an engine is losing on time the test result is considered to be useless.
The specific choice of base time / 100 is something which the stockfish team measured to maximize elo difference(which makes tests pass faster)

# Why self-play
It is not clear why self-play elo translates so well to play against other opponents. One would think that optimizing against the previous version of an engine would lead to issues with not improving or regressing against other engines. But for whatever reason, self-play elo does translate against other opponents. This is great for engine developers, because self-play elo is less computationally expensive to measure than measuring elo before and after a change against other engines.

# What is SPRT
(Note that this is simplifying things a bit)
SPRT is a statistical test that will automatically stop when reaching statistical significance. It is provably optimal in that it takes the least number of samples on average to prove a hypothesis with statistical significance.

# Why opening books
When running self-play tests, it is crucial to use opening books. Essentially, opening books are used to add variety to the games that the engines play. 
Consider the case where testing is only done from the starting position of chess, and that the engines are playing the same few games over and over. Then, even if we play thousands of games, we really only have a few statistical samples that we are using to draw conclusions. It could also be viewed from the perspective that every sample is highly correlated to the rest and the samples are not independent. 
This logic does not apply if we use a high quality opening book with lots of positions.

This can also be viewed from the perspective of optimizing for certain types of positions. If the engine is always playing from a few opening position, it will only optimize for those openings, and it's not guaranteed that it is actually improving at chess in general.

# Game Pairs
It is important to use game pairs when doing any kind of match between engines. That is, each engine should play the opening from both white and black. There are a few reasons for this
1. Most openings are not perfectly balanced. If an engine gets lucky and plas the advantaged side of an opening slightly more often, it may win more often than it should. Game pairs prevent this, as the engine now also has to play the disadvantaged side
2. Game pairs actually reduce the variance when measuring elo, so SPRT tests can be run faster and error bars on elo are smaller
