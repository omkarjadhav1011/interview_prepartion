# 02 — DSA: Linked Lists, Stacks & Queues

> Pointers and the right auxiliary structure. After arrays, this is the next layer that shows up in *every* onsite loop. Linked lists test pointer discipline — the ability to rewire `next` references without losing the list. Stacks test whether you can spot the **monotonic stack** (the single most valuable pattern in this file). Queues test BFS and streaming/design instincts. Every question has a **complete** answer — brute force → optimal, full Java, dry run, complexity proof, follow-ups, and the mistakes that get you rejected.

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the *real* signal. For linked lists it is almost always "can you manipulate pointers without leaks or null derefs while talking through it?"
- **Ideal Answer** — approach reasoning, then production-quality Java, then complexity.
- **Follow-ups** — the scripted branches a real loop takes after you solve the base case.
- **Common mistakes** — the specific things that flip a "lean hire" to "no hire."

The two meta-skills this file builds:

1. **The dummy-head + prev/cur/next discipline** for linked lists. Almost every list problem is solved cleanly by a dummy node and careful pointer ordering. Draw the pointers. Always.
2. **Monotonic stack recognition.** The instant you hear "next greater," "next smaller," "span," "previous warmer day," "largest rectangle," or "sum of subarray minimums," you should think *monotonic stack* before you write anything.

---

## Table of Contents

**Linked Lists**
1. Reverse Linked List (iterative + recursive)
2. Reverse Linked List II (between positions)
3. Detect Cycle (Floyd's)
4. Find Cycle Start (Floyd's — with proof)
5. Merge Two Sorted Lists
6. Merge K Sorted Lists (heap + divide & conquer)
7. Remove Nth Node From End (one pass)
8. Add Two Numbers
9. Add Two Numbers II (forward order)
10. Palindrome Linked List
11. Intersection of Two Linked Lists
12. Copy List with Random Pointer
13. Flatten a Multilevel Doubly Linked List
14. Reverse Nodes in K-Group (hard)
15. Sort List (merge sort on a list)
16. Odd Even Linked List
17. Reorder List
18. Swap Nodes in Pairs
19. Rotate List
20. Remove Duplicates from Sorted List
21. Remove Duplicates from Sorted List II
22. Partition List
23. Middle of the Linked List
24. Remove Linked List Elements
25. LRU Cache (linked list + hashmap focus)
26. LFU Cache

**Stacks**
27. Valid Parentheses
28. Min Stack (O(1) min)
29. Max Stack
30. Evaluate Reverse Polish Notation
31. Daily Temperatures (monotonic stack)
32. Next Greater Element I
33. Next Greater Element II (circular)
34. Largest Rectangle in Histogram (FAANG favorite)
35. Maximal Rectangle
36. Basic Calculator (I — parentheses + signs)
37. Basic Calculator II (precedence)
38. Basic Calculator III (precedence + parentheses)
39. Decode String (nested)
40. Remove K Digits (greedy + stack)
41. Asteroid Collision
42. Online Stock Span
43. Simplify Path
44. Implement Queue using Stacks
45. Implement Stack using Queues
46. The Celebrity Problem
47. Remove All Adjacent Duplicates in String
48. Remove All Adjacent Duplicates in String II
49. Sum of Subarray Minimums (monotonic stack)
50. Trapping Rain Water (stack view)

**Queues**
51. Design Circular Queue
52. Design Circular Deque
53. Rotten Oranges (multi-source BFS)
54. Walls and Gates (multi-source BFS)
55. Moving Average from Data Stream
56. Task Scheduler (greedy + cooldown)
57. Number of Recent Calls
58. Design Hit Counter
59. Design Front Middle Back Queue
60. First Unique Number (stream)

---

# LINKED LISTS

The mental model: a linked list problem is almost always solved by **three pointers and a dummy head**. The dummy node removes the special case of "what if the head changes," and naming pointers `prev`, `cur`, `next` keeps the rewiring honest. The recurring patterns are:

- **Reverse** (iterative pointer flip / recursive).
- **Two pointers** — fast & slow for cycles, midpoints, and nth-from-end.
- **Merge** — the comparison-and-splice loop that powers merge sort on lists.
- **Dummy head** — for any problem where the head might be removed or change.

Standard node used throughout:

```java
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

---

### Q1: Reverse Linked List
**Company:** Amazon, Microsoft, Google, Meta, Apple, Flipkart — universal
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Reverse a singly linked list and return the new head. Provide both an iterative and a recursive solution.

**What interviewer is testing:**
The single most fundamental pointer-manipulation skill. If you can't reverse a list cleanly while narrating each pointer move, nothing harder will go well. They're watching whether you save `next` *before* you overwrite it.

**Ideal Answer:**

*Iterative* — walk the list with three pointers. `prev` trails, `cur` is the node being processed, and you must **save `cur.next` before rewiring** or you lose the rest of the list. Flip `cur.next` to point back at `prev`, then slide all three forward.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next; // SAVE before overwriting
        cur.next = prev;          // flip the pointer
        prev = cur;               // slide prev forward
        cur = next;               // slide cur forward
    }
    return prev;                  // prev is the new head (cur is now null)
}
```

**Dry run** — `1 -> 2 -> 3 -> null`:
- prev=null, cur=1: next=2, 1.next=null, prev=1, cur=2 → list piece: `1->null`
- prev=1, cur=2: next=3, 2.next=1, prev=2, cur=3 → `2->1->null`
- prev=2, cur=3: next=null, 3.next=2, prev=3, cur=null → `3->2->1->null`
- return prev=3. ✅

*Recursive* — reverse the rest of the list, then make the next node point back at the current node. The base case is an empty list or a single node.

```java
public ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) return head; // base case
    ListNode newHead = reverseListRecursive(head.next); // reverse the tail
    head.next.next = head;  // the node after head now points back to head
    head.next = null;       // head becomes the new tail
    return newHead;         // new head bubbles up unchanged
}
```

**Why `head.next.next = head` works:** when recursion returns, `head.next` is still the node that *used* to follow head (we haven't touched head's own pointer yet). That node's `next` should now be `head`. Then we sever head's forward pointer to avoid a cycle.

**Complexity:** Both O(n) time. Iterative O(1) space; recursive O(n) space (call stack).

**Follow-up questions the interviewer will ask:**

1. *"Which would you use in production?"* → Iterative — O(1) space and no stack-overflow risk on long lists. The recursive version blows the stack around ~10⁴–10⁵ nodes.
2. *"Reverse only between positions m and n?"* → That's Q2; dummy head + reverse a sublist in place.
3. *"Reverse in groups of k?"* → Q14, Reverse Nodes in K-Group.

**Common mistakes that get you rejected:**
- Overwriting `cur.next` before saving it — you lose the rest of the list and can't recover.
- Returning `head` instead of `prev` (head is now the tail).
- In the recursive version, forgetting `head.next = null`, which leaves a 2-node cycle at the tail.

---

### Q2: Reverse Linked List II
**Company:** Microsoft, Amazon, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Reverse the nodes of the list from position `left` to position `right` (1-indexed) in one pass, and return the list.

**What interviewer is testing:**
Pointer discipline on a *sublist*, plus the dummy-head trick to handle reversal starting at the head.

**Ideal Answer:**

Use a dummy node before head so that reversing from position 1 is not a special case. Walk `prev` to the node just before `left`. Then use the **head-insertion** technique: repeatedly take the node after `cur` and splice it to the front of the reversed section.

```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy;
    for (int i = 0; i < left - 1; i++) prev = prev.next; // node before 'left'

    ListNode cur = prev.next;            // first node of the segment (stays, drifts right)
    for (int i = 0; i < right - left; i++) {
        ListNode moved = cur.next;       // node to pull to the front
        cur.next = moved.next;           // unlink it
        moved.next = prev.next;          // point it at current segment head
        prev.next = moved;               // make it the new segment head
    }
    return dummy.next;
}
```

**Dry run** — `1->2->3->4->5`, left=2, right=4:
- prev=node(1), cur=node(2).
- iter1: moved=3, cur(2).next=4, 3.next=2, prev(1).next=3 → `1->3->2->4->5`
- iter2: moved=4, cur(2).next=5, 4.next=3, prev(1).next=4 → `1->4->3->2->5` ✅

**Complexity:** O(n) time, O(1) space, single pass.

**Follow-ups:**
1. *"Why a dummy node?"* → If `left == 1`, the head changes; the dummy lets `prev` exist uniformly.
2. *"Reverse the whole list?"* → `left=1, right=n` reduces to Q1.

**Common mistakes:**
- Not using a dummy and crashing when `left == 1`.
- Losing the `cur` reference — `cur` must stay as the *original* first segment node; it naturally drifts to the segment's end.

---

### Q3: Detect Cycle (Linked List Cycle I)
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Determine whether a linked list has a cycle.

**What interviewer is testing:**
Floyd's tortoise & hare. The naive answer is a hashset of visited nodes (O(n) space); they want the O(1)-space two-pointer.

**Ideal Answer:**

*Brute force* — store visited nodes in a `HashSet`; if you revisit one, there's a cycle. O(n) time, O(n) space.

*Optimal — Floyd's:* move `slow` one step and `fast` two steps. If there's a cycle, `fast` laps `slow` and they meet inside the loop. If `fast` hits null, no cycle.

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;        // +1
        fast = fast.next.next;   // +2
        if (slow == fast) return true; // pointers collide inside the cycle
    }
    return false;                // fast reached the end -> no cycle
}
```

**Why they must meet:** Once both are inside the cycle, each step the gap between them shrinks by exactly 1 (fast gains 1 per step). A gap that decreases by 1 every step in a finite cycle must hit 0 — they collide. They can never "jump over" each other because the gap changes by 1, not 2.

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Find where the cycle starts?"* → Q4.
2. *"Length of the cycle?"* → After they meet, keep one pointer fixed and walk the other until it returns; count steps.
3. *"Why step fast by 2 and not 3?"* → Any speed ≥2 works, but +2 guarantees they don't skip past each other and is simplest to reason about.

**Common mistakes:**
- Checking `fast != null` but not `fast.next != null` → NPE on `fast.next.next`.
- Initializing `slow=head, fast=head.next` then comparing before stepping (works but is fiddly; start both at head and step first).

---

### Q4: Find Cycle Start (Linked List Cycle II)
**Company:** Amazon, Google, Microsoft, Meta
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Return the node where the cycle begins, or null if there is no cycle. O(1) space.

**What interviewer is testing:**
Whether you can *prove* the second phase of Floyd's, not just recite it. This is the question where "I memorized it" gets exposed.

**Ideal Answer:**

Phase 1: run tortoise & hare until they meet (or fast hits null → no cycle).
Phase 2: reset one pointer to the head; advance both **one step at a time**; the node where they meet again is the cycle entrance.

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {              // meeting point found
            ListNode p = head;
            while (p != slow) {          // walk both at speed 1
                p = p.next;
                slow = slow.next;
            }
            return p;                    // entrance of the cycle
        }
    }
    return null;                         // no cycle
}
```

**The proof (be ready to write this on the board):**
Let `L` = distance from head to the cycle entrance, `C` = cycle length, and `x` = distance from the entrance to the meeting point (measured along the cycle).

- When they meet, slow has traveled `L + x` and fast has traveled `L + x + nC` for some integer `n ≥ 1` (fast did `n` extra full loops).
- Fast moved twice as far as slow: `2(L + x) = L + x + nC`.
- Simplify: `L + x = nC`, so `L = nC − x`.

Now reset `p` to head. After `L` steps, `p` is at the entrance. Meanwhile slow, starting at the meeting point and moving `L = nC − x` steps, advances `nC − x` along the cycle — which from position `x` lands it at position `x + (nC − x) = nC ≡ 0 (mod C)`, i.e. exactly the entrance. So they meet at the entrance. ∎

**Dry run** — list `3 -> 2 -> 0 -> -4`, with `-4` pointing back to `2`:
- Phase 1: slow/fast meet at node `-4` (or `0` depending on layout); say they meet somewhere in the cycle.
- Phase 2: `p` from head and `slow` from meeting point step together; they collide at node `2` = entrance. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why does phase 2 use equal speed?"* → The algebra `L = nC − x` only holds for both moving at speed 1 from their respective starts.
2. *"Could you do it with a hashset?"* → Yes, O(n) space — the first re-seen node is the entrance. Mention it, then give the O(1) version.

**Common mistakes:**
- Advancing fast at speed 2 in phase 2 (breaks the proof).
- Not handling the no-cycle case (loop exits, return null).

---

### Q5: Merge Two Sorted Lists
**Company:** Amazon, Microsoft, Apple, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Merge two sorted linked lists into one sorted list by splicing the existing nodes.

**What interviewer is testing:**
The merge primitive that underlies merge sort and Merge K Lists. Dummy-head usage and clean tail-splicing.

**Ideal Answer:**

Use a dummy head and a `tail` pointer. Repeatedly attach the smaller of the two front nodes to `tail`, advance that list, and move `tail`. When one list is exhausted, splice the remainder of the other in one move.

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
        else                  { tail.next = l2; l2 = l2.next; }
        tail = tail.next;
    }
    tail.next = (l1 != null) ? l1 : l2; // attach the leftover tail in O(1)
    return dummy.next;
}
```

**Dry run** — `1->2->4` and `1->3->4`:
- compare 1,1 → take l1(1); compare 2,1 → take l2(1); compare 2,3 → take 2; compare 4,3 → take 3; compare 4,4 → take 4; l1 exhausted → attach `4`.
- Result `1->1->2->3->4->4`. ✅

**Complexity:** O(n + m) time, O(1) space (splicing, not copying).

**Recursive variant** (elegant, O(n+m) stack):

```java
public ListNode mergeTwoListsRec(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val <= l2.val) { l1.next = mergeTwoListsRec(l1.next, l2); return l1; }
    else                  { l2.next = mergeTwoListsRec(l1, l2.next); return l2; }
}
```

**Follow-ups:**
1. *"Merge K lists?"* → Q6.
2. *"`<` vs `<=` — does stability matter?"* → Use `<=` to keep the relative order of equal elements (stable merge).

**Common mistakes:**
- Creating new nodes instead of splicing existing ones (wastes space; the prompt usually wants splicing).
- Forgetting to attach the remaining tail.

---

### Q6: Merge K Sorted Lists
**Company:** Amazon, Google, Meta, Microsoft, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Merge `k` sorted linked lists into one sorted list.

**What interviewer is testing:**
Two optimal strategies — a **min-heap** over the k current heads, and **divide & conquer** (pairwise merge). They want you to reject the naive "merge one at a time" O(kN) approach.

**Ideal Answer:**

*Naive:* merge list 1 with 2, result with 3, etc. Merging the accumulator repeatedly costs O(k²·avg) ≈ O(kN) where N is the total number of nodes — reject it.

*Approach A — min-heap:* push the head of each list into a priority queue keyed by value. Pop the smallest, append it, and push its `next`. The heap never holds more than k nodes.

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq =
        new PriorityQueue<>((a, b) -> Integer.compare(a.val, b.val));
    for (ListNode node : lists) if (node != null) pq.offer(node);

    ListNode dummy = new ListNode(0), tail = dummy;
    while (!pq.isEmpty()) {
        ListNode smallest = pq.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) pq.offer(smallest.next); // re-fill from same list
    }
    return dummy.next;
}
```

**Complexity (heap):** O(N log k) time — each of N nodes is pushed/popped once, each heap op is O(log k). Space O(k) for the heap.

*Approach B — divide & conquer:* pair up lists and merge them, halving the count each round, like merge sort's merge step.

```java
public ListNode mergeKListsDC(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    int interval = 1, n = lists.length;
    while (interval < n) {
        for (int i = 0; i + interval < n; i += interval * 2) {
            lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
        }
        interval *= 2;
    }
    return lists[0];
}
// mergeTwoLists from Q5
```

**Why D&C is also O(N log k):** there are log k merge rounds; each round touches all N nodes once. No heap overhead, often the fastest in practice.

