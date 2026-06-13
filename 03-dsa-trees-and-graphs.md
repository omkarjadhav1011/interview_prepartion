# 03 — DSA: Trees & Graphs

> The structural heart of the onsite. Once a loop moves past arrays/strings, it almost always tests whether you can reason about **recursion on trees** and **traversal on graphs**. These two topics share a single mental model — "a graph is a tree that forgot to be acyclic" — and mastering them unlocks BST, heaps, tries, shortest paths, topological order, and union-find. Every question has a **complete** answer — brute force → optimal, full Java, dry run, complexity proof, follow-ups, and the mistakes that get you rejected.

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the *real* signal. On trees this is almost always "can you pick the right traversal and define the right recursive contract"; on graphs it's "can you model the problem as a graph and choose BFS vs DFS vs Dijkstra vs union-find."
- **Ideal Answer** — approach reasoning, then production-quality Java, then complexity.
- **Follow-ups** — where most candidates fail. These are scripted in real loops.
- **Common mistakes** — the specific things that flip a "lean hire" to "no hire."

The meta-skill is **pattern recognition**. By the end you should read a problem and say *"this is post-order because the parent needs child results"* or *"this is multi-source BFS"* or *"this is topological sort because of dependencies"* before writing a line.

A note on the reusable templates: the BFS, DFS, topological-sort, and union-find templates in this file are the ones you should be able to write from muscle memory. Roughly 70% of graph questions are one of those four with a thin wrapper.

---

## Table of Contents

**Binary Trees**
1. Binary Tree Inorder Traversal (recursive + iterative)
2. Binary Tree Preorder Traversal (recursive + iterative)
3. Binary Tree Postorder Traversal (recursive + iterative, one-stack + two-stack)
4. Level Order Traversal (BFS)
5. Maximum Depth of Binary Tree
6. Minimum Depth of Binary Tree
7. Balanced Binary Tree (O(n))
8. Diameter of Binary Tree
9. Same Tree
10. Symmetric Tree
11. Subtree of Another Tree
12. Invert Binary Tree
13. Lowest Common Ancestor of a Binary Tree
14. Path Sum I
15. Path Sum II
16. Path Sum III (prefix sum)
17. Binary Tree Maximum Path Sum
18. Construct Binary Tree from Preorder and Inorder
19. Construct Binary Tree from Postorder and Inorder
20. Serialize and Deserialize Binary Tree
21. Binary Tree Right Side View
22. Flatten Binary Tree to Linked List
23. Vertical Order Traversal
24. Boundary Traversal
25. Binary Tree Zigzag Level Order Traversal
26. Count Complete Tree Nodes (O(log² n))
27. Morris Inorder Traversal (O(1) space)
28. Populating Next Right Pointers in Each Node
29. Sum Root to Leaf Numbers
30. Count Good Nodes in Binary Tree
31. Maximum Width of Binary Tree
32. All Nodes Distance K in Binary Tree
33. House Robber III
34. Binary Tree Cameras

**Binary Search Trees**
35. Search in a BST
36. Insert into a BST
37. Delete Node in a BST
38. Convert Sorted Array to BST
39. Convert Sorted List to BST
40. Validate Binary Search Tree
41. Kth Smallest Element in a BST
42. Kth Largest Element in a BST
43. Inorder Successor in BST
44. BST Iterator
45. Recover Binary Search Tree
46. Lowest Common Ancestor of a BST
47. Two Sum IV — Input is a BST

**Heaps / Priority Queues**
48. Kth Largest Element in an Array (QuickSelect + heap)
49. Find Median from Data Stream
50. Reorganize String
51. K Closest Points to Origin
52. IPO
53. Smallest Range Covering Elements from K Lists
54. Design Twitter
55. Sliding Window Median
56. Ugly Number II
57. Kth Largest Element in a Stream
58. Sort Characters by Frequency
59. Merge K Sorted Lists (heap framing)
60. Task Scheduler

**Tries**
61. Implement Trie (Prefix Tree)
62. Design Add and Search Words Data Structure
63. Word Search II
64. Longest Word in Dictionary
65. Replace Words
66. Maximum XOR of Two Numbers in an Array
67. Map Sum Pairs

**Graphs**
68. Number of Islands (DFS / BFS / Union-Find)
69. Clone Graph
70. Pacific Atlantic Water Flow
71. Surrounded Regions
72. 01 Matrix
73. Rotting Oranges
74. Number of Connected Components in an Undirected Graph
75. Number of Provinces
76. Course Schedule
77. Course Schedule II
78. Alien Dictionary
79. Word Ladder
80. Word Ladder II
81. Union-Find / DSU Template
82. Redundant Connection
83. Accounts Merge
84. Number of Operations to Make Network Connected
85. Graph Valid Tree
86. Is Graph Bipartite?
87. Detect Cycle in a Directed Graph
88. Detect Cycle in an Undirected Graph
89. Minimum Height Trees
90. Dijkstra's Algorithm
91. Network Delay Time
92. Cheapest Flights Within K Stops
93. Path with Maximum Probability
94. Bellman-Ford Algorithm
95. Floyd-Warshall Algorithm
96. Minimum Spanning Tree — Kruskal's
97. Minimum Spanning Tree — Prim's
98. Critical Connections in a Network (Tarjan's Bridges)
99. Reconstruct Itinerary (Hierholzer's)
100. Shortest Path in Binary Matrix

**Reusable Templates**
- BFS template
- DFS template
- Topological sort template
- Union-Find template

---

# BINARY TREES

The single most important tree idea: **decide what each recursive call returns and promises, then trust the recursion.** Most tree solutions are one of three traversals (pre/in/post-order) plus a clearly defined recursive contract. Post-order ("compute children first, then combine") solves the majority of "compute something about the whole tree" problems; BFS solves the "by level" problems.

We use this node definition throughout:

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val; this.left = left; this.right = right;
    }
}
```

---

### Q1: Binary Tree Inorder Traversal
**Company:** Amazon, Microsoft, Google, Bloomberg — everywhere
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Return the inorder traversal (left, root, right) of a binary tree's node values.

**What interviewer is testing:**
Do you know the three DFS orders cold, and can you convert the trivial recursion into an explicit-stack iterative version? The iterative form is the real signal — it shows you understand that recursion is just a stack.

**Ideal Answer:**

*Recursive* — the textbook definition: traverse left subtree, visit node, traverse right subtree.

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    inorder(root, res);
    return res;
}
private void inorder(TreeNode node, List<Integer> res) {
    if (node == null) return;
    inorder(node.left, res);   // left
    res.add(node.val);         // root
    inorder(node.right, res);  // right
}
```

*Iterative with explicit stack* — push left spine, pop and visit, then move right. The invariant: the stack holds the chain of ancestors whose left subtrees we've fully descended but not yet visited.

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {       // go as far left as possible
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();          // leftmost unvisited
        res.add(cur.val);           // visit
        cur = cur.right;            // then its right subtree
    }
    return res;
}
```

**Dry run** — tree `1 -> right 2 -> left 3` (i.e. `[1,null,2]` with `2.left = 3`):
- cur=1: push 1, cur=1.left=null. Pop 1 → visit 1. cur=2.
- cur=2: push 2, cur=2.left=3 → push 3, cur=null. Pop 3 → visit 3. cur=null.
- Pop 2 → visit 2. cur=null. Stack empty.
- Result `[1,3,2]`. ✅

**Complexity:** Time O(n) — each node pushed and popped once. Space O(h) for the stack, where h is height (O(n) worst skewed, O(log n) balanced).

**Pattern:** DFS traversal via explicit stack — the foundation for BST iterator (Q44) and many others.

**Follow-up questions the interviewer will ask:**
1. *"Inorder of a BST gives what?"* → A sorted ascending sequence. This is the workhorse fact behind Validate BST, Kth Smallest, Recover BST.
2. *"Can you do it in O(1) space?"* → Morris traversal (Q27), which temporarily threads the tree using right pointers of predecessors.
3. *"What changes for preorder/postorder iteratively?"* → Preorder visits before descending; postorder is trickier (Q3).

**Common mistakes that get you rejected:**
- Forgetting the `cur != null || !stack.isEmpty()` compound condition — using only `!stack.isEmpty()` terminates early when the stack momentarily empties but `cur` still points to a right subtree.
- Visiting the node before exhausting the left spine (that's preorder, not inorder).

---

### Q2: Binary Tree Preorder Traversal
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return the preorder traversal (root, left, right).

**What interviewer is testing:**
Same as inorder, but the iterative version has a subtle ordering point — push right child *before* left so left is processed first.

**Ideal Answer:**

*Recursive:*

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    preorder(root, res);
    return res;
}
private void preorder(TreeNode node, List<Integer> res) {
    if (node == null) return;
    res.add(node.val);          // root
    preorder(node.left, res);   // left
    preorder(node.right, res);  // right
}
```

*Iterative:* push root; pop and visit; push right then left (so left pops first — a stack reverses order).

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        res.add(node.val);                       // visit on pop
        if (node.right != null) stack.push(node.right); // right first
        if (node.left != null)  stack.push(node.left);  // left on top
    }
    return res;
}
```

**Dry run** — tree `[1,2,3]` (root 1, left 2, right 3): push 1. Pop 1 → visit; push 3, push 2. Pop 2 → visit (no children). Pop 3 → visit. Result `[1,2,3]`. ✅

**Complexity:** O(n) time, O(h) space.

**Pattern:** Preorder = the order you'd serialize a tree top-down; basis for serialize/deserialize (Q20) and clone.

**Follow-ups:**
1. *"Why push right before left?"* → A stack is LIFO; to visit left first you push it last.
2. *"Preorder of a tree uniquely determines it?"* → No, not alone; you need inorder too (or null markers), which is Q18 / Q20.

**Common mistakes:**
- Pushing left before right (reverses the output).
- Visiting on push instead of pop while not maintaining the right order.

---

### Q3: Binary Tree Postorder Traversal
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Return the postorder traversal (left, right, root) iteratively.

**What interviewer is testing:**
Postorder is the hardest of the three to do iteratively because the root must wait for *both* children. Two clean tricks: (a) the "reversed preorder" two-stack trick, and (b) a one-stack approach tracking the last visited node.

**Ideal Answer:**

*Recursive:*

```java
private void postorder(TreeNode node, List<Integer> res) {
    if (node == null) return;
    postorder(node.left, res);
    postorder(node.right, res);
    res.add(node.val);
}
```

*Two-stack trick (cleanest):* Do a modified preorder that visits root, **right, left**, then reverse the result. Modified preorder = root,right,left; reversed = left,right,root = postorder.

```java
public List<Integer> postorderTraversal(TreeNode root) {
    LinkedList<Integer> res = new LinkedList<>();
    if (root == null) return res;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        res.addFirst(node.val);                          // prepend -> reverses
        if (node.left != null)  stack.push(node.left);   // so right pops first
        if (node.right != null) stack.push(node.right);
    }
    return res;
}
```

*One-stack approach (no reversal):* track `lastVisited`; only visit a node after its right child is done.

```java
public List<Integer> postorderOneStack(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root, lastVisited = null;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.left; }
        TreeNode peek = stack.peek();
        // if right child exists and hasn't been visited yet, go right
        if (peek.right != null && lastVisited != peek.right) {
            cur = peek.right;
        } else {
            res.add(peek.val);
            lastVisited = stack.pop();
        }
    }
    return res;
}
```

**Dry run (two-stack)** — `[1,2,3]`: push 1. Pop 1 → res=[1]; push 2, push 3. Pop 3 → res=[3,1]. Pop 2 → res=[2,3,1]. Result `[2,3,1]`. ✅ (left=2, right=3, root=1).

**Complexity:** O(n) time, O(h) space.

**Pattern:** Postorder = "process children before parent" — the backbone of nearly every "compute a value for the whole tree" problem (height, diameter, max path sum, balanced check).

**Follow-ups:**
1. *"Why does reversed (root,right,left) equal postorder?"* → Postorder is (left,right,root); its reverse is (root,right,left), which is exactly the modified preorder.
2. *"When must you use the one-stack version?"* → If you can't afford the O(n) reversal or must process nodes strictly in postorder order on the fly.

**Common mistakes:**
- In the one-stack version, forgetting the `lastVisited != peek.right` guard → infinite loop re-descending the right subtree.

---

### Q4: Binary Tree Level Order Traversal (BFS)
**Company:** Amazon, Microsoft, Meta, Google, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return values level by level, top to bottom, each level as its own list.

**What interviewer is testing:**
The canonical BFS-with-level-separation pattern. Many tree problems ("right side view", "zigzag", "max width", "average of levels") are this template with one line changed.

**Ideal Answer:**

Use a queue. The key trick to separate levels: at the start of each outer iteration, snapshot `queue.size()` — that's exactly the number of nodes on the current level. Process that many, enqueuing children for the next level.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int levelSize = queue.size();          // nodes on this level
        List<Integer> level = new ArrayList<>(levelSize);
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null)  queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        res.add(level);
    }
    return res;
}
```

**Dry run** — `[3,9,20,null,null,15,7]`:
- Level 0: size 1, poll 3, enqueue 9,20 → `[3]`.
- Level 1: size 2, poll 9 (leaf), poll 20 (enqueue 15,7) → `[9,20]`.
- Level 2: size 2, poll 15, poll 7 → `[15,7]`.
- Result `[[3],[9,20],[15,7]]`. ✅

**Complexity:** O(n) time, O(w) space where w is the max level width (up to n/2 for the bottom level of a full tree).

**Pattern:** BFS with level snapshot — the single most reused tree template. Memorize it.

**Follow-up questions the interviewer will ask:**
1. *"Bottom-up level order?"* → Same, but insert each level at the front of the result (or reverse at the end).
2. *"Average of each level?"* → Sum the level, divide by levelSize.
3. *"Can you do level order with DFS?"* → Yes: pass a `depth` parameter and append to `res.get(depth)`, creating the list when `depth == res.size()`.

**Common mistakes:**
- Not snapshotting `queue.size()` before the inner loop — the size changes as you enqueue children, mixing levels.
- Using `null` root without a guard.

---

### Q5: Maximum Depth of Binary Tree
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return the maximum depth (number of nodes on the longest root-to-leaf path).

**What interviewer is testing:**
The simplest post-order recursion — define `depth(node) = 1 + max(depth(left), depth(right))`. The warm-up that establishes whether you can write a clean recursive contract.

**Ideal Answer:**

```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;                     // empty subtree has depth 0
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

*Iterative (BFS levels):* count the number of levels.

```java
public int maxDepthIterative(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    int depth = 0;
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            if (n.left != null)  q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
        depth++;
    }
    return depth;
}
```

**Complexity:** O(n) time, O(h) space (recursion stack) or O(w) for BFS.

**Pattern:** Post-order combine — the parent's answer is a function of children's answers.

**Follow-ups:**
1. *"Minimum depth?"* → Q6; subtle because a node with one child is not a leaf.
2. *"Why O(h) and not O(1) space recursively?"* → The call stack holds the path to the current node.

**Common mistakes:**
- Returning the *edge* count vs the *node* count without clarifying the definition.
- Mixing up max and min.

---

### Q6: Minimum Depth of Binary Tree
**Company:** Amazon, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Return the minimum depth — the number of nodes on the shortest root-to-**leaf** path.

**What interviewer is testing:**
The edge case that trips people: a node with only one child is *not* a leaf, so you can't just take `min(left, right)` — the null child would falsely report depth 0.

**Ideal Answer:**

If a node has only one child, the minimum depth must go through that child (the missing side is not a valid leaf path). Only when both children are null is the node a leaf.

```java
public int minDepth(TreeNode root) {
    if (root == null) return 0;
    if (root.left == null) return 1 + minDepth(root.right); // only right exists
    if (root.right == null) return 1 + minDepth(root.left); // only left exists
    return 1 + Math.min(minDepth(root.left), minDepth(root.right));
}
```

*BFS is actually optimal here* — the first leaf you hit by level order is the shallowest, so you can stop early:

```java
public int minDepthBFS(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    int depth = 1;
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            if (n.left == null && n.right == null) return depth; // first leaf
            if (n.left != null)  q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
        depth++;
    }
    return depth;
}
```

**Dry run** — root 1 with only `left = 2` (2 is a leaf): root.right == null → `1 + minDepth(2)`. minDepth(2): both children null → both single-child guards skip (left==null returns 1+minDepth(null=right)=1+0=1). Total 2. A naive `min(left,right)` would have returned 1 (wrong). ✅

**Complexity:** Recursive O(n) time, O(h) space. BFS O(n) worst but stops at the shallowest leaf — often much faster on wide-shallow trees.

**Follow-ups:**
1. *"Why is BFS better than DFS here?"* → DFS may explore a deep subtree fully before finding the shallow leaf; BFS finds it immediately.
2. *"Single-child handling?"* → Explained above; this is the whole trick.

**Common mistakes:**
- Using `1 + Math.min(minDepth(left), minDepth(right))` unconditionally → returns 1 for a root with one child (wrong).

---

### Q7: Balanced Binary Tree
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
A tree is height-balanced if for *every* node, the heights of the two subtrees differ by at most 1. Determine if the tree is balanced.

**What interviewer is testing:**
Whether you spot the O(n²) trap (computing height at every node independently) and fix it with a single post-order pass that returns height *and* signals imbalance.

**Ideal Answer:**

*Naive O(n²):* for each node, compute left height and right height (each O(n)), check difference, recurse. Repeated height computations make it O(n²).

*Optimal O(n):* a single post-order traversal where each call returns the subtree height, but returns a sentinel (-1) the moment any subtree is unbalanced, short-circuiting upward.

```java
public boolean isBalanced(TreeNode root) {
    return height(root) != -1;
}
// returns height if balanced, -1 if any subtree is unbalanced
private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    if (left == -1) return -1;                  // short-circuit
    int right = height(node.right);
    if (right == -1) return -1;
    if (Math.abs(left - right) > 1) return -1;  // this node unbalanced
    return 1 + Math.max(left, right);
}
```

**Dry run** — root 1, left chain `2 -> 3` (depth 3 on left), right null:
- height(3): leaf → 1. height(2): left=1(via 3? actually 2.left=3), right=0 → diff 1 ok → 2. height(1): left=2, right=0 → diff 2 > 1 → return -1 → unbalanced. ✅

**Complexity:** O(n) time (each node visited once), O(h) space.

**Pattern:** Post-order with a sentinel return value to fuse "compute" and "validate" into one pass — a recurring FAANG trick (also used in Q40 validate BST style problems).

**Follow-up questions the interviewer will ask:**
1. *"Why is the naive version O(n²)?"* → Height is recomputed for every ancestor; worst case (skewed) gives 1+2+...+n = O(n²).
2. *"Return the actual heights too?"* → Use a small wrapper class `{height, balanced}` instead of the -1 sentinel for clarity.
3. *"AVL rotation to fix imbalance?"* → That's a self-balancing BST topic; mention you'd rotate but it's beyond this check.

**Common mistakes:**
- Writing the O(n²) version and not recognizing the redundant work.
- Returning a boolean only (losing height) so the parent can't compute its own height.

---

### Q8: Diameter of Binary Tree
**Company:** Amazon, Google, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
The diameter is the length (number of edges) of the longest path between any two nodes; the path may or may not pass through the root.

**What interviewer is testing:**
The "compute height, but update a global best at every node" pattern. The longest path through a given node = leftHeight + rightHeight. The trick is that height and diameter are computed in the *same* post-order pass.

**Ideal Answer:**

For any node, the longest path that *bends* at that node uses its left subtree's height plus its right subtree's height (in edges). Compute heights recursively and, at each node, update a global `best = max(best, leftH + rightH)`.

```java
private int best = 0;

public int diameterOfBinaryTree(TreeNode root) {
    best = 0;
    height(root);
    return best;
}
private int height(TreeNode node) {
    if (node == null) return 0;        // height in edges; null contributes 0
    int left = height(node.left);
    int right = height(node.right);
    best = Math.max(best, left + right); // path bending here, in edges
    return 1 + Math.max(left, right);    // height to parent
}
```

**Dry run** — tree `[1,2,3,4,5]` (1 has children 2,3; 2 has children 4,5):
- height(4)=1, height(5)=1 → at node 2: best=max(0,1+1)=2, return 2.
- height(3)=1 → at node 1: left=2,right=1, best=max(2,2+1)=3, return 3.
- Diameter = 3 (path 4-2-5 is 2 edges, path 4-2-1-3 is 3 edges). ✅

**Complexity:** O(n) time, O(h) space.

**Pattern:** "Height computation + global update" — identical skeleton to Binary Tree Maximum Path Sum (Q17). If you internalize this, both are free.

**Follow-up questions the interviewer will ask:**
1. *"Return the path itself, not just length?"* → Track the node where best updated and reconstruct, or carry path info up (more bookkeeping).
2. *"Diameter in node count vs edge count?"* → Edges = nodes − 1; clarify the definition with the interviewer. (LeetCode uses edges.)
3. *"How does this relate to Max Path Sum?"* → Same shape; Max Path Sum clamps negative contributions to 0 and sums values instead of heights.

**Common mistakes:**
- Returning the diameter from the recursion instead of the height (you need height to propagate; diameter is a side effect via the global).
- Counting nodes when the problem wants edges (off-by-one).

---

### Q9: Same Tree
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Given two binary trees, check whether they are structurally identical with the same node values.

**What interviewer is testing:**
Clean parallel recursion over two trees and correct null handling.

**Ideal Answer:**

Two trees are the same iff: both null (base case true), or both non-null with equal values and matching left and right subtrees.

```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;     // both empty
    if (p == null || q == null) return false;    // exactly one empty
    if (p.val != q.val) return false;            // value mismatch
    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
```

**Complexity:** O(min(m, n)) time (stops at first mismatch), O(min height) space.

**Pattern:** Parallel two-tree recursion — reused in Symmetric Tree (Q10) and Subtree (Q11).

**Follow-ups:**
1. *"Symmetric tree?"* → Q10; compare left subtree with the *mirror* of the right.
2. *"Iterative?"* → Use a queue/stack of node pairs.

**Common mistakes:**
- Checking `p.val != q.val` before the null checks → NullPointerException.

---

### Q10: Symmetric Tree
**Company:** Amazon, Microsoft, LinkedIn
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Check whether a binary tree is a mirror of itself (symmetric around its center).

**What interviewer is testing:**
Recognizing this as "Same Tree but mirrored" — compare `left.left` with `right.right` and `left.right` with `right.left`.

**Ideal Answer:**

```java
public boolean isSymmetric(TreeNode root) {
    if (root == null) return true;
    return isMirror(root.left, root.right);
}
private boolean isMirror(TreeNode a, TreeNode b) {
    if (a == null && b == null) return true;
    if (a == null || b == null) return false;
    return a.val == b.val
        && isMirror(a.left, b.right)   // outer pair
        && isMirror(a.right, b.left);  // inner pair
}
```

*Iterative* — queue of pairs, enqueue children in mirrored order:

```java
public boolean isSymmetricIterative(TreeNode root) {
    if (root == null) return true;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root.left); q.offer(root.right);
    while (!q.isEmpty()) {
        TreeNode a = q.poll(), b = q.poll();
        if (a == null && b == null) continue;
        if (a == null || b == null || a.val != b.val) return false;
        q.offer(a.left);  q.offer(b.right);  // mirrored
        q.offer(a.right); q.offer(b.left);
    }
    return true;
}
```

**Dry run** — `[1,2,2,3,4,4,3]`: isMirror(2,2): vals equal; isMirror(3,3) ok; isMirror(4,4) ok → true. ✅

**Complexity:** O(n) time, O(h) space.

**Follow-ups:**
1. *"What's the difference from Same Tree?"* → The cross-comparison `(a.left,b.right)` and `(a.right,b.left)`.

**Common mistakes:**
- Comparing `a.left` with `b.left` (that checks identical, not mirror).

---

### Q11: Subtree of Another Tree
**Company:** Amazon, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given `root` and `subRoot`, return true if `subRoot` is a subtree of `root` (a node in root such that the subtree rooted there equals subRoot).

**What interviewer is testing:**
Composition — reuse `isSameTree` while traversing every node of root. Plus an awareness of the optimal O(n+m) string/hashing approach.

**Ideal Answer:**

*Straightforward O(n·m):* at each node of root, check if the subtree there equals subRoot.

```java
public boolean isSubtree(TreeNode root, TreeNode subRoot) {
    if (root == null) return subRoot == null;
    if (isSameTree(root, subRoot)) return true;
    return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
}
private boolean isSameTree(TreeNode a, TreeNode b) {
    if (a == null && b == null) return true;
    if (a == null || b == null) return false;
    return a.val == b.val && isSameTree(a.left, b.left) && isSameTree(a.right, b.right);
}
```

*Optimal O(n+m):* serialize both trees with null markers and a delimiter, then check substring (with KMP for true linear). Serialize using preorder with `#` for null and a leading separator so values can't accidentally concatenate (`12` vs `1,2`).

```java
public boolean isSubtreeOptimal(TreeNode root, TreeNode subRoot) {
    String s = serialize(root), t = serialize(subRoot);
    return s.contains(t);  // use KMP for guaranteed O(n+m)
}
private String serialize(TreeNode node) {
    StringBuilder sb = new StringBuilder();
    dfs(node, sb);
    return sb.toString();
}
private void dfs(TreeNode node, StringBuilder sb) {
    if (node == null) { sb.append(",#"); return; }
    sb.append(",^").append(node.val);   // ^ guards against value-prefix collisions
    dfs(node.left, sb);
    dfs(node.right, sb);
}
```

**Why the `^` and `,`:** Without delimiters, tree with value 12 and tree with values 1,2 could serialize identically. The separators make the serialization injective.

**Complexity:** Brute O(n·m); serialized O(n+m) time and space (with KMP for the substring search).

**Follow-ups:**
1. *"Why might `String.contains` not be truly O(n+m)?"* → Java's `indexOf` is naive O(n·m) worst case; use KMP for the guarantee.
2. *"Subtree vs subgraph?"* → Subtree must include *all* descendants; a partial match doesn't count — that's why `isSameTree` (not "subRoot is contained") is the right check.

**Common mistakes:**
- Treating it as "does subRoot appear anywhere" without requiring the full subtree to match exactly.
- Serialization without delimiters → false positives.

---

### Q12: Invert Binary Tree
**Company:** Google, Amazon, Microsoft (the famous "homebrew" question)
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** OA / Phone

**Question:**
Invert a binary tree — swap every node's left and right children.

**What interviewer is testing:**
Trivial recursion, but a classic warm-up. Show you can also do it iteratively.

