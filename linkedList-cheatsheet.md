# Linked Lists in C++ — Coding Interview Cheat Sheet

Linked list problems are about **pointer manipulation**. Master a handful of patterns — dummy node, two pointers, reversal — and 90% of interview questions fall out.

---

## 0. The Node Definition

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};
```

---

## 1. Core Techniques (learn these cold)

### 1a. Dummy (sentinel) head — avoid special-casing the head

```cpp
ListNode dummy(0);
dummy.next = head;
ListNode* prev = &dummy;
// ... work with prev->next ...
return dummy.next;    // new head
```
Use whenever the head might change (deletion, insertion, merge).

### 1b. Two pointers — slow & fast

```cpp
ListNode* slow = head;
ListNode* fast = head;
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
}
// slow is now at the middle
```
Used for: middle of list, cycle detection, nth-from-end.

### 1c. Iterative reversal (the single most important pattern)

```cpp
ListNode* prev = nullptr;
ListNode* cur  = head;
while (cur) {
    ListNode* nxt = cur->next;   // save
    cur->next = prev;            // reverse pointer
    prev = cur;                  // advance
    cur  = nxt;
}
return prev;                     // new head
```

---

## 2. Reversal Patterns

### Reverse whole list (recursive)

```cpp
ListNode* reverse(ListNode* head) {
    if (!head || !head->next) return head;
    ListNode* newHead = reverse(head->next);
    head->next->next = head;
    head->next = nullptr;
    return newHead;
}
```

### Reverse a sublist [left, right] (LC 92)

```cpp
ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0); dummy.next = head;
    ListNode* prev = &dummy;
    for (int i = 1; i < left; ++i) prev = prev->next;
    ListNode* cur = prev->next;
    for (int i = 0; i < right - left; ++i) {   // head-insertion
        ListNode* nxt = cur->next;
        cur->next = nxt->next;
        nxt->next = prev->next;
        prev->next = nxt;
    }
    return dummy.next;
}
```

---

## 3. Two-Pointer Patterns

### Cycle detection — Floyd's tortoise & hare (LC 141)

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head; ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Find cycle start (LC 142)

```cpp
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head; ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next; fast = fast->next->next;
        if (slow == fast) {                 // meeting point
            ListNode* p = head;
            while (p != slow) { p = p->next; slow = slow->next; }
            return p;                       // cycle start
        }
    }
    return nullptr;
}
```

### Nth node from end (LC 19) — gap of n

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0); dummy.next = head;
    ListNode* fast = &dummy; ListNode* slow = &dummy;
    for (int i = 0; i < n; ++i) fast = fast->next;   // advance gap
    while (fast->next) { fast = fast->next; slow = slow->next; }
    slow->next = slow->next->next;                   // delete
    return dummy.next;
}
```

---

## 4. Merging & Sorting

### Merge two sorted lists (LC 21)

```cpp
ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
    ListNode dummy(0); ListNode* tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else                  { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;      // attach remainder
    return dummy.next;
}
```

### Merge sort on a list (LC 148) — O(n log n), O(1) aux

```cpp
ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;
    // split at middle
    ListNode* slow = head; ListNode* fast = head->next;
    while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
    ListNode* mid = slow->next; slow->next = nullptr;
    return mergeTwoLists(sortList(head), sortList(mid));
}
```

---

## 5. Common Bugs & Interview Tips

- **Null checks**: always guard `cur && cur->next` before `cur->next->next`.
- **Lost pointers**: save `next` *before* rewiring, or you'll orphan the rest of the list.
- **Use a dummy** whenever the head can change — eliminates edge cases.
- **Draw it**: sketch 3–4 nodes and move arrows by hand; pointer bugs vanish.
- **Memory**: on LeetCode you can leak deleted nodes, but mention `delete` in interviews.
- **Off-by-one**: for "nth from end", the gap technique needs a dummy to handle removing the head.
- **Two speeds**: `fast = head->next` vs `fast = head` changes which middle you land on (even-length lists) — pick deliberately.

---

## 6. Complexity Quick Reference

| Operation                     | Time      | Space |
|-------------------------------|-----------|-------|
| Traverse / search             | O(n)      | O(1)  |
| Reverse (iterative)           | O(n)      | O(1)  |
| Reverse (recursive)           | O(n)      | O(n)  |
| Cycle detection (Floyd)       | O(n)      | O(1)  |
| Merge two sorted lists        | O(n + m)  | O(1)  |
| Merge sort a list             | O(n log n)| O(log n) stack |
| Merge k lists (heap)          | O(N log k)| O(k)  |

---

## 7. Must-Practice LeetCode Problems