**Dry run (heap)** — `[[1,4,5],[1,3,4],[2,6]]`:
- heap heads {1,1,2} → pop 1 (from list0), push 4; heap {1,2,4} → pop 1 (list1), push 3; {2,3,4} → pop 2, push 6; ... produces `1->1->2->3->4->4->5->6`. ✅

**Follow-ups:**
1. *"Heap vs divide & conquer trade-off?"* → Same asymptotic time. Heap is simpler to extend to streaming; D&C avoids the heap constant factor and extra object overhead.
2. *"What if k is huge but each list tiny?"* → log k dominates; both still fine. Heap memory is O(k).
3. *"Stability / equal values?"* → Both preserve a consistent order; ensure comparator is by value only.

**Common mistakes:**
- The naive sequential merge (O(kN)) — an instant signal you didn't see the optimization.
- Pushing entire lists into the heap instead of one head per list.
- Forgetting the `if (node != null)` guard for empty input lists → NPE in the comparator.

---

### Q7: Remove Nth Node From End of List
**Company:** Amazon, Microsoft, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Remove the nth node from the end of the list in **one pass** and return the head.

**What interviewer is testing:**
The fixed-gap two-pointer technique, and the dummy node to handle removing the head.

**Ideal Answer:**

*Two-pass:* compute length, then remove the `(length − n + 1)`th node from the front. Simple but two passes.

*One-pass (optimal):* advance a `fast` pointer `n` nodes ahead, then move `fast` and `slow` together until `fast` hits the end. Now `slow` sits just before the target. A dummy head handles the case where the head itself is removed.

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode fast = dummy, slow = dummy;
    for (int i = 0; i < n; i++) fast = fast.next; // create a gap of n
    while (fast.next != null) {                   // move both until fast is last
        fast = fast.next;
        slow = slow.next;
    }
    slow.next = slow.next.next;                   // unlink the nth-from-end
    return dummy.next;
}
```

**Dry run** — `1->2->3->4->5`, n=2:
- fast advances 2 → fast at node 2. Then move together: fast→3 slow→1; fast→4 slow→2; fast→5 slow→3. fast.next null → stop. slow(3).next = node5 → `1->2->3->5`. ✅

**Why the dummy matters:** removing the head (`n == length`) means `slow` must point at something *before* head; the dummy provides it.

**Complexity:** O(n) time (single pass), O(1) space.

**Follow-ups:**
1. *"What if n > length?"* → Validate; either clamp or throw. Mention you'd guard `fast` becoming null in the gap loop.
2. *"Two-pass acceptable?"* → Yes if asked, but interviewers prize the one-pass.

**Common mistakes:**
- No dummy → crash when removing the head.
- Off-by-one in the gap (advance exactly `n`, then stop when `fast.next == null`, leaving slow before target).

---

### Q8: Add Two Numbers
**Company:** Amazon, Microsoft, Google, Bloomberg, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Two numbers are stored as linked lists with digits in **reverse** order (least-significant first). Add them and return the sum as a list.

**What interviewer is testing:**
Carry handling, unequal lengths, and a final carry-out — the elementary-school addition algorithm, but with pointer hygiene.

**Ideal Answer:**

Walk both lists simultaneously, summing digit + digit + carry. Because digits are reverse-ordered, you process least-significant first, which is exactly addition order. Continue while either list has nodes *or* a carry remains.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), tail = dummy;
    int carry = 0;
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        tail.next = new ListNode(sum % 10);
        tail = tail.next;
    }
    return dummy.next;
}
```

**Dry run** — `2->4->3` (342) + `5->6->4` (465) = 807:
- 2+5=7,carry0 → 7; 4+6=10,carry1 → 0; 3+4+1=8,carry0 → 8 → `7->0->8`. ✅
- Edge `9->9` + `1` = `0->0->1` (99+1=100): 9+1=10→0 c1; 9+0+1=10→0 c1; carry1→1. ✅

**Complexity:** O(max(n, m)) time, O(max(n, m)) space for the result.

**Follow-ups:**
1. *"Digits in forward order (most-significant first)?"* → Q9 — reverse, or use two stacks.
2. *"Why loop on `carry != 0` too?"* → A final carry creates a new most-significant digit (the `99+1` case).

**Common mistakes:**
- Stopping when both lists end but ignoring a leftover carry.
- Forgetting unequal-length handling.

---

### Q9: Add Two Numbers II
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Same as Q8 but digits are in **forward** order (most-significant first), and you should not modify the input (ideally avoid reversing).

**What interviewer is testing:**
Recognizing that "process from least-significant but read from most-significant" screams **stack**.

**Ideal Answer:**

Push both lists onto two stacks. Pop together (least-significant first), build the result by **prepending** each new digit to the front (head insertion), carrying as usual.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Deque<Integer> s1 = new ArrayDeque<>(), s2 = new ArrayDeque<>();
    for (ListNode p = l1; p != null; p = p.next) s1.push(p.val);
    for (ListNode p = l2; p != null; p = p.next) s2.push(p.val);

    ListNode head = null;       // we build by prepending
    int carry = 0;
    while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {
        int sum = carry;
        if (!s1.isEmpty()) sum += s1.pop();
        if (!s2.isEmpty()) sum += s2.pop();
        carry = sum / 10;
        head = new ListNode(sum % 10, head); // prepend
    }
    return head;
}
```

**Dry run** — `7->2->4->3` (7243) + `5->6->4` (564) = 7807:
- stacks pop 3,4 → 7 head=[7]; 4,6 → 10 → 0 carry1 head=[0,7]; 2,5+1 → 8 head=[8,0,7]; 7 → 7 head=[7,8,0,7] = `7->8->0->7`. ✅

**Complexity:** O(n + m) time, O(n + m) space (stacks + output).

**Follow-ups:**
1. *"No extra space allowed?"* → Reverse both lists, run Q8, reverse the result. O(1) extra but mutates input.
2. *"Why prepend instead of append?"* → We compute digits least-significant first but the answer must read most-significant first.

**Common mistakes:**
- Appending instead of prepending → reversed result.
- Reversing input when the prompt forbids mutation.

---

### Q10: Palindrome Linked List
**Company:** Amazon, Microsoft, Meta, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Determine whether a singly linked list is a palindrome, ideally in O(n) time and O(1) space.

**What interviewer is testing:**
Combining two primitives — find the middle (fast/slow) and reverse the second half — then compare. Plus the etiquette of restoring the list.

**Ideal Answer:**

*Easy approach:* copy values into an array/list and two-pointer compare. O(n) space.

*Optimal O(1) space:* find the middle with fast/slow, reverse the second half, compare the two halves node by node, then (optionally) restore.

```java
public boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;

    // 1. find middle (slow ends at start of 2nd half for even length)
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // 2. reverse second half
    ListNode second = reverse(slow);
    ListNode firstHalf = head, secondHalf = second;

    // 3. compare
    boolean palindrome = true;
    while (secondHalf != null) {
        if (firstHalf.val != secondHalf.val) { palindrome = false; break; }
        firstHalf = firstHalf.next;
        secondHalf = secondHalf.next;
    }
    // 4. restore the list (good citizenship)
    reverse(second);
    return palindrome;
}

private ListNode reverse(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```

**Dry run** — `1->2->2->1`:
- middle: slow ends at the second `2` (index 2). reverse from there → `1<-2` i.e. second half `1->2`.
- compare firstHalf `1->2`, secondHalf `1->2` → match. Palindrome. ✅
- Odd `1->2->3->2->1`: slow ends at `3`; comparing the shorter second half ignores the center — correct.

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why compare while `secondHalf != null` rather than the first half?"* → The second (reversed) half is the shorter or equal one; the middle node in odd-length lists is naturally skipped.
2. *"Should you restore the list?"* → In production, yes — leaving the input mutated is a side-effect bug. Mention it even if not asked.

**Common mistakes:**
- Not restoring the list (acceptable for the puzzle, but flag it).
- Off-by-one on where `slow` lands for even vs odd lengths.

---

### Q11: Intersection of Two Linked Lists
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Two singly linked lists may merge at some node and share a common tail. Return the node where they intersect, or null. The intersection is by **reference**, not value.

**What interviewer is testing:**
The two-pointer length-equalization trick. They want O(1) space, not a hashset of nodes.

**Ideal Answer:**

*Hashset:* store all nodes of list A, walk B, return first node seen. O(n) space.

*Optimal — pointer switching:* point `a` at headA, `b` at headB. Each step advance both; when one reaches null, redirect it to the *other* list's head. After at most `lenA + lenB` steps, both have traveled the same total distance, so they align at the intersection (or both reach null together).

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    ListNode a = headA, b = headB;
    while (a != b) {                       // also true when both become null
        a = (a == null) ? headB : a.next;  // switch to the other list at the end
        b = (b == null) ? headA : b.next;
    }
    return a;                              // intersection node, or null
}
```

**Why it works:** Let A have `p` nodes before the intersection and `c` shared; B has `q` before and the same `c`. Pointer `a` travels `p + c + q`; pointer `b` travels `q + c + p`. Equal total distances mean they arrive at the intersection node simultaneously. If no intersection, both hit null after `p + q + ...` steps and the loop ends with `a == b == null`.

**Dry run** — A: `4->1->8->4->5`, B: `5->6->1->8->4->5`, intersect at `8`:
- The pointers swap lists once; after equalizing lengths they meet at node `8`. ✅

**Complexity:** O(n + m) time, O(1) space.

**Follow-ups:**
1. *"What if they don't intersect?"* → Both become null at the same time → loop ends, returns null.
2. *"Length-difference approach?"* → Compute both lengths, advance the longer by the difference, then walk together. Equally valid, slightly more code.

**Common mistakes:**
- Comparing values instead of references.
- Redirecting to the *same* list's head (must swap to the other list to equalize lengths).

---

### Q12: Copy List with Random Pointer
**Company:** Amazon, Microsoft, Meta, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Each node has `next` and a `random` pointer to any node or null. Deep-copy the list.

**What interviewer is testing:**
Mapping old→new nodes. The two solutions are a **hashmap** (clean) and **node interleaving** (O(1) extra space — the impressive one).

**Ideal Answer:**

Node:
```java
class Node {
    int val;
    Node next, random;
    Node(int val) { this.val = val; }
}
```

*Approach A — hashmap (old node → new node):* first pass creates clones and maps them; second pass wires `next` and `random` using the map.

```java
public Node copyRandomList(Node head) {
    Map<Node, Node> map = new HashMap<>();
    for (Node cur = head; cur != null; cur = cur.next)
        map.put(cur, new Node(cur.val));          // clone each node
    for (Node cur = head; cur != null; cur = cur.next) {
        map.get(cur).next   = map.get(cur.next);  // null maps to null
        map.get(cur).random = map.get(cur.random);
    }
    return map.get(head);
}
```
O(n) time, O(n) space.

*Approach B — interleaving (O(1) extra space):*
1. Insert each clone right after its original: `A -> A' -> B -> B' -> ...`.
2. Set clones' randoms: `cur.next.random = cur.random.next` (the clone of cur's random sits right after it).
3. Detach the clones to rebuild two separate lists.

```java
public Node copyRandomListInterleave(Node head) {
    if (head == null) return null;
    // 1. interleave clones
    for (Node cur = head; cur != null; cur = cur.next.next) {
        Node copy = new Node(cur.val);
        copy.next = cur.next;
        cur.next = copy;
    }
    // 2. assign randoms
    for (Node cur = head; cur != null; cur = cur.next.next) {
        if (cur.random != null) cur.next.random = cur.random.next;
    }
    // 3. separate the two lists
    Node dummy = new Node(0), copyTail = dummy;
    for (Node cur = head; cur != null; cur = cur.next) {
        copyTail.next = cur.next;     // clone
        copyTail = copyTail.next;
        cur.next = cur.next.next;     // restore original
    }
    return dummy.next;
}
```

**Dry run (interleave)** — original `A->B`, A.random=B:
- after step1: `A->A'->B->B'`. step2: A.random=B so A'.random = B.next = B'. step3: separate → original `A->B`, copy `A'->B'` with A'.random=B'. ✅

**Complexity:** A: O(n)/O(n). B: O(n) time, O(1) extra (besides output).

**Follow-ups:**
1. *"Why does `cur.next.random = cur.random.next` work?"* → After interleaving, every clone sits immediately after its original, so the clone of `cur.random` is exactly `cur.random.next`.
2. *"Recursive with memoization?"* → A DFS with a `visited` map; same as approach A conceptually.

**Common mistakes:**
- In approach B, not restoring the original list (step 3 must rebuild both).
- Forgetting null `random` checks → NPE.

---

### Q13: Flatten a Multilevel Doubly Linked List
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A doubly linked list where nodes also have a `child` pointer to a separate doubly linked list (which may itself have children). Flatten it into a single-level doubly linked list (depth-first), clearing all `child` pointers.

**What interviewer is testing:**
DFS on a list using an explicit stack (or recursion), and careful `prev`/`next`/`child` rewiring on a doubly linked structure.

**Ideal Answer:**

Node:
```java
class Node {
    int val;
    Node prev, next, child;
}
```

When you encounter a node with a child, the child sublist must be spliced in *before* the current node's `next`. An explicit stack handles arbitrary nesting: when descending into a child, push the current `next` so you can resume after the child sublist ends.

```java
public Node flatten(Node head) {
    if (head == null) return null;
    Deque<Node> stack = new ArrayDeque<>();
    Node cur = head, prev = null;

    // pseudo-traversal pushing 'next' when we dive into a child
    stack.push(head);
    while (!stack.isEmpty()) {
        cur = stack.pop();
        if (prev != null) {            // splice cur after prev
            prev.next = cur;
            cur.prev = prev;
        }
        if (cur.next != null) stack.push(cur.next);   // resume here later
        if (cur.child != null) {                      // dive in first
            stack.push(cur.child);
            cur.child = null;                         // clear child pointer
        }
        prev = cur;
    }
    return head;
}
```

Note we push `cur.next` *before* `cur.child`, so the child is popped (processed) first — depth-first, child before sibling.

**Dry run** — `1 - 2 - 3` with `2.child = (7 - 8)`:
- pop1, push next(2). pop2, splice after1; push next(3); push child(7), clear 2.child. pop7, splice after2; push next(8). pop8, splice after7. pop3, splice after8.
- Result `1 - 2 - 7 - 8 - 3`. ✅

**Complexity:** O(n) time, O(d) space where d is max nesting depth (stack).

**Follow-ups:**
1. *"Recursive version?"* → DFS returning the tail of each flattened sublist so the caller can stitch the next sibling.
2. *"Why push next before child?"* → Stack is LIFO; pushing child last means it pops first → depth-first.

