# 06 — DSA: Backtracking, Bit Manipulation & Math

> The "either you've seen the trick or you haven't" file. Backtracking is the universal hammer for *enumerate-all / find-one-valid-configuration* problems — once you internalize the choose→explore→unchoose rhythm, Subsets, Permutations, N-Queens and Sudoku are the *same* function with different constraints. Bit manipulation is where strong engineers separate from average ones in O(1)-space puzzles. Math problems are the ones that look trivial and quietly reject candidates on overflow, edge cases, and the one number-theory identity they didn't know. Every question here has a **complete** answer — brute force → optimal, full Java, dry run / recursion tree, complexity proof, follow-ups, and the mistakes that flip a "hire" to a "no hire."

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the *real* signal. Knowing this changes how you talk while you code.
- **Ideal Answer** — approach reasoning, then production-quality Java, then complexity.
- **Follow-ups** — where most candidates fail. These are scripted in real loops.
- **Common mistakes** — the specific things that flip a "lean hire" to a "no hire."

For **every** backtracking problem you'll find an ASCII recursion tree, the pruning that cuts dead branches, and a complexity argument expressed as **branching factor ^ depth**. That last skill — estimating cost as `b^d` — is what lets you say "this is roughly O(2ⁿ) / O(n!) / O(4ⁿ)" out loud before the interviewer asks.

---

## Table of Contents

**Backtracking** (the choose→explore→unchoose engine)
1. Subsets (I)
2. Subsets II (with duplicates)
3. Permutations (I)
4. Permutations II (with duplicates)
5. Combinations (n choose k)
6. Combination Sum I (unlimited reuse)
7. Combination Sum II (each used once, duplicates in input)
8. Combination Sum III (k numbers, fixed sum, 1–9)
9. Combination Sum IV (count — and why it's actually DP)
10. Letter Combinations of a Phone Number
11. Generate Parentheses
12. Palindrome Partitioning
13. N-Queens (the classic — board + solution count)
14. Sudoku Solver
15. Restore IP Addresses
16. Partition to K Equal Sum Subsets
17. Rat in a Maze
18. Unique Paths III
19. Expression Add Operators
20. Beautiful Arrangement
21. All Paths From Source to Target
22. Word Search (grid DFS backtracking)

**Bit Manipulation**
23. Single Number I (XOR)
24. Single Number II (every other thrice — 3-state bit counting)
25. Single Number III (two singletons — XOR + lowest set bit grouping)
26. Counting Bits (0..n)
27. Number of 1 Bits (Brian Kernighan)
28. Power of Two
29. Power of Four
30. Reverse Bits
31. Sum of Two Integers (no `+`)
32. Bitwise AND of Numbers Range
33. Subsets using a bitmask
34. Maximum XOR of Two Numbers in an Array
35. Minimum Flips to Make a OR b Equal to c
36. Total Hamming Distance
37. Gray Code
38. Essential bit tricks every engineer must know (reference)

**Math & Number Theory**
39. GCD / LCM (Euclidean algorithm)
40. Sieve of Eratosthenes / Count Primes
41. Pow(x, n) — fast exponentiation
42. Sqrt(x) — binary search + Newton's method
43. Happy Number (Floyd's cycle)
44. Factorial Trailing Zeroes (count 5s)
45. Excel Sheet Column Number ↔ Title (base-26)
46. Roman to Integer / Integer to Roman
47. Multiply Strings (big-number multiplication)
48. Fraction to Recurring Decimal (long division)
49. Permutation Sequence (factorial number system)
50. Next Greater Element III
51. Modular arithmetic basics (mod add/mul, modular inverse, Fermat) — reference

---

# BACKTRACKING

Backtracking is **DFS over the space of partial solutions**. You build a candidate incrementally; at each step you *choose* an option, *explore* deeper, then *unchoose* (undo the choice) so the next iteration starts clean. It's brute force made disciplined: you prune branches the moment they can't lead to a valid answer.

## The reusable template

Memorize this skeleton. Every problem below is a specialization of it.

```java
void backtrack(State state, Choices choices, List<Result> results) {
    if (isComplete(state)) {        // base case: a full valid candidate
        results.add(snapshot(state)); // RECORD a copy — state is mutated later
        return;
    }
    for (Choice c : choices(state)) {   // each option from here
        if (!isValid(c, state)) continue;  // PRUNE invalid branches early
        state.apply(c);                 // CHOOSE
        backtrack(state, choices, results); // EXPLORE
        state.undo(c);                  // UNCHOOSE (backtrack)
    }
}
```

**The three moves:**

1. **Choose** — commit to one option (add to the current path, place a queen, fill a cell).
2. **Explore** — recurse to extend the partial solution with that choice locked in.
3. **Unchoose** — undo the choice so the loop can try the next option on a clean slate. This is the single line beginners forget, and it corrupts every sibling branch.

**Why the snapshot matters:** the path/board is a *single mutable object* shared across the whole recursion. If you store the reference directly (`results.add(path)`) instead of a copy (`results.add(new ArrayList<>(path))`), every result ends up pointing at the same object — which is empty by the time recursion unwinds. This is the #1 backtracking bug.

**Two ways to enumerate without duplicates:**
- A **`start` index** that only moves forward → used when order doesn't matter (subsets, combinations). Prevents `[1,2]` and `[2,1]` both appearing.
- A **`used[]` boolean array** → used when order *does* matter (permutations) so you can revisit earlier elements but never the same physical slot twice.

**Pruning** is what turns an exponential brute force into something that finishes. Sort the input so you can `break` (not just `continue`) when the remaining choices can't possibly help — e.g. in Combination Sum, once `candidate > remaining` the rest are larger too, so stop.

**Reading complexity as `b^d`:** count the branching factor `b` (choices per node) and the depth `d` (levels of recursion). Subsets: each element is in/out → `b=2`, `d=n` → `2ⁿ`. Permutations: `n` then `n−1` then … → `n!`. Combination Sum with target T and min value m: depth up to `T/m`, branching up to `n` → bounded but exponential. State this *before* coding.

---

### Q1: Subsets (I)
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an array `nums` of **distinct** integers, return all possible subsets (the power set). The solution set must not contain duplicate subsets. Order of subsets doesn't matter.

**What interviewer is testing:**
The cleanest possible backtracking warm-up: do you understand that each element is an independent **in-or-out** binary choice, and can you produce all `2ⁿ` combinations without duplicates using a `start` index? It's the canonical "explain choose/explore/unchoose" problem.

**Ideal Answer:**

There are three standard ways; know all three and pick backtracking as the lead because it generalizes to Subsets II.

*Approach 1 — Cascading (iterative):* start with `[[]]`; for each number, append it to **copies** of every existing subset. After processing all numbers you have the full power set.

*Approach 2 — Bitmask:* there are exactly `2ⁿ` subsets. Iterate `mask` from `0` to `2ⁿ − 1`; bit `j` set means include `nums[j]`. (This is Q33; clean and iterative.)

*Approach 3 — Backtracking (the one to lead with):* DFS where at index `start` you decide the rest. Crucially, **every node in the recursion tree is a valid subset** — so we record the path at *entry*, not only at the leaves.

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] nums, int start, List<Integer> path,
                       List<List<Integer>> res) {
    res.add(new ArrayList<>(path));        // RECORD every node (snapshot copy!)
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);                 // CHOOSE
        backtrack(nums, i + 1, path, res); // EXPLORE (i+1: never reuse, never go back)
        path.remove(path.size() - 1);      // UNCHOOSE
    }
}
```

**Recursion tree** for `nums = [1,2,3]` (each node = the path recorded there):

```
                       []                       start=0
        /               |               \
      [1]              [2]              [3]      start=1,2,3
     /    \             |
  [1,2]  [1,3]       [2,3]                       start=2,3
    |
 [1,2,3]                                         start=3
```

Every one of the 8 nodes (`[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]`) is added to the result — that's `2³ = 8` subsets. The `i + 1` recursion (not `start + 1`) is what keeps each branch moving strictly rightward, so `[2,1]` can never appear after `[1,2]`.

**Dry run** — entering with `path=[]`: record `[]`. i=0 → add 1 → recurse(start=1): record `[1]`; i=1 → add 2 → recurse(start=2): record `[1,2]`; i=2 → add 3 → record `[1,2,3]`; unwind, remove 3, remove 2; i=2 from start=1 → add 3 → record `[1,3]`; … and so on. ✅

**Complexity:** Time **O(n · 2ⁿ)** — there are `2ⁿ` subsets and copying each path costs up to O(n). Space O(n) recursion depth + O(n·2ⁿ) output. The `b^d` view: branching is "include the next element or stop," depth n → `2ⁿ` leaves *plus* internal nodes, all recorded.

**Follow-ups:**
1. *"Iterative without recursion?"* → Cascading or bitmask (Approaches 1 & 2).
2. *"What if the array has duplicates?"* → Subsets II (Q2): sort + skip duplicate siblings.
3. *"Generate the k-th subset directly?"* → Treat `k` as a bitmask over `n` bits.
4. *"Only subsets of a given size?"* → That's Combinations (Q5) — add a size check at the base case.

**Common mistakes:**
- `res.add(path)` instead of `res.add(new ArrayList<>(path))` → all results alias the same list (ends up all empty). **The** classic bug.
- Recursing with `start + 1` instead of `i + 1` → misses subsets / produces wrong branching.
- Forgetting to record internal nodes (only recording at depth n) → returns only the full set.

---

### Q2: Subsets II (with duplicates)
**Company:** Amazon, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given an integer array `nums` that **may contain duplicates**, return all possible subsets. The solution set must not contain duplicate subsets.

**What interviewer is testing:**
The duplicate-skipping idiom — the single most reused trick across the "II" backtracking problems (Subsets II, Combination Sum II, Permutations II). Can you articulate *why* `i > start && nums[i] == nums[i-1]` is the correct condition?

**Ideal Answer:**

Sort first so equal values are adjacent. The rule: **within a single recursion level (same `start`), use each distinct value only once.** If we're at position `i > start` and `nums[i] == nums[i-1]`, then the branch starting with `nums[i-1]` already produced every subset this branch would — skip it.

The condition `i > start` is precise: it lets us *use* a duplicate when it's the **first** choice at this level (extending the previous pick deeper, e.g. building `[2,2]`), but blocks it as a **sibling** choice (which would re-root an identical subtree).

```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);                     // group duplicates together
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] nums, int start, List<Integer> path,
                       List<List<Integer>> res) {
    res.add(new ArrayList<>(path));
    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate SIBLING
        path.add(nums[i]);
        backtrack(nums, i + 1, path, res);
        path.remove(path.size() - 1);
    }
}
```

**Recursion tree** for `nums = [1,2,2]` (note the pruned branch):

```
                   []                      start=0
        /           |          \
      [1]          [2]      (skip 2nd 2)   i>start & dup → CUT
     /   \          |
  [1,2] (skip)   [2,2]                     start=2
    |
[1,2,2]
```

Without the skip we'd get `[2]` twice and `[1,2]` twice. The cut at the second `2` when it's a sibling removes exactly those duplicates. Result: `[], [1], [1,2], [1,2,2], [2], [2,2]` — 6 unique subsets.

**Dry run** — sorted `[1,2,2]`: record `[]`; i=0 add 1 → record `[1]`; i=1 add 2 → record `[1,2]`; i=2 add 2 → record `[1,2,2]`; back at start=1 under root, i=1 add 2 → record `[2]`; i=2 add 2 → record `[2,2]`; back at root i=2: `i>start(0)` and `nums[2]==nums[1]` → **skip**. ✅

**Complexity:** Time O(n · 2ⁿ) worst case (all distinct), less with duplicates. Space O(n) recursion + output.

**Follow-ups:**
1. *"Why `i > start` and not `i > 0`?"* → `i > 0` would wrongly block legitimately extending the same value deeper (building `[2,2]`). The skip must only apply to *siblings* at the same level.
2. *"Without sorting?"* → Use a `Set<Integer>` of values *used at the current level* and skip a value if already used this level. Works but allocates a set per call.

**Common mistakes:**
- Forgetting to sort → adjacent-duplicate logic breaks entirely.
- Using `i > 0` instead of `i > start` → loses valid subsets like `[2,2]`.
- Trying to dedup with a global `HashSet<List>` of results — works but O(n) hashing per insert and signals you missed the level-skip insight.

---

### Q3: Permutations (I)
**Company:** Amazon, Google, Meta, Microsoft, Apple
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an array `nums` of **distinct** integers, return all possible permutations (in any order).

**What interviewer is testing:**
The shift from "subsets" mechanics (start index, order irrelevant) to "permutations" mechanics (order matters, every element used exactly once). Do you reach for a `used[]` array — or the in-place swap variant — and can you explain the `n!` complexity via `b^d`?

**Ideal Answer:**

Order matters and we use **all** elements, so there's no `start` index — at each depth we may pick any element not already in the path. Track membership with a `boolean[] used`. The path is complete when its length equals `n`.

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, new boolean[nums.length], new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> path,
                       List<List<Integer>> res) {
    if (path.size() == nums.length) {       // base case: full permutation
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;              // skip elements already placed
        used[i] = true;  path.add(nums[i]); // CHOOSE
        backtrack(nums, used, path, res);   // EXPLORE
        used[i] = false; path.remove(path.size() - 1); // UNCHOOSE
    }
}
```

**Recursion tree** for `nums = [1,2,3]`:

```
                         ( )                       depth 0, choose from {1,2,3}
        /                 |                 \
      (1)                (2)                (3)     depth 1
     /    \             /    \             /    \
  (1,2)  (1,3)       (2,1)  (2,3)       (3,1)  (3,2) depth 2
    |      |            |      |            |     |
(1,2,3)(1,3,2)     (2,1,3)(2,3,1)     (3,1,2)(3,2,1) depth 3 = leaves
```

Branching factor shrinks each level: 3 → 2 → 1, giving `3 · 2 · 1 = 3! = 6` leaves, each a permutation.

**Dry run** — pick 1 (used[0]=T) → pick 2 → pick 3 → record `[1,2,3]`; unwind to depth1=(1): unpick 3, pick 3 instead of 2 path → record `[1,3,2]`; etc. ✅

**Complexity:** Time **O(n · n!)** — `n!` permutations, O(n) to copy each. Space O(n) recursion + `used[]`. The `b^d`: product of decreasing branching `n·(n−1)···1 = n!`.

**Alternative — in-place swap (no `used[]`):** swap to fix each prefix position, recurse, swap back. Same `n!` but O(1) auxiliary beyond the array.

```java
public List<List<Integer>> permuteSwap(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    swapBacktrack(nums, 0, res);
    return res;
}
private void swapBacktrack(int[] nums, int k, List<List<Integer>> res) {
    if (k == nums.length) {
        List<Integer> p = new ArrayList<>();
        for (int x : nums) p.add(x);
        res.add(p);
        return;
    }
    for (int i = k; i < nums.length; i++) {
        swap(nums, k, i);                 // fix nums[i] at position k
        swapBacktrack(nums, k + 1, res);
        swap(nums, k, i);                 // restore
    }
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Follow-ups:**
1. *"With duplicates?"* → Permutations II (Q4); sort + skip used-duplicate siblings.
2. *"Next permutation only (not all)?"* → O(n) in-place algorithm (covered in file 01).
3. *"k-th permutation directly?"* → Factorial number system (Q49).

**Common mistakes:**
- Forgetting to reset `used[i] = false` on unchoose → only one permutation generated.
- In the swap version, copying `nums` *and* using a path list (redundant).
- Snapshot-aliasing bug (same as Q1).

---

### Q4: Permutations II (with duplicates)
**Company:** Amazon, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a collection `nums` that **might contain duplicates**, return all **unique** permutations.

**What interviewer is testing:**
Combining the `used[]` permutation engine with duplicate skipping. The skip condition is subtly different from Subsets II because here we revisit earlier indices.

**Ideal Answer:**

Sort so duplicates are adjacent. The rule: at a given depth, among equal values use them **left to right** — only pick `nums[i]` if its identical predecessor `nums[i-1]` has already been used **in this path**. Condition: `nums[i] == nums[i-1] && !used[i-1]` → skip.

Why `!used[i-1]`? If `used[i-1]` is true, the previous equal element is part of the current path *above* us, so picking `nums[i]` now extends a different prefix — legitimate. If it's false, the previous equal element is *available at this same level*, meaning we already had (or will have) a sibling branch starting with it that produces identical permutations — skip to avoid duplicates.

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);                     // group duplicates
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, new boolean[nums.length], new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> path,
                       List<List<Integer>> res) {
    if (path.size() == nums.length) {
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // skip duplicate value whose equal predecessor is unused at this level
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
        used[i] = true;  path.add(nums[i]);
        backtrack(nums, used, path, res);
        used[i] = false; path.remove(path.size() - 1);
    }
}
```

**Recursion tree** for `nums = [1,1,2]` (subscripts distinguish the two 1s):

```
                    ( )
        /            |            \
     (1₀)        (1₁ SKIP)        (2)        1₁ skipped: dup & !used[1₀]
    /     \                      /    \
 (1₀,1₁) (1₀,2)             (2,1₀)  (2,1₁ SKIP)
    |       |                   |
(1₀,1₁,2)(1₀,2,1₁)        (2,1₀,1₁)
```

Three unique permutations: `[1,1,2], [1,2,1], [2,1,1]`. The skips removed the mirror branches that would have duplicated them.

**Dry run** — sorted `[1,1,2]`: i=0 pick 1₀ → i=1 pick 1₁ (used[0]=T so allowed) → pick 2 → `[1,1,2]`; back, pick 2 then 1₁ → `[1,2,1]`; back to root i=1: `nums[1]==nums[0] && !used[0]` → **skip**; i=2 pick 2 → pick 1₀ → pick 1₁ → `[2,1,1]`. ✅

**Complexity:** Time O(n · n!) worst case (all distinct), fewer with duplicates. Space O(n).

**Follow-ups:**
1. *"Why `!used[i-1]` and not `used[i-1]`?"* → Walk through `[1,1]`: with `!used[i-1]` we get exactly one `[1,1]`; with `used[i-1]` we'd get zero (over-pruned). Be ready to trace it.
2. *"Count distinct permutations without generating them?"* → `n! / (∏ count[v]!)` (multinomial coefficient).

**Common mistakes:**
- Flipping the condition to `used[i-1]` → over-prunes, drops valid permutations.
- Forgetting to sort.
- Using a per-level `HashSet` instead (works, but the `!used[i-1]` idiom is the expected answer).

---

### Q5: Combinations (n choose k)
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given two integers `n` and `k`, return all combinations of `k` numbers chosen from the range `[1, n]`.

