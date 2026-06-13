# 05 — DSA: Binary Search, Sorting & Greedy

> Three families that *look* easy and reject more candidates than anything else. Binary search is the highest-leverage pattern in interviews because it converts O(n) scans into O(log n) — and "binary search on the answer space" (Koko, Split Array, Ship Packages) is the single pattern most candidates fail to recognize. Sorting questions test whether you actually *understand* the algorithms (can you write merge/quick/heap sort from scratch, with correct partitioning?) rather than calling `Arrays.sort`. Greedy tests whether you can *prove* a local choice is globally optimal — the part 80% skip and get burned on. Every question here is **complete**: brute → optimal, full Java, dry run, complexity proof, follow-ups, and the mistakes that flip a "hire" to "no hire."

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the *real* signal. Knowing this changes how you talk while you code.
- **Ideal Answer** — approach reasoning, then production-quality Java, then complexity.
- **Follow-ups** — where most candidates fail. These are scripted in real loops.
- **Common mistakes** — the specific things that flip a "lean hire" to "no hire."

The meta-skill across this file: **recognize the invariant.** Binary search is correct only if you maintain a loop invariant ("the answer is always in `[lo, hi]`"). Greedy is correct only if you can name the exchange argument. Sorting is correct only if your partition/merge step is exact. Memorizing code without the invariant is how candidates write infinite loops on a whiteboard.

---

## Table of Contents

**Binary Search Templates (read first)**
- Template 1 — exact match (`lo <= hi`)
- Template 2 — boundary / first-or-last (`lo < hi`)
- Template 3 — search on the answer space (monotonic predicate)

**Binary Search**
1. Classic Binary Search
2. Find First and Last Position of Element
3. Search Insert Position
4. Search in Rotated Sorted Array I
5. Search in Rotated Sorted Array II (with duplicates)
6. Find Minimum in Rotated Sorted Array I
7. Find Minimum in Rotated Sorted Array II (with duplicates)
8. Search a 2D Matrix
9. Search a 2D Matrix II (staircase)
10. Find Peak Element
11. Single Element in a Sorted Array
12. Median of Two Sorted Arrays
13. Koko Eating Bananas (search on answer)
14. Capacity To Ship Packages Within D Days (search on answer)
15. Split Array Largest Sum (search on answer)
16. Allocate Minimum Number of Pages / Painter's Partition (search on answer)
17. Aggressive Cows / Magnetic Force Between Two Balls (search on answer)
18. Nth Root of a Number (precision binary search)
19. Find K-th Smallest Pair Distance
20. Kth Smallest Element in a Sorted Matrix
21. Sqrt(x)
22. Find the Smallest Divisor Given a Threshold

**Sorting**
23. Merge Sort (full implementation)
24. Quick Sort (full implementation + pivot strategy)
25. Heap Sort (full implementation)
26. QuickSelect — Kth Largest Element (O(n) average)
27. Count Inversions (modified merge sort)
28. Sort Colors / Dutch National Flag
29. Merge Sorted Array (in-place, gap method)
30. Custom Comparator Sorting (multi-criteria + overflow trap)
31. Largest Number (custom comparator)
32. Counting Sort & Radix Sort (when to use)
33. Pancake Sorting
34. Wiggle Sort II
35. Relative Sort Array
36. H-Index

**Greedy**
37. Non-overlapping Intervals (sort by end)
38. Minimum Number of Arrows to Burst Balloons
39. Job Sequencing with Deadlines
40. Fractional Knapsack
41. Minimum Number of Platforms
42. Candy (two pass)
43. Gas Station (circular)
44. Partition Labels
45. Queue Reconstruction by Height
46. Assign Cookies
47. Lemonade Change
48. Minimum Deletions to Make Character Frequencies Unique
49. Boats to Save People
50. Hand of Straights / Divide Array in Sets of K Consecutive Numbers
51. Maximum Units on a Truck
52. Two City Scheduling

---

# BINARY SEARCH TEMPLATES

> **This section is the most valuable part of the file.** Almost every binary search bug in an interview is one of: (a) an infinite loop from the wrong `lo`/`hi` update, (b) an off-by-one on the final answer, or (c) using the wrong template for the question. Learn these three templates, learn *when* each applies, and you will never freeze on a binary search problem again.

Binary search is not "search a sorted array." It is: **you have a search space `[lo, hi]` and a way to discard half of it in O(1) per step.** That's the only requirement. The array may not even be sorted (Find Peak Element). The space may be *values* not *indices* (Koko). What makes binary search applicable is a **monotonic predicate**: some boolean test `f(x)` that is `false, false, ..., false, true, true, ..., true` (or the reverse) across the search space. Binary search finds the boundary.

---

## Template 1 — Exact match (`lo <= hi`, search shrinks to empty)

Use when you want to find **a specific element** and return as soon as you find it (or report "not found"). The search space is `[lo, hi]` *inclusive on both ends*, and the loop continues while the range is non-empty.

```java
int binarySearchExact(int[] a, int target) {
    int lo = 0, hi = a.length - 1;        // inclusive bounds
    while (lo <= hi) {                     // <=  : single-element range is still valid
        int mid = lo + (hi - lo) / 2;      // avoid (lo+hi) overflow
        if (a[mid] == target) return mid;  // found
        else if (a[mid] < target) lo = mid + 1;  // discard left half INCLUDING mid
        else hi = mid - 1;                 // discard right half INCLUDING mid
    }
    return -1;                             // not found; lo is the insertion point
}
```

**Why `lo <= hi` and why `mid ± 1`?** Because we *exclude* `mid` after checking it (we already know `a[mid] != target`). The range strictly shrinks every iteration, so the loop always terminates. The single-element range `[lo, lo]` must still be checked, hence `<=`.

**When to use:** "Does X exist?", "return the index of X", Sqrt-style exact answers. Anytime you can definitively accept or reject `mid` and move past it.

**The classic infinite-loop trap:** writing `while (lo < hi)` with `lo = mid` (not `mid+1`). When `lo` and `hi` are adjacent, `mid == lo`, and `lo = mid` never advances. **Rule: if you ever assign `lo = mid` or `hi = mid` (not ±1), you MUST use Template 2's structure with care, or you loop forever.**

---

## Template 2 — Boundary / first-or-last occurrence (`lo < hi`, converge to one index)

Use when you want **the boundary** between "no" and "yes" — the first element satisfying a condition, the last one that doesn't, the leftmost/rightmost insertion point. Here you **keep `mid` as a candidate** rather than discarding it, so you cannot do `±1` on the side that keeps `mid`.

```java
// Find the FIRST index where condition(a[i]) is true.
// condition is monotonic: F F F T T T  -> we want the first T.
int firstTrue(int[] a) {
    int lo = 0, hi = a.length;            // hi = n (one past end) so "no T exists" => returns n
    while (lo < hi) {                      // <  : stop when lo == hi (converged)
        int mid = lo + (hi - lo) / 2;
        if (condition(a[mid])) hi = mid;   // mid MIGHT be the answer -> keep it (no -1)
        else lo = mid + 1;                 // mid is definitely not -> discard it
    }
    return lo;                             // lo == hi == first index where condition holds
}
```

**Why this never infinite-loops:** `mid = lo + (hi-lo)/2` rounds *down*, so `mid < hi` always. The branch `hi = mid` therefore strictly decreases `hi` (since `mid < hi`), and `lo = mid + 1` strictly increases `lo`. The range shrinks every step; loop ends at `lo == hi`.

**The mirror — last index where condition is true (T T T F F F, want last T):**

```java
int lastTrue(int[] a) {
    int lo = 0, hi = a.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;  // round UP — critical!
        if (condition(a[mid])) lo = mid;   // mid might be the answer -> keep it (no +1)
        else hi = mid - 1;
    }
    return lo;
}
```

**Why round up here?** When you do `lo = mid` (keeping mid on the lower side), if `mid` rounded *down* and `lo`/`hi` are adjacent, `mid == lo`, so `lo = mid` never moves → infinite loop. Rounding up with `(hi - lo + 1)/2` makes `mid == hi` in the adjacent case, so `lo = mid` advances. **Memory hook: the side that keeps `mid` (no ±1) dictates the rounding. Keep-on-low → round up. Keep-on-high → round down.**

**When to use:** "first/last occurrence", "smallest element ≥ x" (lower_bound), "largest element ≤ x", search-insert-position, peak/min in rotated arrays.

---

## Template 3 — Search on the answer space (the highest-leverage pattern)

**This is the pattern that confuses most candidates and the one interviewers love.** The trick: instead of searching an array of given values, you binary-search over the **range of possible answers**, using a feasibility function `boolean feasible(candidate)` that is **monotonic** in the candidate.

You recognize it when the problem says:
- *"minimize the maximum ..."* or *"maximize the minimum ..."*
- *"smallest X such that we can do the job in ≤ D units"* (Koko, Ship Packages, Split Array, Allocate Pages)
- *"largest X such that ..."* (Aggressive Cows — maximize the minimum distance)

The structure is **always** the same:

```java
int searchOnAnswer(...) {
    int lo = /* smallest possible answer */;
    int hi = /* largest possible answer  */;
    while (lo < hi) {                       // Template-2 boundary search
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;        // mid works -> maybe a smaller answer also works
        else lo = mid + 1;                  // mid too small -> need bigger
    }
    return lo;                              // smallest feasible answer
}
```

For **maximize-the-minimum** problems (Aggressive Cows), flip the rounding and the keep-side: you want the *largest* feasible candidate, so use the `lastTrue` mirror.

**Why it works — the monotonicity argument (say this out loud in the interview):** "If I can ship all packages in D days with capacity C, I can certainly do it with any capacity larger than C. So `feasible(C)` is monotonic: false for small C, true for large C. Binary search finds the boundary — the smallest C that works." Naming this proof is what separates a hire from a no-hire on these questions.