**Common mistakes:**
- Forgetting to set `prev` pointers (it's a *doubly* linked list).
- Not nulling `child` after splicing → invalid output.

---

### Q14: Reverse Nodes in K-Group
**Company:** Amazon, Google, Microsoft, Meta, Uber — a top-tier FAANG favorite
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Reverse the nodes of a linked list `k` at a time and return the modified list. Nodes left over (fewer than `k`) at the end stay as they are. Use O(1) extra space if possible.

**What interviewer is testing:**
This is *the* linked-list pointer-discipline question. They want to see: (1) you check there are k nodes before reversing, (2) you reverse a bounded segment correctly, and (3) you stitch the reversed segment back to the previous group's tail and the next group's head — without losing any pointers. Getting all the connections right under pressure separates strong candidates.

**Ideal Answer:**

Structure it with a dummy head and a `groupPrev` pointer that always points to the node *before* the current group. For each group:
1. Walk `k` nodes ahead from `groupPrev` to find the `kth` node. If fewer than `k` remain, stop — leave the rest as is.
2. Reverse the `k` nodes between `groupPrev` and the node after `kth`.
3. Re-stitch: the old group head becomes the group tail, and `groupPrev` advances to it.

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0, head);
    ListNode groupPrev = dummy;            // node before the current group

    while (true) {
        // 1. find the kth node from groupPrev
        ListNode kth = groupPrev;
        for (int i = 0; i < k && kth != null; i++) kth = kth.next;
        if (kth == null) break;            // fewer than k nodes left -> done

        ListNode groupNext = kth.next;     // first node of the NEXT group
        // 2. reverse the group: nodes (groupPrev.next ... kth)
        ListNode prev = groupNext, cur = groupPrev.next;
        while (cur != groupNext) {
            ListNode next = cur.next;
            cur.next = prev;               // flip, pointing toward groupNext at the end
            prev = cur;
            cur = next;
        }
        // 3. re-stitch
        ListNode oldGroupHead = groupPrev.next; // becomes the new tail of the group
        groupPrev.next = kth;                   // kth is the new head of the group
        groupPrev = oldGroupHead;               // advance to the new tail for next round
    }
    return dummy.next;
}
```

**Detailed dry run** — `1->2->3->4->5`, k=2:

Group 1:
- groupPrev=dummy. kth walks 2 → kth=node2. groupNext=node3.
- reverse cur from node1, prev=node3: cur=1→ next=2, 1.next=3, prev=1, cur=2; cur=2→ next=3, 2.next=1, prev=2, cur=3 == groupNext stop. Now `2->1->3...`.
- oldGroupHead = node1. groupPrev.next(dummy.next)=node2. groupPrev=node1.
- List: `2->1->3->4->5`.

Group 2:
- groupPrev=node1. kth walks 2 → node4 (node1→node3→node4). groupNext=node5.
- reverse: cur=3, prev=5: 3.next=5,prev=3,cur=4; 4.next=3,prev=4,cur=5 stop. `4->3->5`.
- oldGroupHead=node3. node1.next=node4. groupPrev=node3.
- List: `2->1->4->3->5`.

Group 3:
- groupPrev=node3. kth walks 2 but only node5 then null → kth=null → break. node5 stays.
- Final `2->1->4->3->5`. ✅

**Recursive version** (cleaner to reason about, O(n/k) stack):

```java
public ListNode reverseKGroupRec(ListNode head, int k) {
    ListNode node = head;
    for (int i = 0; i < k; i++) {          // ensure k nodes exist
        if (node == null) return head;     // fewer than k -> leave as is
        node = node.next;
    }
    // 'node' is now the head of the next group; reverse first k nodes
    ListNode prev = reverseKGroupRec(node, k); // recurse on the rest first
    ListNode cur = head;
    for (int i = 0; i < k; i++) {
        ListNode next = cur.next;
        cur.next = prev;                   // attach reversed-rest to the tail
        prev = cur;
        cur = next;
    }
    return prev;                           // new head of this group
}
```

**Why the iterative `prev = groupNext` initialization matters:** by seeding `prev` with `groupNext` (the next group's head) before the in-group reversal, the *last* node we flip automatically points into the next group. That removes a separate stitching step for the group tail.

**Complexity:** O(n) time — every node is visited a constant number of times. Iterative O(1) space; recursive O(n/k) stack.

**Follow-ups:**
1. *"What if the remaining nodes are fewer than k — reverse them anyway?"* → A variant; check the count and reverse unconditionally (drop the `kth == null` early break, handle the partial group).
2. *"Reverse in alternating groups (reverse k, skip k)?"* → Add a toggle; in skip rounds just advance groupPrev by k.
3. *"Why is this O(1) space iteratively but not recursively?"* → The recursion stack holds one frame per group.

**Common mistakes:**
- Reversing a group before confirming k nodes exist → you reverse a partial tail that should stay put.
- Losing the connection between groups — forgetting to update `groupPrev` to the *old* group head (which is the new tail).
- Off-by-one in the k-count loop (walk exactly k nodes, then the (k+1)th check tells you whether a full group exists).

---

### Q15: Sort List
**Company:** Amazon, Google, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Sort a linked list in O(n log n) time. Ideally O(1) extra space (besides recursion).

**What interviewer is testing:**
Why **merge sort** is the natural choice for lists (no random access, so quicksort's pivoting is awkward; merge sort splits and merges cheaply with pointers). Plus finding the middle and merging — primitives from Q5 and Q10.

**Ideal Answer:**

Merge sort: split the list into halves using fast/slow, recursively sort each half, then merge (Q5). Merge sort on a linked list needs no extra array — splitting and merging are pointer operations.

```java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head; // base case

    // 1. split into two halves; 'mid' is start of second half
    ListNode mid = getMiddleAndSplit(head);
    ListNode left = sortList(head);
    ListNode right = sortList(mid);

    // 2. merge two sorted halves
    return merge(left, right);
}

// returns head of the SECOND half, and severs the first half's tail
private ListNode getMiddleAndSplit(ListNode head) {
    ListNode slow = head, fast = head, prev = null;
    while (fast != null && fast.next != null) {
        prev = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    prev.next = null;        // cut the list into two halves
    return slow;             // start of second half
}

private ListNode merge(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0), tail = dummy;
    while (a != null && b != null) {
        if (a.val <= b.val) { tail.next = a; a = a.next; }
        else                { tail.next = b; b = b.next; }
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b;
    return dummy.next;
}
```

**Dry run** — `4->2->1->3`:
- split → `4->2` and `1->3`. Sort `4->2` → split `4`,`2` → merge → `2->4`. Sort `1->3` → `1->3`. merge `2->4` and `1->3` → `1->2->3->4`. ✅

**Why `prev.next = null` and the `prev` pointer:** for an even-length list `4->2->1->3`, slow lands on `1` (start of second half) and `prev` on `2`; cutting `2.next = null` yields `4->2` and `1->3`. Without the cut, the recursion never terminates.

**Complexity:** O(n log n) time. Space O(log n) for recursion (the merge itself is O(1) via splicing). A bottom-up iterative merge sort achieves true O(1) space — mention it.

**Follow-ups:**
1. *"Why not quicksort?"* → No O(1) random access; partitioning a list is clumsy and worst-case O(n²). Merge sort is the canonical list sort.
2. *"True O(1) space?"* → Bottom-up merge sort: merge sublists of size 1, 2, 4, ... iteratively, no recursion stack.
3. *"Stable?"* → Yes, with `<=` in merge.

**Common mistakes:**
- Forgetting to sever the first half (`prev.next = null`) → infinite recursion.
- Using fast/slow without `prev`, then unable to cut cleanly.

---

### Q16: Odd Even Linked List
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Group all nodes at odd indices together followed by nodes at even indices (by position, 1-indexed, not value). O(1) space.

**What interviewer is testing:**
Maintaining two interleaved chains and splicing them, with correct termination.

**Ideal Answer:**

Maintain an `odd` chain and an `even` chain. The `evenHead` is saved so we can attach it after the odd chain at the end. Each iteration extends both chains by one and re-links.

```java
public ListNode oddEvenList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode odd = head, even = head.next, evenHead = even;
    while (even != null && even.next != null) {
        odd.next = even.next;   // odd grabs the node after even
        odd = odd.next;
        even.next = odd.next;   // even grabs the node after the new odd
        even = even.next;
    }
    odd.next = evenHead;        // attach even chain after odd chain
    return head;
}
```

**Dry run** — `1->2->3->4->5`:
- odd=1,even=2,evenHead=2. iter1: odd.next=3,odd=3; even.next=4,even=4 → `1->3->...`, `2->4`. iter2: odd.next=5,odd=5; even.next=null,even=null. attach: 5.next=2 → `1->3->5->2->4`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why save evenHead?"* → The even chain's head is the second node; you need it to splice after the odd chain.
2. *"Group by value parity instead of index?"* → Different problem; partition by value (similar to Q22).

**Common mistakes:**
- Wrong loop condition causing NPE (must check `even != null && even.next != null`).
- Forgetting the final `odd.next = evenHead`.

---

### Q17: Reorder List
**Company:** Amazon, Microsoft, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `L0 -> L1 -> ... -> Ln`, reorder to `L0 -> Ln -> L1 -> Ln-1 -> L2 -> ...` in place.

**What interviewer is testing:**
Composing three primitives: find middle, reverse second half, merge alternately. Same toolkit as palindrome (Q10).

**Ideal Answer:**

1. Find the middle (fast/slow).
2. Reverse the second half.
3. Merge the two halves alternately.

```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // 1. middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // 2. reverse second half (slow.next onward)
    ListNode second = reverse(slow.next);
    slow.next = null;            // cut first half

    // 3. merge alternately
    ListNode first = head;
    while (second != null) {
        ListNode f = first.next, s = second.next;
        first.next = second;
        second.next = f;
        first = f;
        second = s;
    }
}

private ListNode reverse(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```

**Dry run** — `1->2->3->4`:
- middle: slow stops at 2; second half `3->4` reversed → `4->3`; cut → first `1->2`.
- merge: first=1,second=4: 1.next=4,4.next=2,first=2,second=3; first=2,second=3: 2.next=3,3.next=null,first=null,second=null. → `1->4->2->3`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why `fast.next && fast.next.next` here vs `fast && fast.next` elsewhere?"* → Controls where slow lands for even/odd lengths so the first half is ≥ the second; pick the condition to match your cut.
2. *"Without modifying the list?"* → Use a deque of nodes and alternate front/back; O(n) space.

**Common mistakes:**
- Not cutting the first half (`slow.next = null`) → cycle.
- Merge loop overrunning — guard on `second != null` (the reversed second half is the shorter/equal one).

---

### Q18: Swap Nodes in Pairs
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Swap every two adjacent nodes and return the head. You must swap the **nodes**, not just values.

**What interviewer is testing:**
A special case of Reverse-in-K-Group (k=2), tested for clean pointer rewiring with a dummy.

**Ideal Answer:**

Dummy head; `prev` points before each pair. For nodes `a, b` after prev: rewire `prev -> b -> a -> (rest)`.

```java
public ListNode swapPairs(ListNode head) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy;
    while (prev.next != null && prev.next.next != null) {
        ListNode a = prev.next, b = a.next;
        a.next = b.next;   // a points past b
        b.next = a;        // b points to a
        prev.next = b;     // prev points to b (new first of pair)
        prev = a;          // advance to the node now at the pair's tail
    }
    return dummy.next;
}
```

**Dry run** — `1->2->3->4`:
- prev=dummy: a=1,b=2: 1.next=3, 2.next=1, dummy.next=2, prev=1 → `2->1->3->4`.
- prev=1: a=3,b=4: 3.next=null,4.next=3,1.next=4,prev=3 → `2->1->4->3`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Generalize to k?"* → Q14, Reverse Nodes in K-Group.
2. *"Recursive solution?"* → `head.next = swapPairs(head.next.next); ... return second;` — O(n) stack.

**Common mistakes:**
- Swapping values instead of nodes (problem usually forbids it).
- Advancing `prev` incorrectly (it should land on `a`, the new tail of the swapped pair).

---

### Q19: Rotate List
**Company:** Amazon, Microsoft, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Rotate the list to the right by `k` places.

**What interviewer is testing:**
Closing the list into a ring, computing the new break point with `k % len`, and reopening it.

**Ideal Answer:**

Find the length and the tail. Connect tail to head (make a ring). The new tail is at position `len - k%len - 1` from the head; the new head is the node after it. Break the ring there.

```java
public ListNode rotateRight(ListNode head, int k) {
    if (head == null || head.next == null || k == 0) return head;

    // 1. length + tail
    int len = 1;
    ListNode tail = head;
    while (tail.next != null) { tail = tail.next; len++; }

    // 2. normalize k
    k %= len;
    if (k == 0) return head;        // full rotation = no change

    // 3. close the ring, find new tail
    tail.next = head;               // ring
    int stepsToNewTail = len - k;   // new tail is this many steps from head
    ListNode newTail = head;
    for (int i = 1; i < stepsToNewTail; i++) newTail = newTail.next;

    ListNode newHead = newTail.next;
    newTail.next = null;            // reopen
    return newHead;
}
```

**Dry run** — `1->2->3->4->5`, k=2:
- len=5, tail=5. k=2. ring 5->1. stepsToNewTail=3 → newTail walks to node3. newHead=4. break 3.next=null → `4->5->1->2->3`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why `k %= len`?"* → Rotating by len is identity; large k otherwise wastes work or overflows the walk.
2. *"Rotate left by k?"* → New head is `k` steps in (mirror the arithmetic).

**Common mistakes:**
- Not taking `k % len` → over-rotation / out-of-bounds walk.
- Off-by-one finding the new tail (`len - k` steps from head, stop one before).

---

### Q20: Remove Duplicates from Sorted List
**Company:** Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Given a sorted list, delete duplicates so each value appears once.

**Ideal Answer:**

Single pass; when `cur.val == cur.next.val`, skip the next node. Only advance `cur` when no skip happens.

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode cur = head;
    while (cur != null && cur.next != null) {
        if (cur.val == cur.next.val) cur.next = cur.next.next; // skip dup
        else cur = cur.next;                                   // advance
    }
    return head;
}
```

**Dry run** — `1->1->2->3->3`: at 1 sees 1 → skip → `1->2->3->3`; advance to 2; 2≠3 advance; at 3 sees 3 → skip → `...3`; done → `1->2->3`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-up:** Q21 — remove *all* nodes that have duplicates (keep only distinct).

**Common mistakes:** Advancing `cur` even after a skip (could miss a triple duplicate).

---

### Q21: Remove Duplicates from Sorted List II
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a sorted list, delete **all** nodes that have duplicate values, leaving only distinct values.

**What interviewer is testing:**
Dummy head (the head itself might be a duplicate) and the "skip a whole run" logic.

**Ideal Answer:**

Use a dummy because the first node may be deleted. `prev` points to the last confirmed-unique node. When `cur` begins a run of duplicates, skip the entire run, then link `prev.next` past it.

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy, cur = head;
    while (cur != null) {
        if (cur.next != null && cur.val == cur.next.val) {
            int dupVal = cur.val;
            while (cur != null && cur.val == dupVal) cur = cur.next; // skip whole run
            prev.next = cur;            // bypass the duplicates
        } else {
            prev = cur;                 // cur is unique, keep it
            cur = cur.next;
        }
    }
    return dummy.next;
}
```

**Dry run** — `1->2->3->3->4->4->5`:
- 1 unique → prev=1; 2 unique → prev=2; 3==3 → skip run of 3s, cur=4, prev(2).next=4; 4==4 → skip, cur=5, prev(2).next=5; 5 unique → prev=5. → `1->2->5`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Unsorted input?"* → Need a frequency hashmap first (O(n) space), or sort.
2. *"Why a dummy here but optional in Q20?"* → Here the head can be deleted entirely; the dummy gives `prev` a stable anchor.

**Common mistakes:**
- Not using a dummy → can't delete a leading duplicate run.
- Setting `prev.next = cur.next` instead of skipping the *entire* run with the inner while.

---

### Q22: Partition List
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Partition the list around value `x`: all nodes < x come before all nodes ≥ x, preserving the original relative order within each group.

**What interviewer is testing:**
Building two separate chains and stitching them — the stable-partition idiom.

**Ideal Answer:**

Two dummy-headed chains: `less` (values < x) and `greater` (values ≥ x). Append each node to the right chain, then join. Terminate the greater chain to avoid a cycle.

```java
public ListNode partition(ListNode head, int x) {
    ListNode lessDummy = new ListNode(0), greaterDummy = new ListNode(0);
    ListNode less = lessDummy, greater = greaterDummy;
    for (ListNode cur = head; cur != null; cur = cur.next) {
        if (cur.val < x) { less.next = cur; less = less.next; }
        else             { greater.next = cur; greater = greater.next; }
    }
    greater.next = null;             // IMPORTANT: terminate to avoid a cycle
    less.next = greaterDummy.next;   // stitch the two chains
    return lessDummy.next;
}
```

**Dry run** — `1->4->3->2->5->2`, x=3:
- less: 1,2,2 → `1->2->2`; greater: 4,3,5 → `4->3->5`. join → `1->2->2->4->3->5`. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-ups:**
1. *"Why terminate the greater chain?"* → Its last node may still point into the original list, creating a cycle.
2. *"Stable?"* → Yes — nodes are appended in original order within each group.

**Common mistakes:**
- Forgetting `greater.next = null` → infinite loop / cycle.
- Not preserving relative order (e.g., reversing one chain).

---

### Q23: Middle of the Linked List
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return the middle node. If two middles (even length), return the second.

**Ideal Answer:**

Fast/slow: when fast reaches the end, slow is at the middle.

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // for even length, this is the SECOND middle
}
```