**Ideal Answer:**

```java
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode left = invertTree(root.left);   // recurse first
    TreeNode right = invertTree(root.right);
    root.left = right;                        // then swap
    root.right = left;
    return root;
}
```

*Iterative (BFS):* swap children of every dequeued node.

```java
public TreeNode invertTreeBFS(TreeNode root) {
    if (root == null) return null;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        TreeNode n = q.poll();
        TreeNode tmp = n.left; n.left = n.right; n.right = tmp; // swap
        if (n.left != null)  q.offer(n.left);
        if (n.right != null) q.offer(n.right);
    }
    return root;
}
```

**Complexity:** O(n) time, O(h) space.

**Follow-ups:**
1. *"Does it matter if you swap before or after recursing?"* → No, as long as you recurse on the original children before reassigning (or swap first then recurse on the new positions — both visit every node once).

**Common mistakes:**
- Swapping first then recursing on stale references inconsistently (works if careful, but state it clearly).

---

### Q13: Lowest Common Ancestor of a Binary Tree
**Company:** Amazon, Facebook, Microsoft, Google, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a binary tree (not a BST) and two nodes `p` and `q`, find their lowest common ancestor — the deepest node that has both as descendants (a node can be a descendant of itself).

**What interviewer is testing:**
The elegant post-order recursion that "bubbles up" found nodes. This is a top-20 FAANG tree question.

**Ideal Answer:**

Recurse. The contract: `lca(node)` returns a non-null node if `p` or `q` (or their LCA) is found in `node`'s subtree.
- If `node` is null or equals `p` or `q`, return `node`.
- Recurse left and right.
- If *both* sides return non-null, the current node is the split point → it's the LCA.
- Otherwise propagate whichever side is non-null.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root; // p and q split here
    return left != null ? left : right;             // both on one side (or none)
}
```

**Dry run** — tree `[3,5,1,6,2,0,8,null,null,7,4]`, p=5, q=1:
- lca(3): left=lca(5)→returns 5 (matches p), right=lca(1)→returns 1 (matches q). Both non-null → return 3. ✅

p=5, q=4 (4 is descendant of 5):
- lca(5) returns 5 immediately (matches p). Inside lca(3), left=5, right=null → returns 5. Correct: 5 is the ancestor of 4. ✅

**Complexity:** O(n) time, O(h) space.

**Pattern:** Post-order "found-signal bubbling" — when both children report a find, the current node is the answer.

**Follow-up questions the interviewer will ask:**
1. *"What if p or q might NOT exist in the tree?"* → The basic version assumes both exist. To handle absence, track booleans `foundP`, `foundQ` and only return the candidate if both were actually seen.
2. *"Nodes have parent pointers?"* → Walk both up to collect ancestor sets, or use the two-pointer "intersection of linked lists" trick: advance each pointer, switch to the other's start on reaching root — they meet at the LCA.
3. *"Many repeated queries?"* → Preprocess with binary lifting (sparse table of 2^k-th ancestors) for O(log n) per query, or Euler tour + sparse table RMQ for O(1) per query.

**Common mistakes:**
- Returning a boolean instead of the node (you lose the ability to identify the split point).
- Assuming it's a BST and using value comparisons (that's Q46, the easier BST variant).
- Not handling `root == p || root == q` early (a node is its own ancestor).

---

### Q14: Path Sum I
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given a tree and a `targetSum`, return true if there is a root-to-leaf path whose node values sum to `targetSum`.

**What interviewer is testing:**
Top-down recursion carrying a running remainder, plus the correct definition of "leaf."

**Ideal Answer:**

Subtract the current value from the target as you descend. At a leaf, succeed iff the remaining target equals the leaf's value.

```java
public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) return false;                       // empty path
    if (root.left == null && root.right == null)          // leaf
        return targetSum == root.val;
    int remain = targetSum - root.val;
    return hasPathSum(root.left, remain) || hasPathSum(root.right, remain);
}
```

**Dry run** — `[5,4,8,11,null,13,4,7,2]`, target 22: 5→remain17→4→remain13→11→remain2→leaf 7? 7≠2; leaf 2? 2==2 → true (5+4+11+2=22). ✅

**Complexity:** O(n) time, O(h) space.

**Pattern:** Top-down recursion passing accumulated state downward (as opposed to post-order bubbling up).

**Follow-ups:**
1. *"Return all such paths?"* → Q15, backtracking.
2. *"Count paths starting anywhere?"* → Q16, prefix sum.

**Common mistakes:**
- Treating a node with one child as a leaf (the `null` child would falsely match a remaining 0).
- Checking `targetSum == 0` at null instead of at leaves (fails for negative values and single nodes).

---

### Q15: Path Sum II
**Company:** Amazon, Facebook, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Return all root-to-leaf paths whose values sum to `targetSum`.

**What interviewer is testing:**
DFS backtracking — add to the path, recurse, then *remove* on the way out. The remove is the part candidates forget.

**Ideal Answer:**

```java
public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
    List<List<Integer>> res = new ArrayList<>();
    dfs(root, targetSum, new ArrayList<>(), res);
    return res;
}
private void dfs(TreeNode node, int remain, List<Integer> path, List<List<Integer>> res) {
    if (node == null) return;
    path.add(node.val);                       // choose
    if (node.left == null && node.right == null && remain == node.val) {
        res.add(new ArrayList<>(path));       // copy! path is mutated later
    } else {
        dfs(node.left,  remain - node.val, path, res);
        dfs(node.right, remain - node.val, path, res);
    }
    path.remove(path.size() - 1);             // un-choose (backtrack)
}
```

**Complexity:** O(n) time to traverse; O(n·h) worst for copying paths; O(h) recursion + path.

**Pattern:** Backtracking — the choose / recurse / un-choose triad.

**Follow-ups:**
1. *"Why copy the path into the result?"* → `path` is shared and mutated; storing the reference would give all-empty lists at the end.
2. *"Paths to any node, not just leaves?"* → Drop the leaf check and record at every node.

**Common mistakes:**
- Forgetting `path.remove(...)` → paths leak across branches.
- Adding `path` directly to `res` instead of a copy.

---

### Q16: Path Sum III
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Count the number of paths that sum to `targetSum`. The path does **not** need to start at the root or end at a leaf, but must go downward (parent to child).

**What interviewer is testing:**
Applying **prefix sum + hashmap** (the array pattern) to a tree. Recognizing that "count downward paths with sum k" is exactly "count subarrays with sum k" along each root-to-node chain.

**Ideal Answer:**

*Brute force O(n²):* from every node, DFS downward counting paths summing to target.

*Optimal O(n):* maintain a running prefix sum from root to the current node, and a hashmap `prefixSum -> count`. A downward path ending at the current node with sum k exists for each earlier ancestor whose prefix sum equals `curSum - k`. Crucially, **backtrack the map** when leaving a node (decrement) so sibling subtrees don't see each other's prefixes.

```java
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0L, 1);            // empty prefix -> enables paths from root
    return dfs(root, 0L, targetSum, prefixCount);
}
private int dfs(TreeNode node, long curSum, int target, Map<Long, Integer> prefixCount) {
    if (node == null) return 0;
    curSum += node.val;
    // number of ancestors a with prefix curSum - target => path (a, node] sums to target
    int count = prefixCount.getOrDefault(curSum - target, 0);
    prefixCount.merge(curSum, 1, Integer::sum);          // add current prefix
    count += dfs(node.left,  curSum, target, prefixCount);
    count += dfs(node.right, curSum, target, prefixCount);
    prefixCount.merge(curSum, -1, Integer::sum);          // BACKTRACK
    return count;
}
```

**Dry run** — path `10 -> 5 -> 3`, target 8: at 10 curSum=10, look for 2 → 0; map{0:1,10:1}. At 5 curSum=15, look for 7 → 0; map adds 15. At 3 curSum=18, look for 10 → found 1 → count 1 (path 5+3=8). ✅

**Why `long`:** prefix sums can overflow `int` with many large values.

**Complexity:** O(n) time, O(n) space (the map / recursion).

**Pattern:** Prefix sum + hashmap on a tree path, with map backtracking — a beautiful cross-over of array and tree techniques.

**Follow-up questions the interviewer will ask:**
1. *"Why decrement the map on the way out?"* → The map must only reflect prefixes on the *current* root-to-node path. Without backtracking, a left-subtree prefix would incorrectly count for a right-subtree node.
2. *"Why seed `{0:1}`?"* → To count paths that start exactly at the root (where `curSum - target == 0`).
3. *"What if paths could go up then down (any path)?"* → That's the harder Max Path Sum shape (Q17) for sums, or a different formulation entirely.

**Common mistakes:**
- Forgetting to backtrack the map (overcounts).
- Using `int` for curSum (overflow).
- Forgetting the `{0:1}` seed (misses root-starting paths).

---

### Q17: Binary Tree Maximum Path Sum
**Company:** Amazon, Facebook, Google, Microsoft, DE Shaw
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
A path is any sequence of nodes connected by edges; it need not pass through the root and need not be root-to-leaf. The path sum is the sum of node values. Return the maximum path sum of any path in the tree. Values can be negative.

**What interviewer is testing:**
The single most important tree recursion pattern: a helper that returns the best *downward* contribution from a node, while a global tracks the best path that *bends* at each node. This is the harder twin of Diameter (Q8). Mastering this question signals you truly understand tree DP.

**Ideal Answer:**

The key distinction: there are two different quantities at each node.

1. **Gain (what we return up):** the maximum sum of a path that starts at this node and goes *straight down* one branch. A parent can extend at most one child branch (a path can't fork upward). So `gain(node) = node.val + max(0, max(gainLeft, gainRight))` — we clamp negative child gains to 0 (don't include a branch that hurts us).

2. **Best path through this node (the global answer candidate):** a path that bends here uses *both* children: `node.val + max(0, gainLeft) + max(0, gainRight)`. Update the global maximum with this.

```java
private int maxSum;

public int maxPathSum(TreeNode root) {
    maxSum = Integer.MIN_VALUE;     // values can be negative
    gain(root);
    return maxSum;
}
// returns the max gain if we continue the path downward through `node`
private int gain(TreeNode node) {
    if (node == null) return 0;
    int leftGain  = Math.max(gain(node.left), 0);   // clamp negatives to 0
    int rightGain = Math.max(gain(node.right), 0);
    // path that bends at this node (uses both sides) -> candidate for the answer
    int bend = node.val + leftGain + rightGain;
    maxSum = Math.max(maxSum, bend);
    // but to a parent we can only offer one branch
    return node.val + Math.max(leftGain, rightGain);
}
```

**Dry run** — tree `[-10, 9, 20, null, null, 15, 7]`:
- gain(9)=9, candidate bend 9 → maxSum 9.
- gain(15)=15 → maxSum 15. gain(7)=7.
- node 20: leftGain=15, rightGain=7, bend=20+15+7=42 → maxSum 42. returns 20+max(15,7)=35.
- node -10: leftGain=max(9,0)=9, rightGain=max(35,0)=35, bend=-10+9+35=34 → maxSum stays 42. returns -10+35=25.
- Answer **42** (path 15-20-7). ✅

**Dry run (all-negative)** — `[-3]`: gain(-3): leftGain=rightGain=0, bend=-3 → maxSum=-3, return -3. Answer -3. The `Integer.MIN_VALUE` init and clamping both matter here. ✅

**Complexity:** O(n) time (each node once), O(h) space (recursion).

**Pattern:** "Return single-branch gain, update global with the two-branch bend." Identical structure to Diameter — internalize one and you have both.

**Follow-up questions the interviewer will ask:**
1. *"Why clamp child gains to 0?"* → A negative branch can only reduce the path; excluding it (treating as 0) is always at least as good. But we never clamp `node.val` itself — a single negative node may be the entire best path.
2. *"Why init maxSum to MIN_VALUE not 0?"* → If all nodes are negative, the answer is the single largest (least negative) node; starting at 0 would wrongly return 0.
3. *"Reconstruct the actual path?"* → Record the node where `bend` updated the global and which branches contributed; reconstruct by re-descending. More bookkeeping; mention it.
4. *"What if the path must be root-to-leaf?"* → Different, simpler problem — just a downward DFS tracking the max leaf path.

**Common mistakes that get you rejected:**
- Returning `bend` (the two-branch sum) up to the parent — a path can't fork, so the parent can only use one branch. This is the #1 mistake.
- Initializing maxSum to 0 (fails all-negative trees).
- Forgetting to clamp negative gains, dragging down otherwise-good paths.

---

### Q18: Construct Binary Tree from Preorder and Inorder Traversal
**Company:** Amazon, Microsoft, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given `preorder` and `inorder` traversal arrays of a tree with unique values, reconstruct and return the tree.

**What interviewer is testing:**
Understanding the *structure* of traversals: preorder's first element is always the root; inorder splits into left subtree (before root) and right subtree (after root). Plus the O(n) optimization with a value→index map.

**Ideal Answer:**

- `preorder[0]` is the root.
- Find the root in `inorder` at index `i`. Everything left of `i` in inorder is the left subtree (that's `i` nodes); everything right is the right subtree.
- The next `i` elements of preorder (after the root) form the left subtree's preorder; the rest form the right subtree's preorder.
- Recurse.

A hashmap `value -> inorder index` makes the "find root in inorder" step O(1) instead of O(n), giving O(n) total. Use a moving `preIndex` pointer.

```java
private int preIndex;
private Map<Integer, Integer> inIndex;

public TreeNode buildTree(int[] preorder, int[] inorder) {
    preIndex = 0;
    inIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inIndex.put(inorder[i], i);
    return build(preorder, 0, inorder.length - 1);
}
// builds the subtree whose inorder range is [inLeft, inRight]
private TreeNode build(int[] preorder, int inLeft, int inRight) {
    if (inLeft > inRight) return null;
    int rootVal = preorder[preIndex++];        // next preorder element = root
    TreeNode root = new TreeNode(rootVal);
    int mid = inIndex.get(rootVal);            // split point in inorder
    root.left  = build(preorder, inLeft, mid - 1);   // build left FIRST (preorder order)
    root.right = build(preorder, mid + 1, inRight);
    return root;
}
```

**Dry run** — preorder `[3,9,20,15,7]`, inorder `[9,3,15,20,7]`:
- preIndex 0 → root 3, mid (index of 3 in inorder)=1. Left range [0,0], right [2,4].
- Left: preIndex 1 → root 9, mid=0, ranges empty → leaf 9.
- Right: preIndex 2 → root 20, mid=3, left [2,2], right [4,4].
  - Left: preIndex 3 → 15 (leaf). Right: preIndex 4 → 7 (leaf).
- Tree: 3(9, 20(15,7)). ✅

**Complexity:** O(n) time (each node built once, O(1) lookups), O(n) space (map + recursion).

**Pattern:** Divide-and-conquer on traversal arrays. **Build left before right** because preorder consumes the left subtree entirely before the right.

**Follow-up questions the interviewer will ask:**
1. *"Why must you build the left subtree before the right?"* → Preorder lists the entire left subtree before the right; the moving `preIndex` must consume them in that order.
2. *"Postorder + inorder?"* → Q19; consume postorder from the *end* and build *right before left*.
3. *"Preorder + postorder alone?"* → Ambiguous in general (can't distinguish a single child as left vs right) unless the tree is full.
4. *"Duplicate values?"* → The value→index map breaks; you'd need positional disambiguation, generally requiring extra constraints.

**Common mistakes:**
- Re-scanning inorder for the root each time → O(n²).
- Building right subtree before left (corrupts the `preIndex` consumption order).
- Passing preorder bounds *and* inorder bounds and getting them out of sync — using a single global `preIndex` is cleaner.

---

### Q19: Construct Binary Tree from Postorder and Inorder Traversal
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Reconstruct the tree from `postorder` and `inorder` arrays (unique values).

**What interviewer is testing:**
The mirror of Q18. Postorder's *last* element is the root, and you consume postorder from the end, building the **right subtree before the left**.

**Ideal Answer:**

```java
private int postIndex;
private Map<Integer, Integer> inIndex;

public TreeNode buildTree(int[] inorder, int[] postorder) {
    postIndex = postorder.length - 1;          // root is the LAST postorder element
    inIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inIndex.put(inorder[i], i);
    return build(postorder, 0, inorder.length - 1);
}
private TreeNode build(int[] postorder, int inLeft, int inRight) {
    if (inLeft > inRight) return null;
    int rootVal = postorder[postIndex--];      // consume from the end
    TreeNode root = new TreeNode(rootVal);
    int mid = inIndex.get(rootVal);
    root.right = build(postorder, mid + 1, inRight);  // RIGHT first
    root.left  = build(postorder, inLeft, mid - 1);
    return root;
}
```

**Why right first:** postorder is (left, right, root). Reading it backward gives (root, right, left), so after taking the root we encounter the right subtree's nodes next.

**Complexity:** O(n) time, O(n) space.

**Follow-up:** *"Compare to Q18."* → Symmetric: preorder reads front-to-back building left-first; postorder reads back-to-front building right-first.

**Common mistakes:**
- Building left before right (wrong consumption order for backward postorder).
- Forgetting to start `postIndex` at the last element.

---

### Q20: Serialize and Deserialize Binary Tree
**Company:** Amazon, Facebook, Google, LinkedIn, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Design `serialize(TreeNode) -> String` and `deserialize(String) -> TreeNode` so that `deserialize(serialize(root))` reproduces the tree. Values may be any integer; the tree may be arbitrary (not a BST).

**What interviewer is testing:**
Encoding tree *structure* (including nulls) unambiguously, then parsing it back. This is a top FAANG design-flavored tree question. The clean approach is preorder with explicit null markers.

**Ideal Answer:**

**Why preorder + null markers works:** Preorder alone is ambiguous (you can't tell where a subtree ends). But if you write a marker for every null child, the structure becomes fully determined: when deserializing, you read the root, then *recursively* read its left subtree, then its right — the null markers tell you exactly where to stop.

```java
public class Codec {
    private static final String NULL = "#";
    private static final String SEP  = ",";

    // ---- serialize: preorder with null markers ----
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        buildString(root, sb);
        return sb.toString();
    }
    private void buildString(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append(NULL).append(SEP);
            return;
        }
        sb.append(node.val).append(SEP);   // root
        buildString(node.left, sb);        // left
        buildString(node.right, sb);       // right
    }

    // ---- deserialize: consume the preorder stream ----
    public TreeNode deserialize(String data) {
        Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(SEP)));
        return buildTree(tokens);
    }
    private TreeNode buildTree(Queue<String> tokens) {
        String val = tokens.poll();
        if (NULL.equals(val)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left  = buildTree(tokens);    // matches serialize order
        node.right = buildTree(tokens);
        return node;
    }
}
```

**Dry run** — tree `[1,2,3,null,null,4,5]`:
- serialize: "1,2,#,#,3,4,#,#,5,#,#,"
- deserialize: poll 1 → node(1); left = buildTree → poll 2 → node(2); its left poll # → null; its right poll # → null; back to 1's right → poll 3 → node(3); left poll 4 → node(4) with two nulls; right poll 5 → node(5) with two nulls. Reconstructed exactly. ✅

**Complexity:** O(n) time both ways, O(n) space (string + recursion/queue).

*BFS (level-order) alternative:* serialize level by level with null markers; deserialize using a queue, attaching children as you read them. This matches LeetCode's display format. Be ready to write it:

```java
public String serializeBFS(TreeNode root) {
    if (root == null) return "";
    StringBuilder sb = new StringBuilder();
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        TreeNode n = q.poll();
        if (n == null) { sb.append("#,"); continue; }
        sb.append(n.val).append(",");
        q.offer(n.left);
        q.offer(n.right);
    }
    return sb.toString();
}
public TreeNode deserializeBFS(String data) {
    if (data.isEmpty()) return null;
    String[] vals = data.split(",");
    TreeNode root = new TreeNode(Integer.parseInt(vals[0]));
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    int i = 1;
    while (!q.isEmpty()) {
        TreeNode parent = q.poll();
        if (!"#".equals(vals[i])) {
            parent.left = new TreeNode(Integer.parseInt(vals[i]));
            q.offer(parent.left);
        }
        i++;
        if (!"#".equals(vals[i])) {
            parent.right = new TreeNode(Integer.parseInt(vals[i]));
            q.offer(parent.right);
        }
        i++;
    }
    return root;
}
```

**Follow-up questions the interviewer will ask:**
1. *"Why do you need null markers — isn't preorder enough?"* → No. `[1,2]` could be `1` with left child `2` or right child `2`. Null markers disambiguate structure.
2. *"How would you make the string smaller?"* → For a BST you can drop null markers and use the BST property (bounds) to rebuild — fewer tokens. Or use a compact binary encoding.
3. *"Handle negative values / large values?"* → The comma delimiter already handles multi-digit and negative numbers; just don't use a digit as the delimiter.
4. *"Serialize an N-ary tree?"* → Add a child-count per node, or use null markers per child list.

**Common mistakes:**
- No null markers → ambiguous, can't reconstruct.
- Using a delimiter that can appear in a value (e.g., none, so "12" vs "1","2" collide).
- Deserializing left/right in the wrong order relative to serialize.

---

### Q21: Binary Tree Right Side View
**Company:** Amazon, Facebook, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Imagine standing on the right side of the tree; return the values of the nodes you can see, top to bottom (the rightmost node at each level).

**What interviewer is testing:**
The level-order template with a twist — capture the last node of each level. Or a DFS visiting right-first and recording the first node seen at each depth.

**Ideal Answer:**

*BFS:* the rightmost node of each level is the last one dequeued in that level's loop.

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            if (i == size - 1) res.add(n.val);  // last node on this level
            if (n.left != null)  q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
    }
    return res;
}
```

*DFS (right first):* the first node reached at each new depth is the rightmost.

```java
public List<Integer> rightSideViewDFS(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    dfs(root, 0, res);
    return res;
}
private void dfs(TreeNode node, int depth, List<Integer> res) {
    if (node == null) return;
    if (depth == res.size()) res.add(node.val);  // first node at this depth
    dfs(node.right, depth + 1, res);             // RIGHT before left
    dfs(node.left,  depth + 1, res);
}
```

**Complexity:** O(n) time, O(h) or O(w) space.

**Follow-ups:**
1. *"Left side view?"* → BFS capture `i == 0`, or DFS left-first.
2. *"Why does DFS right-first work?"* → The first time we reach a depth, we came down the rightmost path.