**The two hard parts candidates miss:**
1. **Defining `lo` and `hi` correctly.** For "minimize max load," `lo` = the largest single element (you can't go below it — one element must fit in one group), `hi` = the total sum (one group holds everything). Getting these wrong gives wrong answers or out-of-range feasibility checks.
2. **Writing `feasible` as a greedy O(n) sweep.** "Given capacity/speed/limit = mid, can I finish within the constraint?" — usually a single linear pass counting groups/days.

We'll hammer this pattern across Q13–Q17.

---

# BINARY SEARCH

---

### Q1: Classic Binary Search
**Company:** Amazon, Microsoft, Google — universal warm-up
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Given a sorted (ascending) array `nums` and a `target`, return its index, or `-1` if absent. O(log n).

**What interviewer is testing:**
Can you write a *bug-free* binary search? Specifically: the overflow-safe mid, the `<=` loop condition, and the `±1` updates. It's a trust check before harder variants.

**Ideal Answer:**

This is **Template 1** exactly. The invariant: if `target` exists, it is always within `[lo, hi]`.

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;          // overflow-safe midpoint
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

**Dry run** — `nums = [-1,0,3,5,9,12]`, target 9:
- lo=0,hi=5,mid=2 → 3<9 → lo=3
- lo=3,hi=5,mid=4 → 9==9 → return 4. ✅

**Complexity:** Time O(log n) — the range halves each step. Space O(1).

**Why `lo + (hi - lo) / 2` not `(lo + hi) / 2`?** For large indices `lo + hi` can overflow `int`. `lo + (hi - lo)/2` is mathematically identical but never overflows. Interviewers notice this.

**Follow-up questions the interviewer will ask:**
1. *"Make it recursive."* → Same logic; pass `lo, hi` down. Note the O(log n) stack space (vs O(1) iterative) — iterative is preferred.
2. *"What does `-1` represent vs the insertion point?"* → After the loop, `lo` is the index where `target` *would* be inserted to keep the array sorted (that's Q3).
3. *"Descending array?"* → Flip the comparison directions.

**Common mistakes that get you rejected:**
- `(lo + hi) / 2` overflow on large arrays.
- `while (lo < hi)` with `±1` updates — misses the single-element case, returns -1 for a present target.
- Infinite loop from `lo = mid` instead of `lo = mid + 1`.

---

### Q2: Find First and Last Position of Element in Sorted Array
**Company:** Amazon, Microsoft, Meta, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Sorted array with duplicates. Return `[first, last]` indices of `target`, or `[-1,-1]` if absent. O(log n).

**What interviewer is testing:**
**Template 2** (boundary search). Plain binary search finds *an* occurrence; finding the *boundaries* requires the leftmost/rightmost variant. This is the canonical "do you understand boundary binary search" question.

**Ideal Answer:**

Run two boundary searches. For the **first** position, find the leftmost index with `nums[i] >= target` (lower_bound); confirm it equals target. For the **last**, find the leftmost index with `nums[i] > target` and subtract one.

```java
public int[] searchRange(int[] nums, int target) {
    int first = lowerBound(nums, target);            // first index with nums[i] >= target
    if (first == nums.length || nums[first] != target) return new int[]{-1, -1};
    int last = lowerBound(nums, target + 1) - 1;      // index before first nums[i] > target
    return new int[]{first, last};
}

// smallest index i in [0, n] with nums[i] >= key  (returns n if none)
private int lowerBound(int[] nums, int key) {
    int lo = 0, hi = nums.length;                     // hi = n: allows "all smaller" -> returns n
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= key) hi = mid;               // mid is a candidate, keep it
        else lo = mid + 1;
    }
    return lo;
}
```

**Dry run** — `nums = [5,7,7,8,8,10]`, target 8:
- `lowerBound(8)`: converges to index 3 (`nums[3]=8`). first=3, nums[3]==8 ✓.
- `lowerBound(9) - 1`: lowerBound(9)=5 (first ≥9 is 10 at idx5), so last=4.
- Return `[3,4]`. ✅

**Complexity:** Time O(log n) (two boundary searches), Space O(1).

**Follow-ups:**
1. *"Count occurrences of target?"* → `last - first + 1` (or 0 if absent). This is a common variant.
2. *"Why `hi = n` not `n-1`?"* → So the "target larger than everything" case returns `n` cleanly without special-casing.
3. *"Do it with one search that finds left, then searches right within bounds?"* → Possible, but two `lowerBound` calls is the cleanest and most reusable.

**Common mistakes:**
- Finding one occurrence then linearly scanning outward → O(n) worst case (all duplicates) — fails the complexity bar.
- Off-by-one converting `lowerBound(target+1)` to the last index.
- Forgetting to verify `nums[first] == target` (lowerBound returns an insertion point even when absent).

---

### Q3: Search Insert Position
**Company:** Amazon, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Sorted distinct array. Return the index of `target`, or where it would be inserted to keep sorted order. O(log n).

**What interviewer is testing:**
That you recognize this is exactly `lowerBound` — the insertion point *is* the first index ≥ target.

**Ideal Answer:**

```java
public int searchInsert(int[] nums, int target) {
    int lo = 0, hi = nums.length;          // hi = n: insertion at the end is valid
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

**Dry run** — `[1,3,5,6]`, target 5 → returns 2; target 2 → returns 1; target 7 → returns 4. ✅

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
1. *"With duplicates, leftmost insert?"* → Same code (lower_bound). For rightmost, use `>` and upper_bound.
2. *"Why does `hi = n` work for end insertion?"* → The loop can converge to `n`, the valid "append" position.

**Common mistakes:** Using Template 1 and returning `-1`/`lo` inconsistently; `hi = n-1` which can't represent end-insertion.

---

### Q4: Search in Rotated Sorted Array I
**Company:** Amazon, Google, Microsoft, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
A sorted array of **distinct** values was rotated at an unknown pivot (e.g. `[4,5,6,7,0,1,2]`). Find `target`'s index, or -1. O(log n).

**What interviewer is testing:**
Adapting binary search when the array isn't fully sorted. The key insight: **at every step, at least one half (left of mid or right of mid) is sorted** — identify which, and decide whether `target` lies in that sorted half.

**Ideal Answer:**

Compare `nums[mid]` with `nums[lo]` to detect which half is sorted:
- If `nums[lo] <= nums[mid]`, the **left half is sorted**. If `target` is within `[nums[lo], nums[mid])`, go left; else go right.
- Otherwise the **right half is sorted**. If `target` is within `(nums[mid], nums[hi]]`, go right; else go left.

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        if (nums[lo] <= nums[mid]) {                  // left half [lo..mid] is sorted
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;  // target in left
            else lo = mid + 1;
        } else {                                      // right half [mid..hi] is sorted
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;  // target in right
            else hi = mid - 1;
        }
    }
    return -1;
}
```

**Dry run** — `[4,5,6,7,0,1,2]`, target 0:
- lo=0,hi=6,mid=3(7). Left `[4..7]` sorted; 0 not in `[4,7)` → lo=4.
- lo=4,hi=6,mid=5(1). Left `[0..1]` sorted (nums[4]=0 ≤ 1); 0 in `[0,1)` → hi=4.
- lo=4,hi=4,mid=4(0)==0 → return 4. ✅

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
1. *"With duplicates?"* → Q5; the `nums[lo] <= nums[mid]` test becomes ambiguous when equal, forcing an O(n) worst case.
2. *"Find the pivot first, then binary search each half?"* → Valid alternative (Q6 finds the min/pivot). Two log-n searches; the one-pass version above is cleaner.
3. *"Why compare with `nums[lo]` and not `nums[hi]`?"* → Either works if consistent; `nums[lo] <= nums[mid]` is the standard formulation. Be consistent or you'll mis-classify.

**Common mistakes:**
- Using `<` vs `<=` wrong in the sorted-half test → misses targets at boundaries.
- Comparing against `nums[hi]` to detect the sorted half but then checking ranges as if `nums[lo]` — mixing conventions.

---

### Q5: Search in Rotated Sorted Array II (with duplicates)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Same as Q4 but the array **may contain duplicates**. Return whether `target` exists (boolean).

**What interviewer is testing:**
Recognizing that duplicates **break** the "one half is always sorted" detection, and handling the ambiguous case — plus knowing the worst case degrades to O(n).

**Ideal Answer:**

When `nums[lo] == nums[mid] == nums[hi]` we can't tell which half is sorted (e.g. `[1,0,1,1,1]`). The fix: when `nums[lo] == nums[mid]` (the ambiguous boundary), just shrink by `lo++` and retry. Everything else mirrors Q4.

```java
public boolean search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return true;
        if (nums[lo] == nums[mid]) {            // ambiguous: can't classify, shrink
            lo++;
        } else if (nums[lo] < nums[mid]) {      // left half sorted
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else {                                // right half sorted
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return false;
}
```

**Dry run** — `[1,0,1,1,1]`, target 0:
- lo=0,hi=4,mid=2(1). nums[lo]=1==nums[mid]=1 → ambiguous → lo=1.
- lo=1,hi=4,mid=2(1). nums[lo]=0 < 1 → left `[0..1]` sorted; 0 in `[0,1)` → hi=1.
- lo=1,hi=1,mid=1(0)==0 → return true. ✅

**Complexity:** O(log n) average; **O(n) worst case** (all duplicates like `[1,1,1,1,1]`). Space O(1).

**Follow-ups:**
1. *"Why is worst case O(n)?"* → When all elements equal, every step only does `lo++`, scanning linearly. Unavoidable — duplicates destroy the information binary search needs.
2. *"Can we return the index instead of boolean?"* → Yes, but with duplicates the "first" index needs an extra boundary search.

**Common mistakes:**
- Claiming O(log n) worst case — wrong, and interviewers will push on it.
- Decrementing `hi--` instead of `lo++` in the ambiguous case — works too, but be consistent with your sorted-half convention.

---

### Q6: Find Minimum in Rotated Sorted Array I
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Rotated sorted array of **distinct** values. Find the minimum element. O(log n).

**What interviewer is testing:**
**Template 2** boundary search where you compare against the *right* end. The minimum is the unique "inflection point."

**Ideal Answer:**

Compare `nums[mid]` with `nums[hi]`:
- If `nums[mid] > nums[hi]`, the minimum must be **to the right** of mid (the rotation point is there) → `lo = mid + 1`.
- Else `nums[mid] <= nums[hi]`, so mid could be the min or the min is to its left → `hi = mid` (keep mid).

We compare with `hi`, not `lo`, because the right portion is the "small" side after rotation.

```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1;   // min is strictly right of mid
        else hi = mid;                            // mid might be the min
    }
    return nums[lo];                              // lo == hi == index of minimum
}
```

**Dry run** — `[4,5,6,7,0,1,2]`:
- lo=0,hi=6,mid=3(7) > nums[6]=2 → lo=4.
- lo=4,hi=6,mid=5(1) <= 2 → hi=5.
- lo=4,hi=5,mid=4(0) <= nums[5]=1 → hi=4.
- lo==hi==4 → nums[4]=0. ✅

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
1. *"Return the rotation count / pivot index?"* → It's exactly `lo` (the index of the min).
2. *"Why compare with `nums[hi]` not `nums[lo]`?"* → With `nums[lo]`, a non-rotated array `[1,2,3]` would mislead you (mid > lo always). Comparing with `hi` correctly handles the non-rotated case (returns index 0).
3. *"Find max instead?"* → It's the element just before the min, or use the mirror logic.

**Common mistakes:**
- Comparing against `nums[lo]` and breaking on non-rotated input.
- Using `hi = mid - 1` (could skip the actual minimum if mid is the min).

---

### Q7: Find Minimum in Rotated Sorted Array II (with duplicates)
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Same as Q6 but values may **repeat**. Find the minimum.

**What interviewer is testing:**
The duplicate-ambiguity fix again — when `nums[mid] == nums[hi]`, you can't tell which side holds the min, so shrink `hi` by one.

**Ideal Answer:**

```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1;       // min strictly to the right
        else if (nums[mid] < nums[hi]) hi = mid;       // min at mid or to the left
        else hi--;                                     // nums[mid] == nums[hi]: ambiguous, shrink
    }
    return nums[lo];
}
```

**Dry run** — `[2,2,2,0,1,2]`:
- lo=0,hi=5,mid=2(2)==nums[5]=2 → hi=4.
- lo=0,hi=4,mid=2(2) > nums[4]=1 → lo=3.
- lo=3,hi=4,mid=3(0) < nums[4]=1 → hi=3.
- lo==hi==3 → nums[3]=0. ✅

**Complexity:** O(log n) average, **O(n) worst case** (all equal). Space O(1).

**Follow-ups:**
1. *"Why `hi--` and not `lo++` in the tie?"* → Because we compare against `nums[hi]`; dropping `hi` is safe since `nums[hi]` is duplicated at `mid`, so we lose no information about the minimum. Doing `lo++` would be inconsistent with the `nums[hi]` comparison.
2. *"Worst case proof?"* → `[1,1,1,1,1]` forces `hi--` every step → O(n).

**Common mistakes:**
- Returning early or mishandling the tie → wrong answer on `[3,3,1,3]`.
- Claiming guaranteed O(log n).

---

### Q8: Search a 2D Matrix
**Company:** Amazon, Microsoft, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
`m × n` matrix where each row is sorted left-to-right **and** the first integer of each row is greater than the last integer of the previous row (i.e. the whole matrix is sorted if flattened). Find `target`. O(log(mn)).

**What interviewer is testing:**
Recognizing that the matrix is just a sorted 1D array in disguise — map a flat index to `(row, col)` and run one binary search.

**Ideal Answer:**

Treat the matrix as a virtual sorted array of length `m*n`. For flat index `k`: `row = k / n`, `col = k % n`. Binary search over `[0, m*n - 1]`.

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int lo = 0, hi = m * n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];     // map flat index to 2D
        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

**Dry run** — `[[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, target 3, n=4:
- lo=0,hi=11,mid=5 → matrix[1][1]=11 > 3 → hi=4.
- lo=0,hi=4,mid=2 → matrix[0][2]=5 > 3 → hi=1.
- lo=0,hi=1,mid=0 → matrix[0][0]=1 < 3 → lo=1.
- lo=1,hi=1,mid=1 → matrix[0][1]=3 → true. ✅

**Complexity:** O(log(mn)) time, O(1) space.

**Follow-ups:**
1. *"What if rows are sorted but the first-of-row > last-of-prev guarantee is dropped?"* → That's Q9 (Search a 2D Matrix II) — staircase, O(m+n).
2. *"Why `mid / n` and `mid % n`?"* → Row-major flattening; `n` is the column count.

**Common mistakes:**
- Using `mid / m` instead of `mid / n` (dividing by rows not columns).
- Doing a binary search per row → O(m log n), worse than O(log mn) and signals you missed the flattening.

---

### Q9: Search a 2D Matrix II (staircase search)
**Company:** Amazon, Google, Microsoft, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
`m × n` matrix sorted in ascending order **left-to-right in each row** and **top-to-bottom in each column**, but rows are *not* globally chained (the first of a row can be smaller than the last of the previous row). Find `target`. Better than O(mn).

**What interviewer is testing:**
The **staircase / monotonic elimination** technique. Not a binary search at all — it's a clever O(m+n) walk. Recognizing that one corner lets you discard a row or column each step.

**Ideal Answer:**

Start at the **top-right** corner. That element is the largest in its row and smallest in its column, so:
- If it `> target`, the entire column can't contain target (everything below is larger) → move left (`col--`).
- If it `< target`, the entire row can't contain target (everything left is smaller) → move down (`row++`).
- If equal, found.

Each step eliminates a full row or column → at most `m + n` steps.

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int row = 0, col = n - 1;                  // start top-right
    while (row < m && col >= 0) {
        int val = matrix[row][col];
        if (val == target) return true;
        else if (val > target) col--;          // eliminate this column
        else row++;                            // eliminate this row
    }
    return false;
}
```

**Dry run** — target 5 in `[[1,4,7,11],[2,5,8,12],[3,6,9,16]]`:
- (0,3)=11 > 5 → col=2.
- (0,2)=7 > 5 → col=1.
- (0,1)=4 < 5 → row=1.
- (1,1)=5 → true. ✅

**Complexity:** O(m + n) time, O(1) space.

**Follow-ups:**
1. *"Could you binary search each row?"* → O(m log n). Worse than O(m+n) when n is large, and doesn't exploit the column ordering. The staircase is the expected answer.
2. *"Bottom-left corner instead?"* → Symmetric: `> target` → up, `< target` → right. Top-right and bottom-left both work; top-left and bottom-right do **not** (both moves go the same direction).
3. *"Divide-and-conquer O(n log n)?"* → Exists (quad-partition around the middle), but the staircase is simpler and usually better.

**Common mistakes:**
- Starting from top-left/bottom-right (can't eliminate — both increase/decrease together).
- Treating it like Q8's flat binary search (the global-sort guarantee doesn't hold).

---

### Q10: Find Peak Element
**Company:** Google, Amazon, Meta, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
A peak is an element strictly greater than its neighbors. `nums[-1] = nums[n] = -∞`. Return the index of **any** peak. O(log n). Adjacent elements differ.

**What interviewer is testing:**
That binary search works on **unsorted** data when there's a monotonic *trend* to follow. You climb uphill; a peak must exist in the uphill direction.

**Ideal Answer:**

Compare `nums[mid]` with `nums[mid+1]`:
- If `nums[mid] < nums[mid+1]`, we're on an ascending slope; a peak lies to the **right** → `lo = mid + 1`.
- Else we're descending (or at a peak); a peak lies at mid or to the **left** → `hi = mid`.

Because the boundaries are `-∞`, following the upward slope always leads to a peak.

```java
public int findPeakElement(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < nums[mid + 1]) lo = mid + 1;   // ascending -> peak to the right
        else hi = mid;                                 // descending -> peak here or left
    }
    return lo;                                         // lo == hi == a peak index
}
```

**Dry run** — `[1,2,1,3,5,6,4]`:
- lo=0,hi=6,mid=3(3)<nums[4]=5 → lo=4.
- lo=4,hi=6,mid=5(6)>nums[6]=4 → hi=5.
- lo=4,hi=5,mid=4(5)<nums[5]=6 → lo=5.
- lo==hi==5 → index 5 (value 6). ✅

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
1. *"Why does a peak always exist?"* → If you keep moving uphill and the array ends (boundary = -∞), the last element you can't climb past is a peak. The slope argument guarantees it.
2. *"Find a peak in a 2D matrix?"* → Binary search on columns; find the global max of the mid column, compare with horizontal neighbors → O(n log m).
3. *"What if there are equal adjacent elements?"* → The problem usually forbids it; otherwise you may need a linear scan in plateaus.

**Common mistakes:**
- `mid + 1` out of bounds — using `hi = nums.length - 1` and `lo < hi` keeps `mid + 1` in range (mid < hi).
- Trying to require the array be sorted (it isn't — the trick is the slope, not sortedness).

---

### Q11: Single Element in a Sorted Array
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Sorted array where every element appears exactly twice except one. Find the single element. O(log n), O(1).

**What interviewer is testing:**
Recognizing a pairing-index invariant: before the single element, pairs start at even indices; after it, the pairing shifts. Binary search on that.

**Ideal Answer:**

Before the unique element, each pair occupies indices `(even, odd)` → `nums[even] == nums[even+1]`. After it, pairs start at odd indices. Force `mid` to be even; if `nums[mid] == nums[mid+1]`, the single element is to the right; else it's at mid or left.

```java
public int singleNonDuplicate(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid % 2 == 1) mid--;                  // align mid to an even index
        if (nums[mid] == nums[mid + 1]) lo = mid + 2;  // pair intact -> single is right
        else hi = mid;                            // pairing broken -> single at mid or left
    }
    return nums[lo];
}
```

**Dry run** — `[1,1,2,3,3,4,4,8,8]`:
- lo=0,hi=8,mid=4(odd→4 even). nums[4]=3,nums[5]=4 differ → hi=4.
- lo=0,hi=4,mid=2(even). nums[2]=2,nums[3]=3 differ → hi=2.
- lo=0,hi=2,mid=1→0. nums[0]=1==nums[1]=1 → lo=2.
- lo==hi==2 → nums[2]=2. ✅

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
1. *"XOR approach?"* → XOR everything → O(n) but not O(log n); interviewer wants the log-n binary search.
2. *"Why align mid to even?"* → So `nums[mid], nums[mid+1]` is a candidate pair; the parity check then tells you which side the break is on.

**Common mistakes:**
- Not aligning parity → the pair test becomes meaningless.
- Using `lo = mid + 1` instead of `mid + 2` after a confirmed pair (wastes the parity invariant, can break the logic).

---

### Q12: Median of Two Sorted Arrays
**Company:** Google, Amazon, Microsoft, Meta, Adobe
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Two sorted arrays `nums1` (size m) and `nums2` (size n). Return the median of the combined sorted array in **O(log(min(m, n)))**.

**What interviewer is testing:**
The hardest standard binary search. Can you binary-search a **partition** rather than an element? This is a senior-signal question; getting the partition invariant and the edge cases right is rare.

**Ideal Answer:**

*Brute force:* merge both (O(m+n)) and pick the middle. Mention it, then improve.

*Optimal — partition binary search:* The median splits the merged array into a **left half** and **right half** of equal size, where every element in the left ≤ every element in the right. We choose a cut in `nums1` (say `i` elements on the left) and the cut in `nums2` is forced: `j = (m + n + 1)/2 - i`, so the left half always has `⌈(m+n)/2⌉` elements.

We need:
- `maxLeft1 = nums1[i-1] ≤ nums2[j] = minRight2` and
- `maxLeft2 = nums2[j-1] ≤ nums1[i] = minRight1`.

Binary search `i` over `nums1` (the smaller array, so the range is `[0, m]` → O(log min)). Use `±∞` sentinels for the empty-side edges.

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    // Always binary-search over the SMALLER array to guarantee O(log(min(m,n)))
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);
    int m = nums1.length, n = nums2.length;
    int half = (m + n + 1) / 2;          // size of the combined left half
    int lo = 0, hi = m;                  // i can be 0..m (elements of nums1 in left half)

    while (lo <= hi) {
        int i = lo + (hi - lo) / 2;      // cut in nums1
        int j = half - i;                // cut in nums2 (forced)

        // Sentinels handle the cut being at an array boundary
        int maxLeft1  = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
        int minRight1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
        int maxLeft2  = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
        int minRight2 = (j == n) ? Integer.MAX_VALUE : nums2[j];

        if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {   // correct partition
            if ((m + n) % 2 == 1) {                            // odd total: median is max of left
                return Math.max(maxLeft1, maxLeft2);
            } else {                                           // even: average of the two middles
                int maxLeft  = Math.max(maxLeft1, maxLeft2);
                int minRight = Math.min(minRight1, minRight2);
                return (maxLeft + minRight) / 2.0;
            }
        } else if (maxLeft1 > minRight2) {                     // nums1's left too big -> move i left
            hi = i - 1;
        } else {                                               // nums1's left too small -> move i right
            lo = i + 1;
        }
    }
    throw new IllegalArgumentException("Input arrays are not sorted");
}
```

**Dry run** — `nums1 = [1,3]`, `nums2 = [2]`. m=2,n=1; swap so nums1=[2] (smaller), nums2=[1,3]. m=1,n=2, half=(3+1)/2=2.
- lo=0,hi=1,i=0,j=2. maxLeft1=-∞, minRight1=nums1[0]=2; maxLeft2=nums2[1]=3, minRight2=+∞. Check: maxLeft2=3 ≤ minRight1=2? No → maxLeft2 > minRight1, so `lo = i+1 = 1`.
- lo=1,hi=1,i=1,j=1. maxLeft1=nums1[0]=2, minRight1=+∞(i==m); maxLeft2=nums2[0]=1, minRight2=nums2[1]=3. Check: 2≤3 ✓ and 1≤+∞ ✓ → valid. Odd total → median = max(2,1) = 2. ✅

**Dry run 2 (even)** — `nums1=[1,2]`, `nums2=[3,4]`. m=n=2, half=2.
- i=1,j=1. maxLeft1=1,minRight1=2,maxLeft2=3,minRight2=4. 1≤4 ✓, 3≤2? No → maxLeft2>minRight1 → lo=2.
- i=2,j=0. maxLeft1=2,minRight1=+∞,maxLeft2=-∞,minRight2=3. 2≤3 ✓, -∞≤+∞ ✓ → valid. Even → max(2,-∞)=2, min(+∞,3)=3 → (2+3)/2 = 2.5. ✅

**Why O(log(min(m, n)))?** We binary search `i` over `[0, m]` where `m = min`. Each step is O(1). The `(m+n+1)/2` (round up) for `half` ensures odd totals put the extra element in the left half, so the median for odd is `max(maxLeft1, maxLeft2)`.

**Complexity:** Time O(log(min(m, n))), Space O(1).

**Follow-ups:**
1. *"Why search the smaller array?"* → `j = half - i` must stay in `[0, n]`; if we searched the larger array, `j` could go negative. Searching the smaller keeps `j` valid and minimizes log factor.
2. *"Why the `+1` in `half = (m+n+1)/2`?"* → Makes it work uniformly for odd and even: left half gets the extra element on odd totals, so we never index `minRight` for the odd median.
3. *"Generalize to the k-th element of two sorted arrays?"* → Classic "drop k/2 from one array" recursion, also O(log(m+n)). Good to mention as the cousin problem.
4. *"K sorted arrays' median?"* → Heap-based merge or binary search on value; much harder, usually out of scope.

**Common mistakes:**
- Not swapping to search the smaller array → `j` goes out of bounds.
- Missing the `±∞` sentinels for boundary cuts → ArrayIndexOutOfBounds on empty sides.
- Integer division `(maxLeft + minRight) / 2` instead of `/ 2.0` → wrong median for even totals.
- Off-by-one in `half` for odd totals.

---

### Q13: Koko Eating Bananas
**Company:** Amazon, Google, Meta, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
`piles[i]` bananas in pile `i`. Koko eats at speed `k` bananas/hour: each hour she picks one pile and eats `min(k, remaining)` from it (if a pile has fewer than `k`, she finishes it and that hour ends — she doesn't roll over to another pile that hour). Given `h` hours, return the **minimum** integer speed `k` so she finishes all bananas within `h` hours.

**What interviewer is testing:**
**Template 3 — binary search on the answer space.** This is the canonical introduction to the pattern. Can you (a) spot that the answer is a speed in a bounded range, (b) define a monotonic `feasible(k)`, and (c) binary-search the boundary?

**Ideal Answer:**

*Brute force:* try every speed `k = 1, 2, 3, ...` until one finishes within `h`. The hours to finish a pile at speed `k` is `ceil(pile / k)`. O(maxPile × n) — too slow.

*Key observation (monotonicity):* if speed `k` finishes within `h` hours, then any speed `> k` also finishes within `h` (faster never hurts). So `feasible(k)` is `false, false, ..., false, true, true, ...` — perfectly monotonic. **Binary search the smallest feasible `k`.**

- **Search space:** `lo = 1` (must eat at least 1/hr), `hi = max(piles)` (eating faster than the biggest pile gives no benefit — one pile per hour is the cap).
- **`feasible(k)`:** total hours `= Σ ceil(piles[i] / k)`; feasible iff `≤ h`.

```java
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = 0;
    for (int p : piles) hi = Math.max(hi, p);    // fastest useful speed = biggest pile

    while (lo < hi) {                            // Template 2: smallest feasible k
        int mid = lo + (hi - lo) / 2;
        if (hoursNeeded(piles, mid) <= h) hi = mid;   // mid works -> try slower
        else lo = mid + 1;                            // too slow -> need faster
    }
    return lo;                                   // smallest k that finishes within h
}