**Dry run** — `1->2->3->4->5->6`: slow ends at 4 (second middle). ✅ `1->2->3->4->5`: slow ends at 3. ✅

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Return the first middle for even length?"* → Loop on `fast.next != null && fast.next.next != null`.

**Common mistakes:** Wrong loop condition shifting which middle you get; off-by-one beliefs without dry-running both parities.

---

### Q24: Remove Linked List Elements
**Company:** Amazon, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** OA / Phone

**Question:**
Remove all nodes with `val == target`.

**Ideal Answer:**

Dummy head (head itself may match). `prev` walks; skip matching nodes.

```java
public ListNode removeElements(ListNode head, int val) {
    ListNode dummy = new ListNode(0, head);
    ListNode prev = dummy, cur = head;
    while (cur != null) {
        if (cur.val == val) prev.next = cur.next; // skip
        else prev = cur;                          // keep
        cur = cur.next;
    }
    return dummy.next;
}
```

**Complexity:** O(n) time, O(1) space.

**Follow-up:** *"Recursive?"* → `head.next = removeElements(head.next, val); return head.val == val ? head.next : head;`

**Common mistakes:** No dummy → can't remove a leading run of matches; advancing `prev` past a removed node.

---

### Q25: LRU Cache (linked list + hashmap implementation)
**Company:** Amazon, Google, Meta, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Design a Least Recently Used cache with O(1) `get` and `put`. (File 01 introduced LRU; here we focus on the **doubly linked list + hashmap** internals.)

**What interviewer is testing:**
Why a *doubly* linked list is required, the role of sentinel head/tail nodes, and clean O(1) node splicing. This is the canonical "combine two data structures" design question.

**Ideal Answer:**

- **HashMap** `key -> node` gives O(1) lookup.
- **Doubly linked list** maintains recency order: most-recently-used near the head, least near the tail.
- On `get`/`put`, move the touched node to the front. On overflow, evict the tail.
- *Doubly* linked (not singly) so we can unlink a node in O(1) given just the node — we have its `prev` directly.
- **Sentinel** head and tail nodes remove all null-checks for boundary insertions/removals.

```java
class LRUCache {
    private static class Node {
        int key, value;
        Node prev, next;
        Node(int k, int v) { key = k; value = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // sentinel: most-recent side
    private final Node tail = new Node(0, 0); // sentinel: least-recent side

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) return -1;
        moveToFront(node);     // mark as most recently used
        return node.value;
    }

    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {            // update existing
            node.value = value;
            moveToFront(node);
            return;
        }
        if (map.size() == capacity) {  // evict LRU (node before tail)
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node fresh = new Node(key, value);
        map.put(key, fresh);
        addToFront(fresh);
    }

    // --- doubly linked list helpers (all O(1)) ---
    private void remove(Node n) {
        n.prev.next = n.next;
        n.next.prev = n.prev;
    }
    private void addToFront(Node n) {
        n.next = head.next;
        n.prev = head;
        head.next.prev = n;
        head.next = n;
    }
    private void moveToFront(Node n) {
        remove(n);
        addToFront(n);
    }
}
```

**Dry run** — capacity 2: `put(1,1)`, `put(2,2)`, `get(1)`→1 (1 now MRU), `put(3,3)` evicts key 2 (LRU), `get(2)`→-1, `put(4,4)` evicts key 1, `get(1)`→-1, `get(3)`→3, `get(4)`→4. ✅

**Complexity:** O(1) per `get`/`put`; O(capacity) space.

**Follow-ups:**
1. *"Why doubly, not singly, linked?"* → To unlink an arbitrary node in O(1) we need its predecessor; a singly linked list would require an O(n) scan.
2. *"Use Java's `LinkedHashMap`?"* → Yes — override `removeEldestEntry`; access-order mode gives LRU for free. Good to mention, but interviewers usually want the from-scratch DLL.
3. *"Thread-safe version?"* → Wrap in locks or use a concurrent structure; note the contention trade-offs.
4. *"LFU instead?"* → Q26.

**Common mistakes:**
- Singly linked list → O(n) removal, defeating the design.
- Forgetting to update the hashmap on eviction (`map.remove(lru.key)`).
- No sentinel nodes → a thicket of null checks and edge-case bugs at head/tail.

---

### Q26: LFU Cache
**Company:** Amazon, Google, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Design a Least Frequently Used cache with O(1) `get` and `put`. On a tie in frequency, evict the least recently used among them.

**What interviewer is testing:**
Layering structures: a frequency-indexed map of LRU lists. This is LRU "one level up" and tests whether you can compose data structures under O(1) constraints.

**Ideal Answer:**

Maintain:
- `map: key -> Node` (value + frequency).
- `freqMap: freq -> LinkedHashSet<key>` (keys at that frequency, ordered by recency).
- `minFreq`: the smallest frequency currently present, for O(1) eviction.

On access, bump the key's frequency: remove it from its old freq bucket, add to `freq+1`'s bucket, update `minFreq` if its old bucket emptied. On overflow, evict the LRU key in the `minFreq` bucket.

```java
class LFUCache {
    private final int capacity;
    private int minFreq = 0;
    private final Map<Integer, int[]> map = new HashMap<>();          // key -> [value, freq]
    private final Map<Integer, LinkedHashSet<Integer>> freqMap = new HashMap<>(); // freq -> keys

    public LFUCache(int capacity) { this.capacity = capacity; }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        touch(key);
        return map.get(key)[0];
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (map.containsKey(key)) {
            map.get(key)[0] = value;
            touch(key);
            return;
        }
        if (map.size() == capacity) {                 // evict LFU (LRU on tie)
            LinkedHashSet<Integer> minSet = freqMap.get(minFreq);
            int evict = minSet.iterator().next();      // oldest in min bucket
            minSet.remove(evict);
            map.remove(evict);
        }
        map.put(key, new int[]{value, 1});
        freqMap.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
        minFreq = 1;                                   // new key always freq 1
    }

    private void touch(int key) {
        int freq = map.get(key)[1];
        map.get(key)[1] = freq + 1;
        LinkedHashSet<Integer> set = freqMap.get(freq);
        set.remove(key);
        if (set.isEmpty()) {
            freqMap.remove(freq);
            if (minFreq == freq) minFreq++;            // min bucket emptied -> rise
        }
        freqMap.computeIfAbsent(freq + 1, k -> new LinkedHashSet<>()).add(key);
    }
}
```

**Why `LinkedHashSet`:** it gives O(1) add/remove/contains *and* preserves insertion order, so `iterator().next()` is the least-recently-used key in that frequency bucket.

**Dry run** — capacity 2: `put(1,1)` freq{1:[1]}; `put(2,2)` freq{1:[1,2]}; `get(1)`→1, key1 to freq2 → {1:[2],2:[1]}, minFreq still 1; `put(3,3)` size==2, evict minFreq=1 oldest = key2 → {2:[1],1:[3]}; `get(2)`→-1; `get(3)`→3 key3 to freq2; `put(4,4)` evict minFreq... → behaves per LFU+LRU tie rule. ✅

**Complexity:** O(1) per operation, O(capacity) space.

**Follow-ups:**
1. *"Why track `minFreq` instead of scanning?"* → Scanning freqMap for the minimum would be O(#freqs); the counter keeps eviction O(1). `minFreq` only ever increases on touch or resets to 1 on insert.
2. *"DLL-of-DLL alternative?"* → Replace each `LinkedHashSet` with a custom doubly linked list of nodes for tighter control / to store full node objects.

**Common mistakes:**
- Forgetting to update `minFreq` when the min bucket empties.
- Resetting `minFreq` incorrectly on `put` (a brand-new key has freq 1, so `minFreq = 1`).

---


# STACKS

The most valuable pattern in this entire file lives here: the **monotonic stack**. The idea: keep a stack whose elements are always increasing (or always decreasing). When a new element violates the order, pop the violators — and *each pop is the moment you've found that popped element's answer* (its next greater element, the boundary of its rectangle, etc.).

Recognize a monotonic-stack problem when you hear:
- "next/previous **greater** or **smaller** element"
- "**span**" or "consecutive days/elements until a condition"
- "**largest rectangle**" / "maximal area"
- "sum/count over all subarrays involving min or max"

Other stack patterns: **matching/nesting** (parentheses, calculators, decode-string) and **simulation** (asteroids, path simplification). Java tip: use `ArrayDeque` as a stack (`push`/`pop`/`peek`) — never the legacy `Stack` class (synchronized, slower).

---

### Q27: Valid Parentheses
**Company:** Amazon, Microsoft, Google, Meta, Bloomberg — universal
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Given a string of `()[]{}`, determine if brackets are balanced and properly nested.

**What interviewer is testing:**
The fundamental matching-with-a-stack pattern, plus three edge cases: leftover opens, an unmatched close, and mismatched types.

**Ideal Answer:**

Push opening brackets. On a closing bracket, the top of the stack must be its matching open; otherwise it's invalid. At the end the stack must be empty.

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        switch (c) {
            case '(': stack.push(')'); break;   // push the EXPECTED closer
            case '[': stack.push(']'); break;
            case '{': stack.push('}'); break;
            default:  // closing bracket
                if (stack.isEmpty() || stack.pop() != c) return false;
        }
    }
    return stack.isEmpty(); // leftover opens -> invalid
}
```

**Dry run** — `"([])"`: push `)`, push `]`; see `]` pop `]` match; see `)` pop `)` match; empty → true. ✅ `"(]"`: push `)`; see `]`, pop `)` ≠ `]` → false. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Minimum insertions/removals to balance?"* → Track unmatched opens and closes separately (a counter problem).
2. *"Longest valid parentheses substring?"* → Stack of indices or DP; harder variant.
3. *"What if only one bracket type?"* → A simple counter (no stack) suffices.

**Common mistakes:**
- Not checking the stack is empty at the end (catches `"((("`).
- Popping an empty stack (`"]"` first) → must guard `isEmpty()` before pop.

---

### Q28: Min Stack
**Company:** Amazon, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Design a stack supporting `push`, `pop`, `top`, and `getMin` — all in O(1).

**What interviewer is testing:**
The auxiliary-stack idea (or encoding the min into each frame). The trick is that `getMin` must be O(1) even after pops.

**Ideal Answer:**

Keep a second stack of running minimums. Each push records the min so far; each pop pops both. `getMin` reads the top of the min stack.

```java
class MinStack {
    private final Deque<Integer> stack = new ArrayDeque<>();
    private final Deque<Integer> mins = new ArrayDeque<>(); // running minimum

    public void push(int val) {
        stack.push(val);
        mins.push(mins.isEmpty() ? val : Math.min(val, mins.peek()));
    }
    public void pop()    { stack.pop(); mins.pop(); }
    public int top()     { return stack.peek(); }
    public int getMin()  { return mins.peek(); }
}
```

**Dry run** — push 3,5,2: mins → [3],[3,3],[2,3,3]; getMin=2; pop → mins [3,3]; getMin=3. ✅

**Complexity:** O(1) per operation, O(n) space.

**Single-stack optimization (encode min):** store `2*val - currentMin` when pushing a new minimum, so you can recover the previous min on pop. Saves the auxiliary stack but is trickier and risks overflow — mention it, prefer the two-stack version for clarity.

**Follow-ups:**
1. *"O(1) space overhead?"* → The encoding trick; discuss overflow with `long`.
2. *"Max stack too?"* → Symmetric; or see Q29 (which also needs O(log n) `popMax`).

**Common mistakes:**
- Storing only one min and recomputing after pop (becomes O(n)).
- Forgetting to pop the min stack in lockstep.

---

### Q29: Max Stack
**Company:** Amazon, LinkedIn, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Design a stack with `push`, `pop`, `top`, `peekMax`, and `popMax` (remove and return the maximum element; if ties, the topmost).

**What interviewer is testing:**
Unlike Min Stack, `popMax` removes from the *middle*. The naive two-stack approach makes `popMax` O(n). The interview-grade answer uses a balanced structure (TreeMap) + doubly linked list for O(log n).

**Ideal Answer:**

*Simple O(n) popMax:* one stack plus a temp stack to reach the max. Acceptable as a first cut.

```java
class MaxStackSimple {
    private final Deque<Integer> stack = new ArrayDeque<>();
    public void push(int x) { stack.push(x); }
    public int top() { return stack.peek(); }
    public int pop() { return stack.pop(); }
    public int peekMax() {
        int max = Integer.MIN_VALUE;
        for (int v : stack) max = Math.max(max, v);
        return max;
    }
    public int popMax() {                 // O(n): unwind to the topmost max
        int max = peekMax();
        Deque<Integer> buffer = new ArrayDeque<>();
        while (stack.peek() != max) buffer.push(stack.pop());
        stack.pop();                       // remove the max
        while (!buffer.isEmpty()) stack.push(buffer.pop()); // restore
        return max;
    }
}
```

*Optimal O(log n):* a doubly linked list holds the stack order; a `TreeMap<Integer, List<Node>>` maps value → nodes with that value. `popMax` reads the largest key, removes the last node in its list (topmost), and unlinks it in O(1) from the DLL. This gives O(log n) `push`/`popMax`.

```java
class MaxStack {
    private static class Node { int val; Node prev, next; Node(int v){val=v;} }
    private final Node head = new Node(0), tail = new Node(0);
    private final TreeMap<Integer, List<Node>> map = new TreeMap<>();

    public MaxStack() { head.next = tail; tail.prev = head; }

    private void addNode(Node n) {         // add before tail (top of stack)
        n.prev = tail.prev; n.next = tail;
        tail.prev.next = n; tail.prev = n;
    }
    private void unlink(Node n) {
        n.prev.next = n.next; n.next.prev = n.prev;
    }
    public void push(int x) {
        Node n = new Node(x);
        addNode(n);
        map.computeIfAbsent(x, k -> new ArrayList<>()).add(n);
    }
    public int pop() {
        Node n = tail.prev;
        unlink(n);
        removeFromMap(n);
        return n.val;
    }
    public int top() { return tail.prev.val; }
    public int peekMax() { return map.lastKey(); }
    public int popMax() {
        int max = map.lastKey();
        List<Node> list = map.get(max);
        Node n = list.get(list.size() - 1); // topmost with this value
        unlink(n);
        removeFromMap(n);
        return max;
    }
    private void removeFromMap(Node n) {
        List<Node> list = map.get(n.val);
        list.remove(n);                    // removes this exact node reference
        if (list.isEmpty()) map.remove(n.val);
    }
}
```

**Complexity:** Simple version O(n) `popMax`; optimal O(log n) `push`/`pop`/`popMax`, O(1) `top`/`peekMax`.

**Follow-ups:**
1. *"Why does the TreeMap value need a list, not a single node?"* → Duplicates: many nodes can share a value; on a tie `popMax` must remove the topmost (last added).
2. *"Could a single PriorityQueue work?"* → A heap can't remove the *topmost-among-max* or arbitrary middle elements in O(log n) without lazy deletion bookkeeping.

**Common mistakes:**
- Using a heap and failing the "topmost on tie" rule.
- In the optimal version, removing the wrong node on a value tie (must take the last in the list).

---

### Q30: Evaluate Reverse Polish Notation
**Company:** Amazon, Microsoft, LinkedIn, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Evaluate an arithmetic expression in Reverse Polish (postfix) Notation. Operators are `+ - * /`; division truncates toward zero.

**What interviewer is testing:**
The classic stack evaluation of postfix, and operand-order care for non-commutative `-` and `/`.

**Ideal Answer:**

Scan tokens. Push numbers. On an operator, pop two operands (the **second pop is the left operand**), apply, push the result.

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String t : tokens) {
        switch (t) {
            case "+": stack.push(stack.pop() + stack.pop()); break;
            case "*": stack.push(stack.pop() * stack.pop()); break;
            case "-": { int b = stack.pop(), a = stack.pop(); stack.push(a - b); break; }
            case "/": { int b = stack.pop(), a = stack.pop(); stack.push(a / b); break; }
            default:  stack.push(Integer.parseInt(t));
        }
    }
    return stack.pop();
}
```