**What interviewer is testing:**
Subsets with a fixed-size base case, plus a non-obvious but high-value **pruning** optimization (stop early when not enough numbers remain to reach size `k`).

**Ideal Answer:**

Like Subsets, but only record paths of length exactly `k`. Use a `start` index so order doesn't matter.

The key optimization: if we still need `k − path.size()` more numbers but fewer than that remain in `[i, n]`, this branch is hopeless — prune the loop bound. Numbers remaining from `i` is `n − i + 1`; we need `need = k − path.size()`. So `i` can go at most up to `n − need + 1`.

```java
public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(n, k, 1, new ArrayList<>(), res);
    return res;
}

private void backtrack(int n, int k, int start, List<Integer> path,
                       List<List<Integer>> res) {
    if (path.size() == k) {                 // exactly k chosen
        res.add(new ArrayList<>(path));
        return;
    }
    int need = k - path.size();
    // PRUNE: i can't exceed n - need + 1, else not enough numbers left
    for (int i = start; i <= n - need + 1; i++) {
        path.add(i);
        backtrack(n, k, i + 1, path, res);
        path.remove(path.size() - 1);
    }
}
```

**Recursion tree** for `n=4, k=2` (pruned bound in effect):

```
                 ( )  need=2, i<=3
        /         |          \
      (1)        (2)         (3)        i up to 3 (not 4: can't pair)
    /  |  \      /  \          |
 (1,2)(1,3)(1,4)(2,3)(2,4)  (3,4)       need=1, i<=4
```

Six combinations: `[1,2],[1,3],[1,4],[2,3],[2,4],[3,4] = C(4,2)`. Note the root never branches into `(4)` — with `need=2` and only `4` left, it's pruned.

**Dry run** — start=1, need=2, bound=3: pick 1 → start=2,need=1,bound=4: pick 2 → `[1,2]`; pick 3 → `[1,3]`; pick 4 → `[1,4]`; back; pick 2 → pick 3,4 → `[2,3],[2,4]`; pick 3 → pick 4 → `[3,4]`. ✅

**Complexity:** Time O(k · C(n,k)) — `C(n,k)` combinations, O(k) to copy each. Space O(k) recursion depth.

**Follow-ups:**
1. *"Quantify the pruning savings?"* → Without it you'd explore branches that can never reach size k; the bound prunes a constant-ish factor but is the expected polish.
2. *"Combinations with repetition allowed?"* → Recurse with `i` (not `i+1`) — same idea as Combination Sum.

**Common mistakes:**
- Loop bound `i <= n` without pruning (correct but slower; interviewers want the prune).
- Off-by-one in `n - need + 1`.

---

### Q6: Combination Sum I (unlimited reuse)
**Company:** Amazon, Google, Meta, Microsoft, Uber, Airbnb
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an array of **distinct** integers `candidates` and a `target`, return all unique combinations where the chosen numbers sum to `target`. **Each number may be used unlimited times.** Combinations are unordered.

**What interviewer is testing:**
The "reuse allowed" twist: recurse with `i` (not `i + 1`) so the same element can be picked again. Plus sorting-based pruning (`break` when a candidate already exceeds the remaining target).

**Ideal Answer:**

DFS carrying the *remaining* target. At index `i`, either pick `candidates[i]` (and recurse from `i` again, since reuse is allowed) or move on. Sort the candidates so that once `candidates[i] > remaining` we can `break` — all later candidates are larger too.

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates);               // enables the break-prune
    List<List<Integer>> res = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] cand, int remaining, int start,
                       List<Integer> path, List<List<Integer>> res) {
    if (remaining == 0) {                   // exact hit
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i < cand.length; i++) {
        if (cand[i] > remaining) break;     // PRUNE: sorted, so rest are larger too
        path.add(cand[i]);                  // CHOOSE
        backtrack(cand, remaining - cand[i], i, path, res); // reuse: pass i, not i+1
        path.remove(path.size() - 1);       // UNCHOOSE
    }
}
```

**Recursion tree** for `candidates = [2,3,6,7]`, `target = 7`:

```
                         rem=7 start=0
        /            |           |        \
   pick2 rem=5    pick3 rem=4  pick6 rem=1  pick7 rem=0 ✓ [7]
   start=0        start=1      start=2
   /   \            |          (6>1 break)
 pick2  pick3     pick3 rem=1
 rem=3  rem=2     (no hit)
 /  \   ...
pick2 pick3
rem=1 rem=0 ✓
      [2,2,3]
```

Following the leftmost spine: `2,2,3` sums to 7 ✓; the lone `7` branch hits rem=0 ✓. Result: `[[2,2,3],[7]]`.

**Dry run** — sorted `[2,3,6,7]`, target 7: pick 2 (rem5), pick 2 (rem3), pick 2 (rem1) → 2>1 break; pick 3 (rem0) → record `[2,2,3]`; … eventually pick 7 (rem0) → `[7]`. ✅

**Complexity:** Time O(n^(T/m)) where `m` = smallest candidate, `T` = target — depth bounded by `T/m`, branching by `n`. Space O(T/m) recursion depth. The `b^d`: branching ≤ n, depth ≤ T/m.

**Follow-ups:**
1. *"Each number used at most once + duplicates in input?"* → Combination Sum II (Q7).
2. *"Just count the combinations, order matters?"* → Combination Sum IV — that's DP, not backtracking (Q9).
3. *"Why pass `i` not `i+1`?"* → Reuse; passing `i+1` would forbid repeats. Passing `i` (not `0`) avoids permutation-style duplicates.

**Common mistakes:**
- Passing `0` instead of `i` as the next `start` → produces `[2,2,3]` *and* `[2,3,2]` etc. (duplicate permutations).
- `continue` instead of `break` after sorting (correct but misses the prune; with `break` you stop the whole loop).
- Not sorting, then trying to `break` (incorrect).

---

### Q7: Combination Sum II (each used once, duplicates in input)
**Company:** Amazon, Microsoft, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given a collection `candidates` (which **may contain duplicates**) and a `target`, find all unique combinations summing to `target`. **Each number may be used once.** No duplicate combinations.

**What interviewer is testing:**
Fusing Combination Sum's target-tracking with Subsets II's duplicate-sibling skip. The two changes from Q6: recurse with `i + 1` (each used once) and add the `i > start && cand[i]==cand[i-1]` skip.

**Ideal Answer:**

Sort. Recurse from `i + 1` so each physical element is used at most once. Skip a candidate that equals its predecessor *as a sibling* (`i > start`) to avoid duplicate combinations.

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);
    List<List<Integer>> res = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), res);
    return res;
}

private void backtrack(int[] cand, int remaining, int start,
                       List<Integer> path, List<List<Integer>> res) {
    if (remaining == 0) {
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i < cand.length; i++) {
        if (cand[i] > remaining) break;                  // sorted prune
        if (i > start && cand[i] == cand[i - 1]) continue; // skip duplicate sibling
        path.add(cand[i]);
        backtrack(cand, remaining - cand[i], i + 1, path, res); // i+1: use once
        path.remove(path.size() - 1);
    }
}
```

**Recursion tree** for `candidates = [1,1,2,5,6,7,10]`, `target = 8` (showing the dedup):

```
                          rem=8 start=0
      /            \            \         \      ...
  pick1₀ rem=7   (1₁ SKIP)    pick2 rem=6  pick5 rem=3 ...
  start=1                      start=3
   |                            |
 pick1₁ rem=6                 (no completion shown)
 start=2
   |
 pick2 rem=4 ... pick6 rem=0 ✓ [1,1,6]
```

The second `1` (`1₁`) is skipped as a sibling of `1₀` at the root, but *used* when extending `1₀` deeper — yielding `[1,1,6]` exactly once. Other results: `[1,2,5]`, `[1,7]`, `[2,6]`.

**Dry run** — sorted `[1,1,2,5,6,7,10]`, target 8: pick 1₀ (rem7), pick 1₁ (rem6, allowed since i==start here), pick 2 (rem4)… pick 6 → `[1,1,6]`; back to start=1 under 1₀: pick 2 (rem5), pick 5 → wait rem after 2 is 5, pick 5 → `[1,2,5]`; pick 7 → `[1,7]`; back to root i=1: skip 1₁ (sibling dup); i=2 pick 2 (rem6), pick 6 → `[2,6]`. ✅

**Complexity:** Time O(2ⁿ) worst case (subset enumeration with pruning). Space O(n) recursion depth.

**Follow-ups:**
1. *"Difference from Combination Sum I in code?"* → `i+1` vs `i`, plus the dedup skip. Articulate both.
2. *"What if the same number can be used up to its multiplicity only?"* → Exactly what this does; the input multiplicity bounds reuse naturally.

**Common mistakes:**
- Recursing with `i` instead of `i+1` (would allow reuse — wrong problem).
- Omitting the dedup skip → duplicate combinations like `[1,7]` twice (once via each `1`).
- `i > 0` instead of `i > start`.

---

### Q8: Combination Sum III (k numbers, fixed sum, 1–9)
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Find all valid combinations of **`k` distinct numbers** from `1..9` that sum to `n`. Each number used at most once; no duplicate combinations.

**What interviewer is testing:**
Combining a size constraint (`k`) with a sum constraint (`n`) over a fixed small domain. Double pruning: stop when the sum exceeds `n` and when not enough numbers remain.

**Ideal Answer:**

DFS over `1..9` with a `start` index. Base case: `path.size() == k && remaining == 0`. Prune when `remaining < 0` or when the candidate already exceeds remaining (sorted ascending naturally).

```java
public List<List<Integer>> combinationSum3(int k, int n) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(k, n, 1, new ArrayList<>(), res);
    return res;
}

private void backtrack(int k, int remaining, int start,
                       List<Integer> path, List<List<Integer>> res) {
    if (path.size() == k) {
        if (remaining == 0) res.add(new ArrayList<>(path)); // exactly k and sum hit
        return;
    }
    for (int i = start; i <= 9; i++) {
        if (i > remaining) break;            // sorted: rest are larger → prune
        path.add(i);
        backtrack(k, remaining - i, i + 1, path, res);
        path.remove(path.size() - 1);
    }
}
```

**Recursion tree** for `k=3, n=7`:

```
                    rem=7 size=0 start=1
        /          |          \        ...
     (1)rem=6    (2)rem=5    (3)rem=4   (i up to 7)
    /  |  \       /  \         |
 (1,2)(1,3)..  (2,3)(2,4)..  (3,4)..
 rem=4 rem=3    rem=2  ...
   |    |
(1,2,4)(1,3,3✗ can't reuse 3)
 size3 rem0 ✓
```

Result: `[[1,2,4]]` — the only triple of distinct 1–9 numbers summing to 7.

**Dry run** — k=3,n=7: pick 1 (rem6), pick 2 (rem4), pick 4 (rem0, size3) → `[1,2,4]`; pick 3 (rem3, size2)... 3 already used, pick from 4: 4>3 break; back; pick 1, pick 3 (rem3), pick from 4: 4>3 break; … no other completes. ✅

**Complexity:** Time O(C(9,k)) bounded — at most `C(9,k)` combinations. Space O(k).

**Follow-ups:**
1. *"Tighter pruning?"* → Also prune if the maximum possible remaining sum (largest unused numbers) can't reach `remaining`.
2. *"General domain `1..m`?"* → Replace `9` with `m`.

**Common mistakes:**
- Checking `remaining == 0` before checking `path.size() == k` → accepts wrong-size combos.
- Allowing numbers > 9 or reuse.

---

### Q9: Combination Sum IV (count — and why it's actually DP)
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given distinct integers `nums` and a `target`, return the **number** of possible combinations that add up to `target`. **Different orderings count as different** (so `(1,3)` and `(3,1)` are two). Numbers may be reused.

