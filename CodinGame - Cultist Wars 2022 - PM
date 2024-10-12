(Post Mortem for [Community Contest -- Cultist Wars 2022](https://www.codingame.com/contests/cultist-wars))

1st

My bot uses iterative deepening alpha-beta with classic improvements from chess engines (hash move, quiescence, aspiration, futility, etc.).
The point was to make it very efficient. Even a small optimization was giving a noticeable improvement in the agent's strength. Also, I prune many less relevant moves, depending on the position and the depth.
The evaluation was kept simple but fast; just material and distances between units, based only on precomputed BFS values. It was further simplified at the cost of quality to make the search even deeper.

Of course, exact simulation with neutral moves and indirect shoots was obligatory. With these, the rules of the game are only apparently simple, making the creation of an efficient simulation engine non-trivial and very interesting. For instance, it is possible to detect a potentially killable unit with one 128-bit mask operation.

These allowed computing about 120k states/turn in the middle of the game, which translates to an average depth of about 16 of the (pruned) search. With this, I think the benefits from neutral move prediction are much larger and local battles are better solved, as these are the things difficult for evaluation.

Thank @Nixerrr for the game and CG for organizing!
Certainly, it would not be possible to reach such a level of playing in a 10-days contest. Giving at least a month allows for doing some research around and learning new techniques, instead of just coding already known ones as fast as possible (and it is more compatible with normal life).