private long hoursNeeded(int[] piles, int speed) {
    long hours = 0;
    for (int p : piles) {
        hours += (p + speed - 1) / speed;        // ceil(p / speed) without floating point
    }
    return hours;
}
```

**Dry run** — `piles = [3,6,7,11]`, `h = 8`. lo=1, hi=11.
- mid=6: hours = ceil(3/6)+ceil(6/6)+ceil(7/6)+ceil(11/6) = 1+1+2+2 = 6 ≤ 8 → hi=6.
- lo=1,hi=6,mid=3: 1+2+3+4 = 10 > 8 → lo=4.
- lo=4,hi=6,mid=5: 1+2+2+3 = 8 ≤ 8 → hi=5.
- lo=4,hi=5,mid=4: 1+2+2+3 = 8 ≤ 8 → hi=4.
- lo==hi==4 → speed 4. ✅

**Complexity:** Time O(n log(maxPile)) — `log(maxPile)` binary-search steps, each an O(n) feasibility sweep. Space O(1).

**The `ceil` trick:** `(p + speed - 1) / speed` computes `ceil(p/speed)` with integer arithmetic — no floating point, no rounding bugs. Memorize it; it appears in every "search on answer" problem.

**Follow-ups:**
1. *"Why `hi = max(piles)`?"* → At speed = the biggest pile, every pile takes exactly 1 hour (you can't beat one-pile-per-hour). Higher speed wastes nothing more.
2. *"Why use `long` for hours?"* → `n` up to 10⁴ and each pile up to 10⁹ at speed 1 → hours can exceed `int`. Overflow is a silent correctness bug.
3. *"What if `h < n`?"* → Impossible to finish (each pile needs ≥1 hour); the constraints usually guarantee `h ≥ n`. Worth stating.

**Common mistakes that get you rejected:**
- Linear-scanning speeds → TLE; the whole point is the log.
- `lo = 0` (speed 0 means never finishing → division by zero in feasibility).
- Integer overflow in the hours sum.
- Using `lo = mid` / `hi = mid - 1` and infinite-looping — stick to the Template-3 structure.

---

### Q14: Capacity To Ship Packages Within D Days
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
`weights[i]` is the weight of the i-th package; they must be shipped **in order**. Each day the ship loads packages (in order) up to its capacity `C`. Return the **minimum** capacity `C` so all packages ship within `D` days.

**What interviewer is testing:**
The same search-on-answer pattern as Koko, but now `feasible(C)` is a greedy *packing* sweep with an order constraint. Defining `lo`/`hi` correctly is the subtle part.

**Ideal Answer:**

*Monotonicity:* if capacity `C` ships everything in ≤ D days, any larger capacity also does. So `feasible(C)` is monotonic → binary search the smallest feasible `C`.

- **Search space:** `lo = max(weights)` — capacity must hold the heaviest single package, else it never ships. `hi = sum(weights)` — one giant day ships everything.
- **`feasible(C)`:** greedily pack in order; start a new day when the next package would overflow `C`; count days; feasible iff `days ≤ D`.

```java
public int shipWithinDays(int[] weights, int D) {
    int lo = 0, hi = 0;
    for (int w : weights) {
        lo = Math.max(lo, w);    // must fit the heaviest package
        hi += w;                 // everything in one day
    }
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (daysNeeded(weights, mid) <= D) hi = mid;   // capacity works -> try smaller
        else lo = mid + 1;                             // too small -> need bigger
    }
    return lo;
}

private int daysNeeded(int[] weights, int cap) {
    int days = 1, load = 0;
    for (int w : weights) {
        if (load + w > cap) {     // can't fit -> start a new day
            days++;
            load = 0;
        }
        load += w;
    }
    return days;
}
```

**Dry run** — `weights = [1,2,3,4,5,6,7,8,9,10]`, `D = 5`. lo=10, hi=55.
- mid=32: [1..7]=28 (next 8 → 36>32, new day), [8,9,10]=27 → 2 days ≤5 → hi=32.
- lo=10,hi=32,mid=21: [1..6]=21,[7,8]=15,[9,10]=19 → 3 days ≤5 → hi=21.
- lo=10,hi=21,mid=15: [1,2,3,4,5]=15,[6,7]... → 5 days ≤5 → hi=15.
- lo=10,hi=15,mid=12: → 6 days >5 → lo=13.
- lo=13,hi=15,mid=14: → 5 days ≤5 → hi=14.
- lo=13,hi=14,mid=13: → 6 days >5 → lo=14.
- lo==hi==14 → capacity 14. ✅

**Complexity:** Time O(n log(sum)), Space O(1).

**Follow-ups:**
1. *"Why `lo = max(weights)` not 1?"* → A capacity below the heaviest package can never ship it (it overflows every day) → invalid; the answer is bounded below by `max`.
2. *"Difference from Split Array Largest Sum (Q15)?"* → They are the *same problem*. "Ship in D days with min capacity" ≡ "split into D subarrays minimizing the max subarray sum." Saying this signals pattern mastery.
3. *"Packages can be reordered?"* → Then it becomes a bin-packing / load-balancing problem (NP-hard in general); the order constraint is what makes the greedy feasibility check valid.

**Common mistakes:**
- `lo = 1` → feasibility never succeeds for the heaviest package; wrong answer or bad bounds.
- Resetting `load = 0` but forgetting to then add `w` (drops a package).
- Counting `days` starting at 0 instead of 1.

---

### Q15: Split Array Largest Sum
**Company:** Google, Amazon, Microsoft, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Split `nums` (non-negative) into `k` non-empty **contiguous** subarrays. Minimize the **largest** subarray sum among the splits. Return that minimized largest sum.

**What interviewer is testing:**
Recognizing "minimize the maximum" → binary search on answer (the same engine as Ship Packages). The DP solution exists but is O(n²k); the binary search is O(n log(sum)) and is the expected senior answer.

**Ideal Answer:**

*DP (mention it):* `dp[i][j]` = min largest-sum splitting first `i` elements into `j` parts. O(n²k) time, O(nk) space. Correct but slow.

*Binary search on answer (optimal):* Guess a candidate `maxSum = X`. Greedily count how many subarrays are needed if no subarray may exceed `X`. If `partsNeeded(X) ≤ k`, then `X` is achievable (we can always merge to use fewer parts, and merging only lowers the part count). `feasible(X)` is monotonic.

- **Search space:** `lo = max(nums)` (a part can't sum less than its largest element), `hi = sum(nums)` (one part).
- **`feasible(X)`:** greedy sweep counting parts; feasible iff `parts ≤ k`.

```java
public int splitArray(int[] nums, int k) {
    int lo = 0, hi = 0;
    for (int x : nums) { lo = Math.max(lo, x); hi += x; }

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (partsNeeded(nums, mid) <= k) hi = mid;   // X achievable -> try smaller max
        else lo = mid + 1;                           // too small -> need bigger max
    }
    return lo;
}