**What interviewer is testing:**
The trap: this *looks* like a backtracking problem but "count + order matters + reuse" makes the search space explode (it's effectively counting compositions). The right tool is **DP** — this question tests whether you recognize when to *abandon* backtracking.

**Ideal Answer:**

Because order matters and counts can be astronomically large, enumerating with backtracking would be exponential and could overflow. Instead, define `dp[t]` = number of ordered combinations summing to `t`. For each target `t`, the last number added could be any `num ≤ t`, leaving `t − num`:

`dp[t] = Σ dp[t − num]` for each `num` in `nums` with `num ≤ t`, base `dp[0] = 1`.

This is the "coin change — count arrangements" recurrence. The loop order matters: **target outer, nums inner** counts *ordered* sequences (compositions). (Nums outer, target inner would count unordered combinations — that's the classic Coin Change II.)

```java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;                              // one way to make 0: take nothing
    for (int t = 1; t <= target; t++) {     // target OUTER → ordered (permutations)
        for (int num : nums) {
            if (num <= t) dp[t] += dp[t - num];
        }
    }
    return dp[target];
}
```

**Dry run** — `nums = [1,2,3]`, `target = 4`:
- dp[0]=1
- dp[1] = dp[0] = 1 → {(1)}
- dp[2] = dp[1]+dp[0] = 2 → {(1,1),(2)}
- dp[3] = dp[2]+dp[1]+dp[0] = 4 → {(1,1,1),(1,2),(2,1),(3)}
- dp[4] = dp[3]+dp[2]+dp[1] = 4+2+1 = 7 ✅

**Complexity:** Time O(target · n), Space O(target).

**Follow-ups:**
1. *"What if negative numbers are allowed?"* → Infinite combinations possible (e.g. `+1,−1` loops); you'd need an explicit cap on sequence length, and DP no longer applies cleanly.
2. *"Why is this DP and not backtracking?"* → Counting (not listing) + order-sensitive + reuse → overlapping subproblems → DP. The count can be huge; listing is infeasible.
3. *"Order doesn't matter version?"* → Swap loop nesting (nums outer) — that's Coin Change II.

**Common mistakes:**
- Backtracking to enumerate then counting — TLE / overflow.
- Swapping the loop order and silently counting the wrong thing.
- `dp[t]` overflow — values can exceed `int`; problem typically guarantees fit, but mention `long` if asked about large targets.

---

### Q10: Letter Combinations of a Phone Number
**Company:** Amazon, Google, Meta, Microsoft, Uber, Dropbox
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Given a string of digits `2-9`, return all possible letter combinations the number could spell (classic phone keypad mapping). Return in any order. Empty input → empty list.

**What interviewer is testing:**
Backtracking over a **Cartesian product** where the branching factor varies per level (3 or 4 letters per digit). Clean base case and mapping handling.

**Ideal Answer:**

Each digit maps to 3–4 letters. Build the string digit by digit; at depth `d` (the d-th digit), branch over that digit's letters. Base case: depth equals the input length → record.

```java
private static final String[] MAP = {
    "", "",            // 0,1 unused
    "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
};

public List<String> letterCombinations(String digits) {
    List<String> res = new ArrayList<>();
    if (digits == null || digits.isEmpty()) return res;
    backtrack(digits, 0, new StringBuilder(), res);
    return res;
}

private void backtrack(String digits, int index, StringBuilder sb,
                       List<String> res) {
    if (index == digits.length()) {        // built one full combination
        res.add(sb.toString());
        return;
    }
    String letters = MAP[digits.charAt(index) - '0'];
    for (int i = 0; i < letters.length(); i++) {
        sb.append(letters.charAt(i));      // CHOOSE
        backtrack(digits, index + 1, sb, res); // EXPLORE
        sb.deleteCharAt(sb.length() - 1);  // UNCHOOSE
    }
}
```

**Recursion tree** for `digits = "23"` (`2→abc`, `3→def`):

```
                       ""  index=0
        /              |              \
      "a"             "b"             "c"     index=1
    / | \           / | \           / | \
 "ad""ae""af"   "bd""be""bf"    "cd""ce""cf"  index=2 = leaves
```

`3 × 3 = 9` combinations: `ad,ae,af,bd,be,bf,cd,ce,cf`.

**Dry run** — "23": index0 letters "abc": append 'a' → index1 letters "def": append 'd' → index2==len → record "ad"; remove 'd', append 'e' → "ae"; … then back, 'a'→'b', etc. ✅

**Complexity:** Time O(4ⁿ · n) where `n` = number of digits (each contributes ≤4 branches; n to build each string). Space O(n) recursion depth. The `b^d`: b ≤ 4, depth = n → ≤ `4ⁿ` leaves.

**Follow-ups:**
1. *"Iterative?"* → BFS/queue: start with `[""]`, for each digit expand every current string by its letters.
2. *"Very long input — memory?"* → The output itself is exponential; if you only need to *stream* combinations, yield them instead of collecting.

**Common mistakes:**
- Sharing a `String` and concatenating (O(n²) garbage) — use `StringBuilder` and undo.
- Not handling empty input → returning `[""]` instead of `[]`.
- Indexing `MAP` wrong (forgetting digits start at '2').

---

### Q11: Generate Parentheses
**Company:** Amazon, Google, Meta, Microsoft, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given `n` pairs of parentheses, generate all combinations of **well-formed** parentheses.

**What interviewer is testing:**
Constraint-driven pruning. The naive approach generates all `2^(2n)` strings then filters; the elegant one *only ever builds valid prefixes* by maintaining counts of open and close brackets.

**Ideal Answer:**

Track `open` (count of `(` used) and `close` (count of `)` used). Two rules keep every prefix valid:
- Add `(` only while `open < n`.
- Add `)` only while `close < open` (never close more than you've opened).

Base case: total length `2n` (equivalently `open == n && close == n`).

```java
public List<String> generateParenthesis(int n) {
    List<String> res = new ArrayList<>();
    backtrack(n, 0, 0, new StringBuilder(), res);
    return res;
}

private void backtrack(int n, int open, int close, StringBuilder sb,
                       List<String> res) {
    if (sb.length() == 2 * n) {            // used all n pairs
        res.add(sb.toString());
        return;
    }
    if (open < n) {                         // can open another
        sb.append('(');
        backtrack(n, open + 1, close, sb, res);
        sb.deleteCharAt(sb.length() - 1);
    }
    if (close < open) {                     // can close (prefix stays valid)
        sb.append(')');
        backtrack(n, open, close + 1, sb, res);
        sb.deleteCharAt(sb.length() - 1);
    }
}
```

**Recursion tree** for `n = 2` (pruned: invalid prefixes never created):

```
                    ""  (o0 c0)
                     |
                    "("  (o1 c0)
              /            \
          "(("            "()"
        (o2 c0)          (o1 c1)
            |               |
        "(()"            "()("
       (o2 c1)          (o2 c1)
            |               |
       "(())" ✓         "()()" ✓
```

Two valid strings: `(())` and `()()`. Notice the tree never branches into `")"` first or `"())"` — those prefixes are pruned by the rules.

**Dry run** — n=2: "" → "(" (o1) → "((" (o2) → "(()" (c1) → "(())" record; back; "(" → "()" (c1) → "()(" (o2) → "()()" record. ✅

**Complexity:** Time **O(4ⁿ / √n)** — the n-th Catalan number `C(n) = (1/(n+1))·C(2n,n)` counts valid strings, asymptotically `4ⁿ/(n^1.5)`. Space O(n) recursion depth. State "Catalan number of results" — interviewers love that.

**Follow-ups:**
1. *"How many results?"* → The n-th Catalan number. C(0..)=1,1,2,5,14,42,…
2. *"Validate a parentheses string (mixed brackets)?"* → Stack-based matching.
3. *"Count without generating?"* → Catalan formula directly.

**Common mistakes:**
- Generating all `2^(2n)` strings then validating (works, far slower).
- `close < open` written as `close < n` → produces invalid strings like `())(`.
- Forgetting to undo the append.

---

### Q12: Palindrome Partitioning
**Company:** Amazon, Google, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a string `s`, partition it such that every substring of the partition is a palindrome. Return all possible palindrome partitionings.

**What interviewer is testing:**
Backtracking where the "choice" is *where to cut*. Combining substring enumeration with a palindrome check, and recognizing the optional DP precompute to speed up checks.

**Ideal Answer:**

At position `start`, try every possible end `i` (`start..n-1`); if `s[start..i]` is a palindrome, fix that as the next piece and recurse from `i + 1`. Base case: `start == n` (consumed the whole string).

```java
public List<List<String>> partition(String s) {
    List<List<String>> res = new ArrayList<>();
    backtrack(s, 0, new ArrayList<>(), res);
    return res;
}

private void backtrack(String s, int start, List<String> path,
                       List<List<String>> res) {
    if (start == s.length()) {              // fully partitioned
        res.add(new ArrayList<>(path));
        return;
    }
    for (int end = start; end < s.length(); end++) {
        if (isPalindrome(s, start, end)) {  // only cut where the piece is valid
            path.add(s.substring(start, end + 1)); // CHOOSE
            backtrack(s, end + 1, path, res);       // EXPLORE
            path.remove(path.size() - 1);           // UNCHOOSE
        }
    }
}

private boolean isPalindrome(String s, int lo, int hi) {
    while (lo < hi) {
        if (s.charAt(lo++) != s.charAt(hi--)) return false;
    }
    return true;
}
```

**Recursion tree** for `s = "aab"`:

```
                      start=0
        /                          \
   cut "a" (palin)            cut "aa" (palin)   ("aab" not palin → no branch)
   start=1                    start=2
      |                          |
   cut "a"                    cut "b"
   start=2                    start=3 ✓ ["aa","b"]
      |
   cut "b"
   start=3 ✓ ["a","a","b"]
```

Results: `[["a","a","b"], ["aa","b"]]`.

**Dry run** — "aab": start0, end0 "a" palin → start1, end1 "a" palin → start2, end2 "b" palin → start3 record `[a,a,b]`; back; start0 end1 "aa" palin → start2 end2 "b" → record `[aa,b]`; start0 end2 "aab" not palin. ✅

**Complexity:** Time **O(n · 2ⁿ)** — up to `2^(n-1)` partitions (each gap is cut-or-not), O(n) per palindrome check / copy. Space O(n) recursion. The `b^d`: each of `n−1` gaps is a binary cut decision → `2^(n-1)`.

**Optimization — precompute palindromes:** a DP table `pal[i][j] = s[i..j] is palindrome` (O(n²) build) makes each check O(1), reducing repeated work.

```java
// pal[i][j] true if s[i..j] is a palindrome
boolean[][] pal = new boolean[n][n];
for (int i = n - 1; i >= 0; i--)
    for (int j = i; j < n; j++)
        pal[i][j] = s.charAt(i) == s.charAt(j) && (j - i < 2 || pal[i + 1][j - 1]);
```

**Follow-ups:**
1. *"Minimum cuts (Palindrome Partitioning II)?"* → Pure DP, not backtracking: `dp[i]` = min cuts for prefix of length i.
2. *"Count partitions only?"* → DP over the palindrome table.

**Common mistakes:**
- Re-checking palindromes naively inside deep recursion (acceptable but mention the DP precompute).
- Off-by-one in `substring(start, end + 1)`.

---

### Q13: N-Queens (the classic — board + solution count)
**Company:** Google, Amazon, Microsoft, Meta, Adobe
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Place `n` queens on an `n × n` chessboard so that no two attack each other (no two share a row, column, or diagonal). Return all distinct board configurations. (Variant: just **count** solutions — N-Queens II.)

**What interviewer is testing:**
The flagship backtracking problem. Can you (1) place one queen per row to fix the branching, (2) check column + both diagonals in O(1) using sets/arrays, (3) reconstruct and visualize the board, and (4) reason about the `b^d` blowup and how pruning tames it?

**Ideal Answer:**

Place **exactly one queen per row** — that automatically prevents row conflicts and fixes depth = n, branching = n (the columns). For each row, try every column; a column is safe if it's not used and neither diagonal is occupied.

Diagonal identity (the key trick):
- Cells on the same **"/" (anti)diagonal** share `row + col`.
- Cells on the same **"\" diagonal** share `row − col` (offset by `n−1` to index an array, or use a set).

Track three boolean structures: `cols`, `diag1` (`row+col`), `diag2` (`row−col`). O(1) safety check.

```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> res = new ArrayList<>();
    int[] queenCol = new int[n];           // queenCol[r] = column of queen in row r
    boolean[] cols  = new boolean[n];
    boolean[] diag1 = new boolean[2 * n - 1]; // row + col
    boolean[] diag2 = new boolean[2 * n - 1]; // row - col + (n-1)
    backtrack(0, n, queenCol, cols, diag1, diag2, res);
    return res;
}

private void backtrack(int row, int n, int[] queenCol,
                       boolean[] cols, boolean[] diag1, boolean[] diag2,
                       List<List<String>> res) {
    if (row == n) {                         // all rows filled → a valid board
        res.add(build(queenCol, n));
        return;
    }
    for (int col = 0; col < n; col++) {
        int d1 = row + col, d2 = row - col + (n - 1);
        if (cols[col] || diag1[d1] || diag2[d2]) continue; // PRUNE: under attack
        cols[col] = diag1[d1] = diag2[d2] = true;          // CHOOSE
        queenCol[row] = col;
        backtrack(row + 1, n, queenCol, cols, diag1, diag2, res); // EXPLORE
        cols[col] = diag1[d1] = diag2[d2] = false;         // UNCHOOSE
    }
}

private List<String> build(int[] queenCol, int n) {
    List<String> board = new ArrayList<>();
    for (int r = 0; r < n; r++) {
        char[] rowChars = new char[n];
        Arrays.fill(rowChars, '.');
        rowChars[queenCol[r]] = 'Q';
        board.add(new String(rowChars));
    }
    return board;
}
```

**Visualizing a solution** for `n = 4` — there are exactly **2** solutions:

```
Solution 1:        Solution 2:
. Q . .            . . Q .
. . . Q            Q . . .
Q . . .            . . . Q
. . Q .            . Q . .
```

**Recursion tree** for `n = 4` (rows are depth; `×` = pruned by attack):

```
row0:        col0      col1      col2      col3
              |          |
row1:   col2,col3✓?   col3 ✓      ...     (many cols pruned by diagonals)
              ...        |
row2:                  col0 ... eventually a full placement or dead end → backtrack
              ...
row3:    place last queen if a safe col exists, else backtrack
```

Most branches die quickly: after placing row 0 at col 1, row 1 can only use col 3 (cols 0,1,2 attacked by column/diagonal), and so on — the diagonal pruning collapses the `4⁴ = 256` naive placements down to a handful of paths reaching depth 4.

**Dry run** — n=4, start row0 col0: row1 cols 0(col),1(diag),2(diag2? 1-2 ok but check)… place col2; row2: col0 conflicts diag with row1? trace shows dead end → backtrack; eventually row0=col1 leads to the first solution `[.Q.., ...Q, Q..., ..Q.]`. ✅

**Complexity:** Time **O(n!)** upper bound (≤ n choices row 0, ≤ n−1 effective row 1, …), but pruning makes it far less in practice. Space O(n) for the tracking arrays + O(n²) per recorded board. For **counting only** (N-Queens II), drop the board build and just increment a counter.

```java
// N-Queens II: count solutions only
public int totalNQueens(int n) {
    return count(0, n, new boolean[n], new boolean[2*n-1], new boolean[2*n-1]);
}
private int count(int row, int n, boolean[] cols, boolean[] d1, boolean[] d2) {
    if (row == n) return 1;
    int total = 0;
    for (int col = 0; col < n; col++) {
        int a = row + col, b = row - col + n - 1;
        if (cols[col] || d1[a] || d2[b]) continue;
        cols[col] = d1[a] = d2[b] = true;
        total += count(row + 1, n, cols, d1, d2);
        cols[col] = d1[a] = d2[b] = false;
    }
    return total;
}
```

Solution counts: n=1→1, 2→0, 3→0, 4→2, 5→10, 6→4, 7→40, 8→**92**.

**Follow-ups:**
1. *"Bitmask optimization?"* → Represent cols/diag1/diag2 as integer bitmasks; `available = ~(cols|d1|d2) & ((1<<n)-1)`; iterate set bits with `p = avail & -avail`. Faster, O(1) per check, classic optimization.
2. *"Why one queen per row?"* → Eliminates the row-conflict dimension and bounds depth at n; placing arbitrarily would explode the search.
3. *"How many solutions for n=8?"* → 92 (12 fundamental up to symmetry).

**Common mistakes:**
- Forgetting one (or both) diagonals — only checking column.
- Diagonal index arithmetic off by one (`row - col` can be negative → add `n−1`).
- Not undoing all three flags on unchoose.

---

### Q14: Sudoku Solver
**Company:** Google, Amazon, Microsoft, Uber, Apple
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Write a program to solve a 9×9 Sudoku by filling empty cells (`'.'`). The solution must obey: each row, column, and 3×3 box contains digits `1-9` exactly once. A unique solution is guaranteed.

**What interviewer is testing:**
Constraint-propagation backtracking on a 2D grid. Efficient O(1) validity checks (row/col/box sets), finding the next empty cell, and the "return true to stop" early-exit pattern (we want one solution, not all).

**Ideal Answer:**

Backtrack cell by cell. For the next empty cell, try digits `1-9`; place a digit only if it doesn't already appear in that cell's row, column, or 3×3 box. Recurse; if the recursion eventually fills the board, propagate `true` up to stop. Otherwise undo and try the next digit. If no digit works, return `false` (dead end → caller backtracks).

Use three arrays of boolean sets sized `[9][9]` for O(1) checks: `rows[r][d]`, `cols[c][d]`, `boxes[b][d]` where box index `b = (r/3)*3 + c/3`.

```java
public void solveSudoku(char[][] board) {
    boolean[][] rows = new boolean[9][9];
    boolean[][] cols = new boolean[9][9];
    boolean[][] boxes = new boolean[9][9];
    // seed the constraint sets from the given clues
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (board[r][c] != '.') {
                int d = board[r][c] - '1';
                rows[r][d] = cols[c][d] = boxes[box(r, c)][d] = true;
            }
        }
    }
    solve(board, 0, 0, rows, cols, boxes);
}

private int box(int r, int c) { return (r / 3) * 3 + c / 3; }

private boolean solve(char[][] b, int r, int c,
                      boolean[][] rows, boolean[][] cols, boolean[][] boxes) {
    if (r == 9) return true;                       // filled all rows → done
    int nr = (c == 8) ? r + 1 : r;                 // next cell coordinates
    int nc = (c == 8) ? 0 : c + 1;
    if (b[r][c] != '.') {                          // skip pre-filled cells
        return solve(b, nr, nc, rows, cols, boxes);
    }
    int boxIdx = box(r, c);
    for (int d = 0; d < 9; d++) {                  // try digits 1..9
        if (rows[r][d] || cols[c][d] || boxes[boxIdx][d]) continue; // PRUNE
        b[r][c] = (char) ('1' + d);                // CHOOSE
        rows[r][d] = cols[c][d] = boxes[boxIdx][d] = true;
        if (solve(b, nr, nc, rows, cols, boxes)) return true; // EXPLORE; bubble success
        b[r][c] = '.';                             // UNCHOOSE
        rows[r][d] = cols[c][d] = boxes[boxIdx][d] = false;
    }
    return false;                                  // no digit fits → backtrack
}
```

**How the search prunes:** at each empty cell only the digits not already in its row/col/box are tried. Early in the solve a cell might allow 4–5 digits; as the board fills, constraints cascade and most cells allow only 1–2, so the effective branching factor drops far below 9. That's why a `9^81` worst case finishes near-instantly on real puzzles.

**Dry run (conceptual):** find first empty cell, say (0,2). Suppose row0 has {5,3}, col2 has {8}, box0 has {5,3,6}. Allowed digits = {1,2,4,7,9}. Try 1 → recurse to next empty cell; if downstream hits a contradiction (no digit fits some cell), it returns false, we erase the 1, try 2, and so on until the board completes and `true` bubbles all the way up.

**Complexity:** Time O(9^m) worst case where `m` = number of empty cells (heavily pruned in practice). Space O(1) extra (fixed 9×9 sets) + O(m) recursion depth.

**Follow-ups:**
1. *"Optimize cell selection?"* → MRV heuristic: always fill the empty cell with the **fewest** candidate digits first → drastically smaller search tree.
2. *"Validate a Sudoku board (not solve)?"* → Single pass with the same three sets; return false on any duplicate.
3. *"Bitmask the sets?"* → Use a 9-bit `int` per row/col/box; `candidates = ~(rows[r]|cols[c]|boxes[b]) & 0x1FF`.

**Common mistakes:**
- Not propagating `true` up (returning `void`) → can't stop at the first solution and may overwrite it.
- Forgetting to undo the three set flags on backtrack.
- Recomputing row/col/box membership by scanning (O(9) each) instead of O(1) sets — correct but slow.
- Wrong box index formula.

---

### Q15: Restore IP Addresses
**Company:** Amazon, Microsoft, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a string `s` of digits, return all valid IP address strings formable by inserting dots. A valid IPv4 has 4 parts, each `0–255`, no leading zeros (except the single digit `0`).

**What interviewer is testing:**
Backtracking with **multiple validity constraints per segment** (length, range, leading zero) and a fixed structure (exactly 4 segments). Heavy pruning opportunities.

**Ideal Answer:**

Place 3 dots, splitting `s` into 4 segments. Backtrack: at each step pick the next segment of length 1–3, validate it (no leading zero unless "0", value ≤ 255), and recurse. Base case: 4 segments used **and** all characters consumed.

```java
public List<String> restoreIpAddresses(String s) {
    List<String> res = new ArrayList<>();
    if (s.length() < 4 || s.length() > 12) return res; // quick reject
    backtrack(s, 0, 0, new StringBuilder(), res);
    return res;
}

private void backtrack(String s, int start, int part, StringBuilder sb,
                       List<String> res) {
    if (part == 4) {                        // used all 4 segments
        if (start == s.length()) {          // and consumed all digits
            res.add(sb.substring(0, sb.length() - 1)); // drop trailing dot
        }
        return;
    }
    // each segment is 1..3 chars; don't run past the string
    for (int len = 1; len <= 3 && start + len <= s.length(); len++) {
        String seg = s.substring(start, start + len);
        if (!isValid(seg)) continue;        // PRUNE invalid segments
        int restore = sb.length();
        sb.append(seg).append('.');         // CHOOSE
        backtrack(s, start + len, part + 1, sb, res); // EXPLORE
        sb.setLength(restore);              // UNCHOOSE
    }
}

private boolean isValid(String seg) {
    if (seg.length() > 1 && seg.charAt(0) == '0') return false; // leading zero
    return Integer.parseInt(seg) <= 255;    // range (length ≤ 3 so no overflow)
}
```

**Recursion tree** for `s = "25525511135"` (one valid path highlighted):

```
"255" . "255" . "111" . "35"  ✓ 255.255.111.35
"255" . "255" . "11"  . "135" ✓ 255.255.11.135
"255" . "25"  . ...   (other splits, many pruned by range/length)
```

**Dry run** — "25525511135": part0 try "2","25","255"; take "255" → part1 try "2","25","255"; take "255" → part2 try "1","11","111"; take "111" → part3 must consume "35" (len2) → "35" valid, 4 parts, all consumed → record `255.255.111.35`. Backtrack to part2="11" → part3 "135" → `255.255.11.135`. ✅

**Complexity:** Time O(1) effectively — at most `3⁴ = 81` segment combinations regardless of input (bounded because IP length ≤ 12). Space O(1) + recursion depth 4.

**Follow-ups:**
1. *"IPv6?"* → 8 groups of 1–4 hex digits separated by `:`; similar structure.
2. *"Why is this not exponential in `n`?"* → Fixed 4 segments × max length 3 → constant branching bound.

**Common mistakes:**
- Allowing leading zeros ("01", "00").
- Forgetting to check all characters are consumed at part==4.
- Not handling the trailing dot in the result.
- Missing the length pre-check (`s.length()` must be 4–12).

---

### Q16: Partition to K Equal Sum Subsets
**Company:** Amazon, Google, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an array `nums` and integer `k`, determine if it's possible to partition the array into `k` non-empty subsets with **equal sums**.

**What interviewer is testing:**
Backtracking with bucket-filling, plus several crucial prunes: total divisible by k, sort descending to fail fast, skip cells equal to a just-skipped value, and the "if first item of an empty bucket fails, the whole thing fails" cut.

**Ideal Answer:**

Each subset must sum to `target = total / k` (must be an integer). Try to fill `k` buckets to exactly `target`. Backtrack by assigning each number to some bucket.

A cleaner, faster formulation: fill **one bucket at a time**. Track which elements are `used`; greedily complete a bucket to `target`, then start the next. When all `k` buckets are filled, success.

Key prunes:
1. If `total % k != 0` → impossible.
2. Sort **descending** so large numbers are placed first (fail fast; also lets us break when a single element > target).
3. Skip an element equal to the previous one if the previous was not used (same dedup logic as before).
4. If adding `nums[i]` exactly completes a bucket or we can recurse, do so; **if placing the first element into a fresh bucket fails, return false** (symmetry — that element must go *somewhere*, and buckets are interchangeable).

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % k != 0) return false;       // PRUNE 1
    int target = total / k;
    Arrays.sort(nums);                       // ascending; we'll iterate from the back
    if (nums[nums.length - 1] > target) return false; // largest can't fit a bucket
    boolean[] used = new boolean[nums.length];
    return fill(nums, used, k, 0, nums.length - 1, target, target);
}

// curSum builds the current bucket toward target; remainingBuckets counts buckets left
private boolean fill(int[] nums, boolean[] used, int remainingBuckets,
                     int curSum, int start, int target, final int TARGET) {
    if (remainingBuckets == 0) return true;          // all buckets done
    if (curSum == target) {                          // current bucket complete
        // start a fresh bucket from the largest available again
        return fill(nums, used, remainingBuckets - 1, 0, nums.length - 1, target, TARGET);
    }
    for (int i = start; i >= 0; i--) {               // iterate high → low
        if (used[i] || curSum + nums[i] > target) continue; // PRUNE: used or overflow
        // skip duplicate value if the equal predecessor (higher index) was skipped
        if (i < nums.length - 1 && nums[i] == nums[i + 1] && !used[i + 1]) continue;
        used[i] = true;                              // CHOOSE
        if (fill(nums, used, remainingBuckets, curSum + nums[i], i - 1, target, TARGET))
            return true;
        used[i] = false;                             // UNCHOOSE
        // symmetry prune: if this element couldn't start/extend, deeper tries won't help
        if (curSum == 0) break;                      // first element of a bucket failed
    }
    return false;
}
```

**Recursion tree (conceptual)** for `nums = [4,3,2,3,5,2,1]`, `k = 4`, target = `(20)/4 = 5`:

```
Bucket A: 5            → complete (5)
Bucket B: 4 + 1        → complete (5)
Bucket C: 3 + 2        → complete (5)
Bucket D: 3 + 2        → complete (5)  → all 4 done → TRUE
```

The descending order places 5 first (instantly completing a bucket), then 4 (needs a 1), pruning enormous swaths of the search.

**Dry run** — total=20, k=4, target=5. Sorted `[1,2,2,3,3,4,5]`. Bucket1: take 5 → sum5 complete → bucket2: take 4 → sum4, need 1 → take 1 → sum5 complete → bucket3: take 3 → take 2 → 5 → bucket4: take 3, take 2 → 5 → all done → true. ✅

**Complexity:** Time O(k · 2ⁿ) worst case — for each of `k` buckets we may try subsets of the remaining elements. The prunes make typical inputs vastly faster. Space O(n) for `used` + recursion.

**Follow-ups:**
1. *"k = 2?"* → Equal Partition Subset Sum — solvable in O(n·sum) with subset-sum DP / bitset.
2. *"Bitmask DP for small n?"* → `dp[mask]` = the running sum modulo target of the elements in `mask`, processing in a fixed order; O(2ⁿ · n). Great when n ≤ 16.
3. *"Why sort descending / iterate from the back?"* → Place big items first to fail fast and trigger the `> target` prune early.

**Common mistakes:**
- Forgetting the `total % k` and `max > target` prunes → TLE.
- Missing the `curSum == 0 → break` symmetry prune (the big speedup).
- Resetting `start` to 0 incorrectly when continuing the same bucket (should keep moving down).

---

### Q17: Rat in a Maze
**Company:** Amazon, Flipkart, Microsoft (common in Indian product company loops)
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an `n × n` grid where `1` = open cell and `0` = blocked, a rat starts at `(0,0)` and must reach `(n−1,n−1)`. It can move Down, Left, Right, Up. Return all paths as strings of moves (`D`,`L`,`R`,`U`), in lexicographic order. Cells can't be revisited within a path.

**What interviewer is testing:**
Grid DFS backtracking with a `visited` marker and ordered move generation. The "mark on enter, unmark on exit" discipline.

**Ideal Answer:**

DFS from `(0,0)`. At each cell, try the four directions in lexicographic order (`D, L, R, U`). Mark the cell visited before recursing and unmark after (backtrack), so other paths can reuse it. Record the move-string when reaching the destination.

```java
public List<String> findPaths(int[][] maze) {
    int n = maze.length;
    List<String> res = new ArrayList<>();
    if (n == 0 || maze[0][0] == 0 || maze[n - 1][n - 1] == 0) return res;
    boolean[][] visited = new boolean[n][n];
    // directions in lexicographic order: D, L, R, U
    int[] dr = { 1, 0, 0, -1 };
    int[] dc = { 0, -1, 1, 0 };
    char[] mv = { 'D', 'L', 'R', 'U' };
    dfs(maze, 0, 0, visited, dr, dc, mv, new StringBuilder(), res);
    return res;
}

private void dfs(int[][] maze, int r, int c, boolean[][] visited,
                 int[] dr, int[] dc, char[] mv, StringBuilder path,
                 List<String> res) {
    int n = maze.length;
    if (r == n - 1 && c == n - 1) {         // reached destination
        res.add(path.toString());
        return;
    }
    visited[r][c] = true;                   // CHOOSE (occupy this cell)
    for (int d = 0; d < 4; d++) {
        int nr = r + dr[d], nc = c + dc[d];
        if (nr >= 0 && nr < n && nc >= 0 && nc < n
                && maze[nr][nc] == 1 && !visited[nr][nc]) { // valid, open, unvisited
            path.append(mv[d]);
            dfs(maze, nr, nc, visited, dr, dc, mv, path, res);
            path.deleteCharAt(path.length() - 1); // UNCHOOSE the move
        }
    }
    visited[r][c] = false;                  // UNCHOOSE (free this cell)
}
```

**ASCII walk** for a 4×4 maze:

```
maze =            One valid path "DDRDRR":
1 0 0 0           (0,0)→D→(1,0)→D→(2,0)→R→(2,1)→D→(3,1)→R→(3,2)→R→(3,3)
1 1 0 1
1 1 0 0           Path overlay:
0 1 1 1           S . . .
                  X . . .
                  X X . .
                  . X X X   (X = visited cells on this path; S = start)
```

**Dry run** — start (0,0): D→(1,0) valid; from (1,0) try D→(2,0), then R→(2,1), D→(3,1), R→(3,2), R→(3,3)=dest → record "DDRDRR"; backtrack and explore alternatives (L/U pruned by bounds/visited). ✅

**Complexity:** Time O(4^(n²)) worst case (each cell up to 4 directions over up to n² cells), heavily pruned by walls and the visited set. Space O(n²) for `visited` + recursion depth.

**Follow-ups:**
1. *"Shortest path instead of all paths?"* → BFS (unweighted grid) — backtracking is for enumerating *all* paths.
2. *"Count paths only?"* → Same DFS, increment a counter at the destination.
3. *"Diagonal moves allowed?"* → Extend the direction arrays.

**Common mistakes:**
- Forgetting to unmark `visited[r][c] = false` on exit → blocks legitimate alternative paths.
- Wrong move ordering when lexicographic output is required.
- Marking destination as visited but never recording (check destination *before* the loop).

---

### Q18: Unique Paths III
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
On an `m × n` grid: `1` = start, `2` = end, `0` = walkable, `−1` = obstacle. Return the number of distinct paths from start to end that **walk over every non-obstacle square exactly once**.

**What interviewer is testing:**
Hamiltonian-path-style backtracking: you must visit *all* empty cells, so you track a remaining-cell counter and only count a path that lands on the end with zero remaining.

**Ideal Answer:**

Count walkable cells (the `0`s plus the start). DFS from the start; decrement the count as you step. When you reach the end, it's a valid path **only if** every walkable cell has been visited (`remaining == 0`). Mark/unmark cells (set to `−1` temporarily) to enforce "exactly once."

```java
public int uniquePathsIII(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int startR = 0, startC = 0, empty = 0;
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 0) empty++;
            else if (grid[r][c] == 1) { startR = r; startC = c; }
        }
    }
    // +1 to also "consume" the start cell itself
    return dfs(grid, startR, startC, empty + 1);
}

