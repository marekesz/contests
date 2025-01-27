(Post Mortem for [Fall Challenge 2024 -- Selenia City](https://www.codingame.com/contests/fall-challenge-2024) optimization)

1st

I enjoyed the contest and the game was nice, although I prefer bot programming. Mastering the engine was moderately difficult, and the engine was optimizable.

The most troublesome thing was the need to adjust to particular test cases, which, moreover, we could only guess how much they will vary from the public ones. Fine-tuning for tests was a major concern to worry about, i.e., that a totally general agnostic algorithm would not be competitive, and on the other side, that assuming too much would spoil everything. I think it would be better to publish the generator and, for example, say there will be 100 uniformly random seeds or so. This would give the results more fair, not so dependent on untestable assumptions and luck.

The bot has three main ingredients:

* The algorithm is Hill Climbing, split into much-parametrized phases with different action types.
* Heavily optimized exact simulation engine.
* Test-dependent tunings.


### Algorithm

The algorithm is split into phases that differ in parameters and action types. In each phase, it performs each available action and simulates the score obtained in the current month (turn). The eval function depends on this score and the resources left. This is repeated until no improvement can be made, and then the next phase begins. The action types are, for instance, adding a path of a certain thickness (by adding tubes and optionally upgrading) and length together with a pod or two, adding a teleport, replacing a pod, etc. The next phases increase available thickness and the set of available actions.

The problem carries some similarities to the ants' movement optimization from Spring Challenge 2023, where I used a similar algorithm.


### Engine

The efficiency matters a lot on big maps, affecting the number of combinations to check by the HC. The simulation consists of inverse BFS to compute distances for each building type, and then days simulation, which consists of moving pods and astronauts. The nontrivial optimizations are as follows:

* Astronauts move in groups. A group is a set of astronauts of the same type and from the same pad in a continuous interval. On most maps, there are groups larger than one. Sometimes, when a whole group does not fit in one pod, it is split into smaller groups.
 There are no traffic jams. Computing jams considerably slowed the engine (the need to count used tube capacity, etc.), so I decided to disable it: only actions that ensure that pods will not get stuck are applied, thus each added pod comes with suitably new or upgraded tubes.
* I do not sort anything anywhere, always keeping the right order of pods and astronauts.
* When building tubes, they are checked for an intersection only with the tubes currently added in the turn. The other cases are preprocessed before the search.


### Test-dependent tuning

The most important parametrization is the value of left resources, which obviously must vary a lot across the maps. The other fine tunings involve the set of allowed actions in particular phases of HC. I am not too fond of such practice, but there was no other way of including information about the future turns, which were not random either.

The most computational heavy is Test 8 (Grid). For this, the bot brings an additional ingredient, which is computing initial direct tubes greedily without simulation. It was necessary because the number of needed actions is so large, that otherwise the HC does not have time to add them all and spend all resources.

Surprisingly, the general setting without test-specific tunings is not so much worse, getting the score smaller by only about 200k. The general, however, actually means being tuned to the tests giving the largest scores.


### Some results

Here are the results on each of the public tests:

```
Test 1: 53 710
Test 2: 128 830
Test 3: 254 605
Test 4: 261 575
Test 5: 415 110
Test 6: 379 390
Test 7: 739 409
Test 8: 1 393 168
Test 9: 870 763
Test 10: 766 489
Test 11: 933 093
Test 12: 901 280
Total: 7 097 422
```

Example replay with test 8: [https://www.codingame.com/replay/822629219](https://www.codingame.com/replay/822629219) .

Example replay with test 12: [https://www.codingame.com/replay/807001223](https://www.codingame.com/replay/807001223) .

The number of simulations (score computations) for Test 8 (Grid), in the first turn without the mentioned greedy optimization: 98 536 (the average over 5 runs).

And the results on the public validators from the repository, for a version close to the final one (I cannot test them again on CG, since there is no option. And, of course, the results on large tests may differ slightly in each run):

```
Test 13: 95 090
Test 14: 173 600
Test 15: 409 415
Test 16: 400 740
Test 17: 438 416
Test 18: 393 200
Test 19: 727 644
Test 20: 1 501 128
Test 21: 788 257
Test 22: 740 240
Test 23: 934 054
Test 24: 921 655
Total: 7 523 439
```