**Common mistakes:**
- BFS: capturing `i == 0` (that's the left view).
- DFS: recursing left before right (records the leftmost).

---

### Q22: Flatten Binary Tree to Linked List
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Flatten the tree into a "linked list" in *preorder*: each node's `right` points to the next preorder node, and `left` is null. Do it in place.

**What interviewer is testing:**
In-place pointer surgery. The elegant O(1)-space solution uses a Morris-like "find the rightmost node of the left subtree" trick.

**Ideal Answer:**

*O(1) space (Morris-style):* For each node with a left child, find the rightmost node of the left subtree (the in-order predecessor of the right subtree), splice the right subtree onto it, move the left subtree to the right, and null out left.

```java
public void flatten(TreeNode root) {
    TreeNode cur = root;
    while (cur != null) {
        if (cur.left != null) {
            TreeNode prev = cur.left;            // rightmost of left subtree
            while (prev.right != null) prev = prev.right;
            prev.right = cur.right;              // attach original right subtree
            cur.right = cur.left;                // move left subtree to the right
            cur.left = null;                     // flatten requires null left
        }
        cur = cur.right;                         // move on
    }
}
```

*Recursive (reverse preorder):* visit right, then left, prepending to a `prev` pointer.

```java
private TreeNode prev = null;
public void flattenRecursive(TreeNode root) {
    if (root == null) return;
    flattenRecursive(root.right);   // process right subtree first
    flattenRecursive(root.left);
    root.right = prev;              // current node's right = previously processed
    root.left = null;
    prev = root;                    // this node is now the head of processed chain
}
```

**Dry run (O(1))** — tree `[1,2,5,3,4,null,6]`:
- cur=1 has left 2. Rightmost of left subtree (2→3,4) is 4. prev.right(4)=cur.right(5). cur.right=2, cur.left=null. → 1→2→...→ then 5,6 attached after 4. cur moves to 2.
- Continue down; result preorder chain 1,2,3,4,5,6 all on right pointers. ✅

**Complexity:** O(n) time, O(1) space (Morris-style); recursive is O(n)/O(h).

**Follow-ups:**
1. *"Why O(1) space matters here?"* → The interviewer often explicitly asks for it after you give the recursive version.
2. *"Flatten in inorder instead?"* → Adapt the threading order.

**Common mistakes:**
- Losing the right subtree before re-attaching it.
- Forgetting to null out `left`.

---

### Q23: Vertical Order Traversal of a Binary Tree
**Company:** Amazon, Facebook, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Group nodes by vertical column (root at column 0; left child is col−1, right is col+1). Return columns left to right; within a column, top to bottom by row; **ties at the same (row, col) are ordered by value ascending.**

**What interviewer is testing:**
Coordinate assignment + a careful multi-key sort. The value-tie rule is the subtle part that distinguishes the harder version from a simple column grouping.

**Ideal Answer:**

Do a traversal assigning each node a `(row, col)`. Collect nodes, then sort by `(col asc, row asc, val asc)` and group by column.

```java
public List<List<Integer>> verticalTraversal(TreeNode root) {
    // each entry: {col, row, val}
    List<int[]> nodes = new ArrayList<>();
    dfs(root, 0, 0, nodes);
    // sort by col, then row, then value
    nodes.sort((a, b) -> {
        if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
        if (a[1] != b[1]) return Integer.compare(a[1], b[1]);
        return Integer.compare(a[2], b[2]);
    });
    List<List<Integer>> res = new ArrayList<>();
    Integer prevCol = null;
    for (int[] n : nodes) {
        if (prevCol == null || n[0] != prevCol) {   // new column
            res.add(new ArrayList<>());
            prevCol = n[0];
        }
        res.get(res.size() - 1).add(n[2]);
    }
    return res;
}
private void dfs(TreeNode node, int row, int col, List<int[]> nodes) {
    if (node == null) return;
    nodes.add(new int[]{col, row, node.val});
    dfs(node.left,  row + 1, col - 1, nodes);
    dfs(node.right, row + 1, col + 1, nodes);
}
```

**Complexity:** O(n log n) time (the sort), O(n) space.

**Follow-up questions the interviewer will ask:**
1. *"Earlier LeetCode version broke ties by insertion order, not value — what changes?"* → Use BFS so same-cell nodes appear in level order, and drop the value tiebreak. The current version sorts by value for ties.
2. *"Avoid sorting?"* → Use a TreeMap<col, TreeMap<row, PriorityQueue<val>>> for an O(n log n) structured build without a final global sort.

**Common mistakes:**
- Forgetting the value tiebreak for nodes sharing the same (row, col).
- Using a HashMap for columns and failing to output them in sorted column order.

---

### Q24: Boundary Traversal of a Binary Tree
**Company:** Amazon, Microsoft, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return the boundary of the tree in anti-clockwise order: root, then the left boundary (top-down, excluding leaves), then all leaves (left to right), then the right boundary (bottom-up, excluding leaves).

**What interviewer is testing:**
Careful case decomposition and avoiding double-counting (leaves on the boundary must appear exactly once).

**Ideal Answer:**

Three parts, each excluding leaves so leaves are counted only in the leaves pass:
1. Left boundary: from root's left child down, always preferring left (or right if no left), excluding leaves.
2. Leaves: a DFS adding only leaf nodes, left to right.
3. Right boundary: from root's right child down preferring right, excluding leaves — collected then reversed (bottom-up).

```java
public List<Integer> boundaryOfBinaryTree(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;
    if (!isLeaf(root)) res.add(root.val);

    // left boundary (exclude leaves)
    TreeNode cur = root.left;
    while (cur != null) {
        if (!isLeaf(cur)) res.add(cur.val);
        cur = (cur.left != null) ? cur.left : cur.right;
    }
    // leaves, left to right
    addLeaves(root, res);
    // right boundary (exclude leaves), collected bottom-up
    List<Integer> rightSide = new ArrayList<>();
    cur = root.right;
    while (cur != null) {
        if (!isLeaf(cur)) rightSide.add(cur.val);
        cur = (cur.right != null) ? cur.right : cur.left;
    }
    Collections.reverse(rightSide);
    res.addAll(rightSide);
    return res;
}
private boolean isLeaf(TreeNode n) { return n.left == null && n.right == null; }
private void addLeaves(TreeNode node, List<Integer> res) {
    if (node == null) return;
    if (isLeaf(node)) { res.add(node.val); return; }
    addLeaves(node.left, res);
    addLeaves(node.right, res);
}
```

**Complexity:** O(n) time, O(h) space.

**Follow-ups:**
1. *"Why exclude leaves from the left/right boundary loops?"* → Otherwise a corner leaf gets added twice (once in the boundary, once in the leaves pass).
2. *"Single node tree?"* → Root is a leaf; the `!isLeaf` guard skips adding it as boundary, and `addLeaves` adds it once.

**Common mistakes:**
- Double-counting leaf nodes at the corners.
- Forgetting to reverse the right boundary.

---

### Q25: Binary Tree Zigzag Level Order Traversal
**Company:** Amazon, Microsoft, Facebook, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Level order, but alternate direction each level: left-to-right, then right-to-left, and so on.

**What interviewer is testing:**
The BFS template plus a flip flag; cleanly reversing alternate levels (ideally via deque insertion, not a post-hoc reverse).

**Ideal Answer:**

Standard BFS; use a `LinkedList` per level and `addFirst` when going right-to-left so you avoid an O(n) reverse.

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    boolean leftToRight = true;
    while (!q.isEmpty()) {
        int size = q.size();
        LinkedList<Integer> level = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            if (leftToRight) level.addLast(n.val);
            else             level.addFirst(n.val);   // build reversed in place
            if (n.left != null)  q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
        res.add(level);
        leftToRight = !leftToRight;
    }
    return res;
}
```

**Dry run** — `[3,9,20,null,null,15,7]`: L0 `[3]`; L1 right-to-left `[20,9]`; L2 left-to-right `[15,7]`. ✅

**Complexity:** O(n) time, O(w) space.

**Follow-up:** *"Why addFirst instead of reversing?"* → addFirst keeps each level O(level size) without a separate reverse pass; same big-O but cleaner.

**Common mistakes:**
- Forgetting to flip the flag.
- Enqueuing children in reversed order (you must always enqueue left then right; only the *output* order alternates).

---

### Q26: Count Complete Tree Nodes
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Count the nodes in a **complete** binary tree (every level full except possibly the last, which is filled left to right). Do better than O(n).

**What interviewer is testing:**
Exploiting the completeness property for an O(log² n) solution instead of the naive O(n) count.

**Ideal Answer:**

For a complete tree, walk the leftmost path (height `hL`) and the rightmost path (height `hR`). If they're equal, the tree is *perfect* and has `2^h − 1` nodes — O(1) given the heights. Otherwise recurse on both children; only one branch will be "imperfect," so the recursion is shallow.

```java
public int countNodes(TreeNode root) {
    if (root == null) return 0;
    int hL = leftHeight(root);
    int hR = rightHeight(root);
    if (hL == hR) return (1 << hL) - 1;        // perfect subtree: 2^h - 1
    return 1 + countNodes(root.left) + countNodes(root.right);
}
private int leftHeight(TreeNode node) {
    int h = 0;
    while (node != null) { h++; node = node.left; }
    return h;
}
private int rightHeight(TreeNode node) {
    int h = 0;
    while (node != null) { h++; node = node.right; }
    return h;
}
```

**Why O(log² n):** The recursion descends one level at a time (O(log n) depth), and each level computes two heights (O(log n)). At each level only one child is non-perfect, so it's O(log n) recursive calls × O(log n) height work = O(log² n).

**Dry run** — perfect tree of height 3: hL=hR=3 → 2³−1 = 7 immediately. Incomplete last level: left/right heights differ at root → recurse; one side becomes perfect quickly. ✅

**Complexity:** O(log² n) time, O(log n) space.

**Follow-ups:**
1. *"Prove only one child is imperfect per level."* → In a complete tree, the last level fills left to right, so at most one subtree along the path to the last node is partially filled.
2. *"Binary search the last level?"* → Alternative O(log² n): binary search over the index of the last leaf, checking existence via path bits.

**Common mistakes:**
- Just doing an O(n) traversal (misses the point of the question).
- Off-by-one with `(1 << h) - 1` vs `(1 << h)`.

---

### Q27: Morris Inorder Traversal (O(1) Space)
**Company:** Google, Amazon, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Perform an inorder traversal using O(1) extra space (no stack, no recursion).

**What interviewer is testing:**
Deep understanding of tree threading. Morris traversal temporarily creates "threads" from a subtree's rightmost node back to its in-order successor, then removes them.

**Ideal Answer:**

For each node `cur`:
- If `cur` has no left child, visit it and go right.
- Otherwise find `cur`'s in-order predecessor (rightmost node of its left subtree):
  - If the predecessor's right is null, set it to `cur` (create a thread) and move left.
  - If the predecessor's right is already `cur`, the left subtree is done: remove the thread, visit `cur`, go right.

```java
public List<Integer> morrisInorder(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    TreeNode cur = root;
    while (cur != null) {
        if (cur.left == null) {
            res.add(cur.val);            // visit
            cur = cur.right;
        } else {
            TreeNode pred = cur.left;     // find in-order predecessor
            while (pred.right != null && pred.right != cur) pred = pred.right;
            if (pred.right == null) {
                pred.right = cur;         // create thread
                cur = cur.left;
            } else {
                pred.right = null;        // remove thread (restore tree)
                res.add(cur.val);         // visit after left subtree done
                cur = cur.right;
            }
        }
    }
    return res;
}
```

**Why O(1) space:** No stack or recursion; the threads reuse existing null right pointers and are removed after use, so the tree is restored.

**Dry run** — tree `1 -> left 2 -> left 3`:
- cur=1 has left. pred = rightmost of left subtree of 1. Thread created, move left to 2. Eventually visits 3,2,1 in order. ✅

**Complexity:** O(n) time (each edge traversed at most twice), O(1) extra space.

**Follow-ups:**
1. *"Is the tree modified during traversal?"* → Temporarily yes (threads), but fully restored by the end.
2. *"Morris preorder?"* → Visit when creating the thread instead of when removing it.

**Common mistakes:**
- Not removing the thread → infinite loop and corrupted tree.
- Wrong predecessor loop condition (`pred.right != cur` guard is essential).

---

### Q28: Populating Next Right Pointers in Each Node
**Company:** Amazon, Microsoft, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a **perfect** binary tree with a `next` pointer in each node, connect each node's `next` to the node immediately to its right on the same level (rightmost gets null). Use O(1) extra space.

**What interviewer is testing:**
Using already-established `next` pointers of the current level to wire up the next level — the O(1)-space "established links" trick.

**Ideal Answer:**

Process level by level using a `leftmost` pointer. For each node on the current level (traversed via its own `next` chain), connect its children: `node.left.next = node.right`, and `node.right.next = node.next.left` (if `node.next` exists).

```java
class Node { int val; Node left, right, next; }

public Node connect(Node root) {
    if (root == null) return null;
    Node leftmost = root;
    while (leftmost.left != null) {        // while there's a next level
        Node head = leftmost;
        while (head != null) {
            head.left.next = head.right;                       // link siblings
            if (head.next != null)
                head.right.next = head.next.left;              // link across parents
            head = head.next;                                  // move along current level
        }
        leftmost = leftmost.left;          // descend
    }
    return root;
}
```

**Complexity:** O(n) time, O(1) space (we reuse the next pointers as the "queue").

**Follow-ups:**
1. *"What if the tree is NOT perfect (Populating II)?"* → You can't assume both children exist; use a dummy head per level and a moving tail pointer to chain whatever children appear.
2. *"BFS with a queue?"* → Works but uses O(w) space; the interviewer wants O(1).

**Common mistakes:**
- Assuming `head.next.left` exists in the imperfect variant.
- Using a queue when O(1) is requested.

---

### Q29: Sum Root to Leaf Numbers
**Company:** Amazon, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Each root-to-leaf path encodes a number (e.g., 1→2→3 is 123). Return the sum of all such numbers.

**What interviewer is testing:**
Top-down accumulation (`cur = cur*10 + node.val`) summed at leaves.

**Ideal Answer:**

```java
public int sumNumbers(TreeNode root) {
    return dfs(root, 0);
}
private int dfs(TreeNode node, int cur) {
    if (node == null) return 0;
    cur = cur * 10 + node.val;                       // extend the number
    if (node.left == null && node.right == null) return cur;  // leaf -> emit
    return dfs(node.left, cur) + dfs(node.right, cur);
}
```

**Dry run** — `[1,2,3]`: leaf 2 → 12, leaf 3 → 13, sum 25. ✅

**Complexity:** O(n) time, O(h) space.

**Follow-up:** *"Binary digits instead?"* → Use `cur*2 + node.val` (relates to "Sum of Root To Leaf Binary Numbers").

**Common mistakes:**
- Adding `cur` at internal nodes instead of only at leaves.

---

### Q30: Count Good Nodes in Binary Tree
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
A node is "good" if no node on the path from the root to it has a greater value. Count the good nodes.

**What interviewer is testing:**
Top-down DFS carrying the max-so-far along the path.

**Ideal Answer:**

```java
public int goodNodes(TreeNode root) {
    return dfs(root, Integer.MIN_VALUE);
}
private int dfs(TreeNode node, int maxSoFar) {
    if (node == null) return 0;
    int good = node.val >= maxSoFar ? 1 : 0;     // good if >= all ancestors
    int newMax = Math.max(maxSoFar, node.val);
    return good + dfs(node.left, newMax) + dfs(node.right, newMax);
}
```

**Complexity:** O(n) time, O(h) space.

**Follow-up:** *"Why MIN_VALUE init?"* → The root is always good (no ancestors); `>=` against MIN_VALUE guarantees it.

**Common mistakes:**
- Using `>` instead of `>=` (equal values along the path are still good per the definition).

---

### Q31: Maximum Width of Binary Tree
**Company:** Amazon, Facebook, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
The width of a level is the distance between the leftmost and rightmost non-null nodes, counting the null gaps between them as if the level were complete. Return the maximum width.

**What interviewer is testing:**
Indexing nodes as if in a complete tree (left child `2i`, right child `2i+1`) and computing width per level, with overflow awareness.

**Ideal Answer:**

BFS, pairing each node with its position index. The width of a level is `lastIndex − firstIndex + 1`. To avoid overflow on deep trees, **normalize** each level's indices by subtracting the first index.

```java
public int widthOfBinaryTree(TreeNode root) {
    if (root == null) return 0;
    int maxWidth = 0;
    Queue<Pair> q = new LinkedList<>();
    q.offer(new Pair(root, 0));
    while (!q.isEmpty()) {
        int size = q.size();
        int first = 0, last = 0;
        int offset = q.peek().index;          // normalize to prevent overflow
        for (int i = 0; i < size; i++) {
            Pair p = q.poll();
            int idx = p.index - offset;        // shift indices down
            if (i == 0) first = idx;
            if (i == size - 1) last = idx;
            if (p.node.left != null)  q.offer(new Pair(p.node.left,  2 * idx));
            if (p.node.right != null) q.offer(new Pair(p.node.right, 2 * idx + 1));
        }
        maxWidth = Math.max(maxWidth, last - first + 1);
    }
    return maxWidth;
}
class Pair {
    TreeNode node; int index;
    Pair(TreeNode n, int i) { node = n; index = i; }
}
```

**Complexity:** O(n) time, O(w) space.

**Follow-ups:**
1. *"Why normalize indices?"* → A skewed tree's indices double each level; they overflow `int` (even `long`) quickly. Subtracting the level's first index keeps them small.

**Common mistakes:**
- Counting only non-null nodes (the gaps must count).
- Integer overflow without normalization.

---

### Q32: All Nodes Distance K in Binary Tree
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given the root, a `target` node, and integer `k`, return all node values that are distance `k` from `target` (edges in any direction — up via parent or down via children).

**What interviewer is testing:**
Converting a tree into an undirected graph (add parent pointers) then doing BFS from the target. The "distance in any direction" requirement is the key insight.

**Ideal Answer:**

First map each node to its parent (one DFS). Then BFS from `target` outward through three neighbors (left, right, parent), tracking visited, until we reach distance `k`.

```java
public List<Integer> distanceK(TreeNode root, TreeNode target, int k) {
    Map<TreeNode, TreeNode> parent = new HashMap<>();
    buildParents(root, null, parent);

    Queue<TreeNode> q = new LinkedList<>();
    Set<TreeNode> visited = new HashSet<>();
    q.offer(target); visited.add(target);
    int dist = 0;
    while (!q.isEmpty()) {
        if (dist == k) {                       // all nodes at this layer are answers
            List<Integer> res = new ArrayList<>();
            for (TreeNode n : q) res.add(n.val);
            return res;
        }
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            for (TreeNode nei : new TreeNode[]{n.left, n.right, parent.get(n)}) {
                if (nei != null && visited.add(nei)) q.offer(nei);
            }
        }
        dist++;
    }
    return new ArrayList<>();
}
private void buildParents(TreeNode node, TreeNode par, Map<TreeNode, TreeNode> parent) {
    if (node == null) return;
    parent.put(node, par);
    buildParents(node.left, node, parent);
    buildParents(node.right, node, parent);
}
```

**Complexity:** O(n) time and space.

**Pattern:** "Tree → graph via parent pointers → BFS" — a recurring trick whenever distance can travel upward.

**Follow-ups:**
1. *"Without a parent map?"* → A clever recursive solution computes the distance from target found in a subtree and adds downward nodes from ancestors, but it's harder to get right; the parent-map BFS is clearest.
2. *"Return count instead of values?"* → Same BFS, count the layer.

**Common mistakes:**
- Only going downward from target (misses nodes on the other side of the tree).
- Forgetting `visited` → revisits the parent's child you came from.

---

### Q33: House Robber III
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Houses form a binary tree. You can't rob two directly-connected houses (parent and child). Maximize the total robbed.

**What interviewer is testing:**
Tree DP returning two states per node: best if we rob this node, best if we don't. Avoids the exponential recomputation of the naive recursion.

**Ideal Answer:**

Each call returns `[robThis, skipThis]`:
- `robThis = node.val + skipLeft + skipRight` (can't rob children if we rob here).
- `skipThis = max(robLeft, skipLeft) + max(robRight, skipRight)` (children free to choose).

```java
public int rob(TreeNode root) {
    int[] r = robSub(root);
    return Math.max(r[0], r[1]);
}
// returns {maxIfRobThis, maxIfSkipThis}
private int[] robSub(TreeNode node) {
    if (node == null) return new int[]{0, 0};
    int[] left = robSub(node.left);
    int[] right = robSub(node.right);
    int robThis  = node.val + left[1] + right[1];               // children skipped
    int skipThis = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
    return new int[]{robThis, skipThis};
}
```

**Dry run** — `[3,2,3,null,3,null,1]`: optimal = 3(root) + 3 + 1 = 7. ✅

**Complexity:** O(n) time, O(h) space.

**Follow-up:** *"Naive recursion complexity?"* → Without the two-state return, you'd recompute grandchildren repeatedly → exponential. Memoize by node or use this two-state trick.

**Common mistakes:**
- Returning a single number and recomputing grandchildren (exponential).

---

### Q34: Binary Tree Cameras
**Company:** Google, Amazon
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Install cameras on tree nodes; each camera covers its parent, itself, and its children. Return the minimum number of cameras to cover all nodes.

**What interviewer is testing:**
Greedy post-order with three states. Place cameras as low as possible (just above leaves) — a greedy that needs a clean state machine.

**Ideal Answer:**

Each node reports one of three states to its parent:
- `0` = NOT covered (needs a camera above it).
- `1` = covered but has NO camera.
- `2` = has a camera.

Post-order logic:
- If any child is NOT covered (`0`) → place a camera here (return 2, increment count).
- If any child HAS a camera (`2`) → this node is covered without one (return 1).
- Otherwise (both children covered, no camera) → this node is not covered (return 0), pushing the camera decision up.

```java
private int cameras;

public int minCameraCover(TreeNode root) {
    cameras = 0;
    // if root ends up uncovered, it needs its own camera
    if (dfs(root) == 0) cameras++;
    return cameras;
}
// 0 = needs cover, 1 = covered no camera, 2 = has camera
private int dfs(TreeNode node) {
    if (node == null) return 1;           // null is "covered" so leaves aren't forced
    int left = dfs(node.left);
    int right = dfs(node.right);
    if (left == 0 || right == 0) {        // a child is uncovered -> camera here
        cameras++;
        return 2;
    }
    if (left == 2 || right == 2) return 1; // covered by a child's camera
    return 0;                              // both children covered, none here -> uncovered
}
```

**Why null returns 1:** If null counted as "needs cover," every leaf would force a camera on itself; returning 1 (covered) lets the greedy place cameras on the *parents* of leaves, which is optimal.

**Complexity:** O(n) time, O(h) space.

**Follow-ups:**
1. *"Prove the greedy is optimal."* → Placing a camera at the parent of a leaf covers at least as much as placing it at the leaf, and never worse; an exchange argument shows the bottom-up greedy never uses more cameras than optimal.
2. *"DP formulation?"* → A 3-state DP per node also works but is more verbose; the greedy is tighter.

**Common mistakes:**
- Forgetting the final root check (`dfs(root) == 0`).
- Having null return 0, forcing redundant leaf cameras.

---

# BINARY SEARCH TREES

A BST encodes order: for every node, all left-subtree values are smaller and all right-subtree values larger. Two facts power almost every BST question: **(1) an inorder traversal yields sorted ascending order**, and **(2) at each node you can discard half the tree by comparing with the target** (the binary-search property). If a problem mentions "BST" and you're not exploiting one of these two, you're probably overcomplicating it.

---

### Q35: Search in a Binary Search Tree
**Company:** Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** OA / Phone

**Question:**
Return the subtree rooted at the node with the given value, or null.

**What interviewer is testing:**
The binary-search descent — go left if target is smaller, right if larger.

**Ideal Answer:**

```java
public TreeNode searchBST(TreeNode root, int val) {
    while (root != null && root.val != val) {
        root = val < root.val ? root.left : root.right;  // discard half
    }
    return root;
}
```

**Complexity:** O(h) time — O(log n) balanced, O(n) skewed. O(1) iterative space.

**Follow-up:** *"Recursive version?"* → `return val < root.val ? search(root.left,val) : search(root.right,val)` with null/equality base cases.

**Common mistakes:**
- Doing a full traversal instead of the binary-search descent (ignores the BST property).

---

### Q36: Insert into a Binary Search Tree
**Company:** Amazon, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Insert a value into a BST and return the root. The value is guaranteed not to exist already.

**What interviewer is testing:**
Descending to the correct null spot and attaching a new leaf.

**Ideal Answer:**

```java
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);     // found the empty slot
    if (val < root.val) root.left = insertIntoBST(root.left, val);
    else                root.right = insertIntoBST(root.right, val);
    return root;
}
```

*Iterative* — track a parent pointer, descend, attach:

```java
public TreeNode insertIterative(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    TreeNode cur = root;
    while (true) {
        if (val < cur.val) {
            if (cur.left == null) { cur.left = new TreeNode(val); break; }
            cur = cur.left;
        } else {
            if (cur.right == null) { cur.right = new TreeNode(val); break; }
            cur = cur.right;
        }
    }
    return root;
}
```

**Complexity:** O(h) time, O(h)/O(1) space.

**Follow-up:** *"Maintain balance?"* → AVL/Red-Black rotations; out of scope for a plain BST but mention it.

**Common mistakes:**
- Forgetting to reassign `root.left = insert(...)` (the recursive return must be wired back in).

---

### Q37: Delete Node in a BST
**Company:** Amazon, Microsoft, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Delete the node with a given key from a BST and return the (possibly new) root, preserving the BST property.

**What interviewer is testing:**
The three deletion cases — and especially the two-children case, where you replace with the inorder successor.

**Ideal Answer:**

Descend to the target. Then:
1. **No children** → return null (remove it).
2. **One child** → return that child (splice it up).
3. **Two children** → find the inorder successor (smallest value in the right subtree), copy its value into this node, then delete the successor from the right subtree (which now falls into case 1 or 2).

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } else {                                   // found the node to delete
        if (root.left == null) return root.right;   // case 1/2
        if (root.right == null) return root.left;   // case 1/2
        // case 3: two children -> inorder successor (min of right subtree)
        TreeNode succ = root.right;
        while (succ.left != null) succ = succ.left;
        root.val = succ.val;                          // copy successor value up
        root.right = deleteNode(root.right, succ.val);// delete the successor
    }
    return root;
}
```

**Dry run** — delete 3 from BST `5(3(2,4),6(_,7))`: 3 has two children; successor = min of right subtree of 3 = 4. Copy 4 into the node, delete 4 from right subtree → `5(4(2,_),6(_,7))`. ✅

**Complexity:** O(h) time, O(h) space.

**Follow-up questions the interviewer will ask:**
1. *"Why the inorder successor (not predecessor)?"* → Either works; the successor (min of right subtree) is conventional. Using the predecessor (max of left subtree) is equally valid.
2. *"Could deleting unbalance the tree?"* → Yes; a self-balancing BST would rebalance via rotations.

**Common mistakes:**
- Forgetting to actually delete the successor after copying its value (leaves a duplicate).
- Mishandling the one-child case by returning `root` instead of the child.

---

### Q38: Convert Sorted Array to Binary Search Tree
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given a sorted ascending array, build a **height-balanced** BST.

**What interviewer is testing:**
Recognizing that the middle element should be the root to balance the tree, then recursing on halves.

**Ideal Answer:**

Pick the middle as root → left half builds the left subtree, right half the right subtree. This keeps heights balanced by construction.

```java
public TreeNode sortedArrayToBST(int[] nums) {
    return build(nums, 0, nums.length - 1);
}
private TreeNode build(int[] nums, int lo, int hi) {
    if (lo > hi) return null;
    int mid = lo + (hi - lo) / 2;            // middle as root for balance
    TreeNode root = new TreeNode(nums[mid]);
    root.left  = build(nums, lo, mid - 1);
    root.right = build(nums, mid + 1, hi);
    return root;
}
```

**Complexity:** O(n) time, O(log n) space (recursion depth).

**Follow-up:** *"Multiple valid answers?"* → Yes; choosing left-mid vs right-mid gives different but equally balanced trees.

**Common mistakes:**
- `mid = (lo + hi) / 2` overflow on huge ranges (use `lo + (hi-lo)/2`).

---

### Q39: Convert Sorted List to Binary Search Tree
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a sorted singly linked list, build a height-balanced BST.

**What interviewer is testing:**
Either (a) find the middle with slow/fast pointers each recursion (O(n log n)), or (b) the elegant **inorder simulation** that builds the tree in O(n) by consuming the list left to right.

**Ideal Answer:**

*O(n) inorder-simulation:* Count the nodes. Then build the tree via an inorder recursion that "consumes" list nodes in sorted order — because an inorder traversal of a BST visits nodes in ascending order, exactly matching the list order. A shared pointer advances as we build.

```java
private ListNode cur;

public TreeNode sortedListToBST(ListNode head) {
    int n = 0;
    for (ListNode p = head; p != null; p = p.next) n++;
    cur = head;
    return build(0, n - 1);
}
// builds a balanced BST for the next (hi-lo+1) list values, inorder
private TreeNode build(int lo, int hi) {
    if (lo > hi) return null;
    int mid = lo + (hi - lo) / 2;
    TreeNode left = build(lo, mid - 1);   // build left subtree FIRST (inorder)
    TreeNode root = new TreeNode(cur.val); // then consume the current list node
    cur = cur.next;
    root.left = left;
    root.right = build(mid + 1, hi);      // then right subtree
    return root;
}
```

**Why this works:** We never actually search the list for the middle; instead we *simulate* an inorder build. Because inorder of the resulting balanced BST is ascending, and the list is ascending, consuming nodes in inorder order assigns each list value to its correct BST position.

**Complexity:** O(n) time, O(log n) space.

**Follow-ups:**
1. *"The slow/fast approach complexity?"* → O(n log n): finding the middle is O(n) and done at each of O(log n) levels.
2. *"Why is inorder simulation O(n)?"* → Each list node is consumed exactly once.

**Common mistakes:**
- Building right subtree before consuming the current node (breaks the inorder correspondence).

---