**Dry run** — `["2","1","+","3","*"]` = (2+1)*3: push 2,1; `+` → 3; push 3; `*` → 9. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why does operand order matter for `-` and `/`?"* → They're non-commutative; the first popped is the right operand. `+`/`*` are safe either way.
2. *"Convert infix to postfix?"* → Shunting-yard algorithm (precedence-aware).

**Common mistakes:**
- Reversing operands for `-`/`/` (the top of stack is the *right* operand).
- Not handling negative numbers / multi-digit tokens in parsing.

---

### Q31: Daily Temperatures
**Company:** Amazon, Google, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
For each day, how many days until a warmer temperature? `0` if none.

**What interviewer is testing:**
The flagship **monotonic stack** problem. They want you to recognize "for each element, the next greater element" → monotonic decreasing stack of indices.

**Ideal Answer:**

*Brute force:* for each day scan forward — O(n²).

*Monotonic stack (optimal):* keep a stack of indices whose temperatures are in **decreasing** order. When today is warmer than the temperature at the stack's top index, we've found that day's answer — pop and record the distance. Push today's index.

```java
public int[] dailyTemperatures(int[] temps) {
    int n = temps.length;
    int[] answer = new int[n];
    Deque<Integer> stack = new ArrayDeque<>(); // indices, temps decreasing top->down
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temps[i] > temps[stack.peek()]) {
            int prevDay = stack.pop();         // its warmer day is today
            answer[prevDay] = i - prevDay;
        }
        stack.push(i);
    }
    return answer; // indices left on the stack keep answer 0
}
```

**Dry run** — `[73,74,75,71,69,72,76,73]`:
- i0 push0; i1 74>73 pop0 ans[0]=1, push1; i2 75>74 pop1 ans[1]=1, push2; i3 71 push3; i4 69 push4; i5 72>69 pop4 ans[4]=1, 72>71 pop3 ans[3]=2, push5; i6 76>72 pop5 ans[5]=1, 76>75 pop2 ans[2]=4, push6; i7 73 push7.
- answer `[1,1,4,2,1,1,0,0]`. ✅

**Why each element is pushed/popped once → O(n):** every index enters the stack exactly once and leaves at most once; the inner while across the whole run does at most n pops total.

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Next *cooler* day?"* → Flip the comparison to a monotonic *increasing* stack.
2. *"Process from the right instead?"* → Also valid; push as you go right-to-left, popping smaller-or-equal.

**Common mistakes:**
- Using `>=` instead of `>` (a same-temperature day isn't "warmer").
- Storing temperatures instead of indices (you need indices to compute the distance).

---

### Q32: Next Greater Element I
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
For each element of `nums1` (a subset of `nums2`), find its next greater element in `nums2`. Return `-1` if none.

**What interviewer is testing:**
Monotonic stack to precompute next-greater for all of `nums2`, then a hashmap lookup.

**Ideal Answer:**

Build a `value -> nextGreater` map by running a decreasing monotonic stack over `nums2`. Then answer each query in `nums1` via the map.

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>(); // values, decreasing
    for (int x : nums2) {
        while (!stack.isEmpty() && x > stack.peek()) {
            nextGreater.put(stack.pop(), x); // x is the next greater for popped
        }
        stack.push(x);
    }
    // remaining have no next greater -> -1 (handled by getOrDefault)
    int[] res = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        res[i] = nextGreater.getOrDefault(nums1[i], -1);
    }
    return res;
}
```

**Dry run** — nums2 `[1,3,4,2]`: push1; 3>1 pop1→{1:3}, push3; 4>3 pop3→{3:4}, push4; 2<4 push2. map {1:3,3:4}. queries `[4,1,2]` → `[-1,3,-1]`. ✅

**Complexity:** O(n + m) time, O(n) space.

**Follow-ups:**
1. *"Circular array?"* → Q33.
2. *"Values not unique?"* → A value→next map breaks; use index-based stacks instead.

**Common mistakes:**
- Assuming `nums1` order matches `nums2` (it doesn't — use the map).

---

### Q33: Next Greater Element II (Circular)
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
The array is **circular** (the next of the last element is the first). For each element, find the next greater element; `-1` if none.

**What interviewer is testing:**
Handling circularity by iterating `2n` times with modular indexing while keeping the monotonic stack.

**Ideal Answer:**

Iterate `i` from `0` to `2n-1`, using `i % n` to wrap. Only push real indices on the first pass (`i < n`); the second pass just resolves elements that wrap around.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] res = new int[n];
    Arrays.fill(res, -1);
    Deque<Integer> stack = new ArrayDeque<>(); // indices, decreasing values
    for (int i = 0; i < 2 * n; i++) {
        int cur = nums[i % n];
        while (!stack.isEmpty() && cur > nums[stack.peek()]) {
            res[stack.pop()] = cur;
        }
        if (i < n) stack.push(i);   // only push during the first lap
    }
    return res;
}
```

**Dry run** — `[1,2,1]`: i0 push0; i1 2>1 pop0 res[0]=2, push1; i2 1<2 push2; i3(0) 1<2; i4(1) 2: 2>nums[2]=1 pop2 res[2]=2; i5(2). → `[2,-1,2]`. ✅

**Complexity:** O(n) time (2n iterations), O(n) space.

**Follow-ups:**
1. *"Why `2n` and not more?"* → Two laps suffice: anything not resolved after seeing every element twice has no greater element.
2. *"Don't push on the second lap?"* → Correct — `if (i < n)` prevents duplicates.

**Common mistakes:**
- Pushing during the second lap → duplicate/incorrect results.
- Forgetting `% n` indexing.

---

### Q34: Largest Rectangle in Histogram
**Company:** Amazon, Google, Microsoft, Meta, Uber — a marquee FAANG hard
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given bar heights of unit width, find the area of the largest rectangle that fits within the histogram.

**What interviewer is testing:**
The deepest monotonic-stack reasoning in this file: for each bar, the largest rectangle *with that bar as the limiting height* extends left until a shorter bar and right until a shorter bar. The stack finds both boundaries in one pass. Interviewers love this because the naive O(n²) is obvious but the O(n) stack solution requires real insight into *what a pop means*.

**Ideal Answer:**

*Brute force:* for each pair (i, j) compute min height × width — O(n²) or O(n³). Reject.

*Better brute force:* for each bar `i`, expand left and right while bars are ≥ height[i]. O(n²) worst case.

*Optimal — monotonic increasing stack:* maintain a stack of indices with **increasing** heights. When the incoming bar is shorter than the bar at the top, that top bar can't extend further right — so we **pop it and finalize its rectangle**. The popped bar's height is the limiting height; its right boundary is the current index `i`; its left boundary is the new stack top (the previous shorter bar). Width = `i - stack.peek() - 1`.

The trick to flush the stack cleanly at the end: append a sentinel height `0` so every remaining bar gets popped and measured.

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    Deque<Integer> stack = new ArrayDeque<>(); // indices, heights increasing
    int maxArea = 0;
    for (int i = 0; i <= n; i++) {
        int curHeight = (i == n) ? 0 : heights[i];   // sentinel 0 flushes the stack
        while (!stack.isEmpty() && curHeight < heights[stack.peek()]) {
            int height = heights[stack.pop()];        // bar being finalized
            // left boundary is the bar now on top (or -1 if stack empty)
            int leftBoundary = stack.isEmpty() ? -1 : stack.peek();
            int width = i - leftBoundary - 1;         // i is the right boundary (exclusive)
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**The key insight, spelled out:** when we pop bar `p` because the current bar `i` is shorter, we know:
- The **right** boundary of `p`'s maximal rectangle is `i` — bar `i` is the first bar to the right strictly shorter than `p`.
- The **left** boundary is the bar now at the top of the stack — by the increasing-stack invariant, every bar between it and `p` was ≥ `p`'s height (they were taller and already popped, or `p` itself), so `p`'s rectangle spans `(leftBoundary, i)` exclusive.
- Width = `i - leftBoundary - 1`.

**Detailed dry run** — `[2,1,5,6,2,3]`:
- i0 h2: stack empty, push0. stack[0]
- i1 h1: 1<2 → pop0 (h2), left=-1, width=1-(-1)-1=1, area=2; push1. stack[1]
- i2 h5: 5>1 push2. stack[1,2]
- i3 h6: 6>5 push3. stack[1,2,3]
- i4 h2: 2<6 → pop3 (h6), left=2, width=4-2-1=1, area6; 2<5 → pop2 (h5), left=1, width=4-1-1=2, area=**10**; 2>1 push4. stack[1,4]
- i5 h3: 3>2 push5. stack[1,4,5]
- i6 sentinel 0: 0<3 pop5 (h3) left=4 width=6-4-1=1 area3; 0<2 pop4 (h2) left=1 width=6-1-1=4 area8; 0<1 pop1 (h1) left=-1 width=6-(-1)-1=6 area6; push6.
- maxArea = **10** (the 5-6 region: height 5 × width 2). ✅

**Why O(n):** each index is pushed once and popped once across the whole run.

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Maximal rectangle in a binary matrix?"* → Q35 — run this per row treating column heights as a histogram.
2. *"Why the sentinel 0?"* → Without it, bars left on the stack at the end (a non-decreasing suffix) never get measured. The 0 forces them all to pop.
3. *"Alternative without a sentinel?"* → After the loop, pop the rest using `n` as the right boundary; the sentinel is just cleaner.

**Common mistakes:**
- Computing width as `i - stack.peek()` (off by one) instead of `i - leftBoundary - 1`.
- Forgetting the `stack.isEmpty() -> left = -1` case (rectangle reaches the far left edge).
- Not flushing the stack at the end → missing the largest rectangle when heights are non-decreasing.

---

### Q35: Maximal Rectangle
**Company:** Amazon, Google, Microsoft, Facebook
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a binary matrix of `0`s and `1`s, find the area of the largest rectangle containing only `1`s.

**What interviewer is testing:**
Reducing a 2D problem to a sequence of 1D histogram problems (Q34). The reduction insight is the whole game.

**Ideal Answer:**

Process the matrix row by row. Maintain a `heights[]` array where `heights[c]` is the number of consecutive `1`s ending at the current row in column `c`. After updating heights for each row, the largest rectangle whose bottom is on that row equals the Largest Rectangle in Histogram of `heights[]`.

```java
public int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0 || matrix[0].length == 0) return 0;
    int cols = matrix[0].length;
    int[] heights = new int[cols];
    int maxArea = 0;
    for (char[] row : matrix) {
        for (int c = 0; c < cols; c++) {
            heights[c] = (row[c] == '1') ? heights[c] + 1 : 0; // grow or reset
        }
        maxArea = Math.max(maxArea, largestRectangleArea(heights));
    }
    return maxArea;
}
// largestRectangleArea from Q34
```

**Dry run** — matrix
```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```
- row0 heights [1,0,1,0,0] → max 1
- row1 heights [2,0,2,1,1] → histogram max 2
- row2 heights [3,1,3,2,2] → histogram max = **6** (the 3×2 region at the right)
- row3 heights [4,0,0,3,0] → max 4
- Answer 6. ✅

**Complexity:** O(rows × cols) time — each row's histogram is O(cols). O(cols) space.

**Follow-ups:**
1. *"Maximal *square* (not rectangle)?"* → Simpler DP: `dp[i][j] = min(up, left, up-left) + 1` if cell is 1.
2. *"Why does the histogram reduction work?"* → Any all-ones rectangle has a bottom row; we consider every possible bottom row and find the best rectangle resting on it.

**Common mistakes:**
- Not resetting `heights[c] = 0` on a `0` (a gap breaks the column run).
- Re-deriving histogram logic incorrectly instead of reusing Q34.

---

### Q36: Basic Calculator
**Company:** Amazon, Google, Microsoft, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Evaluate a string expression containing non-negative integers, `+`, `-`, `(`, `)`, and spaces. No `*` or `/`.

**What interviewer is testing:**
Handling parentheses and sign propagation with a stack. The elegant approach pushes the running result and sign on `(`.

**Ideal Answer:**

Track a running `result`, the current `number`, and the current `sign` (+1/−1). On `(`, push the current `result` and `sign` so they can be restored when the matching `)` arrives. On `)`, fold the parenthesized result back: `result = result * pushedSign + pushedResult`.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int result = 0, number = 0, sign = 1;
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            number = number * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * number; number = 0; sign = 1;
        } else if (c == '-') {
            result += sign * number; number = 0; sign = -1;
        } else if (c == '(') {
            stack.push(result);     // save context
            stack.push(sign);
            result = 0; sign = 1;   // start fresh inside the parens
        } else if (c == ')') {
            result += sign * number; number = 0;
            result *= stack.pop();  // apply the sign before '('
            result += stack.pop();  // add the result before '('
        }
        // spaces ignored
    }
    return result + sign * number;  // flush the last number
}
```

**Dry run** — `"(1+(4+5+2)-3)+(6+8)"`:
- `(` push(0,1) reset; `1` num1; `+` result1 sign1 num0; `(` push(1,1) reset; `4+5+2` → result11; `)` result11, *pushedSign1 → 11, +pushedResult1 → 12; `-` result12 sign-1; `3` num3; `)` result=12-3=9; `+(6+8)` → 14. Total 23. ✅

**Complexity:** O(n) time, O(n) space (stack depth = nesting).

**Follow-ups:**
1. *"Add `*` and `/`?"* → Q37/Q38 — precedence handling.
2. *"Why push both result and sign?"* → On `)` you must reattach the parenthesized value to the prior result with the sign that preceded `(`.

**Common mistakes:**
- Forgetting to flush the trailing number after the loop.
- Not resetting `result`/`sign` after pushing on `(`.

---

### Q37: Basic Calculator II
**Company:** Amazon, Google, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Evaluate a string with non-negative integers and `+ - * /` (no parentheses). Integer division truncates toward zero. Respect operator precedence.

**What interviewer is testing:**
Precedence via a stack: defer `+`/`-` by pushing signed numbers, but apply `*`/`/` immediately to the stack top.

**Ideal Answer:**

Carry the previous operator. When you finish reading a number and hit the next operator (or string end):
- prev `+` → push `+number`
- prev `-` → push `-number`
- prev `*` → pop, push `top * number`
- prev `/` → pop, push `top / number`

The final answer is the sum of the stack.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int number = 0;
    char op = '+';                 // operator preceding the current number
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) number = number * 10 + (c - '0');
        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            switch (op) {
                case '+': stack.push(number); break;
                case '-': stack.push(-number); break;
                case '*': stack.push(stack.pop() * number); break;
                case '/': stack.push(stack.pop() / number); break;
            }
            op = c;                // remember this operator for the next number
            number = 0;
        }
    }
    int sum = 0;
    for (int v : stack) sum += v;  // add deferred + / - terms
    return sum;
}
```

**Dry run** — `"3+2*2"`: op=+,num3 at `+` push3, op=+,num2 at `*` push2, op=*,num2 at end → pop2*2=4 push4. stack{3,4} sum=7. ✅ `" 3/2 "` → push3/... at end pop... = 1. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Add parentheses?"* → Q38, combine with the recursion/stack-of-contexts from Q36.
2. *"O(1) space?"* → Track only the last term instead of a full stack: keep `result` and `lastNum`; on `*`/`/` adjust `lastNum`.

**Common mistakes:**
- Applying `*`/`/` at the wrong time (must be when the *next* operator is seen, using the *previous* operator).
- Forgetting the `i == last` flush for the final number.

---

### Q38: Basic Calculator III
**Company:** Amazon, Google, Facebook
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Evaluate a string with non-negative integers, `+ - * /`, **and** parentheses, respecting precedence.

**What interviewer is testing:**
Composing precedence (Q37) with parentheses (Q36). The clean approach is recursion: when you hit `(`, recursively evaluate the substring until the matching `)`.

**Ideal Answer:**

Use a single index pointer and a recursive helper that evaluates until it consumes a `)` or the end. Inside, use the Q37 precedence logic; when a `(` appears, recurse to get the parenthesized value as the current "number."

```java
public int calculate(String s) {
    int[] idx = {0};
    return eval(s, idx);
}

private int eval(String s, int[] idx) {
    Deque<Integer> stack = new ArrayDeque<>();
    int number = 0;
    char op = '+';
    while (idx[0] < s.length()) {
        char c = s.charAt(idx[0]);
        idx[0]++;
        if (Character.isDigit(c)) {
            number = number * 10 + (c - '0');
        } else if (c == '(') {
            number = eval(s, idx);          // recurse: value of the parenthesized block
        }
        if ((c != ' ' && !Character.isDigit(c)) || idx[0] == s.length()) {
            switch (op) {
                case '+': stack.push(number); break;
                case '-': stack.push(-number); break;
                case '*': stack.push(stack.pop() * number); break;
                case '/': stack.push(stack.pop() / number); break;
            }
            number = 0;
            op = c;
            if (c == ')') break;            // finished this parenthesized scope
        }
    }
    int sum = 0;
    for (int v : stack) sum += v;
    return sum;
}
```

**Dry run** — `"2*(5+5*2)/3+(6/2+8)"`:
- inner `(5+5*2)` → 15; `2*15/3` → 10; inner `(6/2+8)` → 11; `10 + 11` = 21. ✅

**Complexity:** O(n) time (each char processed once across recursion), O(n) space (recursion + stack).

**Follow-ups:**
1. *"Iterative without recursion?"* → Stack of contexts as in Q36 combined with the precedence handling — more bookkeeping.
2. *"Unary minus / negative numbers?"* → Pre-process or special-case a `-` that follows `(` or another operator.

**Common mistakes:**
- Sharing the index incorrectly across recursion (use a single mutable `int[] idx`).
- Not breaking out of the loop on `)` → consuming beyond the current scope.

---

### Q39: Decode String
**Company:** Amazon, Google, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Decode a string like `"3[a2[c]]"` → `"accaccacc"`. Encoding: `k[encoded]` means repeat `encoded` k times. k is a positive integer.

**What interviewer is testing:**
Nested-structure decoding with **two stacks** (counts and partial strings). A favorite because the nesting trips up naive single-pass attempts.

**Ideal Answer:**

Two stacks: one for repeat counts, one for the string built *before* a `[`. On `[`, push the count and the current string, then reset. On `]`, pop the count and the previous string, and append `count × current` to it.

```java
public String decodeString(String s) {
    Deque<Integer> countStack = new ArrayDeque<>();
    Deque<StringBuilder> strStack = new ArrayDeque<>();
    StringBuilder cur = new StringBuilder();
    int k = 0;
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            k = k * 10 + (c - '0');           // multi-digit counts
        } else if (c == '[') {
            countStack.push(k);
            strStack.push(cur);
            k = 0;
            cur = new StringBuilder();
        } else if (c == ']') {
            int repeat = countStack.pop();
            StringBuilder prev = strStack.pop();
            for (int i = 0; i < repeat; i++) prev.append(cur);
            cur = prev;                        // restore the outer context
        } else {
            cur.append(c);                     // a literal character
        }
    }
    return cur.toString();
}
```

**Dry run** — `"3[a2[c]]"`:
- `3` k3; `[` push(3),push(""), reset; `a` cur="a"; `2` k2; `[` push(2),push("a"),reset; `c` cur="c"; `]` repeat2,prev="a" → "acc", cur="acc"; `]` repeat3,prev="" → "accaccacc". ✅

**Complexity:** O(maxK × n) time (output can blow up), O(n) space for the stacks.

**Follow-ups:**
1. *"Recursive approach?"* → Recurse on each `[...]`; the two-stack version is the iterative equivalent.
2. *"Multi-digit k like `10[a]`?"* → Handled by `k = k*10 + digit`.

**Common mistakes:**
- Single-digit assumption for k.
- Appending in the wrong order on `]` (the prev/outer string must come *before* the repeated inner).

---

### Q40: Remove K Digits
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a numeric string `num` and an integer `k`, remove `k` digits so the resulting number is the smallest possible. No leading zeros (except `"0"` itself).

**What interviewer is testing:**
Greedy + monotonic stack: to minimize, remove a digit whenever it's larger than the digit after it (a "peak"). The stack keeps the result in increasing order.

**Ideal Answer:**

Build a monotonic **increasing** stack of digits. For each digit, while the top is greater than the current digit and we still have removals left, pop (remove the bigger leading digit). If `k` remains after the scan, drop from the end (the suffix is non-decreasing). Finally strip leading zeros.

```java
public String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>(); // increasing from bottom
    for (char c : num.toCharArray()) {
        while (!stack.isEmpty() && k > 0 && stack.peek() > c) {
            stack.pop();    // removing a larger leading digit makes it smaller
            k--;
        }
        stack.push(c);
    }
    while (k > 0) { stack.pop(); k--; } // leftover removals from the end

    // build result from bottom of stack, stripping leading zeros
    StringBuilder sb = new StringBuilder();
    Iterator<Character> it = stack.descendingIterator(); // bottom -> top order
    boolean leadingZero = true;
    while (it.hasNext()) {
        char c = it.next();
        if (leadingZero && c == '0') continue;
        leadingZero = false;
        sb.append(c);
    }
    return sb.length() == 0 ? "0" : sb.toString();
}
```

**Dry run** — `num="1432219"`, k=3:
- 1 push; 4 push; 3<4 pop4(k2) push3; stack[1,3]; 2<3 pop3(k1) push2 [1,2]; 2 push [1,2,2]; 1<2 pop2(k0) push1 [1,2,1]; 9 push.
- result bottom→top "1219". ✅ (smallest is 1219)

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why is greedy correct?"* → Removing the first digit that's greater than its successor always reduces the most significant possible place; an exchange argument shows no better removal exists.
2. *"Largest number after removing k?"* → Flip to a decreasing stack (pop when top < current).

**Common mistakes:**
- Not handling leftover `k` (when digits are non-decreasing, remove from the end).
- Forgetting to strip leading zeros; returning `""` instead of `"0"`.

---

### Q41: Asteroid Collision
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Asteroids move along a line; positive = right, negative = left, value = size. When two collide, the smaller explodes; equal sizes both explode. Return the state after all collisions. (Two moving the same direction never collide.)

**What interviewer is testing:**
Stack-based simulation: a right-mover sits on the stack waiting; a left-mover collides with stack tops until resolved.

**Ideal Answer:**

Push asteroids onto a stack. A collision happens only when the current asteroid moves **left** (`< 0`) and the stack top moves **right** (`> 0`). Resolve repeatedly: smaller explodes; equal → both explode; otherwise the current one survives the smaller top and continues.

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int a : asteroids) {
        boolean alive = true;
        while (alive && a < 0 && !stack.isEmpty() && stack.peek() > 0) {
            int top = stack.peek();
            if (top < -a) {        // top (smaller) explodes; current keeps going
                stack.pop();
                continue;
            } else if (top == -a) { // both explode
                stack.pop();
                alive = false;
            } else {                // current (smaller) explodes
                alive = false;
            }
        }
        if (alive) stack.push(a);
    }
    // build result in order (bottom -> top)
    int[] res = new int[stack.size()];
    for (int i = res.length - 1; i >= 0; i--) res[i] = stack.pop();
    return res;
}
```

**Dry run** — `[5,10,-5]`: push5,push10; -5 vs top10 → 10>5 so -5 explodes; result `[5,10]`. ✅ `[8,-8]`: push8; -8 vs8 equal → both explode; `[]`. ✅ `[10,2,-5]`: push10,2; -5 vs2 → 2<5 pop2; -5 vs10 → 10>5 -5 explodes; `[10]`. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why only collide when current < 0 and top > 0?"* → Same direction never meets; `+ then -` means they move toward each other.
2. *"Return count of survivors only?"* → Track size, no array build.

**Common mistakes:**
- Pushing the left-mover before resolving collisions.
- Mishandling the `equal -> both explode` case.

---

### Q42: Online Stock Span
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Design `StockSpanner.next(price)` returning the span: the number of consecutive days (including today) the price was ≤ today's price, going backward.

**What interviewer is testing:**
A streaming monotonic stack storing (price, span) pairs, collapsing prior days that are ≤ today.

**Ideal Answer:**

Keep a stack of `[price, span]`. For each new price, pop all entries with price ≤ current, accumulating their spans into today's span. Push `[price, accumulatedSpan]`.

```java
class StockSpanner {
    private final Deque<int[]> stack = new ArrayDeque<>(); // [price, span]

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];     // absorb spans of dominated days
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

