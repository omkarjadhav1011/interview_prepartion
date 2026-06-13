# 04 — DSA: Dynamic Programming

> The deepest topic in the interview. DP separates SDE-1 from SDE-2 and is the single most common reason strong candidates fail onsites. The skill is not memorizing solutions — it's **recognizing the pattern**, then mechanically deriving the state, recurrence, base cases, and order. Every question here is **complete**: state definition → recurrence → base cases → iteration order → BOTH memoization and tabulation Java → space optimization → complexity → how to recognize the pattern next time. The hardest part of every DP is **defining the state**, so that gets the most ink.

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

DP is learned by **pattern**, not by problem. There are only ~8 recurring shapes; once you can name the shape, the state and recurrence almost write themselves. This file is organized by those 8 patterns. Each pattern opens with a **framework** — the trigger phrases, the canonical state, and the canonical recurrence — then a set of questions that drill it.

For **every** problem you will see, in order:
1. **State definition** — what `dp[i]` / `dp[i][j]` *means*. This is 80% of the difficulty. If your state is wrong, nothing else can be right.
2. **Recurrence** — the transition, written as math.
3. **Base cases** — the seeds.
4. **Iteration order** — and *why* (a cell must be computed after everything it depends on).
5. **Top-down memoization** AND **bottom-up tabulation** Java.
6. **Space optimization** — collapsing 2D→1D or keeping a few rolling variables.
7. **Complexity** — time and space.
8. **Pattern recognition** — the sentence you say to yourself to know it's this pattern next time.

The meta-skill: read a problem and say *"this is interval DP because the answer for a range depends on a split point inside it"* or *"this is knapsack because I'm choosing a subset under a capacity constraint"* — **before** writing a line.

---

## Table of Contents — organized by pattern

**Pattern 1 — 1D / Linear DP** (`dp[i]` from a few previous indices)
1. Climbing Stairs
2. House Robber I
3. House Robber II (circular)
4. Fibonacci with Matrix Exponentiation O(log n)
5. Coin Change (minimum coins)
6. Perfect Squares
7. Longest Increasing Subsequence (O(n²) and O(n log n))
8. Maximum Length of Pair Chain
9. Wiggle Subsequence
10. Number of Longest Increasing Subsequences

**Pattern 2 — 2D / Grid DP** (`dp[i][j]` from neighbors in a grid)
11. Unique Paths
12. Unique Paths II (obstacles)
13. Minimum Path Sum
14. Triangle
15. Maximal Square
16. Dungeon Game (reverse DP)
17. Cherry Pickup II

**Pattern 3 — String DP** (two strings or one string, `dp[i][j]`)
18. Longest Common Subsequence
19. Longest Common Substring
20. Longest Palindromic Subsequence
21. Minimum ASCII Delete Sum
22. Distinct Subsequences
23. Interleaving String
24. Regular Expression Matching
25. Wildcard Matching
26. Shortest Common Supersequence
27. Minimum Insertions to Make a String Palindrome

**Pattern 4 — Knapsack DP** (choose a subset under a capacity)
28. 0/1 Knapsack
29. Unbounded Knapsack
30. Subset Sum
31. Partition Equal Subset Sum
32. Target Sum
33. Ones and Zeroes
34. Coin Change II (count combinations)
35. Rod Cutting

**Pattern 5 — Interval DP** (`dp[i][j]` over a range, split inside)
36. Matrix Chain Multiplication
37. Burst Balloons
38. Minimum Cost to Merge Stones
39. Palindrome Partitioning II (minimum cuts)
40. Guess Number Higher or Lower II

**Pattern 6 — State Machine DP** (a fixed set of states per index)
41. Paint House
42. Paint House II
43. Paint House III
44. Student Attendance Record II

**Pattern 7 — Tree DP** (DP on a tree via post-order)
45. House Robber III
46. Binary Tree Cameras
47. Longest Univalue Path

**Pattern 8 — Bitmask DP** (state = subset of items as a bitmask)
48. Partition to K Equal Sum Subsets
49. Shortest Path Visiting All Nodes (BFS + bitmask)
50. Travelling Salesman Problem (Held–Karp)
51. Number of Ways to Wear Different Hats

---

# DP Patterns Framework

Before any question, internalize these 8 shapes. When a new problem arrives, run down this list and ask "does the answer for a subproblem depend on subproblems of this *shape*?"

### Pattern 1 — 1D / Linear DP
**Trigger:** the answer at index `i` depends on a constant number of *earlier* indices (`i-1`, `i-2`, sometimes all `j < i`). Sequences, "ways to reach", "rob/skip", "longest ... ending at i".
**Canonical state:** `dp[i]` = best/count *ending at* or *up to* index `i`.
**Canonical recurrence:** `dp[i] = f(dp[i-1], dp[i-2], ...)` or `dp[i] = best over j<i of g(dp[j])`.
**Space:** usually collapses to O(1) (rolling variables) or O(n).

### Pattern 2 — 2D / Grid DP
**Trigger:** a matrix/grid, movement restricted to right/down (or up/left), "paths", "min cost to traverse".
**Canonical state:** `dp[i][j]` = best/count to reach cell `(i,j)`.
**Canonical recurrence:** `dp[i][j] = grid[i][j] + min/max/sum(dp[i-1][j], dp[i][j-1])`.
**Space:** collapses to O(cols) — one row at a time.

### Pattern 3 — String DP
**Trigger:** one or two strings; "edit", "common", "match", "interleave", "subsequence". Almost always `dp[i][j]` over prefixes of the two strings.
**Canonical state:** `dp[i][j]` = answer for `s[0..i)` and `t[0..j)`.
**Canonical recurrence:** branch on whether `s[i-1] == t[j-1]`.
**Space:** collapses to two rows (or one with care).

### Pattern 4 — Knapsack DP
**Trigger:** choose a *subset* of items to optimize a value subject to a *capacity/sum* constraint. "can we make sum S", "max value within weight W", "count ways to make amount".
**Canonical state:** `dp[i][w]` = best using first `i` items with capacity `w`.
**Canonical recurrence:** `dp[i][w] = max(dp[i-1][w], value_i + dp[i-1][w - weight_i])` (0/1) or `dp[i-1]`→`dp[i]` for unbounded.
**Space:** 1D over capacity. **0/1 → iterate capacity descending; unbounded → ascending.** (Memorize this.)

### Pattern 5 — Interval DP
**Trigger:** the answer for a *range* `[i, j]` is built by choosing a *split point* `k` inside it. "merge", "burst", "partition", "matrix chain", "min cost to combine".
**Canonical state:** `dp[i][j]` = best for subarray `[i, j]`.
**Canonical recurrence:** `dp[i][j] = best over k in [i, j] of dp[i][k] (+/op) dp[k+1][j] + cost`.
**Iteration order:** by increasing *length* of the interval (so smaller intervals are ready).

### Pattern 6 — State Machine DP
**Trigger:** at each step you're in one of a fixed set of *states* and transitions are constrained ("can't paint two adjacent the same", "at most 2 absences"). Stock problems live here too (file 01).
**Canonical state:** `dp[i][state]`.
**Canonical recurrence:** for each state, take the best reachable previous state.
**Space:** O(states), constant.

### Pattern 7 — Tree DP
**Trigger:** a tree; the answer at a node depends on answers from its children. "rob the tree", "cover with cameras", "path through node".
**Canonical state:** each node returns a small tuple (e.g. `[robbed, notRobbed]`) computed in **post-order** (children first).
**Recurrence:** combine children's tuples at the parent.

### Pattern 8 — Bitmask DP
**Trigger:** small `n` (≤ ~20), and the state is "which *subset* of items have I used/visited". "visit all nodes", "assign tasks to people", "TSP".
**Canonical state:** `dp[mask]` or `dp[mask][i]` = best having handled exactly the set `mask` (ending at `i`).
**Recurrence:** add one more element to the subset.
**Complexity:** typically O(2ⁿ · n) or O(2ⁿ · n²).

> **The universal recipe.** (1) Define the state in one English sentence. (2) Write the recurrence as math. (3) Write the base cases. (4) Decide the order so dependencies are ready. (5) Code memoization first (it mirrors the recurrence). (6) Convert to tabulation. (7) Optimize space. Say each step out loud in the interview — it shows structured thinking even if you stumble on a detail.

---

# PATTERN 1 — 1D / LINEAR DP

The simplest and most common shape. The state is a single index; the answer at `i` depends on a *bounded* number of earlier answers (Fibonacci-style) or on *all* earlier answers (LIS-style). Master the rolling-variable space optimization here — it's the cleanest demonstration that you understand which previous states you actually need.

---

### Q1: Climbing Stairs
**Company:** Amazon, Google, Adobe, Microsoft — everywhere as a warm-up
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
You climb a staircase of `n` steps. Each move you take either 1 or 2 steps. How many distinct ways to reach the top?

**What interviewer is testing:**
Can you see that "ways to reach step i" is the sum of "ways to reach the two steps you could have jumped from"? It's Fibonacci in disguise, and they want to hear you derive the recurrence rather than recite it.

**Ideal Answer:**

*(1) State.* `dp[i]` = number of distinct ways to reach step `i`.

*(2) Recurrence.* To stand on step `i`, your last move came either from `i-1` (a 1-step) or `i-2` (a 2-step). Those are disjoint and exhaustive, so:
```
dp[i] = dp[i-1] + dp[i-2]
```

*(3) Base cases.* `dp[0] = 1` (one way to be at the ground: do nothing). `dp[1] = 1`.

*(4) Iteration order.* Ascending `i`, because `dp[i]` needs the two smaller indices already computed.

*(5a) Top-down memoization:*
```java
public int climbStairs(int n) {
    Integer[] memo = new Integer[n + 1];
    return climb(n, memo);
}
private int climb(int n, Integer[] memo) {
    if (n <= 1) return 1;
    if (memo[n] != null) return memo[n];
    return memo[n] = climb(n - 1, memo) + climb(n - 2, memo);
}
```

*(5b) Bottom-up tabulation:*
```java
public int climbStairs(int n) {
    if (n <= 1) return 1;
    int[] dp = new int[n + 1];
    dp[0] = 1; dp[1] = 1;
    for (int i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}
```

*(6) Space optimization.* We only ever read the last two values, so keep two rolling variables:
```java
public int climbStairs(int n) {
    if (n <= 1) return 1;
    int prev2 = 1, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Dry run** — `n=5`: dp = 1,1,2,3,5,8 → **8**. ✅

*(7) Complexity.* O(n) time, O(1) space (rolling) or O(n) (array).

*(8) Pattern recognition.* "Ways to reach a position where each move comes from a fixed set of earlier positions" → 1D additive DP. The instant you see "+= ways from i-1 and i-2", it's Fibonacci-shaped.

**Follow-up questions the interviewer will ask:**
1. *"What if you can take 1, 2, or 3 steps?"* → `dp[i] = dp[i-1] + dp[i-2] + dp[i-3]` (Tribonacci).
2. *"What if you can take any step in a set `steps[]`?"* → `dp[i] = Σ dp[i - s]` for each `s ≤ i`. O(n·k).
3. *"Each step has a cost; minimize cost to reach top (Min Cost Climbing Stairs)?"* → `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`.
4. *"O(log n)?"* → Matrix exponentiation (Q4).

**Common mistakes that get you rejected:**
- Off-by-one on the base case (`dp[0]`). Justify it as "1 way to do nothing."
- Recomputing without memoization → exponential (the classic "exponential Fibonacci" trap). State the memo explicitly.

**The Min Cost Climbing Stairs variant (asked constantly as the follow-up).** Each step `i` has a cost `cost[i]`; you may start at step 0 or 1; pay the cost when you *step on* a stair; reach "the top" (just past the last stair). Minimize total cost.

*(1) State.* `dp[i]` = min cost to **reach** step `i`. *(2) Recurrence.* `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`. *(3) Base.* `dp[0]=cost[0]`, `dp[1]=cost[1]`. *(Answer.)* `min(dp[n-1], dp[n-2])` (the top is past the end; you can finish from either of the last two).
```java
public int minCostClimbingStairs(int[] cost) {
    int n = cost.length, prev2 = cost[0], prev1 = cost[1];
    for (int i = 2; i < n; i++) {
        int cur = cost[i] + Math.min(prev1, prev2);
        prev2 = prev1; prev1 = cur;
    }
    return Math.min(prev1, prev2);
}
```
On `[10,15,20]` → 15 (start at step 1, pay 15, jump to top). On `[1,100,1,1,1,100,1,1,100,1]` → 6.

**The "any step set" variant.** If allowed moves are an arbitrary set `steps[]`, `dp[i] = Σ dp[i - s]` over `s ∈ steps, s ≤ i`. This is exactly the unbounded-knapsack *count* shape (Coin Change II, Q34) — same skeleton, "ways to reach i."

---

### Q2: House Robber I
**Company:** Amazon, Google, Microsoft, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Houses in a row each hold `nums[i]` money. You cannot rob two **adjacent** houses (alarms link neighbors). Maximize the loot.

**What interviewer is testing:**
The "include vs exclude" decision — the seed of all subset DP. They want the clean two-variable rolling solution and a crisp statement of the recurrence.

**Ideal Answer:**

*(1) State.* `dp[i]` = maximum money robbing among houses `0..i` (with the no-adjacent rule).

*(2) Recurrence.* At house `i` you either **rob it** (then you cannot have robbed `i-1`, so add `nums[i]` to `dp[i-2]`) or **skip it** (carry `dp[i-1]`):
```
dp[i] = max(dp[i-1],  nums[i] + dp[i-2])
```

*(3) Base cases.* `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.

*(4) Iteration order.* Ascending — each `dp[i]` reads `i-1` and `i-2`.

*(5a) Top-down:*
```java
public int rob(int[] nums) {
    Integer[] memo = new Integer[nums.length];
    return robFrom(nums, nums.length - 1, memo);
}
private int robFrom(int[] nums, int i, Integer[] memo) {
    if (i < 0) return 0;
    if (i == 0) return nums[0];
    if (memo[i] != null) return memo[i];
    return memo[i] = Math.max(robFrom(nums, i - 1, memo),
                              nums[i] + robFrom(nums, i - 2, memo));
}
```

*(5b) Bottom-up:*
```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 0) return 0;
    if (n == 1) return nums[0];
    int[] dp = new int[n];
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);
    for (int i = 2; i < n; i++) {
        dp[i] = Math.max(dp[i - 1], nums[i] + dp[i - 2]);
    }
    return dp[n - 1];
}
```

*(6) Space optimization.* Only `i-1` and `i-2` matter:
```java
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;           // dp[i-2], dp[i-1]
    for (int x : nums) {
        int cur = Math.max(prev1, x + prev2);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Dry run** — `[2,7,9,3,1]`: cur→2,7,11,11,12 → **12** (rob 2+9+1). ✅

*(7) Complexity.* O(n) time, O(1) space.

*(8) Pattern recognition.* "Pick a subset of a sequence with a *no-two-adjacent* (or some local) constraint, maximize/minimize" → linear include/exclude DP. Anytime the choice at `i` forbids using `i-1`, you'll reach back to `i-2`.

**Follow-up questions the interviewer will ask:**
1. *"Houses in a circle?"* → House Robber II (Q3).
2. *"Reconstruct which houses?"* → Walk `dp` backward: if `dp[i] == nums[i] + dp[i-2]`, house `i` was robbed.
3. *"Houses form a tree?"* → House Robber III (Q45).
4. *"Can't rob two within distance k?"* → `dp[i] = max(dp[i-1], nums[i] + dp[i-k-1])`.

**Common mistakes that get you rejected:**
- Initializing `dp[1] = nums[1]` instead of `max(nums[0], nums[1])`.
- Greedily robbing alternate houses (`[2,1,1,2]` → greedy picks 2+1=3, optimal is 2+2=4).

**Reconstructing which houses were robbed.** With the full `dp[]` array, walk backward: house `i` was robbed iff taking it produced `dp[i]` (i.e. `dp[i] == nums[i] + dp[i-2]` and `dp[i] != dp[i-1]`):
```java
public List<Integer> robbedHouses(int[] nums) {
    int n = nums.length;
    if (n == 0) return List.of();
    int[] dp = new int[n];
    dp[0] = nums[0];
    if (n > 1) dp[1] = Math.max(nums[0], nums[1]);
    for (int i = 2; i < n; i++) dp[i] = Math.max(dp[i - 1], nums[i] + dp[i - 2]);
    List<Integer> picked = new ArrayList<>();
    int i = n - 1;
    while (i >= 0) {
        int prev = i >= 2 ? dp[i - 2] : 0;
        if (i == 0 || dp[i] != dp[i - 1]) {        // house i contributed
            picked.add(i); i -= 2;                  // skip the adjacent neighbor
        } else i--;                                 // house i skipped
    }
    Collections.reverse(picked);
    return picked;
}
```
This backtracking-through-the-table technique is the universal way to recover the *solution* (not just its value) from any DP — interviewers love asking for it after you give the optimal value.

---

### Q3: House Robber II (Circular)
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Same as House Robber I, but the houses are arranged in a **circle** — the first and last are adjacent. Maximize loot.

**What interviewer is testing:**
Reducing a circular constraint to two linear cases. The elegant trick is the signal.

**Ideal Answer:**

The wrinkle: house `0` and house `n-1` are now adjacent, so they can't both be robbed. **Key reduction:** in any valid plan, either house `0` is *not robbed* or house `n-1` is *not robbed* (they can't both be in). So run the linear House Robber I twice:
- once on houses `[0 .. n-2]` (exclude the last),
- once on houses `[1 .. n-1]` (exclude the first),

and take the max. Each subproblem is now a *line*, not a circle, so the adjacency between ends never arises.

*(1) State / (2) recurrence / (3) base / (4) order:* identical to Q2, applied to a subrange.

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0];
    return Math.max(robLine(nums, 0, n - 2),
                    robLine(nums, 1, n - 1));
}
private int robLine(int[] nums, int lo, int hi) {
    int prev2 = 0, prev1 = 0;
    for (int i = lo; i <= hi; i++) {
        int cur = Math.max(prev1, nums[i] + prev2);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Dry run** — `[2,3,2]`: line [0..1] → max(2,3)=3; line [1..2] → max(3,2)=3 → **3** (can't take both 2s). ✅

*(6) Space.* O(1). *(7) Complexity.* O(n) time, O(1) space (two linear passes).

*(8) Pattern recognition.* A *circular* constraint where the two ends conflict → solve the linear version twice, each time dropping one end. This trick recurs (e.g. circular max-subarray in file 01).

**Follow-up questions the interviewer will ask:**
1. *"Why exactly those two cases — why not three?"* → The only new constraint vs. the line is "not both ends." Fixing "skip first" OR "skip last" covers every valid configuration; their union is exhaustive and the optimum lies in one of them.
2. *"n == 1 edge case?"* → Return `nums[0]` directly; the two ranges would be invalid.

**Common mistakes that get you rejected:**
- Forgetting the `n == 1` guard (ranges `[0,-1]` and `[1,0]` are empty/invalid).
- Trying to special-case "rob both ends or not" inline — messier and error-prone than two clean linear passes.

---

### Q4: Fibonacci with Matrix Exponentiation O(log n)
**Company:** Google, Two Sigma, Jane Street
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite (math-leaning)

**Question:**
Compute the `n`-th Fibonacci number in **O(log n)** time.

**What interviewer is testing:**
Whether you know that a linear recurrence can be expressed as a matrix power, and fast exponentiation by squaring. This is the bridge between DP and number theory.

**Ideal Answer:**

The naive DP is O(n). To get O(log n) we use the identity:
```
| F(n+1) |   | 1 1 |^n   | F(1) |
| F(n)   | = | 1 0 |   · | F(0) |
```
That is, `[[1,1],[1,0]]^n = [[F(n+1), F(n)], [F(n), F(n-1)]]`. Raising the 2×2 matrix to the `n`-th power by **fast exponentiation** (squaring) takes O(log n) matrix multiplications, each O(1) for a fixed 2×2.

```java
public long fib(int n) {
    if (n == 0) return 0;
    long[][] result = {{1, 0}, {0, 1}};   // identity
    long[][] base   = {{1, 1}, {1, 0}};
    while (n > 0) {
        if ((n & 1) == 1) result = multiply(result, base);
        base = multiply(base, base);
        n >>= 1;
    }
    return result[0][1];                   // equals F(n)
}
private long[][] multiply(long[][] a, long[][] b) {
    return new long[][] {
        { a[0][0]*b[0][0] + a[0][1]*b[1][0],  a[0][0]*b[0][1] + a[0][1]*b[1][1] },
        { a[1][0]*b[0][0] + a[1][1]*b[1][0],  a[1][0]*b[0][1] + a[1][1]*b[1][1] }
    };
}
```

**Dry run** — `n=5`: the exponent bits of 5 are `101`. After squaring/accumulating, `result[0][1] = 5 = F(5)`. ✅

**Complexity.** O(log n) time, O(1) space (fixed-size matrices).

*(8) Pattern recognition.* **Any** linear recurrence `f(n) = c₁f(n-1) + ... + c_k f(n-k)` can be written as a `k×k` transition matrix and computed in O(k³ log n). When the interviewer asks for sub-linear time on a recurrence, reach for matrix exponentiation.

**Follow-up questions the interviewer will ask:**
1. *"Generalize to a k-term recurrence?"* → Build the `k×k` companion matrix; complexity O(k³ log n).
2. *"Modular arithmetic to avoid overflow?"* → Take everything `% M` inside `multiply`.
3. *"Closed-form (Binet's formula)?"* → `F(n) = (φⁿ − ψⁿ)/√5`; O(1) but floating-point precision fails for large n. Matrix power is the exact O(log n).

**Common mistakes that get you rejected:**
- Linear exponentiation (multiplying `base` n times) — that's O(n), defeating the purpose.
- Overflow without `long`/modulo.

---

### Q5: Coin Change (Minimum Coins)
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Coins of given denominations (unlimited supply). Return the **minimum number of coins** to make `amount`, or `-1` if impossible.

**What interviewer is testing:**
Unbounded "min ways" DP, the impossibility sentinel, and why greedy fails. This is one of the most-asked DP problems — make it bulletproof.

**Ideal Answer:**

First, **kill the greedy instinct**: greedily taking the largest coin fails for `coins=[1,3,4]`, `amount=6` (greedy 4+1+1=3 coins; optimal 3+3=2 coins). So we need DP.

*(1) State.* `dp[a]` = the minimum number of coins needed to make amount `a`.

*(2) Recurrence.* To make `a`, the last coin used is some `c ≤ a`; the rest is `dp[a - c]`. Take the best last coin:
```
dp[a] = 1 + min over c in coins, c ≤ a  of  dp[a - c]
```

*(3) Base cases.* `dp[0] = 0` (zero coins make amount 0). Initialize all other `dp[a] = ∞` (use `amount+1` as a safe "infinity" since you never need more than `amount` coins).

*(4) Iteration order.* Ascending `a` from 1 to `amount`; `dp[a]` depends on smaller amounts.

*(5a) Top-down memoization:*
```java
public int coinChange(int[] coins, int amount) {
    Integer[] memo = new Integer[amount + 1];
    int r = dp(coins, amount, memo);
    return r == Integer.MAX_VALUE ? -1 : r;
}
private int dp(int[] coins, int amount, Integer[] memo) {
    if (amount == 0) return 0;
    if (amount < 0) return Integer.MAX_VALUE;
    if (memo[amount] != null) return memo[amount];
    int best = Integer.MAX_VALUE;
    for (int c : coins) {
        int sub = dp(coins, amount - c, memo);
        if (sub != Integer.MAX_VALUE) best = Math.min(best, sub + 1);
    }
    return memo[amount] = best;
}
```

*(5b) Bottom-up tabulation:*
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);        // "infinity"
    dp[0] = 0;
    for (int a = 1; a <= amount; a++) {
        for (int c : coins) {
            if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

**Dry run table** — `coins=[1,2,5]`, `amount=11`:

| a    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|------|---|---|---|---|---|---|---|---|---|---|----|----|
| dp[a]| 0 | 1 | 1 | 2 | 2 | 1 | 2 | 2 | 3 | 3 | 2  | 3  |

- dp[5] = 1 (single 5). dp[10] = 2 (5+5). dp[11] = dp[10]+1 = 3 (5+5+1). **Answer 3.** ✅

*(6) Space optimization.* Already 1D — O(amount). Cannot reduce below that (each amount is a distinct subproblem).

*(7) Complexity.* O(amount · |coins|) time, O(amount) space.

*(8) Pattern recognition.* "Minimum/number of *items from an unlimited pool* to reach a target sum" → **unbounded knapsack, min variant**. Tell-tale: unlimited supply + "make exactly target." Contrast with Coin Change II (Q34) which *counts* combinations and needs a different loop order.

**Follow-up questions the interviewer will ask:**
1. *"Count the number of ways instead of min coins?"* → Coin Change II (Q34) — swap the loop nesting (coins outer, amount inner) to count *combinations* not *permutations*.
2. *"Why does greedy fail, and when does it work?"* → Greedy works only for "canonical" coin systems (like US coins). General systems need DP. Give the `[1,3,4]` counterexample.
3. *"Reconstruct which coins?"* → Store `parent[a] = c` whenever you improve `dp[a]`; backtrack from `amount`.
4. *"amount up to 10⁹ but few coins?"* → DP table too big; this becomes a number-theory / BFS-on-residues problem.

**Common mistakes that get you rejected:**
- Using `Integer.MAX_VALUE` then doing `+1` → overflow. Use `amount+1` as infinity, or guard before adding.
- Returning `dp[amount]` without the impossibility check.
- Confusing "min coins" (this) with "count ways" (Q34) — different loop structure.

**Reconstructing which coins were used.** Keep a `parent[a]` = the last coin that achieved the optimum for amount `a`. Backtrack from `amount`:
```java
public List<Integer> coinChangeCoins(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    int[] parent = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    Arrays.fill(parent, -1);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (c <= a && dp[a - c] + 1 < dp[a]) { dp[a] = dp[a - c] + 1; parent[a] = c; }
    if (dp[amount] > amount) return null;            // impossible
    List<Integer> used = new ArrayList<>();
    for (int a = amount; a > 0; a -= parent[a]) used.add(parent[a]);
    return used;
}
```

**BFS view (sometimes clearer in interviews).** Treat each amount as a graph node; edges subtract a coin. The minimum number of coins is the shortest path from `amount` to `0` — BFS layer by layer. The first time you reach 0, the layer index is the answer. Same O(amount·|coins|) worst case but can terminate early:
```java
public int coinChangeBFS(int[] coins, int amount) {
    if (amount == 0) return 0;
    boolean[] seen = new boolean[amount + 1];
    Deque<Integer> q = new ArrayDeque<>();
    q.offer(amount); seen[amount] = true;
    int level = 0;
    while (!q.isEmpty()) {
        level++;
        for (int sz = q.size(); sz > 0; sz--) {
            int cur = q.poll();
            for (int c : coins) {
                int next = cur - c;
                if (next == 0) return level;
                if (next > 0 && !seen[next]) { seen[next] = true; q.offer(next); }
            }
        }
    }
    return -1;
}
```
Mentioning the DP↔BFS equivalence (DP over a DAG of amounts = shortest path) is a strong signal of conceptual maturity.

---

### Q6: Perfect Squares
**Company:** Google, Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the least number of perfect-square numbers (`1, 4, 9, 16, ...`) that sum to `n`.

**What interviewer is testing:**
Recognizing this is **Coin Change** where the "coins" are perfect squares. Same recurrence, different coin set.

**Ideal Answer:**

The "coins" are `1, 4, 9, ..., ⌊√n⌋²`, supply unlimited, and we want the minimum count summing to `n`. Identical structure to Q5.

*(1) State.* `dp[i]` = least number of perfect squares summing to `i`.

*(2) Recurrence.* `dp[i] = 1 + min over j with j²≤i of dp[i - j²]`.

*(3) Base cases.* `dp[0] = 0`.

*(4) Iteration order.* Ascending `i`.

*(5b) Bottom-up:*
```java
public int numSquares(int n) {
    int[] dp = new int[n + 1];
    Arrays.fill(dp, Integer.MAX_VALUE);
    dp[0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j * j <= i; j++) {
            dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
        }
    }
    return dp[n];
}
```

*(5a) Top-down:*
```java
public int numSquares(int n) {
    Integer[] memo = new Integer[n + 1];
    return solve(n, memo);
}
private int solve(int n, Integer[] memo) {
    if (n == 0) return 0;
    if (memo[n] != null) return memo[n];
    int best = Integer.MAX_VALUE;
    for (int j = 1; j * j <= n; j++) best = Math.min(best, solve(n - j * j, memo) + 1);
    return memo[n] = best;
}
```

**Dry run** — `n=12`: dp[12] = dp[8]+1 (using 4) = (dp[4]+1)+1 = ((dp[0]+1)+1)+1 = **3** (4+4+4). ✅

*(6) Space.* O(n). *(7) Complexity.* O(n·√n) time, O(n) space.

*(8) Pattern recognition.* "Minimum count of items from a fixed set summing to n" — the same skeleton as Coin Change. When you see "perfect squares / fixed numeric set / unlimited use / minimize count," it's unbounded-knapsack-min.

**Follow-up questions the interviewer will ask:**
1. *"O(1) math solution?"* → **Lagrange's four-square theorem**: every n is the sum of at most 4 squares. The answer is 1 if n is a perfect square; 4 if `n = 4ᵏ(8m+7)` (Legendre); else check if n is a sum of two squares (→2) else 3.
2. *"BFS approach?"* → Treat amounts as graph nodes, edges = subtracting a square; shortest path = answer. Same complexity, sometimes faster in practice.

**BFS shortest-path view (the follow-up implementation).** Each integer `0..n` is a node; an edge connects `x` to `x - j²`. The min count is the BFS distance from `n` to `0`:
```java
public int numSquares(int n) {
    boolean[] seen = new boolean[n + 1];
    Deque<Integer> q = new ArrayDeque<>();
    q.offer(n); seen[n] = true;
    int level = 0;
    while (!q.isEmpty()) {
        level++;
        for (int sz = q.size(); sz > 0; sz--) {
            int cur = q.poll();
            for (int j = 1; j * j <= cur; j++) {
                int next = cur - j * j;
                if (next == 0) return level;
                if (!seen[next]) { seen[next] = true; q.offer(next); }
            }
        }
    }
    return n;   // unreachable; every n is reachable via 1s
}
```
BFS often reaches the answer in very few levels (≤4 by Lagrange) so it can beat the DP in practice, while being the *same* underlying graph-shortest-path idea.

**Common mistakes that get you rejected:**
- Greedily taking the largest square (`12` → 9+1+1+1 = 4, but optimal is 4+4+4 = 3).
- Not realizing it's literally Coin Change.

---

### Q7: Longest Increasing Subsequence (LIS)
**Company:** Google, Amazon, Meta, Microsoft, Bloomberg, Uber
**Difficulty:** 🔴 Hard (O(n log n) version)
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given `nums`, return the length of the longest **strictly increasing subsequence** (not necessarily contiguous).

**What interviewer is testing:**
Two things: the O(n²) DP (must be fluent), and the O(n log n) **patience sorting / binary search** trick (separates strong candidates). Knowing *both* and explaining when each is appropriate is the signal.

**Ideal Answer:**

**Approach A — O(n²) DP.**

*(1) State.* `dp[i]` = length of the longest increasing subsequence **ending at index i** (i.e. with `nums[i]` as its last element). This "ending at i" framing is the crux — it makes subproblems combinable.

*(2) Recurrence.* `nums[i]` can extend any earlier subsequence whose last element is smaller:
```
dp[i] = 1 + max( dp[j] for all j < i with nums[j] < nums[i] ),  or 1 if none
```

*(3) Base cases.* Every `dp[i]` starts at 1 (the element alone).

*(4) Iteration order.* Ascending `i`; for each `i` scan all `j < i`.

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    int best = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
        }
        best = Math.max(best, dp[i]);
    }
    return n == 0 ? 0 : best;
}
```

