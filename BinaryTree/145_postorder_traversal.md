# LeetCode 145 — Binary Tree Postorder Traversal

🔗 https://leetcode.com/problems/binary-tree-postorder-traversal/

**Problem:** Return the postorder traversal (left → right → root) of a binary tree's node values.

> **Key insight:** Postorder is the reverse of a modified preorder (root → right → left), which is easier to simulate iteratively — exploit this symmetry.

---

## Approach 1 — Brute Force | Two Stacks

### Algorithm
1. Push root onto **Stack1** (the processing stack).
2. While Stack1 is not empty:
   - Pop a node from Stack1 and push it onto **Stack2** (the result stack).
   - Push left child (if exists) onto Stack1.
   - Push right child (if exists) onto Stack1.
3. Stack2 now holds nodes in reverse postorder (root → right → left from bottom to top).
4. Pop all nodes off Stack2 — that ordering is left → right → root = postorder.

**Why it works:** Stack1 visits nodes in root → right → left order (right child is pushed last so it's popped first). Stack2 reverses this sequence, yielding the exact postorder.

### Pseudocode
```
if root is null → return []

stack1 = [root]
stack2 = []
result = []

while stack1 not empty:
    node ← stack1.pop()
    stack2.push(node)
    if node.left  exists → stack1.push(node.left)
    if node.right exists → stack1.push(node.right)

while stack2 not empty:
    node ← stack2.pop()
    result.append(node.val)

return result
```

### How to Remember
**Mnemonic:** *"Pour into Cup, Pour into Glass"*

Imagine pouring water (nodes) into Cup (Stack1) and then into Glass (Stack2). Stack1 pours right-side-first; when you pour Stack2 into your mouth (result), the order flips to left-first. Two pours = two reversals = correct postorder.

### Complexity
| | |
|---|---|
| **Time** | O(n) — every node is pushed/popped exactly twice (once per stack) |
| **Space** | O(n) — Stack2 holds all n nodes before draining |

### Code
```cpp
#include <vector>
#include <stack>
using namespace std;

// Definition for a binary tree node.
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        if (!root) return result;

        stack<TreeNode*> stack1, stack2;
        stack1.push(root);

        while (!stack1.empty()) {
            TreeNode* node = stack1.top(); stack1.pop();
            stack2.push(node);                    // collect in reverse postorder

            if (node->left)  stack1.push(node->left);   // left pushed first
            if (node->right) stack1.push(node->right);  // right pushed second → popped first
        }

        // Drain stack2: bottom-to-top gives left → right → root
        while (!stack2.empty()) {
            result.push_back(stack2.top()->val);
            stack2.pop();
        }

        return result;
    }
};
```

---

## Approach 2 — Optimized | One Stack + Reverse Modified Preorder

### Algorithm
1. Use a single stack. Simulate a **modified preorder**: root → right → left.
2. Push root. While stack is not empty:
   - Pop node, append its value to result.
   - Push **left child first**, then **right child** (right sits on top → processed next).
3. **Reverse** the result array at the end.
4. Reversing root→right→left gives left→right→root = postorder.

**Why it works:** Classic preorder visits root→left→right by pushing right then left. Swap the push order (push left then right) to visit root→right→left. Reversing that yields postorder — one fewer stack, same idea.

### Pseudocode
```
if root is null → return []

stack  = [root]
result = []

while stack not empty:
    node ← stack.pop()
    result.append(node.val)

    if node.left  exists → stack.push(node.left)   // pushed first → under right
    if node.right exists → stack.push(node.right)  // pushed second → on top → popped next

reverse result
return result
```

### How to Remember
**Mnemonic:** *"Preorder's Evil Twin + Mirror"*

Normal preorder pushes right then left. This is the **evil twin**: push left then right → visit root→right→left. Then hold up a mirror (reverse) → postorder appears. One stack, one reverse, done.

### Complexity
| | |
|---|---|
| **Time** | O(n) — each node pushed/popped once; reversal is O(n) |
| **Space** | O(h) — stack depth equals tree height (O(n) worst case for skewed tree) |

### Code
```cpp
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        if (!root) return result;

        stack<TreeNode*> stk;
        stk.push(root);

        while (!stk.empty()) {
            TreeNode* node = stk.top(); stk.pop();
            result.push_back(node->val);

            // Push left before right so right is on top (processed next)
            // → gives root → right → left order
            if (node->left)  stk.push(node->left);
            if (node->right) stk.push(node->right);
        }

        // Mirror: root→right→left becomes left→right→root
        reverse(result.begin(), result.end());
        return result;
    }
};
```

---

## Approach 3 — Most Optimized | One Stack + Prev Pointer (True Iterative, No Reversal)

### Algorithm
1. Use one stack and a `prev` pointer (last node added to result).
2. `curr` starts at root. Outer loop runs while `curr != null` OR stack is non-empty.
3. **Go left:** while `curr` is not null, push it and walk left.
4. **Peek** at stack top:
   - If top has a **right child** AND that right child was **not just visited** (`prev`) → move `curr` to that right child (explore right subtree first).
   - Otherwise → the node is safe to process: pop it, add to result, set `prev = node`.
5. Repeat until both `curr` is null and stack is empty.

**Why it works:** A node can only be added to result after both subtrees are done. The `prev` pointer distinguishes "coming from right child (done)" vs "right child not yet visited" — the key decision gate.

### Pseudocode
```
result = []
stack  = []
curr   = root
prev   = null

while curr != null OR stack not empty:

    // Phase 1: walk all the way left, pushing nodes
    while curr != null:
        stack.push(curr)
        curr ← curr.left

    // Phase 2: decide to go right or process
    peek ← stack.top()

    if peek.right != null AND peek.right != prev:
        curr ← peek.right      // right subtree not done yet → explore it
    else:
        node ← stack.pop()
        result.append(node.val)
        prev ← node            // mark as last visited

return result
```

### How to Remember
**Mnemonic:** *"Left Wall, Peek, Guard the Right"*

Imagine walking along the **left wall** of each room (push while going left). When you hit a dead end, peek at the door. Is there an **unguarded right room** (`prev` isn't guarding it)? Enter it. Otherwise you're safe to **leave** (pop + record). The guard (`prev`) ensures you never re-enter a room you already exited.

### Complexity
| | |
|---|---|
| **Time** | O(n) — each node is pushed once and popped once |
| **Space** | O(h) — stack holds at most one path root-to-leaf; no extra O(n) storage (output excluded) |

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
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode*> stk;
        TreeNode* curr = root;
        TreeNode* prev = nullptr;

        while (curr != nullptr || !stk.empty()) {
            // Walk left, pushing every node
            while (curr != nullptr) {
                stk.push(curr);
                curr = curr->left;
            }

            TreeNode* peek = stk.top();

            // If right subtree exists and hasn't been visited yet → go right
            if (peek->right != nullptr && peek->right != prev) {
                curr = peek->right;
            } else {
                // Both subtrees done (or absent) → safe to process this node
                stk.pop();
                result.push_back(peek->val);
                prev = peek;   // mark as last processed to guard against re-entry
            }
        }

        return result;
    }
};
```

---

## Quick Reference Summary

| Approach | Time | Space (aux) | Key Idea |
|---|---|---|---|
| Two Stacks | O(n) | O(n) — stack2 holds all nodes | Double-reverse: Stack1 collects root→right→left, Stack2 reverses it |
| One Stack + Reverse | O(n) | O(h) — single stack | Modified preorder (root→right→left) then reverse result |
| One Stack + Prev Pointer | O(n) | O(h) — single stack, no reversal | `prev` guards right subtree; process node only when both children done |

**Variables:** n = total nodes  |  h = tree height (O(log n) balanced, O(n) skewed)

> **Interview tip:** Start with the Two-Stack approach — it's the easiest to explain and code under pressure. Then mention the One Stack + Reverse as a quick optimization. Offer the Prev Pointer approach only if asked for a solution with no result-reversal step, as it demonstrates deeper understanding of iterative tree traversal.