**Dry run** — prices `[100,80,60,70,60,75,85]`:
- 100→1; 80→1; 60→1; 70: pop60(span1) → 2; 60→1; 75: pop60(1), pop70(2) → 4; 85: pop75(4), pop80(1), 100>85 stop → 6.
- spans `[1,1,1,2,1,4,6]`. ✅

**Complexity:** Amortized O(1) per `next` (each price pushed/popped once), O(n) space.

**Follow-ups:**
1. *"Why store spans, not raw prices?"* → Collapsing dominated days into one entry keeps the stack short and the amortized cost O(1).
2. *"Span of *smaller* prices?"* → Flip the comparison.

**Common mistakes:**
- Using `<` instead of `<=` (the problem counts days with price ≤ today).
- Recomputing spans by re-scanning history (O(n) per call).

---

### Q43: Simplify Path
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Simplify a Unix-style absolute file path. Handle `.` (current), `..` (parent), and multiple slashes.

**What interviewer is testing:**
Stack-based path simulation and tokenizing on `/`.

**Ideal Answer:**

Split on `/`. Push real directory names; on `..` pop (if non-empty); ignore `.` and empty tokens. Rebuild with single slashes.

```java
public String simplifyPath(String path) {
    Deque<String> stack = new ArrayDeque<>();
    for (String part : path.split("/")) {
        if (part.isEmpty() || part.equals(".")) {
            continue;                       // skip "" (from //) and "."
        } else if (part.equals("..")) {
            if (!stack.isEmpty()) stack.pop();  // go up, but not above root
        } else {
            stack.push(part);
        }
    }
    StringBuilder sb = new StringBuilder();
    Iterator<String> it = stack.descendingIterator(); // bottom -> top
    while (it.hasNext()) sb.append('/').append(it.next());
    return sb.length() == 0 ? "/" : sb.toString();
}
```

**Dry run** — `"/a/./b/../../c/"`: tokens a,.,b,..,..,c → push a; skip .; push b; .. pop b; .. pop a; push c → `/c`. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Relative paths?"* → Different semantics; `..` at root could remain.
2. *"Symbolic links?"* → Out of scope for pure string simplification.

**Common mistakes:**
- Popping an empty stack on `..` at root → must guard.
- Forgetting the empty-result case returns `"/"`.

---

### Q44: Implement Queue using Stacks
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Implement a FIFO queue using only two stacks, with **amortized O(1)** operations.

**What interviewer is testing:**
The two-stack trick and amortized analysis — moving elements only when the output stack is empty.

**Ideal Answer:**

`in` stack receives pushes. `out` stack serves pops/peeks. When `out` is empty and we need to dequeue, dump all of `in` into `out` (reversing order → FIFO).

```java
class MyQueue {
    private final Deque<Integer> in = new ArrayDeque<>();
    private final Deque<Integer> out = new ArrayDeque<>();

    public void push(int x) { in.push(x); }

    public int pop() {
        transfer();
        return out.pop();
    }
    public int peek() {
        transfer();
        return out.peek();
    }
    public boolean empty() { return in.isEmpty() && out.isEmpty(); }

    private void transfer() {        // only refill out when it's empty
        if (out.isEmpty()) {
            while (!in.isEmpty()) out.push(in.pop());
        }
    }
}
```

**Dry run** — push1,push2,peek,pop,push3,pop: push→in[1,2]; peek→transfer out[1,2] front 1; pop returns 1, out[2]; push3 in[3]; pop→out not empty, returns 2. dequeue order 1,2,3 ✅

**Complexity:** Amortized O(1) per op — each element is moved between stacks at most once. Worst-case single op O(n).

**Follow-ups:**
1. *"Prove amortized O(1)."* → Each element is pushed to `in` once, moved to `out` once, popped once: 3 ops per element total over its lifetime.
2. *"Single stack with recursion?"* → Possible but O(n) per op; the two-stack is preferred.

**Common mistakes:**
- Transferring on every operation (becomes O(n)).
- Transferring when `out` is non-empty (corrupts order).

---

### Q45: Implement Stack using Queues
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥
**Round:** Phone

**Question:**
Implement a LIFO stack using queues.

**What interviewer is testing:**
The reverse problem; the clever single-queue solution makes `push` O(n) and everything else O(1).

**Ideal Answer:**

Single queue. On `push`, enqueue the element, then rotate the queue so the new element comes to the front (rotate `size-1` times). Now the queue's front is always the most recently pushed = stack top.

```java
class MyStack {
    private final Deque<Integer> q = new ArrayDeque<>();

    public void push(int x) {
        q.offer(x);
        for (int i = 1; i < q.size(); i++) q.offer(q.poll()); // rotate
    }
    public int pop()  { return q.poll(); }
    public int top()  { return q.peek(); }
    public boolean empty() { return q.isEmpty(); }
}
```

**Dry run** — push1: q[1]; push2: q[2,1] (rotate 1); push3: q[3,2,1]; pop→3; top→2. LIFO ✅

**Complexity:** `push` O(n), `pop`/`top`/`empty` O(1). Space O(n).

**Follow-ups:**
1. *"Make pop O(n) instead and push O(1)?"* → Two-queue variant moving all-but-one on pop.
2. *"Which is better?"* → Depends on the push/pop ratio; the single-queue O(n) push is simplest.

**Common mistakes:** Off-by-one in the rotation count (`size - 1` rotations).

---

### Q46: The Celebrity Problem
**Company:** Amazon, Microsoft, Google, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Among `n` people, a celebrity is known by everyone but knows no one. Given `knows(a, b)`, find the celebrity (or −1). Minimize calls.

**What interviewer is testing:**
The stack/elimination insight: each `knows` query eliminates exactly one candidate, reducing n candidates to 1 in n−1 queries.

**Ideal Answer:**

*Stack elimination:* push all people. Repeatedly pop two candidates `a`, `b`. If `a knows b`, `a` can't be the celebrity (push `b` back); else `b` can't be (push `a` back). One candidate remains — then verify it (knows nobody, known by everybody).

```java
public int findCelebrity(int n) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < n; i++) stack.push(i);
    while (stack.size() > 1) {
        int a = stack.pop(), b = stack.pop();
        if (knows(a, b)) stack.push(b);  // a is not celeb (knows someone)
        else stack.push(a);              // b is not celeb (someone doesn't know them)
    }
    int cand = stack.pop();
    // verify
    for (int i = 0; i < n; i++) {
        if (i == cand) continue;
        if (knows(cand, i) || !knows(i, cand)) return -1;
    }
    return cand;
}
// stub for compilation: boolean knows(int a, int b)
private boolean knows(int a, int b) { return false; }
```

**Two-pointer equivalent (O(1) space):**
```java
public int findCelebrityTwoPointer(int n) {
    int cand = 0;
    for (int i = 1; i < n; i++) if (knows(cand, i)) cand = i; // eliminate
    for (int i = 0; i < n; i++) {                              // verify
        if (i == cand) continue;
        if (knows(cand, i) || !knows(i, cand)) return -1;
    }
    return cand;
}
```

**Dry run** — n=3, celeb=1 (everyone knows 1, 1 knows no one): elimination finds candidate 1; verification passes. ✅

**Complexity:** O(n) queries to find the candidate + O(n) to verify = O(n) total. Stack version O(n) space; two-pointer O(1).

**Follow-ups:**
1. *"Why does one query eliminate one person?"* → If `a knows b`, `a` is definitely not the celebrity; if not, `b` definitely isn't. Either way one is ruled out.
2. *"Why verify at the end?"* → Elimination guarantees the *only possible* celebrity, not that one exists.