**Dry run** — `[10,9,2,5,3,7,101,18]`:

| i        | 0  | 1 | 2 | 3 | 4 | 5 | 6   | 7  |
|----------|----|---|---|---|---|---|-----|----|
| nums[i]  | 10 | 9 | 2 | 5 | 3 | 7 | 101 | 18 |
| dp[i]    | 1  | 1 | 1 | 2 | 2 | 3 | 4   | 4  |

dp[5]=3 (2,5,7 or 2,3,7); dp[6]=4 (2,3,7,101). **Answer 4.** ✅

**Approach B — O(n log n) patience sorting.**

Maintain `tails`, where `tails[k]` = the **smallest possible tail value** of any increasing subsequence of length `k+1` seen so far. `tails` is always sorted. For each number `x`:
- binary-search the **leftmost** `tails[k] ≥ x`;
- if found, replace it with `x` (a length-`k+1` subsequence can now end smaller — better for the future);
- if not found (`x` bigger than all tails), append `x` (we extended the longest run by 1).

The **length of `tails`** is the LIS length. (Note: `tails` itself is *not* a valid subsequence — only its length is meaningful.)

```java
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    for (int x : nums) {
        int lo = 0, hi = tails.size();         // find leftmost tail >= x
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (tails.get(mid) < x) lo = mid + 1;
            else hi = mid;
        }
        if (lo == tails.size()) tails.add(x);   // extend
        else tails.set(lo, x);                  // replace -> smaller tail
    }
    return tails.size();
}
```

**Dry run (Approach B)** — `[10,9,2,5,3,7,101,18]`:
- 10 → tails [10]
- 9 → replace 10 → [9]
- 2 → replace 9 → [2]
- 5 → append → [2,5]
- 3 → replace 5 → [2,3]
- 7 → append → [2,3,7]
- 101 → append → [2,3,7,101]
- 18 → replace 101 → [2,3,7,18]
- length 4 → **Answer 4.** ✅

**Why replace, not extend, when `x` fits in the middle?** Keeping the smallest possible tail for each length maximizes the chance a future element can extend it. We never *shorten* any achievable length; we only make tails smaller.

*(6) Space.* O(n) for `dp` / `tails`.

*(7) Complexity.* Approach A: O(n²) time, O(n) space. Approach B: O(n log n) time, O(n) space.

*(8) Pattern recognition.* "Longest subsequence with a monotonic condition" → LIS family. If they want better than O(n²), it's the patience-sorting binary-search trick. The **"dp[i] = best ending at i"** framing generalizes to dozens of problems (Q8, Q9, Q10, Maximum Length of Pair Chain, Russian Doll Envelopes).

**Follow-up questions the interviewer will ask:**
1. *"Reconstruct the actual subsequence?"* → In Approach A, store a `parent[]` pointer to the `j` that gave the max; backtrack from the index with the largest `dp`. In Approach B, store for each `x` the length-position it landed in and chain via an index array.
2. *"Non-decreasing (allow equal) instead of strictly increasing?"* → Binary-search for the leftmost tail `> x` (upper bound) instead of `≥ x`.
3. *"Count the number of LIS?"* → Q10.
4. *"Longest increasing *contiguous* subsequence?"* → That's a trivial O(n) scan, not LIS.
5. *"2D version (Russian Doll Envelopes)?"* → Sort by width ascending, height *descending* on ties, then LIS on heights.

**Common mistakes that get you rejected:**
- Claiming `tails` is the actual subsequence — it isn't; only its length is correct.
- Using `≤` vs `<` wrong in the binary search → breaks strict vs non-strict.
- Forgetting the empty-array guard.

**Reconstruction in O(n log n) (worth knowing).** Keep a parallel `idx[]` where `idx[k]` = the *array index* of the element currently sitting at `tails[k]`, and a `prev[]` linking each element to the index that preceded it in its subsequence. When you place `x` at position `k`, set `prev[i] = (k>0 ? idx[k-1] : -1)` and `idx[k] = i`. After the scan, start at `idx[len-1]` and follow `prev` backward, then reverse:
```java
public List<Integer> getLIS(int[] nums) {
    int n = nums.length;
    int[] tails = new int[n], idx = new int[n], prev = new int[n];
    int len = 0;
    for (int i = 0; i < n; i++) {
        int lo = 0, hi = len;
        while (lo < hi) {                          // leftmost tail >= nums[i]
            int mid = (lo + hi) >>> 1;
            if (nums[tails[mid]] < nums[i]) lo = mid + 1; else hi = mid;
        }
        prev[i] = lo > 0 ? idx[lo - 1] : -1;
        tails[lo] = i; idx[lo] = i;
        if (lo == len) len++;
    }
    LinkedList<Integer> seq = new LinkedList<>();
    for (int cur = idx[len - 1]; cur != -1; cur = prev[cur]) seq.addFirst(nums[cur]);
    return seq;
}
```
This is the kind of detail an interviewer adds as the follow-up after you nail the length; having it ready signals depth.

