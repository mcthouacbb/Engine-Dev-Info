Reverse Futility Pruning(RFP) is a really simple yet effective pruning heuristic. 
RFP relies on the null move observation. Specifically, it assumes that the static evaluation of a position is an approximate lower bound on the score of a position. RFP can also be viewed as a generalization of the "stand-pat" idea in quiescence search.

The core idea of RFP is that, if the static evaluation is very far above beta, then it is relatively safe to assume that the node would have failed high. Thus, we can prune the node early without having to search any moves.

RFP should not be done when in check, because
1. The null move observation does not hold when in check
2. Most engines do not have an accurate way of evaluating when in check

RFP should not be done at the root node or in PVS PV nodes.
A PV node generally implies a research of a non-PV node, or the search of the first move. In the former case, we want to avoid pruning so that we can get a score for the first move. In the latter, RFP makes no sense, as the non-PV search must have failed low to cause a research, so a fail high would contradict this. Additionally, PV nodes should usually be allowed to enter the moves loop and starts searching moves, as they are the most critical/important nodes.

RFP should not be done at very high depths, and the margin used for pruning should increase with depth. This is to be more conservative with high depth pruning, i.e. to allow the search to converge.

In it's most basic form, RFP looks like this
```cpp
if (!pvNode && !inCheck && depth <= 6 && staticEval >= beta + 80 * depth)
    return staticEval;
```