**Common mistakes:**
- Skipping verification (a candidate emerges even when no celebrity exists).
- O(n²) brute force when O(n) elimination is expected.

---

### Q47: Remove All Adjacent Duplicates in String
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Repeatedly remove adjacent equal characters until none remain. Return the result.

**What interviewer is testing:**
A stack as a "running result" where the top is compared with the incoming character.

**Ideal Answer:**

Use a stack (or `StringBuilder` as a stack). If the incoming char equals the top, pop (they cancel); otherwise push.

```java
public String removeDuplicates(String s) {
    StringBuilder sb = new StringBuilder(); // used as a stack
    for (char c : s.toCharArray()) {
        int len = sb.length();
        if (len > 0 && sb.charAt(len - 1) == c) sb.deleteCharAt(len - 1); // cancel
        else sb.append(c);
    }
    return sb.toString();
}
```

**Dry run** — `"abbaca"`: a→a; b→ab; b==b pop→a; a==a pop→""; c→c; a→ca → "ca". ✅

**Complexity:** O(n) time, O(n) space.

**Follow-up:** Q48 — remove runs of exactly `k`.

**Common mistakes:** Re-scanning the whole string repeatedly (O(n²)) instead of the single-pass stack.

---

### Q48: Remove All Adjacent Duplicates in String II
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Remove `k` adjacent equal characters, repeatedly, until no such group remains. Return the result.

**What interviewer is testing:**
A stack of (char, count) pairs — counting runs and removing when the count reaches k.

**Ideal Answer:**

Stack of `[char, count]`. For each char, if it matches the top, increment the count; otherwise push a new pair. When the count hits `k`, pop. Rebuild the string at the end.

```java
public String removeDuplicates(String s, int k) {
    Deque<int[]> stack = new ArrayDeque<>(); // [charCode, count]
    for (char c : s.toCharArray()) {
        if (!stack.isEmpty() && stack.peek()[0] == c) {
            if (++stack.peek()[1] == k) stack.pop(); // full group -> remove
        } else {
            stack.push(new int[]{c, 1});
        }
    }
    StringBuilder sb = new StringBuilder();
    Iterator<int[]> it = stack.descendingIterator();
    while (it.hasNext()) {
        int[] pair = it.next();
        for (int i = 0; i < pair[1]; i++) sb.append((char) pair[0]);
    }
    return sb.toString();
}
```

Note: `++stack.peek()[1]` mutates the array in place — valid because `peek()` returns the same reference.

**Dry run** — `"deeedbbcccbdaa"`, k=3:
- d[1]; e[1]→[2]→[3] pop; d[1]; b[1]→[2]; c[1]→[2]→[3] pop; b becomes [3] pop; d... → after full processing "aa". ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"k = 2?"* → Reduces to Q47.
2. *"Why store counts vs pushing each char?"* → Counting collapses a run into one entry; O(1) to detect a full group.

**Common mistakes:**
- Pushing each character individually then scanning for runs (O(n²)).
- Forgetting to pop when count reaches exactly k.

---

### Q49: Sum of Subarray Minimums
**Company:** Amazon, Google, Goldman Sachs
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return the sum of `min(b)` over every contiguous subarray `b` of `arr`, modulo 1e9+7.

**What interviewer is testing:**
The "contribution technique" with a monotonic stack: instead of enumerating O(n²) subarrays, count for each element how many subarrays it is the minimum of.

**Ideal Answer:**

For each element `arr[i]`, count the subarrays where it is the minimum. That equals `left × right`, where:
- `left` = number of consecutive elements to the left that are **strictly greater** (distance to the previous *less-or-equal* element).
- `right` = number to the right that are **greater or equal** (distance to the next *strictly less* element).

The asymmetry (`>` on one side, `>=` on the other) avoids double-counting equal minimums. Then `arr[i]` contributes `arr[i] × left × right`.

Use a monotonic increasing stack to compute previous-less-element (PLE) and next-less-element (NLE) boundaries.

```java
public int sumSubarrayMins(int[] arr) {
    int MOD = 1_000_000_007;
    int n = arr.length;
    int[] ple = new int[n]; // distance to previous strictly-less element
    int[] nle = new int[n]; // distance to next less-or-equal element
    Deque<Integer> stack = new ArrayDeque<>();

    // previous less element (strict): pop while top >= arr[i]
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) stack.pop();
        ple[i] = stack.isEmpty() ? i + 1 : i - stack.peek();
        stack.push(i);
    }
    stack.clear();
    // next less element (non-strict): pop while top > arr[i]
    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && arr[stack.peek()] > arr[i]) stack.pop();
        nle[i] = stack.isEmpty() ? n - i : stack.peek() - i;
        stack.push(i);
    }
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum = (sum + (long) arr[i] * ple[i] * nle[i]) % MOD;
    }
    return (int) sum;
}
```

**Dry run** — `[3,1,2,4]`:
- ple: 3→1; 1→2; 2→1; 4→1.
- nle: 3→1; 1→3; 2→2; 4→1.
- contributions: 3·1·1=3; 1·2·3=6; 2·1·2=4; 4·1·1=4 → sum 17. ✅

**Complexity:** O(n) time, O(n) space.

**Follow-ups:**
1. *"Why strict on one side, non-strict on the other?"* → To count each subarray's minimum exactly once when duplicates exist.
2. *"Sum of subarray *maximums*?"* → Mirror with decreasing stacks.
3. *"Sum of subarray ranges (max − min)?"* → Do both and subtract.

**Common mistakes:**
- Using the same strictness on both sides → double counting with duplicates.
- Integer overflow — `arr[i] * left * right` must be `long`.

---

### Q50: Trapping Rain Water (stack view)
**Company:** Amazon, Google, Goldman Sachs
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Compute trapped rainwater over an elevation map. (File 01 covered the two-pointer solution; here we present the **monotonic stack** view, which interviewers ask as an alternative.)

**What interviewer is testing:**
Whether you can articulate the stack solution: water is trapped in "valleys" bounded by a taller bar on the left (on the stack) and the current taller bar on the right.

**Ideal Answer:**

Maintain a stack of indices with **decreasing** heights. When the current bar is taller than the bar at the top, that top is the bottom of a valley: pop it, and the trapped water above it is bounded by the new stack top (left wall) and the current bar (right wall).

```java
public int trap(int[] height) {
    Deque<Integer> stack = new ArrayDeque<>(); // indices, heights decreasing
    int water = 0;
    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[i] > height[stack.peek()]) {
            int bottom = stack.pop();           // bottom of the valley
            if (stack.isEmpty()) break;         // no left wall -> no water
            int left = stack.peek();
            int boundedHeight = Math.min(height[left], height[i]) - height[bottom];
            int width = i - left - 1;
            water += boundedHeight * width;
        }
        stack.push(i);
    }
    return water;
}
```

**Dry run** — `[0,1,0,2,1,0,1,3,2,1,2,1]`:
- The stack pops valleys as taller bars arrive; e.g. at the `2` (index3) it pops the `0` and `1` valleys, adding water bounded by `1` and `2`. Total water = **6**. ✅

**Complexity:** O(n) time, O(n) space. (The two-pointer version from file 01 is O(1) space and usually preferred — be ready to give both.)

**Follow-ups:**
1. *"Two-pointer vs stack — which is better?"* → Two-pointer is O(1) space and simpler; the stack computes water layer-by-layer horizontally, useful intuition for related valley problems.
2. *"Trapping Rain Water II (2D)?"* → Min-heap from the borders inward (Dijkstra-like), separate hard problem.

**Common mistakes:**
- Adding water before there's a left wall (`stack.isEmpty()` after pop → no water).
- Using `>=` and double-processing flat regions.

---


# QUEUES

Queues power three interview themes: **multi-source BFS** (the shortest-path / spreading pattern — rotten oranges, walls and gates), **sliding/streaming windows** (moving averages, recent calls, hit counters), and **scheduling/design** (task scheduler, custom deques). The mental trigger for BFS: "minimum steps / shortest time / spread simultaneously from several sources" → put **all sources in the queue at once** and expand level by level.

Java tip: use `ArrayDeque` for both stack and queue roles; for a queue use `offer`/`poll`/`peek`.

---

### Q51: Design Circular Queue
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Implement a circular queue (ring buffer) with `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull`, fixed capacity `k`.

**What interviewer is testing:**
Modular index arithmetic and the full-vs-empty disambiguation (track a `count` or leave one slot empty).

**Ideal Answer:**

Use a fixed array with `head` index and a `count` of elements. `enQueue` writes at `(head + count) % capacity`; `deQueue` advances `head`. Tracking `count` cleanly distinguishes full from empty (both would otherwise have `head == tail`).

```java
class MyCircularQueue {
    private final int[] data;
    private final int capacity;
    private int head = 0;     // index of the front element
    private int count = 0;    // current number of elements

    public MyCircularQueue(int k) {
        capacity = k;
        data = new int[k];
    }
    public boolean enQueue(int value) {
        if (isFull()) return false;
        int tail = (head + count) % capacity; // next write slot
        data[tail] = value;
        count++;
        return true;
    }
    public boolean deQueue() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;          // move front forward
        count--;
        return true;
    }
    public int Front() { return isEmpty() ? -1 : data[head]; }
    public int Rear()  { return isEmpty() ? -1 : data[(head + count - 1) % capacity]; }
    public boolean isEmpty() { return count == 0; }
    public boolean isFull()  { return count == capacity; }
}
```

**Dry run** — k=3: enQueue 1,2,3 (full); enQueue 4 → false; Rear=3; deQueue (head→1); enQueue 4 writes at `(1+2)%3=0`; Rear=4. ✅

**Complexity:** O(1) per operation, O(k) space.

**Follow-ups:**
1. *"Why track count instead of head/tail only?"* → With head==tail you can't tell full from empty; `count` (or sacrificing one slot) disambiguates.
2. *"Double-ended version?"* → Q52.

**Common mistakes:**
- Off-by-one in `Rear` index (`head + count - 1`, mod capacity).
- Full/empty confusion when using only two pointers.

---

### Q52: Design Circular Deque
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Implement a circular double-ended queue: insert/delete at both front and rear, fixed capacity `k`.

**What interviewer is testing:**
Extending the ring buffer to both ends; careful wrap-around when decrementing the front.

**Ideal Answer:**

Same array + head + count model. Inserting at the front means decrementing `head` modulo capacity (with a `+capacity` to avoid negatives).

```java
class MyCircularDeque {
    private final int[] data;
    private final int capacity;
    private int head = 0, count = 0;

    public MyCircularDeque(int k) { capacity = k; data = new int[k]; }

    public boolean insertFront(int value) {
        if (isFull()) return false;
        head = (head - 1 + capacity) % capacity; // step back, wrap safely
        data[head] = value;
        count++;
        return true;
    }
    public boolean insertLast(int value) {
        if (isFull()) return false;
        data[(head + count) % capacity] = value;
        count++;
        return true;
    }
    public boolean deleteFront() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;
        count--;
        return true;
    }
    public boolean deleteLast() {
        if (isEmpty()) return false;
        count--;                                  // simply drop the last slot
        return true;
    }
    public int getFront() { return isEmpty() ? -1 : data[head]; }
    public int getRear()  { return isEmpty() ? -1 : data[(head + count - 1) % capacity]; }
    public boolean isEmpty() { return count == 0; }
    public boolean isFull()  { return count == capacity; }
}
```

**Dry run** — k=3: insertLast 1; insertLast 2; insertFront 3 → head steps back, data front=3; getFront=3; getRear=2; deleteLast → drops 2; getRear=1. ✅

**Complexity:** O(1) per operation, O(k) space.

**Follow-ups:**
1. *"Why `(head - 1 + capacity) % capacity`?"* → Java's `%` can yield negatives; adding `capacity` keeps the index in range.
2. *"Linked-list implementation?"* → Doubly linked list with sentinels also gives O(1) at both ends.

**Common mistakes:**
- Negative index from `(head - 1) % capacity` without the `+ capacity`.
- Forgetting that `deleteLast` only needs to decrement count.

---

### Q53: Rotting Oranges
**Company:** Amazon, Google, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
A grid has `0` (empty), `1` (fresh orange), `2` (rotten). Each minute, a rotten orange rots its 4-directionally adjacent fresh oranges. Return the minutes until no fresh orange remains, or `-1` if impossible.

**What interviewer is testing:**
**Multi-source BFS** — all initially-rotten oranges start in the queue together and the rot spreads level-by-level, each level = one minute.

**Ideal Answer:**

Seed the queue with *all* rotten oranges and count fresh ones. BFS level by level; each level is one minute. When a fresh orange is reached, rot it and decrement the fresh count. If fresh oranges remain at the end, return −1.

```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> queue = new ArrayDeque<>();
    int fresh = 0;
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c}); // all sources
            else if (grid[r][c] == 1) fresh++;
        }
    if (fresh == 0) return 0;

    int minutes = 0;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!queue.isEmpty() && fresh > 0) {
        minutes++;
        int size = queue.size();             // process exactly one level
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;        // rot it
                    fresh--;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;        // leftover fresh -> unreachable
}
```

**Dry run** — `[[2,1,1],[1,1,0],[0,1,1]]`:
- minute1: rot the two neighbors of (0,0). minute2: spread. ... minute4: last fresh rots. Answer 4. ✅
- `[[2,1,1],[0,1,1],[1,0,1]]`: the bottom-left fresh orange is isolated → returns -1.

**Why level-by-level matters:** capturing `queue.size()` at the start of each iteration processes exactly the oranges that rotted in the previous minute, so `minutes` counts true elapsed time.

**Complexity:** O(rows × cols) time and space.

**Follow-ups:**
1. *"Why multi-source BFS, not run BFS from each rotten orange?"* → Single combined BFS gives the simultaneous-spread minimum in one pass; per-source BFS would be O(sources × cells).
2. *"8-directional spread?"* → Add the diagonal directions.

**Common mistakes:**
- Not capturing `queue.size()` per level → miscounts minutes.
- Forgetting the `fresh == 0` early return (a grid with no fresh oranges is 0 minutes, not −1).

---

### Q54: Walls and Gates
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A grid of rooms: `-1` is a wall, `0` is a gate, `INF` (2³¹−1) is an empty room. Fill each empty room with its distance to the nearest gate (leave INF if unreachable).

**What interviewer is testing:**
Again **multi-source BFS** — start from *all* gates simultaneously; the first time BFS reaches a room is its shortest distance.

**Ideal Answer:**

Enqueue all gates. BFS outward; each empty room (still INF) gets `currentDistance + 1` the first time it's reached, then is enqueued. Because BFS expands by distance, the first visit is the minimum.

```java
public void wallsAndGates(int[][] rooms) {
    if (rooms.length == 0) return;
    int rows = rooms.length, cols = rooms[0].length;
    final int INF = Integer.MAX_VALUE;
    Queue<int[]> queue = new ArrayDeque<>();
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (rooms[r][c] == 0) queue.offer(new int[]{r, c}); // all gates

    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        for (int[] d : dirs) {
            int nr = cell[0] + d[0], nc = cell[1] + d[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && rooms[nr][nc] == INF) {
                rooms[nr][nc] = rooms[cell[0]][cell[1]] + 1; // distance from nearest gate
                queue.offer(new int[]{nr, nc});
            }
        }
    }
}
```

**Dry run** — a gate at (0,0) fills its neighbors with 1, theirs with 2, etc.; walls (−1) are skipped; rooms boxed off by walls stay INF. ✅

**Why no explicit distance counter:** each room derives its distance from the cell that discovered it (`+1`), so the value is carried in the grid itself.

**Complexity:** O(rows × cols) time and space.

**Follow-ups:**
1. *"Why BFS not DFS?"* → BFS guarantees shortest distance on first visit; DFS could assign a longer path first and require revisits.
2. *"Single gate?"* → Same code; just one source in the queue.

**Common mistakes:**
- Running BFS separately from each gate (slower, and you must take the min).
- Overwriting rooms that already have a smaller distance — the `== INF` check prevents it.

---

### Q55: Moving Average from Data Stream
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Design a class that computes the moving average of the last `size` values from a stream.