| # | Problem | Pattern |
|---|---------|---------|
| 206 | Reverse Linked List | iterative/recursive reversal |
| 21 | Merge Two Sorted Lists | dummy + merge |
| 141 | Linked List Cycle | fast/slow |
| 142 | Linked List Cycle II | fast/slow + math |
| 19 | Remove Nth From End | gap two-pointer |
| 876 | Middle of the Linked List | fast/slow |
| 234 | Palindrome Linked List | reverse half + compare |
| 92 | Reverse Linked List II | sublist reversal |
| 25 | Reverse Nodes in k-Group | grouped reversal |
| 143 | Reorder List | mid + reverse + merge |
| 2 | Add Two Numbers | dummy + carry |
| 138 | Copy List with Random Pointer | hashmap / interleave |
| 148 | Sort List | merge sort |
| 23 | Merge k Sorted Lists | min-heap / divide & conquer |
| 160 | Intersection of Two Lists | length diff / two-pointer |
| 83 | Remove Duplicates (sorted) | in-place skip |
| 82 | Remove Duplicates II | dummy + skip runs |
| 328 | Odd Even Linked List | two chains |

---

## 8. Full Solutions

Each is LeetCode-ready (method inside a `Solution` class, using the `ListNode` above).

### 206. Reverse Linked List

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        while (head) {
            ListNode* nxt = head->next;
            head->next = prev;
            prev = head;
            head = nxt;
        }
        return prev;
    }
};
```

### 21. Merge Two Sorted Lists

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
        ListNode dummy(0); ListNode* tail = &dummy;
        while (a && b) {
            if (a->val <= b->val) { tail->next = a; a = a->next; }
            else                  { tail->next = b; b = b->next; }
            tail = tail->next;
        }
        tail->next = a ? a : b;
        return dummy.next;
    }
};
```

### 141. Linked List Cycle

```cpp
class Solution {
public:
    bool hasCycle(ListNode* head) {
        ListNode* slow = head; ListNode* fast = head;
        while (fast && fast->next) {
            slow = slow->next; fast = fast->next->next;
            if (slow == fast) return true;
        }
        return false;
    }
};
```

### 142. Linked List Cycle II

```cpp
class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        ListNode* slow = head; ListNode* fast = head;
        while (fast && fast->next) {
            slow = slow->next; fast = fast->next->next;
            if (slow == fast) {
                ListNode* p = head;
                while (p != slow) { p = p->next; slow = slow->next; }
                return p;
            }
        }
        return nullptr;
    }
};
```

### 19. Remove Nth Node From End

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode dummy(0); dummy.next = head;
        ListNode* fast = &dummy; ListNode* slow = &dummy;
        for (int i = 0; i < n; ++i) fast = fast->next;
        while (fast->next) { fast = fast->next; slow = slow->next; }
        slow->next = slow->next->next;
        return dummy.next;
    }
};
```

### 876. Middle of the Linked List

```cpp
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        ListNode* slow = head; ListNode* fast = head;
        while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
        return slow;   // second middle for even length
    }
};
```

### 234. Palindrome Linked List

```cpp
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        // find middle
        ListNode* slow = head; ListNode* fast = head;
        while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
        // reverse second half
        ListNode* prev = nullptr;
        while (slow) { ListNode* nxt = slow->next; slow->next = prev; prev = slow; slow = nxt; }
        // compare
        ListNode* l = head; ListNode* r = prev;
        while (r) { if (l->val != r->val) return false; l = l->next; r = r->next; }
        return true;
    }
};
```

### 92. Reverse Linked List II

```cpp
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        ListNode dummy(0); dummy.next = head;
        ListNode* prev = &dummy;
        for (int i = 1; i < left; ++i) prev = prev->next;
        ListNode* cur = prev->next;
        for (int i = 0; i < right - left; ++i) {
            ListNode* nxt = cur->next;
            cur->next = nxt->next;
            nxt->next = prev->next;
            prev->next = nxt;
        }
        return dummy.next;
    }
};
```

### 25. Reverse Nodes in k-Group

```cpp
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        // check there are k nodes
        ListNode* node = head;
        for (int i = 0; i < k; ++i) {
            if (!node) return head;    // fewer than k → leave as is
            node = node->next;
        }
        // reverse first k
        ListNode* prev = reverseKGroup(node, k);  // recurse on the rest
        ListNode* cur = head;
        for (int i = 0; i < k; ++i) {
            ListNode* nxt = cur->next;
            cur->next = prev;
            prev = cur;
            cur = nxt;
        }
        return prev;
    }
};
```

### 143. Reorder List

```cpp
class Solution {
public:
    void reorderList(ListNode* head) {
        if (!head || !head->next) return;
        // 1. find middle
        ListNode* slow = head; ListNode* fast = head;
        while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
        // 2. reverse second half
        ListNode* second = slow->next; slow->next = nullptr;
        ListNode* prev = nullptr;
        while (second) { ListNode* nxt = second->next; second->next = prev; prev = second; second = nxt; }
        // 3. merge two halves
        ListNode* first = head;
        while (prev) {
            ListNode* n1 = first->next; ListNode* n2 = prev->next;
            first->next = prev; prev->next = n1;
            first = n1; prev = n2;
        }
    }
};
```

### 2. Add Two Numbers

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy(0); ListNode* tail = &dummy;
        int carry = 0;
        while (l1 || l2 || carry) {
            int sum = carry;
            if (l1) { sum += l1->val; l1 = l1->next; }
            if (l2) { sum += l2->val; l2 = l2->next; }
            carry = sum / 10;
            tail->next = new ListNode(sum % 10);
            tail = tail->next;
        }
        return dummy.next;
    }
};
```