### Q40: Validate Binary Search Tree
**Company:** Amazon, Microsoft, Facebook, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Determine whether a binary tree is a valid BST (every node's left subtree strictly smaller, right subtree strictly larger — applied recursively, not just to immediate children).

**What interviewer is testing:**
The classic trap: checking only `node.left.val < node.val < node.right.val` is wrong. Two correct approaches — range bounds, or inorder-must-be-strictly-increasing.

**Ideal Answer:**

*Approach 1 — range bounds:* each node must lie in an open interval `(low, high)`. Going left tightens the upper bound to the node's value; going right tightens the lower bound. Use `Long` (or `null`) bounds to avoid edge cases with `Integer.MIN/MAX_VALUE`.

```java
public boolean isValidBST(TreeNode root) {
    return valid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean valid(TreeNode node, long low, long high) {
    if (node == null) return true;
    if (node.val <= low || node.val >= high) return false;  // strict bounds
    return valid(node.left, low, node.val)        // left: upper bound = node.val
        && valid(node.right, node.val, high);     // right: lower bound = node.val
}
```

*Approach 2 — inorder strictly increasing:* do an inorder traversal; if any value is ≤ the previous, it's invalid.

```java
private Integer prev = null;
public boolean isValidBSTInorder(TreeNode root) {
    prev = null;
    return inorder(root);
}
private boolean inorder(TreeNode node) {
    if (node == null) return true;
    if (!inorder(node.left)) return false;
    if (prev != null && node.val <= prev) return false;  // not strictly increasing
    prev = node.val;
    return inorder(node.right);
}
```

**Dry run** — tree `5(1, 4(3,6))`: at node 4, the range from going right of 5 is `(5, ∞)`, but 4 ≤ 5 → invalid. (The naive child-only check would wrongly pass this since 3<4<6.) ✅

**Complexity:** O(n) time, O(h) space.

**Follow-up questions the interviewer will ask:**
1. *"Why use Long bounds?"* → A node value of `Integer.MIN_VALUE` or `MAX_VALUE` would falsely fail `<= low` / `>= high` with int bounds. `Long` (or nullable bounds) sidesteps it.
2. *"Duplicates allowed?"* → Depends on the BST definition; the strict `<` / `>` here disallows them. Change to `<=` if duplicates are allowed on one side.
3. *"Which approach is cleaner?"* → Both are O(n); inorder is intuitive (a BST's inorder is sorted), bounds avoids storing `prev`.

**Common mistakes:**
- Only comparing a node with its immediate children (the #1 rejection cause).
- Using `Integer.MIN/MAX_VALUE` as bounds and breaking on extreme node values.

---

### Q41: Kth Smallest Element in a BST
**Company:** Amazon, Microsoft, Google, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the kth smallest value (1-indexed) in a BST.

**What interviewer is testing:**
Inorder gives ascending order, so the kth visited node is the answer — and you can stop early.

**Ideal Answer:**

*Iterative inorder with early stop:*

```java
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.left; }
        cur = stack.pop();
        if (--k == 0) return cur.val;     // kth node in inorder order
        cur = cur.right;
    }
    throw new IllegalArgumentException("k larger than tree size");
}
```

**Complexity:** O(h + k) time (descend then pop k nodes), O(h) space.

**Follow-up questions the interviewer will ask:**
1. *"What if the BST is modified often and you need many kth-smallest queries?"* → Augment each node with `leftSubtreeSize` (number of nodes in its left subtree). Then kth-smallest is O(h): if `leftSize + 1 == k` return node; if `k <= leftSize` go left; else go right with `k -= leftSize + 1`. Insert/delete update sizes in O(h).
2. *"Kth largest?"* → Reverse inorder (right, node, left), Q42.

**Common mistakes:**
- Doing a full inorder into a list then indexing (works but wastes time/space; early-stop is better).
- Off-by-one on 1-indexed k.

---

### Q42: Kth Largest Element in a BST
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Return the kth largest value in a BST.

**What interviewer is testing:**
Reverse inorder (right → node → left) yields descending order; the kth visited node is the answer.

**Ideal Answer:**

```java
public int kthLargest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.right; }  // go RIGHT first
        cur = stack.pop();
        if (--k == 0) return cur.val;
        cur = cur.left;
    }
    throw new IllegalArgumentException("k larger than tree size");
}
```

**Complexity:** O(h + k) time, O(h) space.

**Follow-up:** *"Same augmentation trick?"* → Yes, with `rightSubtreeSize`, symmetric to Q41.

**Common mistakes:**
- Forgetting to descend right first for descending order.

---

### Q43: Inorder Successor in BST
**Company:** Amazon, Microsoft, Facebook, Pinterest
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a node `p` in a BST, return its inorder successor (the node with the smallest value greater than `p.val`), or null if none.

**What interviewer is testing:**
Two cases driven by the BST structure — and the O(h) solution without a parent pointer.

**Ideal Answer:**

- If `p` has a right subtree, the successor is the leftmost node of that right subtree.
- Otherwise, the successor is the lowest ancestor for which `p` is in the left subtree. Find it by descending from the root: whenever you go left, that node is a successor candidate.

```java
public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
    if (p.right != null) {                  // case 1: leftmost of right subtree
        TreeNode cur = p.right;
        while (cur.left != null) cur = cur.left;
        return cur;
    }
    // case 2: lowest ancestor where p is in its left subtree
    TreeNode succ = null, cur = root;
    while (cur != null) {
        if (p.val < cur.val) { succ = cur; cur = cur.left; }  // candidate; go left
        else                 cur = cur.right;
    }
    return succ;
}
```

**Dry run** — BST `20(10(5,15),30)`, p=15: 15 has no right child. Descend from 20: 15<20 → succ=20, go left to 10; 15>10 → go right to 15; 15<15 false → go right to null. Successor = 20. ✅

**Complexity:** O(h) time, O(1) space.

**Follow-ups:**
1. *"With parent pointers?"* → If no right child, walk up until you come from a left child; that parent is the successor.
2. *"Predecessor?"* → Mirror: left subtree's rightmost, or lowest ancestor where p is in the right subtree.

**Common mistakes:**
- Forgetting the no-right-subtree case (returns null incorrectly).

---

### Q44: BST Iterator
**Company:** Amazon, Facebook, Microsoft, Google, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Implement an iterator over a BST: `next()` returns the next smallest value, `hasNext()` returns whether more exist. `next()` and `hasNext()` should be **average O(1)** and use **O(h) memory**.

**What interviewer is testing:**
A "controlled" / lazy inorder traversal using an explicit stack — pausing the traversal between calls. This is the iterative-inorder pattern (Q1) turned into a class.

**Ideal Answer:**

Keep a stack holding the path to the next node to return. Initialize by pushing the entire left spine from the root. On `next()`, pop a node (that's the answer); if it has a right child, push the right child's entire left spine. `hasNext()` is `!stack.isEmpty()`.

```java
class BSTIterator {
    private final Deque<TreeNode> stack = new ArrayDeque<>();

    public BSTIterator(TreeNode root) {
        pushLeftSpine(root);
    }
    public int next() {
        TreeNode node = stack.pop();        // smallest unvisited
        if (node.right != null) pushLeftSpine(node.right);
        return node.val;
    }
    public boolean hasNext() {
        return !stack.isEmpty();
    }
    private void pushLeftSpine(TreeNode node) {
        while (node != null) { stack.push(node); node = node.left; }
    }
}
```

**Why average O(1):** Each node is pushed and popped exactly once over the whole iteration. A single `next()` may push a whole spine (O(h)), but *amortized* across all n calls it's O(1). Space is O(h) — the stack never holds more than one root-to-node path.

**Dry run** — BST `7(3,15(9,20))`: init pushes 7,3 (left spine). next()→pop 3 (no right) → 3. next()→pop 7, push left spine of 15 → 15,9; → 7. next()→pop 9 → 9. next()→pop 15, push 20 → 15. next()→pop 20 → 20. Order: 3,7,9,15,20. ✅

**Complexity:** Amortized O(1) per `next()`/`hasNext()`, O(h) space.

**Follow-up questions the interviewer will ask:**
1. *"Why is it O(1) amortized, not worst case?"* → A single call can push O(h) nodes, but each node is touched once total across n calls → O(n)/n = O(1) average.
2. *"Add a `prev()` (bidirectional)?"* → Harder; you'd maintain two stacks or fall back to an indexed inorder array (O(n) space) — discuss the trade-off.
3. *"Why not flatten to an array in the constructor?"* → That's O(n) space and O(n) upfront work; the lazy stack is O(h) space, which the problem requires.

**Common mistakes:**
- Flattening the whole tree (violates O(h) memory).
- Forgetting to push the right child's left spine after popping.

---

### Q45: Recover Binary Search Tree
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Exactly two nodes of a BST were swapped by mistake. Recover the tree without changing its structure (swap the two values back). Aim for O(1) extra space.

**What interviewer is testing:**
The inorder-is-sorted insight applied to anomaly detection, plus the subtle "adjacent vs non-adjacent swap" case. The O(1) version uses Morris traversal.

**Ideal Answer:**

In a correct BST, inorder is strictly increasing. When two nodes are swapped, the inorder sequence has either one or two "descents" (a place where `prev.val > cur.val`):
- **Two descents** (non-adjacent swap): the first wrong node is the *larger* element of the first descent (`prev`); the second is the *smaller* element of the second descent (`cur`).
- **One descent** (adjacent swap): the two swapped nodes are exactly `prev` and `cur` of that single descent.

Logic: on the first descent, set `first = prev` and `second = cur`. On any subsequent descent, update `second = cur`. Finally swap `first.val` and `second.val`.

```java
private TreeNode first, second, prev;

public void recoverTree(TreeNode root) {
    first = second = prev = null;
    inorder(root);
    int t = first.val; first.val = second.val; second.val = t;  // swap back
}
private void inorder(TreeNode node) {
    if (node == null) return;
    inorder(node.left);
    if (prev != null && prev.val > node.val) {   // a descent
        if (first == null) first = prev;          // first anomaly: take the larger (prev)
        second = node;                            // always update the smaller (cur)
    }
    prev = node;
    inorder(node.right);
}
```

*O(1) space (Morris):* replace the recursion with Morris inorder, keeping the same `first/second/prev` anomaly logic. The structure is identical; you just thread the tree.

**Dry run** — inorder of broken tree gives `1,3,2,4` (3 and 2 swapped, adjacent). Single descent at (3,2): first=3, second=2. Swap → restores `1,2,3,4`. ✅
Non-adjacent: inorder `3,2,1` from swapping 1 and 3 → descent1 (3,2): first=3, second=2; descent2 (2,1): second=1. Swap first(3) and second(1) → `1,2,3`. ✅

**Complexity:** O(n) time; O(h) recursive or O(1) with Morris.

**Follow-up questions the interviewer will ask:**
1. *"Why take `prev` for the first anomaly but `cur` for the second?"* → In `...A B...` with a descent A>B, on the first descent the misplaced *large* value is A (prev); on the last descent the misplaced *small* value is B (cur). Adjacent swap collapses both into one descent.
2. *"How to get true O(1) space?"* → Morris traversal.

**Common mistakes:**
- Handling only the two-descent case and breaking on adjacent swaps (one descent).
- Setting `first = node` instead of `first = prev` on the first anomaly.

---

### Q46: Lowest Common Ancestor of a Binary Search Tree
**Company:** Amazon, Microsoft, Facebook, LinkedIn
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Find the LCA of two nodes `p` and `q` in a BST.

**What interviewer is testing:**
Exploiting the BST property to do LCA in O(h) without exploring both subtrees (unlike the general tree LCA, Q13).

**Ideal Answer:**

Descend from the root. If both `p` and `q` are smaller than the current node, the LCA is in the left subtree; if both larger, the right. The moment they **split** (one ≤ node ≤ other, or node equals one of them), the current node is the LCA.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode cur = root;
    while (cur != null) {
        if (p.val < cur.val && q.val < cur.val)      cur = cur.left;   // both left
        else if (p.val > cur.val && q.val > cur.val) cur = cur.right;  // both right
        else return cur;                                                // split point
    }
    return null;
}
```

**Dry run** — BST `6(2(0,4),8(7,9))`, p=2, q=8: at 6, 2<6 and 8>6 → split → LCA 6. ✅ p=2, q=4: at 6 both <6 → go left to 2; 2==2 → return 2. ✅

**Complexity:** O(h) time, O(1) space.

**Follow-up:** *"Why is this easier than the general tree LCA?"* → The BST property tells you which subtree contains both nodes without searching; the general version (Q13) must explore both subtrees.

**Common mistakes:**
- Using the general O(n) algorithm and ignoring the BST property.

---

### Q47: Two Sum IV — Input is a BST
**Company:** Amazon, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given a BST and a target, return true if two distinct nodes sum to the target.

**What interviewer is testing:**
Combining traversal with the Two Sum hashset trick, or the two-pointer-on-BST approach using two iterators.

**Ideal Answer:**

*HashSet (simplest):* traverse; for each value check if `target - value` was seen.

```java
public boolean findTarget(TreeNode root, int k) {
    Set<Integer> seen = new HashSet<>();
    return dfs(root, k, seen);
}
private boolean dfs(TreeNode node, int k, Set<Integer> seen) {
    if (node == null) return false;
    if (seen.contains(k - node.val)) return true;
    seen.add(node.val);
    return dfs(node.left, k, seen) || dfs(node.right, k, seen);
}
```

*Two-pointer with two BST iterators (O(h) space, exploits sortedness):* an ascending iterator (inorder) and a descending iterator (reverse inorder); move them toward each other like Two-Sum-Sorted.

**Complexity:** HashSet O(n) time, O(n) space. Two-iterator O(n) time, O(h) space.

**Follow-up:** *"O(h) space using the BST property?"* → The two-iterator approach (BSTIterator forward + a reverse iterator).

**Common mistakes:**
- Using the same node twice (the seen-set check before adding the current value prevents this).

---

# HEAPS / PRIORITY QUEUES

A heap gives you O(log n) insert and O(1) peek of the min (or max). The pattern triggers are unmistakable: **"kth largest/smallest," "top k," "median of a stream," "schedule by priority," "merge sorted things."** Two recurring tricks dominate: (1) a **fixed-size heap of size k** to track the kth element in O(n log k), and (2) **two heaps** (a max-heap of the lower half, a min-heap of the upper half) to track a running median or balance a partition.

In Java: `PriorityQueue` is a min-heap by default; pass `Collections.reverseOrder()` or a comparator for a max-heap.

---

### Q48: Kth Largest Element in an Array
**Company:** Amazon, Facebook, Microsoft, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Return the kth largest element in an unsorted array (the kth largest in sorted order, not the kth distinct).

**What interviewer is testing:**
Two approaches and their trade-offs: a size-k min-heap (O(n log k)), and QuickSelect (O(n) average). Knowing QuickSelect and its partition is a strong signal.

**Ideal Answer:**

*Min-heap of size k:* keep the k largest seen so far in a min-heap; the heap's top is the kth largest. Whenever size exceeds k, poll the smallest.

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>(); // min-heap
    for (int x : nums) {
        heap.offer(x);
        if (heap.size() > k) heap.poll();   // drop the smallest -> keep k largest
    }
    return heap.peek();                       // smallest of the k largest = kth largest
}
```

*QuickSelect (O(n) average):* like quicksort but recurse into only the side containing the kth element. We want the element at sorted index `n - k` (0-indexed).

```java
public int findKthLargestQuickSelect(int[] nums, int k) {
    int target = nums.length - k;             // index in ascending sorted order
    int lo = 0, hi = nums.length - 1;
    Random rnd = new Random();
    while (lo < hi) {
        int pivotIdx = lo + rnd.nextInt(hi - lo + 1);  // random pivot avoids worst case
        int p = partition(nums, lo, hi, pivotIdx);
        if (p == target) return nums[p];
        else if (p < target) lo = p + 1;       // target is to the right
        else hi = p - 1;                        // target is to the left
    }
    return nums[lo];
}
// Lomuto partition around nums[pivotIdx]; returns final pivot index
private int partition(int[] nums, int lo, int hi, int pivotIdx) {
    int pivot = nums[pivotIdx];
    swap(nums, pivotIdx, hi);                  // move pivot to end
    int store = lo;
    for (int i = lo; i < hi; i++) {
        if (nums[i] < pivot) swap(nums, store++, i);
    }
    swap(nums, store, hi);                      // place pivot in its sorted position
    return store;
}
private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
```

**Dry run (heap)** — `[3,2,1,5,6,4]`, k=2: heap keeps 2 largest. After processing: heap holds {5,6}, top=5 = 2nd largest. ✅

**Why QuickSelect is O(n) average:** each partition halves the search range on average, giving `n + n/2 + n/4 + ... = O(n)`. The random pivot makes the O(n²) worst case astronomically unlikely.

**Complexity:** Heap O(n log k) time, O(k) space. QuickSelect O(n) average / O(n²) worst, O(1) space.

**Follow-up questions the interviewer will ask:**
1. *"Heap vs QuickSelect trade-offs?"* → Heap is O(n log k) but works on streams and doesn't modify the array; QuickSelect is O(n) average but mutates the array and is O(n²) worst case (mitigated by random pivots / median-of-medians).
2. *"Guaranteed O(n)?"* → Median-of-medians pivot selection gives worst-case O(n), but with a large constant; rarely needed in practice.
3. *"Kth largest in a stream?"* → That's Q57; the size-k min-heap is the natural fit.

**Common mistakes:**
- Using a max-heap of all n elements then polling k times → O(n + k log n), worse than the size-k heap for large n.
- QuickSelect: forgetting the random pivot → O(n²) on sorted input.
- Off-by-one: kth largest is index `n - k` in ascending order.

---

### Q49: Find Median from Data Stream
**Company:** Amazon, Facebook, Google, Microsoft, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Design a data structure that supports `addNum(int)` and `findMedian()` over a growing stream of integers.

**What interviewer is testing:**
The **two-heap** technique — a max-heap for the lower half and a min-heap for the upper half, kept balanced so the median is at the heaps' tops. A canonical FAANG design question.

**Ideal Answer:**

Maintain:
- `lo` = a **max-heap** of the smaller half (its top is the largest of the small half).
- `hi` = a **min-heap** of the larger half (its top is the smallest of the large half).

Invariants: every element in `lo` ≤ every element in `hi`, and `lo.size()` equals `hi.size()` or is exactly one greater. Then:
- If sizes equal → median is the average of the two tops.
- If `lo` has one more → median is `lo.peek()`.

To add: push to `lo`, then move `lo`'s top to `hi` (keeps ordering), then if `hi` is bigger, move its top back to `lo` (rebalance). This "push then shuffle" guarantees both the ordering and size invariants.

```java
class MedianFinder {
    private final PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> hi = new PriorityQueue<>();                            // min-heap

    public void addNum(int num) {
        lo.offer(num);              // 1. add to lower half
        hi.offer(lo.poll());        // 2. balance ordering: largest of lo -> hi
        if (hi.size() > lo.size())  // 3. keep lo >= hi in size
            lo.offer(hi.poll());
    }
    public double findMedian() {
        if (lo.size() > hi.size()) return lo.peek();           // odd count
        return (lo.peek() + (double) hi.peek()) / 2.0;          // even count
    }
}
```

**Dry run** — add 1: lo[1]→hi[1]→lo[1] (lo bigger). add 2: lo[1,2]... after shuffle lo[1],hi[2]. median = (1+2)/2 = 1.5. add 3: ends lo[1,2], hi[3], median = lo.peek = 2. ✅

**Complexity:** `addNum` O(log n), `findMedian` O(1). Space O(n).

**Follow-up questions the interviewer will ask:**
1. *"What if 99% of numbers are in [0,100]?"* → Use a bucket/count array of size 101 for those, with overflow buckets for outliers; median via prefix counts in O(100).
2. *"Sliding window median?"* → Q55; you additionally need lazy deletion from the heaps.
3. *"Why push to lo then move to hi then maybe back?"* → It guarantees both the ordering invariant (every lo ≤ every hi) and the size invariant in O(log n) without special-casing.

**Common mistakes:**
- Comparing heap *sizes* incorrectly and returning the wrong top for odd counts.
- Not maintaining the ordering invariant (just balancing sizes) → tops aren't the true median.
- Integer overflow when averaging two large tops (cast to double / use long).

---

### Q50: Reorganize String
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Rearrange a string so that no two adjacent characters are the same. Return any valid arrangement, or "" if impossible.

**What interviewer is testing:**
The greedy "always place the most frequent remaining character" via a max-heap, and the feasibility condition.

**Ideal Answer:**

Count frequencies. If any character's count exceeds `(n+1)/2`, it's impossible (it can't be spread out). Otherwise, repeatedly take the two most frequent characters from a max-heap, append both (they differ), decrement, and re-add any still-positive. Holding back the just-used character until the next one is placed guarantees no adjacency.

```java
public String reorganizeString(String s) {
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    // feasibility: most frequent char must fit
    int maxCount = 0;
    for (int c : count) maxCount = Math.max(maxCount, c);
    if (maxCount > (s.length() + 1) / 2) return "";

    // max-heap by frequency
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> b[1] - a[1]); // {charIdx, freq}
    for (int i = 0; i < 26; i++) if (count[i] > 0) heap.offer(new int[]{i, count[i]});

    StringBuilder sb = new StringBuilder();
    while (heap.size() >= 2) {
        int[] a = heap.poll();
        int[] b = heap.poll();
        sb.append((char) ('a' + a[0]));   // place two different chars
        sb.append((char) ('a' + b[0]));
        if (--a[1] > 0) heap.offer(a);
        if (--b[1] > 0) heap.offer(b);
    }
    if (!heap.isEmpty()) {                  // one char left
        int[] last = heap.poll();
        if (last[1] > 1) return "";          // can't place 2+ of the same at the end
        sb.append((char) ('a' + last[0]));
    }
    return sb.toString();
}
```

**Dry run** — "aab": counts a=2,b=1. max 2 ≤ (3+1)/2=2 → feasible. Place a,b → "ab"; a left (1) → "aba". ✅

**Complexity:** O(n log k) where k ≤ 26 (so effectively O(n)), O(k) space.

**Follow-up questions the interviewer will ask:**
1. *"Prove the feasibility bound."* → A char with count > ⌈n/2⌉ would need more than half the positions; even spreading it to every other slot isn't enough.
2. *"Rearrange so same chars are at least k apart (Task Scheduler / Rearrange k Distance)?"* → Use a cooldown queue holding chars until k steps pass (related to Q60).

**Common mistakes:**
- Placing only one char per iteration (can re-adjacent).
- Forgetting the final leftover-char check.

---

### Q51: K Closest Points to Origin
**Company:** Amazon, Facebook, Microsoft, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given points on a plane, return the `k` closest to the origin (by Euclidean distance).

**What interviewer is testing:**
A size-k **max-heap** (so the farthest of the k is on top, ready to evict), or QuickSelect on distances.

**Ideal Answer:**

*Max-heap of size k:* keep the k closest; if a new point is closer than the heap's farthest, swap. Compare squared distances (no need for sqrt).

```java
public int[][] kClosest(int[][] points, int k) {
    // max-heap by squared distance: farthest on top
    PriorityQueue<int[]> heap = new PriorityQueue<>(
        (a, b) -> dist(b) - dist(a));
    for (int[] p : points) {
        heap.offer(p);
        if (heap.size() > k) heap.poll();    // evict the current farthest
    }
    int[][] res = new int[k][2];
    for (int i = 0; i < k; i++) res[i] = heap.poll();
    return res;
}
private int dist(int[] p) { return p[0] * p[0] + p[1] * p[1]; }
```

**Complexity:** O(n log k) time, O(k) space. QuickSelect alternative: O(n) average, O(1) extra.

**Follow-ups:**
1. *"Why squared distance?"* → sqrt is monotonic and unnecessary; avoids floating point and is faster.
2. *"QuickSelect version?"* → Partition by distance around a pivot until the kth boundary, like Q48.

**Common mistakes:**
- Using sqrt (slower, float precision).
- Using a min-heap of size k (you'd need to peek the wrong end to evict).

---

### Q52: IPO
**Company:** Amazon, Google, Two Sigma
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
You have starting capital `w` and can do at most `k` projects. Each project `i` has `capital[i]` (required to start) and `profit[i]` (added to capital on completion). Maximize final capital.

**What interviewer is testing:**
The **two-heap greedy**: a min-heap by required capital (to find affordable projects) and a max-heap by profit (to pick the best affordable one). Greedy choice: among all currently affordable projects, always take the most profitable.

**Ideal Answer:**

Sort projects by required capital (or use a min-heap by capital). Repeat k times: unlock all projects whose capital ≤ current `w` into a max-heap by profit, then take the most profitable. If none affordable, stop.

```java
public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
    int n = profits.length;
    // min-heap by capital required
    PriorityQueue<int[]> byCapital = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {capital, profit}
    for (int i = 0; i < n; i++) byCapital.offer(new int[]{capital[i], profits[i]});
    // max-heap by profit (among affordable)
    PriorityQueue<Integer> byProfit = new PriorityQueue<>(Collections.reverseOrder());

    for (int t = 0; t < k; t++) {
        while (!byCapital.isEmpty() && byCapital.peek()[0] <= w) {
            byProfit.offer(byCapital.poll()[1]);   // unlock affordable projects
        }
        if (byProfit.isEmpty()) break;             // nothing affordable -> stop
        w += byProfit.poll();                       // take the most profitable
    }
    return w;
}
```

**Why greedy works:** taking the most profitable affordable project never hurts — completing it only increases capital, which can only unlock *more* projects later. There's no benefit to deferring a profitable affordable project.

**Complexity:** O(n log n) time, O(n) space.

**Follow-up:** *"Why not just sort by profit?"* → A high-profit project may be unaffordable initially; you must unlock by capital as `w` grows.

**Common mistakes:**
- Picking by profit without checking affordability.
- Not re-unlocking newly-affordable projects after each completion.

---

### Q53: Smallest Range Covering Elements from K Lists
**Company:** Amazon, Google, LinkedIn
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `k` sorted lists, find the smallest range `[a, b]` that includes at least one number from each list.

**What interviewer is testing:**
A min-heap "merge" frontier: hold one element from each list; the range is `[heapMin, currentMax]`. Advance the list that holds the min to try to shrink the range. This is the K-way merge pattern with a twist.

**Ideal Answer:**

Push the first element of each list into a min-heap, tracking the maximum among them. The current candidate range is `[heap.min, curMax]`. Repeatedly pop the min (it's the limiting low end), and push the next element from that same list; update `curMax`. Stop when any list is exhausted (we can no longer cover that list).

```java
public int[] smallestRange(List<List<Integer>> nums) {
    // heap holds {value, listIndex, indexInList}, min by value
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    int curMax = Integer.MIN_VALUE;
    for (int i = 0; i < nums.size(); i++) {
        int v = nums.get(i).get(0);
        heap.offer(new int[]{v, i, 0});
        curMax = Math.max(curMax, v);
    }
    int[] best = {0, Integer.MAX_VALUE};        // smallest range so far
    while (true) {
        int[] top = heap.poll();
        int min = top[0], list = top[1], idx = top[2];
        if (curMax - min < best[1] - best[0]) {  // tighter range
            best[0] = min; best[1] = curMax;
        }
        if (idx + 1 == nums.get(list).size()) break;  // a list exhausted -> done
        int next = nums.get(list).get(idx + 1);
        curMax = Math.max(curMax, next);
        heap.offer(new int[]{next, list, idx + 1});
    }
    return best;
}
```

**Why advancing the min works:** the range must span from the smallest to the largest of the chosen elements. The only way to shrink the range is to raise the minimum, so we advance the list contributing the current min. When that list runs out, no smaller-min covering is possible.

**Complexity:** O(N log k) time where N is total elements, O(k) space.

**Follow-ups:**
1. *"Why stop when one list is exhausted?"* → We can no longer include a value from that list while advancing past its end; any further range can't cover all lists.

**Common mistakes:**
- Advancing the wrong list (must advance the one holding the min).
- Forgetting to track `curMax` as you push.

---

### Q54: Design Twitter
**Company:** Amazon, Twitter, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Design a simplified Twitter: `postTweet(userId, tweetId)`, `getNewsFeed(userId)` (10 most recent tweet IDs from the user and the people they follow, most recent first), `follow(followerId, followeeId)`, `unfollow(followerId, followeeId)`.

**What interviewer is testing:**
OOP modeling plus a **k-way merge via heap** to assemble the feed from multiple tweet timelines, ordered by a global timestamp.

**Ideal Answer:**

Each user has a list of `(timestamp, tweetId)`; a global counter provides monotonic timestamps. A follow-set per user. `getNewsFeed` merges the most recent tweets of the user and all followees using a max-heap by timestamp, pulling the top 10.

```java
class Twitter {
    private int timestamp = 0;
    private final Map<Integer, List<int[]>> tweets = new HashMap<>();   // user -> [time, tweetId]
    private final Map<Integer, Set<Integer>> following = new HashMap<>();

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, x -> new ArrayList<>())
              .add(new int[]{timestamp++, tweetId});
    }

    public List<Integer> getNewsFeed(int userId) {
        // max-heap by timestamp
        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> b[0] - a[0]);
        Set<Integer> users = new HashSet<>(following.getOrDefault(userId, Set.of()));
        users.add(userId);                          // include self
        for (int u : users) {
            List<int[]> ts = tweets.get(u);
            if (ts != null) {
                // only the last few matter; push them
                for (int i = Math.max(0, ts.size() - 10); i < ts.size(); i++)
                    heap.offer(ts.get(i));
            }
        }
        List<Integer> feed = new ArrayList<>();
        while (!heap.isEmpty() && feed.size() < 10) feed.add(heap.poll()[1]);
        return feed;
    }

    public void follow(int followerId, int followeeId) {
        following.computeIfAbsent(followerId, x -> new HashSet<>()).add(followeeId);
    }
    public void unfollow(int followerId, int followeeId) {
        Set<Integer> f = following.get(followerId);
        if (f != null) f.remove(followeeId);
    }
}
```

**Complexity:** post O(1); feed O(F·10·log(F·10)) where F is number followed; follow/unfollow O(1).

**Follow-up questions the interviewer will ask:**
1. *"Scale to millions of followers (celebrity problem)?"* → Fan-out-on-write is too expensive for celebrities; use fan-out-on-read (pull) for high-follower accounts and push for normal accounts — a hybrid. (System design file territory.)
2. *"Why a heap and not sort all tweets?"* → You only need the top 10; a heap over the per-user recent tweets avoids sorting everything.

**Common mistakes:**
- Forgetting to include the user's own tweets in their feed.
- Storing tweets without timestamps (can't order across users).

---

### Q55: Sliding Window Median
**Company:** Amazon, Google, Two Sigma
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an array and window size `k`, return the median of each window as it slides.

**What interviewer is testing:**
The two-heap median (Q49) extended with **lazy deletion** to remove the element leaving the window — because heaps don't support arbitrary removal in O(log n) directly.

**Ideal Answer:**

Use the two-heap median structure. When the window slides, you must remove the outgoing element. Standard heaps can't remove arbitrary elements efficiently, so use **lazy deletion**: keep a `delayed` map of element → pending-deletion count; only physically remove from a heap top when it matches a delayed element. Track logical sizes separately.

```java
public double[] medianSlidingWindow(int[] nums, int k) {
    DualHeap dh = new DualHeap(k);
    int n = nums.length;
    double[] res = new double[n - k + 1];
    for (int i = 0; i < n; i++) {
        dh.insert(nums[i]);
        if (i >= k - 1) {
            res[i - k + 1] = dh.getMedian();
            dh.erase(nums[i - k + 1]);     // remove the element leaving the window
        }
    }
    return res;
}

class DualHeap {
    private final PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> hi = new PriorityQueue<>();                            // min-heap
    private final Map<Integer, Integer> delayed = new HashMap<>();   // value -> pending deletions
    private int loSize = 0, hiSize = 0;
    private final int k;
    DualHeap(int k) { this.k = k; }

    void insert(int num) {
        if (lo.isEmpty() || num <= lo.peek()) { lo.offer(num); loSize++; }
        else { hi.offer(num); hiSize++; }
        balance();
    }
    void erase(int num) {
        delayed.merge(num, 1, Integer::sum);    // mark for lazy deletion
        if (num <= lo.peek()) { loSize--; if (num == lo.peek()) prune(lo); }
        else { hiSize--; if (num == hi.peek()) prune(hi); }
        balance();
    }
    double getMedian() {
        return (k % 2 == 1) ? lo.peek() : ((double) lo.peek() + hi.peek()) / 2.0;
    }
    // keep loSize == hiSize or loSize == hiSize + 1
    private void balance() {
        if (loSize > hiSize + 1) { hi.offer(lo.poll()); loSize--; hiSize++; prune(lo); }
        else if (loSize < hiSize) { lo.offer(hi.poll()); loSize++; hiSize--; prune(hi); }
    }
    // physically discard delayed elements sitting at the top
    private void prune(PriorityQueue<Integer> heap) {
        while (!heap.isEmpty()) {
            int top = heap.peek();
            Integer cnt = delayed.get(top);
            if (cnt == null) break;
            if (cnt == 1) delayed.remove(top); else delayed.put(top, cnt - 1);
            heap.poll();
        }
    }
}
```

**Why lazy deletion:** Removing an arbitrary value from a binary heap is O(n). Instead, mark it deleted and only actually pop it when it bubbles to the top. We track logical sizes (`loSize`, `hiSize`) ignoring delayed elements so the median is computed from the true window contents.

**Complexity:** O(n log k) time, O(k) space.

**Follow-up:** *"Alternative without lazy deletion?"* → A TreeMap (multiset) or two indexed-balanced structures; or an order-statistics tree. The two-heap + lazy deletion is the standard interview answer.

**Common mistakes:**
- Comparing `num <= lo.peek()` after the heap top was already pruned — prune carefully.
- Tracking physical heap sizes instead of logical sizes (delayed elements corrupt the median).
- Integer overflow on median average (cast to double).

---

### Q56: Ugly Number II
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
An ugly number has only 2, 3, 5 as prime factors (1 counts). Return the nth ugly number.

**What interviewer is testing:**
The min-heap generation (simple but with dedup), or the optimal three-pointer DP.

**Ideal Answer:**

*Min-heap:* pop the smallest ugly number, push `×2, ×3, ×5`, dedup with a set, repeat n times.

```java
public int nthUglyNumberHeap(int n) {
    PriorityQueue<Long> heap = new PriorityQueue<>();
    Set<Long> seen = new HashSet<>();
    heap.offer(1L); seen.add(1L);
    long ugly = 1;
    int[] primes = {2, 3, 5};
    for (int i = 0; i < n; i++) {
        ugly = heap.poll();
        for (int p : primes) {
            long next = ugly * p;
            if (seen.add(next)) heap.offer(next);  // add returns false if duplicate
        }
    }
    return (int) ugly;
}
```

*Optimal three-pointer DP (O(n), no heap):* maintain pointers `i2, i3, i5` into the list of ugly numbers; the next ugly is the min of `dp[i2]*2, dp[i3]*3, dp[i5]*5`; advance every pointer that produced the min (to dedup).

```java
public int nthUglyNumber(int n) {
    int[] dp = new int[n];
    dp[0] = 1;
    int i2 = 0, i3 = 0, i5 = 0;
    for (int i = 1; i < n; i++) {
        int next2 = dp[i2] * 2, next3 = dp[i3] * 3, next5 = dp[i5] * 5;
        int next = Math.min(next2, Math.min(next3, next5));
        dp[i] = next;
        if (next == next2) i2++;   // advance ALL that match -> dedup
        if (next == next3) i3++;
        if (next == next5) i5++;
    }
    return dp[n - 1];
}
```

**Complexity:** Heap O(n log n); DP O(n) time, O(n) space.

**Follow-up:** *"Why advance all matching pointers?"* → To skip duplicates like 6 (=2×3=3×2) which appear via multiple pointers.

**Common mistakes:**
- Heap overflow without `long`.
- DP advancing only one pointer on a tie → duplicates.

---

### Q57: Kth Largest Element in a Stream
**Company:** Amazon, Facebook
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Design a class initialized with `k` and an array. `add(val)` returns the kth largest element after adding `val` (the stream grows).

**What interviewer is testing:**
A fixed-size-k min-heap maintained across calls — the heap top is always the kth largest.

**Ideal Answer:**

```java
class KthLargest {
    private final PriorityQueue<Integer> heap = new PriorityQueue<>(); // min-heap of size k
    private final int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int x : nums) add(x);
    }
    public int add(int val) {
        heap.offer(val);
        if (heap.size() > k) heap.poll();   // keep only the k largest
        return heap.peek();                  // kth largest = smallest of the k largest
    }
}
```

**Complexity:** `add` O(log k), O(k) space.

**Follow-up:** *"Why min-heap not max-heap?"* → The min-heap of size k keeps exactly the k largest; its smallest (top) is the kth largest.

**Common mistakes:**
- Keeping all elements and sorting on each add (O(n log n) per call).

---

### Q58: Sort Characters by Frequency
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Sort a string's characters in decreasing order of frequency. Return the rearranged string.

**What interviewer is testing:**
Frequency count + ordering by count (heap, or bucket sort for O(n)).

**Ideal Answer:**

*Max-heap:*

```java
public String frequencySort(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    PriorityQueue<Character> heap = new PriorityQueue<>((a, b) -> freq.get(b) - freq.get(a));
    heap.addAll(freq.keySet());
    StringBuilder sb = new StringBuilder();
    while (!heap.isEmpty()) {
        char c = heap.poll();
        for (int i = 0; i < freq.get(c); i++) sb.append(c);
    }
    return sb.toString();
}
```

*Bucket sort (O(n)):* index buckets by frequency (0..n), then read from high frequency to low.

**Complexity:** Heap O(n + k log k); bucket O(n).

**Follow-up:** *"Beat O(n log n)?"* → Bucket sort by frequency is O(n) since frequencies are bounded by n.

**Common mistakes:**
- Sorting individual characters instead of grouping by frequency.

---

### Q59: Merge K Sorted Lists (heap framing)
**Company:** Amazon, Facebook, Google, Microsoft, LinkedIn
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Merge `k` sorted linked lists into one sorted list.

**What interviewer is testing:**
The k-way merge via a min-heap of size k, plus awareness of the divide-and-conquer alternative with the same complexity.

**Ideal Answer:**

*Min-heap of list heads:* push the head of each list. Repeatedly poll the smallest, append it to the result, and push its `next`. The heap holds at most k nodes.

```java
class ListNode { int val; ListNode next; ListNode(int v){val=v;} }

public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> heap = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) if (node != null) heap.offer(node);
    ListNode dummy = new ListNode(0), tail = dummy;
    while (!heap.isEmpty()) {
        ListNode smallest = heap.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) heap.offer(smallest.next);  // push the next node
    }
    return dummy.next;
}
```

**Why a heap:** at each step the next output element is the minimum among the k current list fronts — exactly what a min-heap of size k gives in O(log k).

**Complexity:** O(N log k) time (N total nodes, each pushed/popped once at O(log k)), O(k) space.

*Divide-and-conquer alternative:* pairwise merge lists (merge list 0&1, 2&3, …), halving the count each round. Also O(N log k), O(1) extra (besides recursion).

**Follow-up questions the interviewer will ask:**
1. *"Heap vs divide-and-conquer?"* → Same O(N log k). D&C has better constant factors and O(1) heap overhead; heap is simpler to reason about and works for streaming.
2. *"Merge K sorted *arrays*?"* → Same heap idea storing `(value, arrayIdx, elemIdx)`.

**Common mistakes:**
- Pushing all N nodes upfront (O(N) heap, O(N log N)) instead of a rolling size-k heap.
- Forgetting the dummy head, complicating the first append.

---

### Q60: Task Scheduler
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given task labels and a cooldown `n` (same task must be ≥ n intervals apart), return the minimum total intervals (including idles) to finish all tasks.

**What interviewer is testing:**
Either a greedy max-heap simulation with a cooldown queue, or the elegant O(1) math formula based on the most frequent task.

**Ideal Answer:**

*Math formula (cleanest):* Let `maxFreq` be the highest task count and `countMax` the number of tasks having that count. The schedule is dominated by the most frequent task forming `(maxFreq - 1)` gaps of size `(n + 1)`, plus the `countMax` tasks that finish last. Idle slots may be filled by other tasks, so the answer is `max(totalTasks, (maxFreq - 1) * (n + 1) + countMax)`.

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    int maxFreq = 0;
    for (int f : freq) maxFreq = Math.max(maxFreq, f);
    int countMax = 0;
    for (int f : freq) if (f == maxFreq) countMax++;
    int slots = (maxFreq - 1) * (n + 1) + countMax;
    return Math.max(tasks.length, slots);   // can't be less than total tasks
}
```

*Heap simulation (when asked for the actual schedule):* a max-heap by remaining count; each "cycle" of `n+1` slots pops up to `n+1` tasks, decrements, and re-adds those still remaining after the cooldown.

**Dry run** — tasks `[A,A,A,B,B,B]`, n=2: maxFreq=3 (A and B), countMax=2. slots = (3-1)*(2+1)+2 = 6+2 = 8. max(6,8)=8 → e.g. `A B _ A B _ A B`. ✅

**Complexity:** Formula O(T) time, O(1) space; heap simulation O(T log 26) ≈ O(T).

**Follow-up questions the interviewer will ask:**
1. *"Why `max(totalTasks, slots)`?"* → If there are many distinct tasks, there are no idle gaps to fill and the answer is simply the number of tasks; the formula only binds when one task dominates.
2. *"Output the actual ordering?"* → Use the heap simulation; the formula only gives the count.

**Common mistakes:**
- Forgetting the `max(totalTasks, ...)` clamp (fails when tasks are diverse, e.g., n small and many distinct labels).
- Miscomputing `countMax` (the number of maximally frequent tasks tacked on at the end).

---

# TRIES

A trie (prefix tree) stores strings by sharing common prefixes along a tree of character edges. The trigger phrases: **"prefix," "autocomplete," "dictionary of words," "search with wildcards," "many words share prefixes," "maximum XOR" (binary trie).** A trie turns "does any word start with this prefix" into an O(prefix length) walk, independent of the dictionary size. The standard node holds an array of 26 children (for lowercase English) plus an `isWord` flag.

---

### Q61: Implement Trie (Prefix Tree)
**Company:** Amazon, Google, Microsoft, Facebook, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Implement a trie with `insert(word)`, `search(word)` (exact word present), and `startsWith(prefix)` (any word with this prefix present).

**What interviewer is testing:**
The fundamental data structure: node design, the walk-and-create insert, and the distinction between `search` (needs `isEnd`) and `startsWith` (just needs the path to exist).

**Ideal Answer:**

```java
class Trie {
    private final TrieNode root = new TrieNode();

    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;          // mark the end of a complete word
    }

    public boolean search(String word) {
        TrieNode node = walk(word);
        return node != null && node.isEnd;   // must be a full word
    }

    public boolean startsWith(String prefix) {
        return walk(prefix) != null;          // path existing is enough
    }

    // walk down following the characters; return the end node or null
    private TrieNode walk(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

**Dry run** — insert "app", "apple"; search("app")→walk to 'p' node which has isEnd=true→true; search("ap")→walk to 'p' (first), isEnd=false→false; startsWith("ap")→path exists→true. ✅

**Complexity:** insert/search/startsWith all O(L) where L is the word length. Space O(total characters × 26) worst.

**Pattern:** The walk-and-create / walk-and-check is the basis for all trie problems.

**Follow-up questions the interviewer will ask:**
1. *"26-array vs HashMap children?"* → Array is O(1) and cache-friendly but wastes space for sparse alphabets; a `HashMap<Character, TrieNode>` is space-efficient for large/unicode alphabets.
2. *"Implement delete?"* → Walk down; if the word exists, unset `isEnd`; then prune nodes bottom-up that have no children and aren't word-ends.
3. *"Count words with a given prefix?"* → Store a `count` per node, incremented on insert along the path.

**Common mistakes:**
- Conflating `search` and `startsWith` (forgetting the `isEnd` check in `search`).
- Not creating missing child nodes during insert.

---

### Q62: Design Add and Search Words Data Structure
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Support `addWord(word)` and `search(word)` where `word` in search may contain `.` matching any single character.

**What interviewer is testing:**
Trie + DFS backtracking on the wildcard — at a `.`, branch into all existing children.

**Ideal Answer:**

Insert like a normal trie. For search, walk normally on real characters; on a `.`, recursively try every non-null child.

```java
class WordDictionary {
    private final TrieNode root = new TrieNode();
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    public void addWord(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }
    private boolean dfs(String word, int i, TrieNode node) {
        if (node == null) return false;
        if (i == word.length()) return node.isEnd;     // matched all chars
        char c = word.charAt(i);
        if (c == '.') {                                  // wildcard: try every child
            for (TrieNode child : node.children) {
                if (dfs(word, i + 1, child)) return true;
            }
            return false;
        } else {
            return dfs(word, i + 1, node.children[c - 'a']);
        }
    }
}
```

**Dry run** — add "bad","dad","mad"; search("..d") → at root, '.' tries b,d,m; then '.' tries the next layer; final 'd' matches → true. ✅

**Complexity:** addWord O(L). search O(L) with no dots, but O(26^d · L) worst with `d` dots (branching). Space O(total chars × 26).

**Follow-ups:**
1. *"Worst-case search with all dots?"* → Explores the whole trie; bounded by the number of nodes.
2. *"Optimize many wildcard queries?"* → Suffix automaton or precomputed structures; usually overkill in interviews.

**Common mistakes:**
- At a `.`, only trying the first child instead of all.
- Forgetting the `node == null` guard when descending a missing real character.

---

### Q63: Word Search II
**Company:** Amazon, Microsoft, Google, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given an `m×n` board of letters and a list of words, return all words that can be formed by sequentially adjacent cells (up/down/left/right), each cell used at most once per word.

**What interviewer is testing:**
The killer combination: **Trie + DFS backtracking on the grid.** Building a trie of all words lets a single DFS from each cell search for *all* words simultaneously, pruning dead prefixes — far better than running Word Search I per word.

**Ideal Answer:**

Build a trie from the word list. DFS from every cell, walking the trie in lockstep with the board path. When a trie node's `word` field is non-null, we've found a word — add it. Mark visited cells (temporarily mutate the board), backtrack after. Key optimization: store the full word at terminal nodes (no need to reconstruct), and **prune** leaf trie nodes after finding to speed up.

```java
public List<String> findWords(char[][] board, String[] words) {
    TrieNode root = buildTrie(words);
    List<String> res = new ArrayList<>();
    int m = board.length, n = board[0].length;
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            dfs(board, r, c, root, res);
    return res;
}

private void dfs(char[][] board, int r, int c, TrieNode node, List<String> res) {
    int m = board.length, n = board[0].length;
    if (r < 0 || r >= m || c < 0 || c >= n) return;
    char ch = board[r][c];
    if (ch == '#') return;                       // already used in this path
    TrieNode next = node.children[ch - 'a'];
    if (next == null) return;                    // prefix not in trie -> prune
    if (next.word != null) {                     // found a complete word
        res.add(next.word);
        next.word = null;                        // de-duplicate: don't add twice
    }
    board[r][c] = '#';                            // mark visited
    dfs(board, r + 1, c, next, res);
    dfs(board, r - 1, c, next, res);
    dfs(board, r, c + 1, next, res);
    dfs(board, r, c - 1, next, res);
    board[r][c] = ch;                             // backtrack (restore)
}

static class TrieNode {
    TrieNode[] children = new TrieNode[26];
    String word = null;     // store the whole word at terminal node
}
private TrieNode buildTrie(String[] words) {
    TrieNode root = new TrieNode();
    for (String w : words) {
        TrieNode node = root;
        for (char c : w.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.word = w;
    }
    return root;
}
```

**Why the trie beats per-word search:** Running Word Search I for each word is O(W · m·n·4^L). The trie shares prefixes so one DFS explores all words at once and abandons any path whose prefix isn't in the trie — dramatic pruning when words share prefixes.

**Complexity:** O(m·n·4^L) worst (L = max word length), with the trie pruning making it far faster in practice. Space O(total characters in words).

**Follow-up questions the interviewer will ask:**
1. *"Why store the word at the node instead of tracking a StringBuilder?"* → Avoids string-building on every step and makes adding the found word O(1).
2. *"Why null out `next.word` after finding?"* → Prevents the same word being added multiple times (it can be reachable via different paths).
3. *"Further optimizations?"* → Prune trie leaves that have no children after a word is found, shrinking the search tree.

**Common mistakes:**
- Running a separate DFS per word (misses the whole point — TLE).
- Forgetting to restore the board cell on backtrack.
- Adding duplicate words (no de-dup via nulling `word`).

---

### Q64: Longest Word in Dictionary
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given a list of words, find the longest word that can be **built one character at a time** by other words in the list (every prefix of the word must also be in the list). On ties, return the lexicographically smallest.

**What interviewer is testing:**
Trie + DFS only descending through nodes that are themselves complete words.

**Ideal Answer:**

Insert all words. DFS from the root; only continue into a child if that child is `isEnd` (a valid word prefix-by-prefix). Track the longest (lexicographically smallest on ties).

```java
public String longestWord(String[] words) {
    TrieNode root = new TrieNode();
    for (String w : words) insert(root, w);
    return dfs(root, new StringBuilder(), "");
}
private String dfs(TrieNode node, StringBuilder path, String best) {
    // update best: longer wins; equal length -> lexicographically smaller
    if (path.length() > best.length()
        || (path.length() == best.length() && path.toString().compareTo(best) < 0)) {
        best = path.toString();
    }
    for (int i = 0; i < 26; i++) {
        TrieNode child = node.children[i];
        if (child != null && child.isEnd) {     // only walk through valid words
            path.append((char) ('a' + i));
            best = dfs(child, path, best);
            path.deleteCharAt(path.length() - 1);
        }
    }
    return best;
}
static class TrieNode { TrieNode[] children = new TrieNode[26]; boolean isEnd = false; }
private void insert(TrieNode root, String w) {
    TrieNode node = root;
    for (char c : w.toCharArray()) {
        int idx = c - 'a';
        if (node.children[idx] == null) node.children[idx] = new TrieNode();
        node = node.children[idx];
    }
    node.isEnd = true;
}
```

**Why only descend through `isEnd` children:** the requirement is that every prefix is itself a word; a child that isn't a complete word breaks the chain.

**Complexity:** O(total characters) time and space.

**Follow-up:** *"Without a trie?"* → Sort words; use a set; greedily check each word's prefixes exist. The trie does this naturally.

**Common mistakes:**
- Not enforcing the lexicographically-smallest tiebreak.
- Descending into non-word children.

---

### Q65: Replace Words
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Given a dictionary of roots and a sentence, replace every word in the sentence with the **shortest root** that is a prefix of it (if any).

**What interviewer is testing:**
Trie lookup for the shortest matching prefix — stop walking at the first `isEnd`.

**Ideal Answer:**

Insert all roots. For each word, walk the trie; the first `isEnd` encountered is the shortest root prefix → replace.

```java
public String replaceWords(List<String> dictionary, String sentence) {
    TrieNode root = new TrieNode();
    for (String w : dictionary) insert(root, w);
    String[] words = sentence.split(" ");
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < words.length; i++) {
        if (i > 0) sb.append(" ");
        sb.append(shortestRoot(root, words[i]));
    }
    return sb.toString();
}
private String shortestRoot(TrieNode root, String word) {
    TrieNode node = root;
    StringBuilder prefix = new StringBuilder();
    for (char c : word.toCharArray()) {
        int idx = c - 'a';
        if (node.children[idx] == null) break;      // no root prefix
        prefix.append(c);
        node = node.children[idx];
        if (node.isEnd) return prefix.toString();    // shortest root found
    }
    return word;                                      // no root applies
}
static class TrieNode { TrieNode[] children = new TrieNode[26]; boolean isEnd = false; }
private void insert(TrieNode root, String w) {
    TrieNode node = root;
    for (char c : w.toCharArray()) {
        int idx = c - 'a';
        if (node.children[idx] == null) node.children[idx] = new TrieNode();
        node = node.children[idx];
    }
    node.isEnd = true;
}
```

**Complexity:** O(total root chars + sentence length).

**Follow-up:** *"Longest matching root instead?"* → Keep walking and remember the *last* `isEnd` seen.

**Common mistakes:**
- Returning the longest root instead of the shortest (stop at the first `isEnd`).

---

### Q66: Maximum XOR of Two Numbers in an Array
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an array of integers, find the maximum XOR of any two numbers.

**What interviewer is testing:**
A **binary trie** (bitwise trie). To maximize XOR, greedily prefer the opposite bit at each level. Insert each number's bits, then query for the best XOR partner.

**Ideal Answer:**

Build a trie keyed on bits from the most significant (say bit 31 or 30 for non-negative ints) down to 0. For each number, walk the trie choosing the *opposite* bit when available (XOR of differing bits is 1, maximizing the result), accumulating the XOR value.

```java
public int findMaximumXOR(int[] nums) {
    TrieNode root = new TrieNode();
    int maxXor = 0;
    int HIGH = 31;                        // top bit to consider (use 30 if all non-negative & small)
    for (int num : nums) {
        // insert num
        TrieNode node = root;
        for (int b = HIGH; b >= 0; b--) {
            int bit = (num >> b) & 1;
            if (node.children[bit] == null) node.children[bit] = new TrieNode();
            node = node.children[bit];
        }
        // query best partner for num
        TrieNode cur = root;
        int curXor = 0;
        for (int b = HIGH; b >= 0; b--) {
            int bit = (num >> b) & 1;
            int want = bit ^ 1;            // prefer the opposite bit
            if (cur.children[want] != null) {
                curXor |= (1 << b);        // this bit contributes 1 to XOR
                cur = cur.children[want];
            } else {
                cur = cur.children[bit];   // forced to take the same bit
            }
        }
        maxXor = Math.max(maxXor, curXor);
    }
    return maxXor;
}
static class TrieNode { TrieNode[] children = new TrieNode[2]; }
```

**Why greedy on the opposite bit works:** XOR is maximized by setting the highest bits to 1. Two bits XOR to 1 iff they differ, so at each level we greedily route to the opposite bit if such a number exists — locking in the highest possible value bit by bit.

**Dry run** — `[3,10,5,25,2,8]`: the max XOR is 5 ^ 25 = 28. The trie lets each number find its best opposite-bit partner. ✅

**Complexity:** O(n · 32) = O(n) time, O(n · 32) space.

**Follow-up questions the interviewer will ask:**
1. *"Why insert and query in the same loop?"* → It still finds the max because every pair is considered from the later-inserted side; alternatively insert all first, then query all.
2. *"Maximum XOR with a constraint (e.g., the other number ≤ some limit)?"* → "Maximum XOR With an Element From Array" — sort queries and insert numbers up to the limit (offline processing).

**Common mistakes:**
- Walking bits from least significant first (must go MSB→LSB to greedily fix high bits).
- Wrong HIGH bit (include enough bits for the max value).

---

### Q67: Map Sum Pairs
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
Implement `insert(key, val)` and `sum(prefix)` returning the sum of values of all keys starting with `prefix`. Inserting an existing key overwrites its value.

**What interviewer is testing:**
A trie augmented with prefix sums, and correct handling of value *updates* (must adjust by the delta, not double count).

**Ideal Answer:**

Store at each trie node the running sum of values of all keys passing through it. On insert, compute the delta (`newVal − oldVal`) and add it along the path. A side map tracks current key→value to compute deltas.

```java
class MapSum {
    private final TrieNode root = new TrieNode();
    private final Map<String, Integer> vals = new HashMap<>();   // key -> current value

    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        int sum = 0;        // sum of values of all keys passing through this node
    }

    public void insert(String key, int val) {
        int delta = val - vals.getOrDefault(key, 0);  // adjust by difference on overwrite
        vals.put(key, val);
        TrieNode node = root;
        for (char c : key.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
            node.sum += delta;                          // propagate the delta
        }
    }

    public int sum(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return 0;   // no keys with this prefix
            node = node.children[idx];
        }
        return node.sum;                                 // precomputed prefix sum
    }
}
```

**Dry run** — insert("apple",3): sums along a-p-p-l-e += 3. sum("ap") → walk to 'p', returns 3. insert("apple",2): delta = 2−3 = −1; sums adjust to 2. sum("ap") → 2. ✅

**Complexity:** insert O(L), sum O(prefix length). Space O(total characters).

**Follow-up:** *"Why the delta on overwrite?"* → Re-inserting "apple" must replace, not add; the delta keeps every node's `sum` consistent.

**Common mistakes:**
- Adding `val` directly on every insert (double-counts on overwrite).
- Recomputing the prefix sum by traversing all descendants (defeats the augmentation).

---

# GRAPHS

The hardest part of graph problems is usually **recognizing that it's a graph problem** and choosing the right algorithm. The decision tree:

- **Reachability / connected components / "fewest steps in an unweighted graph"** → BFS or DFS (BFS for shortest unweighted path).
- **"Process in dependency order" / "is there a cycle in a DAG"** → Topological sort (Kahn's BFS or DFS).
- **Shortest path, non-negative weights** → Dijkstra.
- **Shortest path with negative edges / detect negative cycle** → Bellman-Ford.
- **All-pairs shortest paths, small graph** → Floyd-Warshall.
- **Dynamic connectivity / "are these in the same group" / cycle in undirected** → Union-Find (DSU).
- **Minimum spanning tree** → Kruskal (sort edges + DSU) or Prim (heap).

Master the four templates at the end of this file (BFS, DFS, topo sort, union-find) and most graph questions become a thin wrapper around one of them.

---

### Q68: Number of Islands
**Company:** Amazon, Facebook, Google, Microsoft, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a 2D grid of `'1'` (land) and `'0'` (water), count the number of islands (land connected horizontally/vertically).

**What interviewer is testing:**
Grid traversal as graph traversal. You should be able to give **all three**: DFS flood fill, BFS flood fill, and Union-Find — and discuss trade-offs.

**Ideal Answer:**

*Approach 1 — DFS flood fill:* scan the grid; on each unvisited `'1'`, increment the count and DFS to sink the entire island (mark cells visited).

```java
public int numIslands(char[][] grid) {
    int count = 0, m = grid.length, n = grid[0].length;
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == '1') {
                count++;
                dfs(grid, r, c);     // sink this whole island
            }
        }
    }
    return count;
}
private void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1')
        return;
    grid[r][c] = '0';                // mark visited (sink it)
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}
```

*Approach 2 — BFS flood fill:* same idea but use a queue (avoids deep recursion stack overflow on huge grids).

```java
public int numIslandsBFS(char[][] grid) {
    int count = 0, m = grid.length, n = grid[0].length;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == '1') {
                count++;
                Queue<int[]> q = new LinkedList<>();
                q.offer(new int[]{r, c});
                grid[r][c] = '0';
                while (!q.isEmpty()) {
                    int[] cell = q.poll();
                    for (int[] d : dirs) {
                        int nr = cell[0] + d[0], nc = cell[1] + d[1];
                        if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == '1') {
                            grid[nr][nc] = '0';
                            q.offer(new int[]{nr, nc});
                        }
                    }
                }
            }
        }
    }
    return count;
}
```

*Approach 3 — Union-Find:* union adjacent land cells; the number of distinct sets among land cells is the island count.

```java
public int numIslandsUF(char[][] grid) {
    int m = grid.length, n = grid[0].length;
    UnionFind uf = new UnionFind(grid);
    for (int r = 0; r < m; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == '1') {
                if (r + 1 < m && grid[r+1][c] == '1') uf.union(r*n+c, (r+1)*n+c);
                if (c + 1 < n && grid[r][c+1] == '1') uf.union(r*n+c, r*n+c+1);
            }
        }
    }
    return uf.count;
}
class UnionFind {
    int[] parent, rank; int count = 0;
    UnionFind(char[][] grid) {
        int m = grid.length, n = grid[0].length;
        parent = new int[m*n]; rank = new int[m*n];
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                if (grid[r][c] == '1') { parent[r*n+c] = r*n+c; count++; }
    }
    int find(int x) { while (x != parent[x]) { parent[x] = parent[parent[x]]; x = parent[x]; } return x; }
    void union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return;
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        count--;                            // two islands merged into one
    }
}
```

**Complexity:** All three O(m·n). DFS/BFS O(m·n) space worst (recursion/queue). Union-Find O(m·n·α) ≈ O(m·n) with path compression + union by rank.

**Follow-up questions the interviewer will ask:**
1. *"Which approach for a streaming version (cells added one at a time)?"* → Union-Find shines — "Number of Islands II" adds land incrementally; recomputing with DFS each time is wasteful.
2. *"Diagonal connections count?"* → Add the 4 diagonal directions (8 total).
3. *"Why might BFS be preferred over DFS?"* → DFS recursion can stack-overflow on a giant single island (millions of cells); BFS uses an explicit queue.
4. *"Don't want to mutate the input?"* → Use a separate `visited[][]` boolean array.

**Common mistakes:**
- Forgetting to mark cells visited → infinite recursion / overcount.
- DFS stack overflow on very large grids (mention BFS).
- Off-by-one in bounds checks.

---

### Q69: Clone Graph
**Company:** Amazon, Facebook, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a reference to a node in a connected undirected graph, return a deep copy (clone) of the entire graph. Each node has a value and a list of neighbors.

**What interviewer is testing:**
Traversal (DFS or BFS) plus a `visited` map from original node → cloned node to handle cycles and avoid infinite loops / duplicate clones.

**Ideal Answer:**

Maintain a map `original -> clone`. DFS: if the node is already cloned, return its clone (handles cycles); otherwise create the clone, register it *before* recursing into neighbors (critical to avoid infinite loops), then clone each neighbor.

```java
class Node { int val; List<Node> neighbors; Node(int v){val=v;neighbors=new ArrayList<>();} }

public Node cloneGraph(Node node) {
    if (node == null) return null;
    Map<Node, Node> cloned = new HashMap<>();   // original -> clone
    return dfs(node, cloned);
}
private Node dfs(Node node, Map<Node, Node> cloned) {
    if (cloned.containsKey(node)) return cloned.get(node);   // already done -> reuse
    Node copy = new Node(node.val);
    cloned.put(node, copy);                                   // register BEFORE recursing
    for (Node nei : node.neighbors) {
        copy.neighbors.add(dfs(nei, cloned));                 // clone neighbors
    }
    return copy;
}
```

*BFS alternative:* clone node, queue originals; for each, clone any un-cloned neighbor and wire edges.

**Why register before recursing:** in a cyclic graph, a neighbor may point back to this node; if we haven't registered the clone yet, we'd recurse forever. Putting it in the map first breaks the cycle.

**Complexity:** O(V + E) time, O(V) space (map + recursion).

**Follow-ups:**
1. *"Disconnected graph?"* → The problem gives a connected graph; for disconnected you'd iterate over all nodes.
2. *"Why a map not a set for visited?"* → You need the *mapping* to wire neighbors to the correct clones, not just "seen."

**Common mistakes:**
- Registering the clone after recursing (infinite loop on cycles).
- Creating duplicate clones of the same node (no map).

---

### Q70: Pacific Atlantic Water Flow
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a grid of heights, water flows from a cell to adjacent cells of equal or lower height. The Pacific touches the top and left edges; the Atlantic touches the bottom and right edges. Return all cells from which water can reach **both** oceans.

**What interviewer is testing:**
The "reverse the flow" insight: instead of checking from each cell whether it reaches an ocean (expensive), start *from the ocean borders* and find all cells that can flow *into* each ocean (climbing uphill on the reversed graph). The answer is the intersection.

**Ideal Answer:**

Run two multi-source DFS/BFS: one from all Pacific-border cells, one from all Atlantic-border cells. From a border cell, move to a neighbor if the neighbor's height is ≥ current (reverse of "flows downhill"). Mark reachable cells. A cell in both sets reaches both oceans.

```java
public List<List<Integer>> pacificAtlantic(int[][] heights) {
    int m = heights.length, n = heights[0].length;
    boolean[][] pacific = new boolean[m][n];
    boolean[][] atlantic = new boolean[m][n];
    for (int r = 0; r < m; r++) {           // left col -> Pacific, right col -> Atlantic
        dfs(heights, r, 0, pacific);
        dfs(heights, r, n - 1, atlantic);
    }
    for (int c = 0; c < n; c++) {           // top row -> Pacific, bottom row -> Atlantic
        dfs(heights, 0, c, pacific);
        dfs(heights, m - 1, c, atlantic);
    }
    List<List<Integer>> res = new ArrayList<>();
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            if (pacific[r][c] && atlantic[r][c]) res.add(List.of(r, c));
    return res;
}
private void dfs(int[][] h, int r, int c, boolean[][] reach) {
    reach[r][c] = true;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    for (int[] d : dirs) {
        int nr = r + d[0], nc = c + d[1];
        if (nr >= 0 && nr < h.length && nc >= 0 && nc < h[0].length
            && !reach[nr][nc] && h[nr][nc] >= h[r][c]) {   // climb uphill (reversed flow)
            dfs(h, nr, nc, reach);
        }
    }
}
```

**Why reverse?** Checking forward from every cell is O((m·n)²). Starting from the borders and flooding inward where height is non-decreasing finds all contributing cells in O(m·n).

**Complexity:** O(m·n) time and space.

**Follow-ups:**
1. *"Why `>=` not `>`?"* → Water flows to equal-or-lower cells forward, so reversed we climb to equal-or-higher.

**Common mistakes:**
- Searching forward from each cell (quadratic).
- Wrong comparison direction in the reversed traversal.

---

### Q71: Surrounded Regions
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a board of `'X'` and `'O'`, capture all regions of `'O'` that are fully surrounded by `'X'` (flip them to `'X'`). `'O'`s connected to a border are NOT captured.

**What interviewer is testing:**
The complement insight: mark all border-connected `'O'`s as safe first (flood from borders), then flip everything that remains.

**Ideal Answer:**

From every border `'O'`, DFS/BFS marking the connected region as safe (temporarily `'#'`). Then sweep: every remaining `'O'` is surrounded → flip to `'X'`; restore `'#'` back to `'O'`.

```java
public void solve(char[][] board) {
    if (board.length == 0) return;
    int m = board.length, n = board[0].length;
    // mark border-connected 'O's as safe
    for (int r = 0; r < m; r++) { mark(board, r, 0); mark(board, r, n - 1); }
    for (int c = 0; c < n; c++) { mark(board, 0, c); mark(board, m - 1, c); }
    // flip the rest
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++) {
            if (board[r][c] == 'O') board[r][c] = 'X';   // surrounded -> capture
            else if (board[r][c] == '#') board[r][c] = 'O'; // safe -> restore
        }
}
private void mark(char[][] b, int r, int c) {
    if (r < 0 || r >= b.length || c < 0 || c >= b[0].length || b[r][c] != 'O') return;
    b[r][c] = '#';                       // safe marker
    mark(b, r + 1, c); mark(b, r - 1, c);
    mark(b, r, c + 1); mark(b, r, c - 1);
}
```

**Complexity:** O(m·n) time and space.

**Follow-up:** *"Union-Find version?"* → Union border `'O'`s to a virtual "safe" node; any `'O'` not connected to it gets flipped.

**Common mistakes:**
- Trying to detect "surrounded" directly (hard); the complement (border-connected) is far simpler.
- Forgetting to restore the `'#'` markers.

---

### Q72: 01 Matrix
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a matrix of 0s and 1s, return a matrix where each cell holds the distance to the nearest 0.

**What interviewer is testing:**
**Multi-source BFS** — seed the queue with *all* the 0 cells at once, then BFS outward; the first time you reach a cell is its shortest distance.

**Ideal Answer:**

Initialize distance 0 for all zero cells and put them all in the queue. BFS layer by layer; each unvisited neighbor gets `dist + 1`. Because all sources start simultaneously, the BFS naturally computes the minimum distance to *any* zero.

```java
public int[][] updateMatrix(int[][] mat) {
    int m = mat.length, n = mat[0].length;
    int[][] dist = new int[m][n];
    Queue<int[]> q = new LinkedList<>();
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++) {
            if (mat[r][c] == 0) q.offer(new int[]{r, c});
            else dist[r][c] = -1;          // -1 = not yet visited
        }
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!q.isEmpty()) {
        int[] cell = q.poll();
        for (int[] d : dirs) {
            int nr = cell[0] + d[0], nc = cell[1] + d[1];
            if (nr >= 0 && nr < m && nc >= 0 && nc < n && dist[nr][nc] == -1) {
                dist[nr][nc] = dist[cell[0]][cell[1]] + 1;   // one layer further
                q.offer(new int[]{nr, nc});
            }
        }
    }
    return dist;
}
```

**Why multi-source?** A single BFS from every 1 would be O((m·n)²). Seeding all 0s at once and expanding outward gives every 1 its nearest-0 distance in one O(m·n) sweep.

**Complexity:** O(m·n) time and space.

**Pattern:** Multi-source BFS — the same template as Rotting Oranges (Q73) and "walls and gates."

**Follow-up:** *"DP alternative?"* → Two passes (top-left to bottom-right, then bottom-right to top-left) taking min of neighbors + 1. Also O(m·n).

**Common mistakes:**
- BFS from each 1 separately (quadratic).
- Not marking the initial 0 cells as visited / distance 0.

---

### Q73: Rotting Oranges
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
A grid has `0` (empty), `1` (fresh orange), `2` (rotten). Each minute, a rotten orange rots its 4-adjacent fresh neighbors. Return the minutes until no fresh orange remains, or `-1` if impossible.

**What interviewer is testing:**
Multi-source BFS where the BFS *level* equals the elapsed time. Plus the impossibility check (fresh oranges left unreachable).

**Ideal Answer:**

Seed the queue with all initially rotten oranges. BFS level by level; each level is one minute. Count fresh oranges; decrement as they rot. If any fresh remain at the end, return -1.

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Queue<int[]> q = new LinkedList<>();
    int fresh = 0;
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 2) q.offer(new int[]{r, c});
            else if (grid[r][c] == 1) fresh++;
        }
    if (fresh == 0) return 0;                 // nothing to rot
    int minutes = 0;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!q.isEmpty() && fresh > 0) {
        minutes++;
        int size = q.size();
        for (int i = 0; i < size; i++) {       // process one minute's worth
            int[] cell = q.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;          // rot it
                    fresh--;
                    q.offer(new int[]{nr, nc});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;          // leftover fresh -> impossible
}
```

**Dry run** — `[[2,1,1],[1,1,0],[0,1,1]]`: minute 1 rots neighbors of (0,0); propagates; total 4 minutes; the bottom-right is reachable → 4. If a fresh orange were isolated → -1. ✅

**Complexity:** O(m·n) time and space.

**Follow-up questions the interviewer will ask:**
1. *"Why count fresh oranges?"* → To detect impossibility (unreachable fresh) and to stop early.
2. *"Why is the BFS level the answer?"* → All rotten oranges spread simultaneously; each BFS layer is exactly one minute of simultaneous spreading.

**Common mistakes:**
- Not handling the case of zero fresh oranges (answer 0, not the minutes loop).
- Forgetting the level-size snapshot, mixing minutes.

---

### Q74: Number of Connected Components in an Undirected Graph
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given `n` nodes labeled `0..n-1` and a list of edges, count the connected components.

**What interviewer is testing:**
Either DFS/BFS counting unvisited starts, or Union-Find (start with `n` components, decrement on each successful union).

**Ideal Answer:**

*Union-Find (clean):*

```java
public int countComponents(int n, int[][] edges) {
    int[] parent = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;
    int components = n;                          // each node its own component initially
    for (int[] e : edges) {
        int ra = find(parent, e[0]), rb = find(parent, e[1]);
        if (ra != rb) { parent[ra] = rb; components--; }  // merge -> one fewer
    }
    return components;
}
private int find(int[] parent, int x) {
    while (x != parent[x]) { parent[x] = parent[parent[x]]; x = parent[x]; }
    return x;
}
```

*DFS:* build adjacency list; for each unvisited node, DFS the whole component and increment the count.

**Complexity:** Union-Find O(n + E·α). DFS O(n + E).

**Follow-up:** *"DFS vs Union-Find here?"* → Union-Find is natural when edges arrive incrementally; DFS when you have the full adjacency list and may need to enumerate component members.

**Common mistakes:**
- Initializing `components` wrong (it starts at `n`).
- Forgetting path compression on large inputs.

---

### Q75: Number of Provinces
**Company:** Amazon, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Given an `n×n` adjacency matrix `isConnected` where `isConnected[i][j] == 1` means city i and j are directly connected, return the number of provinces (connected groups of cities).

**What interviewer is testing:**
The same connected-components count, but the input is an adjacency *matrix* — recognizing it's identical to Q74.

**Ideal Answer:**

```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    boolean[] visited = new boolean[n];
    int provinces = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            provinces++;
            dfs(isConnected, visited, i);
        }
    }
    return provinces;
}
private void dfs(int[][] g, boolean[] visited, int i) {
    visited[i] = true;
    for (int j = 0; j < g.length; j++) {
        if (g[i][j] == 1 && !visited[j]) dfs(g, visited, j);
    }
}
```

**Complexity:** O(n²) time (must scan the matrix), O(n) space.

**Follow-up:** *"Union-Find?"* → Union i,j whenever `isConnected[i][j]==1`; count distinct roots.

**Common mistakes:**
- Treating the matrix as directed (it's symmetric/undirected).

---

### Q76: Course Schedule
**Company:** Amazon, Facebook, Google, Microsoft, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
There are `numCourses` courses labeled `0..n-1`. `prerequisites[i] = [a, b]` means you must take `b` before `a`. Return whether you can finish all courses (i.e., the dependency graph has no cycle).

**What interviewer is testing:**
**Cycle detection in a directed graph** = "can this DAG be topologically ordered." Two canonical methods: Kahn's BFS (indegree) and DFS with a recursion-stack/color marking. A top FAANG question — be ready with both.

**Ideal Answer:**

*Kahn's algorithm (BFS topological sort):* compute indegrees; start with all 0-indegree nodes; repeatedly remove a node and decrement its neighbors' indegrees, enqueuing any that hit 0. If you process all `n` nodes, there's no cycle.

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];
    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);    // edge b -> a (take b before a)
        indegree[p[0]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (indegree[i] == 0) q.offer(i);
    int processed = 0;
    while (!q.isEmpty()) {
        int node = q.poll();
        processed++;
        for (int nei : adj.get(node)) {
            if (--indegree[nei] == 0) q.offer(nei);   // all prereqs satisfied
        }
    }
    return processed == numCourses;    // all processed => no cycle
}
```

*DFS with 3-color marking:* WHITE (unvisited), GRAY (in current recursion stack), BLACK (fully done). Finding a GRAY node during DFS means a back edge → cycle.

```java
public boolean canFinishDFS(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] p : prerequisites) adj.get(p[1]).add(p[0]);
    int[] state = new int[numCourses];   // 0=white, 1=gray (in stack), 2=black
    for (int i = 0; i < numCourses; i++) {
        if (state[i] == 0 && hasCycle(adj, state, i)) return false;
    }
    return true;
}
private boolean hasCycle(List<List<Integer>> adj, int[] state, int node) {
    state[node] = 1;                     // mark gray (on the recursion stack)
    for (int nei : adj.get(node)) {
        if (state[nei] == 1) return true;            // back edge -> cycle
        if (state[nei] == 0 && hasCycle(adj, state, nei)) return true;
    }
    state[node] = 2;                     // done -> black
    return false;
}
```

**Dry run (Kahn)** — `numCourses=2`, prereq `[[1,0]]`: indegree[1]=1, indegree[0]=0. Queue starts with 0; process 0, decrement indegree[1]→0, enqueue 1; process 1. processed=2=n → true. With a cycle `[[1,0],[0,1]]`, no node has indegree 0 → queue empty immediately → processed=0≠2 → false. ✅

**Why Kahn detects cycles:** nodes in a cycle never reach indegree 0 (each waits on another in the cycle), so they're never processed; `processed < n` flags the cycle.

**Complexity:** O(V + E) time, O(V + E) space, both approaches.

**Follow-up questions the interviewer will ask:**
1. *"Return the actual order (Course Schedule II)?"* → Q77; record the processing order in Kahn's, or reverse-postorder in DFS.
2. *"Kahn vs DFS — which to prefer?"* → Kahn's also yields a valid topological order and avoids recursion-stack limits; DFS is concise and naturally detects the cycle via the gray state. Both O(V+E).
3. *"Why 3 colors and not a simple visited set in DFS?"* → A node visited in a *previous* DFS (black) is fine; only a node on the *current* stack (gray) indicates a cycle. A 2-state visited can't distinguish these.

**Common mistakes:**
- Reversing the edge direction (`a -> b` vs `b -> a`) — `[a,b]` means b before a, so the edge is `b -> a`.
- DFS with a 2-state visited (false cycle reports on shared subgraphs).
- Not counting processed nodes in Kahn's (the cycle check).

---

### Q77: Course Schedule II
**Company:** Amazon, Facebook, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Return a valid order to take all courses, or an empty array if impossible (cycle).

**What interviewer is testing:**
Producing the topological order itself, not just detecting feasibility. Kahn's algorithm yields it directly.

**Ideal Answer:**

Same as Kahn's in Q76, but record nodes in the order they're dequeued. If fewer than `n` are recorded, a cycle exists → return empty.

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];
    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);
        indegree[p[0]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (indegree[i] == 0) q.offer(i);
    int[] order = new int[numCourses];
    int idx = 0;
    while (!q.isEmpty()) {
        int node = q.poll();
        order[idx++] = node;                     // append to topological order
        for (int nei : adj.get(node)) {
            if (--indegree[nei] == 0) q.offer(nei);
        }
    }
    return idx == numCourses ? order : new int[0];   // empty if cycle
}
```

**Complexity:** O(V + E) time and space.

**Follow-up:** *"DFS-based order?"* → Push nodes onto a stack on finishing (postorder); the reversed stack is a valid topo order.

**Common mistakes:**
- Returning a partial order when a cycle exists (must return empty).

---

### Q78: Alien Dictionary
**Company:** Amazon, Facebook, Google, Airbnb
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a list of words sorted lexicographically by the rules of an unknown alien language, derive a possible ordering of the alphabet. Return "" if no valid order exists.

**What interviewer is testing:**
Modeling order constraints as a directed graph (char → char) then topological sort. Plus the subtle invalid case where a longer word precedes its own prefix.

**Ideal Answer:**

Compare each adjacent pair of words; the first differing character gives an ordering edge `c1 -> c2`. Build the graph, then topologically sort the characters. Handle two edge cases: (a) if word A is longer than B and B is a prefix of A (e.g., "abc" before "ab"), it's invalid; (b) cycles mean no valid order.

```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> adj = new HashMap<>();
    Map<Character, Integer> indegree = new HashMap<>();
    for (String w : words)
        for (char c : w.toCharArray()) {
            adj.putIfAbsent(c, new HashSet<>());
            indegree.putIfAbsent(c, 0);
        }
    // build edges from adjacent word pairs
    for (int i = 0; i + 1 < words.length; i++) {
        String a = words[i], b = words[i + 1];
        int len = Math.min(a.length(), b.length());
        // invalid: a is longer and b is its prefix
        if (a.length() > b.length() && a.startsWith(b)) return "";
        for (int j = 0; j < len; j++) {
            char c1 = a.charAt(j), c2 = b.charAt(j);
            if (c1 != c2) {
                if (adj.get(c1).add(c2)) indegree.merge(c2, 1, Integer::sum);  // new edge
                break;                       // only the FIRST difference matters
            }
        }
    }
    // Kahn's topological sort
    Queue<Character> q = new LinkedList<>();
    for (char c : indegree.keySet()) if (indegree.get(c) == 0) q.offer(c);
    StringBuilder sb = new StringBuilder();
    while (!q.isEmpty()) {
        char c = q.poll();
        sb.append(c);
        for (char nei : adj.get(c)) {
            if (indegree.merge(nei, -1, Integer::sum) == 0) q.offer(nei);
        }
    }
    return sb.length() == indegree.size() ? sb.toString() : "";   // cycle => ""
}
```

**Dry run** — `["wrt","wrf","er","ett","rftt"]`: edges t→f (wrt vs wrf), w→e (wrt vs er), r→t (er vs ett), e→r (ett vs rftt). Topo sort → "wertf". ✅

**Why only the first difference?** Lexicographic comparison stops at the first differing character; subsequent characters carry no ordering information for this pair.

**Complexity:** O(total characters) time, O(unique chars + edges) space.

**Follow-up questions the interviewer will ask:**
1. *"What's the tricky invalid case?"* → "abc" listed before "ab": a longer word cannot precede its own prefix in a valid sort. You must explicitly check it.
2. *"Multiple valid orders?"* → Any valid topological order is acceptable; for the lexicographically smallest, use a priority queue in Kahn's.

**Common mistakes:**
- Comparing all characters of a pair instead of just the first difference.
- Missing the prefix-invalid case (returns a wrong order instead of "").
- Double-counting an edge in indegree (guard with `Set.add` returning true).

---

### Q79: Word Ladder
**Company:** Amazon, Facebook, Google, LinkedIn, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given `beginWord`, `endWord`, and a `wordList`, transform `beginWord` into `endWord` changing one letter at a time, where each intermediate word must be in `wordList`. Return the length of the shortest transformation sequence (number of words), or 0 if none.

**What interviewer is testing:**
Modeling words as graph nodes (edge = differ by one letter) and using **BFS for the shortest unweighted path.** The key optimization is generating neighbors efficiently (don't compare all pairs) — either by trying all single-character substitutions or via wildcard patterns.

**Ideal Answer:**

BFS from `beginWord`. For each word, generate all words differing by one letter (26 substitutions per position) and enqueue any that are in the dictionary and unvisited. The BFS depth gives the shortest sequence length. Use a set for O(1) membership and remove words as visited.

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return 0;       // unreachable
    Queue<String> q = new LinkedList<>();
    q.offer(beginWord);
    int level = 1;                                // count of words in the sequence
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            String word = q.poll();
            if (word.equals(endWord)) return level;
            char[] arr = word.toCharArray();
            for (int j = 0; j < arr.length; j++) {
                char original = arr[j];
                for (char c = 'a'; c <= 'z'; c++) {     // try every substitution
                    if (c == original) continue;
                    arr[j] = c;
                    String next = new String(arr);
                    if (dict.remove(next)) {            // in dict & not yet visited
                        q.offer(next);
                    }
                }
                arr[j] = original;                       // restore
            }
        }
        level++;
    }
    return 0;
}
```

