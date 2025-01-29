2nd

Thanks for the organizers and congratulations to @reCurse, @aCat, and @marwar22.

The Winter term including holidays was very inconvenient, as usual.
The game was interesting and the engine was well defined (e.g., without arbitrary pathfinding or dependency on double arithmetic).
Although it seemed too simple for a longer contest (deterministic, perfect information, flat and quite small grid, predictable opponent), which makes ordinary solutions work well.

### Algo

Nested beam search:
The main beam search is over full turns and is very standard.
However, action sets for a turn are computed in independent beam searches with a very small width (2 or 3).
The depth of the main beam is unlimited but in most cases, it runs out of time between depth 2 and 8.

For the opponent or when time is running up, the beam is replaced with a much chaper fast greedy choice: It evaluates each action separately, sort them, and then considers them in that order to build an action set.

The opponent prediction in the first turn consists of calculating three scenarios (potentially three different opponent's action sets).
Then, own actions are applied together with these opponent's choices.
For the eval of own actions, a weighted average is used, but for the next state for the beam, the worst case is used.

### Eval

The eval is the heaviest part, with many but standard ingredients:
Voronoi balance, filled cells, separate values for certain more relevant cells based on the shortest paths, distances to the resources, progressive evaluation of income and stored resources, threats including the size of the possibly killed subtree.

### Action generator

Generating possible actions suffers with a lot of pruning.
Harvesters, Sporers, and Tentacles are made only under certain conditions unless Basic is not affordable.

### Weaknesses

I simply did not have time to finish everything nor to try all promising ideas. Partly because of the Winter term, but also because of some mistakes.

* I did not optimize with bits. Instead, I started with compressed state representation and heavy eval. Therefore, calculating eval is the most expensive part and I am doing only about 9000 in the middle of a play. Bit representation could change that but requires a different and simpler eval and a different action generator. I spent too much time optimizing the algorithm itself and the action generator.

* I did not overcome timeouts and started to testing them on CG too late. Sometimes a single eval took about 10ms, maybe due to memory issues, thus not fixable without a big redesign.

* I did not use map analysis (identifying key resources, map type) nor used the initial 1s reasonably.


For some interested in what resource is most valuable:
I use C >> B > A > D for stored resources and C > B >> D > A for income.
(>> means a much larger weight.)
But these weights are independent on the map, unfortunately.

Traditionally, I attach results showing the potential of speed optimization (99% CI):
```
MSz[ 25% time] vs MSz:  28.61%  ±1.2
MSz[ 50% time] vs MSz:  41.44%  ±1.3
MSz[150% time] vs MSz:  54.22%  ±1.3
MSz[200% time] vs MSz:  56.07%  ±1.3
MSz[400% time] vs MSz:  59.16%  ±1.2
```