### 138. Copy List with Random Pointer

```cpp
// Node has: int val; Node* next; Node* random;
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if (!head) return nullptr;
        // 1. interleave copies: A -> A' -> B -> B' ...
        for (Node* cur = head; cur; cur = cur->next->next) {
            Node* copy = new Node(cur->val);
            copy->next = cur->next;
            cur->next = copy;
        }
        // 2. assign randoms
        for (Node* cur = head; cur; cur = cur->next->next)
            if (cur->random) cur->next->random = cur->random->next;
        // 3. detach
        Node* dummy = new Node(0); Node* copyTail = dummy;
        for (Node* cur = head; cur; cur = cur->next) {
            copyTail->next = cur->next;
            copyTail = copyTail->next;
            cur->next = cur->next->next;   // restore original
        }
        return dummy->next;
    }
};
```

### 148. Sort List

```cpp
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head || !head->next) return head;
        ListNode* slow = head; ListNode* fast = head->next;
        while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
        ListNode* mid = slow->next; slow->next = nullptr;
        return merge(sortList(head), sortList(mid));
    }
    ListNode* merge(ListNode* a, ListNode* b) {
        ListNode dummy(0); ListNode* tail = &dummy;
        while (a && b) {
            if (a->val <= b->val) { tail->next = a; a = a->next; }
            else                  { tail->next = b; b = b->next; }
            tail = tail->next;
        }
        tail->next = a ? a : b;
        return dummy.next;
    }
};
```

### 23. Merge k Sorted Lists

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
        for (ListNode* l : lists) if (l) pq.push(l);
        ListNode dummy(0); ListNode* tail = &dummy;
        while (!pq.empty()) {
            ListNode* top = pq.top(); pq.pop();
            tail->next = top; tail = top;
            if (top->next) pq.push(top->next);
        }
        return dummy.next;
    }
};
```

### 160. Intersection of Two Linked Lists

```cpp
class Solution {
public:
    ListNode* getIntersectionNode(ListNode* a, ListNode* b) {
        ListNode* pa = a; ListNode* pb = b;
        while (pa != pb) {                 // swap heads at the end
            pa = pa ? pa->next : b;
            pb = pb ? pb->next : a;
        }
        return pa;                         // intersection or nullptr
    }
};
```

### 83. Remove Duplicates from Sorted List

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* cur = head;
        while (cur && cur->next) {
            if (cur->val == cur->next->val) cur->next = cur->next->next;
            else cur = cur->next;
        }
        return head;
    }
};
```

### 82. Remove Duplicates from Sorted List II

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode dummy(0); dummy.next = head;
        ListNode* prev = &dummy; ListNode* cur = head;
        while (cur) {
            if (cur->next && cur->val == cur->next->val) {
                while (cur->next && cur->val == cur->next->val) cur = cur->next;
                prev->next = cur->next;       // skip entire run
            } else {
                prev = prev->next;
            }
            cur = cur->next;
        }
        return dummy.next;
    }
};
```

### 328. Odd Even Linked List

```cpp
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if (!head) return nullptr;
        ListNode* odd = head; ListNode* even = head->next; ListNode* evenHead = even;
        while (even && even->next) {
            odd->next = even->next;  odd = odd->next;
            even->next = odd->next;  even = even->next;
        }
        odd->next = evenHead;        // attach evens after odds
        return head;
    }
};
```

---

**One-liner to remember:** *Save next, rewire, advance.* Use a dummy head, and two pointers solve most of the rest.