**What interviewer is testing:**
A fixed-window queue with a running sum for O(1) updates.

**Ideal Answer:**

A queue holds the window; maintain a running `sum`. On `next`, add the value and evict the oldest if the window overflows, adjusting `sum`.

```java
class MovingAverage {
    private final Queue<Integer> window = new ArrayDeque<>();
    private final int size;
    private double sum = 0;

    public MovingAverage(int size) { this.size = size; }

    public double next(int val) {
        window.offer(val);
        sum += val;
        if (window.size() > size) {
            sum -= window.poll();   // drop the oldest, adjust the running sum
        }
        return sum / window.size();
    }
}
```

**Dry run** — size 3: next(1)→1; next(10)→5.5; next(3)→4.667; next(5)→ window[10,3,5] sum18/3=6. ✅

**Complexity:** O(1) per `next`, O(size) space.

**Follow-ups:**
1. *"Why a running sum instead of summing the window each time?"* → Summing would be O(size) per call; the running sum makes it O(1).
2. *"Circular array instead of a queue?"* → Same idea, fixed buffer; avoids object churn.

**Common mistakes:** Recomputing the sum each call (O(size)); not adjusting `sum` on eviction.

---

### Q56: Task Scheduler
**Company:** Amazon, Google, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given tasks (letters) and a cooldown `n`, the same task must be separated by at least `n` intervals. Return the minimum number of intervals (including idles) to finish all tasks.

**What interviewer is testing:**
Two approaches: a greedy math formula based on the most frequent task, or a max-heap + cooldown queue simulation. The formula is the elegant answer; the simulation generalizes.

**Ideal Answer:**

*Greedy formula:* the busiest task `maxFreq` forces a skeleton of `(maxFreq − 1)` full cooldown frames of length `(n + 1)`, plus a final frame holding all tasks tied at `maxFreq`. So:

`intervals = max(tasks.length, (maxFreq − 1) * (n + 1) + countOfMaxFreqTasks)`

The `max` with `tasks.length` handles the case where there are so many distinct tasks that no idling is ever needed.

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    int maxFreq = 0;
    for (int f : freq) maxFreq = Math.max(maxFreq, f);
    int countMax = 0;
    for (int f : freq) if (f == maxFreq) countMax++;

    int slots = (maxFreq - 1) * (n + 1) + countMax;
    return Math.max(tasks.length, slots);
}
```

*Heap + queue simulation (handles variants / shows the mechanism):*

```java
public int leastIntervalSim(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    PriorityQueue<Integer> heap = new PriorityQueue<>(Collections.reverseOrder());
    for (int f : freq) if (f > 0) heap.offer(f);

    Queue<int[]> cooldown = new ArrayDeque<>(); // [remainingCount, availableTime]
    int time = 0;
    while (!heap.isEmpty() || !cooldown.isEmpty()) {
        time++;
        if (!heap.isEmpty()) {
            int remaining = heap.poll() - 1;       // run one instance
            if (remaining > 0) cooldown.offer(new int[]{remaining, time + n});
        }
        if (!cooldown.isEmpty() && cooldown.peek()[1] == time) {
            heap.offer(cooldown.poll()[0]);        // task cooled down, reinsert
        }
    }
    return time;
}
```

**Dry run** — tasks `[A,A,A,B,B,B]`, n=2: maxFreq=3, countMax=2 → slots = (3-1)*3 + 2 = 8 = `A B _ A B _ A B`. max(6, 8)=8. ✅
- tasks `[A,A,A,B,B,B,C,C,C,D,D,E]`, n=2: enough distinct tasks → answer = tasks.length = 12.

**Complexity:** Formula O(tasks + 26) time, O(1) space. Simulation O(time · log 26) time.

**Follow-ups:**
1. *"Why does the formula work?"* → The most frequent task dictates the schedule's backbone; idles fill gaps only when there aren't enough other tasks.
2. *"Output the actual schedule?"* → Use the heap simulation, recording which task runs each interval.

**Common mistakes:**
- Forgetting the `max(tasks.length, …)` clamp → wrong when distinct tasks fill all idles.
- Mis-counting `countMax` (number of tasks sharing the maximum frequency).

---

### Q57: Number of Recent Calls
**Company:** Amazon, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Implement `RecentCounter.ping(t)`: each call gives a strictly increasing time `t`; return how many pings happened in the window `[t - 3000, t]`.

**What interviewer is testing:**
A sliding window over a stream with a queue — evict timestamps that fell out of the window.

**Ideal Answer:**

Queue of timestamps. On each `ping`, enqueue `t`, then dequeue from the front while the oldest is older than `t - 3000`. The queue size is the answer.

```java
class RecentCounter {
    private final Queue<Integer> queue = new ArrayDeque<>();

    public int ping(int t) {
        queue.offer(t);
        while (queue.peek() < t - 3000) queue.poll(); // drop out-of-window pings
        return queue.size();
    }
}
```

**Dry run** — ping(1)→1; ping(100)→2; ping(3001)→3 (1 still in [1,3001]); ping(3002)→ drop 1 (1 < 3002-3000=2) → 3. ✅

**Complexity:** Amortized O(1) per `ping` (each timestamp enqueued/dequeued once), O(window) space.

**Follow-ups:**
1. *"Different window size at query time?"* → Binary search over a stored list instead of a fixed-window queue.
2. *"Why amortized O(1)?"* → Each timestamp is removed at most once over its lifetime.

**Common mistakes:** Using `<=` vs `<` on the window edge (window is inclusive of `t - 3000`, so evict only when strictly less).

---

### Q58: Design Hit Counter
**Company:** Amazon, Google, Dropbox
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Design a hit counter: `hit(timestamp)` records a hit; `getHits(timestamp)` returns the number of hits in the past 5 minutes (300 seconds).

**What interviewer is testing:**
A sliding window with a queue, plus the scaling follow-up: what if hits are huge? → bucket by second.

**Ideal Answer:**

*Queue approach:* store each hit timestamp; on `getHits`, evict timestamps ≤ `timestamp - 300` and return the size.

```java
class HitCounter {
    private final Queue<Integer> hits = new ArrayDeque<>();

    public void hit(int timestamp) { hits.offer(timestamp); }

    public int getHits(int timestamp) {
        while (!hits.isEmpty() && hits.peek() <= timestamp - 300) hits.poll();
        return hits.size();
    }
}
```

*Bucketed approach (scales to high frequency):* two arrays of size 300 — `times[i]` and `counts[i]`. Bucket index = `timestamp % 300`. If the bucket's stored time differs from the current second, reset it; otherwise increment. `getHits` sums buckets whose stored time is within 300 seconds.

```java
class HitCounterBucketed {
    private final int[] times = new int[300];
    private final int[] counts = new int[300];

    public void hit(int timestamp) {
        int idx = timestamp % 300;
        if (times[idx] != timestamp) {   // stale bucket -> reset
            times[idx] = timestamp;
            counts[idx] = 1;
        } else {
            counts[idx]++;
        }
    }
    public int getHits(int timestamp) {
        int total = 0;
        for (int i = 0; i < 300; i++) {
            if (timestamp - times[i] < 300) total += counts[i];
        }
        return total;
    }
}
```

**Dry run (bucketed)** — hit(1),hit(1),hit(2); getHits(4) → buckets at idx1 (count2,time1) and idx2 (count1,time2), both within 300 → 3. getHits(301) → idx1 time1: 301-1=300 not <300 → excluded; idx2 time2: 301-2=299<300 included(1). → 1. ✅

**Complexity:** Queue: O(1) amortized `hit`, O(hits-in-window) `getHits`. Bucketed: O(1) `hit`, O(300)=O(1) `getHits`, O(300) space regardless of traffic.

**Follow-ups:**
1. *"Billions of hits per second?"* → Bucketed approach is the answer — fixed memory, no per-hit storage.
2. *"Granularity other than seconds?"* → Adjust bucket count and modulus.
3. *"Concurrent hits?"* → Synchronize per-bucket or use atomic counters.

**Common mistakes:**
- Storing every hit when traffic is huge (memory blowup) — mention the bucketed trade-off.
- Off-by-one on the 300-second window edge.

---

### Q59: Design Front Middle Back Queue
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Design a queue supporting push/pop at the **front, middle, and back**. When there are two middles, push to the *front* middle and pop from the *front* middle.

**What interviewer is testing:**
Two deques kept balanced so the middle is always at a deque boundary, giving O(1) middle operations.

**Ideal Answer:**

Two deques `left` and `right`. Maintain the invariant `left.size()` is either equal to `right.size()` or one less (`right` may hold the extra element). After each op, rebalance. The middle sits at the boundary between the two deques.

```java
class FrontMiddleBackQueue {
    private final Deque<Integer> left = new ArrayDeque<>();
    private final Deque<Integer> right = new ArrayDeque<>();

    public void pushFront(int val) {
        left.offerFirst(val);
        balance();
    }
    public void pushMiddle(int val) {
        if (left.size() == right.size()) right.offerFirst(val);
        else left.offerLast(val);   // left had the extra -> push to its end (the middle)
        balance();
    }
    public void pushBack(int val) {
        right.offerLast(val);
        balance();
    }
    public int popFront() {
        if (isEmpty()) return -1;
        int v = !left.isEmpty() ? left.pollFirst() : right.pollFirst();
        balance();
        return v;
    }
    public int popMiddle() {
        if (isEmpty()) return -1;
        int v;
        if (left.size() == right.size()) v = left.pollLast(); // two middles -> front one
        else v = right.pollFirst();                            // right has the extra middle
        balance();
        return v;
    }
    public int popBack() {
        if (isEmpty()) return -1;
        int v = right.pollLast();
        balance();
        return v;
    }
    private boolean isEmpty() { return left.isEmpty() && right.isEmpty(); }

    // invariant: left.size() == right.size() OR left.size() + 1 == right.size()
    private void balance() {
        if (left.size() > right.size()) {
            right.offerFirst(left.pollLast());
        } else if (right.size() > left.size() + 1) {
            left.offerLast(right.pollFirst());
        }
    }
}
```

**Dry run** — pushFront(1): left[],right[1] after balance; pushBack(2): right[1,2]→balance left[1],right[2]; pushMiddle(3): sizes equal → right.offerFirst → right[3,2], balance → left[1],right[3,2]? rebalance keeps middle correct. popMiddle → returns 3. ✅

**Complexity:** O(1) per operation, O(n) space.

**Follow-ups:**
1. *"Why two deques?"* → The middle is always at the boundary; balancing keeps it O(1) to reach, unlike a single list (O(n) middle access).
2. *"Linked-list with a middle pointer?"* → Possible but the middle pointer maintenance on every op is error-prone.

**Common mistakes:**
- Letting the size invariant drift (rebalance after *every* operation).
- Wrong choice of "front middle" when two middles exist.

---

### Q60: First Unique Number
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Design a class on a stream of integers: `showFirstUnique()` returns the first value that has appeared exactly once (or −1); `add(value)` appends to the stream.

**What interviewer is testing:**
Combining a queue (insertion order of candidates) with a frequency/count map — the same "queue + hashmap" pairing as First Unique Character (file 01), extended to a live stream.

**Ideal Answer:**

A queue holds *candidate* unique values in arrival order, and a count map tracks occurrences. On `showFirstUnique`, lazily evict front candidates that are no longer unique (count > 1); the front is then the answer.

```java
class FirstUnique {
    private final Queue<Integer> queue = new ArrayDeque<>();
    private final Map<Integer, Integer> count = new HashMap<>();

    public FirstUnique(int[] nums) {
        for (int n : nums) add(n);
    }
    public int showFirstUnique() {
        // lazily drop non-unique values from the front
        while (!queue.isEmpty() && count.get(queue.peek()) > 1) queue.poll();
        return queue.isEmpty() ? -1 : queue.peek();
    }
    public void add(int value) {
        count.merge(value, 1, Integer::sum);
        if (count.get(value) == 1) queue.offer(value); // only enqueue first sighting
    }
}
```

**Dry run** — add stream `[2,3,5]`: showFirstUnique→2; add(5): count{5:2}; showFirstUnique→2 (5 no longer unique but it isn't at front); add(2): count{2:2}; showFirstUnique→ evict 2 (count2) → 3. ✅

**Why lazy eviction:** we don't remove from the middle of the queue (O(n)); instead, stale candidates are skipped only when they reach the front during a query, keeping operations amortized O(1).

**Complexity:** Amortized O(1) `add` and `showFirstUnique`; O(n) space.

**Follow-ups:**
1. *"Why not a LinkedHashMap?"* → A `LinkedHashMap<value, isUnique>` also works (iterate to the first unique), but the queue + count gives clean amortized O(1) without scanning.
2. *"First unique *character* in a string stream?"* → Same structure over chars (file 01, Q53).

**Common mistakes:**
- Enqueuing a value on every `add` (duplicates pile up); enqueue only on the first sighting.
- Eagerly removing from the middle of the queue (O(n)) instead of lazy front eviction.

---

## Pattern recap — what to recognize on sight

| Trigger in problem statement | Pattern | Examples here |
|---|---|---|
| "reverse a list" / "reverse in groups" | Iterative pointer flip / dummy head | Q1, Q2, Q14, Q18 |
| "cycle / loop / repeated value via pointers" | Floyd's tortoise & hare | Q3, Q4 |
| "middle / nth from end / palindrome list" | Fast & slow two pointers | Q7, Q10, Q23 |
| "merge sorted lists" / "sort a list" | Merge primitive / merge sort | Q5, Q6, Q15 |
| "deep copy with extra pointer" | Hashmap or node interleaving | Q12 |
| "O(1) get/put with eviction" | Doubly linked list + hashmap | Q25, Q26 |
| "matching / nested brackets / decode / calculator" | Stack (matching & nesting) | Q27, Q36, Q37, Q38, Q39 |
| "next/previous greater or smaller" | **Monotonic stack** | Q31, Q32, Q33 |
| "largest rectangle / maximal area" | **Monotonic stack** (boundaries) | Q34, Q35 |
| "span / stock / consecutive until condition" | **Monotonic stack** (streaming) | Q42 |
| "sum/count over all subarrays w/ min or max" | **Monotonic stack** (contribution) | Q49 |
| "remove k to optimize the number" | Greedy + monotonic stack | Q40 |
| "simulate collisions / cancellations" | Stack simulation | Q41, Q47, Q48 |
| "FIFO from LIFO or vice versa" | Two-stack / rotating queue | Q44, Q45 |
| "shortest time / spread from many sources" | **Multi-source BFS** (queue) | Q53, Q54 |
| "moving average / recent calls / hits in window" | Sliding-window queue | Q55, Q57, Q58 |
| "schedule with cooldown" | Greedy formula / heap + cooldown queue | Q56 |
| "front/middle/back ops O(1)" | Two balanced deques | Q59 |
| "first unique in a stream" | Queue + frequency map (lazy evict) | Q60 |

**The three questions to ask yourself on any linked-list / stack / queue problem:**
1. **Pointers:** would a dummy head and named `prev/cur/next` pointers eliminate the head-change edge case? (Almost always yes for lists.)
2. **Monotonic stack:** does the problem mention next/previous greater/smaller, span, area, or per-subarray min/max? If so, reach for a monotonic stack before anything else.
3. **BFS:** is it "minimum steps" or "spread from sources"? Put all sources in a queue and expand level by level.

**Monotonic stack — the one pattern to over-learn.** Every monotonic-stack problem reduces to: maintain an increasing (or decreasing) stack of indices; *the moment you pop an element is the moment you've found its answer* (its next-greater, its rectangle boundary, the subarrays it dominates). Internalize that single sentence and Q31–Q35, Q40, Q42, Q49, and Q50 all collapse into one idea.

---

Created **02-dsa-linked-lists-stacks-queues.md** — 60 questions covered (26 linked lists, 24 stacks, 10 queues), each with brute→optimal, complete Java, dry runs, complexity proofs, follow-ups, and rejection-causing mistakes. The monotonic-stack pattern is threaded through eight problems and called out explicitly. Ready for next? (Next up: **03-dsa-trees-and-graphs.md**.)
