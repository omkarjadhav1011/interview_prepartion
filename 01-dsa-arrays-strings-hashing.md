# 01 — DSA: Arrays, Strings & Hashing

> The foundation. 60–70% of coding rounds start here. Master these patterns and you handle the majority of phone screens and the opening of most onsites. Every question has a **complete** answer — brute force → optimal, full Java, dry run, complexity proof, follow-ups, and the mistakes that get you rejected.

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the *real* signal. Knowing this changes how you talk while you code.
- **Ideal Answer** — approach reasoning, then production-quality Java, then complexity.
- **Follow-ups** — where most candidates fail. These are scripted in real loops.
- **Common mistakes** — the specific things that flip a "lean hire" to "no hire."

The meta-skill is **pattern recognition**. By the end of this file you should be able to read a problem and say *"this is a sliding window / two-pointer / prefix-sum + hashmap / monotonic-structure problem because…"* before writing a line.

---

## Table of Contents

**Arrays**
1. Two Sum
2. Two Sum II — Input Array Is Sorted
3. 3Sum
4. 3Sum Closest
5. 4Sum
6. Pair With Given Difference
7. Best Time to Buy and Sell Stock (single transaction)
8. Best Time to Buy and Sell Stock II (unlimited)
9. Best Time to Buy and Sell Stock with Cooldown
10. Best Time to Buy and Sell Stock with Transaction Fee
11. Best Time to Buy and Sell Stock IV (at most K)
12. Maximum Subarray (Kadane's)
13. Product of Array Except Self
14. Container With Most Water
15. Trapping Rain Water
16. Next Permutation
17. Merge Intervals
18. Insert Interval
19. Meeting Rooms
20. Meeting Rooms II
21. Rotate Array
22. Missing Number
23. Find the Duplicate Number
24. Majority Element (Boyer–Moore)
25. Move Zeroes
26. Remove Duplicates from Sorted Array
27. First Missing Positive
28. Jump Game
29. Jump Game II
30. Maximum Product Subarray
31. Subarray Sum Equals K
32. Sliding Window Maximum
33. Longest Consecutive Sequence

**Strings**
34. Valid Anagram
35. Group Anagrams
36. Longest Substring Without Repeating Characters
37. Longest Palindromic Substring
38. Valid Palindrome
39. Valid Palindrome II
40. String to Integer (atoi)
41. Longest Common Prefix
42. Minimum Window Substring
43. Decode Ways
44. Edit Distance
45. Count and Say
46. String Compression
47. Implement strStr() / KMP
48. Longest Repeating Character Replacement
49. Word Break

**Hashing**
50. Design HashMap
51. Top K Frequent Elements
52. LRU Cache
53. First Unique Character in a String / Stream
54. Isomorphic Strings
55. Count Subarrays With Given XOR
56. 4Sum II
57. Random Pick with Weight
58. Encode and Decode TinyURL
59. Insert Delete GetRandom O(1)

---

# ARRAYS

The single most important array idea: **most "optimal" array solutions trade space for a single pass.** The three workhorse patterns are **two pointers**, **sliding window**, and **prefix sum + hashmap**. Almost everything below is one of those three in disguise.

---

### Q1: Two Sum
**Company:** Google, Amazon, Microsoft, Meta, Flipkart, Adobe — literally everywhere
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Given an array `nums` and an integer `target`, return the **indices** of the two numbers that add up to `target`. Exactly one solution exists; you may not use the same element twice.

**What interviewer is testing:**
Can you go from O(n²) to O(n) by recognizing that "find a complement" is a *lookup* problem? This is the gateway question — they want to hear you say "I'll trade space for time using a hashmap" *before* you write it. It's also a communication warm-up.

**Ideal Answer:**

*Brute force* — check every pair. O(n²) time, O(1) space. State it, then immediately improve.

*Optimal* — As you scan, for each `nums[i]` the number you *need* is `target - nums[i]`. If you've already seen it, you're done. So keep a map of `value -> index` of everything seen so far. One pass.

The key insight: you don't need both numbers at once. By the time you reach the second number of the pair, the first is already in the map.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value -> index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[] { seen.get(complement), i };
        }
        seen.put(nums[i], i);   // put AFTER the check so we never reuse i
    }
    throw new IllegalArgumentException("No two sum solution");
}
```

**Dry run** — `nums = [2, 7, 11, 15]`, `target = 9`:
- i=0: complement = 7, map empty → miss. put {2:0}
- i=1: complement = 2, map has 2 → return `[0, 1]`. ✅

**Complexity:** Time O(n) — single pass, O(1) average map ops. Space O(n) — the map.

**Why put after the check?** If you put first and the array had `[3,3]`, target 6, putting `3:0` then checking complement `3` would return `[0,0]` — reusing one element. Checking first guarantees the complement is a *different* index.

**Follow-up questions the interviewer will ask:**

1. *"What if the array is sorted?"* → Two pointers, O(1) extra space. (That's Q2.) Mention you'd lose original indices unless you store them first.
2. *"What if there are multiple valid pairs and you must return all unique pairs?"* → That generalizes to 3Sum-style dedup logic; sort + two pointers, skip duplicates.
3. *"What if you can't fit the array in memory?"* → External: sort on disk + two pointers, or distribute the hashmap across machines keyed by hash(value).

**Common mistakes that get you rejected:**
- Putting into the map before the complement check (reuses an element).
- Returning the *values* when asked for *indices* (read the problem).
- Jumping straight to code without saying the brute force and the trade-off out loud.

---

### Q2: Two Sum II — Input Array Is Sorted
**Company:** Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
The input array is **sorted ascending**. Return the 1-indexed positions of the two numbers adding to `target`. Use O(1) extra space.

**What interviewer is testing:**
Do you exploit sortedness? The whole point is recognizing that sorted + "find pair sum" = **two pointers**, no hashmap needed.

**Ideal Answer:**

Put one pointer at the start (`lo`), one at the end (`hi`). The sum tells you which way to move:
- If `sum < target`, the only way to increase is `lo++` (the largest available smaller value).
- If `sum > target`, decrease via `hi--`.
- If equal, return.

This works because moving inward never skips a valid pair — it's a proof by exchange argument: if the answer used `lo` but `sum < target`, no larger element pairs with `lo` either (we're at the max), so `lo` can be discarded safely.

```java
public int[] twoSumSorted(int[] numbers, int target) {
    int lo = 0, hi = numbers.length - 1;
    while (lo < hi) {
        int sum = numbers[lo] + numbers[hi];
        if (sum == target) return new int[] { lo + 1, hi + 1 };
        else if (sum < target) lo++;
        else hi--;
    }
    return new int[] { -1, -1 };
}
```

**Complexity:** Time O(n), Space O(1). Beats the hashmap on space.

**Follow-ups:**
1. *"Why does moving the pointer never miss the answer?"* → Exchange argument above; be ready to state it crisply.
2. *"Three numbers summing to target in a sorted array?"* → Fix one, two-pointer the rest = 3Sum (Q3).

**Common mistakes:**
- Using a hashmap here — wastes the O(1) space the problem hands you.
- Off-by-one on 1-indexing.

---

### Q3: 3Sum
**Company:** Google, Amazon, Meta, Microsoft, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return all **unique** triplets `[a, b, c]` such that `a + b + c == 0`. No duplicate triplets.

**What interviewer is testing:**
Two things: reducing a 3-variable problem to a 2-variable one (fix one, two-pointer the rest), and **duplicate handling** — the part 80% of candidates get wrong.

**Ideal Answer:**

Sort the array. Fix `i` as the first element; then it's a Two-Sum-Sorted for target `-nums[i]` on the subarray to the right. Sorting is what enables both the two-pointer scan and clean dedup.

Dedup rules:
- Skip `i` if `nums[i] == nums[i-1]` (same first element already explored).
- After finding a triplet, skip duplicate `lo` and `hi` values.

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    int n = nums.length;
    for (int i = 0; i < n - 2; i++) {
        if (nums[i] > 0) break;                 // smallest is positive -> impossible
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip dup anchor
        int lo = i + 1, hi = n - 1;
        while (lo < hi) {
            int sum = nums[i] + nums[lo] + nums[hi];
            if (sum == 0) {
                res.add(Arrays.asList(nums[i], nums[lo], nums[hi]));
                lo++; hi--;
                while (lo < hi && nums[lo] == nums[lo - 1]) lo++; // skip dup
                while (lo < hi && nums[hi] == nums[hi + 1]) hi--; // skip dup
            } else if (sum < 0) lo++;
            else hi--;
        }
    }
    return res;
}
```

**Dry run** — `[-1,0,1,2,-1,-4]` → sorted `[-4,-1,-1,0,1,2]`:
- i=0 (-4): lo=1,hi=5 → sums never reach 0 (too negative), no triplet.
- i=1 (-1): target 1. lo=2(-1),hi=5(2): sum 0 → add `[-1,-1,2]`; lo=3,hi=4 → (-1)+0+1=0 → add `[-1,0,1]`.
- i=2 (-1): equals nums[1] → skip.
- Result `[[-1,-1,2],[-1,0,1]]`. ✅

**Complexity:** Time O(n²) — outer loop n, inner two-pointer n. Space O(1) ignoring output / O(log n) for sort recursion.

**Follow-ups:**
1. *"4Sum?"* → Two nested fixes + two pointers = O(n³). Generalize to kSum recursion (Q5).
2. *"Why sort instead of a hashset approach?"* → Sorting makes dedup trivial and gives O(1) space; hashset 3Sum needs careful dedup and more space.
3. *"3Sum Closest?"* → Q4, track best diff instead of exact match.

**Common mistakes:**
- Forgetting the three dedup skips → duplicate triplets (instant correctness fail).
- Not handling the `nums[i] > 0` early break (works without it but shows optimization sense).
- Using a `Set<List>` to dedup — works but signals you didn't see the cleaner approach.

---

### Q4: 3Sum Closest
**Company:** Amazon, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Find three integers in `nums` whose sum is **closest** to `target`. Return that sum.

**What interviewer is testing:**
Adapting the 3Sum scaffold to an optimization (minimize |sum − target|) instead of an exact match.

**Ideal Answer:**

Same sort + fix-one + two-pointer skeleton, but instead of looking for exactly 0 we track the closest sum seen and move pointers based on whether we're below or above target.

```java
public int threeSumClosest(int[] nums, int target) {
    Arrays.sort(nums);
    int best = nums[0] + nums[1] + nums[2];
    for (int i = 0; i < nums.length - 2; i++) {
        int lo = i + 1, hi = nums.length - 1;
        while (lo < hi) {
            int sum = nums[i] + nums[lo] + nums[hi];
            if (Math.abs(sum - target) < Math.abs(best - target)) best = sum;
            if (sum == target) return sum;     // can't do better than exact
            else if (sum < target) lo++;
            else hi--;
        }
    }
    return best;
}
```