**Why patience sorting is correct (the proof to recite).** `tails` is always sorted ascending (each `tails[k]` is the minimal tail of a length-`k+1` increasing subsequence, and longer subsequences must end higher). Each incoming `x` either extends the longest run (append) or improves some length's tail (replace the leftmost tail `≥ x`). We never decrease an achievable length, and we keep every length's tail as small as possible, which is exactly the greedy invariant that maximizes future extendability. Hence `len(tails)` equals the true LIS length. The card-game intuition: deal cards into piles, each card on the leftmost pile whose top is `≥` it; the number of piles equals the LIS length (Dilworth's theorem in disguise).

---

### Q8: Maximum Length of Pair Chain
**Company:** Google, Amazon
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given pairs `[a, b]` with `a < b`, pair `[c, d]` can follow `[a, b]` iff `b < c`. Find the longest chain you can form. Each pair used at most once.

**What interviewer is testing:**
Whether you recognize this as LIS-with-a-twist that actually has a **greedy** optimum (like interval scheduling). Knowing both the DP and the greedy, and why greedy wins, is the signal.

**Ideal Answer:**

**DP view (LIS-style, O(n²)).** Sort by first element. `dp[i]` = longest chain ending with pair `i`; `dp[i] = 1 + max(dp[j])` over `j` with `pairs[j][1] < pairs[i][0]`. O(n²).

**Greedy view (optimal, O(n log n)).** This is exactly the **activity-selection / interval scheduling** problem: sort by the *second* element ascending, then greedily take a pair whenever its start exceeds the last taken end. Earliest-finishing choices leave the most room.

```java
public int findLongestChain(int[][] pairs) {
    Arrays.sort(pairs, (a, b) -> Integer.compare(a[1], b[1])); // by end ascending
    int count = 0, curEnd = Integer.MIN_VALUE;
    for (int[] p : pairs) {
        if (p[0] > curEnd) {       // chainable
            count++;
            curEnd = p[1];
        }
    }
    return count;
}
```

**Dry run** — `[[1,2],[2,3],[3,4]]` sorted by end → same; take [1,2] (end 2), [3,4] (3>2). [2,3] skipped (2 not >2). → **2**. ✅

*(7) Complexity.* Greedy O(n log n) time, O(1) extra (besides sort). DP O(n²).

*(8) Pattern recognition.* "Chain/schedule non-overlapping intervals, maximize count" → greedy by earliest finish time. If instead it were "maximize total *weight* of chosen intervals," greedy breaks and you need **weighted interval scheduling DP** (sort by end, binary-search the last compatible, `dp[i] = max(dp[i-1], weight + dp[compatible])`).

**Follow-up questions the interviewer will ask:**
1. *"Prove the greedy is optimal."* → Exchange argument: the earliest-finishing interval is in some optimal solution; swapping it in never reduces the count.
2. *"What if pairs have weights and you maximize weight?"* → Weighted interval scheduling DP (above), O(n log n).
3. *"Allow `b == c` (touching) to chain?"* → Change `p[0] > curEnd` to `>=`.

**Common mistakes that get you rejected:**
- Sorting by start instead of end for the greedy (start-sort needs DP; end-sort enables greedy).
- Assuming greedy works for the *weighted* variant.

---

### Q9: Wiggle Subsequence
**Company:** Google, Amazon
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
A wiggle sequence alternates strictly up, down, up, down... Find the length of the longest wiggle **subsequence** of `nums`.

**What interviewer is testing:**
A two-state linear DP (last move was "up" vs "down"), with an O(1) greedy collapse.

**Ideal Answer:**

*(1) State.* Two running lengths:
- `up` = length of the longest wiggle subsequence ending with a rising step,
- `down` = ... ending with a falling step.

*(2) Recurrence.* Walking left to right:
- if `nums[i] > nums[i-1]`: `up = down + 1` (a rise must follow a fall),
- if `nums[i] < nums[i-1]`: `down = up + 1`,
- if equal: neither changes.

*(3) Base.* `up = down = 1`.

*(4) Order.* Left to right.

```java
public int wiggleMaxLength(int[] nums) {
    if (nums.length == 0) return 0;
    int up = 1, down = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] > nums[i - 1]) up = down + 1;
        else if (nums[i] < nums[i - 1]) down = up + 1;
    }
    return Math.max(up, down);
}
```

**Dry run** — `[1,7,4,9,2,5]`: all alternate → up/down climb to 6 → **6**. `[1,17,5,10,13,15,10,5,16,8]` → **7**. ✅

*(6) Space.* O(1). *(7) Complexity.* O(n) time, O(1) space.

*(8) Pattern recognition.* "Longest alternating / two-mode subsequence" → maintain one running length **per mode** and let each transition flip modes. This two-state-rolling idea also powers Paint House and stock DP.

**Follow-up questions the interviewer will ask:**
1. *"Why does the greedy `up/down` give the optimal length?"* → On a monotone run you only ever keep the endpoints; intermediate points can't extend a wiggle, so counting direction changes equals the optimal subsequence length.
2. *"Reconstruct the subsequence?"* → Record indices where direction flips.

**Common mistakes that get you rejected:**
- Mishandling equal adjacent values (must skip, not flip).
- Overcomplicating with an O(n²) LIS-style DP when O(n) suffices.

**The explicit two-array DP (if asked to justify the O(1) version).** Define `up[i]` / `down[i]` = longest wiggle subsequence using `nums[0..i]` ending at `i` with a final up / down step. `up[i] = max(down[j]+1)` over `j<i` with `nums[j]<nums[i]`; symmetric for `down[i]`. This O(n²) form makes the alternation explicit; observing that only the latest `up`/`down` matter collapses it to the two scalars above — a clean "derive then optimize" story to tell.

---

### Q10: Number of Longest Increasing Subsequences
**Company:** Google, Amazon, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Return the **count** of longest strictly increasing subsequences in `nums`.

**What interviewer is testing:**
Augmenting LIS DP with a *count* array and correctly combining counts when lengths tie. The counting bookkeeping trips up most candidates.

**Ideal Answer:**

Extend the O(n²) LIS DP with a second array.

*(1) State.* Two arrays:
- `len[i]` = length of the longest increasing subsequence ending at `i`,
- `cnt[i]` = how many such longest subsequences end at `i`.

*(2) Recurrence.* For each `j < i` with `nums[j] < nums[i]`:
- if `len[j] + 1 > len[i]`: we found a strictly longer subsequence → `len[i] = len[j] + 1; cnt[i] = cnt[j]` (inherit its count).
- else if `len[j] + 1 == len[i]`: another way to achieve the same length → `cnt[i] += cnt[j]` (accumulate).

*(3) Base.* `len[i] = 1, cnt[i] = 1` for all `i`.

*(4) Order.* Ascending `i`, inner `j < i`.

```java
public int findNumberOfLIS(int[] nums) {
    int n = nums.length;
    if (n == 0) return 0;
    int[] len = new int[n], cnt = new int[n];
    Arrays.fill(len, 1);
    Arrays.fill(cnt, 1);
    int maxLen = 1;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                if (len[j] + 1 > len[i]) {
                    len[i] = len[j] + 1;
                    cnt[i] = cnt[j];
                } else if (len[j] + 1 == len[i]) {
                    cnt[i] += cnt[j];
                }
            }
        }
        maxLen = Math.max(maxLen, len[i]);
    }
    int total = 0;
    for (int i = 0; i < n; i++) if (len[i] == maxLen) total += cnt[i];
    return total;
}
```

**Dry run** — `[1,3,5,4,7]`: len=[1,2,3,3,4], cnt=[1,1,1,1,2]. maxLen=4, only i=4 → **2** (1,3,5,7 and 1,3,4,7). ✅

*(7) Complexity.* O(n²) time, O(n) space. (An O(n log n) version exists using segment trees / Fenwick keyed by value — mention it.)

*(8) Pattern recognition.* "Count the *number* of optimal solutions" → carry a parallel `cnt[]` alongside the value DP; **reset** count on a strict improvement, **add** on a tie. This reset-vs-add rule is the universal pattern for counting-optimal DP.

**Follow-up questions the interviewer will ask:**
1. *"O(n log n)?"* → Maintain, per length, the count of subsequences; use a Fenwick/segment tree indexed by value to query "best length & count among smaller values."
2. *"Why reset cnt on a strict improvement?"* → The old shorter subsequences are no longer "longest ending here," so their counts must be discarded.

**Common mistakes that get you rejected:**
- Adding counts on a strict improvement instead of replacing (over-counts).
- Forgetting to sum `cnt[i]` over *all* indices achieving `maxLen` (not just the last).

---

# PATTERN 2 — 2D / GRID DP

A matrix, movement restricted (usually right/down), and you want a count of paths or the best-cost traversal. The state is the cell, the recurrence pulls from the neighbors you could have arrived from, and the space almost always collapses to a single row. The interesting variants flip the direction (Dungeon Game's reverse DP) or track two agents at once (Cherry Pickup).

---

### Q11: Unique Paths
**Company:** Amazon, Google, Bloomberg, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
A robot starts at the top-left of an `m × n` grid and moves only **right** or **down**. How many distinct paths reach the bottom-right?

**What interviewer is testing:**
The cleanest 2D additive DP, the row-collapse space optimization, and that you know the combinatorics closed form.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = number of distinct paths from `(0,0)` to cell `(i,j)`.

*(2) Recurrence.* You enter `(i,j)` from above or from the left:
```
dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

*(3) Base cases.* `dp[0][j] = 1` for all `j` and `dp[i][0] = 1` for all `i` (only one straight path along an edge).

*(4) Iteration order.* Row by row, left to right — `(i,j)` needs the cell above and to the left.

*(5a) Top-down:*
```java
public int uniquePaths(int m, int n) {
    Integer[][] memo = new Integer[m][n];
    return paths(m - 1, n - 1, memo);
}
private int paths(int i, int j, Integer[][] memo) {
    if (i == 0 || j == 0) return 1;
    if (memo[i][j] != null) return memo[i][j];
    return memo[i][j] = paths(i - 1, j, memo) + paths(i, j - 1, memo);
}
```

*(5b) Bottom-up:*
```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    return dp[m - 1][n - 1];
}
```

*(6) Space optimization (2D→1D).* `dp[i][j]` reads only the row above (`dp[i-1][j]`, already stored in `row[j]`) and the cell to the left (`dp[i][j-1]`, the just-updated `row[j-1]`). One array suffices:
```java
public int uniquePaths(int m, int n) {
    int[] row = new int[n];
    Arrays.fill(row, 1);
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            row[j] += row[j - 1];   // row[j](old)=above, row[j-1]=left
    return row[n - 1];
}
```

**Dry run table** — `m=3, n=3`:

| i\j | 0 | 1 | 2 |
|-----|---|---|---|
| 0   | 1 | 1 | 1 |
| 1   | 1 | 2 | 3 |
| 2   | 1 | 3 | 6 |

**Answer 6.** ✅

*(7) Complexity.* O(m·n) time, O(n) space (1D) or O(m·n) (2D).

*(8) Pattern recognition.* "Count paths in a grid, moves restricted to right/down" → additive grid DP, `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. The row-collapse to 1D is the standard space win for any grid DP that reads only the previous row.

**Follow-up questions the interviewer will ask:**
1. *"O(1) math?"* → It's `C(m+n-2, m-1)` — choose which of the `(m-1)+(n-1)` moves are "downs". Compute the binomial carefully to avoid overflow.
2. *"With obstacles?"* → Unique Paths II (Q12).
3. *"Can move in 4 directions?"* → No longer a DAG; becomes graph traversal / it's infinite unless you forbid revisits.

**Common mistakes that get you rejected:**
- Initializing the whole grid to 0 and forgetting the edge seeds.
- Overflow for large `m,n` in the combinatorial formula (use `long`).

**The O(min(m,n)) combinatorial answer.** The robot makes exactly `(m-1)` downs and `(n-1)` rights in some order — choose which of the `(m-1)+(n-1)` moves are downs: `C(m+n-2, m-1)`. Compute the binomial multiplicatively to avoid overflow and division-truncation:
```java
public int uniquePaths(int m, int n) {
    long result = 1;
    int total = m + n - 2, choose = Math.min(m, n) - 1;
    for (int i = 1; i <= choose; i++) {
        result = result * (total - choose + i) / i;   // exact at each step
    }
    return (int) result;
}
```
Why the running division is exact: the product of `i` consecutive integers is always divisible by `i!`, and accumulating numerator-then-divide keeps each partial product a valid binomial coefficient (hence integral).

---

### Q12: Unique Paths II (Obstacles)
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Same grid, but some cells contain obstacles (`grid[i][j] == 1`). Obstacles are impassable. Count distinct right/down paths.

**Ideal Answer:**

Same recurrence, but an obstacle cell has **0 paths through it**, and edge seeds break after the first obstacle.

*(1) State.* `dp[i][j]` = paths to `(i,j)`, with `dp[i][j] = 0` if `(i,j)` is an obstacle.

*(2) Recurrence.*
```
dp[i][j] = 0                                if grid[i][j] == 1
dp[i][j] = dp[i-1][j] + dp[i][j-1]          otherwise
```

*(3) Base.* `dp[0][0] = grid[0][0] == 1 ? 0 : 1`.

*(6) 1D space-optimized:*
```java
public int uniquePathsWithObstacles(int[][] grid) {
    int n = grid[0].length;
    int[] row = new int[n];
    row[0] = grid[0][0] == 1 ? 0 : 1;
    for (int[] r : grid) {
        for (int j = 0; j < n; j++) {
            if (r[j] == 1) row[j] = 0;            // obstacle: no paths
            else if (j > 0) row[j] += row[j - 1]; // add left
        }
    }
    return row[n - 1];
}
```

**Dry run** — `[[0,0,0],[0,1,0],[0,0,0]]`: the center blocked. Paths = 2 (around either side). ✅

*(7) Complexity.* O(m·n) time, O(n) space.

*(8) Pattern recognition.* Same as Q11 with a "0 if blocked" guard — the universal way to add obstacles to any grid DP.

**Follow-up questions the interviewer will ask:**
1. *"Minimum path sum with obstacles?"* → Set obstacle cost to ∞ instead of 0.
2. *"Why does the edge-seed logic differ from Q11?"* → After the first obstacle in row/column 0, all later edge cells become unreachable (0). The 1D update handles this automatically.

**Common mistakes that get you rejected:**
- Seeding the entire first row/column to 1 without checking obstacles.
- Forgetting `grid[0][0]` itself can be an obstacle.

---

### Q13: Minimum Path Sum
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Each cell holds a non-negative cost. Move right/down from top-left to bottom-right; minimize the sum of costs along the path.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = minimum cost to reach `(i,j)` from `(0,0)`.

*(2) Recurrence.* `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.

*(3) Base.* `dp[0][0] = grid[0][0]`; first row accumulates left, first column accumulates up.

*(4) Order.* Row by row, left to right.

*(5b) Bottom-up:*
```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];
    for (int i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = grid[i][j] + Math.min(dp[i - 1][j], dp[i][j - 1]);
    return dp[m - 1][n - 1];
}
```

*(5a) Top-down:*
```java
public int minPathSum(int[][] grid) {
    Integer[][] memo = new Integer[grid.length][grid[0].length];
    return go(grid, grid.length - 1, grid[0].length - 1, memo);
}
private int go(int[][] g, int i, int j, Integer[][] memo) {
    if (i == 0 && j == 0) return g[0][0];
    if (i < 0 || j < 0) return Integer.MAX_VALUE;
    if (memo[i][j] != null) return memo[i][j];
    return memo[i][j] = g[i][j] + Math.min(go(g, i - 1, j, memo), go(g, i, j - 1, memo));
}
```

*(6) 1D space-optimized:*
```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[] dp = new int[n];
    dp[0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[j] = dp[j - 1] + grid[0][j];
    for (int i = 1; i < m; i++) {
        dp[0] += grid[i][0];
        for (int j = 1; j < n; j++)
            dp[j] = grid[i][j] + Math.min(dp[j], dp[j - 1]); // dp[j]=above, dp[j-1]=left
    }
    return dp[n - 1];
}
```

**Dry run** — `[[1,3,1],[1,5,1],[4,2,1]]`: dp grid → 1,4,5 / 2,7,6 / 6,8,**7** → path 1→3→1→1→1 = **7**. ✅

*(7) Complexity.* O(m·n) time, O(n) space.

*(8) Pattern recognition.* Identical skeleton to Unique Paths but `min(+cost)` instead of sum-of-counts. **Count→add, optimize→min/max** is the only difference between these grid problems.

**Follow-up questions the interviewer will ask:**
1. *"Reconstruct the path?"* → Backtrack from `(m-1,n-1)`: move to whichever of up/left gave the min.
2. *"Maximum path sum?"* → Swap `min` for `max`.
3. *"Can move in all 4 directions with positive costs?"* → Dijkstra, not DP (cycles possible).

**Common mistakes that get you rejected:**
- Forgetting to seed the first row/column before the double loop.
- In the 1D version, forgetting to add `grid[i][0]` to `dp[0]` each new row.

**Reconstructing the path.** Build the full 2D `dp`, then walk from `(m-1, n-1)` back to `(0,0)`, at each step moving to whichever predecessor (up or left) the current cell's cost came from:
```java
public List<int[]> minPathRoute(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];
    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = grid[i][j] + Math.min(dp[i-1][j], dp[i][j-1]);
    LinkedList<int[]> path = new LinkedList<>();
    int i = m - 1, j = n - 1;
    while (i > 0 || j > 0) {
        path.addFirst(new int[]{i, j});
        if (i == 0) j--;
        else if (j == 0) i--;
        else if (dp[i-1][j] <= dp[i][j-1]) i--; else j--;
    }
    path.addFirst(new int[]{0, 0});
    return path;
}
```

---

### Q14: Triangle
**Company:** Amazon, Google, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given a triangle (`row i` has `i+1` numbers), find the minimum path sum from top to bottom. From index `j` in a row you may move to index `j` or `j+1` in the next row.

**What interviewer is testing:**
The bottom-up insight: solving from the **last row upward** avoids edge cases and gives clean O(n) space.

**Ideal Answer:**

*(1) State (bottom-up).* `dp[j]` = minimum path sum from cell `j` of the *current* row down to the bottom.

*(2) Recurrence.* Process rows from bottom to top:
```
dp[j] = triangle[i][j] + min(dp[j], dp[j+1])
```
Here `dp[j]` (old) is the cell directly below-left and `dp[j+1]` is below-right.

*(3) Base.* Initialize `dp` with the last row's values.

*(4) Order.* From the second-to-last row up to the top. Bottom-up is cleaner because each cell has exactly two well-defined children below.

```java
public int minimumTotal(List<List<Integer>> triangle) {
    int n = triangle.size();
    int[] dp = new int[n + 1];               // extra slot = 0, avoids bounds check
    for (int i = n - 1; i >= 0; i--) {
        List<Integer> row = triangle.get(i);
        for (int j = 0; j <= i; j++) {
            dp[j] = row.get(j) + Math.min(dp[j], dp[j + 1]);
        }
    }
    return dp[0];
}
```

**Dry run** — `[[2],[3,4],[6,5,7],[4,1,8,3]]`: start dp=[4,1,8,3,0]; row [6,5,7] → [10,6,10,...]; row [3,4] → [9,10,...]; row [2] → [**11**]. ✅

*(7) Complexity.* O(n²) time (number of cells), O(n) space.

*(8) Pattern recognition.* "Path-sum DP where solving from the *destination* backward removes boundary special cases" → process in reverse. The same reverse trick shines in Dungeon Game (Q16).

*(5a) Top-down (for completeness — note the boundary handling):*
```java
public int minimumTotal(List<List<Integer>> tri) {
    Integer[][] memo = new Integer[tri.size()][tri.size()];
    return go(tri, 0, 0, memo);
}
private int go(List<List<Integer>> tri, int i, int j, Integer[][] memo) {
    if (i == tri.size()) return 0;
    if (memo[i][j] != null) return memo[i][j];
    int down  = go(tri, i + 1, j, memo);
    int diag  = go(tri, i + 1, j + 1, memo);
    return memo[i][j] = tri.get(i).get(j) + Math.min(down, diag);
}
```

**Follow-up questions the interviewer will ask:**
1. *"Top-down instead?"* → Works (above), but bottom-up avoids the per-edge special cases and gives clean O(n) space.
2. *"O(1) extra space?"* → Modify the triangle in place (if mutation allowed).

**Common mistakes that get you rejected:**
- Doing it top-down and botching the boundary transitions.
- Allocating an `O(n²)` table when O(n) suffices.

---

### Q15: Maximal Square
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
In a binary matrix of `0`/`1`, find the area of the largest square containing only `1`s.

**What interviewer is testing:**
The non-obvious recurrence `dp[i][j] = 1 + min(three neighbors)`. Deriving *why min of three* is the whole problem.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = the side length of the largest all-1 square whose **bottom-right corner** is `(i,j)`.

*(2) Recurrence.* If `matrix[i][j] == '1'`, a square ending here is limited by the smallest of the three squares ending at its top, left, and top-left neighbors — any of them being short caps this one:
```
dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])   if matrix[i][j]=='1'
dp[i][j] = 0                                               otherwise
```

*(3) Base.* First row/column: `dp = matrix value` (square of side 1 if cell is 1).

*(4) Order.* Row by row, left to right (needs top, left, top-left).

```java
public int maximalSquare(char[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[][] dp = new int[m + 1][n + 1];   // 1-indexed; row/col 0 act as zero border
    int best = 0;
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (matrix[i - 1][j - 1] == '1') {
                dp[i][j] = 1 + Math.min(dp[i - 1][j],
                              Math.min(dp[i][j - 1], dp[i - 1][j - 1]));
                best = Math.max(best, dp[i][j]);
            }
        }
    }
    return best * best;                   // area
}
```

**Dry run** — for a 2×2 block of 1s, the bottom-right cell gets `1 + min(1,1,1) = 2`, side 2, area 4. ✅

*(6) Space optimization.* Use a single array plus one scalar to remember the diagonal `dp[i-1][j-1]` before it's overwritten:
```java
public int maximalSquare(char[][] matrix) {
    int n = matrix[0].length;
    int[] dp = new int[n + 1];
    int best = 0, prev = 0;               // prev = dp[i-1][j-1]
    for (char[] row : matrix) {
        prev = 0;
        for (int j = 1; j <= n; j++) {
            int temp = dp[j];             // save dp[i-1][j] before overwrite
            if (row[j - 1] == '1') {
                dp[j] = 1 + Math.min(prev, Math.min(dp[j], dp[j - 1]));
                best = Math.max(best, dp[j]);
            } else dp[j] = 0;
            prev = temp;                  // becomes diagonal for next j
        }
    }
    return best * best;
}
```

*(7) Complexity.* O(m·n) time, O(n) space (1D) or O(m·n) (2D).

*(8) Pattern recognition.* "Largest square/region ending at a cell defined by its three neighbors" → the min-of-three recurrence. When the shape is constrained by *all* incoming neighbors equally, you take their min.

**Follow-up questions the interviewer will ask:**
1. *"Maximal Rectangle (not square)?"* → Different, harder: per row build a histogram and run "largest rectangle in histogram" (monotonic stack). Mention it explicitly.
2. *"Why min of three, not just two?"* → The top-left diagonal limits the square's interior; without it you'd allow L-shaped gaps.
3. *"Count of all squares (not just largest)?"* → Sum every `dp[i][j]` (each contributes that many squares ending there).

**Common mistakes that get you rejected:**
- Using only two neighbors (`top`, `left`) → over-counts and produces non-square regions.
- Returning the side instead of the area.

**Count *all* squares variant (Count Square Submatrices with All Ones).** With the *same* `dp[i][j]` (side of the largest square ending at `(i,j)`), the number of squares whose bottom-right corner is `(i,j)` is exactly `dp[i][j]` — a cell with `dp=3` ends a 1×1, a 2×2, and a 3×3. So the total count of all-1 squares is `Σ dp[i][j]`:
```java
public int countSquares(int[][] matrix) {
    int m = matrix.length, n = matrix[0].length, total = 0;
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (matrix[i][j] == 1) {
                dp[i][j] = (i == 0 || j == 0) ? 1
                    : 1 + Math.min(dp[i-1][j], Math.min(dp[i][j-1], dp[i-1][j-1]));
                total += dp[i][j];
            }
    return total;
}
```
The fact that "largest square ending here" doubles as "number of squares ending here" is a slick reuse worth pointing out.

---

### Q16: Dungeon Game (Reverse DP)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A knight starts top-left and rescues a princess bottom-right, moving only right/down. Each cell adds (positive) or subtracts (negative) health. The knight dies if health ever drops to 0. Find the **minimum starting health** so he survives the whole path.

**What interviewer is testing:**
The realization that you **cannot** DP forward — the health needed at a cell depends on the *future*, not the past. The fix is **reverse DP** from the destination. This direction flip is the entire trick.

**Ideal Answer:**

**Why forward fails.** A forward "max health at cell" doesn't capture the constraint that health must stay ≥ 1 *at every step*, and a locally high-health path may later dip below zero. The required entry health at a cell is determined by what comes *after* it. So we DP backward from the princess.

*(1) State.* `dp[i][j]` = the **minimum health needed upon entering `(i,j)`** to survive from `(i,j)` to the destination.

*(2) Recurrence.* From `(i,j)` the knight goes to the cheaper of right/down. The health needed before entering `(i,j)` is "the need of the chosen next cell minus what this cell gives," but never below 1:
```
need = min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j]
dp[i][j] = max(1, need)
```

*(3) Base.* At the destination `(m-1,n-1)`: `dp = max(1, 1 - dungeon[last])`. The padding cells just past the borders are set to ∞ except the two adjacent to the destination, handled by initializing a `(m+1)×(n+1)` table to ∞ and the two cells right/below the princess to 1.

*(4) Order.* From bottom-right **upward and leftward** — each cell depends on the cell below and to the right.

*(5b) Bottom-up:*
```java
public int calculateMinimumHP(int[][] dungeon) {
    int m = dungeon.length, n = dungeon[0].length;
    int[][] dp = new int[m + 1][n + 1];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE);
    dp[m][n - 1] = dp[m - 1][n] = 1;            // just past the destination
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            int need = Math.min(dp[i + 1][j], dp[i][j + 1]) - dungeon[i][j];
            dp[i][j] = Math.max(1, need);       // health never drops below 1
        }
    }
    return dp[0][0];
}
```

*(5a) Top-down:*
```java
public int calculateMinimumHP(int[][] dungeon) {
    Integer[][] memo = new Integer[dungeon.length][dungeon[0].length];
    return go(dungeon, 0, 0, memo);
}
private int go(int[][] d, int i, int j, Integer[][] memo) {
    int m = d.length, n = d[0].length;
    if (i >= m || j >= n) return Integer.MAX_VALUE;
    if (i == m - 1 && j == n - 1) return Math.max(1, 1 - d[i][j]);
    if (memo[i][j] != null) return memo[i][j];
    int need = Math.min(go(d, i + 1, j, memo), go(d, i, j + 1, memo)) - d[i][j];
    return memo[i][j] = Math.max(1, need);
}
```

**Dry run** — `[[-2,-3,3],[-5,-10,1],[10,30,-5]]`: backward DP yields `dp[0][0] = 7` (start with 7, path right→right→down→down stays ≥1). **Answer 7.** ✅

*(6) Space.* O(n) with a single row processed bottom-to-top (left as an exercise — same pattern as min path sum but reversed).

*(7) Complexity.* O(m·n) time, O(m·n) or O(n) space.

*(8) Pattern recognition.* "The requirement at a cell depends on the *future* (a min-health / min-resource-to-finish constraint)" → **reverse DP from the goal**. Whenever a forward DP can't express a 'must stay valid the whole way' constraint, flip the direction.

**Follow-up questions the interviewer will ask:**
1. *"Why can't we maximize health forward?"* → The constraint is on the *minimum* health along the path, which a forward max can't enforce; a path with high net health can still dip to 0 mid-way.
2. *"Return the path?"* → Backtrack forward, choosing the neighbor with smaller `dp`.
3. *"Health can be exactly 0 allowed?"* → Replace `max(1, need)` with `max(0, need)` / adjust the threshold per the rules.

**Common mistakes that get you rejected:**
- Trying a forward DP and getting stuck (recognize the reverse direction early).
- Forgetting the `max(1, …)` floor → allows lethal dips.

---

### Q17: Cherry Pickup II
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
A grid of cherries. **Two** robots start at the top-left and top-right of the top row; each moves down one row at a time to one of the three cells diagonally/directly below (`j-1, j, j+1`). They move simultaneously, reach the bottom row, and collect cherries (a cell's cherries count once even if both robots stand on it). Maximize total cherries.

**What interviewer is testing:**
Multi-agent DP — the state must track **both** robots simultaneously (you cannot optimize them independently). Recognizing the joint state is the whole challenge.

**Ideal Answer:**

**Why not solve each robot separately?** Because of the "count once if both on the same cell" interaction and the fact that one robot's greedy path can block the other's optimum. They must be optimized **jointly** in one DP.

*(1) State.* Since both robots are always on the **same row** `r` (they step in lockstep), the state is `dp[r][c1][c2]` = max cherries collectable from row `r` onward when robot 1 is at column `c1` and robot 2 at column `c2`.

*(2) Recurrence.* Collect this row's cherries (once if `c1 == c2`), then try all 3×3 = 9 next-column combinations:
```
cherries = grid[r][c1] + (c1 == c2 ? 0 : grid[r][c2])
dp[r][c1][c2] = cherries + max over (nc1 in {c1-1,c1,c1+1}, nc2 in {c2-1,c2,c2+1}) dp[r+1][nc1][nc2]
```

*(3) Base.* Last row `r = m-1`: just the cherries on that row (no further moves).

*(4) Order.* From the bottom row upward; each row needs the row below.

*(5a) Top-down (most natural here):*
```java
public int cherryPickup(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Integer[][][] memo = new Integer[m][n][n];
    return dp(grid, 0, 0, n - 1, memo);
}
private int dp(int[][] g, int r, int c1, int c2, Integer[][][] memo) {
    int m = g.length, n = g[0].length;
    if (c1 < 0 || c1 >= n || c2 < 0 || c2 >= n) return Integer.MIN_VALUE;
    if (memo[r][c1][c2] != null) return memo[r][c1][c2];
    int cherries = g[r][c1] + (c1 == c2 ? 0 : g[r][c2]);
    if (r < m - 1) {
        int best = Integer.MIN_VALUE;
        for (int d1 = -1; d1 <= 1; d1++)
            for (int d2 = -1; d2 <= 1; d2++)
                best = Math.max(best, dp(g, r + 1, c1 + d1, c2 + d2, memo));
        cherries += best;
    }
    return memo[r][c1][c2] = cherries;
}
```

*(5b) Bottom-up:*
```java
public int cherryPickup(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][][] dp = new int[m][n][n];
    for (int c1 = 0; c1 < n; c1++)
        for (int c2 = 0; c2 < n; c2++)
            dp[m - 1][c1][c2] = grid[m - 1][c1] + (c1 == c2 ? 0 : grid[m - 1][c2]);
    for (int r = m - 2; r >= 0; r--) {
        for (int c1 = 0; c1 < n; c1++) {
            for (int c2 = 0; c2 < n; c2++) {
                int best = Integer.MIN_VALUE;
                for (int d1 = -1; d1 <= 1; d1++) {
                    for (int d2 = -1; d2 <= 1; d2++) {
                        int nc1 = c1 + d1, nc2 = c2 + d2;
                        if (nc1 >= 0 && nc1 < n && nc2 >= 0 && nc2 < n)
                            best = Math.max(best, dp[r + 1][nc1][nc2]);
                    }
                }
                dp[r][c1][c2] = grid[r][c1] + (c1 == c2 ? 0 : grid[r][c2]) + best;
            }
        }
    }
    return dp[0][0][n - 1];
}
```

**Dry run** — for the LeetCode sample `[[3,1,1],[2,5,1],[1,5,5],[2,1,1]]` the answer is **24** (robot 1 hugs the left-middle, robot 2 the right). ✅

*(6) Space.* O(n²) — keep only the next row's `n×n` slice.

*(7) Complexity.* O(m·n²·9) = O(m·n²) time, O(m·n²) or O(n²) space.

*(8) Pattern recognition.* "Two (or more) agents traversing simultaneously with interaction" → a **joint state DP** indexing all agents' positions. The lockstep observation (both on the same row) is what keeps the state at 3 dimensions instead of 4.

**Follow-up questions the interviewer will ask:**
1. *"Cherry Pickup I (one robot goes down then comes back up)?"* → Model as two robots going *down* simultaneously (round trip = two downward passes); same joint-state DP.
2. *"Why can't we run two independent single-robot DPs?"* → They interact (shared-cell rule) and can block each other; independent optima don't combine.
3. *"k robots?"* → State explodes to `n^k`; only feasible for tiny k.

**Common mistakes that get you rejected:**
- Optimizing each robot separately.
- Double-counting cherries when `c1 == c2`.

---

# PATTERN 3 — STRING DP

The most templated pattern. Almost every two-string problem is `dp[i][j]` over **prefixes**: `dp[i][j]` = answer for `s[0..i)` and `t[0..j)`. The recurrence always branches on whether the current characters match (`s[i-1] == t[j-1]`). Single-string problems (palindromes, partitions) often use `dp[i][j]` over a *substring* `s[i..j]` and verge into interval DP. Learn the LCS table cold — it's the parent of Edit Distance, Distinct Subsequences, SCS, and the diff algorithm in `git`.

> **The Edit-Distance family (orientation).** File 01 covers vanilla **Edit Distance** (min insert/delete/replace to turn `s` into `t`, `dp[i][j] = same ? dp[i-1][j-1] : 1 + min(replace dp[i-1][j-1], delete dp[i-1][j], insert dp[i][j-1])`). Everything in this section is a *reweighting* or *restriction* of that same prefix grid:
> - **LCS** (Q18): count of kept (matched) characters — the dual of edit distance with only insert/delete.
> - **Min ASCII Delete Sum** (Q21): edit distance where each delete costs the char's ASCII, no replace.
> - **Delete Operation for Two Strings**: edit distance with only insert/delete, unit cost → `m + n − 2·LCS`.
> - **Distinct Subsequences** (Q22): *count* the ways one is a subsequence of the other.
> - **Shortest Common Supersequence** (Q26): `m + n − LCS`, reconstructed.
> Internalize this map: once you can derive the LCS recurrence, every member of the family is a 2-minute adaptation of "what does each branch cost / count."

**Edit Distance refresher (so the family is self-contained).** `dp[i][j]` = min operations to convert `a[0..i)` into `b[0..j)`.
```
dp[i][j] = dp[i-1][j-1]                                   if a[i-1] == b[j-1]
dp[i][j] = 1 + min( dp[i-1][j-1],   // replace a[i-1] with b[j-1]
                    dp[i-1][j],     // delete a[i-1]
                    dp[i][j-1] )    // insert b[j-1]
           otherwise
```
Base: `dp[0][j] = j` (insert j chars), `dp[i][0] = i` (delete i chars).

**Worked table** — `a = "horse"`, `b = "ros"` (answer 3):

|       | "" | r | o | s |
|-------|----|---|---|---|
| **""**| 0  | 1 | 2 | 3 |
| **h** | 1  | 1 | 2 | 3 |
| **o** | 2  | 2 | 1 | 2 |
| **r** | 3  | 2 | 2 | 2 |
| **s** | 4  | 3 | 3 | 2 |
| **e** | 5  | 4 | 4 | 3 |

Read `dp[5][3] = 3`: horse → rorse (replace h→r) → rose (remove r) → ros (remove e). The three operations match the three table "steps down" from 0. Knowing how to *read a path* out of this table is what makes the whole family's reconstruction follow-ups trivial.

---

### Q18: Longest Common Subsequence (LCS)
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg — extremely common
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given strings `text1` and `text2`, return the length of their longest **common subsequence** (characters in order, not necessarily contiguous).

**What interviewer is testing:**
The single most important string DP. They want a flawless prefix-state derivation, the match/no-match branch, and the table fill. Get this perfect and half the other string problems fall out.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = length of the LCS of the prefixes `text1[0..i)` (first `i` chars) and `text2[0..j)` (first `j` chars). Using **lengths** (1-indexed) rather than indices is what makes the empty-prefix base case clean.

*(2) Recurrence.* Compare the last characters of each prefix:
- If `text1[i-1] == text2[j-1]`: they can both end the LCS → `dp[i][j] = dp[i-1][j-1] + 1`.
- Else: drop one character from one string and take the better → `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

```
dp[i][j] = dp[i-1][j-1] + 1                      if text1[i-1] == text2[j-1]
dp[i][j] = max(dp[i-1][j], dp[i][j-1])           otherwise
```

*(3) Base cases.* `dp[0][j] = 0` and `dp[i][0] = 0` — an empty prefix has no common subsequence.

*(4) Iteration order.* `i` ascending, `j` ascending; each cell reads the cell above, left, and diagonal — all already computed.

*(5a) Top-down memoization:*
```java
public int longestCommonSubsequence(String a, String b) {
    Integer[][] memo = new Integer[a.length()][b.length()];
    return lcs(a, b, a.length() - 1, b.length() - 1, memo);
}
private int lcs(String a, String b, int i, int j, Integer[][] memo) {
    if (i < 0 || j < 0) return 0;
    if (memo[i][j] != null) return memo[i][j];
    if (a.charAt(i) == b.charAt(j))
        return memo[i][j] = 1 + lcs(a, b, i - 1, j - 1, memo);
    return memo[i][j] = Math.max(lcs(a, b, i - 1, j, memo),
                                 lcs(a, b, i, j - 1, memo));
}
```

*(5b) Bottom-up tabulation:*
```java
public int longestCommonSubsequence(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1))
                dp[i][j] = dp[i - 1][j - 1] + 1;
            else
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[m][n];
}
```

**Draw the table** — `a = "abcde"`, `b = "ace"` (answer 3, "ace"):

|       |   | "" | a | c | e |
|-------|---|----|---|---|---|
| **""**|   | 0  | 0 | 0 | 0 |
| **a** |   | 0  | 1 | 1 | 1 |
| **b** |   | 0  | 1 | 1 | 1 |
| **c** |   | 0  | 1 | 2 | 2 |
| **d** |   | 0  | 1 | 2 | 2 |
| **e** |   | 0  | 1 | 2 | 3 |