// minimum number of contiguous subarrays such that each sum <= maxSum
private int partsNeeded(int[] nums, int maxSum) {
    int parts = 1, cur = 0;
    for (int x : nums) {
        if (cur + x > maxSum) {   // start a new part
            parts++;
            cur = 0;
        }
        cur += x;
    }
    return parts;
}
```

**Dry run** — `nums = [7,2,5,10,8]`, `k = 2`. lo=10, hi=32.
- mid=21: [7,2,5]=14, then 10 → 14+10=24>21 new part [10,8]=18 → 2 parts ≤2 → hi=21.
- lo=10,hi=21,mid=15: [7,2,5]=14, 10>15-14 new [10], 8 → 10+8=18>15 new [8] → 3 parts >2 → lo=16.
- lo=16,hi=21,mid=18: [7,2,5]=14, new [10,8]=18 → 2 parts ≤2 → hi=18.
- lo=16,hi=18,mid=17: [7,2,5]=14, new [10] (10+8=18>17 new [8]) → 3 parts >2 → lo=18.
- lo==hi==18 → answer 18. ✅ (split `[7,2,5]` and `[10,8]`.)

**Complexity:** Time O(n log(sum)), Space O(1). Beats DP.

**Follow-ups:**
1. *"Prove monotonicity."* → If max-sum `X` needs `p` parts, a larger budget `X' > X` needs `≤ p` parts (each part can hold more). So feasibility (parts ≤ k) only improves as `X` grows → monotone.
2. *"Equivalent problems?"* → Ship Packages (Q14), Painter's Partition / Allocate Pages (Q16) are all identical. One mental model.
3. *"Negative numbers?"* → Binary-search-on-sum breaks (prefix sums aren't monotonic); fall back to DP.

**Common mistakes:**
- `lo = 1` or `lo = 0` instead of `max(nums)` → wrong lower bound.
- Greedy that miscounts parts (resets `cur` but re-adds incorrectly).
- Reaching for DP and TLE-ing on large `n` when binary search is O(n log sum).

---

### Q16: Allocate Minimum Number of Pages / Painter's Partition
**Company:** Amazon, Flipkart, Microsoft (very common in Indian product-company loops)
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
`books[i]` = number of pages in book `i`; books must be allocated to `k` students in **contiguous** segments (a student gets a consecutive block). Minimize the **maximum** pages any single student reads. (Painter's Partition is identical: `k` painters, contiguous boards, minimize the max board-length any painter does.) If `k > number of books`, return -1.

**What interviewer is testing:**
Same search-on-answer engine, framed differently. Whether you see through the wording to "minimize the maximum contiguous-segment sum with k segments" = Q15 exactly.

**Ideal Answer:**

Identical to Split Array Largest Sum. Guess a page limit `X`; greedily assign books to a student until adding the next would exceed `X`, then move to the next student. If students needed ≤ k, `X` is feasible.

- **Search space:** `lo = max(books)`, `hi = sum(books)`.
- **Edge case:** if `k > books.length`, you can't give every student a non-empty contiguous block → return -1.

```java
public int findMinPages(int[] books, int k) {
    int n = books.length;
    if (k > n) return -1;                 // not enough books for k students

    int lo = 0, hi = 0;
    for (int p : books) { lo = Math.max(lo, p); hi += p; }

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (studentsNeeded(books, mid) <= k) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

private int studentsNeeded(int[] books, int maxPages) {
    int students = 1, cur = 0;
    for (int p : books) {
        if (cur + p > maxPages) {
            students++;
            cur = 0;
        }
        cur += p;
    }
    return students;
}
```

**Dry run** — `books = [12,34,67,90]`, `k = 2`. lo=90, hi=203.
- mid=146: [12,34,67]=113, new [90] → 2 students ≤2 → hi=146.
- lo=90,hi=146,mid=118: [12,34,67]=113, new [90] → 2 ≤2 → hi=118.
- lo=90,hi=118,mid=104: [12,34]=46, +67=113>104 new [67], +90 new [90] → 3 >2 → lo=105.
- lo=105,hi=118,mid=111: [12,34]=46,+67=113>111 new [67],new [90] → 3 >2 → lo=112.
- lo=112,hi=118,mid=115: [12,34,67]=113, new [90] → 2 ≤2 → hi=115.
- lo=112,hi=115,mid=113: [12,34,67]=113, new [90] → 2 ≤2 → hi=113.
- lo=112,hi=113,mid=112: 113>112 → [12,34]=46,new[67],new[90] → 3 >2 → lo=113.
- lo==hi==113 → answer 113. ✅

**Complexity:** Time O(n log(sum)), Space O(1).

**Follow-ups:**
1. *"Why return -1 when k > n?"* → Each student needs at least one book and blocks are contiguous and non-empty; impossible to satisfy.
2. *"Books can be split across students?"* → Then it's trivial: answer = `ceil(sum / k)`. The contiguity constraint is what makes it interesting.
3. *"Same as which other problems?"* → Ship Packages, Split Array Largest Sum. Name them.

**Common mistakes:**
- Missing the `k > n` guard.
- Same greedy/bounds bugs as Q15.
- Treating books as reorderable (contiguity is essential).

---

### Q17: Aggressive Cows / Magnetic Force Between Two Balls
**Company:** Amazon, Google (LeetCode 1552), classic competitive-programming
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given sorted positions `pos[]` of `n` stalls (or basket positions) and `k` cows (balls), place the `k` cows so that the **minimum** distance between any two is **maximized**. Return that maximum-possible minimum distance.

**What interviewer is testing:**
The **maximize-the-minimum** flavor of search-on-answer. Same engine, but you search for the *largest* feasible distance — note the flipped keep-side (the `lastTrue` mirror from Template 2).

**Ideal Answer:**

*Monotonicity (reversed):* if you can place all `k` cows with minimum gap ≥ `d`, you can also do it for any `d' < d` (smaller gaps are easier). So `feasible(d)` is `true, true, ..., true, false, false` — we want the **largest** true.

- **Search space:** `lo = 1` (smallest meaningful gap), `hi = pos[n-1] - pos[0]` (the widest span).
- **`feasible(d)`:** greedily place the first cow at `pos[0]`, then place each next cow at the first stall ≥ lastPlaced + `d`; feasible iff we place all `k`.

```java
public int maxMinDistance(int[] pos, int k) {
    Arrays.sort(pos);                       // positions must be sorted
    int lo = 1, hi = pos[pos.length - 1] - pos[0];
    int answer = 0;
    while (lo <= hi) {                       // track best feasible (maximize)
        int mid = lo + (hi - lo) / 2;
        if (canPlace(pos, k, mid)) {
            answer = mid;                    // mid works -> try a LARGER gap
            lo = mid + 1;
        } else {
            hi = mid - 1;                    // too big -> shrink
        }
    }
    return answer;
}

private boolean canPlace(int[] pos, int k, int minGap) {
    int placed = 1, last = pos[0];           // first cow at the leftmost stall
    for (int i = 1; i < pos.length; i++) {
        if (pos[i] - last >= minGap) {       // far enough from the last placed cow
            placed++;
            last = pos[i];
            if (placed == k) return true;    // all cows placed
        }
    }
    return placed >= k;
}
```

**Dry run** — `pos = [1,2,4,8,9]`, `k = 3`. Sorted. lo=1, hi=8.
- mid=4: place at 1, next ≥5 → 8 (2), next ≥12 → none → 2 <3 → false → hi=3.
- lo=1,hi=3,mid=2: place 1, ≥3 → 4 (2), ≥6 → 8 (3) → true → answer=2, lo=3.
- lo=3,hi=3,mid=3: place 1, ≥4 → 4 (2), ≥7 → 8 (3) → true → answer=3, lo=4.
- lo=4 > hi=3 → return 3. ✅ (cows at 1, 4, 8; min gap = 3.)

**Complexity:** Time O(n log n) sort + O(n log(span)) search = O(n log n). Space O(1).

**Follow-ups:**
1. *"Why track `answer` instead of returning `lo`?"* → This uses the Template-1 `lo<=hi` form for maximize problems; tracking the last feasible value is clearest. (Alternatively use the `lastTrue` Template-2 mirror with round-up mid.)
2. *"Magnetic Force version?"* → Identical: `m` balls in baskets at sorted positions, maximize the minimum magnetic force (= minimum pairwise distance).
3. *"Place to maximize the *sum* of gaps?"* → Different problem; just spread to the extremes, no binary search.

**Common mistakes:**
- Forgetting to **sort** the positions.
- Searching for the smallest feasible distance (this is a maximize problem — flip the direction).
- Off-by-one / infinite loop from mixing the minimize-template updates into a maximize problem.

---

### Q18: Nth Root of a Number (precision binary search)
**Company:** Amazon, Google, D. E. Shaw
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given `n` and `m`, find the `n`-th root of `m` (i.e. `x` such that `x^n = m`). Return it to a tolerance (or -1 if it's not an exact integer, for the integer variant).

**What interviewer is testing:**
Binary search on a **real-valued** answer space and controlling precision — when to stop (epsilon vs fixed iterations), and avoiding overflow in `x^n`.

**Ideal Answer:**

The root is monotonic in `m`, so binary-search `x` in `[1, m]`. Compute `x^n` carefully (cap to avoid overflow). For floating-point, iterate a fixed number of times (e.g. 100) or until `hi - lo < eps` — never rely on exact equality with doubles.

```java
// Real-valued n-th root to ~1e-7 precision.
public double nthRoot(int n, double m) {
    double lo = 0, hi = Math.max(1.0, m), eps = 1e-7;
    while (hi - lo > eps) {
        double mid = lo + (hi - lo) / 2;
        if (power(mid, n) < m) lo = mid;     // root is larger
        else hi = mid;                       // root is mid or smaller
    }
    return lo;
}

private double power(double base, int exp) {
    double result = 1.0;
    for (int i = 0; i < exp; i++) result *= base;   // exp is small in interviews
    return result;
}

// Integer variant: exact n-th root or -1, with overflow-safe comparison.
public int nthRootInt(int n, int m) {
    int lo = 1, hi = m;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        long p = powCapped(mid, n, m);       // capped so it never overflows past m
        if (p == m) return mid;
        else if (p < m) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

private long powCapped(long base, int exp, long cap) {
    long result = 1;
    for (int i = 0; i < exp; i++) {
        result *= base;
        if (result > cap) return cap + 1;    // early exit: definitely too big
    }
    return result;
}
```

**Dry run (integer)** — `n = 3, m = 27`: lo=1,hi=27,mid=14→14³ capped >27 → hi=13; ... converges to mid=3, 3³=27 → return 3. ✅ `n=4,m=69` → no exact root → -1.

**Complexity:** Integer: O(log m × n). Real: O(log((hi-lo)/eps) × n).

**Follow-ups:**
1. *"Why cap the power?"* → `mid^n` overflows fast (e.g. `mid=10^5, n=5`); capping at `cap+1` avoids overflow while preserving the comparison.
2. *"Fixed iterations vs epsilon?"* → 100 iterations of halving gives ~2^-100 precision — safe and avoids epsilon edge cases. Both are acceptable; state your choice.
3. *"Newton's method?"* → Converges quadratically (faster), but binary search is simpler to get right under pressure and demonstrates the monotonic-search idea.

**Common mistakes:**
- `int` overflow computing `mid^n` (the classic bug here).
- Comparing doubles with `==`.
- `lo = mid + 1` in the *floating-point* version (you must keep `mid` since the answer isn't discrete).

---

### Q19: Find K-th Smallest Pair Distance
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given an integer array `nums`, the distance of a pair `(a, b)` is `|a - b|`. Return the **k-th smallest** distance among all `n(n-1)/2` pairs.

**What interviewer is testing:**
Binary search on the **answer (a distance value)** combined with a sliding-window count. Recognizing that "k-th smallest X" over an implicit huge set is often binary-search-on-value + a counting predicate.

**Ideal Answer:**

Sort the array. Binary-search the distance `d` over `[0, max - min]`. For a candidate `d`, count how many pairs have distance `≤ d` using a sliding window (since sorted, for each right index, the count of valid lefts is `right - left`). `countPairs(d)` is monotonic in `d`, so find the smallest `d` with `count(d) ≥ k`.

```java
public int smallestDistancePair(int[] nums, int k) {
    Arrays.sort(nums);
    int n = nums.length;
    int lo = 0, hi = nums[n - 1] - nums[0];

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countPairsAtMost(nums, mid) >= k) hi = mid;   // enough pairs -> try smaller d
        else lo = mid + 1;                                // too few -> need larger d
    }
    return lo;
}

// number of pairs with distance <= d, via sliding window on the sorted array
private int countPairsAtMost(int[] nums, int d) {
    int count = 0, left = 0;
    for (int right = 0; right < nums.length; right++) {
        while (nums[right] - nums[left] > d) left++;      // shrink until window fits
        count += right - left;                            // all pairs (left..right-1, right)
    }
    return count;
}
```

**Dry run** — `nums = [1,3,1]` → sorted `[1,1,3]`, k=1. lo=0, hi=2.
- mid=1: right=0 count+0; right=1 (1-1=0≤1) count+1; right=2 (3-1=2>1 → left=1; 3-1=2>1 → left=2) count+0 → total 1 ≥1 → hi=1.
- lo=0,hi=1,mid=0: pairs with dist ≤0 → the two 1's → count 1 ≥1 → hi=0.
- lo==hi==0 → answer 0. ✅ (pair (1,1) has distance 0.)

**Complexity:** Time O(n log n + n log(maxDist)) — sort + (each binary-search step does an O(n) window count). Space O(1).

**Follow-ups:**
1. *"Why does the sliding window count correctly?"* → Sorted; for each `right`, all indices in `[left, right)` form a pair with distance ≤ d → `right - left` pairs. Amortized O(n) since `left` only advances.
2. *"k-th *largest* distance?"* → Count pairs with distance `≥ d` (or convert `k → totalPairs - k + 1`).

**Common mistakes:**
- Generating all pairs (O(n²)) → TLE for large n.
- Resetting `left` each iteration instead of letting it advance monotonically → O(n²) window.

---

### Q20: Kth Smallest Element in a Sorted Matrix
**Company:** Amazon, Google, Meta, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
`n × n` matrix where each row and each column is sorted ascending. Return the k-th smallest element (in sorted order, counting duplicates).

**What interviewer is testing:**
Two valid approaches — a min-heap merge (O(k log n)) and **binary search on the value** with a staircase count (O(n log(range))). The binary-search-on-value approach is the impressive one and ties back to the staircase trick from Q9.

**Ideal Answer:**

*Heap approach:* push the first element of each row; pop the smallest k times, pushing the next in that row. O(k log n).

*Binary search on value (optimal for large k):* The answer lies in `[matrix[0][0], matrix[n-1][n-1]]`. For a candidate value `mid`, count elements `≤ mid` using the staircase walk (start bottom-left): `count(mid)` is monotonic. Find the smallest `mid` with `count(mid) ≥ k` — that smallest value is guaranteed to be an actual matrix element.

```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    int lo = matrix[0][0], hi = matrix[n - 1][n - 1];
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countLessEqual(matrix, mid) >= k) hi = mid;  // enough -> shrink value
        else lo = mid + 1;                               // too few -> raise value
    }
    return lo;                                           // smallest value with >= k below it
}

// count of elements <= target, walking from bottom-left in O(n)
private int countLessEqual(int[][] matrix, int target) {
    int n = matrix.length;
    int row = n - 1, col = 0, count = 0;
    while (row >= 0 && col < n) {
        if (matrix[row][col] <= target) {
            count += row + 1;     // entire column up to 'row' is <= target
            col++;
        } else {
            row--;                // this cell too big -> move up
        }
    }
    return count;
}
```

**Dry run** — `matrix = [[1,5,9],[10,11,13],[12,13,15]]`, k=8. lo=1, hi=15.
- mid=8: count≤8 → only 1,5 → 2 <8 → lo=9.
- lo=9,hi=15,mid=12: count≤12 → 1,5,9,10,11,12 → 6 <8 → lo=13.
- lo=13,hi=15,mid=14: count≤14 → adds 13,13 → 8 ≥8 → hi=14.
- lo=13,hi=14,mid=13: count≤13 → 1,5,9,10,11,12,13,13 = 8 ≥8 → hi=13.
- lo==hi==13 → answer 13. ✅

**Why is the returned value an actual element?** We return the smallest value `v` with `count(v) ≥ k`. If `v` weren't in the matrix, `count(v) == count(v-1)`, contradicting `v` being the smallest such value. So `v` must be present.

**Complexity:** Binary search O(n log(max - min)) — `log(range)` steps × O(n) count. Heap O(k log n). For large k, binary search wins; it's also O(1) extra space.

**Follow-ups:**
1. *"Heap vs binary search trade-off?"* → Heap is O(k log n) and intuitive; binary search is O(n log range) independent of k and uses O(1) space. Mention both.
2. *"Why start the count at bottom-left?"* → Like Q9's staircase: that corner lets you add a whole column slice or move up, O(n) per count.

**Common mistakes:**
- Sorting the whole matrix (O(n² log n)) — ignores the structure.
- Buggy staircase count (wrong corner or wrong increment).

---

### Q21: Sqrt(x)
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return the integer square root of a non-negative `x` (floor of √x), without using built-in sqrt.

**What interviewer is testing:**
Boundary binary search (largest `m` with `m² ≤ x`) and overflow awareness.

**Ideal Answer:**

Find the largest `m` with `m * m ≤ x`. Use `long` for `m*m` to avoid overflow.

```java
public int mySqrt(int x) {
    if (x < 2) return x;                  // 0 -> 0, 1 -> 1
    long lo = 1, hi = x;
    while (lo <= hi) {
        long mid = lo + (hi - lo) / 2;
        long sq = mid * mid;
        if (sq == x) return (int) mid;
        else if (sq < x) lo = mid + 1;    // mid could still be the floor; keep searching up
        else hi = mid - 1;
    }
    return (int) hi;                      // hi is the floor of sqrt(x)
}
```

**Dry run** — `x = 8`: lo=1,hi=8,mid=4→16>8 hi=3; mid=2→4<8 lo=3; mid=3→9>8 hi=2; lo>hi → return hi=2. ✅ (⌊√8⌋ = 2.)

**Complexity:** O(log x) time, O(1) space.

**Follow-ups:**
1. *"Why return `hi`?"* → When the loop exits, `hi` is the largest value whose square ≤ x (the floor).
2. *"Floating-point sqrt to precision?"* → Same as Q18 with `n = 2`, epsilon loop.
3. *"Newton's method?"* → `m = (m + x/m) / 2` converges fast; mention as an alternative.

**Common mistakes:**
- `int` overflow on `mid * mid` for large x — use `long`.
- Returning `lo` (off by one — `lo` overshoots).

---

### Q22: Find the Smallest Divisor Given a Threshold
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given `nums` and a `threshold`, find the smallest positive divisor `d` such that `Σ ceil(nums[i] / d) ≤ threshold`.

**What interviewer is testing:**
Another search-on-answer instance (sibling of Koko). Reinforces the pattern: smaller divisor → larger sum, so the sum is monotonically *decreasing* in `d`.

**Ideal Answer:**

`sum(d) = Σ ceil(nums[i] / d)` decreases as `d` increases. We want the smallest `d` with `sum(d) ≤ threshold`. Binary-search `d` in `[1, max(nums)]`.

```java
public int smallestDivisor(int[] nums, int threshold) {
    int lo = 1, hi = 0;
    for (int x : nums) hi = Math.max(hi, x);
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (sumOfCeil(nums, mid) <= threshold) hi = mid;  // works -> try smaller divisor
        else lo = mid + 1;                                // too big a sum -> larger divisor
    }
    return lo;
}

private int sumOfCeil(int[] nums, int d) {
    int sum = 0;
    for (int x : nums) sum += (x + d - 1) / d;            // ceil(x / d)
    return sum;
}
```

**Dry run** — `nums = [1,2,5,9]`, `threshold = 6`. lo=1, hi=9.
- mid=5: ceil = 1+1+1+2 = 5 ≤6 → hi=5.
- lo=1,hi=5,mid=3: 1+1+2+3 = 7 >6 → lo=4.
- lo=4,hi=5,mid=4: 1+1+2+3 = 7 >6 → lo=5.
- lo==hi==5 → divisor 5. ✅

**Complexity:** O(n log(max)) time, O(1) space.

**Follow-ups:**
1. *"How is this Koko?"* → `d` = "eating speed," `threshold` = "hours." Same `ceil` sum, same monotonic feasibility. Identical engine.
2. *"What if threshold < n?"* → Even `d = ∞` gives sum = n (each ceil ≥ 1), so it's infeasible; the problem guarantees `threshold ≥ n`.

**Common mistakes:**
- `lo = 0` (divide by zero).
- Floating-point ceil instead of the integer `(x + d - 1) / d` trick.

---

### Q23: Find in Mountain Array
**Company:** Amazon, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
A *mountain array* strictly increases then strictly decreases (peak in the middle). Using only an opaque `MountainArray` API (`get(i)` and `length()`), return the **smallest index** whose value equals `target`, or -1. Minimize `get` calls.

**What interviewer is testing:**
Composing **three** binary searches: find the peak, then search the ascending side, then the descending side. Plus minimizing expensive API calls (each `get` "costs"), which mirrors real systems where a probe is expensive.

**Ideal Answer:**

1. Binary-search the **peak** (Find Peak Element, Q10's slope idea).
2. Binary-search the **ascending** half `[0, peak]` (normal ascending search) — prefer this first since we want the smallest index.
3. If not found, binary-search the **descending** half `(peak, n-1]` (reverse comparison).

```java
interface MountainArray { int get(int index); int length(); }

public int findInMountainArray(int target, MountainArray arr) {
    int n = arr.length();

    // 1. Find the peak index.
    int lo = 0, hi = n - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr.get(mid) < arr.get(mid + 1)) lo = mid + 1;   // ascending -> peak right
        else hi = mid;                                       // descending -> peak here/left
    }
    int peak = lo;

    // 2. Search ascending side [0, peak] (smallest index first).
    int a = ascendingSearch(target, arr, 0, peak);
    if (a != -1) return a;

    // 3. Search descending side (peak, n-1].
    return descendingSearch(target, arr, peak + 1, n - 1);
}

private int ascendingSearch(int target, MountainArray arr, int lo, int hi) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2, v = arr.get(mid);
        if (v == target) return mid;
        else if (v < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

private int descendingSearch(int target, MountainArray arr, int lo, int hi) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2, v = arr.get(mid);
        if (v == target) return mid;
        else if (v > target) lo = mid + 1;   // descending: larger means go right
        else hi = mid - 1;
    }
    return -1;
}
```

**Dry run** — array `[1,2,3,4,5,3,1]`, target 3: peak at index 4 (value 5). Ascending search `[0,4]` finds value 3 at index 2 → return 2 (the smaller index, even though 3 also appears at index 5). ✅

**Complexity:** O(log n) `get` calls — three log-n searches. Space O(1).

**Follow-ups:**
1. *"Why search the ascending side first?"* → We want the *smallest* index; a match on the ascending side is at a lower index than any on the descending side.
2. *"Cache `get` results?"* → Memoize repeated indices (peak search reuses `mid` and `mid+1`); a HashMap of probed indices cuts redundant API calls.
3. *"What if the array could be all-increasing (no descending part)?"* → The peak search returns `n-1`; the descending search range is empty — still correct.

**Common mistakes:**
- Searching the descending side first → returns a larger index.
- Forgetting to reverse the comparison on the descending half.
- Excessive `get` calls (the problem explicitly limits them).

---

# SORTING

> Interviewers ask you to *implement* sorts to verify you understand recursion, partitioning, and in-place memory — not to test memorization of `Arrays.sort`. The three you must be able to write blind: **merge sort** (stable, O(n log n) guaranteed, O(n) space), **quick sort** (in-place, O(n log n) average but O(n²) worst, pivot choice matters), and **heap sort** (in-place, O(n log n) guaranteed, not stable). Know their trade-offs cold — "which sort and why" is a standard follow-up.

---

### Q24: Merge Sort (full implementation)
**Company:** Amazon, Microsoft, Google, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Implement merge sort. State its complexity and stability.

**What interviewer is testing:**
Divide-and-conquer, a correct **merge** step (the part people botch), and understanding why it's stable and O(n log n) guaranteed.

**Ideal Answer:**

Split the array in half, recursively sort each half, then **merge** the two sorted halves by repeatedly taking the smaller front element. The merge is the heart: use a temp array, two pointers.

```java
public void mergeSort(int[] a) {
    if (a.length < 2) return;
    int[] temp = new int[a.length];      // single reusable buffer (avoid per-call allocation)
    sort(a, temp, 0, a.length - 1);
}

private void sort(int[] a, int[] temp, int lo, int hi) {
    if (lo >= hi) return;                 // base case: 0 or 1 element
    int mid = lo + (hi - lo) / 2;
    sort(a, temp, lo, mid);               // sort left half
    sort(a, temp, mid + 1, hi);           // sort right half
    merge(a, temp, lo, mid, hi);          // merge the two sorted halves
}

private void merge(int[] a, int[] temp, int lo, int mid, int hi) {
    for (int i = lo; i <= hi; i++) temp[i] = a[i];   // copy the range
    int i = lo, j = mid + 1, k = lo;
    while (i <= mid && j <= hi) {
        if (temp[i] <= temp[j]) a[k++] = temp[i++];  // <= keeps it STABLE (left first on tie)
        else a[k++] = temp[j++];
    }
    while (i <= mid) a[k++] = temp[i++];             // drain leftovers (only one runs)
    while (j <= hi) a[k++] = temp[j++];
}
```

**Dry run** — `[5,2,4,1]`: split → `[5,2]`,`[4,1]`; sort → `[2,5]`,`[1,4]`; merge → compare 2,1→1; 2,4→2; 5,4→4; drain 5 → `[1,2,4,5]`. ✅

**Complexity:** Time O(n log n) in **all** cases (log n levels × O(n) merge per level). Space O(n) for the temp buffer + O(log n) recursion stack. **Stable** (the `<=` ensures equal keys keep input order).

**Follow-ups:**
1. *"Why is merge sort stable but quick sort isn't?"* → The `<=` tie-break keeps equal elements in their original relative order; quick sort's partition swaps across the array, scrambling equal keys.
2. *"When prefer merge sort over quick sort?"* → When you need stability (e.g. multi-key sorting), guaranteed O(n log n) (no worst-case quadratic), or external/linked-list sorting (merge sort works well on linked lists with O(1) extra space).
3. *"Bottom-up (iterative) merge sort?"* → Merge runs of size 1, 2, 4, ... — avoids recursion stack. Good for very deep arrays.
4. *"Java's `Arrays.sort` for objects uses what?"* → TimSort (a hybrid of merge sort and insertion sort) — stable; primitives use dual-pivot quicksort (not stable, but primitives have no identity).

**Common mistakes:**
- Allocating a new temp array in every recursive call → O(n log n) allocations / extra GC pressure. Allocate once.
- Using `<` instead of `<=` in the merge → loses stability (silent, fails a "keep order of equal keys" follow-up).
- Off-by-one in `mid+1` / `hi` bounds.

---

### Q25: Quick Sort (full implementation + pivot strategy)
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Implement quick sort. Discuss pivot choice and the worst case.

**What interviewer is testing:**
In-place partitioning (Lomuto or Hoare), why a bad pivot gives O(n²), and how randomization fixes it.

**Ideal Answer:**

Pick a pivot, **partition** so everything ≤ pivot is left and everything > is right, then recurse on both sides. Partitioning is in-place. Use a **randomized pivot** to avoid the sorted-input O(n²) trap.

```java
public void quickSort(int[] a) {
    sort(a, 0, a.length - 1);
}

private void sort(int[] a, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition(a, lo, hi);     // pivot lands at its final sorted position p
    sort(a, lo, p - 1);               // sort left of pivot
    sort(a, p + 1, hi);               // sort right of pivot
}

// Lomuto partition with a RANDOM pivot (defends against sorted / adversarial input)
private int partition(int[] a, int lo, int hi) {
    int r = lo + (int) (Math.random() * (hi - lo + 1));  // random index in [lo, hi]
    swap(a, r, hi);                   // move pivot to the end
    int pivot = a[hi];
    int i = lo - 1;                   // boundary of the "<= pivot" region
    for (int j = lo; j < hi; j++) {
        if (a[j] <= pivot) {
            i++;
            swap(a, i, j);            // grow the <= region
        }
    }
    swap(a, i + 1, hi);               // place pivot just after the <= region
    return i + 1;
}

private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Dry run** — `[3,1,2]` (pivot = last, say 2): j=0 a[0]=3>2 skip; j=1 a[1]=1≤2 → i=0 swap(0,1)→`[1,3,2]`; place pivot swap(i+1=1, hi=2)→`[1,2,3]`, return p=1. Recurse left `[1]`, right `[3]`. → `[1,2,3]`. ✅

**Complexity:** Time O(n log n) **average**, O(n²) **worst** (already-sorted with first/last pivot, or all-equal with Lomuto). Space O(log n) average recursion stack (O(n) worst). **Not stable.**

**Follow-ups:**
1. *"Why randomize the pivot?"* → A fixed first/last pivot on sorted input gives maximally unbalanced partitions → O(n²). Random (or median-of-three) makes the bad case astronomically unlikely.
2. *"All-equal elements?"* → Lomuto degrades to O(n²). Use **3-way partitioning** (Dutch flag, Q28) to group equals in the middle — O(n) on all-equal input.
3. *"Hoare vs Lomuto partition?"* → Hoare does fewer swaps and handles duplicates better but is trickier (pivot doesn't end at a fixed index). Lomuto is easier to write correctly under pressure.
4. *"Tail-call / hybrid optimizations?"* → Recurse on the smaller side first and loop on the larger (caps stack at O(log n)); switch to insertion sort for small subarrays (~16 elements).

**Common mistakes:**
- Fixed pivot → O(n²) on sorted input (and an interviewer *will* hand you sorted input).
- Off-by-one in the partition boundary (`i` management).
- Recursing on the wrong subranges (`p` vs `p±1`) → infinite recursion or unsorted output.

---

### Q26: Heap Sort (full implementation)
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Implement heap sort in place. State complexity and stability.

**What interviewer is testing:**
Building a max-heap in an array (sift-down), the heapify-then-extract pattern, and why it's O(n log n) guaranteed with O(1) extra space.

**Ideal Answer:**

Build a **max-heap** in place (so the largest is at the root, index 0). Then repeatedly swap the root with the last element, shrink the heap, and sift the new root down. This sorts ascending in place.

```java
public void heapSort(int[] a) {
    int n = a.length;
    // 1. Build max-heap: heapify all internal nodes bottom-up. O(n).
    for (int i = n / 2 - 1; i >= 0; i--) siftDown(a, i, n);
    // 2. Repeatedly extract the max to the end, shrink, re-heapify.
    for (int end = n - 1; end > 0; end--) {
        swap(a, 0, end);            // move current max to its final position
        siftDown(a, 0, end);        // restore heap on the reduced range [0, end)
    }
}

// Sift a[i] down within the heap of size 'size' to restore the max-heap property.
private void siftDown(int[] a, int i, int size) {
    while (true) {
        int largest = i, l = 2 * i + 1, r = 2 * i + 2;
        if (l < size && a[l] > a[largest]) largest = l;
        if (r < size && a[r] > a[largest]) largest = r;
        if (largest == i) break;    // heap property holds here
        swap(a, i, largest);
        i = largest;                // continue sifting down
    }
}

private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Dry run** — `[3,1,2]`: build heap (i=0): children 1,2 → largest=0 (3) → heap `[3,1,2]`. Extract: swap(0,2)→`[2,1,3]`, siftDown(0,2): child 1 → 2>1, no swap → `[2,1,3]`; swap(0,1)→`[1,2,3]`. → `[1,2,3]`. ✅

**Complexity:** Time O(n log n) in all cases (build heap is O(n); n extractions × O(log n) sift). Space O(1) — fully in place. **Not stable.**

**Why is build-heap O(n) not O(n log n)?** Nodes near the bottom (most of them) sift down only a little. Summing the work over all levels gives a convergent series ≈ 2n → O(n). Mention this; it's a classic follow-up.

**Follow-ups:**
1. *"Heap sort vs quick sort — why is quick sort usually faster in practice despite the same average complexity?"* → Heap sort has poor cache locality (jumps around the array via `2i+1`); quick sort scans contiguously. Heap sort's edge is guaranteed O(n log n) and O(1) space.
2. *"Min-heap to sort ascending?"* → You'd need to extract into a separate output array (extracting min to the end builds descending in place). Max-heap is the in-place ascending choice.
3. *"Where is the array-as-heap indexing?"* → Parent of `i` is `(i-1)/2`; children are `2i+1`, `2i+2`. Be ready to write these.

**Common mistakes:**
- Sifting **up** to build the heap (O(n log n)) instead of sifting down from `n/2 - 1` (O(n)).
- Forgetting to shrink the heap size during extraction (re-includes sorted elements).
- Index errors in `2i+1` / `2i+2` / `(i-1)/2`.

---

### Q27: QuickSelect — Kth Largest Element in an Array
**Company:** Amazon, Meta, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the k-th **largest** element in an unsorted array (not the k-th distinct — counting duplicates). Aim for average O(n).

**What interviewer is testing:**
QuickSelect — partition-based selection that finds the k-th element without fully sorting. Knowing it's O(n) average and how it differs from a heap solution.

**Ideal Answer:**

*Heap approach (mention):* a min-heap of size k → O(n log k), O(k) space. Great when k is small or for streaming.

*QuickSelect (optimal average):* k-th largest = `(n - k)`-th smallest (0-indexed). Partition like quicksort, but **only recurse into the side containing the target index** → O(n) average (n + n/2 + n/4 + ... = 2n). Randomize the pivot to avoid O(n²).

```java
public int findKthLargest(int[] nums, int k) {
    int n = nums.length;
    int targetIdx = n - k;                 // k-th largest = (n-k)-th smallest, 0-indexed
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        int p = partition(nums, lo, hi);
        if (p == targetIdx) return nums[p];
        else if (p < targetIdx) lo = p + 1;   // target is to the right
        else hi = p - 1;                       // target is to the left
    }
    return -1;  // unreachable for valid input
}

private int partition(int[] a, int lo, int hi) {
    int r = lo + (int) (Math.random() * (hi - lo + 1));
    swap(a, r, hi);
    int pivot = a[hi], i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (a[j] <= pivot) swap(a, ++i, j);
    }
    swap(a, ++i, hi);
    return i;
}

private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Dry run** — `nums = [3,2,1,5,6,4]`, k=2 → targetIdx = 6-2 = 4 (the 2nd largest = 5). Partitioning narrows until the element at index 4 is in place; that's 5. ✅

**Complexity:** Time O(n) **average**, O(n²) worst (mitigated by random pivot). Space O(1) iterative (no recursion). Heap alternative is O(n log k).

**Follow-ups:**
1. *"Guarantee O(n) worst case?"* → Median-of-medians pivot selection (BFPRT) gives deterministic O(n), but it's complex and rarely needed; randomized QuickSelect is the practical answer.
2. *"k-th largest in a *stream*?"* → A min-heap of size k (the heap top is the answer); QuickSelect needs all data at once.
3. *"Why recurse into only one side?"* → After partition, the pivot is at its final sorted index; comparing it with the target index tells you which side the answer is in — the other half is irrelevant.

**Common mistakes:**
- Fully sorting (O(n log n)) when O(n) average is asked.
- Confusing k-th largest with `(n-k)`-th vs `(n-k+1)`-th — get the index conversion right (`n - k`, 0-indexed).
- Fixed pivot → O(n²) on sorted input.

---

### Q28: Count Inversions (modified merge sort)
**Company:** Amazon, Google, Goldman Sachs, Flipkart
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Count the number of inversions: pairs `(i, j)` with `i < j` and `a[i] > a[j]`. (Measures how far the array is from sorted.)

**What interviewer is testing:**
Embedding a count into the merge step of merge sort to get O(n log n) instead of the O(n²) brute force.

**Ideal Answer:**

During the **merge**, when an element from the *right* half is placed before remaining elements of the *left* half, all those remaining left elements are greater → they each form an inversion with it. Count `mid - i + 1` at that moment.

```java
public long countInversions(int[] a) {
    int[] temp = new int[a.length];
    return sort(a, temp, 0, a.length - 1);
}

private long sort(int[] a, int[] temp, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    long count = sort(a, temp, lo, mid) + sort(a, temp, mid + 1, hi);
    count += merge(a, temp, lo, mid, hi);
    return count;
}

private long merge(int[] a, int[] temp, int lo, int mid, int hi) {
    for (int x = lo; x <= hi; x++) temp[x] = a[x];
    long inv = 0;
    int i = lo, j = mid + 1, k = lo;
    while (i <= mid && j <= hi) {
        if (temp[i] <= temp[j]) {
            a[k++] = temp[i++];            // no inversion (left <= right)
        } else {
            a[k++] = temp[j++];            // right element smaller -> inversions with all remaining left
            inv += (mid - i + 1);          // every left element from i..mid is > this right element
        }
    }
    while (i <= mid) a[k++] = temp[i++];
    while (j <= hi) a[k++] = temp[j++];
    return inv;
}
```

**Dry run** — `[2,4,1,3,5]`: total inversions = (2,1),(4,1),(4,3) = 3. The merge step counts exactly these as it interleaves halves. ✅

**Complexity:** Time O(n log n), Space O(n) (temp buffer). Brute force is O(n²).

**Follow-ups:**
1. *"Why `mid - i + 1`?"* → When `temp[j]` is placed, indices `i..mid` of the (sorted) left half are all still unplaced and all `> temp[j]`, so each is an inversion with it.
2. *"Use `long` why?"* → Up to ~n²/2 inversions (n=10⁵ → ~5×10⁹) overflows `int`.
3. *"Count via BIT/Fenwick tree instead?"* → Also O(n log n): process right-to-left, query "how many smaller seen so far." Mention as an alternative.

**Common mistakes:**
- Counting `1` per swap instead of `mid - i + 1` (only counts adjacent inversions).
- `int` overflow on the count.
- Using `<` instead of `<=` in merge → over-counts equal elements as inversions.

---

### Q29: Sort Colors / Dutch National Flag
**Company:** Amazon, Microsoft, Meta, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Array of 0s, 1s, 2s. Sort in place in **one pass**, O(1) space (don't just count).

**What interviewer is testing:**
The **3-way partition** (Dutch National Flag) — three pointers, single pass. This same partition makes quicksort robust to duplicates.

**Ideal Answer:**

Three regions: `[0, low)` all 0s, `[low, mid)` all 1s, `(high, n)` all 2s; `[mid, high]` is unknown. Scan `mid`:
- `0` → swap into the 0-region (`low`), advance both.
- `1` → in place, advance `mid`.
- `2` → swap to the 2-region (`high`), shrink `high`, **don't advance `mid`** (the swapped-in value is unexamined).

```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        switch (nums[mid]) {
            case 0: swap(nums, low++, mid++); break;  // 0 -> front
            case 1: mid++;                    break;  // 1 -> stays
            case 2: swap(nums, mid, high--);  break;  // 2 -> back; recheck new nums[mid]
        }
    }
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Dry run** — `[2,0,2,1,1,0]`: 
- mid0=2→swap(0,5)`[0,0,2,1,1,2]`,high4. mid0=0→swap(0,0)low1,mid1. mid1=0→swap(1,1)low2,mid2. mid2=2→swap(2,4)`[0,0,1,1,2,2]`,high3. mid2=1→mid3. mid3=1→mid4>high3 stop. → `[0,0,1,1,2,2]`. ✅

**Complexity:** O(n) time, O(1) space, single pass.

**Follow-ups:**
1. *"Why not advance `mid` on a 2?"* → The element swapped in from `high` hasn't been examined; it could be a 0 or 1 needing placement. Advancing would skip it.
2. *"Counting sort alternative?"* → Two passes (count, then overwrite). Works but the one-pass DNF is the intended O(1) single-pass answer.
3. *"Generalize to k colors / sort by a pivot?"* → That's exactly 3-way quicksort partitioning around a pivot value.

**Common mistakes:**
- Advancing `mid` after the `2`-swap (skips the unexamined value).
- Using `mid < high` instead of `mid <= high` (misses the last element).

---

### Q30: Merge Sorted Array (in-place, gap method)
**Company:** Amazon, Microsoft, Meta
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
`nums1` has length `m + n` with the first `m` elements sorted and `n` trailing zeros; `nums2` has `n` sorted elements. Merge `nums2` into `nums1` in place, sorted. O(1) extra space.

**What interviewer is testing:**
The **fill-from-the-back** trick: writing from the end of `nums1` avoids overwriting unprocessed elements. (The "gap method" is a follow-up for the no-trailing-space variant.)

**Ideal Answer:**

Place the largest remaining element at the end and walk backward. Since `nums1`'s tail is empty, writing back-to-front never clobbers an unread value.

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int i = m - 1;        // last real element of nums1
    int j = n - 1;        // last element of nums2
    int k = m + n - 1;    // write position (end of nums1)
    while (j >= 0) {                       // nums2 must be fully merged
        if (i >= 0 && nums1[i] > nums2[j]) {
            nums1[k--] = nums1[i--];       // take larger from nums1
        } else {
            nums1[k--] = nums2[j--];       // take from nums2
        }
    }
}
```

**Dry run** — `nums1=[1,2,3,0,0,0]`, m=3, `nums2=[2,5,6]`, n=3: 
- k5: nums1[2]=3 vs nums2[2]=6 → 6, k4,j1. k4: 3 vs 5 → 5,k3,j0. k3: 3 vs 2 → 3,k2,i1. k2: 2 vs 2 (not >) → 2(from nums2),k1,j-1. stop. → `[1,2,2,3,5,6]`. ✅

**Complexity:** O(m + n) time, O(1) space.

**The gap method (follow-up):** When two sorted arrays must be merged with **no spare space** (e.g. merge in place across two separate arrays for an in-place merge sort), use the **Shell-sort-style gap**: start with `gap = ceil((m+n)/2)`, compare elements `gap` apart, swap if out of order, halve the gap each round. O((m+n) log(m+n)) time, O(1) space.

```java
// Gap method: merge two sorted arrays with O(1) extra space (treats them as one virtual array).
public void mergeGap(int[] a, int[] b) {
    int n = a.length, m = b.length, total = n + m;
    int gap = (total + 1) / 2;             // ceil
    while (gap > 0) {
        int i = 0, j = gap;
        while (j < total) {
            int left  = (i < n) ? a[i] : b[i - n];
            int right = (j < n) ? a[j] : b[j - n];
            if (left > right) {            // out of order across the gap -> swap
                if (i < n && j < n)        { int t=a[i]; a[i]=a[j]; a[j]=t; }
                else if (i < n)            { int t=a[i]; a[i]=b[j-n]; b[j-n]=t; }
                else                       { int t=b[i-n]; b[i-n]=b[j-n]; b[j-n]=t; }
            }
            i++; j++;
        }
        gap = (gap == 1) ? 0 : (gap + 1) / 2;
    }
}
```

**Follow-ups:**
1. *"Why fill from the back?"* → Front-filling would overwrite `nums1` elements you haven't merged yet; the empty tail makes back-filling safe.
2. *"What if `nums1` had no extra space?"* → The gap method above, O(1) space.
3. *"Why does the loop condition only check `j >= 0`?"* → If `nums2` is exhausted, the remaining `nums1` elements are already in place (smaller and at the front).

**Common mistakes:**
- Filling from the front and clobbering unread data.
- Looping on `i >= 0 || j >= 0` and re-copying already-placed `nums1` elements (only `j >= 0` is needed).

---

### Q31: Custom Comparator Sorting (multi-criteria + the overflow trap)
**Company:** Amazon, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Sort a list of records by multiple criteria (e.g. by score descending, then name ascending). Also: explain the integer-overflow comparator bug.

**What interviewer is testing:**
Writing correct `Comparator`s and knowing the **`a - b` overflow trap** — a real production bug that throws `IllegalArgumentException: Comparison method violates its general contract`.

**Ideal Answer:**

Chain comparators with `thenComparing`. Never subtract for comparison — use `Integer.compare` / `Long.compare`.

```java
record Person(String name, int score) {}

public void sortPeople(List<Person> people) {
    people.sort(
        Comparator.comparingInt(Person::score).reversed()   // score DESC
                  .thenComparing(Person::name)              // tie-break: name ASC
    );
}
```

**The overflow trap.** A comparator that returns `a - b` looks fine but **overflows** for large/negative values, violating the comparator contract:

```java
// BUG: a - b overflows. e.g. a = Integer.MIN_VALUE, b = 1 -> wraps to a positive number.
Comparator<Integer> wrong = (a, b) -> a - b;

// CORRECT: never overflows.
Comparator<Integer> right = (a, b) -> Integer.compare(a, b);
```

If `a = Integer.MIN_VALUE` and `b = 1`, `a - b` overflows to a large *positive* value, so the comparator says `a > b` — wrong, and inconsistent across pairs. With TimSort's contract checks, this throws `IllegalArgumentException` at runtime.

**Sorting a primitive array by a custom order** requires boxing (no `Comparator` for `int[]`):

```java
// To sort int[] by a custom comparator, box to Integer[] (or use a stream).
Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new);
Arrays.sort(boxed, Comparator.reverseOrder());   // descending
```

**Complexity:** O(n log n) comparisons; each comparator call is O(1) (or O(L) for string compares).

**Follow-ups:**
1. *"Why does `a - b` throw, not just sort wrong?"* → Java's TimSort verifies the comparator's transitivity/consistency; an overflow-induced contradiction trips its self-check.
2. *"Stable sort guarantee?"* → `Collections.sort` / `Arrays.sort(Object[])` are stable (TimSort); `Arrays.sort(int[])` is not stable (but primitives have no identity, so it's moot).
3. *"Sort by absolute value, then descending?"* → `Comparator.comparingInt(Math::abs).thenComparing(Comparator.reverseOrder())`.

**Common mistakes:**
- `a - b` / `b - a` comparators (overflow → runtime exception).
- Trying `Arrays.sort(int[], comparator)` — doesn't compile; must box.
- Forgetting `.reversed()` placement (it reverses the *preceding* comparator chain — order matters).

---

### Q32: Largest Number (custom comparator)
**Company:** Amazon, Microsoft, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given non-negative integers, arrange them to form the **largest** number. Return it as a string (e.g. `[3,30,34,5,9]` → `"9534330"`).

**What interviewer is testing:**
Designing a *comparison rule* for an unusual ordering: compare `a+b` vs `b+a` as concatenated strings. Plus the leading-zero edge case.

**Ideal Answer:**

Two numbers `a`, `b` should be ordered so the concatenation that's lexicographically larger comes first: compare `(a + b)` vs `(b + a)` as strings. This is a valid total order (provably transitive).

```java
public String largestNumber(int[] nums) {
    String[] arr = new String[nums.length];
    for (int i = 0; i < nums.length; i++) arr[i] = String.valueOf(nums[i]);

    // Put the pair forming the larger concatenation first.
    Arrays.sort(arr, (a, b) -> (b + a).compareTo(a + b));   // descending by concat

    if (arr[0].equals("0")) return "0";   // all zeros -> "00.." should be just "0"

    StringBuilder sb = new StringBuilder();
    for (String s : arr) sb.append(s);
    return sb.toString();
}
```

**Dry run** — `[3,30,34,5,9]`: compare "3" vs "30" → "303" vs "330" → "330" bigger → 3 before 30. Final order `9,5,34,3,30` → `"9534330"`. ✅

**Complexity:** O(n log n × L) where L is the max string length (each compare concatenates). Space O(n) for the string array.

**Follow-ups:**
1. *"Why is `(a+b) vs (b+a)` a valid comparator (transitive)?"* → It can be proven that this ordering is consistent; it behaves like comparing the infinite repetition of each digit string. Be ready to assert transitivity.
2. *"Leading-zero case?"* → `[0,0]` would build `"00"`; check `arr[0].equals("0")` → return `"0"`.
3. *"Smallest number instead?"* → Flip the comparator to `(a+b).compareTo(b+a)` and handle leading zeros differently.

**Common mistakes:**
- Comparing by numeric value or by length → wrong (`30` vs `3`).
- Missing the all-zeros edge case → returns `"000"`.
- Integer overflow if you try to compare numerically — strings avoid it.

---

### Q33: Counting Sort & Radix Sort (when to use)
**Company:** Amazon, Google (conceptual / OA)
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Implement counting sort and explain radix sort. When do these beat comparison sorts?

**What interviewer is testing:**
Knowing that the O(n log n) comparison-sort lower bound can be **beaten** for bounded-integer keys, and the space/range trade-offs.

**Ideal Answer:**

**Counting sort** — for integers in a small known range `[0, k]`: count occurrences, then write back. O(n + k) time, O(k) space. Stable if you build from cumulative counts and iterate the input from the right.

```java
public int[] countingSort(int[] a, int maxVal) {
    int[] count = new int[maxVal + 1];
    for (int x : a) count[x]++;                  // tally
    for (int i = 1; i <= maxVal; i++) count[i] += count[i - 1];   // prefix sums = positions
    int[] out = new int[a.length];
    for (int i = a.length - 1; i >= 0; i--) {    // right-to-left keeps it STABLE
        out[--count[a[i]]] = a[i];
    }
    return out;
}
```

**Radix sort** — sort by digit, least-significant first, using counting sort as the stable subroutine for each digit. For `d` digits over base `b`: O(d × (n + b)). Great for fixed-width integers / strings.

```java
public void radixSort(int[] a) {
    int max = Arrays.stream(a).max().orElse(0);
    for (int exp = 1; max / exp > 0; exp *= 10) {     // one pass per decimal digit
        countingSortByDigit(a, exp);
    }
}
private void countingSortByDigit(int[] a, int exp) {
    int n = a.length;
    int[] out = new int[n], count = new int[10];
    for (int x : a) count[(x / exp) % 10]++;
    for (int i = 1; i < 10; i++) count[i] += count[i - 1];
    for (int i = n - 1; i >= 0; i--) {                 // stable
        int d = (a[i] / exp) % 10;
        out[--count[d]] = a[i];
    }
    System.arraycopy(out, 0, a, 0, n);
}
```

**Complexity:** Counting O(n + k); radix O(d(n + b)). Both **beat O(n log n)** when k (or d) is small relative to n.

**When to use (the actual interview point):**
- **Counting sort:** integer keys in a small, known range (ages 0–120, exam scores 0–100). Useless if the range is huge (e.g. 32-bit ints → 4-billion-size count array).
- **Radix sort:** fixed-width keys (32-bit ints, fixed-length strings, dates). Beats comparison sorts when `d` is small.

**Follow-ups:**
1. *"Doesn't this break the O(n log n) lower bound?"* → That bound is for *comparison* sorts. Counting/radix don't compare elements — they use keys as indices, so the bound doesn't apply.
2. *"Counting sort with negative numbers?"* → Offset by `-min` so indices start at 0.
3. *"Why right-to-left fill?"* → Combined with cumulative counts, it preserves input order of equal keys (stability) — essential for radix sort's correctness.

**Common mistakes:**
- Using counting sort for a huge value range → O(k) blows up memory.
- Non-stable digit sort inside radix → radix produces wrong results.
- Forgetting to offset negatives.

---

### Q34: Pancake Sorting
**Company:** Google, Amazon
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
You can only "flip" a prefix of the array (reverse `arr[0..k]`). Return a sequence of flip sizes that sorts the array.

**What interviewer is testing:**
A constructive selection-sort-style greedy: repeatedly bring the current max to the front, then flip it to its final position.

**Ideal Answer:**

For each target position from the end down: find the max in the unsorted prefix, flip it to the front (index 0), then flip the whole unsorted prefix so the max lands at its correct slot. Two flips per element.

```java
public List<Integer> pancakeSort(int[] arr) {
    List<Integer> flips = new ArrayList<>();
    for (int size = arr.length; size > 1; size--) {
        int maxIdx = 0;
        for (int i = 1; i < size; i++) if (arr[i] > arr[maxIdx]) maxIdx = i;
        if (maxIdx == size - 1) continue;            // already in place
        if (maxIdx != 0) {                           // bring max to front
            flip(arr, maxIdx);
            flips.add(maxIdx + 1);                   // flips are 1-indexed sizes
        }
        flip(arr, size - 1);                         // send max to its position
        flips.add(size);
    }
    return flips;
}
private void flip(int[] arr, int k) {                // reverse arr[0..k]
    int i = 0, j = k;
    while (i < j) { int t = arr[i]; arr[i++] = arr[j]; arr[j--] = t; }
}
```

**Dry run** — `[3,2,4,1]`: size4 maxIdx2(4)→flip(2)`[4,2,3,1]`add3, flip(3)`[1,3,2,4]`add4. size3 maxIdx1(3)→flip(1)`[3,1,2,4]`add2, flip(2)`[2,1,3,4]`add3. size2 maxIdx0(2)→flip(1)`[1,2,3,4]`add2. → flips `[3,4,2,3,2]`. ✅

**Complexity:** O(n²) time (n elements × O(n) max-find + flip), at most `2n` flips. Space O(1) plus output.

**Follow-ups:**
1. *"Minimum number of flips?"* → That's NP-hard in general (the "pancake number" problem); the 2n-flip greedy is the standard expected answer.
2. *"Why bring max to front first?"* → A single prefix-flip can only reverse a prefix; getting the max to index 0 lets the second flip drop it anywhere.

**Common mistakes:**
- Off-by-one in flip size (it's `index + 1`, a 1-indexed size).
- Forgetting the "already in place" skip (extra useless flips).

---

### Q35: Wiggle Sort II
**Company:** Google, Amazon, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Reorder `nums` so that `nums[0] < nums[1] > nums[2] < nums[3] ...` (strict). Duplicates allowed in input.

**What interviewer is testing:**
The non-obvious interleaving of the sorted halves and why naively interleaving fails with duplicates (the median must go to opposite ends).

**Ideal Answer:**

Sort, then split into a smaller half and a larger half. Fill the **even** indices with the smaller half **from its largest down**, and the **odd** indices with the larger half **from its largest down**. Filling from the largest of each half pushes equal medians to opposite ends, preventing adjacent equals.

```java
public void wiggleSort(int[] nums) {
    int n = nums.length;
    int[] sorted = nums.clone();
    Arrays.sort(sorted);

    int mid = (n - 1) / 2;          // index of the upper element of the smaller half
    int j = mid, k = n - 1;         // pointers into smaller half (j) and larger half (k)
    for (int i = 0; i < n; i++) {
        // even indices get the smaller half (descending), odd get the larger half (descending)
        nums[i] = (i % 2 == 0) ? sorted[j--] : sorted[k--];
    }
}
```

**Dry run** — `[1,5,1,1,6,4]` → sorted `[1,1,1,4,5,6]`, mid=2. j=2,k=5.
- i0(even):sorted[2]=1,j1. i1(odd):sorted[5]=6,k4. i2:sorted[1]=1,j0. i3:sorted[4]=5,k3. i4:sorted[0]=1,j-1. i5:sorted[3]=4. → `[1,6,1,5,1,4]`. Check: 1<6>1<5>1<4 ✓. ✅

**Complexity:** O(n log n) time (sort), O(n) space (the clone). An O(n)/O(1) version exists using QuickSelect for the median + virtual indexing, but it's intricate — mention it as the follow-up.

**Follow-ups:**
1. *"Why fill from the largest of each half?"* → If you filled smaller-half ascending into evens and larger-half ascending into odds, equal medians could land adjacent. Descending from the median pushes duplicates apart.
2. *"O(1) space?"* → Find the median with QuickSelect (O(n)), then 3-way partition around it, then apply an index-mapping `(1 + 2*i) % (n | 1)` to place in-place. Hard; describe the idea.

**Common mistakes:**
- Naive "sort then swap adjacent pairs" → fails with duplicates (e.g. `[1,1,2,2]`).
- Filling halves ascending → adjacent equal medians.

---

### Q36: Relative Sort Array
**Company:** Amazon, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** Phone

**Question:**
Sort `arr1` so elements appear in the relative order given by `arr2`; elements not in `arr2` go at the end in ascending order.

**What interviewer is testing:**
A custom comparator keyed by a rank map, or a counting-sort variant. Handling the "not in arr2" fallback cleanly.

**Ideal Answer:**

Since values are bounded, **counting sort** is cleanest: tally `arr1`, then emit in `arr2`'s order, then emit the leftovers in ascending value order.

```java
public int[] relativeSortArray(int[] arr1, int[] arr2) {
    int[] count = new int[1001];           // values bounded by problem constraints
    for (int x : arr1) count[x]++;
    int[] res = new int[arr1.length];
    int idx = 0;
    for (int x : arr2) {                   // emit in arr2's order
        while (count[x]-- > 0) res[idx++] = x;
    }
    for (int v = 0; v <= 1000; v++) {      // remaining values ascending
        while (count[v]-- > 0) res[idx++] = v;
    }
    return res;
}
```

**Dry run** — `arr1=[2,3,1,3,2,4,6,7,9,2,19]`, `arr2=[2,1,4,3,9,6]`: emit 2,2,2,1,4,3,3,9,6 then leftovers 7,19 → `[2,2,2,1,4,3,3,9,6,7,19]`. ✅

**Complexity:** O(n + m + maxVal) time, O(maxVal) space.

**Follow-ups:**
1. *"Values unbounded / could be large?"* → Use a `HashMap` rank for `arr2` and a comparator: rank if present else `Integer.MAX_VALUE`, tie-break by value. O(n log n).
2. *"Why does `count[x]--` after emitting work for the leftover loop?"* → Emitted values are decremented to 0, so the second loop only emits values never in `arr2`.

**Common mistakes:**
- Forgetting to sort the leftover (not-in-arr2) elements ascending.
- Hardcoding the wrong value bound.

---

### Q37: H-Index
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given `citations[i]` (citations of the i-th paper), return the researcher's **h-index**: the max `h` such that `h` papers each have ≥ `h` citations.

**What interviewer is testing:**
Either a sort + scan or an O(n) counting-bucket approach, and getting the definition's boundary exactly right.

**Ideal Answer:**

*Sort descending and scan:* after sorting descending, the largest `h` where `citations[h-1] >= h` is the answer — when the (1-indexed) paper count exceeds its citation count, stop.

```java
public int hIndex(int[] citations) {
    Arrays.sort(citations);                 // ascending
    int n = citations.length;
    for (int i = 0; i < n; i++) {
        int h = n - i;                      // number of papers with >= citations[i] citations
        if (citations[i] >= h) return h;    // first index where this holds gives the max h
    }
    return 0;
}
```

*O(n) counting buckets (better):* cap citation counts at `n` (h can't exceed `n`), bucket counts, then sweep from high to low accumulating papers until `papers >= h`.

```java
public int hIndexCounting(int[] citations) {
    int n = citations.length;
    int[] bucket = new int[n + 1];
    for (int c : citations) bucket[Math.min(c, n)]++;   // cap at n
    int papers = 0;
    for (int h = n; h >= 0; h--) {
        papers += bucket[h];
        if (papers >= h) return h;          // h papers each with >= h citations
    }
    return 0;
}
```

**Dry run** — `[3,0,6,1,5]` sorted `[0,1,3,5,6]`: i=0 h=5,0<5; i=1 h=4,1<4; i=2 h=3,3>=3 → return 3. ✅

**Complexity:** Sort approach O(n log n)/O(1); counting approach O(n)/O(n).

**Follow-ups:**
1. *"Citations already sorted ascending — H-Index II?"* → Binary search: find smallest `i` with `citations[i] >= n - i`; answer `n - i`. O(log n).
2. *"Why cap citations at n?"* → h ≤ total papers = n; citations beyond n don't change the bucket sweep.

**Common mistakes:**
- Off-by-one on `h = n - i`.
- Returning the citation value instead of the count `h`.

---

### Q38: Maximum Gap (bucket sort)
**Company:** Amazon, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Given an unsorted array, return the maximum difference between two **successive** elements in its sorted form. Must run in **linear time and space** (so you can't just sort comparison-style).

**What interviewer is testing:**
The pigeonhole + **bucket sort** insight: the max gap can't be smaller than `ceil((max-min)/(n-1))`, so if you bucket by that width, the max gap must occur *between* adjacent non-empty buckets — you only need each bucket's min and max, never a full sort.

**Ideal Answer:**

With `n` numbers spanning `[min, max]`, by pigeonhole the largest gap is at least `gap = ceil((max-min)/(n-1))`. Create buckets of that width. Two numbers in the *same* bucket differ by less than `gap`, so the maximum gap is always between the **max of one bucket and the min of the next non-empty bucket**. Track only each bucket's min/max.

```java
public int maximumGap(int[] nums) {
    int n = nums.length;
    if (n < 2) return 0;
    int min = Integer.MAX_VALUE, max = Integer.MIN_VALUE;
    for (int x : nums) { min = Math.min(min, x); max = Math.max(max, x); }
    if (min == max) return 0;                       // all equal

    int bucketSize = Math.max(1, (max - min) / (n - 1));   // gap lower bound
    int bucketCount = (max - min) / bucketSize + 1;
    int[] bucketMin = new int[bucketCount];
    int[] bucketMax = new int[bucketCount];
    Arrays.fill(bucketMin, Integer.MAX_VALUE);
    Arrays.fill(bucketMax, Integer.MIN_VALUE);

    for (int x : nums) {
        int b = (x - min) / bucketSize;             // which bucket
        bucketMin[b] = Math.min(bucketMin[b], x);
        bucketMax[b] = Math.max(bucketMax[b], x);
    }

    int maxGap = 0, prevMax = min;                  // last non-empty bucket's max
    for (int b = 0; b < bucketCount; b++) {
        if (bucketMin[b] == Integer.MAX_VALUE) continue;   // empty bucket
        maxGap = Math.max(maxGap, bucketMin[b] - prevMax); // gap across the empty span
        prevMax = bucketMax[b];
    }
    return maxGap;
}
```

**Dry run** — `[3,6,9,1]`: min1,max9,n4 → bucketSize=(8)/3=2, buckets cover [1,2],[3,4],[5,6],[7,8],[9..]. Numbers land: 1→b0,3→b1,6→b2 (5-6),9→b4. Sweep gaps: (3-1)=2,(6-3)? 6 in b2 min6, prevMax3 → 3,(9-6)=3 → maxGap 3. Sorted `[1,3,6,9]` successive diffs 2,3,3 → 3. ✅

**Complexity:** O(n) time, O(n) space. Beats the O(n log n) sort-and-scan and satisfies the linear requirement.

**Follow-ups:**
1. *"Why is the answer never inside a bucket?"* → Bucket width ≤ the gap lower bound, so two numbers in one bucket differ by less than the guaranteed max gap. The max gap must straddle buckets.
2. *"Radix sort alternative?"* → Also O(n) (Q32) — sort then scan adjacent diffs. Bucketing is the more elegant pigeonhole argument.
3. *"Why `n-1` buckets-worth of width?"* → `n` numbers create `n-1` gaps; spreading them as evenly as possible gives each gap `(max-min)/(n-1)` — the lower bound on the largest.

**Common mistakes:**
- `bucketSize = 0` when `(max-min) < n-1` → use `Math.max(1, ...)`.
- Comparing within buckets (unnecessary and breaks linearity).
- Forgetting the all-equal / single-element edge cases.

---

# GREEDY

> Greedy is the trap family: a greedy solution is short and often *feels* right but is only correct if a local choice provably leads to a global optimum. The skill interviewers grade is **the exchange argument** — "if any optimal solution differs from my greedy choice, I can swap to match my greedy choice without making it worse." If you can't justify the greedy, prefer DP. Below, each problem names *why* the greedy is safe; saying that proof out loud is the difference between hire and no-hire. (Jump Game / Jump Game II are covered in file 01; Task Scheduler, Reorganize String, and Meeting Rooms live in other files — reference them, don't repeat.)

---

### Q39: Non-overlapping Intervals (sort by end)
**Company:** Amazon, Google, Microsoft, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given intervals, return the **minimum number to remove** so the rest don't overlap. (Equivalent: max number of non-overlapping intervals you can keep = activity selection.)

**What interviewer is testing:**
The canonical greedy: **sort by end time, always keep the interval that finishes earliest.** Knowing *why* end-time sorting (not start) is correct.

**Ideal Answer:**

Sort by **end** time. Greedily keep an interval if it starts at/after the last kept interval's end; otherwise it overlaps → remove it. Choosing the earliest-finishing compatible interval leaves the most room for future intervals.

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) return 0;
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));   // by END time
    int kept = 1, lastEnd = intervals[0][1];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= lastEnd) {     // no overlap -> keep it
            kept++;
            lastEnd = intervals[i][1];
        }
        // else it overlaps -> "remove" (skip), lastEnd unchanged
    }
    return intervals.length - kept;           // removals = total - kept
}
```

**Dry run** — `[[1,2],[2,3],[3,4],[1,3]]` sorted by end → `[[1,2],[2,3],[1,3],[3,4]]`: keep [1,2] end2; [2,3] 2>=2 keep end3; [1,3] 1<3 skip; [3,4] 3>=3 keep end4. kept=3, removals = 4 - 3 = 1. ✅

**Complexity:** O(n log n) time (sort), O(1) space.

**Exchange argument (say this):** "If an optimal solution doesn't pick the earliest-finishing interval, I can swap that interval in — it ends no later, so it can't conflict with anything the optimal kept afterward. So picking earliest-finish is always safe."

**Follow-ups:**
1. *"Why sort by end, not start?"* → Earliest finish leaves maximum remaining time. Sorting by start can pick a long interval that blocks many short ones. Counter-example: `[1,100],[2,3],[3,4]` — start-sort keeps [1,100] (1 interval), end-sort keeps [2,3],[3,4] (2).
2. *"Touching intervals `[1,2],[2,3]` overlap?"* → Usually *not* (use `>=`). If the problem says touching is overlap, use `>`.
3. *"Max non-overlapping intervals (activity selection)?"* → That's exactly `kept`.

**Common mistakes:**
- Sorting by start → wrong on the blocking counter-example.
- `>` vs `>=` confusion at touching boundaries.

---

### Q40: Minimum Number of Arrows to Burst Balloons
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Balloons as `[start, end]` on the x-axis. An arrow at `x` bursts every balloon with `start ≤ x ≤ end`. Return the minimum arrows to burst all.

**What interviewer is testing:**
Same end-sort greedy as Q37, framed as "max overlapping groups." Shoot an arrow at the end of the earliest-finishing balloon to also catch all that overlap it.

**Ideal Answer:**

Sort by end. Place an arrow at the first balloon's end; it bursts all balloons that start ≤ that end. Skip those; when a balloon starts after the current arrow, fire a new arrow at its end.

```java
public int findMinArrowShots(int[][] points) {
    if (points.length == 0) return 0;
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));   // by END
    int arrows = 1;
    long arrowX = points[0][1];                  // long: coordinates can be Integer.MIN/MAX
    for (int[] p : points) {
        if (p[0] > arrowX) {                     // this balloon starts after the arrow -> need a new one
            arrows++;
            arrowX = p[1];
        }
    }
    return arrows;
}
```

**Dry run** — `[[10,16],[2,8],[1,6],[7,12]]` sorted by end → `[[1,6],[2,8],[7,12],[10,16]]`: arrow at 6 (bursts [1,6],[2,8] since 2<=6); [7,12] 7>6 → arrow at 12 (bursts [7,12],[10,16]); → 2 arrows. ✅

**Complexity:** O(n log n) time, O(1) space.

**Follow-ups:**
1. *"Why sort by end and shoot at the end?"* → The earliest end is the rightmost point guaranteed inside that balloon; shooting there maximizes coverage of overlapping balloons.
2. *"Why `long arrowX`?"* → Coordinates can be `Integer.MAX_VALUE`; comparisons are fine but using `long` avoids any overflow if you do arithmetic.
3. *"Relation to Q37?"* → Min arrows = number of groups of mutually overlapping intervals = `n - (max non-overlapping kept)` is *not* it; it's the count of arrow groups. Both use end-sort.

**Common mistakes:**
- Using `>=` instead of `>` (touching balloons `[1,6],[6,9]` *share* x=6, one arrow suffices → must use `>`).
- Sorting by start.

---

### Q41: Job Sequencing with Deadlines
**Company:** Amazon, Flipkart, Samsung
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Each job has a `deadline` and `profit`; each job takes one unit of time and only one job runs at a time. Maximize total profit by scheduling jobs to finish by their deadlines. Return `[jobsDone, maxProfit]`.

**What interviewer is testing:**
Greedy by profit descending + scheduling each job in the latest free slot before its deadline (union-find or a boolean slot array).

**Ideal Answer:**

Sort jobs by **profit descending**. For each job, try to place it in the latest free time slot `≤ its deadline` (latest, to leave earlier slots open for tighter-deadline jobs). A boolean slot array (size = max deadline) tracks occupancy.

```java
public int[] jobScheduling(int[][] jobs) {     // jobs[i] = {deadline, profit}
    Arrays.sort(jobs, (a, b) -> Integer.compare(b[1], a[1]));   // profit DESC
    int maxDeadline = 0;
    for (int[] j : jobs) maxDeadline = Math.max(maxDeadline, j[0]);

    boolean[] slot = new boolean[maxDeadline + 1];   // slot[t] = is time t taken (1-indexed)
    int count = 0, profit = 0;
    for (int[] j : jobs) {
        for (int t = j[0]; t >= 1; t--) {            // latest free slot <= deadline
            if (!slot[t]) {
                slot[t] = true;
                count++;
                profit += j[1];
                break;
            }
        }
    }
    return new int[]{count, profit};
}
```

**Dry run** — jobs `{(2,100),(1,19),(2,27),(1,25),(3,15)}` sorted by profit: (2,100),(2,27),(1,25),(1,19),(3,15). Place 100 at t2; 27 at t1; 25 → t1 taken, no slot; 19 → no slot; 15 at t3. → count 3, profit 142. ✅

**Complexity:** O(n log n + n × maxDeadline) with the array; O(n log n × α) with a union-find "find next free slot." Space O(maxDeadline).

**Follow-ups:**
1. *"Speed up the slot search?"* → Union-find: `parent[t]` points to the latest free slot ≤ t; near-O(1) amortized per job → O(n log n) total.
2. *"Why place in the latest slot, not earliest?"* → Reserving earlier slots for jobs with tighter deadlines maximizes how many high-profit jobs fit.
3. *"Jobs with arbitrary durations?"* → No longer this clean greedy; becomes weighted scheduling (DP / more complex).

**Common mistakes:**
- Placing jobs in the earliest slot (blocks tight-deadline jobs).
- Sorting by deadline instead of profit.
- Off-by-one on 1-indexed slots.

---

### Q42: Fractional Knapsack
**Company:** Amazon, Flipkart, Samsung
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Items with `(value, weight)`; capacity `W`. You may take **fractions** of items. Maximize total value.

**What interviewer is testing:**
Recognizing that *fractional* (unlike 0/1) knapsack is **greedy** — take items by value-per-weight ratio, splitting the last one. And knowing why 0/1 is NOT greedy (it's DP).

**Ideal Answer:**

Sort by `value/weight` ratio descending. Take whole items greedily; when the next item doesn't fit, take the fraction that fills the remaining capacity. The ratio-greedy is optimal *because* you can split — there's no "wasted partial slot."

```java
public double fractionalKnapsack(int[][] items, int W) {   // items[i] = {value, weight}
    // Sort by value/weight descending. Use cross-multiplication to avoid float compare issues.
    Arrays.sort(items, (a, b) -> Long.compare((long) b[0] * a[1], (long) a[0] * b[1]));

    double total = 0;
    int cap = W;
    for (int[] it : items) {
        int value = it[0], weight = it[1];
        if (weight <= cap) {                 // take the whole item
            total += value;
            cap -= weight;
        } else {                             // take the fraction that fits, then stop
            total += value * ((double) cap / weight);
            break;
        }
    }
    return total;
}
```

**Dry run** — items `{(60,10),(100,20),(120,30)}`, W=50. Ratios 6, 5, 4. Take (60,10) cap40; (100,20) cap20; (120,30) fraction 20/30 → +80. Total 240. ✅

**Complexity:** O(n log n) time (sort), O(1) space.

**Follow-ups:**
1. *"Why is 0/1 knapsack not solvable greedily?"* → You can't split items, so a high-ratio item might leave wasted capacity that a different combination fills better. 0/1 needs DP (O(nW)). This contrast is the key signal.
2. *"Why cross-multiply instead of comparing `a[0]/a[1]` as doubles?"* → Avoids floating-point precision/division-by-zero and is exact. (Watch overflow → use `long`.)
3. *"Items with the same ratio?"* → Order among them doesn't matter for the total value.

**Common mistakes:**
- Applying the ratio-greedy to **0/1** knapsack (wrong — produces suboptimal answers).
- Float comparison of ratios → precision bugs; cross-multiply.
- Forgetting to `break` after the fractional take.

---

### Q43: Minimum Number of Platforms
**Company:** Amazon, Flipkart, Goldman Sachs
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given train `arrivals[]` and `departures[]`, find the minimum number of platforms so no train waits. (Same as Meeting Rooms II, framed as trains.)

**What interviewer is testing:**
The sweep-line / two-pointer over sorted arrival and departure times, tracking max concurrent trains.

**Ideal Answer:**

Sort arrivals and departures separately. Sweep with two pointers: on an arrival (the next event chronologically), a platform is needed (`platforms++`); when the earliest departure has passed (`dep[j] < arr[i]`), free one (`platforms--`). Track the maximum.

```java
public int minPlatforms(int[] arr, int[] dep) {
    Arrays.sort(arr);
    Arrays.sort(dep);
    int n = arr.length;
    int platforms = 0, maxPlatforms = 0;
    int i = 0, j = 0;
    while (i < n) {
        if (arr[i] <= dep[j]) {       // a train arrives before/at the next departure -> need platform
            platforms++;
            maxPlatforms = Math.max(maxPlatforms, platforms);
            i++;
        } else {                       // a train has departed -> free a platform
            platforms--;
            j++;
        }
    }
    return maxPlatforms;
}
```

**Dry run** — arr `[900,940,950,1100,1500,1800]`, dep `[910,1200,1120,1130,1900,2000]` sorted dep `[910,1120,1130,1200,1900,2000]`: peak concurrency reaches 3 (e.g. 940,950 overlap before 910? trace yields max 3). ✅

**Complexity:** O(n log n) time, O(1) space.

**Follow-ups:**
1. *"Why `<=` for arrival vs departure (a train arriving exactly when another departs)?"* → If a platform must be vacated *before* the next train uses it, treat same-time arrival as needing a new platform (`<=`). Clarify the convention with the interviewer.
2. *"This is which other problem?"* → Meeting Rooms II — identical. Name it.
3. *"Return *which* platform each train uses?"* → Min-heap of (departureTime, platformId), reuse freed IDs.

**Common mistakes:**
- Pairing `arr[i]` with `dep[i]` (wrong — must sort independently; arrivals and departures are decoupled events).
- `<` vs `<=` boundary ambiguity unaddressed.

---

### Q44: Candy
**Company:** Amazon, Google, Microsoft, Flipkart
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
`n` children in a row with `ratings[]`. Each gets ≥1 candy; a child with a higher rating than an immediate neighbor must get more candy than that neighbor. Minimize total candy.

**What interviewer is testing:**
The **two-pass greedy**: left-to-right satisfies the left neighbor constraint, right-to-left the right neighbor; combine with `max`.

**Ideal Answer:**

Each child must satisfy *both* neighbor constraints. Do two passes:
- Left→right: if `ratings[i] > ratings[i-1]`, give `candy[i] = candy[i-1] + 1`.
- Right→left: if `ratings[i] > ratings[i+1]`, give `candy[i] = max(candy[i], candy[i+1] + 1)`.

The `max` ensures both directions are satisfied with the minimum.

```java
public int candy(int[] ratings) {
    int n = ratings.length;
    int[] candy = new int[n];
    Arrays.fill(candy, 1);                       // everyone starts with 1

    for (int i = 1; i < n; i++)                  // left -> right
        if (ratings[i] > ratings[i - 1]) candy[i] = candy[i - 1] + 1;

    for (int i = n - 2; i >= 0; i--)             // right -> left
        if (ratings[i] > ratings[i + 1]) candy[i] = Math.max(candy[i], candy[i + 1] + 1);

    int total = 0;
    for (int c : candy) total += c;
    return total;
}
```

**Dry run** — `[1,0,2]`: after L→R `[1,1,2]`; after R→L: i1 0>2? no; i0 1>0? yes → max(1, 1+1)=2 → `[2,1,2]`. Total 5. ✅

**Complexity:** O(n) time, O(n) space. (An O(1)-space single-pass version using up/down slope counting exists — mention it as a follow-up.)

**Follow-ups:**
1. *"Why two passes, why `max`?"* → One pass only respects one neighbor. The left pass handles increasing-from-left runs; the right pass handles decreasing runs; `max` reconciles a child that's a local peak between both.
2. *"O(1) space?"* → Track up-slope length, down-slope length, and peak length; add the arithmetic-series candy counts. Trickier but constant space.
3. *"Equal adjacent ratings?"* → No constraint between equal neighbors — they can both keep 1 (the conditions are strict `>`).

**Common mistakes:**
- Single pass → fails on valleys/peaks like `[1,3,2,1]`.
- Forgetting `max` in the right pass → overwrites the left-pass result and breaks the left constraint.

---

### Q45: Gas Station
**Company:** Amazon, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Circular route of `n` stations; `gas[i]` is fuel at station `i`, `cost[i]` is fuel to reach `i+1`. Start with an empty tank. Return the starting index to complete the circuit once, or -1 if impossible. Unique answer if it exists.

**What interviewer is testing:**
The two greedy insights: (1) if total gas ≥ total cost, a solution exists; (2) if the running tank goes negative at station `i`, no start in `[start, i]` works — jump start to `i+1`.

**Ideal Answer:**

Track `total` (overall surplus) and a `tank` since the candidate start. Whenever `tank` drops below 0, the candidate start can't reach here, so reset start to the next station and zero the tank. If `total >= 0`, the final candidate start is the answer.

```java
public int canCompleteCircuit(int[] gas, int[] cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < gas.length; i++) {
        int diff = gas[i] - cost[i];
        total += diff;
        tank += diff;
        if (tank < 0) {        // can't reach i+1 from current start
            start = i + 1;     // new candidate start
            tank = 0;          // reset tank for the new start
        }
    }
    return total >= 0 ? start : -1;   // feasible iff total surplus is non-negative
}
```

**Dry run** — `gas=[1,2,3,4,5]`, `cost=[3,4,5,1,2]`: diffs `[-2,-2,-2,3,3]`. i0 tank-2<0 start1 tank0; i1 tank-2<0 start2 tank0; i2 tank-2<0 start3 tank0; i3 tank3; i4 tank6. total = 1 ≥0 → start 3. ✅

**Complexity:** O(n) time, O(1) space. (Single pass — beats the O(n²) "try every start.")

**Follow-ups:**
1. *"Why does resetting `start` to `i+1` not skip a valid start?"* → If the tank went negative at `i` starting from `start`, then *any* station `j` in `[start, i]` also can't reach `i+1` (the prefix sum from `j` is even smaller plus the deficit). So none of them can be the answer.
2. *"Why does `total >= 0` guarantee the final `start` works?"* → If a solution exists, it's unique, and the greedy's surviving start is it; the `total` check confirms feasibility.
3. *"Multiple valid starts?"* → The problem guarantees uniqueness; otherwise you'd enumerate.

**Common mistakes:**
- O(n²) brute force trying every start → TLE.
- Resetting `tank` but not `start`, or vice versa.
- Returning `start` without the `total >= 0` feasibility check.

---

### Q46: Partition Labels
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Partition string `s` into as many parts as possible so each letter appears in at most one part. Return the list of part sizes.

**What interviewer is testing:**
Precompute each character's **last occurrence**, then greedily extend the current partition's end to the farthest last-occurrence seen — cut when the scan index reaches that end.

**Ideal Answer:**

Record the last index of each character. Sweep; keep extending `end` to `max(end, lastIndex[c])`. When `i == end`, every character in this window appears nowhere later → cut here.

```java
public List<Integer> partitionLabels(String s) {
    int[] last = new int[26];
    for (int i = 0; i < s.length(); i++) last[s.charAt(i) - 'a'] = i;   // last occurrence

    List<Integer> result = new ArrayList<>();
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, last[s.charAt(i) - 'a']);   // extend to farthest needed
        if (i == end) {                                  // partition complete
            result.add(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

**Dry run** — `"ababcbacadefegdehijhklij"`: last('a')=8 etc. Scan extends end to 8 (the 'a','b','c' cluster) → cut at 9 (size 9); next cluster 'd..h' ends at 15 → size 7; 'i..j' → size 8. → `[9,7,8]`. ✅

**Complexity:** O(n) time, O(1) space (26-letter array).

**Follow-ups:**
1. *"Why greedy correct?"* → A partition can't end before the last occurrence of any character it contains; extending `end` to that maximum is forced, and cutting at the first index where the window self-contains gives the *most* parts.
2. *"Return the substrings, not sizes?"* → Slice `s.substring(start, end+1)`.
3. *"Unicode / arbitrary alphabet?"* → Use a `HashMap<Character,Integer>` for last indices.

**Common mistakes:**
- Cutting at the first repeated char instead of the farthest last-occurrence.
- Off-by-one in the size (`end - start + 1`).

---

### Q47: Queue Reconstruction by Height
**Company:** Amazon, Google, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
People as `[h, k]`: height `h` and `k` = number of people in front who are taller-or-equal. Reconstruct the queue order.

**What interviewer is testing:**
The greedy insight: sort by height **descending**, ties by `k` ascending, then **insert each person at index `k`**. Shorter people inserted later don't disturb the `k` count of already-placed taller people.

**Ideal Answer:**

Place tallest first. When inserting a person at position `k`, all already-placed people are taller-or-equal, so position = `k` exactly satisfies their requirement. Later (shorter) insertions slide them right but don't affect their "taller in front" count.

```java
public int[][] reconstructQueue(int[][] people) {
    // Tallest first; among equal heights, smaller k first.
    Arrays.sort(people, (a, b) -> a[0] != b[0]
            ? Integer.compare(b[0], a[0])      // height DESC
            : Integer.compare(a[1], b[1]));    // k ASC

    List<int[]> queue = new ArrayList<>();
    for (int[] p : people) queue.add(p[1], p);  // insert at index k
    return queue.toArray(new int[0][]);
}
```

**Dry run** — `[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]` sorted → `[[7,0],[7,1],[6,1],[5,0],[5,2],[4,4]]`. Insert: [7,0]@0; [7,1]@1; [6,1]@1; [5,0]@0; [5,2]@2; [4,4]@4 → `[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]`. ✅

**Complexity:** O(n²) time (n inserts into an ArrayList, each O(n) shift), O(n) space. (A Fenwick-tree placement can reach O(n log n) — mention it.)

**Follow-ups:**
1. *"Why height descending then insert at k?"* → After placing all taller people, inserting the next at index `k` puts exactly `k` taller-or-equal in front; subsequent shorter inserts can only push it rightward without adding taller people in front.
2. *"O(n log n)?"* → Sort shortest-first and use a BIT to find the (k+1)-th empty slot. Harder; describe the idea.

**Common mistakes:**
- Sorting ascending by height (the invariant breaks).
- Wrong tie-break (`k` must be ascending among equal heights).

---

### Q48: Assign Cookies
**Company:** Amazon, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** OA / Phone

**Question:**
Children have greed factors `g[]`; cookies have sizes `s[]`. A child `i` is content if assigned a cookie `j` with `s[j] >= g[i]`. Maximize the number of content children.

**What interviewer is testing:**
Sort both, two-pointer greedy: give the smallest sufficient cookie to the least greedy child.

**Ideal Answer:**

Sort greed and cookies ascending. Walk both; if the current cookie satisfies the current child, content them and advance both; else try the next bigger cookie.

```java
public int findContentChildren(int[] g, int[] s) {
    Arrays.sort(g);
    Arrays.sort(s);
    int child = 0, cookie = 0;
    while (child < g.length && cookie < s.length) {
        if (s[cookie] >= g[child]) child++;   // this cookie satisfies the child
        cookie++;                              // either way, move past this cookie
    }
    return child;                              // number of content children
}
```

**Dry run** — `g=[1,2,3]`, `s=[1,1]`: child0 needs1, cookie0=1>=1 → child1,cookie1; cookie1=1>=2? no → cookie2 end. → 1. ✅

**Complexity:** O(n log n + m log m) time (sorts), O(1) space.

**Follow-ups:**
1. *"Why give the smallest sufficient cookie?"* → Saving bigger cookies for greedier children maximizes the count (exchange argument).
2. *"Each child can get multiple cookies summing to greed?"* → Different problem (becomes a packing variant).

**Common mistakes:**
- Not sorting both arrays.
- Advancing the child pointer even when the cookie is too small.

---

### Q49: Lemonade Change
**Company:** Amazon, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** OA / Phone

**Question:**
Lemonade costs $5. Customers pay with $5, $10, or $20 bills (in order). You start with no change. Return whether you can give correct change to everyone.

**What interviewer is testing:**
Greedy change-making: when giving change for a $20, prefer a ($10 + $5) over (three $5s) to conserve flexible $5 bills.

**Ideal Answer:**

Track counts of $5 and $10 bills. For $20, prefer to use one $10 + one $5 (keeps more $5s for future $10-payers); fall back to three $5s.

```java
public boolean lemonadeChange(int[] bills) {
    int five = 0, ten = 0;
    for (int bill : bills) {
        if (bill == 5) {
            five++;
        } else if (bill == 10) {
            if (five == 0) return false;     // need a $5 as change
            five--; ten++;
        } else {                             // $20: give $15 change
            if (ten > 0 && five > 0) {       // prefer $10 + $5
                ten--; five--;
            } else if (five >= 3) {          // fall back to three $5s
                five -= 3;
            } else {
                return false;
            }
        }
    }
    return true;
}
```

**Dry run** — `[5,5,5,10,20]`: five→1,2,3; $10→five2,ten1; $20→ten0,five1. true. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why prefer $10+$5 over three $5s for a $20?"* → $5 bills are the only change for $10 payers; conserving them is strictly safer (exchange argument).
2. *"Could a non-greedy choice ever win?"* → No — giving away $5s when a $10 was available can only hurt future $10 customers.

**Common mistakes:**
- Using three $5s when a $10 is available (depletes the flexible $5s).
- Forgetting you never have change to break a $5 (it's the base unit).

---

### Q50: Minimum Deletions to Make Character Frequencies Unique
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Delete the minimum number of characters so that no two distinct characters have the same frequency. Return the deletion count.

**What interviewer is testing:**
Greedy with a "seen frequencies" set: for each frequency, decrement until it's unique (or zero), counting deletions.

**Ideal Answer:**

Count character frequencies. Sort frequencies descending (so larger ones keep their value and smaller ones get reduced). For each, while the frequency is already used and > 0, decrement it (one deletion each), then mark it used.

```java
public int minDeletions(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;

    Set<Integer> used = new HashSet<>();
    int deletions = 0;
    for (int f : freq) {
        while (f > 0 && used.contains(f)) {   // collision -> delete one occurrence
            f--;
            deletions++;
        }
        if (f > 0) used.add(f);
    }
    return deletions;
}
```

**Dry run** — `"aaabbbcc"`: freqs a=3,b=3,c=2. a:3 not used → used{3}. b:3 used → 2 used → del1 → used{3,2}. c:2 used → 1 → del1 → used{3,2,1}. Total 2. ✅

**Complexity:** O(n + K²) worst (K=26 alphabet; the while loop bounded), effectively O(n). Space O(K).

**Follow-ups:**
1. *"Why is decrementing greedy correct?"* → Each frequency only needs to drop to the next free integer; deleting more would be wasteful, deleting from a higher-frequency char isn't needed since we process and reserve values.
2. *"Does processing order matter?"* → Using a `used` set and decrementing handles any order correctly; sorting descending is a common, clear framing.

**Common mistakes:**
- Stopping the decrement at the wrong condition (must continue while the value is taken *and* > 0).
- Adding 0 to the used set (frequency 0 means the char is gone — multiple can be 0).

---

### Q51: Boats to Save People
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
`people[i]` is a weight; each boat carries at most 2 people with total weight ≤ `limit`. Return the minimum number of boats.

**What interviewer is testing:**
Sort + two-pointer greedy: pair the heaviest with the lightest if they fit, else the heaviest goes alone.

**Ideal Answer:**

Sort ascending. Two pointers from both ends. If the lightest + heaviest ≤ limit, they share a boat (advance both); otherwise the heaviest goes alone (advance the heavy pointer). Each step uses one boat.

```java
public int numRescueBoats(int[] people, int limit) {
    Arrays.sort(people);
    int lo = 0, hi = people.length - 1, boats = 0;
    while (lo <= hi) {
        if (people[lo] + people[hi] <= limit) lo++;   // lightest pairs with heaviest
        hi--;                                          // heaviest always boards now
        boats++;
    }
    return boats;
}
```

**Dry run** — `people=[3,2,2,1]`, limit=3 → sorted `[1,2,2,3]`. lo0hi3: 1+3=4>3 → hi2,boats1. lo0hi2: 1+2=3<=3 → lo1,hi1,boats2. lo1hi1: 2+2=4>3 → hi0,boats3. → 3 boats. ✅

**Complexity:** O(n log n) time (sort), O(1) space.

**Follow-ups:**
1. *"Why pair lightest with heaviest?"* → If the heaviest can't pair with even the lightest, it can't pair with anyone → it boards alone. If it can pair with the lightest, doing so is optimal (the lightest is hardest to pair otherwise). Exchange argument.
2. *"Boats hold up to 3 people?"* → The clean two-pointer breaks; becomes harder (greedy + cases or DP).

**Common mistakes:**
- Pairing two heavy people (wastes capacity).
- Forgetting that the heavy pointer *always* decrements (the heaviest boards every iteration).

---

### Q52: Hand of Straights / Divide Array in Sets of K Consecutive Numbers
**Company:** Google, Amazon
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Can the array be rearranged into groups of `groupSize` consecutive integers each? (Hand of Straights uses `W`; the LeetCode "K consecutive" variant uses `k`.) Return boolean.

**What interviewer is testing:**
Greedy with a sorted frequency map (TreeMap): always start a group from the smallest remaining number and consume the next `groupSize - 1` consecutive values.

**Ideal Answer:**

If `n % groupSize != 0`, impossible. Otherwise use a TreeMap (sorted counts). Repeatedly take the smallest available number as a group start; decrement it and the next `groupSize-1` consecutive numbers. If any is missing, fail.

```java
public boolean isPossibleDivide(int[] nums, int groupSize) {
    if (nums.length % groupSize != 0) return false;
    TreeMap<Integer, Integer> count = new TreeMap<>();
    for (int x : nums) count.merge(x, 1, Integer::sum);

    while (!count.isEmpty()) {
        int start = count.firstKey();             // smallest remaining must start a group
        for (int v = start; v < start + groupSize; v++) {
            Integer c = count.get(v);
            if (c == null) return false;          // missing consecutive value -> impossible
            if (c == 1) count.remove(v);
            else count.put(v, c - 1);
        }
    }
    return true;
}
```

**Dry run** — `nums=[1,2,3,6,2,3,4,7,8]`, groupSize=3: smallest 1 → group {1,2,3}; smallest 2 → {2,3,4}; smallest 6 → {6,7,8}. All consumed → true. ✅

**Complexity:** O(n log n) time (TreeMap ops), O(n) space.

**Follow-ups:**
1. *"Why must the smallest always start a group?"* → The smallest number has no smaller neighbor to be the *middle/end* of a group, so it must be a group's start. Forced choice → greedy is safe.
2. *"groupSize == 1?"* → Always true (each element its own group).
3. *"Speed up?"* → Same idea with a plain HashMap and a sorted unique-keys list; TreeMap is cleanest.

**Common mistakes:**
- Not checking `n % groupSize == 0` first.
- Starting groups from arbitrary numbers instead of the smallest.

---

### Q53: Maximum Units on a Truck
**Company:** Amazon
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
`boxTypes[i] = [numberOfBoxes, unitsPerBox]`; truck holds at most `truckSize` boxes. Maximize total units loaded.

**What interviewer is testing:**
Trivial greedy: load boxes with the most units-per-box first. A warm-up to confirm you sort by the right key.

**Ideal Answer:**

Sort by `unitsPerBox` descending; greedily take boxes until the truck is full.

```java
public int maximumUnits(int[][] boxTypes, int truckSize) {
    Arrays.sort(boxTypes, (a, b) -> Integer.compare(b[1], a[1]));   // units/box DESC
    int units = 0;
    for (int[] box : boxTypes) {
        int take = Math.min(box[0], truckSize);   // can't exceed remaining capacity
        units += take * box[1];
        truckSize -= take;
        if (truckSize == 0) break;
    }
    return units;
}
```

**Dry run** — `[[1,3],[2,2],[3,1]]`, truckSize=4: sorted by units → `[[1,3],[2,2],[3,1]]`. Take 1×3=3 (cap3); 2×2=4 (cap1); 1×1=1 (cap0). Total 8. ✅

**Complexity:** O(n log n) time, O(1) space.

**Follow-ups:**
1. *"Why is greedy obviously correct here?"* → Every box slot is interchangeable; filling slots with the highest unit value maximizes the total — pure exchange argument, no fractional issue.
2. *"Boxes have weights/sizes too?"* → Becomes a knapsack variant; greedy no longer optimal.

**Common mistakes:**
- Sorting by `numberOfBoxes` instead of `unitsPerBox`.
- Forgetting `Math.min` with remaining capacity (overfilling the truck).

---

### Q54: Two City Scheduling
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
`2n` people; `costs[i] = [costToA, costToB]`. Send exactly `n` to city A and `n` to city B. Minimize total cost.

**What interviewer is testing:**
The clever greedy: send everyone to A, then "refund" by sending the `n` people with the largest savings `(costB - costA)` to B instead. Equivalent: sort by `costA - costB`.

**Ideal Answer:**

Sort people by `costA - costB` ascending. The first `n` (most negative → cheaper in A relative to B) go to A; the rest go to B. Negative `costA - costB` means A is cheaper for that person.

```java
public int twoCitySchedCost(int[][] costs) {
    // Ascending by (costA - costB): those most advantageous in A come first.
    Arrays.sort(costs, (a, b) -> Integer.compare(a[0] - a[1], b[0] - b[1]));
    int n = costs.length / 2;
    int total = 0;
    for (int i = 0; i < costs.length; i++) {
        total += (i < n) ? costs[i][0] : costs[i][1];   // first n -> A, rest -> B
    }
    return total;
}
```

**Dry run** — `[[10,20],[30,200],[400,50],[30,20]]`, n=2. Diffs: -10, -170, 350, 10. Sorted by diff → `[[30,200],[10,20],[30,20],[400,50]]`. First 2 → A: 30+10=40; last 2 → B: 20+50=70. Total 110. ✅

**Complexity:** O(n log n) time, O(1) space.

**Follow-ups:**
1. *"Why sort by `costA - costB`?"* → Send all to A as a baseline (`Σ costA`); switching person `i` to B changes total by `costB - costA`. To minimize, switch the `n` people with the *smallest* `costB - costA` (= largest savings) → equivalently keep the `n` smallest `costA - costB` in A. The sort encodes exactly this.
2. *"DP alternative?"* → `dp[i][a]` = min cost for first i people with a sent to A; O(n²). The greedy is O(n log n) and slicker.

**Common mistakes:**
- Sorting by `costA` or `costB` alone (ignores the *relative* advantage).
- Off-by-one splitting at `n`.

---

### Q55: Minimum Cost to Connect Sticks / Ropes
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
You have sticks with positive lengths. Combining two sticks of lengths `x` and `y` costs `x + y` and yields a stick of length `x + y`. Combine all sticks into one; return the **minimum** total cost. (Same as "Minimum Cost to Connect Ropes" — very common in Amazon OAs.)

**What interviewer is testing:**
Recognizing this as **Huffman-style greedy**: always combine the two *smallest* available sticks (a min-heap). Smaller sticks get combined earlier so their lengths are counted in *more* of the later sums — combining them first minimizes total counting.

**Ideal Answer:**

Use a min-heap. Repeatedly poll the two smallest, add their sum to the cost, and push the combined stick back. The smallest sticks should participate in the fewest combinations' running totals — which is exactly what "combine smallest first" achieves (each combination's cost cascades into all later sums).

```java
public int connectSticks(int[] sticks) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();   // min-heap
    for (int s : sticks) heap.add(s);

    int totalCost = 0;
    while (heap.size() > 1) {
        int a = heap.poll();          // two smallest
        int b = heap.poll();
        int combined = a + b;
        totalCost += combined;        // cost of this merge
        heap.add(combined);           // resulting stick re-enters the pool
    }
    return totalCost;
}
```

**Dry run** — `sticks = [2,4,3]`: heap{2,3,4}. Poll 2,3 → cost 5, push 5 → {4,5}. Poll 4,5 → cost 9, total 14. → 14. (Combining 2+3 first, then +4, beats 2+4 first which gives 6+9=15.) ✅

**Complexity:** O(n log n) time (n heap operations of O(log n)), O(n) space.

**Why greedy (the Huffman argument):** Each stick's length is added to the total once per merge it participates in. A stick merged earlier is folded into more subsequent sums. So the *smallest* lengths should be merged earliest (counted most), and the largest latest (counted least) — exactly the Huffman optimality argument. Combining the two smallest at each step is provably optimal.

**Follow-ups:**
1. *"Prove the greedy / why a min-heap?"* → It's Huffman coding: the optimal merge tree gives the longest sticks the shallowest depth. Always merging the two smallest builds that optimal tree bottom-up.
2. *"What if you could merge `k` sticks at once for cost = their sum?"* → Generalized Huffman with k-ary merges; pad with zero-length sticks so `(n-1) % (k-1) == 0`, then poll k at a time.
3. *"Return the merge order, not just the cost?"* → Store node objects with children pointers in the heap to reconstruct the tree.

**Common mistakes:**
- Combining the two *largest* or in input order (suboptimal — larger sticks counted in more sums).
- Forgetting to push the combined stick back into the heap.
- Using a max-heap by mistake.

---

## Pattern recap — what to recognize on sight

| Trigger in problem statement | Pattern | Examples here |
|---|---|---|
| "sorted array, find/exists X" | Binary search Template 1 (`lo<=hi`) | Q1, Q21 |
| "first/last/insert position", "smallest ≥ x" | Boundary search Template 2 (`lo<hi`) | Q2, Q3, Q11 |
| "minimize the maximum" / "smallest X s.t. feasible in ≤ D" | Search on answer (Template 3) | Q13, Q14, Q15, Q16, Q22 |
| "maximize the minimum distance/force" | Search on answer (maximize side) | Q17 |
| "rotated sorted array" | Which-half-is-sorted binary search | Q4, Q5, Q6, Q7 |
| "matrix sorted rows+cols" | Flat binary search OR staircase | Q8, Q9, Q20 |
| "peak / local max", unsorted | Slope-following binary search | Q10 |
| "k-th smallest in huge implicit set" | Binary search on value + count predicate | Q19, Q20 |
| "median of two sorted arrays" | Partition binary search | Q12 |
| "implement / sort with constraints" | Merge / Quick / Heap / Counting / Radix | Q23–Q28, Q32 |
| "k-th largest/smallest, no full sort" | QuickSelect | Q26 |
| "count pairs out of order" | Merge-sort inversion count | Q27 |
| "arrange to form largest/relative order" | Custom comparator | Q30, Q31, Q35 |
| "intervals: remove/cover/min resources" | Sort by end, sweep | Q37, Q38, Q41 |
| "two-pass fix-up (neighbors both sides)" | Two-pass greedy | Q42 |
| "circular feasibility / running deficit" | Prefix-deficit greedy | Q43 |
| "partition by last occurrence" | Greedy farthest-reach | Q44 |
| "pair lightest+heaviest / sorted two-pointer" | Sort + two-pointer greedy | Q46, Q49 |
| "split into equal groups / send n each side" | Sort by relative cost/benefit greedy | Q50, Q52 |

**The three questions to ask yourself on any problem here:**
1. **Is there a monotonic predicate over a bounded answer space?** ("Can I do it within budget X? — if yes for X, yes for all larger X.") → *binary search on the answer* (the highest-leverage pattern: Koko, Ship Packages, Split Array, Allocate Pages, Aggressive Cows).
2. **Is the data sorted or can sorting expose structure?** → binary search (Template 1/2) or a two-pointer/sweep greedy.
3. **Can I justify a greedy with an exchange argument?** If yes, greedy (O(n log n)); if I can't prove it, fall back to DP. Never hand-wave a greedy — name the swap that proves it.

**Binary search debugging checklist (memorize):**
- Overflow-safe mid: `lo + (hi - lo) / 2`.
- Keep `mid` on a side? Then *no* `±1` on that side, and watch the rounding (`lastTrue` rounds up).
- Discard `mid`? Use `±1` and `lo <= hi`.
- Search-on-answer: define `lo`/`hi` from problem bounds (e.g. `max(elem)` to `sum`), and write `feasible` as an O(n) greedy sweep.

---

Created **05-dsa-binary-search-sorting-greedy.md** — 52 questions covered (22 binary search, 14 sorting, 16 greedy), each with brute→optimal, complete Java, dry runs, complexity proofs, follow-ups, and rejection-causing mistakes. The three binary-search templates and the search-on-answer pattern (Koko / Ship Packages / Split Array / Allocate Pages / Aggressive Cows) are covered exhaustively as the highest-leverage material.