**Complexity:** O(n²) time, O(1) space.

**Follow-ups:**
1. *"Return the triplet instead of the sum?"* → store the triple when you update `best`.
2. *"Early termination?"* → return immediately on exact equality (shown).

**Common mistakes:**
- Initializing `best` to `0` or `Integer.MAX_VALUE` (breaks the `|best - target|` comparison or overflows). Initialize to an actual valid triplet sum.

---

### Q5: 4Sum
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return all unique quadruplets summing to `target`. Note `target` and values can be large — watch for int overflow.

**What interviewer is testing:**
Generalization (kSum recursion) and overflow awareness.

**Ideal Answer:**

Two nested anchors + two pointers gives O(n³). The clean version is a **kSum recursion**: reduce kSum to (k-1)Sum until k=2, then two-pointer. Use `long` for sums to avoid overflow.

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    Arrays.sort(nums);
    return kSum(nums, (long) target, 0, 4);
}

private List<List<Integer>> kSum(int[] nums, long target, int start, int k) {
    List<List<Integer>> res = new ArrayList<>();
    int n = nums.length;
    if (start == n) return res;
    // pruning: smallest possible / largest possible sum bounds
    long avg = target / k;
    if (nums[start] > avg || nums[n - 1] < avg) return res;

    if (k == 2) return twoSum(nums, target, start);

    for (int i = start; i < n; i++) {
        if (i > start && nums[i] == nums[i - 1]) continue;
        for (List<Integer> sub : kSum(nums, target - nums[i], i + 1, k - 1)) {
            List<Integer> quad = new ArrayList<>();
            quad.add(nums[i]);
            quad.addAll(sub);
            res.add(quad);
        }
    }
    return res;
}

private List<List<Integer>> twoSum(int[] nums, long target, int start) {
    List<List<Integer>> res = new ArrayList<>();
    int lo = start, hi = nums.length - 1;
    while (lo < hi) {
        long sum = nums[lo] + nums[hi];
        if (sum < target || (lo > start && nums[lo] == nums[lo - 1])) lo++;
        else if (sum > target || (hi < nums.length - 1 && nums[hi] == nums[hi + 1])) hi--;
        else {
            res.add(Arrays.asList(nums[lo++], nums[hi--]));
        }
    }
    return res;
}
```

**Complexity:** O(n^{k-1}) = O(n³) for 4Sum. Space O(k) recursion depth + output.

**Follow-ups:**
1. *"Why long?"* → Four `int`s near `Integer.MAX_VALUE` overflow when summed.
2. *"kSum for arbitrary k?"* → This recursion already handles it.

**Common mistakes:**
- Int overflow (the classic 4Sum trap).
- Dedup only at the top level, forgetting it in the recursion / two-pointer base.

---

### Q6: Pair With Given Difference
**Company:** Amazon, Goldman Sachs
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** OA / Phone

**Question:**
Given an array and a value `k ≥ 0`, find if there exist two elements with `|a - b| = k`.

**What interviewer is testing:**
Recognizing that "difference k" is also a complement-lookup (`a + k` or `a - k`), plus the `k == 0` edge case.

**Ideal Answer:**

For each element `x`, we want `x + k` to also exist. A set lookup answers this in O(1). Special-case `k == 0`: we then need a *duplicate*, so count occurrences.

```java
public boolean hasPairWithDifference(int[] arr, int k) {
    if (k < 0) return false;
    Set<Integer> set = new HashSet<>();
    if (k == 0) {                       // need duplicates
        for (int x : arr) {
            if (!set.add(x)) return true;
        }
        return false;
    }
    for (int x : arr) set.add(x);
    for (int x : arr) {
        if (set.contains(x + k)) return true;
    }
    return false;
}
```

**Complexity:** O(n) time, O(n) space. (Sorted-array variant: two pointers, O(1) extra.)

**Follow-ups:**
1. *"Sorted array, O(1) space?"* → Two pointers where both move forward (a "same-direction" two-pointer): advance `j` until `arr[j]-arr[i] >= k`.
2. *"Count all such pairs?"* → Adapt with frequency map.

**Common mistakes:**
- Missing the `k == 0` duplicate case.
- Checking both `x+k` and `x-k` (redundant — every pair gets found from its smaller element via `x+k`).

---

### Q7: Best Time to Buy and Sell Stock (Single Transaction)
**Company:** Amazon, Microsoft, Meta, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
`prices[i]` is the stock price on day `i`. Buy once and sell once (sell after buy). Max profit, or 0 if none.

**What interviewer is testing:**
Single-pass thinking — track the minimum so far and the best profit. It's Kadane's in disguise.

**Ideal Answer:**

The best profit selling on day `i` is `prices[i] - min(prices[0..i])`. So track the running minimum buy price and the running max profit.

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;
    int best = 0;
    for (int p : prices) {
        if (p < minPrice) minPrice = p;          // best day to have bought so far
        else best = Math.max(best, p - minPrice); // sell today
    }
    return best;
}
```

**Dry run** — `[7,1,5,3,6,4]`: min→7,1,1,1,1,1; profit→0,0,4,4,**5**,5. Answer 5 (buy at 1, sell at 6). ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:** Variants Q8–Q11 (unlimited, cooldown, fee, K transactions).

**Common mistakes:**
- Allowing sell-before-buy (using global max − global min).
- Returning negative profit instead of 0.

---

### Q8: Best Time to Buy and Sell Stock II (Unlimited Transactions)
**Company:** Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
You may complete as many transactions as you like (buy/sell repeatedly, but hold at most one share at a time). Max profit.

**What interviewer is testing:**
The greedy insight: capture every upward step.

**Ideal Answer:**

Any rising sequence's total gain equals the sum of consecutive positive differences. Mathematically, `p[k]-p[0] = (p[1]-p[0]) + (p[2]-p[1]) + ...`, so summing only the positive day-over-day deltas captures every profitable climb.

