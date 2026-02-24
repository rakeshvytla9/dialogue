W1 — Utilitarian Welfare (Average Satisfaction)
Formula:

$$W_1 = \frac{1}{N} \sum_{i=1}^{N} s_i$$

Where $s_i$ is the per-member constraint satisfaction rate for member $i$ (i.e., what fraction of that member's preferences were satisfied in the final plan), and $N$ is the number of group members.

Intuition: "What's the average happiness?" Maximizes total group happiness. A plan could score well here even if one member gets 0% and another gets 100% (average = 50%).

Note: W1 is mathematically identical to M10 (member_condition_satisfaction_rate) — both compute the mean across members.

W2 — Egalitarian Welfare (Worst-off Member)
Formula:

$$W_2 = \min_{i \in {1..N}} s_i$$

Simply the minimum satisfaction across all members.

Intuition: "How happy is the least happy person?" Based on Rawls' maximin principle — a just plan is one that maximizes the welfare of the worst-off member. This is the harshest metric: if even one member gets 0 satisfaction, W2 = 0.

Why it matters for your paper: W2 shows the clearest C4 > C3 treatment effect because social theory strategies like Least Misery and Elder Stamina explicitly protect the worst-off member from being ignored.

W3 — Nash Welfare (Geometric Mean)
Formula:

$$W_3 = \left(\prod_{i=1}^{N} s_i\right)^{1/N}$$

The geometric mean of all members' satisfaction rates.

Intuition: "Is satisfaction balanced?" The geometric mean naturally penalizes inequality — if one member has very low satisfaction, it drags the whole product down multiplicatively. It's a middle ground between W1 (pure average) and W2 (pure worst-case):

More forgiving than W2 (a single low score doesn't zero it out, unless it's literally 0)
More equity-sensitive than W1 (can't compensate a 0 with a 1)


<img width="722" height="584" alt="image" src="https://github.com/user-attachments/assets/413e9349-cbf1-4e54-b74f-eface95a140e" />
<img width="727" height="538" alt="image" src="https://github.com/user-attachments/assets/785a1043-da79-4998-b9a9-58f35d20f6c5" />


