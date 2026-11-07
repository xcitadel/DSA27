# 🔗 Lab 5: Linked Lists — Your Quiz Survival Guide

> **The short version:** the professor says Lab 5 is the base, but the main focus is likely an **operation on a linked list**. The exact question is not known.
>
> **بالمصري:** المطلوب غالباً تعمل operation على الـ linked list. أهم حاجة تفهم الأسهم بتتغير إزاي، مش تحفظ الكود بس.

## 🧭 Where do I start?

| If you have… | Do this |
|---|---|
| **10 minutes** | Read the visual crash course immediately below, then memorize the edge-case checklist. |
| **30 minutes** | Also practice [pointer diagrams](#4-the-pointer-diagrams-you-must-be-able-to-draw) and [common bugs](#9-common-bugs-to-recognize-instantly). |
| **1 hour or more** | Try the [mock quiz](#10-mock-quiz--try-before-checking-answers), then practice the [extra operations](#6-additional-likely-operation-scenarios). |

**Don't start by memorizing the whole class.** First understand the four pictures below. The full code is folded away in section 3 when you need it.

---

## 🚀 Visual crash course: the 4 moves

### First: what are we looking at?

```text
head → [ 10 | next ] → [ 20 | next ] → [ 30 | None ]
          value              value              value
```

- A **node** is a box with a `value` and a link called `next`.
- `head` tells us where the list starts; `None` means there is no next node.
- `self._size` counts nodes. Add 1 when you insert; subtract 1 when you delete.
- **مهم:** `node = node.next` moves *your pointer*. It does **not** delete anything.

### 🟦 Move 1 — Walk through every node

```text
head → [10] → [20] → [30] → None
         ↑
       node  → move right one step at a time
```

```python
node = self.head
while node is not None:
    print(node.value)     # use this node
    node = node.next      # go to the next node
```

Use this pattern to **search, count, sum, or print**. It works even if the list is empty.

### 🟩 Move 2 — Insert a node

**Example:** put `15` after `10`.

```text
BEFORE:   head → [10] ─────────→ [20] → None
                    new [15]     (not connected yet)

STEP 1:   new.next = node.next    # 15 points to 20
STEP 2:   node.next = new         # 10 points to 15

AFTER:    head → [10] → [15] → [20] → None
```

```python
# node already points to 10
new = Node(15)
new.next = node.next
node.next = new
self._size += 1
```

**Why that order?** If you overwrite `node.next` first without saving the old link, you can lose the rest of the list. For insertion at the **head**, use `new.next = self.head`, then `self.head = new`.

### 🟥 Move 3 — Delete a node

**Example:** delete `20`, where `prev` points to `10`.

```text
BEFORE:   head → [10] → [20] → [30] → None
                  ↑
                 prev

          prev.next = prev.next.next

AFTER:    head → [10] ───────→ [30] → None
```

```python
prev.next = prev.next.next
self._size -= 1
```

**Special case:** To delete the *first* node, there is no previous node: use `self.head = self.head.next`. Check that `self.head` is not `None` first.

### 🟨 Move 4 — Reverse the links

```text
START:    None    1 → 2 → 3 → None
          prev    node

STEP 1:   None ← 1    2 → 3 → None
                 prev  node

STEP 2:   None ← 1 ← 2    3 → None
                      prev  node

DONE:     head → 3 → 2 → 1 → None
```

```python
prev = None
node = self.head
while node is not None:
    following = node.next  # 1. save the rest
    node.next = prev       # 2. reverse the arrow
    prev = node            # 3. move prev forward
    node = following       # 4. move node forward
self.head = prev
```

🧠 **Memory phrase:** save next → flip arrow → move both → change head. Size stays the same.

### ✅ Before you hand in any operation

```text
[ ] Empty list?       [ ] One node?         [ ] Head changes?
[ ] Tail changes?     [ ] Value missing?    [ ] Duplicates?
[ ] _size correct?    [ ] Return / exception matches the question?
```

---

## 📚 Full study guide

## 1. What the professor's message means

> «اللاب الخامس ككل كـ base، لكن في الأساس هيبقى operation تتطبق على Linked List»

My interpretation: **all of Lab 5 is foundational, but the main question will likely ask you to apply or implement an operation on a linked list.** This is not a guarantee of the exact question or quiz format. Be ready to write code, trace pointers, handle edge cases, and explain complexity. No one can predict every possible operation, but the patterns below cover a wide range.

### Source material in this repository
- `docs/labs/lab05-linked-lists.md`: the actual lab, with checkpoints and answers.
- `dsa/linked_list.py`: the starter class and method contracts. The methods are intentionally unfinished.
- `solutions/dsa/linked_list.py`: the official reference implementation.
- `tests/test_linked_list.py`: the graded lab tests.
- `docs/question-bank/week05-questions.md` and `week05-answers.md`: extra questions and solutions.
- `notebooks/05-linked-list.ipynb`: linked-list notebook.

**Highest priority:** traversal, insert, delete, reverse, empty/head/tail cases, and `_size`. Next: variations that combine those patterns. If short on time, learn sections 2–6 and do the mock quiz in section 10.

## 2. Mental model: values versus nodes

```text
head
  ↓
[3 | next] → [7 | next] → [1 | None]
```

Each `Node` contains `value` and `next`. `head` refers to the first **node**, not its value. The last node's `next` is `None`. An empty list has `head is None` and `_size == 0`.

```python
node = self.head          # a reference to a node, possibly None
node.value                # value in that node
node.next                 # next node, possibly None
node = node.next          # move the reference; does NOT remove a node
node.next = node.next.next  # changes a link; skips a node
```

`node.next` is invalid when `node is None`. `node.next.value` is invalid when `node.next is None`. Python `and` short-circuits, so check existence first:

```python
while node is not None and node.next is not None:
    ...
```

**Golden rule:** before changing a link, save anything you still need. Draw the before and after arrows. When a node is removed it becomes unreachable from `head`, but other references might still point to it; Python reclaims it only when it is no longer reachable.

### Which loop?

```python
# Visit EVERY node, including the last; safe for an empty list.
node = self.head
while node is not None:
    print(node.value)
    node = node.next

# Stop ON the last node; only safe if node is not None at the start.
node = self.head
if node is not None:
    while node.next is not None:
        node = node.next
```

The second loop does not itself process the last node inside the loop. Use it to find the last node for appending, not for a normal search.

## 3. Lab 5 core operations — reference implementation

The following class uses the **exact Lab 5 conventions**: singly linked, no tail or sentinel, zero-based positions, and `_size` maintained. It is standalone, so paste it into a `.py` file to practice. The `__iter__` is included so `list(ll)` works.

<details>
<summary><strong>▶ Open the complete runnable reference class</strong> (read the pictures first)</summary>

```python
class Node:
    def __init__(self, value, next=None):
        self.value = value
        self.next = next

class LinkedList:
    def __init__(self, values=()):
        self.head = None
        self._size = 0
        for value in values:
            self.append(value)

    def __len__(self):
        return self._size

    def __iter__(self):
        node = self.head
        while node is not None:
            yield node.value
            node = node.next

    def push_front(self, value):
        new = Node(value)
        new.next = self.head       # 1. link to OLD head
        self.head = new            # 2. move head
        self._size += 1

    def append(self, value):
        new = Node(value)
        if self.head is None:
            self.head = new
        else:
            node = self.head
            while node.next is not None:
                node = node.next
            node.next = new
        self._size += 1

    def find(self, value):
        node = self.head
        index = 0
        while node is not None:
            if node.value == value:
                return index       # first match
            node = node.next
            index += 1
        return -1

    def __getitem__(self, index):
        if not 0 <= index < self._size:
            raise IndexError(index)
        node = self.head
        for _ in range(index):
            node = node.next
        return node.value

    def insert_at(self, index, value):
        if not 0 <= index <= self._size:
            raise IndexError(index)
        if index == 0:
            self.push_front(value)  # already updates _size
            return
        prev = self.head
        for _ in range(index - 1):
            prev = prev.next
        new = Node(value)
        new.next = prev.next        # keep the rest of the chain
        prev.next = new
        self._size += 1

    def pop_front(self):
        if self.head is None:
            raise IndexError('pop from empty list')
        value = self.head.value
        self.head = self.head.next
        self._size -= 1
        return value

    def remove(self, value):
        if self.head is None:
            return False
        if self.head.value == value:
            self.pop_front()        # already updates _size
            return True
        prev = self.head
        while prev.next is not None:
            if prev.next.value == value:
                prev.next = prev.next.next
                self._size -= 1
                return True         # remove first match only
            prev = prev.next
        return False

    def reverse(self):
        prev = None
        node = self.head
        while node is not None:
            following = node.next   # save before changing the link
            node.next = prev
            prev = node
            node = following
        self.head = prev
```

</details>

### Contracts to memorize

| Method | Result / special rule |
|---|---|
| `push_front(x)` | add x at head, returns `None` |
| `append(x)` | add x at end, returns `None` |
| `insert_at(i, x)` | valid **0 ≤ i ≤ size**, including insertion at end; otherwise `IndexError` |
| `pop_front()` | removes **and returns value**; empty → `IndexError` |
| `remove(x)` | removes **first** match, returns `True`; missing → `False` |
| `find(x)` | index of first match; missing → `-1` |
| `ll[i]` | valid **0 ≤ i < size**; otherwise `IndexError`; negative indices rejected in this lab |
| `reverse()` | changes links in place, no new nodes, returns `None`, size unchanged |

A professor may define different contracts on a quiz. Follow the question if it specifies different indexing, return values, duplicate policy, or list structure.

## 4. The pointer diagrams you must be able to draw

**Insert 9 after 3:**

```text
Before: prev → 3 → 7 → 1 → None
new.next = prev.next:  9 → 7 → 1 → None
prev.next = new:       3 → 9 → 7 → 1 → None
```

Do not set `prev.next = new` first and then `new.next = prev.next`: that makes 9 point to itself and loses the rest unless you saved it separately. For inserting at index `i > 0`, stop at `i - 1`. Index 0 changes `head`.

**Delete the node after `prev`:**

```text
Before: prev → 3 → 7 → 1 → None
prev.next = prev.next.next
After:  prev → 3 → 1 → None
```

The head has no predecessor in this lab: if the head itself is to be deleted, use `head = head.next`. Update `_size` exactly once per removed node.

**Reverse 1 → 2 → 3:**

| Stage | `prev` chain | `node` (remaining chain) |
|---|---|---|
| Start | None | 1 → 2 → 3 → None |
| After 1 | 1 → None | 2 → 3 → None |
| After 2 | 2 → 1 → None | 3 → None |
| After 3 | 3 → 2 → 1 → None | None |

Finally set `head = prev`. Remember: **save next → reverse link → move prev → move node**. Empty and one-node lists work without a special branch.

## 5. How to solve an unfamiliar operation

1. Clarify input/output: class method or `head` function? Return a value, new head, boolean, or nothing? First match or all? Is indexing zero-based?
2. Draw empty, one-node, and a 3–4-node example. Where must `head` change?
3. Choose a pattern: one traversal, previous/current, fast/slow, or two references separated by k.
4. Write the loop guard before accessing `.next`. Save the remaining chain before rewiring.
5. If using the lab class, maintain `_size` for every addition/removal, and only then. A raw-head function cannot automatically update a separate list object's `_size`.
6. Trace empty, singleton, head, middle, tail, not found, duplicates, invalid indices. Count time and extra space.

## 6. Additional likely operation scenarios

These are **practice possibilities, not predictions**. The following are methods for the class above. Paste them inside the class, at the same indentation as `find`. They use the same no-tail convention. Unless stated, they return `None` for mutations. In an exam, use whatever signature the professor asks for.

### A. Count / sum / maximum / search

```python
def count(self, value):
    result = 0
    node = self.head
    while node is not None:
        if node.value == value:
            result += 1
        node = node.next
    return result

def sum_values(self):
    total = 0  # assumes numeric values
    node = self.head
    while node is not None:
        total += node.value
        node = node.next
    return total

def maximum(self):
    if self.head is None:
        raise ValueError('empty list')
    best = self.head.value
    node = self.head.next
    while node is not None:
        if node.value > best:
            best = node.value
        node = node.next
    return best
```

Time O(n), extra space O(1). Using `best = 0` is wrong for an all-negative list. `count` returns 0 and `sum_values` returns 0 for empty lists.

### B. Insert after the first matching value

```python
def insert_after(self, target, value):
    node = self.head
    while node is not None:
        if node.value == target:
            new = Node(value, node.next)
            node.next = new
            self._size += 1
            return True
        node = node.next
    return False
```

Works after the tail too: its old next is `None`. Empty or absent → `False`, no change. O(n) to search, O(1) to link once the node is known.

### C. Delete at an index (return deleted value)

```python
def delete_at(self, index):
    if not 0 <= index < self._size:
        raise IndexError(index)
    if index == 0:
        return self.pop_front()
    prev = self.head
    for _ in range(index - 1):
        prev = prev.next
    victim = prev.next
    prev.next = victim.next
    self._size -= 1
    return victim.value
```

Deleting the last item works because `victim.next` is `None`. O(n) worst case.

### D. Delete all matches (including consecutive matches)

```python
def remove_all(self, value):
    removed = 0
    while self.head is not None and self.head.value == value:
        self.head = self.head.next
        self._size -= 1
        removed += 1
    prev = self.head
    while prev is not None and prev.next is not None:
        if prev.next.value == value:
            prev.next = prev.next.next
            self._size -= 1
            removed += 1
            # Do not advance prev: its new next may also match.
        else:
            prev = prev.next
    return removed
```

Example: `[2, 2, 1, 2, 2]` removing 2 → `[1]`, returns 4. O(n), O(1) extra space.

### E. Delete the last node

```python
def pop_back(self):
    if self.head is None:
        raise IndexError('pop from empty list')
    if self.head.next is None:
        return self.pop_front()
    prev = self.head
    while prev.next.next is not None:
        prev = prev.next
    value = prev.next.value
    prev.next = None
    self._size -= 1
    return value
```

Must find the **second-to-last** node. Even a tail pointer does not give you its predecessor in a singly linked list. O(n) worst case.

### F. Find the middle (slow/fast)

```python
def middle(self):
    if self.head is None:
        raise ValueError('empty list')
    slow = fast = self.head
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
    return slow.value
```

`[1, 2, 3, 4, 5]` → 3; `[1, 2, 3, 4]` → 3 (the **second** middle). If asked for first middle, change the convention and loop accordingly. O(n) time, O(1) extra space.

### G. Remove duplicates in a SORTED list

```python
def remove_duplicates_sorted(self):
    node = self.head
    while node is not None and node.next is not None:
        if node.value == node.next.value:
            node.next = node.next.next
            self._size -= 1
        else:
            node = node.next
```

`[1, 1, 1, 2, 3, 3]` → `[1, 2, 3]`. Only correct for duplicates that are adjacent; sorted input ensures that. O(n), O(1) extra space.

### H. kth from the end, 1-based

```python
def kth_from_end(self, k):
    if k < 1:
        raise IndexError(k)
    lead = self.head
    for _ in range(k):
        if lead is None:
            raise IndexError(k)
        lead = lead.next
    trail = self.head
    while lead is not None:
        lead = lead.next
        trail = trail.next
    return trail.value
```

`[10, 20, 30]`: k=1 → 30, k=3 → 10, k=4 or 0 → `IndexError`. O(n), O(1) space. If the question uses zero-based k, adjust accordingly.

### I. Cycle detection

```python
def has_cycle(self):
    slow = fast = self.head
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:  # same node, not merely same value
            return True
    return False
```

O(n) time, O(1) extra space. Two distinct nodes can hold equal values; use `is` for node identity. A normal traversal never terminates on a reachable cycle.

### J. Merge two sorted chains (advanced)

This is a standalone **raw-node** function, not a class method. It reuses and rewires input nodes; the input chains should be sorted, acyclic, and disjoint. It does not maintain any `LinkedList` object's `_size`.

```python
def merge_sorted(first, second):
    dummy = Node(None)  # temporary sentinel, not part of returned chain
    tail = dummy
    while first is not None and second is not None:
        if first.value <= second.value:
            tail.next = first
            first = first.next
        else:
            tail.next = second
            second = second.next
        tail = tail.next
    tail.next = first if first is not None else second
    return dummy.next
```

Example: 1 → 4 and 2 → 3 → 5 becomes 1 → 2 → 3 → 4 → 5. O(n+m) time, O(1) extra space. The lab's class has no permanent sentinel; this temporary dummy simplifies the merge.

## 7. Edge-case checklist

| Situation | What to check |
|---|---|
| Empty list | Do not dereference `None`; correct return or exception? |
| One node | Does deleting it set head to `None` and size to 0? |
| Target is head | Update `head` rather than a nonexistent predecessor. |
| Target is tail | New last node's `next` must be `None`. |
| Target absent | Don't accidentally delete; leave size unchanged. |
| Repeated values | First or all? After removing, should the pointer stay put? |
| Position bounds | Insertion permits `size`; reading/deletion do not. |
| Reverse | Save old next; update head only after traversal; size unchanged. |
| With a tail pointer | Also update tail on empty/singleton, last deletion, reverse, etc. Lab core does not have one. |
| Cycle | Standard loops may run forever; only use cycle-safe algorithms when relevant. |

## 8. Complexity: assume n nodes and no tail pointer

| Operation | Time | Extra space |
|---|---:|---:|
| `len`, with maintained `_size` | O(1) | O(1) |
| push/pop front | O(1) | O(1) |
| append | O(n) worst case | O(1) |
| find, access by index, remove by value | O(n) worst case | O(1) |
| insert/delete at index | O(n) worst case | O(1) |
| insert after / delete after a node already held | O(1) link work | O(1) |
| reverse | O(n) | O(1) |
| traverse once | O(n) | O(1) |

Keeping a tail makes append O(1), but deleting the last node remains O(n) in a singly linked list: you need to find its predecessor. Python-style indexed loops `for i in range(len(ll)): ll[i]` are O(n²), because each lookup starts at the head. `for value in ll:` is O(n). Constructing n nodes with repeated `append` and no tail is also O(n²); pushing front n times and reversing once is O(n).

Array contrast: index access O(1), insertion at front O(n), end append amortized O(1). Linked lists trade direct indexing and memory locality for easy link changes at known positions.

## 9. Common bugs to recognize instantly

1. **`self.head = new; new.next = self.head`** → new points to itself, old chain is lost from head. Link new to old head first.
2. **Search with `while node.next is not None`** → skips checking last node and crashes on empty. Use `while node is not None`.
3. **`prev.next = new; new.next = prev.next`** → self-cycle. Save/link old next first.
4. **During reverse, `node.next = prev; node = node.next`** → moves backward to `prev`, losing the unprocessed chain. Save `following` first.
5. **Increment/decrement size twice** when calling `push_front` or `pop_front` from another method. The helper already changes size.
6. **Using `==` instead of `is` for node identity in cycle detection** can compare values/equality semantics rather than reference identity.
7. **Advancing after removing a duplicate** can skip consecutive matches.
8. **Setting `node = node.next` expecting deletion** merely moves a local reference; change `head` or the previous node's `next` instead.

## 10. Mock quiz — try before checking answers

Assume the core class above. Do not look below the line until you finish.

1. Trace: start `[1, 2, 3]`; `push_front(0)`; `insert_at(2, 9)`; `pop_front()`; `remove(2)`; `append(5)`. Give final list, returned value of pop, length, and `find(3)`.
2. Given `head → 4 → 8 → 6 → None`, write two assignments to insert 5 after 4 without losing 8.
3. Write a method that returns the number of even values. Handle the empty list.
4. Write a method to remove the first occurrence of a value. Specify the result when absent, and keep `_size` accurate.
5. What is wrong with `while node.next is not None: ...` when searching? Give two distinct cases.
6. Trace reverse for `1 → 2 → 3`. After two iterations, what are the `prev` chain and `node` chain?
7. On `[7, 7, 8, 7]`, what should `remove(7)` and `remove_all(7)` each produce, starting from the original list independently?
8. For size 3, which of `insert_at(3, 5)`, `ll[3]`, and `insert_at(-1, 5)` are valid? Evaluate each independently.
9. Why is repeated indexed access to all nodes O(n²)? How do you get O(n)?
10. A list has `[1, 2, 3, 4]`; what does the middle method in section 6 return? What if empty?
11. Implement deletion of the last node. What happens on empty and singleton lists?
12. What does a tail reference improve? Can it alone make deleting the last node O(1) in a singly linked list?

<details>
<summary><strong>▶ Reveal mock quiz answers</strong></summary>

### Answers

1. `[1, 9, 3, 5]`; pop returns `0`; length 4; `find(3)` = 2.
2. `new = Node(5)`; `new.next = head.next`; `head.next = new` (plus the construction). In the class, increase `_size`.
3. Same as `count` in section 6, but test `node.value % 2 == 0`; start count at 0 and traverse while node is not None.
4. See `remove` in section 3. Missing returns `False`, match returns `True` and decreases size once.
5. Empty list raises `AttributeError`; a nonempty search never checks the last node.
6. `prev`: 2 → 1 → None; `node`: 3 → None.
7. `remove`: `[7, 8, 7]`, returns `True`. `remove_all`: `[8]`, returns 3.
8. Only insertion at 3 is valid. Access at 3 and negative insertion raise `IndexError` in the Lab 5 contract.
9. Each access walks from head: 0+1+…+(n−1). Iterate directly with `for value in ll` or a node pointer.
10. 3, the second middle; empty raises `ValueError` under that example's contract.
11. See `pop_back` in section 6: empty raises `IndexError`, singleton delegates to `pop_front`, otherwise find second-to-last.
12. Append becomes O(1); deleting the last is still O(n) without a predecessor link.

</details>

## 11. Practice workflow

1. Reproduce `push_front`, `append`, `remove`, `insert_at`, and `reverse` on paper without looking.
2. Solve mock questions 1–12, including writing methods from memory. Draw pointers for questions 2, 4, and 6.
3. Use `docs/labs/lab05-linked-lists.md` checkpoints and `docs/question-bank/week05-questions.md`; check answers only after attempting.
4. To practice with course tests, implement the methods in `dsa/linked_list.py`, then run `pytest tests/test_linked_list.py -q`. The starter deliberately raises `NotImplementedError`, and most tests will fail until you implement it. Compare against `solutions/dsa/linked_list.py` **after** trying. You can also practice with the standalone class in section 3 without modifying the course starter.
5. Repeat with empty, singleton, first, middle, last, absent, and repeated values. Aim to explain every pointer change aloud.