**Why BFS not DFS:** the graph is unweighted; BFS finds the shortest path (fewest transformations) by exploring level by level. DFS could find a path but not guarantee the shortest.

**Why `dict.remove(next)` as the visited check:** removing on first encounter marks the word visited *and* checks membership in one O(1) operation — and BFS guarantees the first time we reach a word is via the shortest path.

**Dry run** — begin "hit", end "cog", dict `[hot,dot,dog,lot,log,cog]`: hit→hot (1+1)→dot/lot→dog/log→cog. Length 5: hit,hot,dot,dog,cog. ✅

**Complexity:** O(N · L · 26) where N = words, L = word length. The 26·L neighbor generation per word dominates. Space O(N · L).

**Follow-up questions the interviewer will ask:**
1. *"Bidirectional BFS optimization?"* → Search from both `beginWord` and `endWord`, expanding the smaller frontier each step; they meet in the middle, roughly halving the explored nodes (`b^(d/2) + b^(d/2)` vs `b^d`). A strong optimization for this problem.
2. *"Wildcard preprocessing?"* → Build a map from pattern (`h*t`) to words; neighbors share a pattern. Trades 26-substitution generation for a precomputed adjacency — useful when L is large.
3. *"Return the actual path (Word Ladder II)?"* → Q80; BFS to build a predecessor DAG, then DFS/backtrack to enumerate shortest paths.