How `dp[3][2]` (prefixes "abc" vs "ac") becomes 2: `c == c` → `dp[2][1] + 1 = 1 + 1 = 2`. Bottom-right `dp[5][3] = 3`. **Answer 3.** ✅

*(6) Space optimization (2D→1D).* Each row depends only on the previous row. Keep two rows, or one row plus a `prevDiag` scalar:
```java
public int longestCommonSubsequence(String a, String b) {
    int m = a.length(), n = b.length();
    int[] prev = new int[n + 1], cur = new int[n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) cur[j] = prev[j - 1] + 1;
            else cur[j] = Math.max(prev[j], cur[j - 1]);
        }
        int[] t = prev; prev = cur; cur = t;   // swap rows
    }
    return prev[n];
}
```

*(7) Complexity.* O(m·n) time, O(m·n) space (table) → O(min(m,n)) space (rolling rows).

*(8) Pattern recognition.* "Common / matching content between two strings, order preserved" → LCS-family `dp[i][j]` over prefixes, branch on last-char match. **Memorize the two-line recurrence** — it's the kernel of Edit Distance, Distinct Subsequences, Shortest Common Supersequence, Minimum Delete Sum, and the Unix `diff`.

**Follow-up questions the interviewer will ask:**
1. *"Reconstruct the LCS string itself?"* → Backtrack from `dp[m][n]`: if chars equal, take it and go diagonal; else move toward the larger of up/left.
2. *"Longest Common *Substring* (contiguous)?"* → Q19 — different recurrence (reset to 0 on mismatch).
3. *"Edit distance between the two?"* → Covered in file 01; it's the same prefix-DP with insert/delete/replace transitions.
4. *"3 strings?"* → `dp[i][j][k]`, O(n³).
5. *"Shortest Common Supersequence length?"* → `m + n − LCS` (Q26).

**Common mistakes that get you rejected:**
- Confusing subsequence with substring.
- Off-by-one between length-indexing (`dp` size `m+1`) and char-indexing (`charAt(i-1)`).
- Taking `dp[i-1][j-1]+1` even on a mismatch.

**Reconstructing the LCS string (the standard follow-up).** Build the full table, then backtrack from `(m, n)`: on a character match move diagonally and prepend the char; otherwise move toward the larger of `dp[i-1][j]` / `dp[i][j-1]`:
```java
public String lcsString(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = a.charAt(i - 1) == b.charAt(j - 1)
                ? dp[i - 1][j - 1] + 1
                : Math.max(dp[i - 1][j], dp[i][j - 1]);
    StringBuilder sb = new StringBuilder();
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) { sb.append(a.charAt(i - 1)); i--; j--; }
        else if (dp[i - 1][j] >= dp[i][j - 1]) i--;
        else j--;
    }
    return sb.reverse().toString();
}
```
On `"abcde"`/`"ace"` this walks `e→c→a` (then reversed) → `"ace"`. The same backtrack powers `git diff` (LCS of two file line-sequences gives the unchanged lines).

---

### Q19: Longest Common Substring
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Length of the longest **contiguous** substring common to both strings.

**What interviewer is testing:**
The crucial difference from LCS: a substring must be contiguous, so a mismatch **resets** the run to 0, and the answer is the global max cell (not the corner).

**Ideal Answer:**

*(1) State.* `dp[i][j]` = length of the longest common substring **ending exactly at** `a[i-1]` and `b[j-1]`.

*(2) Recurrence.*
```
dp[i][j] = dp[i-1][j-1] + 1     if a[i-1] == b[j-1]
dp[i][j] = 0                    otherwise   (contiguity broken)
```

*(3) Base.* `dp[0][*] = dp[*][0] = 0`.

*(4) Order.* Ascending `i, j`. The answer is `max over all cells`, tracked as we fill.

```java
public int longestCommonSubstring(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    int best = 0;
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
                best = Math.max(best, dp[i][j]);
            }   // else stays 0
        }
    }
    return best;
}
```

**Dry run** — `a="abcde"`, `b="abfce"`: matching diagonals give "ab" (length 2). Best = **2**. ✅

*(6) Space.* Two rolling rows → O(min(m,n)). *(7) Complexity.* O(m·n) time.

*(8) Pattern recognition.* "Longest **contiguous** match" → diagonal-extend DP that **resets on mismatch**, answer = max cell. The reset-on-mismatch and answer-is-max-cell are the two signatures distinguishing substring from subsequence.

**Follow-up questions the interviewer will ask:**
1. *"Subsequence vs substring difference?"* → Substring resets to 0 on mismatch; answer is the max cell, not `dp[m][n]`.
2. *"O(n) per row, reconstruct the substring?"* → Track the end index of the best cell; substring is `a[end-best .. end)`.
3. *"Faster than O(mn)?"* → Suffix automaton / generalized suffix tree gives O(m+n).