```java
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++) {
        if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
    }
    return profit;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Prove the greedy."* → Telescoping sum argument above.
2. *"With a transaction fee?"* → Greedy breaks; need DP (Q10).

**Common mistakes:**
- Overcomplicating with DP when greedy is correct here.

---

### Q9: Best Time to Buy and Sell Stock with Cooldown
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Unlimited transactions, but after you sell you must **cooldown one day** before buying again.

**What interviewer is testing:**
State-machine DP. Three states: holding, sold-today (cooldown), rest (can buy).

**Ideal Answer:**

Define per day:
- `hold` = max profit while holding a stock.
- `sold` = max profit having just sold today (next day is cooldown).
- `rest` = max profit not holding and free to buy.

Transitions:
- `hold = max(prevHold, prevRest - price)` (keep holding, or buy from a rest day).
- `sold = prevHold + price` (sell what we held).
- `rest = max(prevRest, prevSold)` (stay free, or come out of cooldown).

```java
public int maxProfit(int[] prices) {
    int hold = Integer.MIN_VALUE, sold = 0, rest = 0;
    for (int price : prices) {
        int prevSold = sold;
        sold = hold + price;
        hold = Math.max(hold, rest - price);
        rest = Math.max(rest, prevSold);
    }
    return Math.max(sold, rest); // never end holding
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Cooldown of k days?"* → `rest` becomes a window of the last k `sold` states (or a rolling array).

**Common mistakes:**
- Forgetting `hold` must initialize to −∞ (you can't "have bought" before day 0 with profit 0).
- Updating states in the wrong order so they read each other's new values — snapshot `prevSold`.

---

### Q10: Best Time to Buy and Sell Stock with Transaction Fee
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Unlimited transactions; pay `fee` per transaction. Max profit.

**Ideal Answer:**

Two states: `cash` (not holding) and `hold` (holding). Pay the fee on sell.

```java
public int maxProfit(int[] prices, int fee) {
    long cash = 0, hold = -prices[0];
    for (int i = 1; i < prices.length; i++) {
        cash = Math.max(cash, hold + prices[i] - fee); // sell, pay fee
        hold = Math.max(hold, cash - prices[i]);       // buy
    }
    return (int) cash;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Why pay the fee on sell not buy?"* → Either is fine as long as you're consistent; charging on sell keeps `hold` simple.

**Common mistakes:** Double-charging the fee, or charging it on a hold day.

---

### Q11: Best Time to Buy and Sell Stock IV (At Most K Transactions)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
At most `k` transactions. Max profit.

**What interviewer is testing:**
2D DP and the crucial optimization: if `k >= n/2`, the constraint is unlimited (Q8).

**Ideal Answer:**

Let `buy[j]` = best profit after at most `j` buys while currently holding; `sell[j]` = best after at most `j` complete transactions. Iterate prices, updating each j.

```java
public int maxProfit(int k, int[] prices) {
    int n = prices.length;
    if (n == 0 || k == 0) return 0;
    if (k >= n / 2) {                        // unlimited transactions case
        int profit = 0;
        for (int i = 1; i < n; i++)
            if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
        return profit;
    }
    int[] buy = new int[k + 1];
    int[] sell = new int[k + 1];
    Arrays.fill(buy, Integer.MIN_VALUE);
    for (int price : prices) {
        for (int j = 1; j <= k; j++) {
            buy[j]  = Math.max(buy[j], sell[j - 1] - price); // j-th buy
            sell[j] = Math.max(sell[j], buy[j] + price);     // j-th sell
        }
    }
    return sell[k];
}
```

**Complexity:** O(n·k) time, O(k) space.

**Follow-ups:**
1. *"Why the k ≥ n/2 shortcut?"* → Each transaction needs ≥2 days; with n days you can't make more than n/2, so the cap stops binding.
2. *"Reconstruct the actual buy/sell days?"* → Keep parent pointers / a 2D table.

**Common mistakes:**
- Skipping the k ≥ n/2 case → TLE on large k.
- Initializing `buy` to 0 instead of −∞.

---

### Q12: Maximum Subarray (Kadane's Algorithm)
**Company:** Amazon, Microsoft, Google, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Find the contiguous subarray with the largest sum; return that sum.

**What interviewer is testing:**
Kadane's — and whether you understand the *intuition*, not just memorized the loop.

**Ideal Answer:**

The intuition: a running prefix is only worth keeping if it's positive. If the sum so far is negative, it can only drag down whatever comes next — drop it and start fresh at the current element.

So `cur = max(nums[i], cur + nums[i])`: either extend the previous subarray or restart at `i`. Track the global best.

```java
public int maxSubArray(int[] nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < nums.length; i++) {
        cur = Math.max(nums[i], cur + nums[i]);  // extend or restart
        best = Math.max(best, cur);
    }
    return best;
}
```

**Dry run** — `[-2,1,-3,4,-1,2,1,-5,4]`: cur→-2,1,-2,4,3,5,6,1,5; best→-2,1,1,4,4,5,**6**,6,6. Answer 6 (`[4,-1,2,1]`). ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Return the subarray indices, not just the sum?"* → Track start when you restart, and end when you update best.
2. *"All negatives?"* → Initializing with `nums[0]` (not 0) handles it correctly.
3. *"Maximum sum circular subarray?"* → Answer = `max(normalKadane, totalSum − minSubarray)`, with an all-negative guard.

**Common mistakes:**
- Initializing `best = 0` → wrong for all-negative arrays.
- Confusing "subarray" (contiguous) with "subsequence."

---

### Q13: Product of Array Except Self
**Company:** Amazon, Microsoft, Meta, Apple
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return `out[i] = product of all elements except nums[i]`, **without division**, in O(n) and ideally O(1) extra space (output array doesn't count).

**What interviewer is testing:**
Prefix/suffix products, and the no-division constraint (which blocks the obvious "total / nums[i]" and forces handling zeros properly).

**Ideal Answer:**

`out[i] = (product of everything left of i) × (product of everything right of i)`. Compute left products in `out` in a forward pass, then multiply by running right products in a backward pass.

```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] out = new int[n];
    out[0] = 1;
    for (int i = 1; i < n; i++) out[i] = out[i - 1] * nums[i - 1]; // prefix
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        out[i] *= right;       // prefix * suffix
        right *= nums[i];      // extend suffix
    }
    return out;
}
```

**Dry run** — `[1,2,3,4]`: prefix `out=[1,1,2,6]`; backward: i=3 out=6*1=6,right=4; i=2 out=2*4=8,right=12; i=1 out=1*12=12,right=24; i=0 out=1*24=24 → `[24,12,8,6]`. ✅

**Complexity:** O(n) time, O(1) extra (output excluded).

**Follow-ups:**
1. *"Why not division?"* → Fails when any element is 0 (and the all-pairs product needs special casing for one vs. multiple zeros).
2. *"How would the division approach handle zeros?"* → Count zeros: 0 zeros → divide; 1 zero → only that index nonzero; ≥2 zeros → all zero.

**Common mistakes:**
- Using extra arrays for prefix and suffix (works but misses the O(1) reuse of the output).
- Integer overflow on large inputs — mention `long` if values can be big.

---

### Q14: Container With Most Water
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
`height[i]` are vertical lines. Pick two lines that with the x-axis form a container holding the most water. Return the max area.

**What interviewer is testing:**
Two pointers with a **correctness proof** for why moving the shorter line is safe.

**Ideal Answer:**

Area = `width × min(height[lo], height[hi])`. Start with the widest container (lo=0, hi=n−1). The shorter wall caps the area, and moving it inward is the only move that *could* increase area — moving the taller wall keeps the same limiting height but shrinks width, so it can never improve. Therefore discard the shorter wall.

```java
public int maxArea(int[] height) {
    int lo = 0, hi = height.length - 1, best = 0;
    while (lo < hi) {
        int h = Math.min(height[lo], height[hi]);
        best = Math.max(best, h * (hi - lo));
        if (height[lo] < height[hi]) lo++;
        else hi--;
    }
    return best;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Prove moving the shorter line never skips the optimum."* → If we moved the taller one, width drops and height ≤ old min, so area strictly can't grow; thus the optimum involving the current shorter wall is already considered. Be ready to articulate this.

**Common mistakes:**
- Moving the taller pointer.
- Using `max` of heights instead of `min` (water spills over the shorter side).

---

### Q15: Trapping Rain Water
**Company:** Amazon, Google, Goldman Sachs, Flipkart
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given elevation heights of unit width, compute total trapped rainwater.

**What interviewer is testing:**
Multiple solution levels and the optimal two-pointer. Water above bar `i` = `min(maxLeft[i], maxRight[i]) − height[i]`.

**Ideal Answer:**

*Approach 1 — brute force:* for each i, scan left and right for maxes. O(n²).

*Approach 2 — prefix arrays:* precompute `maxLeft[]` and `maxRight[]`, then sum. O(n) time, O(n) space.

*Approach 3 — two pointers (optimal):* Maintain `leftMax`, `rightMax`. Whichever side has the smaller running max bounds the water there, so process it. O(n) time, O(1) space.

```java
public int trap(int[] height) {
    int lo = 0, hi = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (lo < hi) {
        if (height[lo] < height[hi]) {       // left side is the bottleneck
            leftMax = Math.max(leftMax, height[lo]);
            water += leftMax - height[lo];
            lo++;
        } else {
            rightMax = Math.max(rightMax, height[hi]);
            water += rightMax - height[hi];
            hi--;
        }
    }
    return water;
}
```

*Approach 4 — monotonic stack:* push decreasing bars; when a taller bar arrives, pop and add water in the "valley" bounded by the new bar and the next on the stack. O(n)/O(n). Good to mention; the two-pointer is cleaner.

**Why two pointers works:** When `height[lo] < height[hi]`, we *know* there's a wall on the right at least as tall as `height[hi] > height[lo]`, so water over `lo` depends only on `leftMax`. We never need the exact rightMax — just that it's ≥ current.

**Complexity:** Optimal O(n) time, O(1) space.

**Follow-ups:**
1. *"Trapping Rain Water II (2D grid)?"* → Min-heap from the border inward (Dijkstra-like). Mention it; it's a separate hard problem.
2. *"Stack approach details?"* → Be ready to sketch the valley computation.

**Common mistakes:**
- Adding water before updating the running max.
- Off-by-one in the prefix-array version at the boundaries.

---

### Q16: Next Permutation
**Company:** Google, Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Rearrange `nums` into the lexicographically next greater permutation, in place. If none (descending), wrap to the smallest (ascending). O(1) extra space.

**What interviewer is testing:**
Whether you understand the *algorithm*, not memorized steps.

**Ideal Answer:**

To get the next permutation:
1. Scan from the right to find the first index `i` where `nums[i] < nums[i+1]` (the "pivot"). The suffix after `i` is non-increasing — already the largest arrangement of those elements.
2. If no such `i`, the whole array is descending → reverse it (smallest).
3. Otherwise find the rightmost element `j > i` with `nums[j] > nums[i]` (the smallest value greater than pivot in the suffix), swap `i` and `j`.
4. Reverse the suffix after `i` to make it ascending (smallest).

```java
public void nextPermutation(int[] nums) {
    int n = nums.length, i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;   // find pivot
    if (i >= 0) {
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;             // smallest > pivot in suffix
        swap(nums, i, j);
    }
    reverse(nums, i + 1, n - 1);                    // suffix ascending
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
private void reverse(int[] a, int l, int r) { while (l < r) swap(a, l++, r--); }
```

**Dry run** — `[1,2,3]`: pivot i=1 (2<3); j=2 (3>2); swap→`[1,3,2]`; reverse suffix [2..2] → `[1,3,2]`. ✅ `[3,2,1]`: no pivot → reverse → `[1,2,3]`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Previous permutation?"* → Mirror: find first `i` with `nums[i] > nums[i+1]`, etc.

**Common mistakes:**
- `>` vs `>=` in the pivot/swap scans (handles duplicates correctly).
- Forgetting the descending wrap-around case.

---

### Q17: Merge Intervals
**Company:** Amazon, Google, Microsoft, Meta, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given intervals, merge all overlapping ones.

**What interviewer is testing:**
The "sort by start, then sweep" pattern — the basis of half a dozen interval problems.

**Ideal Answer:**

Sort by start time. Walk through; if the current interval's start ≤ the last merged interval's end, they overlap → extend the end. Otherwise start a new interval.

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    List<int[]> out = new ArrayList<>();
    int[] cur = intervals[0];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= cur[1]) {
            cur[1] = Math.max(cur[1], intervals[i][1]); // overlap -> extend
        } else {
            out.add(cur);
            cur = intervals[i];
        }
    }
    out.add(cur);
    return out.toArray(new int[0][]);
}
```

**Dry run** — `[[1,3],[2,6],[8,10],[15,18]]`: cur=[1,3]; [2,6] overlaps→[1,6]; [8,10] no→push [1,6], cur=[8,10]; [15,18] no→push [8,10], cur=[15,18]; push. → `[[1,6],[8,10],[15,18]]`. ✅

**Complexity:** O(n log n) time (sort dominates), O(n) output.

**Follow-ups:**
1. *"Insert interval into a sorted non-overlapping list?"* → Q18, O(n) no sort.
2. *"Why `Math.max` on the end?"* → `[1,5],[2,3]`: the second is nested; don't shrink the end.

**Common mistakes:**
- Sorting by end instead of start.
- Mutating shared interval references unexpectedly.

---

### Q18: Insert Interval
**Company:** Google, Amazon, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a sorted, non-overlapping interval list and a new interval, insert it and merge. O(n).

**Ideal Answer:**

Three phases: add all intervals ending before the new one starts; merge all that overlap the new one; add the rest.

```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> out = new ArrayList<>();
    int i = 0, n = intervals.length;
    while (i < n && intervals[i][1] < newInterval[0]) out.add(intervals[i++]); // before
    while (i < n && intervals[i][0] <= newInterval[1]) {                       // overlap
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    out.add(newInterval);
    while (i < n) out.add(intervals[i++]);                                     // after
    return out.toArray(new int[0][]);
}
```

**Complexity:** O(n) time, O(n) output.

**Follow-up:** *"Could you binary-search the insertion point?"* → Yes for the "before" boundary; still O(n) due to merging.

**Common mistakes:** `<` vs `<=` on the overlap boundary (touching intervals like `[1,2],[2,3]` should merge).

---

### Q19: Meeting Rooms
**Company:** Amazon, Google, Meta
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given meeting intervals, can a person attend all of them (no overlaps)?

**Ideal Answer:**

Sort by start; if any meeting starts before the previous ends, conflict.

```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) return false;
    }
    return true;
}
```

**Complexity:** O(n log n) time, O(1) space.

**Follow-up:** Q20 — minimum rooms to hold all meetings.

**Common mistakes:** Treating `[1,2],[2,3]` (touching) as a conflict — typically allowed.

---

### Q20: Meeting Rooms II
**Company:** Amazon, Google, Microsoft, Uber, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Minimum number of conference rooms required for all meetings.

**What interviewer is testing:**
Two classic approaches — min-heap of end times, or the sweep-line (chronological events). This is one of the most-asked scheduling problems.

**Ideal Answer:**

*Heap approach:* sort by start. A min-heap holds end times of ongoing meetings. For each meeting, if the earliest-ending room is free (its end ≤ this start), reuse it (poll); always add this meeting's end. Heap size = rooms in use; its max is the answer.

```java
public int minMeetingRooms(int[][] intervals) {
    if (intervals.length == 0) return 0;
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    PriorityQueue<Integer> endTimes = new PriorityQueue<>(); // min-heap of end times
    for (int[] meet : intervals) {
        if (!endTimes.isEmpty() && endTimes.peek() <= meet[0]) {
            endTimes.poll();         // a room freed up
        }
        endTimes.add(meet[1]);
    }
    return endTimes.size();
}
```

*Sweep-line alternative:* separate sorted start[] and end[] arrays; two pointers; +1 room on a start, −1 when an end passes; track max concurrent. Also O(n log n).

```java
public int minMeetingRoomsSweep(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n], ends = new int[n];
    for (int i = 0; i < n; i++) { starts[i] = intervals[i][0]; ends[i] = intervals[i][1]; }
    Arrays.sort(starts); Arrays.sort(ends);
    int rooms = 0, max = 0, e = 0;
    for (int s = 0; s < n; s++) {
        while (e < n && ends[e] <= starts[s]) { rooms--; e++; }
        rooms++;
        max = Math.max(max, rooms);
    }
    return max;
}
```

**Complexity:** O(n log n) time, O(n) space.

**Follow-ups:**
1. *"Which rooms exactly (assign IDs)?"* → Heap of (endTime, roomId).
2. *"Stream of meetings, can't sort?"* → Approximate; or balanced BST of events.

**Common mistakes:**
- Using `<` vs `<=` inconsistently between the two approaches.
- Forgetting to always add the current meeting's end.

---

### Q21: Rotate Array
**Company:** Amazon, Microsoft, Apple
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Rotate `nums` right by `k` steps, in place, O(1) extra space.

**What interviewer is testing:**
The three-reversal trick (or cyclic replacement).

**Ideal Answer:**

`k %= n` first. Reverse the whole array, then reverse the first `k`, then reverse the rest. Each element lands in its rotated position.

```java
public void rotate(int[] nums, int k) {
    int n = nums.length;
    k %= n;
    reverse(nums, 0, n - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, n - 1);
}
private void reverse(int[] a, int l, int r) {
    while (l < r) { int t = a[l]; a[l++] = a[r]; a[r--] = t; }
}
```

**Dry run** — `[1,2,3,4,5,6,7]`, k=3: reverse all `[7,6,5,4,3,2,1]`; reverse[0..2] `[5,6,7,4,3,2,1]`; reverse[3..6] `[5,6,7,1,2,3,4]`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Cyclic replacement approach?"* → Move each element to its target, count moves until n placed; handle gcd cycles.
2. *"Why `k %= n`?"* → Rotating by n is identity; avoids out-of-bounds.

**Common mistakes:** Forgetting `k %= n`; using an O(n) temp array when O(1) is asked.

---

### Q22: Missing Number
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Array of `n` distinct numbers in range `[0, n]`. One is missing; find it.

**What interviewer is testing:**
Knowing multiple tricks — sum formula, XOR (overflow-safe).

**Ideal Answer:**

*Sum formula:* expected `n(n+1)/2` minus actual sum = missing. Simple but can overflow for large n.

*XOR (no overflow):* XOR all indices `0..n` with all values; pairs cancel, leaving the missing number.

```java
public int missingNumber(int[] nums) {
    int xor = nums.length;            // include n
    for (int i = 0; i < nums.length; i++) xor ^= i ^ nums[i];
    return xor;
}
```

**Dry run** — `[3,0,1]`: xor=3; i0: ^0^3=0; i1:^1^0=1; i2:^2^1=2 → 2. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Sum vs XOR trade-off?"* → XOR avoids overflow.
2. *"Two numbers missing?"* → Sum + sum-of-squares, or XOR-and-partition.

**Common mistakes:** Overflow with the sum approach on large inputs (use `long` or XOR).

---

### Q23: Find the Duplicate Number
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
`n+1` integers each in `[1, n]`. Exactly one value is duplicated (possibly many times). Find it **without modifying the array** and O(1) extra space.

**What interviewer is testing:**
Floyd's cycle detection applied to an array-as-linked-list. The constraints rule out sorting (modifies) and hashset (O(n) space).

**Ideal Answer:**

Treat `i -> nums[i]` as a linked list. Since values are in `[1,n]` and there are `n+1` of them, there's a cycle, and the duplicate value is the cycle's entrance. Apply Floyd's tortoise & hare.

```java
public int findDuplicate(int[] nums) {
    int slow = nums[0], fast = nums[0];
    do {                                  // phase 1: find a meeting point in the cycle
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    slow = nums[0];                       // phase 2: find cycle entrance
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

**Why it works:** The duplicate value means two indices point to the same node — that node is where the cycle begins. Phase 2's equal-speed walk from the head and the meeting point converge at the entrance (standard Floyd proof).

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Binary search on value approach?"* → Count how many nums ≤ mid; if more than mid, duplicate is in the lower half. O(n log n), also no modification.
2. *"Multiple duplicates?"* → Problem guarantees one duplicate *value*; Floyd still works.

**Common mistakes:**
- Sorting or using a set (violates constraints).
- Getting the two-phase Floyd structure wrong (slow advances 1, fast 2 in phase 1; both 1 in phase 2).

---

### Q24: Majority Element (Boyer–Moore Voting)
**Company:** Amazon, Microsoft, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
An element appears more than ⌊n/2⌋ times. Find it. O(1) space.

**What interviewer is testing:**
Boyer–Moore voting and *why* it works.

**Ideal Answer:**

Maintain a `candidate` and a `count`. When count hits 0, adopt the current element as candidate. Increment if it matches the candidate, else decrement. The majority survives because every non-majority element can cancel at most one majority vote, and majority has > n/2 votes.

```java
public int majorityElement(int[] nums) {
    int candidate = nums[0], count = 0;
    for (int x : nums) {
        if (count == 0) candidate = x;
        count += (x == candidate) ? 1 : -1;
    }
    return candidate;
}
```

**Dry run** — `[2,2,1,1,1,2,2]`: (cand,count): (2,1)(2,2)(2,1)(2,0)→(1,1)(1,0)→(2,1)(2,2) → 2. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Elements appearing > n/3?"* → At most two such; Boyer–Moore with two candidates, then verify counts.
2. *"No guarantee a majority exists?"* → Second pass to verify the candidate's count.

**Common mistakes:** Assuming a majority always exists when the problem doesn't guarantee it — verify.

---

### Q25: Move Zeroes
**Company:** Meta, Amazon, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Move all 0s to the end while keeping non-zero order, in place.

**Ideal Answer:**

Two pointers: `insert` marks where the next non-zero goes. Scan; on each non-zero, write it at `insert++`. Fill the rest with zeros (or swap as you go).

```java
public void moveZeroes(int[] nums) {
    int insert = 0;
    for (int x : nums) if (x != 0) nums[insert++] = x;
    while (insert < nums.length) nums[insert++] = 0;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Minimize writes?"* → Swap-based version only writes when a zero is behind a non-zero.

**Common mistakes:** Breaking relative order; using extra arrays.

---

### Q26: Remove Duplicates from Sorted Array
**Company:** Microsoft, Amazon
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Remove duplicates in place from a sorted array; return new length. First k elements must hold the unique values.

**Ideal Answer:**

Slow/fast pointers. `slow` is the last unique position; advance `fast`, and when `nums[fast] != nums[slow]`, write it at `++slow`.

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int slow = 0;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) nums[++slow] = nums[fast];
    }
    return slow + 1;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Allow each value at most twice?"* → Compare against `nums[slow-1]` and start slow at 1.

**Common mistakes:** Off-by-one on the returned length (`slow + 1`, not `slow`).

---

### Q27: First Missing Positive
**Company:** Amazon, Google, Stripe
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Find the smallest missing **positive** integer. O(n) time, O(1) extra space.

**What interviewer is testing:**
Index-as-hash trick. The answer is always in `[1, n+1]`, so the array itself can be the hashtable.

**Ideal Answer:**

For an array of length `n`, the answer is in `[1, n+1]`. Place each value `v` (where `1 ≤ v ≤ n`) at index `v−1` via swaps (cyclic sort). Then scan: the first index `i` where `nums[i] != i+1` gives answer `i+1`. If all match, answer is `n+1`.

```java
public int firstMissingPositive(int[] nums) {
    int n = nums.length;
    for (int i = 0; i < n; i++) {
        // place nums[i] at its correct slot if in range and not already there
        while (nums[i] >= 1 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            int t = nums[nums[i] - 1];
            nums[nums[i] - 1] = nums[i];
            nums[i] = t;
        }
    }
    for (int i = 0; i < n; i++) {
        if (nums[i] != i + 1) return i + 1;
    }
    return n + 1;
}
```

**Dry run** — `[3,4,-1,1]`: place 3→idx2: `[−1,4,3,1]`; place... 4→idx3: `[−1,1,3,4]`; place 1→idx0: `[1,−1,3,4]`; scan: idx1 holds −1≠2 → answer **2**. ✅

**Complexity:** O(n) time (each element moved at most once into place), O(1) space.

**Follow-ups:**
1. *"Why is the answer ≤ n+1?"* → With n slots, if `1..n` all present the answer is `n+1`; otherwise one of `1..n` is missing.
2. *"Negative/zero handling?"* → Ignored during placement; they just occupy "wrong" slots that flag the answer.

**Common mistakes:**
- Using a `while` not an `if` for the swap (a single swap may bring another out-of-place value).
- Infinite loop when duplicates exist — the `nums[nums[i]-1] != nums[i]` guard prevents it.

---

### Q28: Jump Game
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
`nums[i]` = max jump length from `i`. Can you reach the last index?

**What interviewer is testing:**
Greedy: track the farthest reachable index.

**Ideal Answer:**

Track `reach`, the farthest index reachable so far. If you ever stand on an index beyond `reach`, you're stuck. Otherwise extend reach.

```java
public boolean canJump(int[] nums) {
    int reach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > reach) return false;            // can't even get here
        reach = Math.max(reach, i + nums[i]);
        if (reach >= nums.length - 1) return true;
    }
    return true;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** Q29 — minimum jumps.

**Common mistakes:** DP O(n²) when greedy O(n) suffices; not handling `i > reach`.

---

### Q29: Jump Game II
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Minimum number of jumps to reach the last index (guaranteed reachable).

**Ideal Answer:**

Greedy BFS by "levels." `curEnd` is the farthest reachable with the current number of jumps; `farthest` is the best we can extend to. When `i` reaches `curEnd`, we must jump: increment `jumps`, set `curEnd = farthest`.

```java
public int jump(int[] nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {  // no need to act at last index
        farthest = Math.max(farthest, i + nums[i]);
        if (i == curEnd) {                       // exhausted current jump's range
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

**Dry run** — `[2,3,1,1,4]`: i0 far=2,i==curEnd0→jumps1,curEnd2; i1 far=4; i2 i==curEnd2→jumps2,curEnd4; i3 far=4. → 2. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Why is this BFS?"* → Each "level" = indices reachable with the same jump count; you expand the frontier.

**Common mistakes:** Looping to the last index and over-counting a final jump.

---

### Q30: Maximum Product Subarray
**Company:** Amazon, LinkedIn, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Contiguous subarray with the largest **product**. Return the product.

**What interviewer is testing:**
Handling negatives (a negative flips min↔max) and zeros (reset).

**Ideal Answer:**

Unlike sum, a large negative product can become the max when multiplied by another negative. So track **both** the running max and min product; on a negative number, swap them before updating.

```java
public int maxProduct(int[] nums) {
    int max = nums[0], min = nums[0], best = nums[0];
    for (int i = 1; i < nums.length; i++) {
        int x = nums[i];
        if (x < 0) { int t = max; max = min; min = t; } // negative flips roles
        max = Math.max(x, max * x);
        min = Math.min(x, min * x);
        best = Math.max(best, max);
    }
    return best;
}
```

**Dry run** — `[2,3,-2,4]`: start max=min=best=2; x3: max6,min3,best6; x-2: swap→max3,min6; max=max(-2,-6)=-2,min=min(-2,-12)=-12,best6; x4: max=max(4,-8)=4,min=-48,best6. → 6. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why track min?"* → A negative × current min (very negative) can yield the new max.
2. *"Zeros?"* → `Math.max(x, ...)` naturally restarts the window at the next element.

**Common mistakes:** Only tracking max (fails on `[-2,3,-4]`).

---

### Q31: Subarray Sum Equals K
**Company:** Amazon, Google, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Count the number of contiguous subarrays summing to exactly `k`. Values can be negative.

**What interviewer is testing:**
**Prefix sum + hashmap** — the single most important array pattern. (Sliding window fails because of negatives.)

**Ideal Answer:**

Let `pre[i]` = sum of `nums[0..i]`. A subarray `(j, i]` sums to k iff `pre[i] − pre[j] = k`, i.e. `pre[j] = pre[i] − k`. So as we scan and maintain the running prefix sum, count how many earlier prefix sums equal `cur − k`. A hashmap of `prefixSum -> frequency` gives O(1) lookups. Seed it with `{0: 1}` to count subarrays starting at index 0.

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    freq.put(0, 1);                  // empty prefix
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += freq.getOrDefault(sum - k, 0);
        freq.merge(sum, 1, Integer::sum);
    }
    return count;
}
```

**Dry run** — `[1,1,1]`, k=2: freq{0:1}; x1 sum1 count+=freq[-1]=0,freq{0:1,1:1}; x1 sum2 count+=freq[0]=1,freq{...,2:1}; x1 sum3 count+=freq[1]=1→2. → 2. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why not sliding window?"* → Negatives break the monotonic-window assumption.
2. *"Longest subarray summing to k?"* → Store the *first* index of each prefix sum and track max length.
3. *"Subarray with sum divisible by k?"* → Bucket prefix sums by `sum % k`.

**Common mistakes:**
- Forgetting `{0:1}` seed → misses subarrays starting at index 0.
- Incrementing the map before counting → can count the current prefix against itself.

---

### Q32: Sliding Window Maximum
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given a window of size `k` sliding across `nums`, return the max in each window.

**What interviewer is testing:**
Monotonic **deque** — achieving O(n) instead of O(n·k) or O(n log k).

**Ideal Answer:**

Maintain a deque of **indices** whose values are in decreasing order. The front is always the current window's max.
- Before adding `i`, pop from the back any indices whose values are ≤ `nums[i]` (they can never be the max while `i` is in the window).
- Pop from the front if it's outside the window (`<= i - k`).
- Once `i >= k - 1`, the front is that window's answer.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] res = new int[n - k + 1];
    Deque<Integer> dq = new ArrayDeque<>(); // indices, values decreasing
    for (int i = 0; i < n; i++) {
        if (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst(); // out of window
        while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast(); // smaller -> useless
        dq.offerLast(i);
        if (i >= k - 1) res[i - k + 1] = nums[dq.peekFirst()];
    }
    return res;
}
```

**Dry run** — `[1,3,-1,-3,5,3,6,7]`, k=3: windows max → 3,3,5,5,6,7. (Deque keeps decreasing indices; front is max.) ✅

**Complexity:** O(n) time (each index pushed/popped once), O(k) space.

**Follow-ups:**
1. *"Sliding window minimum?"* → Mirror: increasing deque.
2. *"Why store indices not values?"* → Need indices to evict out-of-window elements.
3. *"Heap approach?"* → O(n log k) with lazy deletion; mention as the simpler-but-slower alternative.

**Common mistakes:** Storing values (can't check window bounds); wrong pop order.

---

### Q33: Longest Consecutive Sequence
**Company:** Google, Amazon, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Find the length of the longest run of consecutive integers (not necessarily contiguous in the array). O(n) time.

**What interviewer is testing:**
Achieving O(n) with a hashset — and the key trick of only starting a count from sequence **beginnings**.

**Ideal Answer:**

Put all numbers in a set. For each number, only begin counting if `num - 1` is **not** in the set (i.e., it's the start of a run). Then walk `num, num+1, ...` while present. Because each number is visited at most twice (once on the scan, once during a run), it's O(n) despite the nested loop.

```java
public int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int x : nums) set.add(x);
    int best = 0;
    for (int x : set) {
        if (!set.contains(x - 1)) {       // x is a sequence start
            int len = 1, cur = x;
            while (set.contains(cur + 1)) { cur++; len++; }
            best = Math.max(best, len);
        }
    }
    return best;
}
```

**Dry run** — `[100,4,200,1,3,2]`: starts are 100,200,1; from 1 → 1,2,3,4 len 4. → 4. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why is it O(n) despite the while loop?"* → Inner walk only runs from starts; total walked steps ≤ n.
2. *"Sorting approach?"* → O(n log n); simpler but slower — mention then beat it.

**Common mistakes:**
- Counting from every element (without the `x-1` check) → O(n²).
- Iterating the array instead of the set (duplicates cause redundant work).

---

# STRINGS

Strings in Java are immutable — every concat creates a new object. Use `StringBuilder` in loops. The dominant string patterns are **sliding window** (substring problems), **two pointers** (palindromes), **frequency counting** (anagrams), and **DP** (edit distance, decode ways).

---

### Q34: Valid Anagram
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Are two strings anagrams of each other?

**Ideal Answer:**

Count character frequencies; they must match. For lowercase letters, a 26-int array beats a hashmap.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    for (int c : count) if (c != 0) return false;
    return true;
}
```

