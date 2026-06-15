# LeetCode 94 — Binary Tree Inorder Traversal

🔗 https://leetcode.com/problems/binary-tree-inorder-traversal/

**Problem:** Return the inorder (left → root → right) traversal of a binary tree's node values as a list.

> **Key insight:** Recursion is trivial; interviewers want to see the iterative stack version. Morris Traversal is the O(1)-space flex.

---

## Approach 1 — Brute Force | Recursive DFS

### Algorithm
1. Base case: if the current node is null, return immediately.
2. Recurse into the **LEFT** subtree (go as deep left as possible).
3. **Record (visit)** the current node's value.
4. Recurse into the **RIGHT** subtree.
5. The language's call stack handles all backtracking for free.

### Pseudocode
```
function helper(node, result):
    if node is null → return
    helper(node.left,  result)     // ① go left
    result.append(node.val)        // ② visit
    helper(node.right, result)     // ③ go right

function inorderTraversal(root):
    result ← empty list
    helper(root, result)
    return result
```

### How to Remember
**Mnemonic:** *"Left sandwich first, then Me, then Right — LMR"*

Picture yourself eating a sandwich: you always bite the Left side first, then the Middle (yourself), then the Right side. The recursion stack is your napkin — it remembers where you left off so you can come back and finish the right side.

### Complexity
| | |
|---|---|
| **Time** | O(n) — every node is visited exactly once |
| **Space** | O(h) — recursion call stack depth equals tree height h *(O(log n) balanced, O(n) worst-case skewed)* |

### Code
```cpp
#include <vector>
using namespace std;

// Definition for a binary tree node (provided by LeetCode).
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        dfs(root, result);
        return result;
    }

private:
    void dfs(TreeNode* node, vector<int>& result) {
        if (!node) return;               // base case
        dfs(node->left,  result);        // ① left subtree
        result.push_back(node->val);     // ② visit node
        dfs(node->right, result);        // ③ right subtree
    }
};
```

---

## Approach 2 — Optimized | Iterative with Explicit Stack

### Algorithm
1. Use an explicit stack to simulate what the call stack does in the recursive approach.
2. Maintain a pointer `curr` starting at root.
3. Loop while `curr` is non-null **OR** the stack is non-empty:
   - **DRILL LEFT:** push `curr` onto the stack, move `curr` left, repeat until `curr` is null.
   - **BACKTRACK:** pop the top node from the stack, visit it (record its value).
   - **PIVOT RIGHT:** set `curr` to the popped node's right child and repeat.
4. When both `curr` is null and the stack is empty, traversal is complete.

### Pseudocode
```
stack   ← empty
curr    ← root
result  ← empty list

while curr ≠ null OR stack not empty:
    while curr ≠ null:           // drill left phase
        push curr onto stack
        curr ← curr.left
    curr ← stack.pop()           // backtrack
    result.append(curr.val)      // visit
    curr ← curr.right            // pivot right

return result
```

### How to Remember
**Mnemonic:** *"Drill Down, Pop Up, Pivot Right — DPP"*

Think of a submarine diving straight down (drill left), then surfacing (pop), blowing its horn (record value), then turning one notch right and diving again. No GPS (call stack) needed — the explicit stack is the submarine's depth log.

### Complexity
| | |
|---|---|
| **Time** | O(n) — each node is pushed and popped exactly once |
| **Space** | O(h) — stack holds at most h nodes simultaneously *(O(log n) balanced, O(n) worst-case skewed)* |

### Code
```cpp
#include <vector>
#include <stack>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode*> stk;
        TreeNode* curr = root;

        while (curr || !stk.empty()) {
            // Phase 1: drill as far left as possible
            while (curr) {
                stk.push(curr);
                curr = curr->left;
            }
            // Phase 2: backtrack to nearest unvisited ancestor
            curr = stk.top();
            stk.pop();
            result.push_back(curr->val);  // visit

            // Phase 3: pivot to right subtree
            curr = curr->right;
        }

        return result;
    }
};
```

---

## Approach 3 — Most Optimized | Morris Traversal (O(1) Space)

### Algorithm
Morris Traversal threads the tree temporarily, eliminating the need for any stack or recursion — true O(1) extra space.

1. Start with `curr = root`.
2. While `curr` is not null:
   - If `curr` has **no left child** → visit `curr.val`, move `curr` to `curr.right`
   - If `curr` **has a left child** → find curr's **inorder predecessor** (rightmost node in left subtree):
     - If `predecessor.right` is null (not yet threaded): set `predecessor.right = curr` *(create thread)*, move `curr` to `curr.left`
     - If `predecessor.right == curr` (already threaded): restore `predecessor.right = null` *(remove thread)*, visit `curr.val`, move `curr` to `curr.right`
3. Temporary threads allow upward traversal without a stack; they are always restored, leaving the tree unchanged.

### Pseudocode
```
curr   ← root
result ← empty list

while curr ≠ null:
    if curr.left is null:
        result.append(curr.val)
        curr ← curr.right
    else:
        pred ← curr.left
        while pred.right ≠ null AND pred.right ≠ curr:
            pred ← pred.right

        if pred.right is null:
            pred.right ← curr         // thread forward
            curr ← curr.left
        else:
            pred.right ← null         // remove thread
            result.append(curr.val)
            curr ← curr.right

return result
```

### How to Remember
**Mnemonic:** *"Leave a sticky note going down, rip it off coming back"*

Imagine hiking a forest maze with no map. Before descending left, you tape a note on the furthest-right tree of that path pointing back to you. When you stumble upon your own note, you tear it off (restore the tree), log the spot (visit), and head right. Zero backpack (stack) needed — just sticky notes.

### Complexity
| | |
|---|---|
| **Time** | O(n) — each node is visited at most twice (thread + unthread); total work is still O(n) |
| **Space** | O(1) — no stack, no recursion; reuses existing null right-child pointers in the tree |

### Code
```cpp
#include <vector>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        TreeNode* curr = root;

        while (curr) {
            if (!curr->left) {
                // No left child: visit and move right
                result.push_back(curr->val);
                curr = curr->right;
            } else {
                // Find inorder predecessor (rightmost in left subtree)
                TreeNode* pred = curr->left;
                while (pred->right && pred->right != curr) {
                    pred = pred->right;
                }

                if (!pred->right) {
                    // First visit: thread predecessor back to curr
                    pred->right = curr;
                    curr = curr->left;
                } else {
                    // Second visit: restore tree, then visit curr
                    pred->right = nullptr;
                    result.push_back(curr->val);
                    curr = curr->right;
                }
            }
        }

        return result;
    }
};
```

---

## Quick Reference Summary

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| Recursive DFS | O(n) | O(h) | LMR via language call stack |
| Iterative Stack | O(n) | O(h) | Drill left, pop, pivot right |
| Morris Traversal | O(n) | O(1) | Temporary predecessor threads |

**Variables:** `n` = number of nodes &nbsp;|&nbsp; `h` = tree height
- `h = O(log n)` for a balanced tree
- `h = O(n)` for a completely skewed tree

> **Interview tip:** Lead with the iterative stack (Approach 2) to show you grasp the mechanics; mention Morris Traversal only if the interviewer probes for O(1) space — it's impressive but error-prone under time pressure.