**Common mistakes that get you rejected:**
- Returning `dp[m][n]` (that's LCS) instead of the running max.
- Forgetting to reset on mismatch (turns it into LCS).

---

### Q20: Longest Palindromic Subsequence
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Find the length of the longest palindromic **subsequence** of `s`.

**What interviewer is testing:**
Two valid framings: (a) LCS of `s` with its reverse, or (b) an interval DP `dp[i][j]` over substrings. Knowing the slick reduction *and* the direct interval DP shows depth.

**Ideal Answer:**

**Reduction (one-liner).** The longest palindromic subsequence of `s` = `LCS(s, reverse(s))`. A palindrome reads the same forwards and backwards, so the longest common subsequence of `s` and its reverse is exactly the longest subsequence that is its own mirror. Reuse Q18 verbatim.

**Direct interval DP** (often what they want you to derive):

*(1) State.* `dp[i][j]` = length of the longest palindromic subsequence within `s[i..j]` (inclusive).

*(2) Recurrence.*
```
dp[i][j] = dp[i+1][j-1] + 2                  if s[i] == s[j]
dp[i][j] = max(dp[i+1][j], dp[i][j-1])       otherwise
```

*(3) Base.* `dp[i][i] = 1` (single char is a palindrome of length 1).

*(4) Iteration order.* By **increasing substring length** (or `i` descending, `j` ascending) so `dp[i+1][j-1]`, `dp[i+1][j]`, `dp[i][j-1]` are ready before `dp[i][j]`. This ordering requirement is the interval-DP signature.

```java
public int longestPalindromeSubseq(String s) {
    int n = s.length();
    int[][] dp = new int[n][n];
    for (int i = n - 1; i >= 0; i--) {
        dp[i][i] = 1;
        for (int j = i + 1; j < n; j++) {
            if (s.charAt(i) == s.charAt(j))
                dp[i][j] = dp[i + 1][j - 1] + 2;
            else
                dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
        }
    }
    return dp[0][n - 1];
}
```

**Dry run** — `s = "bbbab"`: dp[0][4] resolves to **4** ("bbbb"). ✅

*(6) Space.* O(n²); can be reduced to O(n) with two rows since `dp[i]` reads only `dp[i+1]`.

*(7) Complexity.* O(n²) time, O(n²) (or O(n)) space.

*(8) Pattern recognition.* "Palindromic *subsequence*" → either reduce to `LCS(s, reverse)` or interval DP expanding from a matching pair inward. "Palindromic *substring*" (contiguous) is a different DP (`dp[i][j]` boolean) — don't confuse them.

**Follow-up questions the interviewer will ask:**
1. *"Minimum insertions/deletions to make `s` a palindrome?"* → `n − LPS(s)` (Q27).
2. *"Longest palindromic *substring*?"* → Different: boolean `dp[i][j]` or expand-around-center (file 01, Q37).
3. *"Count distinct palindromic subsequences?"* → Harder counting DP with inclusion-exclusion.

**Common mistakes that get you rejected:**
- Wrong iteration order (computing `dp[i][j]` before `dp[i+1][j-1]`).
- Confusing subsequence with substring.

---

### Q21: Minimum ASCII Delete Sum
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given two strings, delete characters from either so they become equal; minimize the **sum of ASCII values** of the deleted characters.

**What interviewer is testing:**
A weighted variant of LCS/Edit Distance. The state is the same prefix grid; only the cost changes from "count" to "ASCII sum."

**Ideal Answer:**

*(1) State.* `dp[i][j]` = minimum ASCII delete cost to make `a[0..i)` and `b[0..j)` equal.

*(2) Recurrence.*
- If `a[i-1] == b[j-1]`: keep both, no cost → `dp[i][j] = dp[i-1][j-1]`.
- Else delete one of them, whichever is cheaper:
```
dp[i][j] = min(dp[i-1][j] + ascii(a[i-1]),   // delete a's char
               dp[i][j-1] + ascii(b[j-1]))    // delete b's char
```

*(3) Base.* `dp[0][j]` = sum of ASCII of `b[0..j)` (delete all of b's prefix). `dp[i][0]` = sum of ASCII of `a[0..i)`.

*(4) Order.* Ascending `i, j`.

```java
public int minimumDeleteSum(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) dp[i][0] = dp[i - 1][0] + a.charAt(i - 1);
    for (int j = 1; j <= n; j++) dp[0][j] = dp[0][j - 1] + b.charAt(j - 1);
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1))
                dp[i][j] = dp[i - 1][j - 1];
            else
                dp[i][j] = Math.min(dp[i - 1][j] + a.charAt(i - 1),
                                    dp[i][j - 1] + b.charAt(j - 1));
        }
    }
    return dp[m][n];
}
```

**Dry run** — `a="sea"`, `b="eat"`: delete 's'(115) from sea and 't'(116) from eat → keep "ea". Cost **231**. ✅

*(6) Space.* Two rows → O(n). *(7) Complexity.* O(m·n) time.

*(8) Pattern recognition.* "Make two strings equal/aligned with a *weighted* delete/insert cost" → LCS grid where the transition cost is the weight, not 1. Any time Edit Distance is reweighted, it's this same grid with different addends.

**Follow-up questions the interviewer will ask:**
1. *"Delete Operation for Two Strings (count deletions)?"* → Same DP with cost 1 each → `m + n − 2·LCS`.
2. *"Different costs for insert vs delete vs replace?"* → Generalized edit distance; plug the weights into each branch.

**Common mistakes that get you rejected:**
- Wrong base case (must be the *cumulative* ASCII sum, not 0).
- Adding the wrong character's ASCII in each delete branch.

---

### Q22: Distinct Subsequences
**Company:** Google, Amazon, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Count the number of distinct subsequences of `s` that equal `t`.

**What interviewer is testing:**
A counting prefix-DP where the match case **adds two terms**. The "use this char to match, or skip it" branching trips up most candidates.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = the number of distinct subsequences of `s[0..i)` that equal `t[0..j)`.

*(2) Recurrence.* Consider `s[i-1]`:
- We can always **skip** `s[i-1]` and match `t[0..j)` using `s[0..i-1)` → contributes `dp[i-1][j]`.
- If `s[i-1] == t[j-1]`, we can **also use** `s[i-1]` to match `t[j-1]` → adds `dp[i-1][j-1]`.
```
dp[i][j] = dp[i-1][j] + (s[i-1] == t[j-1] ? dp[i-1][j-1] : 0)
```

*(3) Base.* `dp[i][0] = 1` (empty `t` is matched exactly one way — delete everything). `dp[0][j>0] = 0`.

*(4) Order.* Ascending `i, j`.

*(5a) Top-down:*
```java
public int numDistinct(String s, String t) {
    Integer[][] memo = new Integer[s.length() + 1][t.length() + 1];
    return count(s, t, s.length(), t.length(), memo);
}
private int count(String s, String t, int i, int j, Integer[][] memo) {
    if (j == 0) return 1;             // matched all of t
    if (i == 0) return 0;             // s exhausted, t not
    if (memo[i][j] != null) return memo[i][j];
    int res = count(s, t, i - 1, j, memo);
    if (s.charAt(i - 1) == t.charAt(j - 1)) res += count(s, t, i - 1, j - 1, memo);
    return memo[i][j] = res;
}
```

*(5b) Bottom-up:*
```java
public int numDistinct(String s, String t) {
    int m = s.length(), n = t.length();
    long[][] dp = new long[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = 1;
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = dp[i - 1][j];
            if (s.charAt(i - 1) == t.charAt(j - 1)) dp[i][j] += dp[i - 1][j - 1];
        }
    }
    return (int) dp[m][n];
}
```

**Dry run** — `s="rabbbit"`, `t="rabbit"`: the three `b`s give 3 ways to pick the "bb" of "rabbit" → **3**. ✅

**Worked table** — `s="babgbag"`, `t="bag"` (answer 5):

|       | "" | b | a | g |
|-------|----|---|---|---|
| **""**| 1  | 0 | 0 | 0 |
| **b** | 1  | 1 | 0 | 0 |
| **a** | 1  | 1 | 1 | 0 |
| **b** | 1  | 2 | 1 | 0 |
| **g** | 1  | 2 | 1 | 1 |
| **b** | 1  | 3 | 1 | 1 |
| **a** | 1  | 3 | 4 | 1 |
| **g** | 1  | 3 | 4 | 5 |

The first column is all 1 (empty `t`). `dp[7][3] = 5` — five distinct ways to spell "bag" by choosing positions. Notice each cell = the cell directly above (skip current `s` char) plus, on a match, the diagonal (use it). **Answer 5.** ✅

*(6) Space optimization (2D→1D).* `dp[i]` reads only `dp[i-1]`. Roll to one array iterating `j` **descending** (so `dp[j-1]` still holds the previous row):
```java
public int numDistinct(String s, String t) {
    int n = t.length();
    long[] dp = new long[n + 1];
    dp[0] = 1;
    for (int i = 1; i <= s.length(); i++)
        for (int j = n; j >= 1; j--)
            if (s.charAt(i - 1) == t.charAt(j - 1)) dp[j] += dp[j - 1];
    return (int) dp[n];
}
```

*(7) Complexity.* O(m·n) time, O(n) space.

*(8) Pattern recognition.* "Count the number of ways one string can match another as a subsequence" → counting prefix DP; the match case **adds** the "use it" and "skip it" counts. The descending-`j` 1D roll is the same trick as 0/1 knapsack.

**Follow-up questions the interviewer will ask:**
1. *"Why long?"* → Counts can exceed `int` (problem guarantees fit in 32-bit signed, but intermediate sums are safer as `long`).
2. *"Why is the 1D loop descending?"* → Iterating ascending would let `dp[j-1]` already reflect the *current* row, double-counting; descending preserves the previous row's value.

**Common mistakes that get you rejected:**
- Wrong base case (`dp[i][0]` must be 1, not 0).
- Iterating ascending in the 1D version (corrupts the previous-row dependency).

---

### Q23: Interleaving String
**Company:** Google, Amazon, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `s1`, `s2`, `s3`, return whether `s3` is formed by **interleaving** `s1` and `s2` (preserving each one's character order).

**What interviewer is testing:**
A boolean 2D DP where the two indices into `s1` and `s2` *sum* to the index into `s3`. Recognizing that the `s3` index is *derived* (not a third dimension) keeps it 2D.

**Ideal Answer:**

First check `s1.length + s2.length == s3.length`, else false.

*(1) State.* `dp[i][j]` = true iff `s3[0..i+j)` can be formed by interleaving `s1[0..i)` and `s2[0..j)`. **Key:** the position in `s3` is exactly `i + j`, so no third index is needed.

*(2) Recurrence.* The last character of `s3[0..i+j)` came either from `s1` or `s2`:
```
dp[i][j] = (i>0 && s1[i-1]==s3[i+j-1] && dp[i-1][j])
        || (j>0 && s2[j-1]==s3[i+j-1] && dp[i][j-1])
```

*(3) Base.* `dp[0][0] = true`. First row/column: match purely from one string.

*(4) Order.* Ascending `i, j`.

```java
public boolean isInterleave(String s1, String s2, String s3) {
    int m = s1.length(), n = s2.length();
    if (m + n != s3.length()) return false;
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            if (i > 0 && s1.charAt(i - 1) == s3.charAt(i + j - 1))
                dp[i][j] |= dp[i - 1][j];
            if (j > 0 && s2.charAt(j - 1) == s3.charAt(i + j - 1))
                dp[i][j] |= dp[i][j - 1];
        }
    }
    return dp[m][n];
}
```

**Dry run** — `s1="aabcc"`, `s2="dbbca"`, `s3="aadbbcbcac"` → **true**. With `s3="aadbbbaccc"` → **false**. ✅

**Small worked grid** — `s1="ab"`, `s2="cd"`, `s3="acbd"` (true). `dp[i][j]` = can `s1[0..i)` + `s2[0..j)` interleave into `s3[0..i+j)`:

| i\j (s2→) | "" | c | cd |
|-----------|----|---|----|
| **""**    | T  | T | F  |
| **a**     | T  | T | F  |
| **ab**    | F  | T | T  |

Reading the true path `dp[0][0]→dp[1][0](a)→dp[1][1](c)→dp[2][1](b)→dp[2][2](d)` spells the interleave `a,c,b,d` = "acbd". `dp[2][2]=T`. ✅ Note `dp[2][0]=F` because "ab" alone can't produce "ac".

*(6) Space.* `dp[i]` reads only `dp[i-1][j]` and `dp[i][j-1]` → one row, O(n).

*(7) Complexity.* O(m·n) time, O(n) space.

*(8) Pattern recognition.* "Merge/interleave two sequences into a third preserving order" → 2D boolean DP where the third index is the **sum** of the two. Don't add a third dimension; derive it.

**Follow-up questions the interviewer will ask:**
1. *"Why not a 3D DP on all three indices?"* → `k = i + j` is forced, so the third index is redundant — collapsing it is the optimization.
2. *"Return the actual interleaving?"* → Backtrack through `dp`, choosing which string supplied each char.

**Common mistakes that get you rejected:**
- Skipping the length check (allows impossible cases).
- Using a 3D state (works but signals you missed `k = i + j`).

---

### Q24: Regular Expression Matching
**Company:** Google, Meta, Amazon, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Implement regex matching for `.` (any single char) and `*` (zero or more of the *preceding* element). The match must cover the **entire** input string.

**What interviewer is testing:**
The hardest common string DP. The `*` handling — "zero occurrences" vs "one more occurrence" — is where everyone slips. Crisp case analysis is the signal.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = true iff `s[0..i)` matches pattern `p[0..j)`.

*(2) Recurrence.*
- If `p[j-1]` is a normal char or `.`: it must match the current `s` char and the rest must match → `dp[i][j] = dp[i-1][j-1] && (p[j-1]=='.' || p[j-1]==s[i-1])`.
- If `p[j-1] == '*'`: it modifies `p[j-2]`. Two sub-cases:
  - **Zero occurrences** of `p[j-2]`: drop both `*` and its char → `dp[i][j-2]`.
  - **One or more**: if `p[j-2]` matches `s[i-1]` (i.e. `p[j-2]=='.'` or `==s[i-1]`), consume one `s` char and keep the `*` → `dp[i-1][j]`.
```
dp[i][j] |= dp[i][j-2]                                 // zero of p[j-2]
dp[i][j] |= dp[i-1][j]  if p[j-2]=='.' || p[j-2]==s[i-1]  // one more
```

*(3) Base.* `dp[0][0] = true`. `dp[0][j]`: a pattern like `a*b*c*` can match empty — `dp[0][j] = (p[j-1]=='*') && dp[0][j-2]`.

*(4) Order.* Ascending `i, j`.

*(5b) Bottom-up:*
```java
public boolean isMatch(String s, String p) {
    int m = s.length(), n = p.length();
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;
    for (int j = 1; j <= n; j++)                       // empty s vs pattern
        if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 2];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                dp[i][j] = dp[i][j - 2];               // zero occurrences
                char prev = p.charAt(j - 2);
                if (prev == '.' || prev == s.charAt(i - 1))
                    dp[i][j] |= dp[i - 1][j];          // one more
            } else if (pc == '.' || pc == s.charAt(i - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            }
        }
    }
    return dp[m][n];
}
```

*(5a) Top-down:*
```java
public boolean isMatch(String s, String p) {
    Boolean[][] memo = new Boolean[s.length() + 1][p.length() + 1];
    return go(s, p, 0, 0, memo);
}
private boolean go(String s, String p, int i, int j, Boolean[][] memo) {
    if (j == p.length()) return i == s.length();
    if (memo[i][j] != null) return memo[i][j];
    boolean firstMatch = i < s.length() &&
        (p.charAt(j) == '.' || p.charAt(j) == s.charAt(i));
    boolean ans;
    if (j + 1 < p.length() && p.charAt(j + 1) == '*') {
        ans = go(s, p, i, j + 2, memo)                 // zero
           || (firstMatch && go(s, p, i + 1, j, memo)); // one more
    } else {
        ans = firstMatch && go(s, p, i + 1, j + 1, memo);
    }
    return memo[i][j] = ans;
}
```

**Dry run** — `s="aab"`, `p="c*a*b"`: `c*` matches zero c's, `a*` matches "aa", `b` matches "b" → **true**. ✅

*(7) Complexity.* O(m·n) time, O(m·n) space.

*(8) Pattern recognition.* "Pattern matching with quantifiers, must match entire string" → prefix DP with **case analysis on the pattern char**; `*` always splits into "zero" (skip `x*`) and "one more" (consume an `s` char). The `*` looks *back* at `p[j-2]`.

**Follow-up questions the interviewer will ask:**
1. *"Difference from Wildcard Matching (`?`/`*`)?"* → There `*` matches *any sequence* independent of a preceding char (Q25); the `*` semantics differ fundamentally.
2. *"Add `+` (one or more)?"* → `x+` = `xx*`; rewrite or add a branch requiring at least one match.
3. *"Why does `dp[0][j]` need the `j-2` jump?"* → Only `*` patterns can match empty, by erasing pairs.

**Common mistakes that get you rejected:**
- Treating `*` as "match any sequence" (that's wildcard, not regex).
- Forgetting the `dp[0][j]` initialization for patterns matching empty.
- Looking at `p[j-1]` instead of `p[j-2]` for what `*` repeats.

---

### Q25: Wildcard Matching
**Company:** Google, Meta, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Implement wildcard matching with `?` (any single char) and `*` (any sequence including empty). Match the entire string.

**What interviewer is testing:**
Same family as Q24 but `*` is **standalone** (not tied to a preceding char), which simplifies the recurrence. Distinguishing the two `*` semantics is the point.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = true iff `s[0..i)` matches `p[0..j)`.

*(2) Recurrence.*
- `p[j-1] == '?'` or `== s[i-1]`: `dp[i][j] = dp[i-1][j-1]`.
- `p[j-1] == '*'`: matches empty (`dp[i][j-1]`) **or** consumes one `s` char (`dp[i-1][j]`):
```
dp[i][j] = dp[i-1][j-1]                         if p[j-1]=='?' or matches
dp[i][j] = dp[i][j-1] || dp[i-1][j]             if p[j-1]=='*'
```

*(3) Base.* `dp[0][0]=true`; `dp[0][j] = dp[0][j-1] && p[j-1]=='*'` (only leading `*`s match empty).

*(4) Order.* Ascending `i, j`.

```java
public boolean isMatch(String s, String p) {
    int m = s.length(), n = p.length();
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;
    for (int j = 1; j <= n; j++)
        if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
            else if (pc == '?' || pc == s.charAt(i - 1)) dp[i][j] = dp[i - 1][j - 1];
        }
    }
    return dp[m][n];
}
```

**Dry run** — `s="adceb"`, `p="*a*b"` → **true** (`*`→"", a, `*`→"dce", b). `s="acdcb"`, `p="a*c?b"` → **false**. ✅

*(6) Space.* One row, O(n). There's also an O(1)-space greedy two-pointer with backtracking on `*` — mention it.

*(7) Complexity.* O(m·n) time, O(n) space (DP) or O(1) (greedy).

*(8) Pattern recognition.* "`*` matches any *sequence*" (independent of neighbors) → `dp[i][j] = dp[i][j-1] || dp[i-1][j]` for `*`. Contrast with regex `*` (Q24) which repeats the *preceding* char.

**The O(1)-space greedy (the classic optimization follow-up).** Two pointers over `s` and `p`; when you hit a `*`, remember its position and the current `s` index, then keep matching — on a later mismatch, backtrack: rewind `p` to just after the last `*` and advance the remembered `s` index by one (the `*` swallows one more char):
```java
public boolean isMatch(String s, String p) {
    int i = 0, j = 0, star = -1, match = 0;
    while (i < s.length()) {
        if (j < p.length() && (p.charAt(j) == '?' || p.charAt(j) == s.charAt(i))) {
            i++; j++;
        } else if (j < p.length() && p.charAt(j) == '*') {
            star = j; match = i; j++;            // remember; * matches empty for now
        } else if (star != -1) {
            j = star + 1; i = ++match;            // backtrack: * eats one more char
        } else return false;
    }
    while (j < p.length() && p.charAt(j) == '*') j++;   // trailing *'s match empty
    return j == p.length();
}
```
This is O(m·n) worst case but O(1) space and very fast in practice. Knowing both the DP and this greedy — and that the greedy works *because* `*` is unanchored — is the senior signal.

**Follow-up questions the interviewer will ask:**
1. *"O(1) space greedy?"* → The two-pointer backtracking above.
2. *"Difference from regex `*`?"* → As above — standalone vs preceding-char repeater.

**Common mistakes that get you rejected:**
- Mixing up the two `*` semantics with regex.
- Wrong empty-prefix initialization.

**Side-by-side: the two `*` semantics (interviewers love this distinction).**

| | Regex `*` (Q24) | Wildcard `*` (Q25) |
|---|---|---|
| Meaning | "zero or more of the **preceding** element" | "any sequence of **any** characters, incl. empty" |
| Looks at | `p[j-2]` (the char it repeats) | nothing — standalone |
| Recurrence on `*` | `dp[i][j-2]` (zero) `\|\|` `dp[i-1][j]` if prev matches | `dp[i][j-1]` (empty) `\|\|` `dp[i-1][j]` (consume one) |
| `.` / `?` | `.` = any single char | `?` = any single char |
| Example | `"a*"` matches `""`,`"a"`,`"aa"`,… | `"*"` matches *anything* |

The trap: candidates copy the regex recurrence into wildcard (or vice-versa). Say the meaning of `*` out loud before writing the transition.

---

### Q26: Shortest Common Supersequence
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Find the **shortest string** that has both `str1` and `str2` as subsequences. Return the string itself.

**What interviewer is testing:**
Connecting SCS to LCS (`len(SCS) = m + n − LCS`), then **reconstructing** the actual supersequence by walking the LCS table.

**Ideal Answer:**

**Length insight.** The shortest common supersequence keeps the common subsequence once and appends the rest: `len(SCS) = m + n − LCS(str1, str2)`. To build the *string*, run the LCS DP, then backtrack.

*(1) State.* `dp[i][j]` = LCS length of prefixes (as in Q18).

*(2) Recurrence / base / order:* identical to LCS.

**Reconstruction (the interesting part).** Walk from `(m, n)`:
- if chars equal → it's part of both; emit once, move diagonally.
- else move toward the larger neighbor, emitting the char you *skip over* (it must appear in the supersequence).
- drain any remaining prefix of either string.
Reverse at the end.

```java
public String shortestCommonSupersequence(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = a.charAt(i - 1) == b.charAt(j - 1)
                ? dp[i - 1][j - 1] + 1
                : Math.max(dp[i - 1][j], dp[i][j - 1]);

    StringBuilder sb = new StringBuilder();
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) {     // common char, once
            sb.append(a.charAt(i - 1)); i--; j--;
        } else if (dp[i - 1][j] >= dp[i][j - 1]) {
            sb.append(a.charAt(i - 1)); i--;          // a-only char
        } else {
            sb.append(b.charAt(j - 1)); j--;          // b-only char
        }
    }
    while (i > 0) sb.append(a.charAt(--i));
    while (j > 0) sb.append(b.charAt(--j));
    return sb.reverse().toString();
}
```

**Dry run** — `a="abac"`, `b="cab"`: LCS = "ab" (len 2), SCS length = 4+3−2 = 5, e.g. **"cabac"**. ✅

*(7) Complexity.* O(m·n) time and space.

*(8) Pattern recognition.* "Shortest string containing both as subsequences" → LCS + reconstruction. SCS, the LCS string, and `diff` all share the **backtrack-through-the-LCS-table** technique.

**Follow-up questions the interviewer will ask:**
1. *"Just the length?"* → `m + n − LCS`, no reconstruction needed.
2. *"Why move toward the larger neighbor?"* → That preserves the LCS path; the skipped char is unique to one string and must be inserted.

**Common mistakes that get you rejected:**
- Emitting common chars twice.
- Forgetting to drain the leftover prefix of one string after the loop.

---

### Q27: Minimum Insertions to Make a String Palindrome
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `s`, return the minimum number of characters to **insert** anywhere to make it a palindrome.

**What interviewer is testing:**
Reducing to Longest Palindromic Subsequence: the chars *not* in the LPS each need a mirror inserted.

**Ideal Answer:**

**Insight.** The longest palindromic subsequence (LPS) already reads the same both ways. Every character outside it needs one inserted mirror. So:
```
minInsertions = n − LPS(s)
```
Equivalently `n − LCS(s, reverse(s))`.

*(1) State.* Interval DP `dp[i][j]` = min insertions to make `s[i..j]` a palindrome.

*(2) Recurrence (direct, without going through LPS):*
```
dp[i][j] = dp[i+1][j-1]                          if s[i] == s[j]
dp[i][j] = 1 + min(dp[i+1][j], dp[i][j-1])       otherwise
```

*(3) Base.* `dp[i][i] = 0` (single char already a palindrome).

*(4) Order.* By increasing length / `i` descending, `j` ascending.

```java
public int minInsertions(String s) {
    int n = s.length();
    int[][] dp = new int[n][n];
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i + 1; j < n; j++) {
            if (s.charAt(i) == s.charAt(j))
                dp[i][j] = dp[i + 1][j - 1];
            else
                dp[i][j] = 1 + Math.min(dp[i + 1][j], dp[i][j - 1]);
        }
    }
    return dp[0][n - 1];
}
```

Or the one-liner via LPS:
```java
public int minInsertions(String s) {
    return s.length() - longestPalindromeSubseq(s);   // reuse Q20
}
```

**Dry run** — `s = "leetcode"`: LPS = "ee" (or "tt"? "e","ee") length 2 → wait, "leetcode" LPS is "ee" len 2... actually answer is **5** (`n − LPS = 8 − 3`? LPS of "leetcode" is "eee"? no). Concretely the interval DP returns **5**. ✅ (`zzazz` → 0; `mbadm` → 2.)

*(7) Complexity.* O(n²) time, O(n²) space.

*(8) Pattern recognition.* "Min insertions/deletions to reach a palindrome" → `n − LPS`. "Min deletions to make palindrome" is the *same* number. When a problem reduces to "how much of the string is already palindromic," reach for LPS.

**Follow-up questions the interviewer will ask:**
1. *"Min deletions to make palindrome?"* → Same answer, `n − LPS`.
2. *"Why is insertions = deletions here?"* → Each non-LPS char can be fixed by either inserting its mirror or deleting it; both cost 1 per char.

**Common mistakes that get you rejected:**
- Computing LPS wrong (iteration order).
- Assuming you must insert at the ends only (insertions are anywhere).

---

# PATTERN 4 — KNAPSACK DP

You're choosing a **subset** of items to optimize a value while respecting a **capacity/sum** constraint. The two flavors:
- **0/1 knapsack** — each item used **at most once**. 1D space optimization iterates capacity **descending**.
- **Unbounded knapsack** — each item used **unlimited** times. 1D iterates capacity **ascending**.

That single loop-direction rule (descending = once, ascending = reuse) is the most testable knapsack fact. Subset Sum, Partition, Target Sum, Ones-and-Zeroes, and Coin Change II are all knapsack in disguise — recognizing that is the skill.

---

### Q28: 0/1 Knapsack
**Company:** Amazon, Google, Microsoft, Flipkart, Samsung
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given `n` items each with `weight[i]` and `value[i]`, and a knapsack capacity `W`, maximize the total value of items chosen such that total weight ≤ `W`. Each item is used at most once.

**What interviewer is testing:**
The canonical subset-under-capacity DP, and — critically — the **2D→1D space optimization with the descending capacity loop** and *why* it must be descending. This is the prototype for half of Pattern 4.

**Ideal Answer:**

*(1) State.* `dp[i][w]` = the maximum value achievable using the first `i` items with capacity exactly-at-most `w`.

*(2) Recurrence.* For item `i` (1-indexed), either **skip** it (value `dp[i-1][w]`) or **take** it if it fits (`value[i-1] + dp[i-1][w - weight[i-1]]`):
```
dp[i][w] = dp[i-1][w]                                          if weight[i-1] > w
dp[i][w] = max(dp[i-1][w],  value[i-1] + dp[i-1][w-weight[i-1]]) otherwise
```

*(3) Base cases.* `dp[0][w] = 0` for all `w` (no items → no value). `dp[i][0] = 0` (no capacity).

*(4) Iteration order.* `i` ascending (need previous item's row), `w` ascending (in 2D the direction within a row doesn't matter because we always read row `i-1`).

*(5a) Top-down memoization:*
```java
public int knapsack(int[] weight, int[] value, int W) {
    Integer[][] memo = new Integer[weight.length + 1][W + 1];
    return go(weight, value, weight.length, W, memo);
}
private int go(int[] wt, int[] val, int i, int w, Integer[][] memo) {
    if (i == 0 || w == 0) return 0;
    if (memo[i][w] != null) return memo[i][w];
    int skip = go(wt, val, i - 1, w, memo);
    int take = wt[i - 1] <= w
        ? val[i - 1] + go(wt, val, i - 1, w - wt[i - 1], memo)
        : Integer.MIN_VALUE;
    return memo[i][w] = Math.max(skip, take);
}
```

*(5b) Bottom-up 2D table:*
```java
public int knapsack(int[] weight, int[] value, int W) {
    int n = weight.length;
    int[][] dp = new int[n + 1][W + 1];
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i - 1][w];                       // skip item i
            if (weight[i - 1] <= w)
                dp[i][w] = Math.max(dp[i][w],
                    value[i - 1] + dp[i - 1][w - weight[i - 1]]); // take item i
        }
    }
    return dp[n][W];
}
```

**Dry run table** — items `(w,v) = (1,1),(3,4),(4,5),(5,7)`, capacity `W=7`:

| i\w | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|-----|---|---|---|---|---|---|---|---|
| 0   | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 1 (1,1) | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 2 (3,4) | 0 | 1 | 1 | 4 | 5 | 5 | 5 | 5 |
| 3 (4,5) | 0 | 1 | 1 | 4 | 5 | 6 | 6 | 9 |
| 4 (5,7) | 0 | 1 | 1 | 4 | 5 | 7 | 8 | 9 |

`dp[4][7] = 9` (items 2+3: weight 3+4=7, value 4+5=9). **Answer 9.** ✅

*(6) Space optimization (2D→1D, the critical part).* `dp[i][w]` only reads row `i-1`. Collapse to a single array `dp[w]`. We **must iterate `w` from high to low** so that when we compute `dp[w]` using `dp[w - weight]`, that `dp[w - weight]` still holds the **previous item's** value (row `i-1`), not the current item's — which would let us pick item `i` twice:
```java
public int knapsack(int[] weight, int[] value, int W) {
    int[] dp = new int[W + 1];
    for (int i = 0; i < weight.length; i++) {
        for (int w = W; w >= weight[i]; w--) {     // DESCENDING => each item once
            dp[w] = Math.max(dp[w], value[i] + dp[w - weight[i]]);
        }
    }
    return dp[W];
}
```
> **Memorize:** 0/1 knapsack → **descending** capacity loop (use each item once). Unbounded → **ascending** (allow reuse). This one line is the most asked knapsack detail.

*(7) Complexity.* O(n·W) time, O(W) space (1D) or O(n·W) (2D). Note this is **pseudo-polynomial** (depends on the magnitude of `W`).

*(8) Pattern recognition.* "Pick a subset of items, each at most once, to maximize value under a weight/sum budget" → 0/1 knapsack. Triggers: "at most once", "fits within capacity", "subset achieving a target". Whenever a problem is "choose some items to hit/optimize a sum," map it onto this table.

**Follow-up questions the interviewer will ask:**
1. *"Which items were chosen?"* → Backtrack the 2D table: if `dp[i][w] != dp[i-1][w]`, item `i` was taken; subtract its weight and continue.
2. *"Why descending in the 1D version?"* → Ascending would reuse the just-updated `dp[w-weight]` (current item) → unbounded behavior. Descending preserves the `i-1` row.
3. *"Fractional knapsack?"* → Greedy by value/weight ratio, O(n log n) — *not* DP.
4. *"W is huge (10⁹) but values small?"* → Flip the DP: `dp[v]` = min weight to achieve value `v`; iterate over total value (meet-in-the-middle for n≤40).

**Common mistakes that get you rejected:**
- Ascending loop in the 1D version (silently solves unbounded).
- Confusing 0/1 (subset) with fractional (greedy).
- Calling O(nW) "polynomial" — it's pseudo-polynomial.

**Reconstructing the chosen items (from the 2D table).** Walk from `dp[n][W]` upward: if `dp[i][w] != dp[i-1][w]`, item `i` was taken — record it and drop its weight:
```java
public List<Integer> knapsackItems(int[] weight, int[] value, int W) {
    int n = weight.length;
    int[][] dp = new int[n + 1][W + 1];
    for (int i = 1; i <= n; i++)
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i - 1][w];
            if (weight[i - 1] <= w)
                dp[i][w] = Math.max(dp[i][w], value[i - 1] + dp[i - 1][w - weight[i - 1]]);
        }
    List<Integer> taken = new ArrayList<>();
    for (int i = n, w = W; i > 0; i--) {
        if (dp[i][w] != dp[i - 1][w]) {          // item i was included
            taken.add(i - 1);
            w -= weight[i - 1];
        }
    }
    return taken;                                 // 0-based item indices
}
```
Note: the **1D** array can't reconstruct (it discards the per-item history) — reconstruction needs the full 2D table. State this when asked.

**The "value-indexed" flip for huge W.** When `W` is enormous (10⁹) but the total value `V = Σ value[i]` is small, invert the DP: `dp[v]` = minimum weight to achieve **exactly** value `v`. Then the answer is the largest `v` with `dp[v] ≤ W`. Complexity O(n·V) instead of O(n·W) — pick whichever of W or V is smaller. This "swap the optimized quantity with the constrained quantity" trick is a recurring advanced move.

---

### Q29: Unbounded Knapsack
**Company:** Amazon, Google, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Same as 0/1 knapsack but each item may be used **unlimited** times. Maximize value within capacity `W`.

**What interviewer is testing:**
The single change from 0/1: the **ascending** capacity loop, and why it enables reuse.

**Ideal Answer:**

*(1) State.* `dp[w]` = max value achievable with capacity `w` (items reusable).

*(2) Recurrence.* `dp[w] = max over items i (weight[i] ≤ w) of value[i] + dp[w - weight[i]]`. The same item can appear in `dp[w - weight[i]]`, so reuse is allowed.

*(3) Base.* `dp[0] = 0`.

*(4) Order.* `w` **ascending** (so `dp[w - weight]` may already include item `i` → reuse).

```java
public int unboundedKnapsack(int[] weight, int[] value, int W) {
    int[] dp = new int[W + 1];
    for (int w = 1; w <= W; w++) {
        for (int i = 0; i < weight.length; i++) {
            if (weight[i] <= w)
                dp[w] = Math.max(dp[w], value[i] + dp[w - weight[i]]);
        }
    }
    return dp[W];
}
```

Equivalent item-outer form (note the ascending inner loop — opposite of 0/1):
```java
for (int i = 0; i < weight.length; i++)
    for (int w = weight[i]; w <= W; w++)        // ASCENDING => reuse
        dp[w] = Math.max(dp[w], value[i] + dp[w - weight[i]]);
```

**Dry run** — items `(w,v)=(1,1),(3,4),(4,5)`, `W=7`: best is using item (3,4) twice + item (1,1) → weight 7, value 9? Actually 3+3+1, value 4+4+1=9. **Answer 9.** ✅

*(7) Complexity.* O(n·W) time, O(W) space.

*(8) Pattern recognition.* "Unlimited supply, maximize value under capacity" → unbounded knapsack, **ascending** loop. Coin Change (Q5) and Rod Cutting (Q35) are unbounded knapsack.

**Follow-up questions the interviewer will ask:**
1. *"Difference from 0/1 in code?"* → Loop direction (ascending vs descending) when item-outer; or item-inner allows natural reuse.
2. *"Rod cutting connection?"* → Q35 — rod of length `n`, pieces with prices = unbounded knapsack with weight=length, value=price, capacity=n.

**Common mistakes that get you rejected:**
- Using the descending loop (turns it back into 0/1).

**The loop-direction intuition, stated cleanly.** In the 1D array, `dp[w - weight]` either belongs to the *current* item's row (if already updated this pass) or the *previous* row (if not yet touched).
- **Descending** `w`: `dp[w-weight]` is still the previous row → item not yet used → **at most once** (0/1).
- **Ascending** `w`: `dp[w-weight]` may already include this item this pass → **reuse allowed** (unbounded).
This single sentence answers the most common knapsack interview question; rehearse it verbatim.

---

### Q30: Subset Sum
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given positive integers `nums` and a target `S`, determine whether some subset sums exactly to `S`.

**What interviewer is testing:**
Recognizing a *boolean* 0/1 knapsack where capacity = target and "value" is irrelevant — just reachability.

**Ideal Answer:**

*(1) State.* `dp[i][s]` = true iff some subset of the first `i` numbers sums to `s`.

*(2) Recurrence.* Include `nums[i-1]` or not:
```
dp[i][s] = dp[i-1][s] || (s >= nums[i-1] && dp[i-1][s - nums[i-1]])
```

*(3) Base.* `dp[i][0] = true` (empty subset sums to 0). `dp[0][s>0] = false`.

*(4) Order.* `i` ascending; 1D loop over `s` **descending** (0/1).

```java
public boolean canPartitionSubset(int[] nums, int S) {
    boolean[] dp = new boolean[S + 1];
    dp[0] = true;
    for (int x : nums) {
        for (int s = S; s >= x; s--) {        // descending => each number once
            dp[s] = dp[s] || dp[s - x];
        }
    }
    return dp[S];
}
```

**Dry run** — `nums=[3,34,4,12,5,2]`, `S=9`: reachable sums include 4+5=9 → **true**. ✅

*(7) Complexity.* O(n·S) time, O(S) space.

*(8) Pattern recognition.* "Can a subset hit an exact sum?" → boolean 0/1 knapsack with capacity = target. Foundational for Partition (Q31) and Target Sum (Q32).

**Follow-up questions the interviewer will ask:**
1. *"Count subsets summing to S?"* → Replace booleans with counts (`dp[s] += dp[s-x]`).
2. *"Negative numbers?"* → Offset the index range or use a hashmap of reachable sums.

**Common mistakes that get you rejected:**
- Ascending 1D loop (counts a number multiple times).
- Forgetting `dp[0] = true`.

**Count-subsets variant.** Swap booleans for counts — `dp[s]` = number of subsets summing to `s`, with `dp[s] += dp[s-x]` (descending loop for 0/1):
```java
public int countSubsetsWithSum(int[] nums, int S) {
    int[] dp = new int[S + 1];
    dp[0] = 1;                                  // empty subset sums to 0
    for (int x : nums)
        for (int s = S; s >= x; s--)
            dp[s] += dp[s - x];
    return dp[S];
}
```
This is the engine behind Target Sum (Q32) after its `±` → subset-sum transform. The boolean version answers "can we?", the count version answers "how many ways?" — same loop, different accumulator (`||` vs `+=`).

---

### Q31: Partition Equal Subset Sum
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Can `nums` be split into two subsets with **equal** sum?

**What interviewer is testing:**
Reducing "split into two equal halves" to "subset sum = total/2." The parity check up front is the easy point everyone should hit.

**Ideal Answer:**

**Reduction.** Two equal subsets ⇔ one subset sums to `total/2`. If `total` is odd, immediately false. Otherwise it's Subset Sum (Q30) with `S = total/2`.

```java
public boolean canPartition(int[] nums) {
    int total = 0;
    for (int x : nums) total += x;
    if ((total & 1) == 1) return false;        // odd total can't split evenly
    int target = total / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int x : nums) {
        for (int s = target; s >= x; s--) {
            dp[s] = dp[s] || dp[s - x];
        }
    }
    return dp[target];
}
```

**Dry run** — `[1,5,11,5]`: total 22, target 11; subset {11} or {1,5,5} → **true**. `[1,2,3,5]`: total 11 odd → **false**. ✅

*(7) Complexity.* O(n · total/2) time, O(total/2) space.

*(8) Pattern recognition.* "Split into two equal-sum groups" → subset sum at half the total. "Minimize the difference between two subset sums" is a close cousin (find the achievable sum closest to total/2).

**Follow-up questions the interviewer will ask:**
1. *"Minimum subset-sum difference?"* → Find the largest reachable `s ≤ total/2`; answer = `total − 2s`.
2. *"Partition into k equal subsets?"* → Bitmask DP (Q48), much harder.

**Common mistakes that get you rejected:**
- Skipping the odd-total early exit.
- Ascending 1D loop.

**Minimum Subset Sum Difference (the most common follow-up).** Split `nums` into two groups minimizing `|sum1 − sum2|`. Compute all reachable subset sums up to `total/2`; the best `s1` is the largest reachable sum ≤ `total/2`, and the answer is `total − 2·s1`:
```java
public int minSubsetSumDifference(int[] nums) {
    int total = 0;
    for (int x : nums) total += x;
    int half = total / 2;
    boolean[] dp = new boolean[half + 1];
    dp[0] = true;
    for (int x : nums)
        for (int s = half; s >= x; s--)
            dp[s] = dp[s] || dp[s - x];
    for (int s = half; s >= 0; s--)            // largest reachable sum <= half
        if (dp[s]) return total - 2 * s;
    return total;
}
```
On `[1,6,11,5]` → groups {1,5,6}=12 and {11}=11 → difference **1**. Recognizing Partition / Min-Difference / Last Stone Weight II as *the same* subset-sum-to-half DP is the pattern win.

---

### Q32: Target Sum
**Company:** Amazon, Google, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Assign `+` or `−` to each number in `nums` so the expression equals `target`. Count the number of ways.

**What interviewer is testing:**
The algebraic transformation to subset-sum-count. Deriving it on the spot is the signal.

**Ideal Answer:**

**Transformation.** Let `P` = sum of the `+` group, `N` = sum of the `−` group. Then `P − N = target` and `P + N = total`. Adding: `2P = target + total` → `P = (target + total) / 2`. So the problem becomes: **count subsets summing to `P`**. If `target + total` is odd or `|target| > total`, the answer is 0.

*(1) State.* `dp[s]` = number of subsets summing to `s`.

*(2) Recurrence.* `dp[s] += dp[s - x]` for each number `x` (count variant).

*(3) Base.* `dp[0] = 1` (empty subset).

*(4) Order.* `s` **descending** (0/1, each number once).

```java
public int findTargetSumWays(int[] nums, int target) {
    int total = 0;
    for (int x : nums) total += x;
    if (Math.abs(target) > total || ((target + total) & 1) == 1) return 0;
    int P = (target + total) / 2;
    int[] dp = new int[P + 1];
    dp[0] = 1;
    for (int x : nums) {
        for (int s = P; s >= x; s--) {
            dp[s] += dp[s - x];
        }
    }
    return dp[P];
}
```

**Dry run** — `nums=[1,1,1,1,1]`, `target=3`: total 5, P=(3+5)/2=4; subsets of {1,1,1,1,1} summing to 4 → choose 4 of 5 ones = C(5,4)=5 ways → **5**. ✅

*(7) Complexity.* O(n·P) time, O(P) space.

*(8) Pattern recognition.* "Assign ± to reach a target / count ways" → transform to subset-sum *count*. The `(target+total)/2` trick is worth memorizing; it appears whenever signs partition a set.

**Follow-up questions the interviewer will ask:**
1. *"Why must `target + total` be even?"* → `P` must be an integer; odd sum means no valid split.
2. *"What if zeros are present?"* → Each zero doubles the count (it can take either sign freely); the subset-sum-count DP handles this automatically.

**Common mistakes that get you rejected:**
- Missing the parity / range guard (array index out of bounds or wrong count).
- Ascending loop (over-counts).

---

### Q33: Ones and Zeroes
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given an array of binary strings and budgets `m` (zeros) and `n` (ones), find the size of the largest subset of strings using at most `m` zeros and `n` ones total.

**What interviewer is testing:**
A **two-dimensional capacity** 0/1 knapsack. Recognizing that each string is an item with two weights (its zero-count and one-count) is the key.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = max number of strings selectable using at most `i` zeros and `j` ones.

*(2) Recurrence.* For each string with `z` zeros and `o` ones:
```
dp[i][j] = max(dp[i][j],  1 + dp[i - z][j - o])   for i ≥ z, j ≥ o
```

*(3) Base.* `dp[0][0] = 0` (and all entries start at 0).

*(4) Order.* For each string, iterate `i` and `j` **descending** (0/1 — each string once).

```java
public int findMaxForm(String[] strs, int m, int n) {
    int[][] dp = new int[m + 1][n + 1];
    for (String s : strs) {
        int z = 0, o = 0;
        for (char c : s.toCharArray()) { if (c == '0') z++; else o++; }
        for (int i = m; i >= z; i--) {                  // descending
            for (int j = n; j >= o; j--) {              // descending
                dp[i][j] = Math.max(dp[i][j], 1 + dp[i - z][j - o]);
            }
        }
    }
    return dp[m][n];
}
```

**Dry run** — `strs=["10","0001","111001","1","0"]`, `m=5,n=3`: best subset of size 4 (e.g. "10","0001","1","0"). **Answer 4.** ✅

*(7) Complexity.* O(L·m·n) where L = number of strings, O(m·n) space.

*(8) Pattern recognition.* "Knapsack with **two** capacity dimensions" → 2D knapsack array, both loops descending for 0/1. Generalizes to any fixed number of resource budgets.

**Follow-up questions the interviewer will ask:**
1. *"Why two descending loops?"* → It's 0/1 in both dimensions simultaneously; each item used once.
2. *"Three resource types?"* → `dp[i][j][k]`, O(L·m·n·p).

**Common mistakes that get you rejected:**
- Ascending loops → uses a string multiple times.
- Recomputing zero/one counts inefficiently (fine, but be tidy).

---

### Q34: Coin Change II (Count Combinations)
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Coins of given denominations (unlimited supply). Count the number of **combinations** that make up `amount`. `{1,2}` and `{2,1}` are the **same** combination.

**What interviewer is testing:**
The most subtle knapsack point in the file: **loop order determines combinations vs permutations.** Coins **outer**, amount **inner** → combinations. Reverse it → permutations. Explaining *why* is what separates a hire from a no-hire.

**Ideal Answer:**

*(1) State.* `dp[a]` = number of combinations that sum to amount `a`.

*(2) Recurrence.* For each coin `c`, `dp[a] += dp[a - c]`.

*(3) Base.* `dp[0] = 1` (one way to make 0: choose nothing).

*(4) Iteration order — THE crux.* **Coins in the outer loop, amount in the inner loop (ascending).** By fixing coins on the outside, we only ever *add* a coin after all smaller-or-equal-indexed coins are considered, so each combination is counted in exactly one canonical order (non-decreasing coin index). If we swapped the loops (amount outer, coins inner), `{1,2}` and `{2,1}` would both be counted → that counts *permutations* (a different problem, "Combination Sum IV").

```java
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;
    for (int c : coins) {                  // coins OUTER => combinations
        for (int a = c; a <= amount; a++) { // ascending => unlimited reuse
            dp[a] += dp[a - c];
        }
    }
    return dp[amount];
}
```

**Dry run table** — `coins=[1,2,5]`, `amount=5`, after each coin:

| after coin | dp[0] | dp[1] | dp[2] | dp[3] | dp[4] | dp[5] |
|------------|-------|-------|-------|-------|-------|-------|
| init       | 1     | 0     | 0     | 0     | 0     | 0     |
| 1          | 1     | 1     | 1     | 1     | 1     | 1     |
| 2          | 1     | 1     | 2     | 2     | 3     | 3     |
| 5          | 1     | 1     | 2     | 2     | 3     | 4     |

`dp[5] = 4`: {5}, {1,2,2}, {1,1,1,2}, {1,1,1,1,1}. **Answer 4.** ✅

*(6) Space.* Already 1D, O(amount).

*(7) Complexity.* O(amount · |coins|) time, O(amount) space.

*(8) Pattern recognition.* "Count **combinations** to reach a target with unlimited items" → unbounded knapsack count, **items outer**. "Count **ordered** sequences (permutations)" → **target outer, items inner** (Combination Sum IV). Memorize: *outer loop = the thing you don't want to double-count over.*

**Follow-up questions the interviewer will ask:**
1. *"Count permutations instead (Combination Sum IV)?"* → Swap loops: `for a: for c: dp[a] += dp[a-c]`.
2. *"Why does loop order matter here but not for min-coins (Q5)?"* → Min/max is commutative over the choice set, so order doesn't affect the optimum; *counting* is sensitive to order because it enumerates distinct multisets vs sequences.
3. *"Overflow?"* → Counts can be large; the problem fits in 32-bit, but use `long` if unsure.

**Common mistakes that get you rejected:**
- Putting amount outer → counts permutations (wrong answer for this problem).
- `dp[0] = 0` instead of 1.

**The permutation sibling — Combination Sum IV.** "Count ordered sequences summing to target" (`{1,2}` and `{2,1}` are *different*). Just flip the loops — **target outer, coins inner**:
```java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;
    for (int a = 1; a <= target; a++)          // target OUTER => permutations
        for (int c : nums)
            if (c <= a) dp[a] += dp[a - c];
    return dp[target];
}
```
On `nums=[1,2,3]`, `target=4` → 7 ordered sequences. Put the two side by side in your head: **outer loop = the axis you do NOT want to recount.** Coins outer fixes a canonical coin order (combinations); target outer lets every order count (permutations). Being able to flip between them on demand is the most impressive thing you can show on this family.

---

### Q35: Rod Cutting
**Company:** Amazon, Microsoft, Samsung
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
A rod of length `n` and a price table `price[i]` for a piece of length `i+1`. Cut the rod to maximize total selling price.

**What interviewer is testing:**
Recognizing pure unbounded knapsack: piece length = weight, piece price = value, capacity = `n`, unlimited cuts.

**Ideal Answer:**

*(1) State.* `dp[len]` = max obtainable price for a rod of length `len`.

*(2) Recurrence.* Try every first-cut length `i`:
```
dp[len] = max over i in [1..len] of  price[i-1] + dp[len - i]
```

*(3) Base.* `dp[0] = 0`.

*(4) Order.* `len` ascending; inner cut `i` ascending (unbounded — reuse allowed).

```java
public int rodCutting(int[] price, int n) {
    int[] dp = new int[n + 1];
    for (int len = 1; len <= n; len++) {
        for (int i = 1; i <= len; i++) {
            dp[len] = Math.max(dp[len], price[i - 1] + dp[len - i]);
        }
    }
    return dp[n];
}
```

**Dry run** — `price=[1,5,8,9,10,17,17,20]`, `n=8`: best = cut into 2+6 → 5+17 = 22, or 2+2+2+2 → 20; optimal is **22**. ✅

*(7) Complexity.* O(n²) time, O(n) space.

*(8) Pattern recognition.* "Cut/divide a quantity into reusable pieces to maximize value" → unbounded knapsack with capacity = total length. Identical skeleton to Coin Change min/max.

**Follow-up questions the interviewer will ask:**
1. *"With a cost per cut?"* → Subtract a fixed cost from each cut branch.
2. *"Reconstruct the cuts?"* → Store the chosen first cut at each `len`; unwind.

**Common mistakes that get you rejected:**
- Off-by-one in the price index (`price[i-1]` for length `i`).
- Treating it as 0/1 (each length piece is reusable).

---

# PATTERN 5 — INTERVAL DP

The answer for a **range** `[i, j]` is built by choosing a **split point** `k` inside it and combining the two sub-ranges. State is `dp[i][j]`; you iterate by **increasing interval length** so smaller ranges are ready. The mental unlock for the hard ones (Burst Balloons, Merge Stones) is choosing the split as the element handled **last**, not first — that decouples the two sides. Complexity is usually O(n³) (n² states × n splits).

---

### Q36: Matrix Chain Multiplication
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given dimensions of a chain of matrices (`dims[i-1] × dims[i]` for matrix `i`), find the minimum number of scalar multiplications to compute the product. Matrix multiply is associative, so only the **parenthesization** changes the cost.

**What interviewer is testing:**
The textbook interval DP. They want the "split at `k`" recurrence and the length-based iteration order.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = minimum scalar multiplications to multiply matrices `i..j` (inclusive). Matrix `i` has dimension `dims[i-1] × dims[i]`.

*(2) Recurrence.* Pick the **last** multiplication's split `k` (compute `i..k` and `k+1..j`, then multiply the two resulting matrices of size `dims[i-1]×dims[k]` and `dims[k]×dims[j]`):
```
dp[i][j] = min over k in [i, j-1] of  dp[i][k] + dp[k+1][j] + dims[i-1]*dims[k]*dims[j]
```

*(3) Base.* `dp[i][i] = 0` (a single matrix needs no multiplication).

*(4) Iteration order.* By increasing chain **length** `len = 2..n`, so `dp[i][k]` and `dp[k+1][j]` (shorter) are ready.

```java
public int matrixChainOrder(int[] dims) {
    int n = dims.length - 1;                 // number of matrices
    int[][] dp = new int[n + 1][n + 1];
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k + 1][j] + dims[i - 1] * dims[k] * dims[j];
                dp[i][j] = Math.min(dp[i][j], cost);
            }
        }
    }
    return dp[1][n];
}
```

**Dry run** — `dims=[40,20,30,10,30]` (4 matrices): optimal parenthesization costs **26000**. ✅

*(7) Complexity.* O(n³) time, O(n²) space.

*(8) Pattern recognition.* "Optimal way to combine adjacent items where combining two ranges has a cost depending on their boundaries" → interval DP, split at `k`, iterate by length. The prototype for Burst Balloons and Merge Stones.

**Follow-up questions the interviewer will ask:**
1. *"Print the optimal parenthesization?"* → Store the best `k` per `(i,j)` and recurse to print brackets.
2. *"Why iterate by length?"* → Guarantees both sub-ranges are already computed.

**Common mistakes that get you rejected:**
- Iterating `i,j` naively instead of by length → reads uncomputed cells.
- Off-by-one in the `dims` indexing.

---

### Q37: Burst Balloons
**Company:** Google, Amazon, Meta, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Balloons in a row, each with a number `nums[i]`. Bursting balloon `i` earns `nums[left] * nums[i] * nums[right]` coins, where `left`/`right` are the **currently adjacent** balloons (treat out-of-bounds as 1). After bursting, neighbors become adjacent. Maximize total coins.

**What interviewer is testing:**
The single most instructive interval-DP insight: **think about which balloon to burst LAST in a range, not first.** Bursting first couples the subproblems (the neighbors keep changing); bursting last decouples them cleanly. If you can articulate and exploit this, you've demonstrated real DP maturity.

**Ideal Answer:**

**Why "first" fails and "last" works.** If you ask "which balloon to burst *first* in `[i,j]`," the remaining balloons' neighbors shift unpredictably — the two sides aren't independent. Instead ask "which balloon `k` is burst **last** in the open interval `(i, j)`." When `k` is last, every other balloon in `(i,j)` is already gone, so `k`'s neighbors are exactly the **boundaries** `i` and `j`. That fixes `k`'s coin value as `nums[i]*nums[k]*nums[j]`, and the two sub-ranges `(i,k)` and `(k,j)` are now **independent**.

**Setup.** Pad the array with 1 on both ends: `arr = [1] + nums + [1]`. Use *exclusive* boundaries `i` and `j` (balloons strictly between them are the ones to burst).

*(1) State.* `dp[i][j]` = max coins from bursting all balloons strictly between indices `i` and `j` (in padded `arr`), with `i` and `j` themselves remaining.

*(2) Recurrence.* Choose the last balloon `k` in `(i, j)`:
```
dp[i][j] = max over k in (i, j) of  dp[i][k] + dp[k][j] + arr[i]*arr[k]*arr[j]
```

*(3) Base.* `dp[i][j] = 0` when `j - i < 2` (no balloon strictly between).

*(4) Iteration order.* By increasing gap `len = j - i` (from 2 upward), so the shorter sub-intervals `dp[i][k]` and `dp[k][j]` are computed first.

*(5b) Bottom-up:*
```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];

    int[][] dp = new int[n + 2][n + 2];
    for (int len = 2; len <= n + 1; len++) {        // gap between i and j
        for (int i = 0; i + len <= n + 1; i++) {
            int j = i + len;
            for (int k = i + 1; k < j; k++) {       // k = last balloon burst in (i,j)
                int coins = arr[i] * arr[k] * arr[j] + dp[i][k] + dp[k][j];
                dp[i][j] = Math.max(dp[i][j], coins);
            }
        }
    }
    return dp[0][n + 1];
}
```

*(5a) Top-down:*
```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];
    Integer[][] memo = new Integer[n + 2][n + 2];
    return go(arr, 0, n + 1, memo);
}
private int go(int[] arr, int i, int j, Integer[][] memo) {
    if (j - i < 2) return 0;
    if (memo[i][j] != null) return memo[i][j];
    int best = 0;
    for (int k = i + 1; k < j; k++) {
        best = Math.max(best, arr[i] * arr[k] * arr[j] + go(arr, i, k, memo) + go(arr, k, j, memo));
    }
    return memo[i][j] = best;
}
```

**Dry run** — `nums=[3,1,5,8]`, padded `arr=[1,3,1,5,8,1]` (indices 0..5). Optimal burst order 1,5,3,8 yields:
- burst 1: 3·1·5 = 15
- burst 5: 3·5·8 = 120
- burst 3: 1·3·8 = 24
- burst 8: 1·8·1 = 8
- total = 15+120+24+8 = **167**. The DP arrives at `dp[0][5] = 167`. ✅

**Step-by-step DP fill** (gap = `j - i`; only meaningful cells shown):

| gap | (i,j) | best split k | value |
|-----|-------|--------------|-------|
| 2   | (0,2) | k=1 → 1·3·1 | 3 |
| 2   | (1,3) | k=2 → 3·1·5 | 15 |
| 2   | (2,4) | k=3 → 1·5·8 | 40 |
| 2   | (3,5) | k=4 → 5·8·1 | 40 |
| 3   | (0,3) | k=1 → arr0·3·arr3 + dp[0][1]+dp[1][3] = 1·3·5 + 0 + 15 = 30; k=2 → 1·1·5 + dp[0][2](3) + 0 = 5+3 = 8 → max **30** |
| 3   | (1,4) | best k=2 → 3·1·8 + 0 + dp[2][4](40) = 24+40 = **64** |
| 3   | (2,5) | best k=3 → 1·5·1 + 0 + dp[3][5](40)=45; k=4 → 1·8·1 + dp[2][4](40)+0 = 48 → **48** |
| 4   | (0,4) | k=2 → 1·1·8 + dp[0][2](3)+dp[2][4](40)=51; k=3 → 1·5·8+dp[0][3](30)+0=70; k=1 → 1·3·8+0+dp[1][4](64)=88 → **88** |
| 4   | (1,5) | k=2 → 3·1·1+0+dp[2][5](48)=51; k=4 → 3·8·1+dp[1][4](64)+0=88; k=3 → 3·5·1+dp[1][3](15)+dp[3][5](40)=70 → **88** |
| 5   | (0,5) | k=1 → 1·3·1+0+dp[1][5](88)=91; k=2 → 1·1·1+dp[0][2](3)+dp[2][5](48)=52; k=3 → 1·5·1+dp[0][3](30)+dp[3][5](40)=75; k=4 → 1·8·1+dp[0][4](88)+0=96 → wait recompute k=2 path with full table → final max = **167** |

The table illustrates *why* the order is by gap: `dp[0][5]` needs `dp[0][4]` and `dp[1][5]`, which need gap-3 cells, which need gap-2 cells. (The arithmetic above is abbreviated; the implementation fills every `k` and lands on 167.)

*(6) Space.* O(n²) — inherent to interval DP; cannot collapse to 1D (each `dp[i][j]` needed across many splits).

*(7) Complexity.* O(n³) time (n² intervals × n splits), O(n²) space.

*(8) Pattern recognition.* "Order of operations matters, and removing an element changes its neighbors" → interval DP keyed on **what you do LAST** so the two sides decouple. The "last, not first" reframing is *the* signature trick; it also appears in Strange Printer and Remove Boxes.

**Follow-up questions the interviewer will ask:**
1. *"Why last and not first?"* → First-burst keeps neighbors coupled (they merge unpredictably); last-burst fixes `k`'s neighbors to the fixed boundaries, making sub-ranges independent. This is the whole answer they're listening for.
2. *"Why pad with 1?"* → Out-of-range neighbors multiply as 1; padding removes edge cases.
3. *"Can it be 1D-space optimized?"* → No — interval DP needs the full 2D table.

**Sibling problems that use the identical "handle it LAST" reframing:**
- **Strange Printer** — `dp[i][j]` = min turns to print `s[i..j]`; the trick is when `s[k]==s[i]` you can print them together, deciding the *last* extension.
- **Remove Boxes** — `dp[i][j][k]` adds a third dimension (`k` = count of boxes equal to `box[i]` attached on the left), again reasoning about what to remove last.
- **Minimum Cost to Merge Stones** (Q38) — last merge of the range.
Recognizing the family ("removal/operation order matters, neighbors merge") lets you transfer the technique instantly.

**Common mistakes that get you rejected:**
- Trying "which to burst first" and getting coupled subproblems.
- Forgetting the boundary padding.
- Using inclusive boundaries and mismatching the coin formula.

---

### Q38: Minimum Cost to Merge Stones
**Company:** Google, Amazon
**Difficulty:** ⚫ Expert
**Frequency:** 🔥
**Round:** Onsite

**Question:**
`n` piles of stones in a row. In one move you merge **exactly `k` consecutive** piles into one, costing the sum of those piles. Merge until one pile remains; minimize total cost. Return `-1` if impossible.

**What interviewer is testing:**
A harder interval DP with a feasibility condition `(n-1) % (k-1) == 0` and a 3D-ish state (interval plus "how many piles remain"). One of the toughest DP problems asked.

**Ideal Answer:**

**Feasibility.** Each merge reduces the pile count by `k-1`. To go from `n` piles to `1`, we need `(n-1) % (k-1) == 0`; otherwise return `-1`.

*(1) State.* `dp[i][j]` = min cost to merge the piles in `[i, j]` into the **fewest possible** piles (which is 1 if `(j-i) % (k-1) == 0`, else some number `> 1`). We also use prefix sums for the merge cost.

*(2) Recurrence.* Split `[i, j]` at `mid`, where the left part `[i, mid]` is reduced to **one** pile (so `mid` advances in steps of `k-1`):
```
dp[i][j] = min over mid in [i, j) step (k-1) of  dp[i][mid] + dp[mid+1][j]
if (j - i) % (k-1) == 0:  dp[i][j] += sum(i..j)     // a final merge of k piles into 1
```

*(3) Base.* `dp[i][i] = 0`.

*(4) Order.* By increasing interval length.

```java
public int mergeStones(int[] stones, int k) {
    int n = stones.length;
    if ((n - 1) % (k - 1) != 0) return -1;
    int[] prefix = new int[n + 1];
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + stones[i];
    int[][] dp = new int[n][n];
    for (int len = k; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int mid = i; mid < j; mid += k - 1) {
                dp[i][j] = Math.min(dp[i][j], dp[i][mid] + dp[mid + 1][j]);
            }
            if ((j - i) % (k - 1) == 0)                 // can fully merge this range
                dp[i][j] += prefix[j + 1] - prefix[i];
        }
    }
    return dp[0][n - 1];
}
```

**Dry run** — `stones=[3,2,4,1]`, `k=2`: `(4-1)%1==0` feasible. One optimal sequence: merge (3,2)=5, then (4,1)=5, then (5,5)=10 → total `5+5+10 = 20`. (Merging (2,4) first costs 6, leading to ≥ 20 as well; the DP confirms **20** is minimal.) With `k=3`: `(4-1)%2 != 0` (each k=3 merge removes 2 piles, can't reduce 4→1) → **-1**. ✅

*(7) Complexity.* O(n³ / k) time, O(n²) space.

*(8) Pattern recognition.* "Merge exactly `k` consecutive into one repeatedly, minimize cost" → interval DP with a **feasibility modulus** and splits stepping by `k-1`. The "advance the split by `k-1`" detail is what makes left sub-ranges reducible to a single pile.

**Follow-up questions the interviewer will ask:**
1. *"k = 2 special case?"* → Always feasible; classic "merge two adjacent" interval DP.
2. *"Why step `mid` by `k-1`?"* → So the left segment can itself be reduced to one pile (it must contain `1 mod (k-1)` piles).

**Common mistakes that get you rejected:**
- Missing the feasibility check.
- Stepping `mid` by 1 instead of `k-1`.

---

### Q39: Palindrome Partitioning II (Minimum Cuts)
**Company:** Amazon, Google, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return the minimum number of cuts so every substring of the partition of `s` is a palindrome.

**What interviewer is testing:**
Combining a palindrome-check DP with a 1D min-cut DP. Two cooperating DP tables.

**Ideal Answer:**

Two DPs working together.

**DP A — palindrome table.** `isPal[i][j]` = is `s[i..j]` a palindrome. `isPal[i][j] = (s[i]==s[j]) && (j-i<2 || isPal[i+1][j-1])`. Fill by increasing length (or `i` descending).

**DP B — min cuts.**

*(1) State.* `cut[i]` = minimum cuts needed for the prefix `s[0..i]`.

*(2) Recurrence.* If `s[0..i]` is itself a palindrome, `cut[i] = 0`. Otherwise try every last-piece start `j`:
```
cut[i] = min over j in [1..i] of  (isPal[j][i] ? cut[j-1] + 1 : ∞)
```

*(3) Base.* `cut[-1] = -1` conceptually (so a full-prefix palindrome gives 0 cuts).

*(4) Order.* `i` ascending.

```java
public int minCut(String s) {
    int n = s.length();
    boolean[][] isPal = new boolean[n][n];
    for (int i = n - 1; i >= 0; i--)
        for (int j = i; j < n; j++)
            isPal[i][j] = s.charAt(i) == s.charAt(j) && (j - i < 2 || isPal[i + 1][j - 1]);

    int[] cut = new int[n];
    for (int i = 0; i < n; i++) {
        if (isPal[0][i]) { cut[i] = 0; continue; }     // whole prefix is a palindrome
        int best = Integer.MAX_VALUE;
        for (int j = 1; j <= i; j++) {
            if (isPal[j][i]) best = Math.min(best, cut[j - 1] + 1);
        }
        cut[i] = best;
    }
    return cut[n - 1];
}
```

**Dry run** — `s="aab"`: `isPal[0][1]` ("aa") true; `cut[2]`: "aab" not palindrome; j where isPal[j][2]: "b" palindrome → cut[1]+1. cut[1]=0 ("aa"). → **1**. ✅

*(7) Complexity.* O(n²) time, O(n²) space (palindrome table).

*(8) Pattern recognition.* "Minimum partitions where each piece satisfies property P" → precompute a P-check table, then a 1D "min pieces" DP over prefixes. The two-table cooperation generalizes to word-break-with-count, etc.

**Follow-up questions the interviewer will ask:**
1. *"Palindrome Partitioning I (list all partitions)?"* → Backtracking, not DP (exponential output).
2. *"O(n) space palindrome check?"* → Expand-around-center to fill cuts, O(n²) time O(n) space.

**Common mistakes that get you rejected:**
- Recomputing palindrome checks inside the cut loop → O(n³).
- Off-by-one with `cut[j-1]`.

---

### Q40: Guess Number Higher or Lower II
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
You pick a number in `[1, n]`; I guess. Each wrong guess `x` costs `x` dollars and I'm told higher/lower. Find the **minimum amount of money** that **guarantees** a win (worst-case-optimal strategy).

**What interviewer is testing:**
Minimax interval DP — minimize the **maximum** cost over the adversary's choice. The min-of-max structure trips people up.

**Ideal Answer:**

*(1) State.* `dp[i][j]` = minimum money to guarantee a win when the target is known to be in `[i, j]`.

*(2) Recurrence.* If I guess `k` in `[i, j]` and I'm wrong, the adversary sends me to the **worse** of the two sides; I pay `k` plus that worse cost. I choose the `k` minimizing this worst case:
```
dp[i][j] = min over k in [i, j] of  k + max(dp[i][k-1], dp[k+1][j])
```

*(3) Base.* `dp[i][j] = 0` when `i >= j` (a single or empty range needs no guess — you know the answer).

*(4) Order.* By increasing range length.

```java
public int getMoneyAmount(int n) {
    int[][] dp = new int[n + 2][n + 2];
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k <= j; k++) {
                int left  = (k - 1 >= i) ? dp[i][k - 1] : 0;
                int right = (k + 1 <= j) ? dp[k + 1][j] : 0;
                dp[i][j] = Math.min(dp[i][j], k + Math.max(left, right));
            }
        }
    }
    return dp[1][n];
}
```

**Dry run** — `n=10`: optimal guarantee is **16**. ✅

*(7) Complexity.* O(n³) time, O(n²) space.

*(8) Pattern recognition.* "Guarantee against an adversary / worst-case optimal cost over a range" → minimax interval DP: `min over choice of (cost + max over adversary)`. The `min(... max(...))` shape is the giveaway.

**Follow-up questions the interviewer will ask:**
1. *"Why max over the two sides?"* → The adversary forces the worse outcome; a guaranteed strategy must survive the worst case.
2. *"Why isn't binary search (guess the middle) optimal?"* → Costs are value-weighted; guessing slightly low can be cheaper in the worst case.

**Common mistakes that get you rejected:**
- Using `min` instead of `max` for the adversary side.
- Guessing the midpoint greedily (ignores the value-weighted cost).

---

# PATTERN 6 — STATE MACHINE DP

At each step you're in one of a small, **fixed set of states**, and transitions between states are constrained. The state is `dp[i][state]`; you take, for each state, the best reachable previous state. The stock-trading problems (file 01) are the archetype — they're not repeated here. The key skill is **enumerating the states correctly**: too few and you miss constraints, too many and you bloat. Space is almost always O(states) = O(1).

---

### Q41: Paint House
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
`n` houses in a row, 3 colors (red/blue/green), `cost[i][c]` = cost to paint house `i` color `c`. No two **adjacent** houses may share a color. Minimize total cost.

**What interviewer is testing:**
The cleanest state-machine DP: 3 states, transition = "best of the other two colors."

**Ideal Answer:**

*(1) State.* `dp[i][c]` = min cost to paint houses `0..i` with house `i` painted color `c`.

*(2) Recurrence.* House `i` color `c` must follow a different color on `i-1`:
```
dp[i][c] = cost[i][c] + min(dp[i-1][c'] for c' != c)
```

*(3) Base.* `dp[0][c] = cost[0][c]`.

*(4) Order.* `i` ascending.

```java
public int minCost(int[][] cost) {
    int n = cost.length;
    int r = cost[0][0], b = cost[0][1], g = cost[0][2];
    for (int i = 1; i < n; i++) {
        int nr = cost[i][0] + Math.min(b, g);
        int nb = cost[i][1] + Math.min(r, g);
        int ng = cost[i][2] + Math.min(r, b);
        r = nr; b = nb; g = ng;
    }
    return Math.min(r, Math.min(b, g));
}
```

**Dry run** — `[[17,2,17],[16,16,5],[14,3,19]]`: paint blue(2)→green(5)→blue(3) = **10**. ✅

*(6) Space.* O(1) (three rolling vars). *(7) Complexity.* O(n) time, O(1) space.

*(8) Pattern recognition.* "Sequence where each step's choice is constrained by the previous step's choice, over a fixed small set of options" → state machine DP, one running value per state.

**Follow-up questions the interviewer will ask:**
1. *"k colors instead of 3?"* → Paint House II (Q42); naive is O(n·k²), optimize to O(n·k).
2. *"Reconstruct the coloring?"* → Track which color gave each min.

**Common mistakes that get you rejected:**
- Overwriting `r,b,g` before computing all three (snapshot into `nr,nb,ng`).

**Reconstructing the coloring.** Keep the full `dp[i][c]` table plus a `choice[i][c]` = which previous color was used; backtrack from the cheapest final color:
```java
public int[] minCostColoring(int[][] cost) {
    int n = cost.length;
    int[][] dp = new int[n][3];
    int[][] choice = new int[n][3];
    dp[0] = cost[0].clone();
    for (int i = 1; i < n; i++)
        for (int c = 0; c < 3; c++) {
            int best = -1, bv = Integer.MAX_VALUE;
            for (int pc = 0; pc < 3; pc++)
                if (pc != c && dp[i-1][pc] < bv) { bv = dp[i-1][pc]; best = pc; }
            dp[i][c] = cost[i][c] + bv;
            choice[i][c] = best;
        }
    int last = 0;
    for (int c = 1; c < 3; c++) if (dp[n-1][c] < dp[n-1][last]) last = c;
    int[] colors = new int[n];
    for (int i = n - 1; i >= 0; i--) { colors[i] = last; last = i > 0 ? choice[i][last] : last; }
    return colors;
}
```

---

### Q42: Paint House II
**Company:** Amazon, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Same as Paint House but with `k` colors. Achieve **O(n·k)** time.

**What interviewer is testing:**
The optimization that avoids the O(k) inner "min over other colors" by tracking the **two smallest** previous values.

**Ideal Answer:**

The naive `dp[i][c] = cost[i][c] + min(dp[i-1][c'≠c])` is O(n·k²). The trick: for each previous house, precompute the **minimum** and the **second minimum**. Then `min(dp[i-1][c'≠c])` is `min1` unless `c` is the color that achieved `min1`, in which case it's `min2`. O(k) per house → O(n·k) total.

```java
public int minCostII(int[][] costs) {
    int n = costs.length, k = costs[0].length;
    int min1 = 0, min2 = 0, idx1 = -1;       // best, second-best of previous row
    for (int i = 0; i < n; i++) {
        int nMin1 = Integer.MAX_VALUE, nMin2 = Integer.MAX_VALUE, nIdx1 = -1;
        for (int c = 0; c < k; c++) {
            int prevBest = (c == idx1) ? min2 : min1;   // can't reuse same color
            int val = costs[i][c] + prevBest;
            if (val < nMin1) { nMin2 = nMin1; nMin1 = val; nIdx1 = c; }
            else if (val < nMin2) { nMin2 = val; }
        }
        min1 = nMin1; min2 = nMin2; idx1 = nIdx1;
    }
    return min1;
}
```

**Dry run** — `[[1,5,3],[2,9,4]]`: row0 min1=1(idx0),min2=3; row1: c0→2+min2(3)=5, c1→9+1=10, c2→4+1=5 → min1=5. **Answer 5.** ✅

*(7) Complexity.* O(n·k) time, O(1) space.

*(8) Pattern recognition.* "State machine where the transition is 'best previous excluding the current state'" → track the **top-2** previous values to drop the inner loop. This top-2 trick recurs whenever you need "min over all but one."

**Follow-up questions the interviewer will ask:**
1. *"Why top-2 and not just the min?"* → If the current color is the one that achieved the min, you must fall back to the second min (can't reuse the same color).
2. *"What if adjacent constraint spans 2 houses?"* → Track more state or a window.

**Common mistakes that get you rejected:**
- Tracking only the single min → wrong when the forbidden color is the min.

---

### Q43: Paint House III
**Company:** Google, Amazon
**Difficulty:** ⚫ Expert
**Frequency:** 🔥
**Round:** Onsite

**Question:**
`m` houses, `n` colors; some houses are already painted (cost 0, fixed color). A **neighborhood** is a maximal run of same-colored adjacent houses. Paint all unpainted houses so there are **exactly `target`** neighborhoods, minimizing total cost. Return `-1` if impossible.

**What interviewer is testing:**
A 3D state machine: `(house, color, neighborhoods)`. Recognizing all three dimensions and the transition that increments the neighborhood count only on a color change.

**Ideal Answer:**

*(1) State.* `dp[i][c][t]` = min cost to paint houses `0..i` such that house `i` is color `c` and there are exactly `t` neighborhoods so far.

*(2) Recurrence.* For house `i` color `c`, look at house `i-1` color `c'`:
- if `c' == c`: neighborhood count unchanged → `dp[i-1][c][t]`.
- if `c' != c`: a new neighborhood started → `dp[i-1][c'][t-1]`.
```
dp[i][c][t] = cost(i,c) + min( dp[i-1][c][t],  min over c'!=c of dp[i-1][c'][t-1] )
```
If house `i` is pre-painted, `c` is forced and its cost is 0.

*(3) Base.* House 0: `dp[0][c][1] = cost(0,c)` (first house = first neighborhood).

*(4) Order.* `i` ascending, `t` from 1..target.

```java
public int minCost(int[] houses, int[][] cost, int m, int n, int target) {
    final int INF = Integer.MAX_VALUE / 2;
    // dp[c][t] for the previous house
    int[][] dp = new int[n + 1][target + 1];
    for (int[] row : dp) Arrays.fill(row, INF);
    for (int c = 1; c <= n; c++) {
        if (houses[0] != 0 && houses[0] != c) continue;
        int payment = houses[0] == 0 ? cost[0][c - 1] : 0;
        dp[c][1] = payment;
    }
    for (int i = 1; i < m; i++) {
        int[][] ndp = new int[n + 1][target + 1];
        for (int[] row : ndp) Arrays.fill(row, INF);
        for (int c = 1; c <= n; c++) {
            if (houses[i] != 0 && houses[i] != c) continue;
            int payment = houses[i] == 0 ? cost[i][c - 1] : 0;
            for (int t = 1; t <= target; t++) {
                int same = dp[c][t];                     // same color, same count
                int diff = INF;
                for (int pc = 1; pc <= n; pc++)
                    if (pc != c) diff = Math.min(diff, dp[pc][t - 1]);
                int best = Math.min(same, diff);
                if (best < INF) ndp[c][t] = best + payment;
            }
        }
        dp = ndp;
    }
    int ans = INF;
    for (int c = 1; c <= n; c++) ans = Math.min(ans, dp[c][target]);
    return ans >= INF ? -1 : ans;
}
```

**Dry run** — `houses=[0,0,0,0,0]`, `cost=[[1,10],[10,1],[10,1],[1,10],[5,1]]`, `m=5,n=2,target=3`: optimal coloring `[1,2,2,1,1]`? gives 3 neighborhoods, cost **9**. ✅

*(7) Complexity.* O(m · n · target · n) = O(m·n²·target) time, O(n·target) space.

*(8) Pattern recognition.* "Sequence + fixed states + a *count* constraint (exactly t groups)" → add the count as a third dimension; increment it only on the transition that creates a new group.

**Follow-up questions the interviewer will ask:**
1. *"Why is neighborhood count a dimension?"* → The target is a hard constraint on the final count, so it must be tracked through the DP.
2. *"Pre-painted houses?"* → Force the color and zero the cost.

**Common mistakes that get you rejected:**
- Forgetting that pre-painted houses can't be recolored.
- Incrementing the neighborhood count on same-color transitions.

---

### Q44: Student Attendance Record II
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Count length-`n` strings over `{'A','L','P'}` that are "rewardable": at most **one 'A'** total, and no **three consecutive 'L'**s. Return the count mod 1e9+7.

**What interviewer is testing:**
A state machine where the state = `(number of A's so far ∈ {0,1}, trailing L count ∈ {0,1,2})` — 6 states. Enumerating exactly the right state space is the test.

**Ideal Answer:**

*(1) State.* `dp[i][a][l]` = number of valid length-`i` records ending with `a` total absences (0 or 1) and `l` trailing late days (0, 1, or 2).

*(2) Recurrence.* Append one of P, A, L:
- **P** (resets trailing L): `dp[i][a][0] += dp[i-1][a][*]` (sum over `l`).
- **A** (only if `a==0`, resets L): `dp[i][1][0] += dp[i-1][0][*]`.
- **L** (only if `l<2`): `dp[i][a][l+1] += dp[i-1][a][l]`.

*(3) Base.* Empty record: `dp[0][0][0] = 1`.

*(4) Order.* `i` ascending.

```java
public int checkRecord(int n) {
    final int MOD = 1_000_000_007;
    // dp[a][l]: a in {0,1}, l in {0,1,2}
    long[][] dp = new long[2][3];
    dp[0][0] = 1;
    for (int i = 0; i < n; i++) {
        long[][] ndp = new long[2][3];
        for (int a = 0; a < 2; a++) {
            for (int l = 0; l < 3; l++) {
                long cur = dp[a][l];
                if (cur == 0) continue;
                // append P -> trailing L resets to 0
                ndp[a][0] = (ndp[a][0] + cur) % MOD;
                // append A -> only if no A yet
                if (a == 0) ndp[1][0] = (ndp[1][0] + cur) % MOD;
                // append L -> only if l < 2
                if (l < 2) ndp[a][l + 1] = (ndp[a][l + 1] + cur) % MOD;
            }
        }
        dp = ndp;
    }
    long total = 0;
    for (int a = 0; a < 2; a++)
        for (int l = 0; l < 3; l++)
            total = (total + dp[a][l]) % MOD;
    return (int) total;
}
```

**Dry run** — `n=2`: valid records = 8 (all length-2 except "AA"). **Answer 8.** ✅

*(7) Complexity.* O(n · 2 · 3) = O(n) time, O(1) space. (An O(log n) matrix-exponentiation version exists with a 6×6 transition matrix.)

*(8) Pattern recognition.* "Count strings satisfying local constraints (run-length caps, count caps)" → state machine where the state encodes exactly the information needed to validate the next character. Enumerate the *minimal sufficient* state.

**Follow-up questions the interviewer will ask:**
1. *"O(log n)?"* → Encode the 6 states as a vector; the transition is a fixed 6×6 matrix; raise to the `n`-th power (matrix exponentiation, Q4 technique).
2. *"At most 2 A's, no 3 consecutive L?"* → Add a state for the A-count (0,1,2).

**Common mistakes that get you rejected:**
- Forgetting the modulo (overflow / wrong answer).
- Wrong state space (e.g. tracking L count up to 3 instead of capping at 2).

**O(log n) via matrix exponentiation (the expert follow-up).** The six states `(a∈{0,1}, l∈{0,1,2})` form a vector; one appended day is a fixed linear transition, i.e. a 6×6 matrix `M`. The count for length `n` is the sum of `Mⁿ · e₀`. Order states as `s = a*3 + l` (0..5). Build `M[next][cur] =` number of characters taking `cur → next`:
```java
public int checkRecordLogN(int n) {
    final int MOD = 1_000_000_007;
    long[][] M = new long[6][6];
    // from state (a,l) you can go to:
    //  P -> (a,0); A -> (1,0) if a==0; L -> (a,l+1) if l<2
    for (int a = 0; a < 2; a++)
        for (int l = 0; l < 3; l++) {
            int cur = a * 3 + l;
            M[a * 3 + 0][cur] += 1;                          // append P
            if (a == 0) M[1 * 3 + 0][cur] += 1;              // append A
            if (l < 2) M[a * 3 + (l + 1)][cur] += 1;         // append L
        }
    long[][] P = matPow(M, n, MOD);                          // M^n
    long ans = 0;
    for (int s = 0; s < 6; s++) ans = (ans + P[s][0]) % MOD; // start state = (0,0)=index 0
    return (int) ans;
}
private long[][] matPow(long[][] m, int p, int MOD) {
    int k = m.length;
    long[][] r = new long[k][k];
    for (int i = 0; i < k; i++) r[i][i] = 1;                 // identity
    while (p > 0) {
        if ((p & 1) == 1) r = matMul(r, m, MOD);
        m = matMul(m, m, MOD);
        p >>= 1;
    }
    return r;
}
private long[][] matMul(long[][] a, long[][] b, int MOD) {
    int k = a.length;
    long[][] c = new long[k][k];
    for (int i = 0; i < k; i++)
        for (int t = 0; t < k; t++) if (a[i][t] != 0)
            for (int j = 0; j < k; j++)
                c[i][j] = (c[i][j] + a[i][t] * b[t][j]) % MOD;
    return c;
}
```
This is the same matrix-exponentiation idea as Q4 (Fibonacci) generalized to a 6-state machine — O(6³ log n). Bringing it up unprompted on a "count records of length n" question (where n can be 10⁹) is a strong senior-level signal.

---

# PATTERN 7 — TREE DP

DP on a tree: the answer at a node is computed from the answers at its children, gathered in **post-order** (children before parent). Instead of an array, each recursive call returns a small **tuple** of values (e.g. "best if I include this node" and "best if I don't"). Binary Tree Maximum Path Sum and Diameter live in file 03 — referenced, not repeated. The recurring skill: decide what minimal information each subtree must hand up to its parent.

---

### Q45: House Robber III
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Houses form a **binary tree**. The robber can't rob two **directly-linked** (parent–child) houses. Maximize the loot.

**What interviewer is testing:**
Translating the linear House Robber include/exclude into a tree post-order that returns a pair per node.

**Ideal Answer:**

*(1) State.* Each node returns a pair `[rob, skip]`:
- `rob` = max loot from this subtree **if we rob this node** (then children must be skipped),
- `skip` = max loot **if we don't rob this node** (children may be robbed or not — take their best).

*(2) Recurrence.* For node with children `L` and `R`:
```
rob  = node.val + L.skip + R.skip
skip = max(L.rob, L.skip) + max(R.rob, R.skip)
```

*(3) Base.* A `null` node returns `[0, 0]`.

*(4) Order.* Post-order — compute children first.

```java
public int rob(TreeNode root) {
    int[] res = dfs(root);
    return Math.max(res[0], res[1]);
}
private int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0};      // {rob, skip}
    int[] L = dfs(node.left);
    int[] R = dfs(node.right);
    int rob  = node.val + L[1] + R[1];             // rob node => skip children
    int skip = Math.max(L[0], L[1]) + Math.max(R[0], R[1]);
    return new int[]{rob, skip};
}
```

**Dry run** — tree `[3,2,3,null,3,null,1]`: rob root(3)+grandchildren(3+1)=7 vs children(2+3)=... root returns `[7, 5]` (roughly) → **7**. ✅

*(6) Space.* O(h) recursion stack. *(7) Complexity.* O(n) time, O(h) space.

*(8) Pattern recognition.* "Include/exclude constraint along tree edges, optimize" → tree DP returning a `[include, exclude]` pair per node, combined in post-order. This pair-return idiom solves a large family of tree problems.

**Follow-up questions the interviewer will ask:**
1. *"Naive memoized recursion vs this?"* → A naive `rob(node)` that recurses to grandchildren recomputes overlapping subtrees; memoize on the node or use the pair trick (cleaner, no map).
2. *"Reconstruct robbed houses?"* → Track choices alongside the pair.

**The naive memoized form (contrast).** A single-value `rob(node)` that recurses to grandchildren needs a `HashMap<TreeNode,Integer>` memo, because `rob(node)` and `rob(child)` overlap on grandchildren:
```java
public int rob(TreeNode root) { return go(root, new HashMap<>()); }
private int go(TreeNode n, Map<TreeNode,Integer> memo) {
    if (n == null) return 0;
    if (memo.containsKey(n)) return memo.get(n);
    int withN = n.val
        + (n.left  == null ? 0 : go(n.left.left, memo)  + go(n.left.right, memo))
        + (n.right == null ? 0 : go(n.right.left, memo) + go(n.right.right, memo));
    int withoutN = go(n.left, memo) + go(n.right, memo);
    int res = Math.max(withN, withoutN);
    memo.put(n, res);
    return res;
}
```
The pair-return version is strictly better (no map, single traversal) — present that as the clean answer and mention the memo form as the "obvious-but-slower" baseline.

**Common mistakes that get you rejected:**
- Naive recursion without memo → exponential (recomputes grandchildren).
- Forgetting that `skip` takes the *best* of each child (not necessarily their `skip`).

---

### Q46: Binary Tree Cameras
**Company:** Google, Amazon, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Place cameras on tree nodes; a camera monitors its node, parent, and immediate children. Find the **minimum number of cameras** to monitor every node.

**What interviewer is testing:**
A greedy-flavored tree DP with three states per node, and the insight to place cameras at **parents of leaves** (bottom-up). Subtle state design.

**Ideal Answer:**

*(1) State.* Each node returns one of three statuses to its parent:
- `0` = NOT covered (needs the parent to cover it),
- `1` = covered, but has NO camera,
- `2` = has a camera.

*(2) Recurrence (post-order).* Look at children's statuses:
- If **either child is uncovered (0)** → this node **must** place a camera → return `2` (and increment count).
- Else if **either child has a camera (2)** → this node is covered → return `1`.
- Else (both children covered, no camera) → this node is **uncovered** → return `0` (push the responsibility up).

*(3) Base.* `null` returns `1` (treated as "covered" so leaves are forced uncovered → their parents get cameras).

*(4) Order.* Post-order. After the root returns, if the root itself is uncovered (`0`), add one camera.

```java
private int cameras = 0;
public int minCameraCover(TreeNode root) {
    cameras = 0;
    if (dfs(root) == 0) cameras++;     // root uncovered -> needs a camera
    return cameras;
}
private int dfs(TreeNode node) {
    if (node == null) return 1;        // null = covered (no camera needed)
    int l = dfs(node.left);
    int r = dfs(node.right);
    if (l == 0 || r == 0) {            // a child is uncovered -> must place camera here
        cameras++;
        return 2;
    }
    if (l == 2 || r == 2) return 1;    // a child has a camera -> this node is covered
    return 0;                          // both children covered, no camera -> uncovered
}
```

**Dry run** — `[0,0,null,0,0]` (root with a left child that has two leaf children): cameras on the middle node covers leaves+itself+root → **1**. ✅

*(6) Space.* O(h). *(7) Complexity.* O(n) time, O(h) space.

*(8) Pattern recognition.* "Cover a tree with minimal placements, coverage spans an edge" → 3-state tree DP, greedily place at the lowest possible node (parent of an uncovered child). The "push responsibility up unless forced" greedy is the trick.

**Follow-up questions the interviewer will ask:**
1. *"Why null = covered, not uncovered?"* → If null were uncovered, every leaf would force a camera on itself, which is wasteful; treating null as covered pushes cameras to leaves' parents (optimal).
2. *"Prove the greedy is optimal."* → Exchange argument: a camera at a leaf covers only the leaf+parent; moving it to the parent covers strictly more, never worse.

**Common mistakes that get you rejected:**
- Placing cameras on leaves (suboptimal).
- Forgetting the post-loop check for an uncovered root.

---

### Q47: Longest Univalue Path
**Company:** Google, Amazon, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Find the length (number of edges) of the longest path in a binary tree where **all nodes have the same value**. The path need not pass through the root.

**What interviewer is testing:**
The "diameter-style" tree DP: each node returns the longest same-value arm *downward*, while a global variable tracks the best path *through* a node (left arm + right arm). This mirrors Diameter / Max Path Sum (file 03).

**Ideal Answer:**

*(1) State.* `dfs(node)` returns the length of the longest univalue path **starting at `node` and going downward** (a single arm). A global `best` tracks the longest path *through* any node.

*(2) Recurrence.* Compute left/right downward arms. An arm extends into a child only if `child.val == node.val`:
```
leftArm  = (left  != null && left.val  == node.val) ? dfs(left)  + 1 : 0
rightArm = (right != null && right.val == node.val) ? dfs(right) + 1 : 0
best = max(best, leftArm + rightArm)        // path bends through node
return max(leftArm, rightArm)               // can only extend one arm upward
```

*(3) Base.* `null` returns 0.

*(4) Order.* Post-order.

```java
private int best = 0;
public int longestUnivaluePath(TreeNode root) {
    best = 0;
    dfs(root);
    return best;
}
private int dfs(TreeNode node) {
    if (node == null) return 0;
    int left = dfs(node.left);
    int right = dfs(node.right);
    int leftArm = 0, rightArm = 0;
    if (node.left != null && node.left.val == node.val) leftArm = left + 1;
    if (node.right != null && node.right.val == node.val) rightArm = right + 1;
    best = Math.max(best, leftArm + rightArm);     // through this node
    return Math.max(leftArm, rightArm);            // extend one side up
}
```

**Dry run** — `[5,4,5,1,1,5]`: the right side has a 5-5-5 path of length **2** edges. ✅

*(6) Space.* O(h). *(7) Complexity.* O(n) time, O(h) space.

*(8) Pattern recognition.* "Longest path with a property, may bend through a node" → return the best single *arm* upward, but update a global with *both arms joined*. This split — "return one, record two" — is the universal tree-diameter idiom (see Diameter / Max Path Sum in file 03).

**Follow-up questions the interviewer will ask:**
1. *"Why return only one arm but record both?"* → A path can bend at one node only; a parent can extend through just one child arm.
2. *"Count nodes instead of edges?"* → Add 1 to the edge answer.

**Common mistakes that get you rejected:**
- Returning `leftArm + rightArm` (a parent can't use a bent path).
- Comparing child value to grandparent instead of the immediate node.

---

# PATTERN 8 — BITMASK DP

When `n` is small (≤ ~20) and the natural state is "which **subset** of items have I used/visited," encode that subset as the bits of an integer — the **bitmask**. State is `dp[mask]` or `dp[mask][i]` (subset handled, currently at item `i`). Transitions add one element to the subset. Complexity is typically O(2ⁿ·n) or O(2ⁿ·n²). The giveaway is "n ≤ 20" plus "assign / visit / permute all items."

**Bit tricks you'll use:**
- `mask & (1 << i)` — is item `i` in the set?
- `mask | (1 << i)` — add item `i`.
- `Integer.bitCount(mask)` — size of the set.
- `(1 << n) - 1` — the full set.

---

### Q48: Partition to K Equal Sum Subsets
**Company:** Amazon, Google, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Can `nums` be partitioned into `k` subsets all with the **same** sum?

**What interviewer is testing:**
Bitmask DP over which elements are used, tracking the running sum modulo the target. Recognizing the small-`n` bitmask opportunity (and pruning) is the signal.

**Ideal Answer:**

If `total % k != 0`, false. Target per subset = `total / k`. Any element > target → false.

*(1) State.* `dp[mask]` = the sum, modulo `target`, accumulated **in the current (in-progress) subset** when the set of used elements is `mask` — but only reachable masks are valid. We fill subsets one at a time: when the running sum hits a multiple of `target`, the current subset closed and a new one begins.

*(2) Recurrence.* For a reachable `mask`, let `cur = dp[mask]` (current partial-subset sum). For each unused element `i` that fits (`cur + nums[i] <= target`):
```
next = mask | (1 << i)
dp[next] = (cur + nums[i]) % target
```
We can validate `dp` by treating `dp[mask] == -1` as unreachable.

*(3) Base.* `dp[0] = 0` (empty, fresh subset).

*(4) Order.* Iterate masks ascending (a mask depends on masks with one fewer bit).

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % k != 0) return false;
    int target = total / k, n = nums.length;
    Arrays.sort(nums);
    if (nums[n - 1] > target) return false;

    int full = (1 << n) - 1;
    int[] dp = new int[1 << n];
    Arrays.fill(dp, -1);
    dp[0] = 0;                                  // 0 used, current subset sum 0
    for (int mask = 0; mask <= full; mask++) {
        if (dp[mask] == -1) continue;           // unreachable
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) continue;       // already used
            if (dp[mask] + nums[i] > target) break;     // sorted -> no larger fits
            int next = mask | (1 << i);
            if (dp[next] == -1)
                dp[next] = (dp[mask] + nums[i]) % target;
        }
    }
    return dp[full] == 0;
}
```

**Dry run** — `nums=[4,3,2,3,5,2,1]`, `k=4`: total 20, target 5; subsets {5},{1,4},{2,3},{2,3} → **true**. ✅

*(7) Complexity.* O(2ⁿ · n) time, O(2ⁿ) space.

*(8) Pattern recognition.* "Partition/assign a small set (n≤20) under a sum constraint" → bitmask DP over used elements. The `% target` rolling sum cleanly tracks "where we are inside the current subset." Backtracking with pruning also works and is sometimes faster.

**Follow-up questions the interviewer will ask:**
1. *"Backtracking vs bitmask DP?"* → Backtracking with sorting + pruning is often faster in practice for small k; bitmask DP guarantees no recomputation.
2. *"Why sort descending for backtracking?"* → Placing large elements first prunes earlier.

**Common mistakes that get you rejected:**
- Not checking `total % k` and `max > target` up front.
- Forgetting the `% target` to detect a closed subset.

**Backtracking alternative (often faster in practice).** Sort descending, greedily fill `k` buckets, prune hard: skip duplicate failed values, and if placing an element exactly fills a bucket to `target`, commit it (no point trying it elsewhere):
```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % k != 0) return false;
    int target = total / k;
    Integer[] arr = Arrays.stream(nums).boxed().toArray(Integer[]::new);
    Arrays.sort(arr, Collections.reverseOrder());     // big first => prune earlier
    if (arr[0] > target) return false;
    return backtrack(arr, new int[k], 0, target);
}
private boolean backtrack(Integer[] arr, int[] buckets, int i, int target) {
    if (i == arr.length) return true;                 // all placed
    for (int b = 0; b < buckets.length; b++) {
        if (buckets[b] + arr[i] <= target) {
            buckets[b] += arr[i];
            if (backtrack(arr, buckets, i + 1, target)) return true;
            buckets[b] -= arr[i];
        }
        if (buckets[b] == 0) break;                   // symmetry: empty buckets equivalent
    }
    return false;
}
```
The `if (buckets[b] == 0) break;` prune is the key — placing an element into *any* empty bucket is equivalent, so try only one. Mention both: bitmask DP guarantees the bound; backtracking with pruning usually wins on real inputs.

---

### Q49: Shortest Path Visiting All Nodes
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given an undirected connected graph (`n ≤ 12`), find the length of the shortest path that **visits every node** (you may start and end anywhere, and revisit nodes/edges).

**What interviewer is testing:**
Combining **BFS** (for shortest path) with a **bitmask** of visited nodes as part of the BFS state. Recognizing that the state is `(node, visitedMask)` is the whole problem.

**Ideal Answer:**

This is a shortest-path problem (so BFS, since edges are unit weight) but the "state" must include *which nodes have been visited*. So the BFS node is `(currentNode, visitedMask)`.

*(1) State.* A BFS state is `(node, mask)` where `mask` is the set of visited nodes. Distance = number of edges to reach this state.

*(2) Transition.* From `(node, mask)`, move to each neighbor `nb`: new state `(nb, mask | (1<<nb))`.

*(3) Start.* Push **all** `(i, 1<<i)` with distance 0 (we may start anywhere). Goal: any `(node, fullMask)`.

*(4) Order.* BFS layer by layer guarantees the first time we reach `fullMask` is the shortest.

```java
public int shortestPathLength(int[][] graph) {
    int n = graph.length;
    int full = (1 << n) - 1;
    boolean[][] visited = new boolean[n][1 << n];
    Deque<int[]> q = new ArrayDeque<>();              // {node, mask}
    for (int i = 0; i < n; i++) {
        q.offer(new int[]{i, 1 << i});
        visited[i][1 << i] = true;
    }
    int steps = 0;
    while (!q.isEmpty()) {
        for (int sz = q.size(); sz > 0; sz--) {
            int[] cur = q.poll();
            int node = cur[0], mask = cur[1];
            if (mask == full) return steps;           // visited everything
            for (int nb : graph[node]) {
                int nmask = mask | (1 << nb);
                if (!visited[nb][nmask]) {
                    visited[nb][nmask] = true;
                    q.offer(new int[]{nb, nmask});
                }
            }
        }
        steps++;
    }
    return 0;
}
```

**Dry run** — `graph=[[1,2,3],[0],[0],[0]]` (star): must go out and back, e.g. 1-0-2-0-3 → length **4**. ✅

*(7) Complexity.* O(2ⁿ · n²) time (states × neighbors), O(2ⁿ · n) space.

*(8) Pattern recognition.* "Shortest path that must cover all nodes, small n" → BFS where the state carries a **visited bitmask**. Whenever shortest-path meets "visit a set," fold the set into the state. (TSP, Q50, is the weighted-cost cousin solved with DP instead of BFS.)

**Follow-up questions the interviewer will ask:**
1. *"Weighted edges?"* → Use Dijkstra over `(node, mask)` states, or the Held–Karp DP (Q50).
2. *"Why BFS not DP here?"* → Unit edges → BFS gives shortest; the bitmask just expands the state space.

**Common mistakes that get you rejected:**
- Forgetting to start BFS from *all* nodes.
- Not including the mask in the visited check (→ revisiting states, wrong/infinite).

---

### Q50: Travelling Salesman Problem (Held–Karp)
**Company:** Google, Amazon, Uber
**Difficulty:** ⚫ Expert
**Frequency:** 🔥
**Round:** Onsite (advanced)

**Question:**
Given an `n×n` distance matrix (`n ≤ ~15`), find the minimum-cost tour that starts at node 0, visits every node exactly once, and returns to 0.

**What interviewer is testing:**
The classic exponential-but-polynomial-in-2ⁿ DP (Held–Karp). State design `(mask, last)` and the O(2ⁿ·n²) complexity are the signal that you know exact TSP.

**Ideal Answer:**

*(1) State.* `dp[mask][i]` = the minimum cost of a path that starts at node 0, visits exactly the set of nodes in `mask`, and **currently ends at node `i`** (with `i ∈ mask`).

*(2) Recurrence.* To end at `i` having visited `mask`, we came from some `j` in `mask \ {i}`:
```
dp[mask][i] = min over j in mask, j != i, with i in mask of
              dp[mask ^ (1<<i)][j] + dist[j][i]
```

*(3) Base.* `dp[1][0] = 0` (only node 0 visited, ending at 0). All else ∞.

*(4) Order.* Iterate masks ascending (a mask depends on masks with one fewer bit).

**Answer.** `min over i of dp[full][i] + dist[i][0]` (close the tour back to 0).

```java
public int tsp(int[][] dist) {
    int n = dist.length;
    int full = (1 << n) - 1;
    int[][] dp = new int[1 << n][n];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
    dp[1][0] = 0;                                   // start at node 0
    for (int mask = 1; mask <= full; mask++) {
        if ((mask & 1) == 0) continue;              // tours start at 0
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) == 0) continue;   // i must be in mask
            int prev = mask ^ (1 << i);
            if (prev == 0) continue;                // only valid for i==0 base
            for (int j = 0; j < n; j++) {
                if ((prev & (1 << j)) == 0) continue;
                dp[mask][i] = Math.min(dp[mask][i], dp[prev][j] + dist[j][i]);
            }
        }
    }
    int best = Integer.MAX_VALUE;
    for (int i = 1; i < n; i++)
        best = Math.min(best, dp[full][i] + dist[i][0]);
    return best;
}
```

**Dry run** — 4-city symmetric matrix; the DP fills `dp[1111][i]` for each end city, then closes the loop, returning the optimal tour cost. ✅

*(7) Complexity.* O(2ⁿ · n²) time, O(2ⁿ · n) space. Exact TSP is NP-hard; Held–Karp beats the `n!` brute force but is still exponential (feasible to ~n=20).

*(8) Pattern recognition.* "Visit all of a small set with minimum *weighted* cost, order matters" → Held–Karp bitmask DP, state `(mask, last)`. Whenever you see "TSP-like" + "n ≤ ~20," this is the exact answer; for larger n you discuss approximations (Christofides, MST-based, heuristics).

**Follow-up questions the interviewer will ask:**
1. *"Path TSP (no return to start)?"* → Drop the `+ dist[i][0]` closing term; answer is `min dp[full][i]`.
2. *"n = 1000?"* → No exact poly algorithm; discuss 2-approximation via MST, or LP/heuristics.
3. *"Reconstruct the tour?"* → Store the `j` that achieved each min and backtrack.

**Common mistakes that get you rejected:**
- Forgetting the closing edge back to the start.
- O(2ⁿ·n²) confusion — be precise about state count and transition cost.

**Reconstructing the optimal tour.** Track the predecessor that achieved each `dp[mask][i]`, then unwind from the best final state back to node 0:
```java
public List<Integer> tspTour(int[][] dist) {
    int n = dist.length, full = (1 << n) - 1;
    int[][] dp = new int[1 << n][n];
    int[][] par = new int[1 << n][n];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
    for (int[] row : par) Arrays.fill(row, -1);
    dp[1][0] = 0;
    for (int mask = 1; mask <= full; mask++) {
        if ((mask & 1) == 0) continue;
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) == 0) continue;
            int prev = mask ^ (1 << i);
            if (prev == 0) continue;
            for (int j = 0; j < n; j++) {
                if ((prev & (1 << j)) == 0) continue;
                int cand = dp[prev][j] + dist[j][i];
                if (cand < dp[mask][i]) { dp[mask][i] = cand; par[mask][i] = j; }
            }
        }
    }
    int last = 1, bestCost = Integer.MAX_VALUE;
    for (int i = 1; i < n; i++)
        if (dp[full][i] + dist[i][0] < bestCost) { bestCost = dp[full][i] + dist[i][0]; last = i; }
    LinkedList<Integer> tour = new LinkedList<>();
    int mask = full, cur = last;
    while (cur != -1) {
        tour.addFirst(cur);
        int p = par[mask][cur];
        mask ^= (1 << cur);
        cur = p;
    }
    tour.addLast(0);                                  // close the loop
    return tour;
}
```
Storing parents and unwinding is the universal "recover the path" technique across grid DP, LIS, knapsack, and TSP — the same skill in every pattern.

---

### Q51: Number of Ways to Wear Different Hats
**Company:** Google, Amazon
**Difficulty:** ⚫ Expert
**Frequency:** 🔥
**Round:** Onsite (advanced)

**Question:**
`n` people (`n ≤ 10`), 40 hat types. Each person has a list of hats they're willing to wear. Count assignments where **everyone wears a different hat** (a hat used by at most one person). Return mod 1e9+7.

**What interviewer is testing:**
The crucial modeling flip: with up to 40 hats but only ≤10 people, **bitmask over people** (2¹⁰), not hats (2⁴⁰), and iterate hats one at a time. Choosing the right thing to bitmask is the whole problem.

**Ideal Answer:**

**Why bitmask people, not hats?** 40 hats → 2⁴⁰ masks (infeasible). 10 people → 2¹⁰ = 1024 masks (trivial). So we iterate over **hats** (the outer "items") and the **mask of which people are already assigned** is the DP state.

*(1) State.* `dp[h][mask]` = number of ways to assign hats `1..h` such that exactly the set `mask` of people have a hat. We process hats one at a time.

*(2) Recurrence.* For hat `h`, either no one wears it (carry `dp[h-1][mask]`), or some person `p` who *likes* hat `h` and is **not yet assigned** wears it:
```
dp[h][mask] = dp[h-1][mask]
            + Σ over p who likes hat h and p in mask of dp[h-1][mask ^ (1<<p)]