**Common mistakes:**
- Using DFS (doesn't give the shortest length).
- Building the full graph by comparing all word pairs (O(N²·L) — too slow for large N).
- Forgetting to restore `arr[j]` after substitution.
- Off-by-one on the level count (the sequence length includes both endpoints).

---

### Q80: Word Ladder II
**Company:** Amazon, Facebook, Google
**Difficulty:** ⚫ Expert
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Return **all** shortest transformation sequences from `beginWord` to `endWord`.

**What interviewer is testing:**
The hardest variant: BFS to find shortest distances / build a predecessor graph, then DFS backtracking to reconstruct *all* shortest paths. Combining BFS layering with path reconstruction without exploding into non-shortest paths.

**Ideal Answer:**

Two phases:
1. **BFS by levels** building a `parents` map (for each word, which words on the previous level can reach it). Crucially, remove a level's words from the dictionary only *after* fully processing the level — otherwise you'd miss alternate shortest predecessors at the same level.
2. **DFS backtracking** from `endWord` up through `parents` to `beginWord`, collecting every shortest path.

```java
public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
    List<List<String>> res = new ArrayList<>();
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return res;

    Map<String, List<String>> parents = new HashMap<>();   // word -> its predecessors
    Set<String> currentLevel = new HashSet<>();
    currentLevel.add(beginWord);
    boolean found = false;

    while (!currentLevel.isEmpty() && !found) {
        dict.removeAll(currentLevel);                       // mark this level visited
        Set<String> nextLevel = new HashSet<>();
        for (String word : currentLevel) {
            char[] arr = word.toCharArray();
            for (int j = 0; j < arr.length; j++) {
                char original = arr[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    arr[j] = c;
                    String next = new String(arr);
                    if (dict.contains(next)) {
                        nextLevel.add(next);
                        parents.computeIfAbsent(next, x -> new ArrayList<>()).add(word);
                        if (next.equals(endWord)) found = true;
                    }
                }
                arr[j] = original;
            }
        }
        currentLevel = nextLevel;
    }

    if (found) {
        LinkedList<String> path = new LinkedList<>();
        path.add(endWord);
        backtrack(endWord, beginWord, parents, path, res);
    }
    return res;
}
private void backtrack(String word, String begin, Map<String, List<String>> parents,
                       LinkedList<String> path, List<List<String>> res) {
    if (word.equals(begin)) {
        res.add(new ArrayList<>(path));
        return;
    }
    for (String parent : parents.getOrDefault(word, List.of())) {
        path.addFirst(parent);                 // prepend (we build from end to start)
        backtrack(parent, begin, parents, path, res);
        path.removeFirst();                     // backtrack
    }
}
```

**Why remove the whole level at once:** Two different words on the same BFS level might both be valid predecessors of a next-level word. Removing words mid-level would drop some of these equally-short predecessors, losing valid shortest paths.

**Complexity:** Exponential in the number of shortest paths (output-sensitive); BFS phase O(N·L·26).

**Follow-ups:**
1. *"Why build parents during BFS instead of a full adjacency?"* → It records only shortest-path predecessors, so backtracking yields only shortest paths.

**Common mistakes:**
- Removing words from the dictionary as soon as they're seen (mid-level), losing alternate shortest predecessors.
- Reconstructing paths with plain DFS over the whole graph (includes non-shortest paths).

---

### Q81: Union-Find / Disjoint Set Union (DSU) Template
**Company:** Amazon, Google, Facebook, Microsoft — foundational
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Implement a Union-Find data structure with path compression and union by rank, supporting `find`, `union`, and `connected`. This is the reusable building block for many graph problems.

**What interviewer is testing:**
A clean, optimized DSU you can write from memory — the backbone of Q82, Q83, Q84, Q96, and the Union-Find variants of Q68/Q74.

**Ideal Answer:**

Two optimizations make DSU nearly O(1) per operation (inverse-Ackermann α):
- **Path compression** in `find`: flatten the tree by pointing nodes directly to the root.
- **Union by rank** (or size): attach the shorter tree under the taller to keep trees shallow.

```java
class UnionFind {
    private final int[] parent;
    private final int[] rank;     // upper bound on tree height
    private int count;            // number of disjoint sets

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        count = n;
        for (int i = 0; i < n; i++) parent[i] = i;   // each element its own set
    }

    // find with path compression (point nodes directly toward the root)
    public int find(int x) {
        while (x != parent[x]) {
            parent[x] = parent[parent[x]];   // path halving: skip a level
            x = parent[x];
        }
        return x;
    }

    // union by rank; returns false if already connected (useful for cycle detection)
    public boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;          // already in same set -> would form a cycle
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }  // ra is taller
        parent[rb] = ra;                     // attach shorter under taller
        if (rank[ra] == rank[rb]) rank[ra]++;
        count--;                             // two sets merged
        return true;
    }

    public boolean connected(int a, int b) { return find(a) == find(b); }
    public int getCount() { return count; }
}
```

**Why nearly O(1):** With both optimizations, a sequence of m operations on n elements runs in O(m · α(n)) where α is the inverse Ackermann function — effectively constant (α(n) ≤ 4 for any practical n).

**Complexity:** `find`/`union`/`connected` amortized O(α(n)) ≈ O(1). Space O(n).

**Follow-up questions the interviewer will ask:**
1. *"Path compression vs union by rank — does one alone suffice?"* → Either alone gives O(log n) amortized; together they give O(α(n)). Always include both in interviews.
2. *"Union by size vs by rank?"* → Equivalent asymptotically; size attaches the smaller-count tree under the larger, rank uses an estimated height. Both fine.
3. *"Detect a cycle while building a graph?"* → `union` returning false means the two endpoints were already connected → the new edge closes a cycle (basis of Q82, Q85).

**Common mistakes:**
- Omitting one optimization (degrades to O(log n) or O(n) worst case).
- `find` without path compression on large inputs → TLE.
- Forgetting to decrement the component count on a successful union.

---

### Q82: Redundant Connection
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A tree of `n` nodes had one extra edge added, creating exactly one cycle. Given the edge list, return the edge that can be removed so the result is a tree. If multiple, return the one occurring last in the input.

**What interviewer is testing:**
Union-Find for cycle detection: process edges in order; the first edge that connects two already-connected nodes is the redundant one.

**Ideal Answer:**

```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    UnionFind uf = new UnionFind(n + 1);     // nodes are 1-indexed
    for (int[] e : edges) {
        if (!uf.union(e[0], e[1])) {          // union returns false => already connected
            return e;                          // this edge closes the cycle
        }
    }
    return new int[0];   // shouldn't happen per problem guarantee
}
// (uses the UnionFind class from Q81)
```

**Why it returns the last redundant edge:** processing in input order, the cycle-closing edge encountered is exactly the one to remove; since there's only one extra edge, the first union failure is the answer (and it's the latest such edge in the input by the problem's structure).

**Complexity:** O(n · α(n)) ≈ O(n) time, O(n) space.

**Follow-up:** *"Redundant Connection II (directed graph)?"* → Harder: must handle a node with two parents AND a cycle; combine indegree checks with Union-Find.

**Common mistakes:**
- 0-indexing the DSU when nodes are 1-indexed (size `n+1`).
- Using DFS cycle detection (works but Union-Find is cleaner and incremental).

---

### Q83: Accounts Merge
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Each account is `[name, email1, email2, ...]`. Two accounts belong to the same person if they share any email. Merge accounts: output `[name, sorted unique emails...]` per person.

**What interviewer is testing:**
Union-Find over emails (the connecting key), then grouping by root. A classic "union by shared attribute" problem.

**Ideal Answer:**

Assign each unique email an id. Union all emails within the same account (they belong to one person). Then group emails by their DSU root, attach the owner's name (any account containing one of the emails), and sort.

```java
public List<List<String>> accountsMerge(List<List<String>> accounts) {
    Map<String, Integer> emailId = new HashMap<>();   // email -> id
    Map<String, String> emailName = new HashMap<>();  // email -> owner name
    int id = 0;
    for (List<String> acc : accounts) {
        String name = acc.get(0);
        for (int i = 1; i < acc.size(); i++) {
            String email = acc.get(i);
            if (!emailId.containsKey(email)) emailId.put(email, id++);
            emailName.put(email, name);
        }
    }
    UnionFind uf = new UnionFind(id);
    for (List<String> acc : accounts) {
        int first = emailId.get(acc.get(1));
        for (int i = 2; i < acc.size(); i++) {
            uf.union(first, emailId.get(acc.get(i)));   // all emails in an account unite
        }
    }
    // group emails by root
    Map<Integer, List<String>> groups = new HashMap<>();
    for (String email : emailId.keySet()) {
        int root = uf.find(emailId.get(email));
        groups.computeIfAbsent(root, x -> new ArrayList<>()).add(email);
    }
    List<List<String>> res = new ArrayList<>();
    for (List<String> emails : groups.values()) {
        Collections.sort(emails);                       // emails sorted
        List<String> entry = new ArrayList<>();
        entry.add(emailName.get(emails.get(0)));        // owner name
        entry.addAll(emails);
        res.add(entry);
    }
    return res;
}
// (uses the UnionFind class from Q81)
```

**Complexity:** O(N·K·log(N·K)) dominated by sorting emails, where N accounts each with up to K emails.

**Follow-up:** *"DFS/BFS alternative?"* → Build an email graph (edges between emails in the same account) and find connected components; Union-Find is cleaner here.

**Common mistakes:**
- Unioning by name (different people can share a name).
- Forgetting to dedup and sort emails.

---

### Q84: Number of Operations to Make Network Connected
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
There are `n` computers and a list of `connections` (cables). You can unplug a cable and replug it elsewhere. Return the minimum operations to connect all computers, or -1 if impossible.

**What interviewer is testing:**
The key counting insight: to connect `c` components you need `c − 1` extra cables, and you have enough spare cables iff the number of edges ≥ `n − 1`.

**Ideal Answer:**

First feasibility: you need at least `n − 1` cables to connect `n` computers; if `connections.length < n − 1`, return -1. Otherwise count connected components `c` with Union-Find; the answer is `c − 1` (you always have enough redundant cables to move).

```java
public int makeConnected(int n, int[][] connections) {
    if (connections.length < n - 1) return -1;     // not enough cables at all
    UnionFind uf = new UnionFind(n);
    for (int[] c : connections) uf.union(c[0], c[1]);
    return uf.getCount() - 1;                       // components - 1 cables to move
}
// (uses the UnionFind class from Q81)
```

**Why `components − 1`:** Connecting `c` separate components into one requires exactly `c − 1` connections. Any cycle edge within a component is redundant and can be repurposed; with ≥ n−1 total cables there are always enough redundant ones.

**Complexity:** O(E · α(n)) ≈ O(E) time, O(n) space.

**Follow-up:** *"Prove you always have enough spare cables when edges ≥ n−1."* → A spanning structure of `c` components uses `n − c` edges; the rest (`edges − (n − c)`) are redundant. Since edges ≥ n − 1, redundant ≥ c − 1, exactly what's needed.

**Common mistakes:**
- Forgetting the `< n − 1` impossibility check.
- Returning `components` instead of `components − 1`.

---

### Q85: Graph Valid Tree
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given `n` nodes and an edge list, determine whether the graph forms a valid tree (connected and acyclic).

**What interviewer is testing:**
The two tree conditions and the slick check: a graph with `n` nodes is a tree iff it has exactly `n − 1` edges AND is connected (or equivalently, n−1 edges and acyclic — any two imply the third).

**Ideal Answer:**

Quick check: a tree on `n` nodes has exactly `n − 1` edges. If not, it's not a tree. Then verify connectivity (or acyclicity) — with `n − 1` edges, connected ⟺ acyclic, so checking either suffices. Union-Find: if any union fails (cycle) it's not a tree; otherwise the edge count guarantees connectivity.

```java
public boolean validTree(int n, int[][] edges) {
    if (edges.length != n - 1) return false;     // tree must have exactly n-1 edges
    UnionFind uf = new UnionFind(n);
    for (int[] e : edges) {
        if (!uf.union(e[0], e[1])) return false;  // a cycle -> not a tree
    }
    return true;   // n-1 edges + no cycle => connected & acyclic => tree
}
// (uses the UnionFind class from Q81)
```

**Why the edge-count check is powerful:** With exactly `n − 1` edges, the graph is a tree iff it's acyclic (no cycle means all edges contributed to connecting, forcing connectivity). So one Union-Find pass with the edge-count guard fully decides it.

**Complexity:** O(E · α(n)) ≈ O(n) time, O(n) space.

**Follow-up:** *"BFS/DFS version?"* → Check `edges == n-1`, then BFS/DFS from node 0 and confirm all `n` nodes are reached (connectivity).

**Common mistakes:**
- Checking only connectivity or only acyclicity without the edge count (need two of the three properties).

---

### Q86: Is Graph Bipartite?
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an undirected graph as an adjacency list, return whether it is bipartite (nodes can be 2-colored so every edge connects different colors).

**What interviewer is testing:**
2-coloring via BFS/DFS — assign alternating colors; a conflict (adjacent same color) means not bipartite. Must handle disconnected components.

**Ideal Answer:**

BFS/DFS each uncolored component, coloring neighbors the opposite color. If a neighbor is already colored the same, it's not bipartite.

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];        // 0 = uncolored, 1 / -1 = two colors
    for (int start = 0; start < n; start++) {
        if (color[start] != 0) continue;        // already colored
        Queue<Integer> q = new LinkedList<>();
        q.offer(start);
        color[start] = 1;
        while (!q.isEmpty()) {
            int node = q.poll();
            for (int nei : graph[node]) {
                if (color[nei] == 0) {
                    color[nei] = -color[node];   // opposite color
                    q.offer(nei);
                } else if (color[nei] == color[node]) {
                    return false;                 // same color on an edge -> not bipartite
                }
            }
        }
    }
    return true;
}
```

**Complexity:** O(V + E) time, O(V) space.

**Pattern:** Bipartite check = 2-coloring; an odd-length cycle is exactly what breaks it.

**Follow-up:** *"Relation to odd cycles?"* → A graph is bipartite iff it has no odd-length cycle.

**Common mistakes:**
- Not iterating over all start nodes (misses disconnected components).
- Coloring conflict check logic inverted.

---

### Q87: Detect Cycle in a Directed Graph
**Company:** Amazon, Microsoft, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given a directed graph, determine whether it contains a cycle.

**What interviewer is testing:**
DFS with 3-color (white/gray/black) marking — the same mechanism as Course Schedule's DFS variant (Q76), isolated as its own pattern.

**Ideal Answer:**

A directed graph has a cycle iff DFS encounters a **back edge** — an edge to a node currently on the recursion stack (gray).

```java
public boolean hasCycleDirected(int n, List<List<Integer>> adj) {
    int[] state = new int[n];        // 0=white, 1=gray (on stack), 2=black (done)
    for (int i = 0; i < n; i++) {
        if (state[i] == 0 && dfs(adj, state, i)) return true;
    }
    return false;
}
private boolean dfs(List<List<Integer>> adj, int[] state, int node) {
    state[node] = 1;                 // gray: entering recursion
    for (int nei : adj.get(node)) {
        if (state[nei] == 1) return true;             // back edge -> cycle
        if (state[nei] == 0 && dfs(adj, state, nei)) return true;
    }
    state[node] = 2;                 // black: fully explored
    return false;
}
```

**Why 3 colors:** a black node was finished in a *previous* DFS path and re-reaching it is fine (it's a cross/forward edge, not a cycle). Only a gray node — still on the current stack — signals a cycle.

**Complexity:** O(V + E) time, O(V) space.

**Follow-up:** *"Kahn's BFS alternative?"* → If topological sort processes fewer than `n` nodes, there's a cycle (Q76).

**Common mistakes:**
- Using a 2-state visited set → false positives on shared subgraphs.

---

### Q88: Detect Cycle in an Undirected Graph
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Determine whether an undirected graph contains a cycle.

**What interviewer is testing:**
The difference from the directed case: every edge is bidirectional, so you must ignore the edge back to your parent — a cycle is reaching an *already-visited* node that isn't the parent. Union-Find is also clean here.

**Ideal Answer:**

*DFS tracking parent:* a visited neighbor that isn't the parent means a cycle.

```java
public boolean hasCycleUndirected(int n, List<List<Integer>> adj) {
    boolean[] visited = new boolean[n];
    for (int i = 0; i < n; i++) {
        if (!visited[i] && dfs(adj, visited, i, -1)) return true;
    }
    return false;
}
private boolean dfs(List<List<Integer>> adj, boolean[] visited, int node, int parent) {
    visited[node] = true;
    for (int nei : adj.get(node)) {
        if (!visited[nei]) {
            if (dfs(adj, visited, nei, node)) return true;
        } else if (nei != parent) {        // visited and not the parent -> cycle
            return true;
        }
    }
    return false;
}
```

*Union-Find:* iterate edges; if both endpoints already share a root, the edge closes a cycle.

**Complexity:** O(V + E) time, O(V) space.

**Follow-up:** *"Why the parent check?"* → In an undirected graph the edge u-v appears as v in u's list and u in v's list; without skipping the parent you'd falsely flag the immediate back-edge as a cycle.

**Common mistakes:**
- Forgetting the parent exclusion (false cycle on every edge).
- With parallel edges/self-loops, the simple parent check needs refinement (count edges).

---

### Q89: Minimum Height Trees
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a tree (`n` nodes, `n−1` edges), find all root nodes that minimize the tree's height. Return the list of such roots (there are at most 2).

**What interviewer is testing:**
The "trim leaves layer by layer" insight — the answer is the center(s) of the tree, found by repeatedly removing leaves until 1 or 2 nodes remain (a topological-sort-like peeling).

**Ideal Answer:**

The minimum-height roots are the graph's centroids. Peel leaves (degree-1 nodes) layer by layer; the last 1 or 2 remaining nodes are the centers.

```java
public List<Integer> findMinHeightTrees(int n, int[][] edges) {
    if (n == 1) return List.of(0);
    List<Set<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new HashSet<>());
    for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }

    List<Integer> leaves = new ArrayList<>();
    for (int i = 0; i < n; i++) if (adj.get(i).size() == 1) leaves.add(i);

    int remaining = n;
    while (remaining > 2) {                 // until 1 or 2 centers remain
        remaining -= leaves.size();
        List<Integer> newLeaves = new ArrayList<>();
        for (int leaf : leaves) {
            int neighbor = adj.get(leaf).iterator().next();
            adj.get(neighbor).remove(leaf);          // remove the leaf edge
            if (adj.get(neighbor).size() == 1) newLeaves.add(neighbor); // became a leaf
        }
        leaves = newLeaves;
    }
    return leaves;                          // the 1 or 2 centers
}
```

**Why centers minimize height:** rooting at a node far from center makes one side very deep; the center balances the longest paths. The longest path (diameter) has 1 or 2 middle nodes — exactly the survivors of leaf-peeling.

**Complexity:** O(V + E) time, O(V + E) space.

**Follow-up:** *"Why at most 2 answers?"* → A tree's diameter has either one middle node (odd length) or two (even length); those are the only height-minimizing roots.

**Common mistakes:**
- Trying every node as root (O(n²)).
- Not handling `n == 1` (single node) and `n == 2` (both are centers).

---

### Q90: Dijkstra's Algorithm
**Company:** Amazon, Google, Facebook, Microsoft, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given a weighted graph with **non-negative** edge weights and a source, compute the shortest distance from the source to every node.

**What interviewer is testing:**
The priority-queue implementation, *why* it's correct (the greedy "closest unfinalized node is final"), and why it fails with negative edges.

**Ideal Answer:**

Maintain tentative distances (∞ initially, 0 for the source) and a min-heap of `(distance, node)`. Repeatedly extract the closest unfinalized node, finalize it, and **relax** its outgoing edges (if going through it improves a neighbor's distance, update and push). Using a lazy heap (push duplicates, skip stale entries) keeps it simple.

```java
public int[] dijkstra(int n, List<int[]>[] adj, int src) {
    // adj[u] = list of {v, weight}
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    // min-heap by distance: {distance, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, src});
    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int d = top[0], u = top[1];
        if (d > dist[u]) continue;                  // stale entry -> skip
        for (int[] edge : adj[u]) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {             // relaxation
                dist[v] = dist[u] + w;
                pq.offer(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
```

**Proof of correctness (greedy):** When we extract node `u` with the smallest tentative distance, that distance is final. Suppose not — a shorter path to `u` exists. That path leaves the finalized set through some node `x` with `dist[x] + (rest) < dist[u]`. But all edge weights are non-negative, so `dist[x] ≤ dist[x] + (rest) < dist[u]`, contradicting that `u` was the closest unfinalized node (x would have been extracted first). Hence `dist[u]` is final. **This proof relies on non-negative weights** — a negative edge could make a longer-looking path actually shorter, breaking the greedy.

**Dry run** — nodes 0,1,2; edges 0→1 (4), 0→2 (1), 2→1 (2): start dist[0]=0. Pop 0; relax → dist[2]=1, dist[1]=4. Pop 2 (dist 1); relax 2→1: 1+2=3 < 4 → dist[1]=3. Pop 1 (dist 3) final. Distances {0,3,1}. ✅

**Complexity:** O((V + E) log V) time with a binary heap, O(V) space. (O(E + V log V) with a Fibonacci heap, rarely used in practice.)

**Follow-up questions the interviewer will ask:**
1. *"Why does Dijkstra fail with negative edges?"* → The greedy finality proof breaks; a node finalized early might later be reachable via a cheaper path through a negative edge. Use Bellman-Ford (Q94).
2. *"Lazy vs eager (decrease-key) deletion?"* → Lazy (push duplicates, skip stale) is simpler and standard in Java since `PriorityQueue` lacks decrease-key; eager uses an indexed heap.
3. *"Reconstruct the path?"* → Keep a `prev[]` array updated during relaxation; backtrack from the target.
4. *"Single target?"* → Stop early when you pop the target node.

**Common mistakes:**
- Using it with negative edges.
- Forgetting the stale-entry skip (`d > dist[u]`) → still correct but slower / can reprocess.
- Updating `dist` but not pushing the new entry.

---

### Q91: Network Delay Time
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
A signal starts at node `k` in a directed weighted graph. `times[i] = [u, v, w]` is the travel time on edge u→v. Return the time for all `n` nodes to receive the signal, or -1 if some node is unreachable.

**What interviewer is testing:**
A direct Dijkstra application: the answer is the *maximum* shortest distance from `k` to any node.

**Ideal Answer:**

Run Dijkstra from `k`. The signal reaches everyone at `max(dist)`; if any node is still ∞ (unreachable), return -1.

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<int[]>[] adj = new List[n + 1];
    for (int i = 1; i <= n; i++) adj[i] = new ArrayList<>();
    for (int[] t : times) adj[t[0]].add(new int[]{t[1], t[2]});

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, k});
    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int d = top[0], u = top[1];
        if (d > dist[u]) continue;
        for (int[] e : adj[u]) {
            if (dist[u] + e[1] < dist[e[0]]) {
                dist[e[0]] = dist[u] + e[1];
                pq.offer(new int[]{dist[e[0]], e[0]});
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;   // unreachable
        ans = Math.max(ans, dist[i]);                   // last node to receive
    }
    return ans;
}
```

**Complexity:** O((V + E) log V) time, O(V + E) space.

**Follow-up:** *"Why the max?"* → All nodes receive in parallel along their shortest paths; the slowest (max shortest distance) determines when *all* have received.

**Common mistakes:**
- Returning the sum instead of the max.
- Off-by-one with 1-indexed nodes.

---

### Q92: Cheapest Flights Within K Stops
**Company:** Amazon, Google, Facebook
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Given flights `[from, to, price]`, find the cheapest price from `src` to `dst` with at most `k` stops (so at most `k+1` edges). Return -1 if no such route.

**What interviewer is testing:**
The stop constraint breaks plain Dijkstra's optimality. The clean fix is **Bellman-Ford limited to k+1 relaxation rounds**, or Dijkstra augmented with a stops dimension.

**Ideal Answer:**

*Bellman-Ford with k+1 rounds:* relax all edges `k+1` times, but each round must use distances from the *previous* round only (clone the array) so a single round can't chain multiple flights.

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    for (int i = 0; i <= k; i++) {              // at most k+1 edges
        int[] prev = dist.clone();               // use last round's distances only
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (prev[u] != Integer.MAX_VALUE && prev[u] + w < dist[v]) {
                dist[v] = prev[u] + w;
            }
        }
    }
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

**Why clone `prev`:** Without it, edges relaxed earlier in the same round could be reused later in that same round, effectively allowing more than one flight per round — violating the stop limit. The clone enforces "exactly one more edge per round."

**Dry run** — n=3, flights `[[0,1,100],[1,2,100],[0,2,500]]`, src=0, dst=2, k=1: round 0: dist[1]=100, dist[2]=500. round 1: from prev, dist[2]=min(500, prev[1]+100=200)=200. Answer 200 (2 edges, 1 stop). ✅

**Complexity:** O(k · E) time, O(V) space.

**Follow-up questions the interviewer will ask:**
1. *"Dijkstra with a stops dimension?"* → Push `(cost, node, stopsUsed)`; allow revisiting a node with more stops if cheaper. Works but the bounded Bellman-Ford is cleaner and avoids subtle pruning bugs.
2. *"Why not plain Dijkstra?"* → Dijkstra finalizes a node's min cost ignoring stop count; the cheapest route may exceed k stops while a pricier route within k stops is the real answer.

**Common mistakes:**
- Not cloning `prev` (allows multi-hop in one round → undercounts stops).
- Off-by-one: `k` stops means `k+1` edges → loop `k+1` times.

---

### Q93: Path with Maximum Probability
**Company:** Amazon, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given an undirected graph where each edge has a success probability, find the path from `start` to `end` with the maximum product of probabilities. Return that probability (0 if no path).

**What interviewer is testing:**
Recognizing this as Dijkstra with a **max-heap** maximizing a product instead of minimizing a sum. Probabilities are in [0,1], so multiplying only shrinks — analogous to non-negative weights.

**Ideal Answer:**

Run a Dijkstra variant: a max-heap by probability. Start with probability 1 at `start`; relax edges by multiplying; keep the best probability to each node.

```java
public double maxProbability(int n, int[][] edges, double[] succProb, int start, int end) {
    List<double[]>[] adj = new List[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int i = 0; i < edges.length; i++) {
        int u = edges[i][0], v = edges[i][1];
        adj[u].add(new double[]{v, succProb[i]});
        adj[v].add(new double[]{u, succProb[i]});   // undirected
    }
    double[] prob = new double[n];
    prob[start] = 1.0;
    // max-heap by probability
    PriorityQueue<double[]> pq = new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));
    pq.offer(new double[]{start, 1.0});
    while (!pq.isEmpty()) {
        double[] top = pq.poll();
        int u = (int) top[0]; double p = top[1];
        if (u == end) return p;                      // first time we pop end -> max
        if (p < prob[u]) continue;                   // stale
        for (double[] e : adj[u]) {
            int v = (int) e[0]; double np = p * e[1];
            if (np > prob[v]) {                       // better probability
                prob[v] = np;
                pq.offer(new double[]{v, np});
            }
        }
    }
    return 0.0;
}
```

**Why Dijkstra applies:** multiplying probabilities (all ≤ 1) only decreases the path value, mirroring the "edges don't reduce distance" property that makes Dijkstra's greedy valid. So the first time we pop `end` we have its maximum probability.

**Complexity:** O((V + E) log V) time, O(V + E) space.

**Follow-up:** *"Log transform?"* → Taking −log turns the max-product into a min-sum of non-negative weights — standard Dijkstra. The direct max-heap version avoids floating-point log error.

**Common mistakes:**
- Using a min-heap (you want the maximum product first).
- Forgetting the graph is undirected (add both edge directions).

---

### Q94: Bellman-Ford Algorithm
**Company:** Amazon, Google, Two Sigma
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Compute shortest paths from a source in a graph that may have **negative edge weights**, and detect negative-weight cycles.

**What interviewer is testing:**
The relax-all-edges-V−1-times algorithm, why V−1 rounds suffice, and the Vth-round negative-cycle detection.

**Ideal Answer:**

Relax every edge `V − 1` times. After `i` rounds, all shortest paths using at most `i` edges are correct; since a simple shortest path has at most `V − 1` edges, `V − 1` rounds finalize all distances. A `V`th round that still relaxes something proves a negative cycle (no finite shortest path exists).

```java
public int[] bellmanFord(int n, int[][] edges, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    for (int i = 0; i < n - 1; i++) {              // V-1 rounds
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;             // relax
            }
        }
    }
    // Vth round: any further relaxation => negative cycle
    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            throw new IllegalStateException("Negative weight cycle detected");
        }
    }
    return dist;
}
```

**Why V−1 rounds:** a shortest path in a graph without negative cycles is simple (no repeated vertices), so it has at most `V − 1` edges. Each round extends correct shortest-path estimates by at least one edge, so after `V − 1` rounds every shortest path is found.

**Dry run** — nodes 0,1,2; edges 0→1 (4), 1→2 (−2), 0→2 (5): round1: dist[1]=4, dist[2]=min(5, 4−2=2)=2. Stable thereafter. {0,4,2}. ✅

**Complexity:** O(V · E) time, O(V) space.

**Follow-up questions the interviewer will ask:**
1. *"When use Bellman-Ford over Dijkstra?"* → Negative edges, or when you must detect negative cycles. Dijkstra is faster (O(E log V)) but requires non-negative weights.
2. *"Find the nodes affected by a negative cycle?"* → On the Vth round, mark relaxed nodes and BFS/DFS to mark everything reachable from them as `−∞`.
3. *"SPFA optimization?"* → A queue-based Bellman-Ford that only re-relaxes changed nodes; faster on average, same worst case.

**Common mistakes:**
- Relaxing `V` times instead of `V − 1` (wastes a round; or confuses the cycle check).
- Integer overflow: `dist[u] + w` when `dist[u]` is MAX_VALUE — guard with the `!= MAX_VALUE` check.

---

### Q95: Floyd-Warshall Algorithm
**Company:** Amazon, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Compute shortest paths between **all pairs** of nodes in a weighted graph (negative edges allowed, no negative cycles).

**What interviewer is testing:**
The triple-loop DP and the crucial loop ordering (intermediate node `k` must be the outermost loop).

**Ideal Answer:**

`dist[i][j]` = shortest path from i to j using only nodes `{0..k}` as intermediates. For each candidate intermediate `k`, update `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`. The intermediate node `k` is the **outer** loop — this order ensures we consider paths through progressively larger intermediate sets.

```java
public int[][] floydWarshall(int n, int[][] graph) {
    final int INF = Integer.MAX_VALUE / 2;        // avoid overflow on addition
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : graph) dist[e[0]][e[1]] = Math.min(dist[e[0]][e[1]], e[2]);

    for (int k = 0; k < n; k++) {                  // intermediate node: OUTERMOST
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    return dist;
}
```

**Why `k` must be outermost:** the DP relies on having all shortest paths using intermediates `{0..k−1}` finalized before introducing `k`. If `i` or `j` were outermost, you'd use partially-computed `dist[i][k]`/`dist[k][j]` values and get wrong answers.

**Complexity:** O(V³) time, O(V²) space.

**Follow-up questions the interviewer will ask:**
1. *"When is Floyd-Warshall the right choice?"* → Small dense graphs needing all-pairs distances; simpler than running Dijkstra/Bellman-Ford from every source.
2. *"Detect a negative cycle?"* → If any `dist[i][i] < 0` after the algorithm, node i is on a negative cycle.
3. *"Use `INF/2`?"* → To prevent `INF + INF` integer overflow during the addition.

**Common mistakes:**
- Wrong loop order (k not outermost) → incorrect results.
- Overflow from adding two INF values (use `INF = MAX/2`).

---

### Q96: Minimum Spanning Tree — Kruskal's Algorithm
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a connected weighted undirected graph, find the total weight of a minimum spanning tree (MST).

**What interviewer is testing:**
Kruskal's greedy: sort edges, add the cheapest that doesn't form a cycle (detected via Union-Find), until `V − 1` edges are chosen.

**Ideal Answer:**

Sort all edges ascending by weight. Iterate; use Union-Find to add an edge only if its endpoints are in different components (no cycle). Stop after `V − 1` edges.

```java
public int kruskalMST(int n, int[][] edges) {
    Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));   // by weight
    UnionFind uf = new UnionFind(n);
    int totalWeight = 0, used = 0;
    for (int[] e : edges) {
        if (uf.union(e[0], e[1])) {        // union returns true if they were separate
            totalWeight += e[2];
            if (++used == n - 1) break;     // MST complete
        }
    }
    return totalWeight;
}
// (uses the UnionFind class from Q81)
```

**Why greedy works (cut property):** the lightest edge crossing any cut between two disjoint vertex sets is safe to include in some MST. Kruskal repeatedly takes the globally lightest edge that connects two components — exactly such a crossing edge — so it never makes a wrong choice.

**Complexity:** O(E log E) time (sorting dominates), O(V) space.

**Follow-up questions the interviewer will ask:**
1. *"Kruskal vs Prim?"* → Kruskal sorts edges + Union-Find, great for sparse graphs / edge lists. Prim grows from a node with a heap, great for dense graphs / adjacency matrices.
2. *"Maximum spanning tree?"* → Sort edges descending; same algorithm.
3. *"Disconnected graph?"* → Kruskal yields a minimum spanning forest; you'd get fewer than V−1 edges.

**Common mistakes:**
- Not stopping at `V − 1` edges (harmless but wasteful).
- Forgetting Union-Find optimizations on large edge sets.

---

### Q97: Minimum Spanning Tree — Prim's Algorithm
**Company:** Amazon, Google
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Find the MST weight using Prim's algorithm (heap-based).

**What interviewer is testing:**
The "grow the tree one node at a time, always adding the cheapest edge crossing into the tree" greedy via a min-heap.

**Ideal Answer:**

Start from any node. Maintain a min-heap of edges crossing from the tree to outside. Repeatedly take the cheapest edge to a new (unvisited) node, add its weight, mark it visited, and push its outgoing edges.

```java
public int primMST(int n, List<int[]>[] adj) {
    // adj[u] = list of {v, weight}
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]); // {node, edgeWeight}
    pq.offer(new int[]{0, 0});            // start from node 0, cost 0
    int totalWeight = 0, count = 0;
    while (!pq.isEmpty() && count < n) {
        int[] top = pq.poll();
        int u = top[0], w = top[1];
        if (inMST[u]) continue;            // already in the tree -> skip
        inMST[u] = true;
        totalWeight += w;
        count++;
        for (int[] e : adj[u]) {
            if (!inMST[e[0]]) pq.offer(new int[]{e[0], e[1]});  // crossing edges
        }
    }
    return count == n ? totalWeight : -1;   // -1 if graph disconnected
}
```

**Why greedy works:** same cut property — the minimum-weight edge crossing the cut between the current tree and the rest is always safe to add. Prim grows the tree by repeatedly choosing that crossing edge.

**Complexity:** O(E log V) time with a binary heap, O(V + E) space.

**Follow-up:** *"Dense graph?"* → An adjacency-matrix Prim's with an array (no heap) is O(V²), better than O(E log V) when E ≈ V².

**Common mistakes:**
- Adding a node already in the MST (must skip via `inMST`).
- Pushing edges to already-included nodes and not skipping stale heap entries.

---

### Q98: Critical Connections in a Network (Tarjan's Bridges)
**Company:** Amazon, Google, Facebook
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a connected undirected graph, find all **bridges** — edges whose removal disconnects the graph.

**What interviewer is testing:**
Tarjan's bridge-finding via DFS discovery times and "low-link" values. A genuinely hard algorithm; knowing it signals strong graph depth.

**Ideal Answer:**

DFS assigning each node a discovery time `disc[u]`. `low[u]` = the lowest discovery time reachable from `u`'s subtree via at most one back edge. An edge `(u, v)` (v a child of u) is a bridge iff `low[v] > disc[u]` — meaning v's subtree has no back edge reaching u or above, so the edge is the only connection.

```java
public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    List<Integer>[] adj = new List[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (List<Integer> c : connections) {
        adj[c.get(0)].add(c.get(1));
        adj[c.get(1)].add(c.get(0));
    }
    int[] disc = new int[n], low = new int[n];
    Arrays.fill(disc, -1);                       // -1 = unvisited
    List<List<Integer>> bridges = new ArrayList<>();
    timer = 0;
    dfs(0, -1, adj, disc, low, bridges);
    return bridges;
}
private int timer;
private void dfs(int u, int parent, List<Integer>[] adj,
                int[] disc, int[] low, List<List<Integer>> bridges) {
    disc[u] = low[u] = timer++;                  // set discovery time & init low
    for (int v : adj[u]) {
        if (v == parent) continue;               // don't go back along the same edge
        if (disc[v] == -1) {                     // tree edge: v unvisited
            dfs(v, u, adj, disc, low, bridges);
            low[u] = Math.min(low[u], low[v]);   // propagate child's reach up
            if (low[v] > disc[u]) {              // no back edge bypasses (u,v) -> bridge
                bridges.add(List.of(u, v));
            }
        } else {                                 // back edge: update low
            low[u] = Math.min(low[u], disc[v]);
        }
    }
}
```

**Why `low[v] > disc[u]` means a bridge:** `low[v]` is the earliest node v's subtree can reach. If it can't reach `u` or any ancestor (`low[v] > disc[u]`), then removing edge (u,v) leaves v's subtree with no other route back — it's a bridge.

**Complexity:** O(V + E) time, O(V) space.

**Follow-up questions the interviewer will ask:**
1. *"Articulation points (cut vertices)?"* → Similar Tarjan DFS with `low[v] >= disc[u]` (and a special root-with-2-children rule).
2. *"Handle parallel edges?"* → Track the edge id used to enter, not just the parent node, so a true parallel edge isn't mistaken for the back edge.

**Common mistakes:**
- Using `>=` instead of `>` (that's articulation points, not bridges).
- `v == parent` skip failing with multi-edges.

---

### Q99: Reconstruct Itinerary (Hierholzer's Algorithm)
**Company:** Amazon, Google, Facebook
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Given a list of airline tickets `[from, to]`, reconstruct the itinerary starting from "JFK", using all tickets exactly once. If multiple valid itineraries exist, return the lexicographically smallest.

**What interviewer is testing:**
Recognizing this as finding an **Eulerian path** (use every edge once) and applying Hierholzer's algorithm with a greedy lexicographic order via min-heaps / sorted adjacency.

**Ideal Answer:**

Build adjacency lists sorted lexicographically (use a min-heap or sorted structure per source). Hierholzer's: DFS greedily down the smallest destination, consuming edges; when a node has no more outgoing edges, prepend it to the route (post-order). Reverse the post-order to get the itinerary.

```java
public List<String> findItinerary(List<List<String>> tickets) {
    Map<String, PriorityQueue<String>> adj = new HashMap<>();
    for (List<String> t : tickets) {
        adj.computeIfAbsent(t.get(0), x -> new PriorityQueue<>()).offer(t.get(1));
    }
    LinkedList<String> route = new LinkedList<>();
    dfs("JFK", adj, route);
    return route;
}
private void dfs(String airport, Map<String, PriorityQueue<String>> adj, LinkedList<String> route) {
    PriorityQueue<String> dests = adj.get(airport);
    while (dests != null && !dests.isEmpty()) {
        String next = dests.poll();         // smallest lexicographic destination
        dfs(next, adj, route);
    }
    route.addFirst(airport);                // post-order: add when no edges left
}
```

**Why post-order + reverse:** When we reach a node with no outgoing edges, it must be the *end* of the itinerary (a dead end). Adding nodes to the front as they're "finished" naturally reverses the dead-end-first order into a valid Eulerian path. Greedily taking the smallest destination gives the lexicographically smallest result.

**Dry run** — tickets `[[JFK,SFO],[JFK,ATL],[SFO,ATL],[ATL,JFK],[ATL,SFO]]`: greedily explore; the algorithm produces `JFK ATL JFK SFO ATL SFO`. ✅

**Complexity:** O(E log E) time (heap operations), O(E) space.

**Follow-up questions the interviewer will ask:**
1. *"Why not plain DFS backtracking?"* → A naive greedy DFS can get stuck (consume an edge needed later); Hierholzer's post-order handles this — a stuck dead-end becomes the tail.
2. *"Eulerian path existence conditions?"* → For a directed graph: at most one vertex with (outdeg − indeg = 1) as start, at most one with (indeg − outdeg = 1) as end, all others balanced, and the graph connected on edges.

**Common mistakes:**
- Building the route in forward order (gets stuck at dead ends).
- Not sorting destinations (fails the lexicographic requirement).

---

### Q100: Shortest Path in Binary Matrix
**Company:** Amazon, Facebook, Google
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone / Onsite

**Question:**
Given an `n×n` binary matrix, return the length of the shortest clear path from top-left to bottom-right, moving in **8 directions**, through `0` cells only. Return -1 if none.

**What interviewer is testing:**
Plain BFS for shortest path in an unweighted grid, with 8-directional movement — a clean application of the BFS template.

**Ideal Answer:**

BFS from `(0,0)`; the path length is the BFS depth. 8 directions. Mark cells visited (e.g., set to 1) to avoid revisits.

```java
public int shortestPathBinaryMatrix(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] != 0 || grid[n-1][n-1] != 0) return -1;   // blocked endpoints
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}};
    Queue<int[]> q = new LinkedList<>();
    q.offer(new int[]{0, 0});
    grid[0][0] = 1;                          // mark visited
    int pathLen = 1;
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            int[] cell = q.poll();
            if (cell[0] == n - 1 && cell[1] == n - 1) return pathLen;  // reached end
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0) {
                    grid[nr][nc] = 1;        // visited
                    q.offer(new int[]{nr, nc});
                }
            }
        }
        pathLen++;
    }
    return -1;
}
```

**Complexity:** O(n²) time and space.

**Follow-up:** *"A* search?"* → With a heuristic (e.g., Chebyshev distance to the goal) A* can prune; for this grid plain BFS is optimal and simplest.

**Common mistakes:**
- Checking the start/end blocked cells (immediate -1).
- 4-directional instead of 8 (wrong for this problem).
- Counting edges vs cells (the answer is the number of *cells* visited).

---

# REUSABLE TEMPLATES

Memorize these four. The vast majority of graph problems are one of them with a thin problem-specific wrapper.

## BFS Template (shortest path / level order in unweighted graphs)

```java
// Returns shortest distances from `src` in an unweighted graph.
int[] bfs(int n, List<List<Integer>> adj, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, -1);                  // -1 = unvisited
    Queue<Integer> q = new LinkedList<>();
    q.offer(src);
    dist[src] = 0;
    while (!q.isEmpty()) {
        int node = q.poll();
        for (int nei : adj.get(node)) {
            if (dist[nei] == -1) {           // first visit = shortest in unweighted graph
                dist[nei] = dist[node] + 1;
                q.offer(nei);
            }
        }
    }
    return dist;
}
```

Variations: **multi-source** (seed the queue with all sources at distance 0 — Q72, Q73); **level snapshot** (`int size = q.size()` per layer — Q4, Q21, Q25); **grid** (neighbors via direction arrays).

## DFS Template (connectivity, components, cycle detection, flood fill)

```java
// Recursive DFS marking a visited set.
void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    // ... pre-order work here ...
    for (int nei : adj.get(node)) {
        if (!visited[nei]) dfs(nei, adj, visited);
    }
    // ... post-order work here (e.g., topo order, low-link) ...
}
```

For **directed cycle detection** use 3-color state (white/gray/black) instead of a boolean (Q76, Q87). For **grids**, recurse on direction-offset neighbors and mark cells in place (Q68, Q71). Watch recursion depth on huge inputs — switch to an explicit stack or BFS.

## Topological Sort Template (Kahn's BFS — dependency ordering)

```java
// Returns a topological order, or empty list if a cycle exists.
List<Integer> topoSort(int n, List<List<Integer>> adj) {
    int[] indegree = new int[n];
    for (int u = 0; u < n; u++)
        for (int v : adj.get(u)) indegree[v]++;
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++) if (indegree[i] == 0) q.offer(i);
    List<Integer> order = new ArrayList<>();
    while (!q.isEmpty()) {
        int node = q.poll();
        order.add(node);
        for (int nei : adj.get(node)) {
            if (--indegree[nei] == 0) q.offer(nei);
        }
    }
    return order.size() == n ? order : new ArrayList<>();   // empty => cycle
}
```

Drives Course Schedule I/II (Q76, Q77), Alien Dictionary (Q78), Minimum Height Trees' leaf-peeling variant (Q89). For lexicographically smallest order, swap the queue for a `PriorityQueue`.

## Union-Find Template (dynamic connectivity, cycle detection, MST)

```java
class UnionFind {
    int[] parent, rank; int count;
    UnionFind(int n) {
        parent = new int[n]; rank = new int[n]; count = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {                        // path compression (halving)
        while (x != parent[x]) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
    boolean union(int a, int b) {            // union by rank; false if already joined
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        count--;
        return true;
    }
}
```

Powers Number of Islands / Components (Q68, Q74, Q75), Redundant Connection (Q82), Accounts Merge (Q83), Network Connected (Q84), Graph Valid Tree (Q85), Kruskal's MST (Q96). `union` returning false = the edge closes a cycle.

---

## Pattern recap — what to recognize on sight

| Trigger in problem statement | Pattern | Examples here |
|---|---|---|
| "compute X about the whole tree" (height, diameter, sum) | Post-order recursion, combine children | Q5, Q7, Q8, Q17, Q33 |
| "by level" / "shortest in unweighted" / "nearest" | BFS (with level snapshot or multi-source) | Q4, Q21, Q25, Q72, Q73, Q79, Q100 |
| "is it a valid BST" / "kth smallest" / "successor" | Inorder = sorted; exploit ordering | Q40, Q41, Q43, Q44, Q45 |
| "reconstruct tree from traversals" | Divide & conquer with index map | Q18, Q19 |
| "encode/rebuild a tree" | Preorder + null markers | Q20 |
| "kth largest/smallest" / "top k" | Fixed-size-k heap or QuickSelect | Q48, Q51, Q57 |
| "median of a stream" / "balance two halves" | Two heaps | Q49, Q55 |
| "schedule by priority" / "greedy pick best" | Max-heap greedy | Q50, Q52, Q60 |
| "prefix" / "autocomplete" / "dictionary of words" | Trie | Q61–Q65, Q67 |
| "maximum XOR" | Binary (bitwise) trie | Q66 |
| "count groups/islands/components" | DFS/BFS flood fill or Union-Find | Q68, Q74, Q75 |
| "dependency order" / "cycle in a DAG" | Topological sort (Kahn / DFS 3-color) | Q76, Q77, Q78, Q87 |
| "shortest path, non-negative weights" | Dijkstra (min-heap) | Q90, Q91, Q93 |
| "shortest path, ≤ k edges / negative weights" | Bellman-Ford (bounded rounds) | Q92, Q94 |
| "all-pairs shortest paths, small graph" | Floyd-Warshall (k outermost) | Q95 |
| "dynamic connectivity / cycle in undirected" | Union-Find | Q82, Q85, Q88 |
| "minimum spanning tree" | Kruskal (sort+DSU) or Prim (heap) | Q96, Q97 |
| "edge whose removal disconnects" | Tarjan's bridges (disc/low) | Q98 |
| "use every edge once" (Eulerian path) | Hierholzer's algorithm | Q99 |
| "tree → distances in any direction" | Build parent map, then BFS | Q32 |

**The four questions to ask yourself on any tree/graph problem:**
1. Tree: does the parent need its children's results first? → post-order. Do I need things grouped by depth? → BFS.
2. Graph: unweighted shortest path? → BFS. Dependencies / ordering? → topological sort. Weighted shortest path? → Dijkstra (non-negative) or Bellman-Ford (negative).
3. Am I repeatedly asking "are these two things connected / in the same group"? → Union-Find.
4. Is it secretly a graph? (grids, word transformations, course prerequisites, account merges are all graphs.) Model the nodes and edges first, then pick a template.

---

Created **03-dsa-trees-and-graphs.md** — 100 questions covered (34 binary trees, 13 BSTs, 13 heaps/priority queues, 7 tries, 33 graphs), each with brute→optimal, complete Java, dry runs, complexity, follow-ups, and rejection-causing mistakes, plus reusable BFS / DFS / topological-sort / Union-Find templates. Ready for next? (Next up: **04-dsa-dynamic-programming.md**.)