private int dfs(int[][] grid, int r, int c, int remaining) {
    int m = grid.length, n = grid[0].length;
    if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] == -1) return 0; // blocked/OOB
    if (grid[r][c] == 2) {                  // reached end
        return remaining == 0 ? 1 : 0;      // valid only if all cells used
    }
    int temp = grid[r][c];
    grid[r][c] = -1;                        // CHOOSE: occupy (mark visited)
    int paths = dfs(grid, r + 1, c, remaining - 1)
              + dfs(grid, r - 1, c, remaining - 1)
              + dfs(grid, r, c + 1, remaining - 1)
              + dfs(grid, r, c - 1, remaining - 1);
    grid[r][c] = temp;                      // UNCHOOSE: restore
    return paths;
}
```

**Note the counting convention:** we pass `empty + 1` (including the start) and decrement *before* recursing into a neighbor, so by the time we step onto the end cell, `remaining == 0` means every non-obstacle square was traversed exactly once.

**Dry run** — small grid `[[1,0,0,0],[0,0,0,0],[0,0,2,-1]]`: there are 4 distinct Hamiltonian-style paths from start to end covering all `0`s (LeetCode's canonical answer is 2 for one example; the count depends on the grid). The DFS explores all 4 directions, marking/unmarking, and only tallies paths that end with `remaining == 0`. ✅

**Complexity:** Time O(4^(m·n)) worst case (Hamiltonian path enumeration is exponential), pruned by obstacles and the visited marks. Space O(m·n) recursion depth.

**Follow-ups:**
1. *"Why mark with −1 instead of a visited matrix?"* → Reuses the obstacle check (`== -1` already means impassable), so one condition covers both. A separate `visited[][]` is equally fine.
2. *"Bitmask DP for small grids?"* → State = (cell, visited-mask); O(m·n · 2^(m·n)). Feasible only for tiny grids.

**Common mistakes:**
- Forgetting the `+1` for the start cell → off-by-one in `remaining`.
- Not restoring `grid[r][c]` (corrupts sibling branches).
- Counting paths that reach the end without covering all cells.

---

### Q19: Expression Add Operators
**Company:** Google, Meta, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a string `num` of digits and a `target`, insert binary operators `+`, `−`, `*` (or nothing) between digits so the resulting expression evaluates to `target`. Return all valid expressions. No number may have a leading zero.

**What interviewer is testing:**
One of the hardest backtracking problems: you must handle operator precedence (`*` binds tighter) **inside** the recursion by tracking the last operand, plus the leading-zero rule and overflow (`long`).

**Ideal Answer:**

Backtrack over where to cut the next operand and which operator precedes it. The trick for `*`: keep `prevOperand` — the value of the last additively-combined term. When multiplying, undo the previous addition and re-add `prev * cur`:
- For `+ cur`: new running value `= val + cur`; new `prev = +cur`.
- For `− cur`: `val − cur`; `prev = −cur`.
- For `* cur`: `val − prev + prev * cur`; new `prev = prev * cur`.

Use `long` for the running value and operands to avoid overflow. Disallow multi-digit operands starting with `0`.

```java
public List<String> addOperators(String num, int target) {
    List<String> res = new ArrayList<>();
    if (num == null || num.isEmpty()) return res;
    backtrack(num, target, 0, 0, 0, new StringBuilder(), res);
    return res;
}

// pos: index into num; val: current evaluated value; prev: last operand (signed)
private void backtrack(String num, int target, int pos, long val, long prev,
                       StringBuilder expr, List<String> res) {
    if (pos == num.length()) {
        if (val == target) res.add(expr.toString());
        return;
    }
    int len = expr.length();
    long cur = 0;
    for (int i = pos; i < num.length(); i++) {
        if (i > pos && num.charAt(pos) == '0') break; // no leading-zero multi-digit
        cur = cur * 10 + (num.charAt(i) - '0');       // extend the operand
        String s = num.substring(pos, i + 1);
        if (pos == 0) {                               // first operand: no operator
            expr.append(s);
            backtrack(num, target, i + 1, cur, cur, expr, res);
            expr.setLength(len);
        } else {
            // +
            expr.append('+').append(s);
            backtrack(num, target, i + 1, val + cur, cur, expr, res);
            expr.setLength(len);
            // -
            expr.append('-').append(s);
            backtrack(num, target, i + 1, val - cur, -cur, expr, res);
            expr.setLength(len);
            // *  (fix precedence: undo prev addition, apply multiplication)
            expr.append('*').append(s);
            backtrack(num, target, i + 1, val - prev + prev * cur, prev * cur, expr, res);
            expr.setLength(len);
        }
    }
}
```

**Why the `*` math works:** suppose so far the expression evaluated to `val` and the *last* additive term was `prev` (e.g. `... + 2`, so `prev = 2`, and `2` is included in `val`). To turn that `+2` into `+2*3`, we must *remove* the `2` we already added (`val − prev`) and add `2*3` instead (`+ prev*cur`). The new `prev` becomes `prev*cur = 6`, so a subsequent `*4` undoes 6 and adds 24. Precedence handled without a parser.

**Dry run** — `num = "123"`, `target = 6`:
- `1+2+3` = 6 ✓ (val: 1 → +2=3, prev2 → +3=6, prev3)
- `1*2*3` = 6 ✓ (val: 1 → *2: 1−1+1*2=2, prev2 → *3: 2−2+2*3=6, prev6)
- Result: `["1+2+3", "1*2*3"]`. ✅

**Complexity:** Time O(4ⁿ) — at each of `n−1` gaps we choose among {nothing-extend, +, −, *} and operands can have varying lengths; bounded by `4^(n−1)` paths. Space O(n) recursion + expression buffer.

**Follow-ups:**
1. *"Add division?"* → Much harder (integer vs. real, divide-by-zero); usually out of scope.
2. *"Why `long`?"* → Operands and intermediate products of a long digit string overflow `int`.
3. *"Single value (no leading-zero) handling?"* → `break` when `num.charAt(pos)=='0'` after the first digit allows the lone `"0"` but blocks `"05"`.

**Common mistakes:**
- Mishandling `*` precedence (treating it like `+`/`−`).
- Forgetting leading-zero rule.
- `int` overflow.
- Not skipping the operator for the first operand.

---

### Q20: Beautiful Arrangement
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Count the permutations `perm` of `1..n` such that for every position `i` (1-indexed), either `perm[i] % i == 0` or `i % perm[i] == 0`. Return the count.

**What interviewer is testing:**
Constrained permutation counting via backtracking, with the divisibility constraint enabling strong pruning (place numbers into positions only where divisibility holds).

**Ideal Answer:**

Backtrack position by position (`pos` from 1 to n). For each position try every unused number `v` that satisfies `v % pos == 0 || pos % v == 0`. Count complete arrangements. The divisibility filter prunes most branches.

```java
public int countArrangement(int n) {
    boolean[] used = new boolean[n + 1];
    return backtrack(n, 1, used);
}

private int backtrack(int n, int pos, boolean[] used) {
    if (pos > n) return 1;                  // filled all positions → one arrangement
    int count = 0;
    for (int v = 1; v <= n; v++) {
        if (!used[v] && (v % pos == 0 || pos % v == 0)) { // divisibility constraint
            used[v] = true;                 // CHOOSE
            count += backtrack(n, pos + 1, used); // EXPLORE
            used[v] = false;                // UNCHOOSE
        }
    }
    return count;
}
```

**Recursion tree** for `n = 2`:

```
              pos=1 (any v works: x%1==0)
        /                    \
   v=1 used                v=2 used
   pos=2: v=2 (2%2==0) ✓   pos=2: v=1 (2%1==0) ✓
   → count 1               → count 1
Total = 2  →  [1,2] and [2,1]
```

**Dry run** — n=2: pos1 v=1 (1%1) → pos2 v=2 (2%2) → arrangement → +1; pos1 v=2 (2%1) → pos2 v=1 (2%1) → +1. Total 2. ✅

**Complexity:** Time O(k) where k = number of valid arrangements ≤ n!, but divisibility pruning makes it far smaller. Space O(n).

**Follow-ups:**
1. *"Place from the back?"* → Assigning the *largest* positions first prunes harder (fewer divisors), a common optimization.
2. *"Bitmask DP?"* → `dp[mask]` over which numbers are used; O(2ⁿ · n). Good for memoization since the same set-of-used-numbers leads to the same count regardless of order.

**Common mistakes:**
- Iterating positions but forgetting the divisibility check both ways (`v%pos` *and* `pos%v`).
- Using 0-indexing (the constraint is 1-indexed).

---

### Q21: All Paths From Source to Target
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given a **DAG** with `n` nodes (`graph[i]` lists nodes reachable from `i`), return all paths from node `0` to node `n−1`.

**What interviewer is testing:**
DFS path enumeration on a graph. Because it's a DAG (no cycles), you don't even need a visited set — but you do need the choose/explore/unchoose path discipline.

**Ideal Answer:**

DFS from `0`, carrying the current path. At the target node `n−1`, record the path. For each neighbor, append, recurse, pop. No visited array needed (acyclic guarantees termination).

```java
public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    path.add(0);                            // always start at node 0
    dfs(graph, 0, path, res);
    return res;
}

private void dfs(int[][] graph, int node, List<Integer> path,
                 List<List<Integer>> res) {
    if (node == graph.length - 1) {         // reached target n-1
        res.add(new ArrayList<>(path));
        return;
    }
    for (int next : graph[node]) {
        path.add(next);                     // CHOOSE
        dfs(graph, next, path, res);        // EXPLORE
        path.remove(path.size() - 1);       // UNCHOOSE
    }
}
```

**Recursion tree** for `graph = [[1,2],[3],[3],[]]` (n=4, target=3):

```
                 [0]
           /            \
        [0,1]          [0,2]
          |              |
       [0,1,3] ✓      [0,2,3] ✓
```

Two paths: `[0,1,3]` and `[0,2,3]`.

**Dry run** — start [0]; neighbors of 0 = {1,2}; go 1 → path [0,1]; neighbors of 1 = {3}; go 3 = target → record [0,1,3]; back; go 2 → [0,2] → 3 → record [0,2,3]. ✅

**Complexity:** Time O(2ⁿ · n) worst case — a DAG can have exponentially many source-target paths; O(n) to copy each. Space O(n) recursion + path.

**Follow-ups:**
1. *"Graph has cycles?"* → Need a visited set to avoid infinite loops; enumerating simple paths becomes NP-hard in general.
2. *"Count paths only?"* → DFS with memoization on node → number of paths to target (O(V+E)).
3. *"Shortest path?"* → BFS / topological DP, not full enumeration.

**Common mistakes:**
- Adding a visited set unnecessarily (and then never unmarking, blocking valid paths).
- Snapshot-aliasing bug when recording.

---

### Q22: Word Search (grid DFS backtracking)
**Company:** Amazon, Microsoft, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an `m × n` board of characters and a `word`, return true if the word exists in the grid, formed by sequentially adjacent (horizontally/vertically) cells, where each cell is used at most once.

**What interviewer is testing:**
Grid DFS with in-place backtracking marking. The "mark cell, recurse 4 directions, unmark" pattern and early termination. (Word Search **II** — many words via a trie — is covered in file 03; here we focus purely on the backtracking mechanics for a single word.)

**Ideal Answer:**

Try starting the search from every cell whose character matches `word[0]`. DFS: at cell `(r,c)` matching `word[k]`, mark it (overwrite with a sentinel like `'#'`), recurse into the 4 neighbors for `word[k+1]`, then restore the character (backtrack). Success when `k` reaches `word.length()`.

```java
public boolean exist(char[][] board, String word) {
    int m = board.length, n = board[0].length;
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (dfs(board, word, r, c, 0)) return true; // try each start cell
        }
    }
    return false;
}