**Complexity:** O(n) time, O(1) space (fixed 26).

**Follow-ups:**
1. *"Unicode?"* → Use a `HashMap<Character,Integer>` (or code-point map).
2. *"Sort approach?"* → O(n log n); simpler, mention then beat with counting.

**Common mistakes:** Forgetting the length check; assuming ASCII when Unicode is in play.

---

### Q35: Group Anagrams
**Company:** Amazon, Google, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Group strings that are anagrams of each other.

**What interviewer is testing:**
Designing a canonical **key** for a group (sorted string, or frequency signature).

**Ideal Answer:**

All anagrams share a canonical form. Two key choices:
- Sorted characters: `"eat" -> "aet"`. O(k log k) per word.
- Frequency signature: `"a1b0c0..."`. O(k) per word — faster for long words over a small alphabet.

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] c = s.toCharArray();
        Arrays.sort(c);
        String key = new String(c);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
```

Frequency-key variant (no sorting):
```java
String key() { /* build "1#0#0#..#" from int[26] counts */ }
```

**Complexity:** Sorted key O(n·k log k); freq key O(n·k). Space O(n·k).

**Follow-up:** *"Which key is better?"* → Frequency key for long strings over small alphabets; sorted key is simpler.

**Common mistakes:** Using the original string as key; not handling empty strings.

---

### Q36: Longest Substring Without Repeating Characters
**Company:** Amazon, Google, Meta, Microsoft, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Length of the longest substring with all distinct characters.

**What interviewer is testing:**
The **variable-size sliding window** template — the single most-asked string pattern.

**Ideal Answer:**

Expand a window with the right pointer. Keep a map `char -> last index`. When you hit a repeat *inside* the window, jump the left pointer to just past the previous occurrence. Track max window size.

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> last = new HashMap<>(); // char -> last seen index
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (last.containsKey(c) && last.get(c) >= left) {
            left = last.get(c) + 1;       // shrink window past the duplicate
        }
        last.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Dry run** — `"abcabcbb"`: window grows abc (3); at second a, left jumps to 1; ... max stays 3. ✅

**Complexity:** O(n) time, O(min(n, charset)) space.

**Follow-ups:**
1. *"At most K distinct characters?"* → Generalized sliding window with a frequency map; shrink when distinct > K.
2. *"Return the substring itself?"* → Track the best window's start.

**Common mistakes:**
- Moving `left` backward (use `last.get(c) >= left` guard — the previous occurrence might be outside the current window).
- Off-by-one in window length (`right - left + 1`).

---

### Q37: Longest Palindromic Substring
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the longest palindromic substring.

**What interviewer is testing:**
Expand-from-center (O(n²)); Manacher's for the ⚫ variant.

**Ideal Answer:**

A palindrome mirrors around a center. There are `2n−1` centers (each char, and each gap between chars). Expand outward from each center while characters match; track the longest.

```java
public String longestPalindrome(String s) {
    if (s.length() < 2) return s;
    int start = 0, maxLen = 1;
    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);     // odd-length center
        int len2 = expand(s, i, i + 1); // even-length center
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = i - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}
private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;   // length of palindrome
}
```

**Complexity:** O(n²) time, O(1) space. Manacher's is O(n) but rarely required.

**Follow-ups:**
1. *"O(n) solution?"* → Manacher's algorithm; describe transform with separators and the radius array.
2. *"Count all palindromic substrings?"* → Same expansion, count each successful step.

**Common mistakes:** Forgetting even-length centers; off-by-one when recovering `start` from length.

---

### Q38: Valid Palindrome
**Company:** Meta, Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Is the string a palindrome considering only alphanumerics, ignoring case?

**Ideal Answer:**

Two pointers from both ends, skipping non-alphanumerics, comparing case-insensitively.

```java
public boolean isPalindrome(String s) {
    int lo = 0, hi = s.length() - 1;
    while (lo < hi) {
        while (lo < hi && !Character.isLetterOrDigit(s.charAt(lo))) lo++;
        while (lo < hi && !Character.isLetterOrDigit(s.charAt(hi))) hi--;
        if (Character.toLowerCase(s.charAt(lo)) != Character.toLowerCase(s.charAt(hi)))
            return false;
        lo++; hi--;
    }
    return true;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** Q39 — allow one deletion.

**Common mistakes:** Building a cleaned copy (O(n) space) when two pointers suffice; forgetting case folding.

---

### Q39: Valid Palindrome II
**Company:** Meta, Amazon
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Can the string become a palindrome by deleting **at most one** character?

**What interviewer is testing:**
Two pointers with a single branch on mismatch.

**Ideal Answer:**

Move inward; on the first mismatch, you may skip either the left or the right character — check if either resulting substring is a palindrome.

```java
public boolean validPalindrome(String s) {
    int lo = 0, hi = s.length() - 1;
    while (lo < hi) {
        if (s.charAt(lo) != s.charAt(hi)) {
            return isPali(s, lo + 1, hi) || isPali(s, lo, hi - 1);
        }
        lo++; hi--;
    }
    return true;
}
private boolean isPali(String s, int lo, int hi) {
    while (lo < hi) {
        if (s.charAt(lo++) != s.charAt(hi--)) return false;
    }
    return true;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"At most K deletions?"* → DP on longest palindromic subsequence; check `n - LPS <= k`.

**Common mistakes:** Only trying one side of the deletion.

---

### Q40: String to Integer (atoi)
**Company:** Amazon, Microsoft, Google, Goldman Sachs
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Implement `atoi`: skip leading whitespace, optional sign, read digits, stop at first non-digit, clamp to int range.

**What interviewer is testing:**
Edge-case discipline and **overflow handling** before it happens.

**Ideal Answer:**

Process in order: whitespace → sign → digits, clamping on overflow *during* accumulation (check before multiplying by 10).

```java
public int myAtoi(String s) {
    int i = 0, n = s.length();
    while (i < n && s.charAt(i) == ' ') i++;           // 1. whitespace
    if (i == n) return 0;
    int sign = 1;
    if (s.charAt(i) == '+' || s.charAt(i) == '-') {    // 2. sign
        if (s.charAt(i) == '-') sign = -1;
        i++;
    }
    int result = 0;
    while (i < n && Character.isDigit(s.charAt(i))) {   // 3. digits
        int d = s.charAt(i) - '0';
        // overflow check BEFORE updating
        if (result > (Integer.MAX_VALUE - d) / 10) {
            return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
        }
        result = result * 10 + d;
        i++;
    }
    return result * sign;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why check overflow before multiply?"* → After overflow the value is already corrupted; you can't detect it reliably afterward.
2. *"Why does MIN_VALUE clamp work?"* → We accumulate as positive and apply sign; the boundary case `−2147483648` is covered by returning MIN_VALUE on overflow.

**Common mistakes:** Detecting overflow after it occurs; mishandling lone `+`/`-`; not stopping at the first non-digit.

---

### Q41: Longest Common Prefix
**Company:** Amazon, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Find the longest common prefix among an array of strings.

**Ideal Answer:**

Vertical scan: compare character `j` across all strings; stop at the first mismatch or end of any string.

```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    for (int j = 0; j < strs[0].length(); j++) {
        char c = strs[0].charAt(j);
        for (int i = 1; i < strs.length; i++) {
            if (j == strs[i].length() || strs[i].charAt(j) != c) {
                return strs[0].substring(0, j);
            }
        }
    }
    return strs[0];
}
```

**Complexity:** O(S) where S = total characters; O(1) space.

**Follow-up:** *"Many prefix queries?"* → Build a Trie once, then each query is O(query length).

**Common mistakes:** Index out of bounds when one string is shorter (the `j == length` check).

---

### Q42: Minimum Window Substring
**Company:** Amazon, Google, Meta, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Smallest substring of `s` containing all characters of `t` (with multiplicity). Empty string if none.

**What interviewer is testing:**
The **sliding window with a need-count** template at its hardest. This is the canonical hard sliding-window.

**Ideal Answer:**

Count needed chars from `t`. Expand `right`, decrementing needs; when a char's need crosses from >0 to 0 you've satisfied one required char (`formed++`). Once all are formed, shrink `left` to minimize while still valid, recording the best window.

```java
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    int required = t.length();             // total chars still needed (with multiplicity)
    int left = 0, bestLen = Integer.MAX_VALUE, bestStart = 0;
    for (int right = 0; right < s.length(); right++) {
        if (need[s.charAt(right)]-- > 0) required--; // consumed a needed char
        while (required == 0) {                      // valid window -> try to shrink
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }
            if (need[s.charAt(left)]++ == 0) required++; // releasing a needed char
            left++;
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}
```

**Why the `need[]` can go negative:** extra copies of a char in `s` drive the count below 0; only when releasing a char brings it back to exactly 0 do we re-require it. This elegantly tracks multiplicity.

**Complexity:** O(|s| + |t|) time, O(1) space (128 fixed).

**Follow-ups:**
1. *"Window must contain t as a contiguous match?"* → That's substring search (KMP, Q47), different problem.
2. *"Generalize the template."* → expand-until-valid, then shrink-while-valid; reused in dozens of problems.

**Common mistakes:**
- Recomputing window validity from scratch each step (O(n·m)).
- Mismanaging `required` increments/decrements (the `> 0` / `== 0` guards are the crux).

---

### Q43: Decode Ways
**Company:** Amazon, Meta, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A→1 … Z→26. Count how many ways a digit string can be decoded. `"226"` → 3 ("BZ","VF","BBF").

**What interviewer is testing:**
1D DP on a string with tricky zero handling.

**Ideal Answer:**

`dp[i]` = ways to decode the first `i` chars. `dp[i]` adds `dp[i-1]` if the single digit `s[i-1]` is valid (1–9), and `dp[i-2]` if the two-digit `s[i-2..i-1]` is in 10–26. Space-optimize to two variables.

```java
public int numDecodings(String s) {
    if (s.isEmpty() || s.charAt(0) == '0') return 0;
    int prev2 = 1;          // dp[0]
    int prev1 = 1;          // dp[1]
    for (int i = 2; i <= s.length(); i++) {
        int cur = 0;
        char one = s.charAt(i - 1);
        char ten = s.charAt(i - 2);
        if (one != '0') cur += prev1;                         // single digit
        int two = (ten - '0') * 10 + (one - '0');
        if (two >= 10 && two <= 26) cur += prev2;             // two digits
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Dry run** — `"226"`: dp[0]=1,dp[1]=1; i2 "2","22"→cur=1+1=2; i3 "6","26"→cur=2+1=3 → 3. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Leading or embedded zeros like `"06"`?"* → A standalone 0 or invalid pair ("30") yields 0 ways for that path.
2. *"Decode Ways II with `*` wildcards?"* → Multiply counts by valid digit choices.

**Common mistakes:** Mishandling `'0'` (e.g., "10","20" valid only as pairs; "30" invalid).

---

### Q44: Edit Distance
**Company:** Google, Amazon, Microsoft, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Minimum operations (insert, delete, replace) to convert word1 into word2.

**What interviewer is testing:**
The classic 2D string DP — defining state, recurrence, and the table.

**Ideal Answer:**

`dp[i][j]` = edit distance between `word1[0..i)` and `word2[0..j)`.
- If last chars match: `dp[i][j] = dp[i-1][j-1]` (no op).
- Else `1 + min(dp[i-1][j]` (delete), `dp[i][j-1]` (insert), `dp[i-1][j-1]` (replace)`)`.
- Base: converting to/from empty string costs the other's length.

```java
public int minDistance(String w1, String w2) {
    int m = w1.length(), n = w2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;   // delete all
    for (int j = 0; j <= n; j++) dp[0][j] = j;   // insert all
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (w1.charAt(i - 1) == w2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],
                            Math.min(dp[i - 1][j], dp[i][j - 1]));
            }
        }
    }
    return dp[m][n];
}
```

**Table for "horse"→"ros":** answer 3 (replace h→r, replace o→o keep, delete r, ... → 3). ✅

**Complexity:** O(m·n) time, O(m·n) space (reducible to O(min(m,n)) with rolling rows).

**Follow-ups:**
1. *"Space optimize to 1D?"* → Keep previous row; save the diagonal in a temp.
2. *"Reconstruct the operations?"* → Backtrack through the table.
3. *"Only insert/delete allowed?"* → That's `m + n − 2·LCS`.

**Common mistakes:** Wrong base cases; mixing up which neighbor is insert vs delete.

---

### Q45: Count and Say
**Company:** Amazon, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
The sequence: 1, 11, 21, 1211, 111221, ... Each term describes the previous (run-length encoding). Return the nth term.

**Ideal Answer:**

Iterate, run-length encoding the previous string each step.

```java
public String countAndSay(int n) {
    String s = "1";
    for (int i = 1; i < n; i++) {
        StringBuilder sb = new StringBuilder();
        int j = 0;
        while (j < s.length()) {
            char c = s.charAt(j);
            int count = 0;
            while (j < s.length() && s.charAt(j) == c) { count++; j++; }
            sb.append(count).append(c);
        }
        s = sb.toString();
    }
    return s;
}
```

**Complexity:** O(n · L) where L is the length of the longest term (grows ~1.3ⁿ).

**Common mistakes:** Off-by-one (start `s="1"` then iterate n−1 times).

---

### Q46: String Compression
**Company:** Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Compress in place: `["a","a","b","b","c","c","c"]` → `["a","2","b","2","c","3"]`, return new length. Single chars stay uncompressed (no "1").

**What interviewer is testing:**
In-place two-pointer with a read pointer and a write pointer.

**Ideal Answer:**

`read` scans runs; `write` emits the char then the count digits (only if count > 1).

```java
public int compress(char[] chars) {
    int write = 0, read = 0, n = chars.length;
    while (read < n) {
        char c = chars[read];
        int count = 0;
        while (read < n && chars[read] == c) { read++; count++; }
        chars[write++] = c;
        if (count > 1) {
            for (char digit : Integer.toString(count).toCharArray()) {
                chars[write++] = digit;
            }
        }
    }
    return write;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Counts > 9?"* → The `Integer.toString` writes multiple digit chars (handled).

**Common mistakes:** Writing "1" for singletons; assuming single-digit counts.

---

### Q47: Implement strStr() / KMP
**Company:** Google, Amazon, Microsoft
**Difficulty:** 🟡 Medium (KMP variant 🔴)
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return the index of the first occurrence of `needle` in `haystack`, or −1.

**What interviewer is testing:**
Whether you know the O(n+m) KMP and can explain the failure function — not just the O(nm) brute force.

**Ideal Answer:**

*Brute force:* try each start, compare. O(n·m).

*KMP (optimal):* precompute the **LPS** array (longest proper prefix that is also a suffix) for the needle. On mismatch, instead of restarting, jump using `lps` so we never re-scan matched text. O(n+m).

```java
public int strStr(String haystack, String needle) {
    if (needle.isEmpty()) return 0;
    int[] lps = buildLps(needle);
    int i = 0, j = 0;                  // i over haystack, j over needle
    while (i < haystack.length()) {
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++; j++;
            if (j == needle.length()) return i - j;
        } else if (j > 0) {
            j = lps[j - 1];            // fall back without moving i
        } else {
            i++;
        }
    }
    return -1;
}
private int[] buildLps(String p) {
    int[] lps = new int[p.length()];
    int len = 0, i = 1;
    while (i < p.length()) {
        if (p.charAt(i) == p.charAt(len)) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1];        // try a shorter prefix
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

**LPS intuition:** `lps[i]` tells how much of the matched portion is also a prefix, so on mismatch we resume from there instead of from 0.

**Complexity:** O(n + m) time, O(m) space.

**Follow-ups:**
1. *"Rabin–Karp?"* → Rolling hash, O(n+m) average but O(nm) worst; great for *multiple* pattern search.
2. *"When KMP vs Rabin–Karp?"* → KMP for single pattern guaranteed linear; Rabin–Karp for many patterns or 2D search.

**Common mistakes:** Off-by-one in LPS construction; resetting `i` on mismatch (KMP's whole point is not to).

---

### Q48: Longest Repeating Character Replacement
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
You may replace at most `k` characters. Find the longest substring of a single repeated character achievable.

**What interviewer is testing:**
Sliding window where validity = `windowLength − maxFreq ≤ k`.

**Ideal Answer:**

For a window to be fixable with ≤ k replacements, the number of non-majority chars (`windowLen − maxFreqInWindow`) must be ≤ k. Expand right; if invalid, shrink left by one. A subtle but accepted optimization: `maxFreq` need not shrink — the answer only grows, so the window never shrinks below the best.

```java
public int characterReplacement(String s, int k) {
    int[] count = new int[26];
    int left = 0, maxFreq = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        maxFreq = Math.max(maxFreq, ++count[s.charAt(right) - 'A']);
        while (right - left + 1 - maxFreq > k) {   // too many replacements needed
            count[s.charAt(left) - 'A']--;
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Why not recompute maxFreq when shrinking?"* → The window length never decreases past the best; a stale (too-high) maxFreq only over-allows, but since `best` already captured a valid larger window, correctness holds. Be ready to argue this — it's the trickiest part.

**Common mistakes:** Using `if` to shrink fully (correct too) vs `while` — both fine; the conceptual trap is thinking maxFreq must be exact.

---

### Q49: Word Break
**Company:** Amazon, Google, Meta, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Can `s` be segmented into a space-separated sequence of words from a dictionary?

**What interviewer is testing:**
1D DP over string positions (or memoized DFS).

**Ideal Answer:**

`dp[i]` = can `s[0..i)` be segmented. `dp[i]` is true if there's a `j < i` with `dp[j]` true and `s[j..i)` in the dictionary.

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true;                       // empty prefix
    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[s.length()];
}
```

**Dry run** — `s="leetcode"`, dict {leet, code}: dp[4] (leet) true; dp[8] from j=4 "code" → true. ✅

**Complexity:** O(n²) substrings × O(n) substring/hash = O(n³) worst (or O(n²·L) bounding word length); O(n) space.

**Follow-ups:**
1. *"Return all sentences (Word Break II)?"* → Backtracking with memoization mapping start → list of valid suffixes.
2. *"Optimize substring checks?"* → Bound inner loop by max word length; use a Trie.

**Common mistakes:** Forgetting `dp[0] = true`; pure recursion without memo → exponential TLE.

---

# HASHING

Hashing turns "have I seen this?" and "how many of these?" into O(1). The recurring designs: **frequency maps**, **prefix-sum maps** (Q31, Q55), **value→index maps** (Q1), and **hashmap + auxiliary structure** (LRU = map + DLL; GetRandom = map + array).

---

### Q50: Design HashMap
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Implement a hashmap from scratch (`put`, `get`, `remove`) without using built-in map libraries.

**What interviewer is testing:**
Understanding of hashmap internals: buckets, hashing, collision via chaining.

**Ideal Answer:**

Array of buckets; each bucket is a linked list (separate chaining). Hash the key to a bucket index; on collision, search/append within the chain.

```java
class MyHashMap {
    private static class Node {
        int key, value;
        Node next;
        Node(int k, int v) { key = k; value = v; }
    }
    private final Node[] buckets;
    private static final int SIZE = 769; // prime reduces clustering

    public MyHashMap() { buckets = new Node[SIZE]; }

    private int hash(int key) { return Integer.hashCode(key) % SIZE; }

    public void put(int key, int value) {
        int i = hash(key);
        if (buckets[i] == null) { buckets[i] = new Node(key, value); return; }
        Node cur = buckets[i];
        while (true) {
            if (cur.key == key) { cur.value = value; return; }
            if (cur.next == null) break;
            cur = cur.next;
        }
        cur.next = new Node(key, value);
    }

    public int get(int key) {
        for (Node cur = buckets[hash(key)]; cur != null; cur = cur.next) {
            if (cur.key == key) return cur.value;
        }
        return -1;
    }

    public void remove(int key) {
        int i = hash(key);
        Node cur = buckets[i], prev = null;
        while (cur != null) {
            if (cur.key == key) {
                if (prev == null) buckets[i] = cur.next;
                else prev.next = cur.next;
                return;
            }
            prev = cur; cur = cur.next;
        }
    }
}
```

**Complexity:** O(1) average per op, O(n/buckets) worst.

**Follow-ups:**
1. *"Dynamic resizing / load factor?"* → Track size; when `size/buckets > 0.75`, double buckets and rehash. (Java's HashMap does this and treeifies long chains to red-black trees at threshold 8.)
2. *"Why a prime bucket count?"* → Reduces collision clustering for poorly distributed hashes.

**Common mistakes:** Updating value by inserting a duplicate node instead of overwriting; not handling head removal.

---

### Q51: Top K Frequent Elements
**Company:** Amazon, Meta, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the `k` most frequent elements.

**What interviewer is testing:**
Frequency map + selection: heap O(n log k) or **bucket sort** O(n).

**Ideal Answer:**

Count frequencies. Then either:
- **Min-heap of size k**: keep the k highest frequencies. O(n log k).
- **Bucket sort**: index buckets by frequency (0..n), collect from the high end. O(n).

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int x : nums) freq.merge(x, 1, Integer::sum);

    // bucket[f] = list of numbers with frequency f
    List<Integer>[] buckets = new List[nums.length + 1];
    for (var e : freq.entrySet()) {
        int f = e.getValue();
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(e.getKey());
    }
    int[] res = new int[k];
    int idx = 0;
    for (int f = buckets.length - 1; f >= 0 && idx < k; f--) {
        if (buckets[f] != null) {
            for (int num : buckets[f]) {
                if (idx < k) res[idx++] = num;
            }
        }
    }
    return res;
}
```

**Complexity:** Bucket sort O(n) time, O(n) space. Heap variant O(n log k).

**Follow-ups:**
1. *"k close to n — which is better?"* → Bucket sort stays O(n); heap degrades.
2. *"Streaming data?"* → Min-heap of size k (you can't bucket an unbounded stream's max freq easily).
3. *"QuickSelect on frequencies?"* → O(n) average, in-place, but trickier.

**Common mistakes:** Sorting the whole frequency map O(n log n) when O(n) is achievable; off-by-one on bucket size (`n+1`).

---

### Q52: LRU Cache
**Company:** Amazon, Google, Microsoft, Meta, Uber — the most-asked design coding question
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Design a cache with `get` and `put` both in O(1), evicting the least-recently-used entry when capacity is exceeded.

**What interviewer is testing:**
HashMap + doubly linked list combination. O(1) on both ops *requires* both structures: the map for lookup, the DLL for ordering with O(1) move/remove.

**Ideal Answer:**

- **HashMap** `key -> node` for O(1) lookup.
- **Doubly linked list** ordered by recency: most-recent near `head`, least-recent near `tail`. Move to front on access; evict from tail on overflow.
- Use dummy head/tail sentinels to avoid null checks.

```java
class LRUCache {
    private static class Node {
        int key, value;
        Node prev, next;
        Node(int k, int v) { key = k; value = v; }
    }
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // dummy
    private final Node tail = new Node(0, 0); // dummy

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    private void remove(Node n) {
        n.prev.next = n.next;
        n.next.prev = n.prev;
    }
    private void addFront(Node n) {          // right after head = most recent
        n.next = head.next;
        n.prev = head;
        head.next.prev = n;
        head.next = n;
    }

    public int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        remove(n);
        addFront(n);                          // mark as recently used
        return n.value;
    }

    public void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) {
            n.value = value;
            remove(n);
            addFront(n);
            return;
        }
        if (map.size() == capacity) {         // evict LRU (node before tail)
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node node = new Node(key, value);
        map.put(key, node);
        addFront(node);
    }
}
```

**Why both structures:** The map alone can't track order in O(1); a list alone can't look up in O(1). Together each is O(1).

**Complexity:** O(1) `get`/`put`, O(capacity) space.

**Follow-ups:**
1. *"Thread-safe version?"* → Synchronize, or use `ConcurrentHashMap` + locking; mention `LinkedHashMap(accessOrder=true)` as the built-in shortcut.
2. *"LFU cache?"* → Frequency buckets, each a DLL; harder (separate problem).
3. *"Why not `LinkedHashMap`?"* → In an interview they want the manual DLL to prove you understand it; in production, `LinkedHashMap` with `removeEldestEntry` is fine.

**Common mistakes:**
- Forgetting to update recency on `get` (not just `put`).
- Not removing the evicted key from the map.
- Null-pointer bugs without sentinel nodes.

---

### Q53: First Unique Character in a String / Stream
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Return the index of the first non-repeating character in a string. Variant: a *stream* of characters — report the first unique at any time.

**Ideal Answer:**

*String:* two passes — count, then scan for the first with count 1.

```java
public int firstUniqChar(String s) {
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (int i = 0; i < s.length(); i++) {
        if (count[s.charAt(i) - 'a'] == 1) return i;
    }
    return -1;
}
```

*Stream:* keep counts plus a queue of candidate chars in arrival order; pop from the front while the front's count > 1.

```java
class FirstUnique {
    private final int[] count = new int[128];
    private final Deque<Character> q = new ArrayDeque<>();
    public void add(char c) {
        count[c]++;
        q.offerLast(c);
        while (!q.isEmpty() && count[q.peekFirst()] > 1) q.pollFirst();
    }
    public char firstUnique() { return q.isEmpty() ? '#' : q.peekFirst(); }
}
```

**Complexity:** String O(n)/O(1). Stream O(1) amortized per char.

**Common mistakes:** Re-scanning the whole stream each query (O(n) per query) instead of the queue.

---

### Q54: Isomorphic Strings
**Company:** Amazon, LinkedIn
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Are `s` and `t` isomorphic — a consistent **one-to-one** char mapping turns one into the other?

**What interviewer is testing:**
Bijection requires mapping in **both** directions.

**Ideal Answer:**

Two maps (or two index arrays): `s→t` and `t→s` must be consistent. A common slick trick: compare the index of the last occurrence of each character.

```java
public boolean isIsomorphic(String s, String t) {
    int[] mapS = new int[256], mapT = new int[256];
    for (int i = 0; i < s.length(); i++) {
        char a = s.charAt(i), b = t.charAt(i);
        if (mapS[a] != mapT[b]) return false;  // mismatch in last-seen position
        mapS[a] = i + 1;                        // store i+1 (0 = unseen)
        mapT[b] = i + 1;
    }
    return true;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Word pattern (pattern of words)?"* → Same bijection idea with a `char↔word` map.

**Common mistakes:** Mapping only one direction (e.g., "ab"→"aa" wrongly passes).

---

### Q55: Count Subarrays With Given XOR
**Company:** Amazon, Flipkart, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Count subarrays whose XOR equals `k`.

**What interviewer is testing:**
Prefix-XOR + hashmap — the XOR analog of subarray-sum (Q31).

**Ideal Answer:**

Let `pre` be the running prefix XOR. A subarray `(j, i]` has XOR k iff `pre[i] ^ pre[j] = k`, i.e. `pre[j] = pre[i] ^ k`. Count earlier prefix XORs equal to `pre ^ k`.

```java
public int countSubarraysWithXor(int[] arr, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    freq.put(0, 1);
    int pre = 0, count = 0;
    for (int x : arr) {
        pre ^= x;
        count += freq.getOrDefault(pre ^ k, 0);
        freq.merge(pre, 1, Integer::sum);
    }
    return count;
}
```

**Dry run** — `[4,2,2,6,4]`, k=6: walk prefix XORs and accumulate; result 4. (The `pre ^ k` lookup mirrors the sum version's `sum − k`.) ✅

**Complexity:** O(n) time, O(n) space.

**Follow-up:** *"Why XOR uses `pre ^ k` instead of `pre − k`?"* → XOR is its own inverse: `a ^ b = k ⇒ b = a ^ k`.

**Common mistakes:** Forgetting the `{0:1}` seed; using subtraction instead of XOR.

---

### Q56: 4Sum II
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Four arrays A, B, C, D of length n. Count tuples `(i,j,k,l)` with `A[i]+B[j]+C[k]+D[l] == 0`.

**What interviewer is testing:**
Meet-in-the-middle hashing — split 4 sums into 2+2.

**Ideal Answer:**

Brute force is O(n⁴). Instead, hash all sums `A[i]+B[j]` into a map (count of each sum), then for every `C[k]+D[l]` add the count of its negation. O(n²).

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    Map<Integer, Integer> ab = new HashMap<>();
    for (int a : A) for (int b : B) ab.merge(a + b, 1, Integer::sum);
    int count = 0;
    for (int c : C) for (int d : D) count += ab.getOrDefault(-(c + d), 0);
    return count;
}
```

**Complexity:** O(n²) time, O(n²) space.

**Follow-up:** *"Six arrays?"* → Split 3+3, same idea.

**Common mistakes:** Trying to dedup (the problem counts index tuples, so duplicates count).

---

### Q57: Random Pick with Weight
**Company:** Amazon, Google, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `w[i]` weights, implement `pickIndex()` returning `i` with probability proportional to `w[i]`.

**What interviewer is testing:**
Prefix sums + binary search to map a uniform random into weighted buckets.

**Ideal Answer:**

Build prefix sums. Pick a uniform random in `[1, total]`; binary-search the first prefix ≥ it. Larger weights span larger ranges → picked more often.

```java
class Solution {
    private final int[] prefix;
    private final int total;
    private final Random rnd = new Random();

    public Solution(int[] w) {
        prefix = new int[w.length];
        int sum = 0;
        for (int i = 0; i < w.length; i++) { sum += w[i]; prefix[i] = sum; }
        total = sum;
    }

    public int pickIndex() {
        int target = rnd.nextInt(total) + 1;   // 1..total
        int lo = 0, hi = prefix.length - 1;
        while (lo < hi) {                        // first prefix >= target
            int mid = (lo + hi) >>> 1;
            if (prefix[mid] < target) lo = mid + 1;
            else hi = mid;
        }
        return lo;
    }
}
```

**Complexity:** Construct O(n); `pickIndex` O(log n); O(n) space.

**Follow-ups:**
1. *"Weights change dynamically?"* → Fenwick/Segment tree for O(log n) updates.
2. *"Why `nextInt(total)+1`?"* → Map to `[1,total]` so the search aligns with inclusive prefix sums.

**Common mistakes:** Off-by-one in the random range or the binary-search bound (must find the *first* prefix ≥ target).

---

### Q58: Encode and Decode TinyURL
**Company:** Amazon, Google, Bit.ly-style
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite / Design-lite

**Question:**
Design `encode(longUrl)` → short URL, and `decode(shortUrl)` → original.

**What interviewer is testing:**
Hash/ID design and the lookup map. Bridges into system design (Q in file 08).

**Ideal Answer:**

Map a generated short code to the long URL. Simplest reliable approach: an incrementing counter encoded in base-62, stored in a map both ways.

```java
class Codec {
    private final Map<String, String> codeToUrl = new HashMap<>();
    private final Map<String, String> urlToCode = new HashMap<>();
    private static final String BASE62 =
        "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private int counter = 1;
    private static final String DOMAIN = "http://tinyurl.com/";

    public String encode(String longUrl) {
        if (urlToCode.containsKey(longUrl)) return DOMAIN + urlToCode.get(longUrl);
        String code = toBase62(counter++);
        codeToUrl.put(code, longUrl);
        urlToCode.put(longUrl, code);
        return DOMAIN + code;
    }

    public String decode(String shortUrl) {
        String code = shortUrl.substring(DOMAIN.length());
        return codeToUrl.get(code);
    }

    private String toBase62(int n) {
        StringBuilder sb = new StringBuilder();
        while (n > 0) { sb.append(BASE62.charAt(n % 62)); n /= 62; }
        return sb.reverse().toString();
    }
}
```

**Complexity:** O(1) average per op.

**Follow-ups:**
1. *"Random codes vs counter?"* → Random avoids predictability/enumeration but needs collision checks; counter is dense and predictable.
2. *"Distributed system?"* → Snowflake-style IDs or a key-generation service (covered in system design file 08).
3. *"Collision handling for hash-based codes?"* → Re-hash with salt on collision.

**Common mistakes:** Using `longUrl.hashCode()` directly (collisions, negatives); not storing the reverse map for decode.

---

### Q59: Insert Delete GetRandom O(1)
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Design a set supporting `insert`, `remove`, and `getRandom` — all average O(1).

**What interviewer is testing:**
The **array + hashmap** combo. Array gives O(1) random access; map gives O(1) lookup; the trick is O(1) removal via swap-with-last.

**Ideal Answer:**

- A list/array holds the values (for O(1) random indexing).
- A map `value -> index in list` for O(1) lookup.
- **Remove in O(1):** swap the target with the last element, update its index in the map, then pop the last (so no shifting).

```java
class RandomizedSet {
    private final List<Integer> list = new ArrayList<>();
    private final Map<Integer, Integer> idx = new HashMap<>(); // value -> index
    private final Random rnd = new Random();

    public boolean insert(int val) {
        if (idx.containsKey(val)) return false;
        idx.put(val, list.size());
        list.add(val);
        return true;
    }

    public boolean remove(int val) {
        Integer i = idx.get(val);
        if (i == null) return false;
        int last = list.get(list.size() - 1);
        list.set(i, last);              // move last into the hole
        idx.put(last, i);
        list.remove(list.size() - 1);   // O(1) remove from the end
        idx.remove(val);
        return true;
    }

    public int getRandom() {
        return list.get(rnd.nextInt(list.size()));
    }
}
```

**Why swap-with-last:** removing from the middle of an ArrayList is O(n) due to shifting; swapping the doomed element with the last makes removal O(1).

**Complexity:** O(1) average all ops, O(n) space.

**Follow-ups:**
1. *"Allow duplicates (Insert Delete GetRandom — Duplicates allowed)?"* → Map value → **set of indices**; remove any one index with the same swap trick.
2. *"getRandom weighted?"* → That's Q57.

**Common mistakes:**
- `list.remove(i)` in the middle (O(n)) instead of swap-with-last.
- Forgetting to update the moved element's index in the map.

---

## Pattern recap — what to recognize on sight

| Trigger in problem statement | Pattern | Examples here |
|---|---|---|
| "pair/triplet summing to X", sorted | Two pointers | Q2, Q3, Q4, Q14 |
| "longest/shortest substring with property" | Sliding window | Q36, Q42, Q48 |
| "count subarrays with sum/xor = k", negatives allowed | Prefix sum/xor + hashmap | Q31, Q55 |
| "have I seen X / complement" | Hashset / hashmap | Q1, Q6, Q33 |
| "max/min in every window" | Monotonic deque | Q32 |
| "next greater / trapped water" | Two pointers / monotonic stack | Q15 |
| "in place, O(1) space, values in [1,n]" | Index-as-hash / cyclic sort | Q22, Q27 |
| "O(1) get + ordering / random" | Hashmap + auxiliary structure | Q52, Q59 |
| "convert/transform string optimally" | DP (1D/2D) | Q43, Q44, Q49 |
| "majority / cancellation" | Boyer–Moore voting | Q24 |

**The three questions to ask yourself on any array/string problem:**
1. Is it sorted (or can I sort)? → two pointers / binary search.
2. Am I asked about contiguous ranges with a sum/count property? → prefix sum + hashmap or sliding window (window only if all-positive / monotonic).
3. Do I need O(1) membership or frequency? → hashset / hashmap.

---

Created **01-dsa-arrays-strings-hashing.md** — 59 questions covered (33 arrays, 16 strings, 10 hashing), each with brute→optimal, complete Java, dry runs, complexity, follow-ups, and rejection-causing mistakes. Ready for next? (Next up: **02-dsa-linked-lists-stacks-queues.md**.)