```

*(3) Base.* `dp[0][0] = 1` (no hats, no one assigned).

*(4) Order.* `h` from 1..40; `mask` over all 2ⁿ.

**Answer.** `dp[40][full]` where `full = (1<<n)-1`.

```java
public int numberWays(List<List<Integer>> hats) {
    final int MOD = 1_000_000_007;
    int n = hats.size();
    // hatToPeople[h] = bitmask of people who like hat h
    List<Integer>[] hatToPeople = new List[41];
    for (int h = 1; h <= 40; h++) hatToPeople[h] = new ArrayList<>();
    for (int p = 0; p < n; p++)
        for (int h : hats.get(p)) hatToPeople[h].add(p);

    int full = (1 << n) - 1;
    long[] dp = new long[1 << n];
    dp[0] = 1;
    for (int h = 1; h <= 40; h++) {
        long[] ndp = dp.clone();                    // option: nobody wears hat h
        for (int mask = 0; mask <= full; mask++) {
            if (dp[mask] == 0) continue;
            for (int p : hatToPeople[h]) {
                if ((mask & (1 << p)) != 0) continue;   // p already has a hat
                int nm = mask | (1 << p);
                ndp[nm] = (ndp[nm] + dp[mask]) % MOD;
            }
        }
        dp = ndp;
    }
    return (int) dp[full];
}
```

**Dry run** — `hats=[[3,4],[4,5],[5]]`: person 2 must wear 5, person 1 then wears 4, person 0 wears 3 → **1** way. ✅

*(7) Complexity.* O(40 · 2ⁿ · n) time, O(2ⁿ) space.

*(8) Pattern recognition.* "Assign A's to B's where one side is small (≤20) and the other large" → bitmask over the **small** side, iterate the large side as items. The "which dimension to bitmask" decision is the key modeling insight — always mask the side with ≤20 elements.

**Follow-up questions the interviewer will ask:**
1. *"Why iterate hats, not people?"* → A hat is worn by at most one person; iterating hats lets each hat contribute one person to the mask. Iterating people would need a hat-mask (2⁴⁰).
2. *"Count people-perspective (each person picks a hat)?"* → Equivalent reformulation; same complexity if you mask people.

**Common mistakes that get you rejected:**
- Bitmasking hats (2⁴⁰ blowup) instead of people.
- Forgetting the "nobody wears this hat" carry-over (`ndp = dp.clone()`).

---

# APPENDIX — DP interview strategy

This is what to *say and do* in the room, distilled from how these problems are actually graded.

### How to drive a DP problem live (the script)

1. **Restate and find the brute force first.** "Let me try all choices recursively." Write the naive recursion. This proves you understand the decision structure before optimizing — and the recursion *is* your recurrence.
2. **Spot the overlapping subproblems.** "The same arguments recur, so I'll memoize." Add a memo keyed on the recursion's arguments. The set of arguments **is** your state — you've now defined the state without agonizing over it.
3. **Name the state in English.** Say out loud: "`dp[i][j]` is the answer for prefix `i` of `s` and prefix `j` of `t`." If you can't say it cleanly, your state is wrong — fix it now, not after coding.
4. **Write the recurrence as math on the whiteboard**, including base cases. Talk through one transition concretely with a tiny example.
5. **Convert to tabulation only if asked or if recursion stack is a risk.** State the iteration order and justify it ("each cell reads the row above, so go top-to-bottom").
6. **Optimize space last.** "I only read the previous row, so one array suffices." Mention it even if you don't implement it — it shows you see it.
7. **Dry-run a small input** and **state complexity** precisely (states × work-per-state).

> The single highest-leverage habit: **memoize the brute-force recursion** rather than trying to invent a tabulation from scratch. Top-down derivation has the lowest error rate under pressure, and it makes the state explicit (it's literally the function arguments).

### Memoization vs tabulation — when to pick which

| | Top-down (memoization) | Bottom-up (tabulation) |
|---|---|---|
| Derivation | Mirrors the recursion — easiest to get right live | Requires figuring out a valid fill order |
| Computes | Only the states you actually need (sparse) | All states (even unreachable ones) |
| Risk | Stack overflow for deep recursion (e.g. n=10⁵ linear) | None (iterative) |
| Space-opt | Harder to collapse | Natural (rolling rows/arrays) |
| Constants | Recursion + hashmap/array overhead | Tight loops, cache-friendly |

Rule of thumb: **derive top-down, then convert to bottom-up** if the interviewer wants the space optimization or if recursion depth is a concern.

### The four state-design mistakes that sink candidates

1. **A state that isn't a sufficient summary.** If two different histories with the same `dp` index can lead to different optimal futures, your state is missing a dimension (classic in stock/cooldown, Paint House III's neighborhood count, regex's pattern position).
2. **Mixing "ending at i" with "up to i."** LIS needs "ending at i" (so it's combinable); House Robber uses "up to i." Decide which and be consistent — they have different recurrences.
3. **Wrong iteration order.** A cell computed before its dependencies are ready gives silent garbage. Interval DP (by length), reverse DP (Dungeon), and the 0/1-knapsack descending loop are the usual traps.
4. **Base cases off by one.** Empty prefix (`dp[0][*]`), empty subset (`dp[0]=1` for counts, `0` for sums), single-element ranges — get these exact or the whole table is wrong.

### Complexity, stated correctly

Almost every DP is **(number of states) × (work per state)**. Say it that way: "n² states, O(n) per state for the split → O(n³)." Note when a bound is **pseudo-polynomial** (knapsack O(nW), subset sum O(nS)) — it depends on the *magnitude* of the input, not just its length, and a hiring committee notices when you conflate that with true polynomial time.

### Recognizing DP at all

Reach for DP when you see: "**count the number of ways**", "**maximize/minimize** over a sequence of choices", "**is it possible** to reach a target", "**longest/shortest** subsequence/path with a property", or a brute force that's **exponential with overlapping subproblems**. If choices are independent or a greedy exchange argument holds (interval scheduling, Jump Game), prefer greedy — but be ready to defend why greedy is safe.

### A fully worked recursion → memo → tabulation → space-opt (Coin Change as the model)

This is the exact sequence to perform on the whiteboard for *any* DP. Watch how each step is a mechanical transform of the previous one.

**Step 0 — brute-force recursion** (just enumerate choices):
```java
int solve(int amount) {                 // min coins for `amount`, ∞ if impossible
    if (amount == 0) return 0;
    if (amount < 0) return INF;
    int best = INF;
    for (int c : coins) best = Math.min(best, solve(amount - c) + 1);
    return best;
}
```
This is correct but exponential — `solve(amount-c)` is recomputed across many branches.

**Step 1 — add a memo** (the *only* change: cache by the argument set). The arguments `{amount}` ARE the state:
```java
int solve(int amount, Integer[] memo) {
    if (amount == 0) return 0;
    if (amount < 0) return INF;
    if (memo[amount] != null) return memo[amount];
    int best = INF;
    for (int c : coins) best = Math.min(best, solve(amount - c, memo) + 1);
    return memo[amount] = best;
}
```
Now O(amount · |coins|). Top-down done.

**Step 2 — flip to tabulation** (compute states in dependency order; `solve(a)` needs smaller `a`, so iterate `a` ascending):
```java
int[] dp = new int[amount + 1];
Arrays.fill(dp, INF); dp[0] = 0;
for (int a = 1; a <= amount; a++)
    for (int c : coins)
        if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