private boolean dfs(char[][] board, String word, int r, int c, int k) {
    if (k == word.length()) return true;    // matched the whole word
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length
            || board[r][c] != word.charAt(k)) {
        return false;                       // OOB or mismatch
    }
    char saved = board[r][c];
    board[r][c] = '#';                      // CHOOSE: mark visited
    boolean found = dfs(board, word, r + 1, c, k + 1)
                 || dfs(board, word, r - 1, c, k + 1)
                 || dfs(board, word, r, c + 1, k + 1)
                 || dfs(board, word, r, c - 1, k + 1);
    board[r][c] = saved;                    // UNCHOOSE: restore
    return found;
}
```

**ASCII walk** searching `"ABCCED"` in:

```
A B C E       Path A(0,0)→B(0,1)→C(0,2)→C(1,2)? no, →  follow:
S F C S       A(0,0)→B(0,1)→C(0,2)→C(1,2)→E? trace the real route:
A D E E       A(0,0)→B(0,1)→C(0,2)→C(1,2)... actual: A→B→C→C→E→D
              A B C
              . . C
              . D E   (cells used, forming A-B-C-C-E-D snake) → true
```

**Dry run** — word "ABCCED": start at (0,0)='A'=word[0]; mark '#'; try right (0,1)='B'=word[1]; mark; right (0,2)='C'=word[2]; mark; down (1,2)='C'=word[3]; mark; down (2,2)='E'=word[4]; mark; left (2,1)='D'=word[5]; k=6==len → true. ✅

**Complexity:** Time O(m·n·4^L) where `L` = word length (each cell starts a search, branching ≤4 down to depth L, the first direction has 4 then 3 options). Space O(L) recursion depth (in-place marking uses no extra grid).

**Follow-ups:**
1. *"Search many words at once?"* → Word Search II: build a **trie** of all words and DFS once, pruning by trie prefixes. (See file 03.)
2. *"Why mark in place instead of a visited matrix?"* → Saves O(m·n) space; restore on unwind keeps it correct.
3. *"Reuse cells allowed?"* → Drop the marking — but then it's a different (often trivial) problem.

**Common mistakes:**
- Forgetting to restore the cell (`board[r][c] = saved`) → blocks valid paths that reuse the cell in a *different* search.
- Checking bounds/match in the caller instead of at the top of `dfs` (clutters the four recursive calls).
- Using the sentinel character as a real board character (pick one guaranteed absent).

---

# BIT MANIPULATION

The mental model: an `int` is **32 independent on/off switches**. Most bit problems reduce to one of a handful of identities — XOR cancels pairs, `n & (n-1)` clears the lowest set bit, `n & (-n)` isolates it. The interviewer is usually testing whether you *know the identity*, not whether you can re-derive it under pressure. Memorize the reference section (Q38); it pays off in dozens of problems.

Two facts you'll lean on constantly:
- **XOR is its own inverse:** `x ^ x = 0`, `x ^ 0 = x`, and XOR is commutative/associative — so XOR-ing a stream cancels every value that appears an even number of times.
- **Two's complement:** `-n == ~n + 1`. That's why `n & (-n)` isolates the lowest set bit.

---

### Q23: Single Number I (XOR)
**Company:** Amazon, Google, Microsoft, Meta, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Every element appears **twice** except one. Find that single element. Linear time, O(1) extra space.

**What interviewer is testing:**
Do you know XOR's pair-cancellation property? It's the gateway bit question — the expected answer is one line.

**Ideal Answer:**

XOR all elements. Pairs cancel (`x ^ x = 0`), leaving the lone element (`0 ^ unique = unique`). Order doesn't matter because XOR is commutative and associative.

```java
public int singleNumber(int[] nums) {
    int xor = 0;
    for (int x : nums) xor ^= x;   // duplicates cancel in pairs
    return xor;
}
```

**Dry run** — `[4,1,2,1,2]`: 0^4=4; 4^1=5; 5^2=7; 7^1=6; 6^2=4 → 4. ✅ (The 1s and 2s cancel; 4 remains.)

**Complexity:** O(n) time, O(1) space.

**Why not a hashset?** A set works (add/remove, what remains is the answer) but uses O(n) space — XOR wins the constant-space requirement.

**Follow-ups:**
1. *"Every element thrice except one?"* → Single Number II (Q24) — XOR no longer cancels triples.
2. *"Two unique elements (rest in pairs)?"* → Single Number III (Q25).
3. *"Prove XOR gives the answer."* → Commutativity/associativity let you regroup into `(a^a)^(b^b)^...^unique = unique`.

**Common mistakes:**
- Reaching for a hashmap (misses the O(1) space point).
- Initializing `xor` to `nums[0]` then double-counting it.

---

### Q24: Single Number II (every other thrice — 3-state bit counting)
**Company:** Amazon, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Every element appears **three times** except one (which appears once). Find it. Linear time, O(1) space.

**What interviewer is testing:**
Two solution levels: the intuitive bit-counting approach, then the slick two-variable state machine. This is a genuine "do you really understand bits" filter.

**Ideal Answer:**

*Approach 1 — count bits mod 3:* for each of the 32 bit positions, count how many numbers have that bit set. Since every number except the unique one appears 3 times, each bit's count is a multiple of 3 plus (0 or 1 from the unique number). So `count % 3` reconstructs the unique number's bits.

```java
public int singleNumberCount(int[] nums) {
    int result = 0;
    for (int b = 0; b < 32; b++) {          // each bit position
        int sum = 0;
        for (int x : nums) sum += (x >> b) & 1; // count set bits at position b
        if (sum % 3 != 0) result |= (1 << b);   // the unique number set this bit
    }
    return result;
}
```
O(32n) time, O(1) space. Clear and always correct (handles negatives via the sign bit naturally).

*Approach 2 — two-bit state machine (the elegant one):* track two masks, `ones` and `twos`, encoding how many times (mod 3) each bit has been seen: 00→01→10→00. After processing everything, `ones` holds the unique number.

```java
public int singleNumber(int[] nums) {
    int ones = 0, twos = 0;
    for (int x : nums) {
        // a bit enters 'ones' when seen the 1st time (and not in twos)
        ones = (ones ^ x) & ~twos;
        // a bit enters 'twos' when seen the 2nd time (and not in ones)
        twos = (twos ^ x) & ~ones;
    }
    return ones; // bits seen exactly once (mod 3)
}
```

**Why it works:** think of `(twos, ones)` as a 2-bit counter per bit position cycling 00→01→10→00 (i.e. count mod 3). On the third occurrence both reset to 0. The update order matters: `ones` is computed first, then `twos` uses the *new* `ones`. After all numbers, only the bits appearing once (mod 3) survive in `ones`.

**Dry run (count approach, reliable to trace)** — `[2,2,3,2]` (binary 2=`10`, 3=`11`): bit0 set only in {3} → count 1, `1%3=1` → set bit0; bit1 set in {2,2,3,2} → count 4, `4%3=1` → set bit1. Result = `11` = **3**. ✅ (Unique element is 3, which appears once; the three 2s cancel mod 3.)

**Complexity:** Both O(n) time; bit-count is O(32n), state machine is O(n) with tiny constant. O(1) space.

**Follow-ups:**
1. *"Generalize to k occurrences?"* → The bit-count mod-k approach generalizes directly; the state machine needs ⌈log₂k⌉ masks.
2. *"Lead with which?"* → Present the count approach first (always correct, easy to reason about), then offer the state machine as the optimization.

**Common mistakes:**
- Getting the state-machine update order wrong (must compute `ones` then `twos` using new `ones`).
- Bit-count approach: forgetting it naturally handles the sign bit at position 31 (so negatives work).

---

### Q25: Single Number III (two singletons — XOR + lowest set bit grouping)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Exactly **two** elements appear once; every other element appears twice. Return the two single numbers (any order). Linear time, O(1) space.

**What interviewer is testing:**
Extending XOR with a *partitioning* idea: XOR everything to get `a ^ b`, then use any set bit of that result to split all numbers into two groups, isolating each unique number.

**Ideal Answer:**

1. XOR all numbers → `xorAll = a ^ b` (the two uniques; pairs cancel).
2. `a != b` so `xorAll != 0`; pick its **lowest set bit** `diff = xorAll & (-xorAll)`. This bit differs between `a` and `b`.
3. Partition all numbers by that bit. `a` and `b` fall into different groups; every duplicate pair lands together in the same group. XOR each group separately → one yields `a`, the other `b`.

```java
public int[] singleNumber(int[] nums) {
    int xorAll = 0;
    for (int x : nums) xorAll ^= x;         // a ^ b
    int diff = xorAll & (-xorAll);          // lowest set bit (where a,b differ)
    int a = 0;
    for (int x : nums) {
        if ((x & diff) != 0) a ^= x;        // group with this bit set → isolates a
    }
    int b = xorAll ^ a;                      // recover b from a ^ b
    return new int[] { a, b };
}
```

**Dry run** — `[1,2,1,3,2,5]`: xorAll = 1^2^1^3^2^5 = 3^5 = 6 (binary `110`). diff = 6 & -6 = `010` (bit1). Group with bit1 set: {2,3,2} → 2^3^2 = 3 → a=3. b = 6 ^ 3 = 5. Result {3,5}. ✅

**Complexity:** O(n) time, O(1) space.

**Why `xorAll & (-xorAll)`?** In two's complement, `-x = ~x + 1`, which flips all bits above the lowest set bit and keeps it — so ANDing isolates exactly the lowest set bit. Any differing bit works; the lowest is the cleanest to extract.

**Follow-ups:**
1. *"Why does a set bit of `a^b` guarantee a and b differ there?"* → A 1 in `a^b` means the operands differ at that position by definition of XOR.
2. *"Three unique numbers?"* → No longer a clean two-group split; needs different techniques (and the problem rarely appears).

**Common mistakes:**
- Forgetting both uniques could share many bits — you only need *one* differing bit.
- Trying to find both with a single pass without the partition.

---

### Q26: Counting Bits (0..n)
**Company:** Amazon, Google, Meta, Apple
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given `n`, return an array `ans` of length `n+1` where `ans[i]` = number of set bits in `i`. Aim for O(n) total.

**What interviewer is testing:**
Spotting the DP recurrence between a number and a smaller number obtained by clearing a bit — turning 32 popcounts per number into O(1) per number.

**Ideal Answer:**

*Brute force:* call popcount on each `i` → O(n log n).

*DP — clear lowest set bit:* `i` has exactly one more set bit than `i & (i-1)` (which is `i` with its lowest set bit removed). Since `i & (i-1) < i`, it's already computed.

```java
public int[] countBits(int n) {
    int[] ans = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        ans[i] = ans[i & (i - 1)] + 1;  // one more bit than i with lowest bit cleared
    }
    return ans;
}
```

*Alternative DP — right shift:* `ans[i] = ans[i >> 1] + (i & 1)` (bits of i/2 plus the last bit). Equally O(n).

**Dry run** — n=5: ans[0]=0; ans[1]=ans[0]+1=1; ans[2]=ans[0]+1=1; ans[3]=ans[2]+1=2; ans[4]=ans[0]+1=1; ans[5]=ans[4]+1=2 → `[0,1,1,2,1,2]`. ✅

**Complexity:** O(n) time, O(n) output.

**Follow-ups:**
1. *"Both DP recurrences — which is better?"* → Both O(n); `i & (i-1)` is the classic, `i>>1` is arguably more intuitive.
2. *"Just popcount of a single number?"* → Brian Kernighan (Q27).

**Common mistakes:**
- O(n log n) by calling `Integer.bitCount` per element when O(n) DP is expected.
- Off-by-one on array size (`n+1`).

---

### Q27: Number of 1 Bits (Brian Kernighan)
**Company:** Amazon, Microsoft, Apple, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Return the number of set bits (Hamming weight) of an unsigned 32-bit integer.

**What interviewer is testing:**
Knowing Brian Kernighan's trick — `n & (n-1)` clears the lowest set bit — so the loop runs once per set bit, not 32 times.

**Ideal Answer:**

*Naive:* check all 32 bits with `(n >> i) & 1`. O(32).

*Brian Kernighan:* `n & (n - 1)` removes the lowest set bit. Loop until `n == 0`, counting iterations → runs exactly `popcount` times.

```java
public int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1);   // clear the lowest set bit
        count++;
    }
    return count;
}
```

**Why `n & (n-1)` clears the lowest set bit:** subtracting 1 flips the lowest set bit to 0 and all bits below it to 1; ANDing with the original zeroes out that lowest set bit and everything below stays 0. e.g. `12 (1100) - 1 = 11 (1011)`, `1100 & 1011 = 1000`.

**Dry run** — n=11 (`1011`): 1011&1010=1010 (cnt1); 1010&1001=1000 (cnt2); 1000&0111=0000 (cnt3) → 3. ✅

**Complexity:** O(popcount) time (≤32), O(1) space.

**Follow-ups:**
1. *"In Java, why handle as unsigned?"* → Java `int` is signed; the loop works regardless since we just clear bits. `Integer.bitCount(n)` is the library equivalent.
2. *"Hamming distance between two numbers?"* → `Integer.bitCount(a ^ b)`.

**Common mistakes:**
- Using `>>` (signed shift) on a negative number in a 32-iteration loop → infinite loop from sign extension; use `>>>` or the Kernighan loop (which avoids the issue).

---

### Q28: Power of Two
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return true if `n` is a power of two.

**What interviewer is testing:**
The single-set-bit characterization of powers of two and the `n & (n-1) == 0` trick.

**Ideal Answer:**

A power of two has exactly **one** set bit. `n & (n-1)` clears the lowest set bit; if the result is 0 *and* `n > 0`, there was exactly one bit → power of two.

```java
public boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;  // exactly one set bit
}
```

**Dry run** — n=16 (`10000`): 16&15 = `10000` & `01111` = 0 → true. n=6 (`110`): 6&5 = `110`&`101` = `100` ≠ 0 → false. ✅

**Complexity:** O(1) time and space.

**Follow-ups:**
1. *"Power of four?"* → Q29 (also need the single bit to be at an even position).
2. *"`n & (-n) == n` alternative?"* → Also isolates lowest set bit; equals `n` iff single bit. Equivalent.

**Common mistakes:**
- Forgetting the `n > 0` guard: `0 & -1 == 0` would falsely report 0 as a power of two; negatives also slip through.

---

### Q29: Power of Four
**Company:** Amazon, Microsoft, Two Sigma
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** Phone

**Question:**
Return true if `n` is a power of four.

**What interviewer is testing:**
Combining "single set bit" (power of two) with "that bit is at an **even** position" (powers of four are 1, 4, 16, 64 → bits 0,2,4,6…).

**Ideal Answer:**

Power of four ⟹ power of two (single bit) AND the bit sits at an even index. Mask `0x55555555` has 1s at all even positions (`0101...`); ANDing isolates whether the single bit is even-positioned.

```java
public boolean isPowerOfFour(int n) {
    return n > 0
        && (n & (n - 1)) == 0          // single set bit (power of two)
        && (n & 0x55555555) != 0;      // that bit is at an even position
}
```

**Dry run** — n=16 (`10000`, bit4 even): single bit ✓; 16 & 0x55555555 → bit4 set in the mask → nonzero → true. n=8 (`1000`, bit3 odd): single bit ✓; 8 & 0x55555555 = 0 → false. ✅

**Complexity:** O(1).

**Alternative:** `(n - 1) % 3 == 0` combined with power-of-two — because `4^k − 1` is always divisible by 3.

**Follow-ups:**
1. *"Why `0x55555555`?"* → It's `0101...0101`, ones at even bit positions (0,2,4,…), which is where powers of four land.
2. *"Power of three?"* → No clean bit trick; use the max-power-of-3-int divisibility trick or logarithms.

**Common mistakes:**
- Forgetting the even-position check → 8 (a power of two but not four) passes.

---

### Q30: Reverse Bits
**Company:** Amazon, Apple, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Reverse the bits of a 32-bit unsigned integer (bit 0 ↔ bit 31, etc.).

**What interviewer is testing:**
Bit-by-bit construction, and possibly the divide-and-conquer swap optimization.

**Ideal Answer:**

*Bit-by-bit:* pull the LSB of `n`, push it onto `result` shifted left; repeat 32 times.

```java
public int reverseBits(int n) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result = (result << 1) | (n & 1); // append n's LSB to result
        n >>>= 1;                          // unsigned shift to next bit
    }
    return result;
}
```

**Dry run** — reversing `00000010100101000001111010011100` yields `00111001011110000010100101000000` (LeetCode's example). Each iteration shifts `result` left and ORs in the current LSB of `n`. ✅

**Complexity:** O(1) (fixed 32 iterations), O(1) space.

*Divide-and-conquer (O(log 32) = 5 swaps):* swap halves, then quarters, then bytes, etc., using masks.

```java
public int reverseBitsFast(int n) {
    n = (n >>> 16) | (n << 16);
    n = ((n & 0xff00ff00) >>> 8) | ((n & 0x00ff00ff) << 8);
    n = ((n & 0xf0f0f0f0) >>> 4) | ((n & 0x0f0f0f0f) << 4);
    n = ((n & 0xcccccccc) >>> 2) | ((n & 0x33333333) << 2);
    n = ((n & 0xaaaaaaaa) >>> 1) | ((n & 0x55555555) << 1);
    return n;
}
```

**Follow-ups:**
1. *"Called many times — optimize?"* → Cache reversed bytes in a lookup table (256 entries), reverse 4 bytes per call.
2. *"Why `>>>` not `>>`?"* → Signed `>>` sign-extends, corrupting the high bits for negative `n`.

**Common mistakes:**
- Using `>>` instead of `>>>` → sign extension bug.

---

### Q31: Sum of Two Integers (no `+` operator)
**Company:** Amazon, Google, Apple, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Compute `a + b` without using `+` or `−`.

**What interviewer is testing:**
Understanding addition at the bit level: XOR is sum-without-carry; AND-then-shift is the carry. Iterate until no carry remains.

**Ideal Answer:**

- `a ^ b` adds bit-by-bit **ignoring carries**.
- `(a & b) << 1` is the **carry** (a 1 is carried where both bits are 1).
- Repeat: add the carry into the partial sum until carry becomes 0.

```java
public int getSum(int a, int b) {
    while (b != 0) {                 // b holds the carry
        int carry = (a & b) << 1;    // where both bits are 1 → carry shifts left
        a = a ^ b;                   // sum without carry
        b = carry;                   // fold carry in next iteration
    }
    return a;
}
```

**Dry run** — `a=2 (010)`, `b=3 (011)`:
- carry = (010&011)<<1 = (010)<<1 = 100; a = 010^011 = 001; b = 100
- carry = (001&100)<<1 = 0; a = 001^100 = 101; b = 0 → return 101 = 5. ✅

**Complexity:** O(1) (≤32 iterations), O(1) space.

**Why it terminates:** each iteration pushes the carry one bit higher; after at most 32 steps it shifts out, so `b` becomes 0. Java's defined two's-complement overflow makes negatives work automatically.

**Follow-ups:**
1. *"Subtraction without `−`?"* → `a - b = getSum(a, getSum(~b, 1))` (add the two's complement of b).
2. *"Multiplication without `*`?"* → Shift-and-add (Russian peasant).

**Common mistakes:**
- Recursing without a base case → stack issues; the iterative loop is cleaner.
- Forgetting that the carry must be shifted **left** by 1.

---

### Q32: Bitwise AND of Numbers Range
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given `[left, right]`, return the bitwise AND of all integers in that inclusive range.

**What interviewer is testing:**
The insight that the answer is the **common binary prefix** of `left` and `right` — any bit that changes within the range gets ANDed to 0.

**Ideal Answer:**

If any bit position flips somewhere in `[left, right]`, then both a 0 and a 1 occur there, so the AND of that bit is 0. Only the bits where `left` and `right` agree in their high prefix survive. So: strip differing low bits until `left == right`; that common prefix (padded with zeros) is the answer.

*Approach 1 — shift off differing bits:* right-shift both until equal, counting shifts, then shift back.

```java
public int rangeBitwiseAnd(int left, int right) {
    int shift = 0;
    while (left < right) {        // remove bits that differ
        left >>= 1;
        right >>= 1;
        shift++;
    }
    return left << shift;         // common prefix, low bits zeroed
}
```

*Approach 2 — Brian Kernighan:* repeatedly clear the lowest set bit of `right` until `right <= left`; the result is the common prefix.

```java
public int rangeBitwiseAndBK(int left, int right) {
    while (left < right) {
        right &= (right - 1);     // drop lowest set bit of right
    }
    return right;
}
```

**Dry run** — `[5,7]`: 5=`101`, 7=`111`. Approach1: 101/111 → 10/11 → 1/1 equal after 2 shifts; `1 << 2 = 100 = 4`. Check: 5&6&7 = `101`&`110`&`111` = `100` = 4. ✅

**Complexity:** O(log range) time, O(1) space.

**Follow-ups:**
1. *"Why is it the common prefix?"* → Within the range, the lowest differing bit takes both values → ANDs to 0; everything below it likewise.
2. *"Bitwise OR of the range?"* → Different problem; relates to filling bits up to the highest differing bit.

**Common mistakes:**
- Looping `left..right` and ANDing (TLE for large ranges).
- Off-by-one with the shift count.

---

### Q33: Subsets using a bitmask
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Generate all subsets of a set of `n` distinct elements using bit masks (iterative, no recursion).

**What interviewer is testing:**
The bijection between subsets and integers `0..2ⁿ−1`: bit `j` of the mask decides whether element `j` is included. A clean alternative to backtracking (Q1).

**Ideal Answer:**

There are exactly `2ⁿ` subsets. For each `mask` in `0..2ⁿ−1`, include `nums[j]` iff bit `j` of `mask` is set.

```java
public List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> res = new ArrayList<>();
    for (int mask = 0; mask < (1 << n); mask++) {   // 2^n masks
        List<Integer> subset = new ArrayList<>();
        for (int j = 0; j < n; j++) {
            if ((mask & (1 << j)) != 0) subset.add(nums[j]); // bit j set → include
        }
        res.add(subset);
    }
    return res;
}
```

**Dry run** — `nums = [1,2,3]`, masks 0..7:
- `000`→[], `001`→[1], `010`→[2], `011`→[1,2], `100`→[3], `101`→[1,3], `110`→[2,3], `111`→[1,2,3]. ✅ All 8.

**Complexity:** O(n · 2ⁿ) time (2ⁿ masks, n bits each), O(n·2ⁿ) output.

**Bonus — iterate submasks of a mask** (a competitive-programming staple, e.g. subset-sum DP over subsets):

```java
// enumerate every submask 'sub' of 'mask' (including 0 and mask itself)
for (int sub = mask; ; sub = (sub - 1) & mask) {
    // ... use sub ...
    if (sub == 0) break;
}
```
This runs in O(3ⁿ) total across all masks (each element is in/out/neither) — useful for SOS (sum-over-subsets) DP.

**Follow-ups:**
1. *"n > 31?"* → A 32-bit int can't hold the mask; use `long` (n ≤ 63) or a different representation.
2. *"Subsets with duplicates?"* → Bitmask generates duplicate *subsets* when input has equal values; dedup or use the backtracking skip (Q2).

**Common mistakes:**
- `1 << j` overflow for large n (use `1L << j`).
- Looping `mask <= (1<<n)` (off-by-one — should be `<`).

---

### Q34: Maximum XOR of Two Numbers in an Array
**Company:** Google, Amazon, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an array, find the maximum value of `nums[i] XOR nums[j]`.

**What interviewer is testing:**
Going from O(n²) brute force to O(n) using a **binary trie** (a greedy bit-by-bit approach). The trie construction itself is detailed in file 03; here the focus is the bit-greedy reasoning. A clean prefix-set alternative also exists.

**Ideal Answer:**

*Brute force:* all pairs → O(n²).

*Bit-greedy with a prefix HashSet (O(32n)):* build the answer from the most significant bit down. At each step, assume we *can* extend the current best prefix with a `1` in this bit (greedy — a higher bit beats all lower bits combined). Check feasibility: collect the high-bit prefixes of all numbers into a set; if some pair of prefixes XORs to the candidate, keep the `1`.

```java
public int findMaximumXOR(int[] nums) {
    int max = 0, mask = 0;
    for (int bit = 31; bit >= 0; bit--) {
        mask |= (1 << bit);                 // consider this bit and all above
        Set<Integer> prefixes = new HashSet<>();
        for (int x : nums) prefixes.add(x & mask); // high-bit prefix of each number
        int candidate = max | (1 << bit);   // greedily try to set this bit in the answer
        for (int p : prefixes) {
            // if two prefixes XOR to 'candidate', that bit is achievable:
            // because p ^ q == candidate  ⟺  q == p ^ candidate
            if (prefixes.contains(p ^ candidate)) {
                max = candidate;
                break;
            }
        }
    }
    return max;
}
```

**Why greedy works:** a `1` in bit `k` contributes `2^k`, which exceeds the sum of all lower bits (`2^k > 2^(k-1)+...+1`). So always prefer setting a higher bit if any pair allows it; commit to that bit before considering lower ones.

**Dry run** — `[3,10,5,25,2,8]`: the maximum XOR is `5 ^ 25 = 28`. The algorithm builds 28 bit-by-bit from the top, at each bit checking whether two prefixes can produce the candidate. ✅

**Complexity:** O(32n) time, O(n) space. The trie version (file 03) is the same asymptotically; this prefix-set version avoids explicit node objects.

**Trie alternative (mentioned, detailed in file 03):** insert each number's bits MSB→LSB into a binary trie; for each number, greedily walk the *opposite* bit at each level to maximize XOR. Also O(32n).

**Follow-ups:**
1. *"Max XOR with a query value / in a subarray range?"* → Persistent trie or offline processing.
2. *"Why MSB first?"* → Higher bits dominate the value; greedy on them is optimal.

**Common mistakes:**
- Trying all pairs (O(n²)) when O(n) is expected.
- Getting the XOR-feasibility identity wrong (`p ^ candidate` must also be a prefix).

---

### Q35: Minimum Flips to Make a OR b Equal to c
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given three integers `a`, `b`, `c`, return the minimum number of bit flips (in `a` or `b`) so that `a | b == c`. A flip toggles a single bit.

**What interviewer is testing:**
Per-bit case analysis: examine each bit of a, b, c independently and count required flips.

**Ideal Answer:**

For each bit position, look at `(abit, bbit, cbit)`:
- If `cbit == 1`: we need `abit | bbit == 1`. If both are 0, flip **one** of them → +1. Otherwise 0 flips.
- If `cbit == 0`: we need `abit | bbit == 0`, so **both** must be 0. Flip however many of `abit, bbit` are 1 → `abit + bbit` flips.

```java
public int minFlips(int a, int b, int c) {
    int flips = 0;
    for (int i = 0; i < 32; i++) {
        int abit = (a >> i) & 1;
        int bbit = (b >> i) & 1;
        int cbit = (c >> i) & 1;
        if (cbit == 1) {
            if (abit == 0 && bbit == 0) flips++;      // need at least one 1
        } else { // cbit == 0
            flips += abit + bbit;                     // both must become 0
        }
    }
    return flips;
}
```

**Bit-trick version** (conceptual): bits where `c==0` but `a|b==1` each need clearing — `Integer.bitCount((a | b) & ~c)`, plus bits where `c==1` but `a|b==0` — `Integer.bitCount(~(a | b) & c)`.

**Dry run** — `a=2 (010)`, `b=6 (110)`, `c=5 (101)`:
- bit0: a0,b0,c1 → both 0, c=1 → +1
- bit1: a1,b1,c0 → c=0, both 1 → +2
- bit2: a0,b1,c1 → c=1, has a 1 → 0
- Total 3. ✅

**Complexity:** O(1) (32 bits), O(1) space.

**Follow-ups:**
1. *"Why does cbit==0 cost both bits?"* → OR is 0 only if every operand bit is 0; each set bit must be individually flipped.
2. *"Can a single flip fix two bits?"* → No — a flip toggles one bit in one number.

**Common mistakes:**
- When `cbit==1`, flipping both a and b (only one is needed).
- When `cbit==0`, flipping only one of two set bits.

---

### Q36: Total Hamming Distance
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Hamming distance between two numbers is the count of differing bit positions. Given an array, return the **sum** of Hamming distances between **all pairs**.

**What interviewer is testing:**
The combinatorial reframing: instead of O(n²) pairwise comparisons, count per bit position how many numbers have it set vs. unset, and multiply.

**Ideal Answer:**

For each bit position independently: if `k` of the `n` numbers have that bit set, then `(n − k)` have it unset. Every set/unset pairing contributes 1 to the total distance → `k * (n − k)` pairs differ at this bit. Sum over all 32 bits.

```java
public int totalHammingDistance(int[] nums) {
    int n = nums.length;
    long total = 0;                                  // long: k*(n-k) over 32 bits can be large
    for (int bit = 0; bit < 32; bit++) {
        int ones = 0;
        for (int x : nums) ones += (x >> bit) & 1;   // count set bits at this position
        total += (long) ones * (n - ones);           // set×unset pairs differ here
    }
    return (int) total;
}
```

**Dry run** — `[4,14,2]` (`100`, `1110`, `010`):
- bit1: set in {2=`010` yes, 14=`1110` yes, 4=`100` no} → ones=2, n−ones=1 → 2·1=2
- bit2: set in {4=`100` yes, 14=`1110` yes, 2 no} → ones=2 → 2·1=2
- bit3: set in {14 only} → ones=1 → 1·2=2
- Total = 6 (LeetCode answer). ✅

**Complexity:** O(32n) time, O(1) space. Brute force is O(n²·32).

**Follow-ups:**
1. *"Why count per bit instead of per pair?"* → Bits are independent; per-bit counting collapses O(n²) pairs into O(n) per bit.
2. *"Single pair Hamming distance?"* → `Integer.bitCount(a ^ b)`.

**Common mistakes:**
- O(n²) all-pairs loop (TLE).
- Overflow on large `n` (use `long` for the running total — shown above).

---

### Q37: Gray Code
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Return an `n`-bit Gray code sequence: a permutation of `0..2ⁿ−1` where consecutive numbers (and the first/last) differ by exactly **one** bit.

**What interviewer is testing:**
Knowing the closed-form Gray code formula `g = i ^ (i >> 1)`, or the reflect-and-prefix construction.

**Ideal Answer:**

*Formula approach:* the `i`-th Gray code is `i ^ (i >> 1)`. Iterate `i` from `0` to `2ⁿ−1`.

```java
public List<Integer> grayCode(int n) {
    List<Integer> res = new ArrayList<>();
    for (int i = 0; i < (1 << n); i++) {
        res.add(i ^ (i >> 1));      // binary-to-Gray conversion
    }
    return res;
}
```

*Reflect-and-prefix (constructive intuition):* start with `[0,1]`. To extend from `k` bits to `k+1`: take the current list, mirror it, and prefix the second half with a leading `1` (i.e. add `1 << k`). Adjacent entries still differ by one bit, and the mirror seam differs only in the new high bit.

```java
public List<Integer> grayCodeReflect(int n) {
    List<Integer> res = new ArrayList<>();
    res.add(0);
    for (int k = 0; k < n; k++) {
        int high = 1 << k;
        for (int i = res.size() - 1; i >= 0; i--) { // mirror, prefix with high bit
            res.add(res.get(i) | high);
        }
    }
    return res;
}
```

**Dry run (formula)** — n=2: i=0→0^0=0; i=1→1^0=1; i=2→2^1=3; i=3→3^1=2 → `[0,1,3,2]` = `00,01,11,10`. Each consecutive pair differs by one bit, and `10→00` (last to first) differs by one. ✅

**Complexity:** O(2ⁿ) time, O(2ⁿ) output.

**Follow-ups:**
1. *"Why does `i ^ (i>>1)` give Gray code?"* → It's the standard binary→Gray transform; consecutive integers `i, i+1` map to codes differing in exactly one bit (provable by the carry structure).
2. *"Gray → binary back?"* → XOR-prefix: `b = g; while (g >>= 1) b ^= g;`.

**Common mistakes:**
- Producing a valid one-bit-difference sequence but forgetting the wrap-around (first and last must also differ by one) — the formula handles it automatically.

---

### Q38: Essential bit tricks every engineer must know (reference)
**Company:** Universal — these appear *inside* dozens of other problems
**Difficulty:** 🟢 Easy (to recall) / 🔴 Hard (to derive cold)
**Frequency:** 🔥🔥🔥
**Round:** All

**Question:**
A reference of the bit manipulations you should be able to write instantly. Interviewers expect these as *building blocks*, not as standalone problems.

**What interviewer is testing:**
Fluency. If you pause to derive "how do I clear the nth bit" mid-problem, it signals you don't work with bits often.

**Ideal Answer — the canonical toolbox:**

```java
// ── Single-bit operations (0-indexed from the LSB) ──
int  getBit   (int x, int n) { return (x >> n) & 1; }       // read nth bit (0 or 1)
int  setBit   (int x, int n) { return x | (1 << n); }       // force nth bit to 1
int  clearBit (int x, int n) { return x & ~(1 << n); }      // force nth bit to 0
int  toggleBit(int x, int n) { return x ^ (1 << n); }       // flip nth bit
// update nth bit to value v (0/1) branchlessly:
int  updateBit(int x, int n, int v) { return (x & ~(1 << n)) | (v << n); }

