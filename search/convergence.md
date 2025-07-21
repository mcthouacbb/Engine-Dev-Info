A fundamental idea regarding search is that you want to relax pruning conditions and reduction factors at high depth.
This allows for low depth searches to be optimized to be extremely fast, while allowing higher depth searches to search more thoroughly.
Due to the nature of iterative deepening, a low depth search likely becomes a high depth search at higher root depths, so a low depth search which misses something has a chance to catch it at a higher depth
