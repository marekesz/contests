(Post Mortem for [Spring Challenge 2023 (Ants)](https://www.codingame.com/contests/spring-challenge-2023))

1st

# Overall

I enjoyed the contest and the game. It was very non-trivial, requiring several different algorithms to work together. And still, it is not clear to me what could be an optimal strategy.

**Thanks to the organizers!**

Also, special thanks to @aCat for support and discussions and to the rest of the University of Wrocław group. And of course, thanks to the great opponents!

The overall idea of my bot relies on two phases and is similar to the others. In the first phase, it makes a general project for current path connections (tree), and in the second phase, it runs micromanagement modifying beacon weights. For calculating beacon placement, I did not see an alternative way to enforce the best move than trying many configurations by simulation, so the branching factor is huge. As in the Cultist Wars contest, I tried to make efficiency to be my strength; this time, instead of exploring deeper, the effort was to explore the game tree wider, which just means trying more own moves.

I spent the first days encoding the algorithm from the referee and optimizing it. Additionally, I had to adapt to constantly-changing rules. At that time, without any reasonable AI algorithm, I found it hard to reach even Bronze. But Bronze was required to test the full rules. The situation changed immediately after adding any path-making algorithm with equal beacons. However, as I saw improving the micromanagement phase affects the results much more and actually compensates for the weakness of the high-level design, I neglected the development of the first phase and meta-decisions for too long. Yet, they were also very important when encountering strong mature agents.

There were also some glitches worth noting: The rules are quite difficult (because the movement algorithm is a part of the rules, although undescribed) and they were changing through the first half of the contest. Therefore, implementing the engine early was risky, and indeed I had to redesign it a bit after some changes. While I understand and accept the concept that ants cannot be controlled directly, the algorithm behind it looks artificial, because it affects the ants globally. This globality also makes it difficult to design moves by hand, in favour of simulation. In my option, a more natural way to model ants would be, e.g., when an ant goes to the nearest beacon that it sees, follows other ants if possible.


# Technical details

## The tree phase

The tree is computed in a greedy/Hill Climbing way. The resources (crystals or eggs) are added iteratively, each time evaluating the resulting constructions. The estimation is based on the value of the extracted resources divided by the build cost. The resource is connected to the existing tree with the cheapest path, assuming, e.g., that going through existing ants is cheaper and promoting going through other resources.
This is computed with the standard Dijkstra algorithm. It is not a minimum spanning tree (MST) on resources, because we can connect a new resource directly to a part of an existing path; if we knew what exactly resources we want to connect (but we do not), it would be a variant of the Steiner tree problem.
Instead of adding a new resource, it is also possible to increase the width of an already existing path.
The proposed width (the number of wanted ants) at each cell is also a part of the project, and it takes the opponent's attack chains into account, increasing the width wherever needed along the way up to the base.

The construction is complete when its size exceeds the number of ants, with a slight surplus allowed.

## The micromanagement phase

This is absolutely crucial for the strength of my agent. The variant with the tree only has only about 0.82% win ratio against the full agent. Improving this part was a game-changer several times. The goal of micromanagement is not just moving ants as closest as possible to the given high-level project, but it can extract nearby resources apart from the plan, adjust the width of chains, block the opponent, and maximize the extraction. Actually, it may lead to a trap of ignoring these factors in the first phase and I fell in that for a few days.

Here, the algorithm takes the initial tree project and modifies it in a Hill Climbing manner. The initial project is applied and the algorithm identifies places that might be good to change: where there are fewer or more ants than proposed, not forgetting about nearby resources. There are several iterations, modifying either a pair of beacons or a single beacon (increase or decrease). If time permits, more candidates to try are added. What may be distinguishing from other solutions, I do not try random changes but rather search through all possible single changes exhaustively. And in this phase, I do not keep the constraint relating the number of beacons to the number of ants, allowing exploring the full spectrum of possible moves.

The evaluation uses a simulation of two turns, applying the same move and adding the value after the second turn with a strong decay. The main components are, of course, the score and the number of ants, and also the agreement level with the initial project, as well as the formed attack chains against the opponent. Initially, I was using three turns. However, after the evaluation became more complex, the third turn stopped being profitable. And surprisingly, this is not just because it is more expensive, but the third turn prediction seems already too unreliable and useless. The bot using three turns achieves a win ratio of about 48% against the same bot using two turns when they are both limited to evaluate at most 10 000 moves instead of a time limit. Of course, its situation is worse if we take the time overhead into account (then it achieves only 45%).

Additionally, before the entire phase, a simplified and short micromanagement for predicting the opponent is run, which is slightly better than assuming that it does not move. The predicted move consists of beacons at the places occupied by the opponent's ants and adjacent to them.

### Optimization

I was seeing a big difference in results depending on the time budget, so I paid much attention to optimization, which is also the part that I like the most. The original movement algorithm is implemented in a very inefficient way. I got rid of generating and sorting pairs by computing them on the fly. For this, it was enough to precompute for each cell and each distance, the list of cells at this distance. Also, the list of ant cells is dynamically modified. This still has worst-case quadratic time complexity but works much faster when the beacons are close to ants (usual case) and there are enough beacons to assign the ants shortly.

As to the ant movement, it depends on the cell neighbors ordering, which is affected by the assignment (target cell), other ants (game state), and beacons (move). I sort the neighbors using dedicated sorting networks for 2--6 elements. Under a sensitive measurement, it appeared slightly faster to sort in this way than selecting from unsorted neighbors, which is most likely because we may need to do this several times for the same ant cell. Here, I regret the change in the rules that beacons also affect ant's path selection. Before that change, the neighbors order depended only on the state, so could be precomputed for faster performance.

For computing the attack and harvest chains, I implemented Dial's algorithm (Dijkstra with buckets as the priority queue). The maximum number of ants on one cell is usually small, so it works faster than Dijkstra with the traditional heap. Also note that updating the chains is cheaper, and there are some cases when updating is not needed.

Finally, everything that was possible was precomputed, like resource cells, quantities in different areas, and distances of various types.

How important was the efficiency? See the following comparison of my bot with different time limits (in milliseconds) with itself:

```
   MSz(5 ms) vs MSz(100 ms): 15%
  MSz(10 ms) vs MSz(100 ms): 23%
  MSz(30 ms) vs MSz(100 ms): 36%
  MSz(50 ms) vs MSz(100 ms): 42%
 MSz(200 ms) vs MSz(100 ms): 55%
MSz(1000 ms) vs MSz(100 ms): 60%
```

Conclusion: If it was 3 times slower, it would be of similar strength to @DomiKo bot (based on its the estimated win ratio of 34% after the final recalculation).

## Meta

Designing meta strategies was also quite important. The most important rule is: harvest on the border first (the cells at an equal distance from the players' bases and around), and when there is nothing left, go back to own area. The resources threatened by the opponent but not in its area are also considered a priority. Additionally, when detecting that it will lose because there are not enough crystals, it tries to steal something from the opponent's area; however, when such a situation occurs then it is usually hopeless anyway.

Finally, there were many parameters to tune (e.g., egg values, localization bonuses, path cost, eval parameters, and the values controlling micromanagement). I find that there are more of them compared to other of my agents for other games.

## Not working ideas

It is natural that some improvement attempts do not work, and I also had a lot of them. From those ideas I remember, e.g., hashing the moves to avoid repeated eval computation, trying several paths from one resource, estimating the local density of resources to prefer rich locations, and predicting the game ending after 100 turns to affect meta decisions.