// ── Lowest set bit ──
int lowestSetBit     (int x) { return x & (-x); }    // isolate lowest 1-bit (1100 → 0100)
int clearLowestSetBit(int x) { return x & (x - 1); } // remove lowest 1-bit (Kernighan)
// (-x == ~x + 1 in two's complement, which is why x & -x isolates the lowest bit)

// ── Lowest set bit's index ──
int lowestSetBitIndex(int x) { return Integer.numberOfTrailingZeros(x); }

// ── Powers of two ──
boolean isPowerOfTwo(int x) { return x > 0 && (x & (x - 1)) == 0; }
int     nextPowerOfTwo(int x) { return Integer.highestOneBit(x - 1) << 1; }

// ── Counting / scanning ──
int popcount(int x) { return Integer.bitCount(x); }   // number of set bits
// manual Kernighan popcount:
int popcountManual(int x) { int c = 0; while (x != 0) { x &= x - 1; c++; } return c; }
int leadingZeros (int x) { return Integer.numberOfLeadingZeros(x); }
int trailingZeros(int x) { return Integer.numberOfTrailingZeros(x); }

// ── Whole-number tricks ──
int  absNoBranch(int x) { int m = x >> 31; return (x + m) ^ m; }   // abs via sign mask
boolean oppositeSigns(int a, int b) { return (a ^ b) < 0; }        // sign bit differs
// XOR swap (illustrative only — fails if a and b alias the same location):
// a ^= b; b ^= a; a ^= b;
int  multiplyBy2 (int x) { return x << 1; }   // shift = ×2
int  divideBy2   (int x) { return x >> 1; }   // arithmetic shift = ÷2 (floor for ≥0)
boolean isEven(int x) { return (x & 1) == 0; } // LSB test
```

**Mask cookbook:**

```java
// lowest k bits set:           (1 << k) - 1
// clear lowest k bits:         x & ~((1 << k) - 1)
// extract bits [i, j):         (x >> i) & ((1 << (j - i)) - 1)
// turn off all bits below i:   x & (~0 << i)
```

**The "why" behind the three most important ones:**

1. **`x & (x-1)` clears the lowest set bit.** Subtracting 1 flips the lowest 1 to 0 and the trailing 0s to 1s; the AND zeroes that bit and leaves higher bits intact. Foundation of Kernighan popcount, power-of-two test, Counting Bits DP.

2. **`x & (-x)` isolates the lowest set bit.** `-x = ~x + 1` inverts everything above the lowest set bit and restores it, so the AND keeps only that bit. Foundation of Fenwick/BIT trees and Single Number III.

3. **XOR cancels pairs.** `x ^ x = 0`, `x ^ 0 = x`. Foundation of all the Single Number problems, Missing Number, and swap-without-temp.

**Subset / mask iteration patterns** (competitive programming):

```java
// iterate all 2^n subsets of an n-element set
for (int mask = 0; mask < (1 << n); mask++) { /* ... */ }

// iterate all submasks of 'mask' (descending), O(3^n) total over all masks
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) { /* ... */ }

