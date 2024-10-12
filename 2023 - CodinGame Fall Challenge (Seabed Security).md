(Post Mortem for [Fall Challenge 2023 (Seabed Security)](https://www.codingame.com/contests/fall-challenge-2023))

2nd.

Thanks organizers for the contest and competitors for high level!

I have a one-turn simulation with samples, iterative improvement with opponent prediction, backed by a pixel-perfect engine. The ingredients are as follows:

1. **Inferer** provides information on where fishes and monsters can be and which ones we know exactly. It operates on geometric figures: rectangles, circles, and rounded rectangles. Rectangles come from radar, positive circles from opponent's scans, and negative ones from not seeing. The inference is performed backwards (to update previous turns based on the current one) and forward (from the past up to now). When an item goes out of sight, it is kept exactly as long as no unknown collision can affect it. All symmetries are taken into account, and the rectangles grow according to the possible speed of the tracked item. For determining if a position is possible, information from the current and past turns is used. Nevertheless, updating the whole thing uses less than 1 ms per turn.

2. **Samples** are a mechanism to somehow capture the probability distribution of positions and vectors. I keep a few hundred states. Initialized at random exactly as in the referee, they are fixed whenever they are refuted by inferenced knowledge. In that case, a fish or a monster is replaced with a new random one. Since the eval on multiple samples is costly, I sort the current samples by quality (the number and time of fixes) and use only a few best ones for evaluation. In this way, the best samples hopefully contain positions and vectors most similar to the real ones.

3. **Eval** contains a few components. There are, e.g., win, draw, loss values, score if both players go up now, current scans value, the estimated number of turns to scan and to kill a fish, and a penalty for using a lamp. Going up is simulated up to the end by a greedy selection of moves, avoiding monsters and including scanned fishes on the way. For the estimated numbers of turns to scan or kill, one turn simulation for monsters is used, then distance. Finally, to obtain the eval value, these components are mixed together without any sense. The eval is crappy and inconsistent, duplicating some features in other ways, and it works differently for the opponent just because it was worse otherwise.
I did not find any real improvement from avoiding monsters other than not picking directions that lead to being eaten. Maybe except for a simple small penalty for having a close monster.

4. **Move search**. I check maximal vectors around, and halves of the best ones. The moves are assigned and improved in iterations until the end of time. An iteration consecutively reassigns the action for each of the four drones assuming the others apply the already assigned moves, so there is opponent prediction. Each next iteration uses a larger granularity. The eval is taken as the average eval over a few best samples.

5. **Pixel-perfect engine**. The rules of the game are extremely complicated, depending on the exact order of double arithmetic in the referee. Preparing a correct and moderately efficient implementation took the first half of the contest or more. The perfectness does not improve the results much, but it was required to properly debug the exact inference and other mechanisms. As a bonus, I can also safely pass monsters by a pixel.
The overwhelming number of minor cases and inaccuracies were consistently attacked from all the parts until perfection. Definitely the hardest engine I ever needed. The following are some elements that surprised me and I had to take care to achieve the perfectness. You can also use them as a test of knowing the rules.

* You can slide on the ocean floor by using WAIT, so move without scaring fishes. Not useful, of course.
* Math.round works differently than std::round for negative numbers. So Java rounding must be emulated.
* A fish at normal speed (200) can change its vector by subsequent normalization and rounding, i.e., when stops being scared, its vector is normalized to 200; then, in the next turn, it is normalized again with a different result by one pixel. Hence, fish vectors must be always normalized if not scared.
* To the contrary, the monster vectors must not be normalized every time, but only when the vector length exceeds their normal speed (270).
* If we pick up a far target and normalize the vector from the drone to the target, and then MOVE by this vector, we can end up at a different position by one pixel since the normalization of a normalized vector is not identical.
* The normalization must be performed exactly as in the referee, so first divide, then multiply by the desired length. The usual way (multiply by the length divided), which is better in terms of efficiency and numerical accuracy, unfortunately, gives slightly different results.
* Monsters can get the zero vector and thus stop in rare cases.
* The visibility radius uses sharp inequality, whereas the other circles use less or equal.
* The boundary reflection of a monster is calculated after rounding the speed, whereas that for a fish uses the normalized speed before the round. This means that a fish can bounce one pixel earlier than it would be required.

Finally, I have answers to some questions that arose during the contest:

* How much the full information about the fishes (i.e., all fishes visible) improves the results?
  
`MSz-knowing-all-fishes vs MSz: 75.00% ±0.3 (99% CI)`

For weaker earlier versions, the improvement was not that much.

* How much the full information about the monsters improves the results?

`MSz-knowing-all-monsters vs MSz: 51.51% ±0.4`

* And together?

`MSz-knowing-both vs MSz: 78.84% ±0.3`

In these tests, I have not taken into account that the inference and samples become unnecessary, so the bot could use time better.

* What if I have twice larger time limit?

`MSz-100ms vs MSz: 52.06% ±0.4`

### Summary

The engine was very difficult, beating the record of complication so far.

I tried to optimize everything further to increase the number of directions / use more samples / deepen the simulation. However, these did not improve the results much, whereas a deeper simulation would be far too expensive. The engine could not be significantly optimized; simply, there is no other way than just following the referee's algorithm. The only more notable optimizations are: precomputing all 4800 maximal drone vectors and applying the action to a partially computed state (other drones, fishes, and monsters moved).

So unfortunately, it seemed that tuning the crappy eval is more important, and I could not find any feature that allows getting me an advantage. I also noted a problem with meta strategies, but they only appeared for early/mid versions, where, e.g., a bot going up earlier wins more often with weaker bots because they do not kill fishes effectively. Later versions were consistently monotonic.