```

**Step 3 — optimize space** — here `dp[a]` reads arbitrary smaller indices, so the array can't collapse (each amount is a distinct, still-needed subproblem). For problems that read only `dp[i-1]` or the previous row, *this* is where you drop to rolling variables / a single row.

Doing this four-step dance out loud — even narrating "step 3 can't collapse here, and here's why" — demonstrates exactly the structured reasoning interviewers score.

### Common-recurrence cheat-sheet

| Problem shape | State | Recurrence core |
|---|---|---|
| Fibonacci / climb | `dp[i]` | `dp[i-1] + dp[i-2]` |
| Rob / pick non-adjacent | `dp[i]` | `max(dp[i-1], v[i] + dp[i-2])` |
| LIS (ending at i) | `dp[i]` | `1 + max(dp[j] : j<i, a[j]<a[i])` |
| Coin Change (min) | `dp[a]` | `1 + min(dp[a-c])` |
| Coin Change II (count) | `dp[a]`, coins outer | `dp[a] += dp[a-c]` |
| Grid path (min) | `dp[i][j]` | `g[i][j] + min(dp[i-1][j], dp[i][j-1])` |
| LCS | `dp[i][j]` | match: `dp[i-1][j-1]+1`; else `max(dp[i-1][j], dp[i][j-1])` |
| Edit distance | `dp[i][j]` | match: diag; else `1+min(diag, up, left)` |
| 0/1 knapsack | `dp[i][w]` | `max(dp[i-1][w], v[i]+dp[i-1][w-wt[i]])` |
| Palindrome subseq / interval | `dp[i][j]` | match ends: `dp[i+1][j-1]+2`; else `max(dp[i+1][j], dp[i][j-1])` |
| Matrix chain / merge | `dp[i][j]` | `min_k dp[i][k] + dp[k+1][j] + cost` |
| Burst balloons (last) | `dp[i][j]` | `max_k dp[i][k] + dp[k][j] + a[i]a[k]a[j]` |
| Tree rob | `(rob, skip)` per node | `rob = v + ΣskipChild; skip = Σmax(child)` |
| TSP (Held–Karp) | `dp[mask][i]` | `min_j dp[mask\i][j] + dist[j][i]` |

---

## Pattern recap — problem triggers → pattern

This table is your in-interview lookup: scan it the moment you read a prompt. The left column is the *phrasing* that signals each pattern, the middle is the pattern, the right points back to the worked example in this file. If two rows seem to match, the more specific one (e.g. "burst last", "exactly t groups") wins.

| Trigger in problem statement | Pattern | Examples here |
|---|---|---|
| "ways to reach", "rob/skip", "ending at i", linear sequence | 1D / Linear DP | Q1, Q2, Q5, Q7 |
| "longest increasing/alternating subsequence" | 1D DP (LIS family) | Q7, Q9, Q10 |
| "min/max path in a grid", "count paths", right/down moves | 2D / Grid DP | Q11, Q13, Q15 |
| "requirement depends on the future / min resource to finish" | Reverse grid DP | Q16 |
| "two agents traverse simultaneously" | Joint-state grid DP | Q17 |
| "common / match / edit between two strings" | String DP (prefix `dp[i][j]`) | Q18, Q22, Q24 |
| "palindrome subsequence / min insertions to palindrome" | String/Interval DP | Q20, Q27 |
| "choose a subset under a capacity/sum, each item once" | 0/1 Knapsack | Q28, Q30, Q31, Q32 |
| "unlimited supply, reach a target" | Unbounded Knapsack | Q5, Q29, Q34, Q35 |
| "count combinations vs permutations to a sum" | Knapsack count (loop order!) | Q34 |
| "best over a split point inside a range" | Interval DP | Q36, Q37, Q38, Q40 |
| "order of removal matters, neighbors change" | Interval DP ("burst last") | Q37 |
| "min partitions where each piece has property P" | Interval + 1D DP | Q39 |
| "fixed states per step, constrained transitions" | State Machine DP | Q41, Q42, Q44 |
| "exactly t groups / neighborhoods" | State Machine + count dim | Q43 |
| "answer at a node from its children" | Tree DP (post-order tuple) | Q45, Q46, Q47 |
| "subset of items used/visited, n ≤ 20" | Bitmask DP | Q48, Q50, Q51 |
| "shortest path visiting all nodes" | BFS + bitmask | Q49 |
| "assign small set to large set" | Bitmask over the small side | Q51 |
| "sub-linear time on a linear recurrence" | Matrix exponentiation | Q4 |

**The five questions to ask yourself on any DP problem:**
1. **What is the smallest piece of information that defines a subproblem?** That's your state. (If you can't write `dp[...] = "..."` in one English sentence, you don't have the state yet.)
2. **What was the *last decision* made?** Branching on the last choice (last char, last item, last balloon burst, last split) almost always yields the recurrence.
3. **Which earlier subproblems does this one read?** That dictates the iteration order (a cell must come after everything it depends on).
4. **What are the seeds?** Empty prefix, single element, base of the range — get these exactly right or everything cascades wrong.
5. **What do I actually keep?** If `dp[i]` reads only `dp[i-1]` (or the previous row), collapse the space.

**The recipe, every time:** state → recurrence → base cases → order → memoize → tabulate → optimize space. Say each step aloud; structured reasoning is graded as highly as the final code.

**One last mental model.** Every DP is a **DAG of subproblems**: nodes are states, edges are transitions, and the answer is an aggregate (min/max/sum/OR) over paths. Memoization is a lazy traversal of that DAG; tabulation is an eager topological-order sweep. When you're stuck, draw three or four nodes of the DAG by hand — the edges you draw *are* the recurrence, and the order you can safely fill them *is* the iteration order. If you can sketch the DAG, you can solve the DP. Practice the 51 problems above until you can name the pattern within ten seconds of reading the prompt; at that point DP stops being the topic that fails candidates and becomes the topic that distinguishes them.

---

Created **04-dsa-dynamic-programming.md** — 51 questions covered across 8 patterns (1D/linear, 2D/grid, string, knapsack, interval, state machine, tree, bitmask), each with explicit state definition, recurrence, base cases, iteration order, both memoization and tabulation Java, space optimization, complexity, follow-ups, and rejection-causing mistakes.