// check if element i is in mask: (mask & (1 << i)) != 0
// add element i:                 mask |  (1 << i)
// remove element i:              mask & ~(1 << i)
// size of the set:               Integer.bitCount(mask)
```

**Common mistakes (the ones that bite even seniors):**
- `>>` vs `>>>` — signed vs unsigned shift; sign extension corrupts high bits and causes infinite loops.
- `1 << 31` overflows into a negative `int`; use `1L << 31` for bit 31+ or when masking 64-bit.
- Operator precedence: `&`, `|`, `^` bind *looser* than `==`/`+`. Always parenthesize: `(x & 1) == 0`, not `x & 1 == 0` (the latter parses as `x & (1 == 0)` → a compile error in Java, but the habit matters in C/C++).
- XOR swap fails when both refer to the same memory location (sets it to 0) — use only when distinct.

**Follow-up the interviewer may ask:** *"Implement popcount four different ways."* → Kernighan loop, lookup table per byte, the SWAR parallel-bit-count (`0x55/0x33/0x0f` masks), and `Integer.bitCount`. Knowing all four is a strong signal.

---

# MATH & NUMBER THEORY

These problems look elementary and reject more candidates than the hard graph problems — because the failure modes are *overflow*, *edge cases* (0, negative, MIN_VALUE), and *one identity you didn't memorize*. The recurring tools: the Euclidean algorithm, the Sieve, fast exponentiation by squaring, and modular arithmetic. Know them cold and you'll convert "looks easy" into "actually easy."

A standing warning that applies to almost every problem below: **think about overflow before you write the loop.** `Integer.MIN_VALUE` has no positive counterpart; `n!` overflows fast; `a * b` can overflow even when `a` and `b` fit. Reach for `long`, or do the math modularly, the moment products or sums can grow.

---

### Q39: GCD / LCM (Euclidean algorithm)
**Company:** Amazon, Google, Microsoft, Goldman Sachs
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Compute the greatest common divisor of two integers, and the least common multiple.

**What interviewer is testing:**
The Euclidean algorithm (`gcd(a,b) = gcd(b, a mod b)`) and the LCM identity (`lcm = a / gcd * b`, dividing *before* multiplying to avoid overflow).

**Ideal Answer:**

GCD via Euclid: repeatedly replace `(a, b)` with `(b, a mod b)` until `b == 0`; the answer is `a`. Each step shrinks the numbers quickly (proven O(log min) by the worst case being consecutive Fibonacci numbers).

```java
public int gcd(int a, int b) {
    while (b != 0) {
        int t = a % b;
        a = b;
        b = t;
    }
    return a;                 // gcd; a >= 0 if inputs non-negative
}

// recursive form (just as common in interviews):
public int gcdRec(int a, int b) {
    return b == 0 ? a : gcdRec(b, a % b);
}

public long lcm(int a, int b) {
    if (a == 0 || b == 0) return 0;
    return (long) a / gcd(a, b) * b;   // divide first → avoids overflow
}
```

**Dry run** — gcd(48, 18): (48,18)→(18, 48%18=12)→(12, 18%12=6)→(6, 12%6=0) → 6. lcm = 48/6*18 = 8*18 = 144. ✅

**Complexity:** O(log min(a,b)) time, O(1) space (iterative).

**Why divide before multiply for LCM?** `a * b` may overflow `int` even when `lcm` fits; `a / gcd` is exact (gcd divides a), so `(a/gcd) * b` stays smaller and is still correct.

**Follow-ups:**
1. *"GCD of an array?"* → Fold: `g = gcd(g, arr[i])`; a single 1 short-circuits to 1.
2. *"Extended Euclidean (find x, y with ax + by = gcd)?"* → Needed for modular inverse (Q51).
3. *"Binary GCD (Stein's algorithm)?"* → Uses shifts and subtraction; avoids division, faster on some hardware.

**Common mistakes:**
- `a * b` overflow in LCM.
- Negative inputs — take absolute values if the domain allows negatives (`gcd` of negatives is usually defined as positive).

---

### Q40: Sieve of Eratosthenes / Count Primes
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Count the number of primes strictly less than a non-negative integer `n`.

**What interviewer is testing:**
The Sieve, and two key optimizations: start crossing multiples from `i*i`, and only sieve `i` up to `√n`.

**Ideal Answer:**

Mark composites by crossing off multiples of each prime. Optimizations:
- Begin crossing from `i*i` (smaller multiples like `2*i, 3*i` were already crossed by smaller primes).
- Only iterate `i` while `i*i < n` — beyond `√n`, every remaining unmarked number is prime.

```java
public int countPrimes(int n) {
    if (n < 3) return 0;                     // no primes below 2
    boolean[] composite = new boolean[n];    // composite[k] == true → k is not prime
    int count = 0;
    for (int i = 2; (long) i * i < n; i++) {
        if (!composite[i]) {                 // i is prime
            for (int j = i * i; j < n; j += i) { // cross off i*i, i*i+i, ...
                composite[j] = true;
            }
        }
    }
    for (int i = 2; i < n; i++) {
        if (!composite[i]) count++;          // count primes < n
    }
    return count;
}
```

**Dry run** — n=10: sieve i=2 (cross 4,6,8), i=3 (cross 9; `3*3=9<10`); i=4 stops since `4*4=16≥10`. Unmarked in [2,9]: 2,3,5,7 → count 4. ✅

**Complexity:** Time **O(n log log n)** (the classic Sieve bound), Space O(n).

**Follow-ups:**
1. *"Why start at `i*i`?"* → Multiples below `i*i` have a smaller prime factor and were already crossed.
2. *"List the primes, not just count?"* → Collect unmarked indices.
3. *"Segmented sieve for huge n?"* → Sieve in `√n`-sized blocks to bound memory.
4. *"Linear sieve?"* → O(n) variant that marks each composite exactly once via its smallest prime factor.

**Common mistakes:**
- `i * i` overflow → use `(long) i * i` in the loop condition.
- Off-by-one (problem asks primes **strictly less** than n).
- Starting the inner loop at `2*i` (correct but slower).

---

### Q41: Pow(x, n) — fast exponentiation
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Implement `pow(x, n)` (x is a double, n is an int, possibly negative) in O(log n).

**What interviewer is testing:**
Exponentiation by squaring, and the **`Integer.MIN_VALUE` overflow trap** when negating `n`.

**Ideal Answer:**

`x^n` via binary exponentiation: `x^n = (x^(n/2))^2`, times an extra `x` if `n` is odd. For negative `n`, compute `1 / x^(-n)` — but `-n` overflows when `n == Integer.MIN_VALUE`, so convert to `long` first.

```java
public double myPow(double x, int n) {
    long N = n;                  // widen to long BEFORE negating (MIN_VALUE trap)
    if (N < 0) {
        x = 1 / x;
        N = -N;                  // safe now: long can hold -Integer.MIN_VALUE
    }
    double result = 1.0;
    while (N > 0) {
        if ((N & 1) == 1) result *= x;  // odd exponent → multiply in current x
        x *= x;                          // square the base
        N >>= 1;                         // halve the exponent
    }
    return result;
}
```

**Recursive form** (also common):

```java
public double myPowRec(double x, int n) {
    long N = n;
    if (N < 0) { x = 1 / x; N = -N; }
    return fastPow(x, N);
}
private double fastPow(double x, long n) {
    if (n == 0) return 1.0;
    double half = fastPow(x, n / 2);
    return (n % 2 == 0) ? half * half : half * half * x;
}
```

**Dry run** — `myPow(2, 10)`: N=10(`1010`). bit0=0: x=4,N=5; bit0=1: result=4,x=16,N=2; bit0=0: x=256,N=1; bit0=1: result=4*256=1024,x=…,N=0 → 1024. ✅

**Complexity:** O(log n) time (exponent halves each step), O(1) iterative / O(log n) recursion stack.

**Follow-ups:**
1. *"Why widen to long?"* → `-Integer.MIN_VALUE` overflows back to `Integer.MIN_VALUE` (negative); `long N = n; N = -N;` is safe.
2. *"Modular exponentiation `(x^n) mod m`?"* → Same loop, take `% m` after every multiply (Q51).
3. *"Matrix power (e.g. Fibonacci in O(log n))?"* → Replace scalar multiply with matrix multiply.

**Common mistakes:**
- `n = -n` overflow at `Integer.MIN_VALUE`.
- Naive O(n) loop multiplying x n times (TLE for large n).
- Forgetting `x = 1/x` for negative exponents.

---

### Q42: Sqrt(x) — binary search + Newton's method
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Compute and return the integer square root of a non-negative `x` (floor of the true square root). No built-in `sqrt`.

**What interviewer is testing:**
Binary search on the answer (with overflow-safe comparison), and bonus points for Newton's method.

**Ideal Answer:**

*Binary search:* the floor sqrt lies in `[0, x]`. Search for the largest `m` with `m*m <= x`. Use `m <= x / m` (or `long`) to avoid `m*m` overflow.

```java
public int mySqrt(int x) {
    if (x < 2) return x;                 // 0→0, 1→1
    int lo = 1, hi = x / 2, ans = 1;     // sqrt(x) <= x/2 for x >= 2
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if ((long) mid * mid <= x) {     // long avoids overflow
            ans = mid;                   // candidate; try larger
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

*Newton's method (faster convergence):* iterate `r = (r + x/r) / 2` until it stops decreasing. Converges quadratically.

```java
public int mySqrtNewton(int x) {
    if (x < 2) return x;
    long r = x;
    while (r * r > x) {
        r = (r + x / r) / 2;             // Newton step toward the root
    }
    return (int) r;
}
```

**Dry run (binary search)** — x=8: lo=1,hi=4. mid=2 (4≤8) ans=2,lo=3; mid=3 (9>8) hi=2; loop ends → 2 (since `2²=4 ≤ 8 < 9=3²`). ✅

**Complexity:** Binary search O(log x); Newton O(log log x) iterations. O(1) space.

**Follow-ups:**
1. *"Float precision sqrt?"* → Newton with a tolerance `while (abs(r*r - x) > eps)`.
2. *"Why `(long) mid * mid`?"* → `mid*mid` overflows int for large x (e.g. x near 2³¹).
3. *"Valid Perfect Square?"* → Same search; return whether `m*m == x` exactly.

**Common mistakes:**
- `mid * mid` int overflow.
- Returning `lo` vs `hi` incorrectly (track `ans` explicitly to avoid the off-by-one).

---

### Q43: Happy Number (Floyd's cycle)
**Company:** Amazon, Google, Microsoft, Uber
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
A number is "happy" if repeatedly replacing it with the sum of the squares of its digits eventually reaches 1. Unhappy numbers loop forever. Return whether `n` is happy.

**What interviewer is testing:**
Cycle detection. Either a HashSet of seen values, or Floyd's tortoise-and-hare for O(1) space.

**Ideal Answer:**

The process either reaches 1 (happy) or enters a cycle (unhappy). Detect the cycle.

*HashSet:* record each value; if you revisit one before hitting 1, it's unhappy.

```java
public boolean isHappy(int n) {
    Set<Integer> seen = new HashSet<>();
    while (n != 1 && !seen.contains(n)) {
        seen.add(n);
        n = squareDigitSum(n);
    }
    return n == 1;
}
private int squareDigitSum(int n) {
    int sum = 0;
    while (n > 0) {
        int d = n % 10;
        sum += d * d;
        n /= 10;
    }
    return sum;
}
```

*Floyd's cycle (O(1) space):* run a slow pointer (one step) and a fast pointer (two steps); they meet inside a cycle, or fast reaches 1.

```java
public boolean isHappyFloyd(int n) {
    int slow = n, fast = squareDigitSum(n);
    while (fast != 1 && slow != fast) {
        slow = squareDigitSum(slow);
        fast = squareDigitSum(squareDigitSum(fast));
    }
    return fast == 1;
}
```

**Dry run** — n=19: 1²+9²=82 → 8²+2²=68 → 6²+8²=100 → 1²+0²+0²=1 → happy → true. n=2: 4→16→37→58→89→145→42→20→4 (cycle back to 4) → unhappy. ✅

**Complexity:** O(log n) per step (digits), and the values stay bounded (any number ≤ 3 digits maps below 243), so O(1) amortized cycle length. Floyd uses O(1) space; HashSet O(log n)-ish.

**Follow-ups:**
1. *"Why does it always terminate (no infinite growth)?"* → For numbers with d digits, the square-digit-sum is at most `81d`, which is far smaller than the number once it exceeds 3 digits — so values are bounded and must cycle or hit 1.
2. *"Floyd vs HashSet trade-off?"* → Floyd is O(1) space; HashSet is simpler to reason about.

**Common mistakes:**
- Assuming an unhappy number grows unboundedly (it doesn't — it cycles).
- Forgetting the `n != 1` exit before re-checking the set.

---

### Q44: Factorial Trailing Zeroes (count 5s)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given `n`, return the number of trailing zeroes in `n!`.

**What interviewer is testing:**
The number-theory insight: trailing zeros come from factors of 10 = 2×5, and 5s are rarer than 2s, so the answer is the count of factor-5s in `n!` — computed via Legendre's formula.

**Ideal Answer:**

Each trailing zero needs a pair (2, 5). 2s are abundant; the bottleneck is 5s. Count factors of 5 in `1..n`: `⌊n/5⌋` numbers contribute one 5, `⌊n/25⌋` contribute an *extra* 5, `⌊n/125⌋` another, etc. (Legendre's formula). Sum `⌊n/5^k⌋`.

```java
public int trailingZeroes(int n) {
    int count = 0;
    for (long pow = 5; pow <= n; pow *= 5) {  // 5, 25, 125, ... (long avoids overflow)
        count += n / pow;
    }
    return count;
}
```

**Dry run** — n=100: ⌊100/5⌋=20, ⌊100/25⌋=4, ⌊100/125⌋=0 → 20+4 = 24 trailing zeros. ✅

**Complexity:** O(log₅ n) time, O(1) space.

**Why count 5s not 2s?** In `n!`, factors of 2 vastly outnumber factors of 5 (every even number gives a 2), so the number of 10s = min(count2, count5) = count5.

**Follow-ups:**
1. *"Why divide by 25, 125 too?"* → 25 = 5², contributes *two* 5s; the `n/25` term counts the second one, `n/125` the third, etc.
2. *"Trailing zeros in base b?"* → Factor b into primes; for each prime power, count its occurrences and take the min over the limiting prime.
3. *"`pow` overflow?"* → Use `long` (shown) since `5^k` can exceed int.

**Common mistakes:**
- Counting only `n/5` (misses higher powers of 5 like 25, 125).
- `int` overflow on `pow`.

---

### Q45: Excel Sheet Column Number ↔ Title (base-26)
**Company:** Amazon, Microsoft, Google, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
(a) Convert an Excel column title (`"A"`, `"AB"`, `"ZY"`…) to its column number. (b) Convert a column number back to its title.

**What interviewer is testing:**
**Bijective base-26** (1-indexed, no zero digit) — the off-by-one that trips people: `A=1`, …, `Z=26`, `AA=27`. It's *not* plain base-26.

**Ideal Answer:**

*Title → Number:* standard positional evaluation, treating `A..Z` as `1..26`: `result = result * 26 + (c - 'A' + 1)`.

```java
public int titleToNumber(String s) {
    int result = 0;
    for (int i = 0; i < s.length(); i++) {
        result = result * 26 + (s.charAt(i) - 'A' + 1); // 1-indexed digit
    }
    return result;
}
```

*Number → Title:* repeatedly take `(n-1) % 26` for the digit and `n = (n-1) / 26`. The `−1` converts the 1-indexed system to a 0-indexed remainder.

```java
public String convertToTitle(int n) {
    StringBuilder sb = new StringBuilder();
    while (n > 0) {
        n--;                              // shift to 0-indexed for the modulo
        sb.append((char) ('A' + n % 26)); // current least-significant letter
        n /= 26;
    }
    return sb.reverse().toString();
}
```

**Dry run** — titleToNumber("AB"): 0*26+1=1 (A); 1*26+2=28 (B) → 28. convertToTitle(28): n=27 → 'A'+27%26='A'+1='B', n=27/26=1; n=0 → 'A'+0='A', n=0 → "BA" reversed = "AB". ✅

**Complexity:** O(len) time, O(1)/O(len) space.

**Follow-ups:**
1. *"Why `n--` before modulo?"* → There's no "0" digit; `Z` is 26 (not 0), so we map to 0-indexed by subtracting 1 each step.
2. *"Overflow?"* → For very long titles use `long`; standard Excel columns fit in `int`.

**Common mistakes:**
- Treating it as plain base-26 (forgetting the 1-indexing / the `n--`).
- Forgetting to reverse the built string.

---

### Q46: Roman to Integer / Integer to Roman
**Company:** Amazon, Microsoft, Google, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
(a) Convert a Roman numeral string to an integer. (b) Convert an integer (1–3999) to a Roman numeral.

**What interviewer is testing:**
Handling the subtractive notation (`IV=4`, `IX=9`, `CM=900`). For the reverse, a greedy table-driven approach.

**Ideal Answer:**

*Roman → Integer:* scan left to right; if a symbol's value is **less than** the next symbol's, it's subtractive (subtract it); otherwise add it.

```java
public int romanToInt(String s) {
    Map<Character, Integer> val = Map.of(
        'I',1,'V',5,'X',10,'L',50,'C',100,'D',500,'M',1000);
    int total = 0;
    for (int i = 0; i < s.length(); i++) {
        int cur = val.get(s.charAt(i));
        // if a smaller value precedes a larger one → subtractive (e.g. IV, IX)
        if (i + 1 < s.length() && cur < val.get(s.charAt(i + 1))) {
            total -= cur;
        } else {
            total += cur;
        }
    }
    return total;
}
```

*Integer → Roman:* greedy with a value→symbol table that **includes** the subtractive forms (900=CM, 400=CD, 90=XC, 40=XL, 9=IX, 4=IV). Subtract the largest fitting value repeatedly.

```java
public String intToRoman(int num) {
    int[] values =   {1000,900,500,400,100,90,50,40,10,9,5,4,1};
    String[] symbols = {"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < values.length && num > 0; i++) {
        while (num >= values[i]) {        // greedily take as many as fit
            sb.append(symbols[i]);
            num -= values[i];
        }
    }
    return sb.toString();
}
```

**Dry run** — romanToInt("MCMXCIV"): M(1000)+; C(100)<M? next is M(1000)>100 → −100; M(1000)+; X(10)<C(100) → −10; C(100)+; I(1)<V(5) → −1; V(5)+ → 1000−100+1000−10+100−1+5 = 1994. ✅
intToRoman(1994): 1000→"M"(994); 900→"CM"(94); 90→"XC"(4); 4→"IV"(0) → "MCMXCIV". ✅

**Complexity:** Both O(1) effectively (input bounded by 3999 / 15 symbols), O(1) space.

**Follow-ups:**
1. *"Why include 900/400/etc. in the table?"* → Greedy then handles subtractive cases automatically without special logic.
2. *"Validate a Roman numeral?"* → Check ordering rules and that no symbol repeats more than allowed.

**Common mistakes:**
- Roman→Int: hardcoding only the standard subtractive pairs instead of the elegant "less-than-next" rule.
- Int→Roman: omitting the subtractive entries from the table → produces `IIII` instead of `IV`.

---

### Q47: Multiply Strings (big-number multiplication)
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given two non-negative integers as strings `num1` and `num2`, return their product as a string. Do not use built-in BigInteger or directly convert to int (numbers can be huge).

**What interviewer is testing:**
Grade-school multiplication implemented on digit arrays, with the crucial **index identity**: digit `i` of num1 times digit `j` of num2 contributes to result positions `i+j` and `i+j+1`.

**Ideal Answer:**

The product of an `m`-digit and `n`-digit number has at most `m+n` digits. Multiply each digit pair, placing the product into a result array where `num1[i] * num2[j]` lands at positions `[i+j, i+j+1]` (carry into `i+j`). Then strip leading zeros.

```java
public String multiply(String num1, String num2) {
    if (num1.equals("0") || num2.equals("0")) return "0";
    int m = num1.length(), n = num2.length();
    int[] result = new int[m + n];               // max m+n digits
    for (int i = m - 1; i >= 0; i--) {
        int d1 = num1.charAt(i) - '0';
        for (int j = n - 1; j >= 0; j--) {
            int d2 = num2.charAt(j) - '0';
            int sum = d1 * d2 + result[i + j + 1]; // add to the low position
            result[i + j + 1] = sum % 10;          // keep the ones digit
            result[i + j] += sum / 10;             // carry into the higher position
        }
    }
    StringBuilder sb = new StringBuilder();
    for (int d : result) {
        if (!(sb.length() == 0 && d == 0)) sb.append(d); // skip leading zeros
    }
    return sb.length() == 0 ? "0" : sb.toString();
}
```

**Why positions `i+j` and `i+j+1`?** Digit `i` (from the right, counting from index m-1) has place value `10^(m-1-i)`. Their product's place value is `10^((m-1-i)+(n-1-j))`, which in the length-`(m+n)` result array (indexed left-to-right) maps the units of the product to index `i+j+1` and the carry to `i+j`.

**Dry run** — `"12" * "34"`: result size 4. i=1('2'),j=1('4'): 2*4+0=8 → result[3]=8. i=1,j=0('3'): 2*3+result[2]=6 → result[2]=6. i=0('1'),j=1: 1*4+result[2]=4+6=10 → result[2]=0, result[1]+=1. i=0,j=0: 1*3+result[1]=3+1=4 → result[1]=4. result=`[0,4,0,8]` → strip leading 0 → "408". ✅

**Complexity:** O(m·n) time, O(m+n) space.

**Follow-ups:**
1. *"Add two number strings?"* → Single pass, digit-wise with carry — a simpler cousin.
2. *"Karatsuba for very large numbers?"* → O(n^1.585) divide-and-conquer; mention it, but grade-school is expected here.
3. *"Negative numbers / decimals?"* → Track sign separately; handle the decimal point by counting fractional digits.

**Common mistakes:**
- Wrong index mapping (`i+j` vs `i+j+1`).
- Forgetting the carry propagation into `i+j`.
- Not stripping leading zeros (or returning `""` for product 0).

---

### Q48: Fraction to Recurring Decimal (long division)
**Company:** Google, Amazon, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given numerator and denominator, return the fraction as a string. If the fractional part repeats, enclose the repeating part in parentheses. e.g. `1/2 = "0.5"`, `2/3 = "0.(6)"`, `4/333 = "0.(012)"`.

**What interviewer is testing:**
Simulating long division and detecting the repeating cycle via a **remainder → position** hashmap. Plus sign handling and the overflow trap (`Integer.MIN_VALUE`).

**Ideal Answer:**

Do long division. The integer part is `num/den`. For the fractional part, repeatedly multiply the remainder by 10 and divide. **A repeating decimal begins when a remainder repeats** — so map each remainder to the position in the output where it first appeared; when it recurs, wrap from that position with parentheses.

Use `long` throughout (negating `Integer.MIN_VALUE` overflows), and compute the sign first.

```java
public String fractionToDecimal(int numerator, int denominator) {
    if (numerator == 0) return "0";
    StringBuilder sb = new StringBuilder();
    // sign: negative if exactly one operand is negative
    if ((numerator < 0) ^ (denominator < 0)) sb.append('-');
    long num = Math.abs((long) numerator);     // widen to long → MIN_VALUE safe
    long den = Math.abs((long) denominator);
    sb.append(num / den);                       // integer part
    long rem = num % den;
    if (rem == 0) return sb.toString();         // exact division
    sb.append('.');
    Map<Long, Integer> seen = new HashMap<>();  // remainder -> index in sb
    while (rem != 0) {
        if (seen.containsKey(rem)) {            // cycle detected
            int idx = seen.get(rem);
            sb.insert(idx, '(');
            sb.append(')');
            break;
        }
        seen.put(rem, sb.length());             // record where this remainder's digit starts
        rem *= 10;
        sb.append(rem / den);                   // next decimal digit
        rem %= den;
    }
    return sb.toString();
}
```

**Dry run** — `4/333`: int part 0, rem 4. `.`; rem4 not seen, record idx2; 40/333=0, rem40; rem40 record idx3; 400/333=1, rem67; rem67 record idx4; 670/333=2, rem4; **rem4 seen at idx2** → insert '(' at 2, append ')' → "0.(012)". ✅

**Complexity:** O(den) worst case (at most `den` distinct remainders before repeat), O(den) space for the map.

**Follow-ups:**
1. *"Why does a repeating remainder mean a repeating decimal?"* → The next digit is fully determined by the current remainder; once a remainder recurs, the entire digit sequence repeats.
2. *"Why `long`?"* → `Math.abs(Integer.MIN_VALUE)` overflows; widening first avoids it.
3. *"Number of digits in the repeating block?"* → Related to the multiplicative order of 10 mod (den/gcd).

**Common mistakes:**
- Sign bugs (use XOR on the sign checks, append `-` once).
- `Integer.MIN_VALUE` overflow on `Math.abs`.
- Inserting the `(` at the wrong index (must be the *stored* position, not current length).

---

### Q49: Permutation Sequence (factorial number system)
**Company:** Google, Amazon, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `n` and `k`, return the `k`-th permutation sequence of `1..n` (1-indexed), directly — without generating all permutations.

**What interviewer is testing:**
The **factorial number system**: instead of enumerating `k` permutations (O(k·n)), determine each digit directly using `(n-1)!` block sizes. A clean math-over-brute-force win.

**Ideal Answer:**

The `n!` permutations group into `n` blocks of `(n-1)!` each, by their first element. So the first element's index is `(k-1) / (n-1)!`; the remainder selects within that block. Recurse/iterate, removing the used number each time.

Work with `k-1` (0-indexed) and a mutable list of available digits.

```java
public String getPermutation(int n, int k) {
    // precompute factorials 0!..n!
    int[] fact = new int[n + 1];
    fact[0] = 1;
    for (int i = 1; i <= n; i++) fact[i] = fact[i - 1] * i;

    List<Integer> digits = new ArrayList<>();
    for (int i = 1; i <= n; i++) digits.add(i);   // available numbers 1..n

    k--;                                          // convert to 0-indexed
    StringBuilder sb = new StringBuilder();
    for (int i = n; i >= 1; i--) {
        int blockSize = fact[i - 1];              // (i-1)! permutations per choice
        int idx = k / blockSize;                  // which available digit to pick
        sb.append(digits.remove(idx));            // use it, remove from pool
        k %= blockSize;                           // descend into that block
    }
    return sb.toString();
}
```

**Dry run** — n=3, k=3 (permutations: 123,132,213,231,312,321). k-1=2. i=3: blockSize=2!=2, idx=2/2=1 → digits[1]=2, remove → "2", k=2%2=0. i=2: blockSize=1!=1, idx=0 → digits now [1,3], pick [0]=1 → "21", k=0. i=1: blockSize=0!=1, idx=0 → pick 3 → "213". ✅ (3rd permutation is "213".)

**Complexity:** O(n²) time (the `digits.remove(idx)` is O(n) each over n steps), O(n) space. Can be O(n log n) with a Fenwick tree for the removal.

**Follow-ups:**
1. *"Inverse — given a permutation, find its rank k?"* → For each position, count how many smaller unused digits remain, multiply by the factorial block size, sum.
2. *"n large (n > 20)?"* → `n!` overflows `int`/`long`; but k is bounded, so you only need factorials up to where they exceed k.

**Common mistakes:**
- Forgetting the `k--` (1-indexed vs 0-indexed).
- Factorial overflow for larger n (use `long`, and only as far as needed).
- Not removing the used digit from the pool.

---

### Q50: Next Greater Element III
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a positive integer `n`, find the smallest integer that has exactly the same digits and is **greater** than `n`. If no such number exists or it exceeds the 32-bit signed int range, return `-1`.

**What interviewer is testing:**
This is **Next Permutation** (file 01) applied to the digits, plus the overflow check. Recognizing the connection is the whole point.

**Ideal Answer:**

Apply the next-permutation algorithm to the digit array:
1. Scan from the right for the first digit smaller than its successor (the "pivot").
2. If none, the digits are descending → no greater arrangement → return `-1`.
3. Find the smallest digit to the right of the pivot that's larger than it; swap.
4. Reverse the suffix after the pivot (making it ascending = smallest).
Finally, parse to `long` and check it fits in `int`.

```java
public int nextGreaterElement(int n) {
    char[] d = Integer.toString(n).toCharArray();
    int i = d.length - 2;
    while (i >= 0 && d[i] >= d[i + 1]) i--;     // find pivot (first ascent from right)
    if (i < 0) return -1;                       // fully descending → no greater number

    int j = d.length - 1;
    while (d[j] <= d[i]) j--;                    // smallest digit > pivot in suffix
    swap(d, i, j);
    reverse(d, i + 1, d.length - 1);             // suffix ascending → smallest

    long val = Long.parseLong(new String(d));
    return val > Integer.MAX_VALUE ? -1 : (int) val; // overflow guard
}
private void swap(char[] a, int i, int j) { char t = a[i]; a[i] = a[j]; a[j] = t; }
private void reverse(char[] a, int l, int r) {
    while (l < r) { char t = a[l]; a[l++] = a[r]; a[r--] = t; }
}
```

**Dry run** — n=12443322: digits `12443322`. Pivot scan from right: ...2≤2≤3? compare 2&3 → `d[i]=2 < d[i+1]=3`? Find first ascent: indices ...`1 2 4 4 3 3 2 2`; from right `2≥2`, `2≤3`? `3≥2` yes descend... pivot is the `2` at index1 (`2<4`). j = smallest digit >2 in suffix `4433222`→ a `3`. Swap → `13443222`... then reverse suffix ascending. Result is the next greater permutation. (For n=12, answer is 21; for n=21, answer is -1.) ✅

**Complexity:** O(D) where D = number of digits (≤10), O(D) space.

**Follow-ups:**
1. *"This is just Next Permutation — why the extra step?"* → The 32-bit overflow check; the digit string could form a number exceeding `Integer.MAX_VALUE`.
2. *"Previous smaller arrangement?"* → Mirror the algorithm (find first descent, etc.).

**Common mistakes:**
- Forgetting the overflow check → returning a wrapped negative `int`.
- Reimplementing permutation logic incorrectly (off-by-one in pivot/swap scans).
- Not handling the descending (no-answer) case.

---

### Q51: Modular arithmetic basics (mod add/mul, modular inverse, Fermat) — reference
**Company:** Competitive programming / Google, and any problem with "answer mod 1e9+7"
**Difficulty:** 🔴 Hard (to derive) / 🟢 Easy (to apply)
**Frequency:** 🔥🔥
**Round:** Onsite / contests

**Question:**
A reference for modular arithmetic: how to add, multiply, exponentiate, and *divide* under a modulus `M` (typically `1_000_000_007`). When a problem says "return the answer modulo 1e9+7," this is the toolbox.

**What interviewer is testing:**
Whether you can keep intermediate results bounded (no overflow), and whether you know that division under a modulus means multiplying by a **modular inverse** (via Fermat's little theorem when M is prime).

**Ideal Answer — the rules and code:**

The modulus distributes over addition, subtraction, and multiplication (but **not** plain division):

```java
static final int MOD = 1_000_000_007;

// keep every intermediate in [0, MOD); use long to avoid int overflow on multiply
long add(long a, long b) { return (a + b) % MOD; }
long sub(long a, long b) { return ((a - b) % MOD + MOD) % MOD; } // +MOD: keep non-negative
long mul(long a, long b) { return (a % MOD) * (b % MOD) % MOD; }  // long*long fits in 63 bits
```

**Modular exponentiation** — `(base^exp) mod M`, by fast exponentiation:

```java
long modPow(long base, long exp, long mod) {
    long result = 1;
    base %= mod;
    while (exp > 0) {
        if ((exp & 1) == 1) result = result * base % mod; // odd bit → multiply
        base = base * base % mod;                          // square
        exp >>= 1;
    }
    return result;
}
```

**Modular inverse via Fermat's Little Theorem** — when `M` is **prime** and `gcd(a, M) = 1`:

> Fermat: `a^(M-1) ≡ 1 (mod M)`  ⟹  `a^(M-2) ≡ a^(-1) (mod M)`.

So the modular inverse of `a` is `a^(M-2) mod M`. Dividing by `a` under the modulus = multiplying by this inverse.

```java
long modInverse(long a, long mod) {  // mod must be prime, a not a multiple of mod
    return modPow(a, mod - 2, mod);
}
long modDivide(long a, long b, long mod) {  // (a / b) mod prime
    return mul(a % mod, modInverse(b, mod));
}
```

**Modular inverse via Extended Euclid** — works for any `M` coprime to `a` (not just prime):

```java
// returns gcd(a,b) and sets x,y such that a*x + b*y = gcd(a,b)
long[] extGcd(long a, long b) {
    if (b == 0) return new long[]{a, 1, 0};
    long[] r = extGcd(b, a % b);
    long g = r[0], x1 = r[1], y1 = r[2];
    return new long[]{g, y1, x1 - (a / b) * y1};
}
long modInverseEuclid(long a, long mod) {
    long[] r = extGcd(((a % mod) + mod) % mod, mod);
    if (r[0] != 1) return -1;                 // inverse doesn't exist (not coprime)
    return ((r[1] % mod) + mod) % mod;
}
```

**Worked example — `nCr mod 1e9+7`** (a constant in competitive programming): precompute factorials and inverse factorials.

```java
long nCr(int n, int r, long mod, long[] fact, long[] invFact) {
    if (r < 0 || r > n) return 0;
    return fact[n] * invFact[r] % mod * invFact[n - r] % mod;
}
// invFact[i] = modInverse(fact[i], mod), or build invFact[n] once then
// invFact[i] = invFact[i+1] * (i+1) % mod  (downward, one inverse total)
```

**Dry run (Fermat inverse)** — inverse of 3 mod 7: `3^(7-2) = 3^5 = 243 = 5 (mod 7)`. Check: `3 * 5 = 15 = 1 (mod 7)` ✓. So dividing by 3 mod 7 = multiplying by 5.

**Complexity:** modPow / modInverse are O(log exp). Factorial precompute for nCr is O(n) with a single inverse if you build invFact downward.

**Follow-ups:**
1. *"Why can't you just divide then mod?"* → Integer division loses information; `(a/b) mod M ≠ (a mod M)/(b mod M)` in general. Use the inverse.
2. *"What if M isn't prime?"* → Fermat doesn't apply; use Extended Euclid (requires only `gcd(a,M)=1`), or Euler's theorem with `φ(M)`.
3. *"Why `long` everywhere?"* → `(MOD-1)² ≈ 10¹⁸` exceeds `int` (~2·10⁹) but fits in `long` (~9·10¹⁸).

**Common mistakes:**
- `int` overflow on `a * b` before taking mod (must cast to `long`).
- Negative results from subtraction (`((a - b) % MOD + MOD) % MOD`).
- Using Fermat's inverse when `M` is not prime, or when `a` is a multiple of `M` (no inverse exists).

---

## Recap — what to recognize on sight

| Trigger in problem statement | Technique | Examples here |
|---|---|---|
| "all subsets / combinations / permutations" | Backtracking (choose-explore-unchoose) | Q1–Q9, Q11 |
| "with duplicates, no duplicate output" | Sort + skip `i > start && a[i]==a[i-1]` (or `!used[i-1]`) | Q2, Q4, Q7 |
| "place items under constraints (board/grid/cells)" | Backtracking with O(1) validity sets | Q13, Q14, Q17, Q22 |
| "count ordered ways / reuse, just the number" | DP, not backtracking | Q9 |
| "appears once/twice/thrice, find the odd one" | XOR / bit-count mod k | Q23, Q24, Q25 |
| "count set bits / power of two / single bit" | `n & (n-1)`, `n & (-n)` | Q26, Q27, Q28 |
| "no `+` / reverse bits / max XOR" | Bit-level simulation / greedy MSB | Q31, Q30, Q34 |
| "answer modulo 1e9+7 / division under mod" | Modular arithmetic + Fermat inverse | Q51 |
| "x^n in O(log n) / huge exponent" | Fast exponentiation by squaring | Q41, Q51 |
| "count primes / factor stuff up to n" | Sieve of Eratosthenes | Q40 |
| "trailing zeros / digits in a base" | Factor counting / base conversion | Q44, Q45 |
| "next bigger with same digits" | Next Permutation on digits | Q50 |
| "repeating decimal / cycle" | Remainder→position map / Floyd | Q48, Q43 |

**The three questions to ask yourself:**
1. **Am I enumerating configurations?** → Backtracking. Identify the choice, the constraint to prune on, and whether order matters (start-index vs. used-array). State the `b^d` complexity out loud.
2. **Is this an O(1)-space / "find the odd one" / parity puzzle?** → Bits. Reach for XOR, `n&(n-1)`, `n&(-n)` before anything else.
3. **Does it smell like elementary math?** → Watch for overflow (`MIN_VALUE`, factorials, products), recall the relevant identity (Euclid, Sieve, Legendre, Fermat), and prefer the O(log n) math trick over brute force.

---

Created **06-dsa-backtracking-bit-manipulation-math.md** — 51 questions covered (22 backtracking, 16 bit manipulation, 13 math & number theory), each with brute→optimal, complete Java, recursion trees / dry runs, complexity via branching^depth, follow-ups, and rejection-causing mistakes.
