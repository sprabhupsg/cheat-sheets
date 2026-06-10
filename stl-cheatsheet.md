# C++ STL Cheat Sheet

Comprehensive reference for the Standard Template Library — containers, iterators, algorithms, and utilities with complexity guarantees.

---

## Containers Overview

### Sequence Containers

| Container | Header | Description | Random Access | Ordered |
|---|---|---|:---:|:---:|
| `array<T, N>` | `<array>` | Fixed-size contiguous array | Yes | By index |
| `vector<T>` | `<vector>` | Dynamic contiguous array | Yes | By index |
| `deque<T>` | `<deque>` | Double-ended queue | Yes | By index |
| `list<T>` | `<list>` | Doubly-linked list | No | By insertion |
| `forward_list<T>` | `<forward_list>` | Singly-linked list | No | By insertion |

### Associative Containers (Sorted — Red-Black Tree)

| Container | Header | Description | Duplicates | Ordered |
|---|---|---|:---:|:---:|
| `set<T>` | `<set>` | Unique sorted keys | No | Sorted |
| `multiset<T>` | `<set>` | Sorted keys with duplicates | Yes | Sorted |
| `map<K, V>` | `<map>` | Unique key → value, sorted | No | Sorted by key |
| `multimap<K, V>` | `<map>` | Key → value with duplicate keys | Yes | Sorted by key |

### Unordered Containers (Hash Table)

| Container | Header | Description | Duplicates | Ordered |
|---|---|---|:---:|:---:|
| `unordered_set<T>` | `<unordered_set>` | Unique keys, hash-based | No | No |
| `unordered_multiset<T>` | `<unordered_set>` | Hash-based with duplicates | Yes | No |
| `unordered_map<K, V>` | `<unordered_map>` | Key → value, hash-based | No | No |
| `unordered_multimap<K, V>` | `<unordered_map>` | Hash key → value, duplicates | Yes | No |

### Container Adaptors

| Container | Header | Underlying | Description |
|---|---|---|---|
| `stack<T>` | `<stack>` | `deque` | LIFO stack |
| `queue<T>` | `<queue>` | `deque` | FIFO queue |
| `priority_queue<T>` | `<queue>` | `vector` | Max-heap by default |

---

## Container Complexity Table

### Sequence Containers

| Operation | `array` | `vector` | `deque` | `list` | `forward_list` |
|---|:---:|:---:|:---:|:---:|:---:|
| Access `[i]` | O(1) | O(1) | O(1) | O(n) | O(n) |
| `front()` | O(1) | O(1) | O(1) | O(1) | O(1) |
| `back()` | O(1) | O(1) | O(1) | O(1) | O(n) |
| `push_back` | — | O(1)* | O(1)* | O(1) | — |
| `pop_back` | — | O(1) | O(1) | O(1) | — |
| `push_front` | — | O(n) | O(1)* | O(1) | O(1) |
| `pop_front` | — | O(n) | O(1) | O(1) | O(1) |
| Insert middle | — | O(n) | O(n) | O(1)† | O(1)† |
| Erase middle | — | O(n) | O(n) | O(1)† | O(1)† |
| `find` (linear) | O(n) | O(n) | O(n) | O(n) | O(n) |

\* amortized · † given iterator to position

### Associative Containers (Sorted)

| Operation | `set` / `multiset` | `map` / `multimap` |
|---|:---:|:---:|
| `insert` | O(log n) | O(log n) |
| `erase(key)` | O(log n) | O(log n) |
| `find` | O(log n) | O(log n) |
| `count` | O(log n + k) | O(log n + k) |
| `lower_bound` / `upper_bound` | O(log n) | O(log n) |
| Iteration (in order) | O(n) | O(n) |

### Unordered Containers (Hash)

| Operation | Average | Worst Case |
|---|:---:|:---:|
| `insert` | O(1) | O(n) |
| `erase` | O(1) | O(n) |
| `find` | O(1) | O(n) |
| `count` | O(1) | O(n) |
| `operator[]` (map) | O(1) | O(n) |

### Container Adaptors

| Operation | `stack` | `queue` | `priority_queue` |
|---|:---:|:---:|:---:|
| `push` | O(1)* | O(1)* | O(log n) |
| `pop` | O(1) | O(1) | O(log n) |
| `top` / `front` | O(1) | O(1) | O(1) |

\* amortized

---

## vector

```cpp
#include <vector>

// Construction
vector<int> v;                        // empty
vector<int> v(10);                    // 10 elements, value-initialized (0)
vector<int> v(10, -1);               // 10 elements, all -1
vector<int> v{1, 2, 3, 4, 5};       // initializer list
vector<int> v(other.begin(), other.end());  // from range
vector<vector<int>> grid(m, vector<int>(n, 0));  // 2D grid m×n

// Size & capacity
v.size();                             // number of elements
v.empty();                            // true if size == 0
v.capacity();                         // allocated storage
v.reserve(100);                       // pre-allocate (avoids reallocation)
v.shrink_to_fit();                    // release unused capacity
v.resize(20);                         // resize, new elements value-initialized
v.resize(20, -1);                     // resize, new elements = -1

// Access
v[0];                                 // unchecked access — UB if out of range
v.at(0);                              // checked — throws out_of_range
v.front();                            // first element
v.back();                             // last element
v.data();                             // raw pointer to underlying array

// Modification
v.push_back(42);                      // append — O(1) amortized
v.emplace_back(42);                   // construct in-place — avoids copy
v.pop_back();                         // remove last — O(1)
v.insert(v.begin() + 2, 99);         // insert at index 2 — O(n)
v.erase(v.begin() + 2);              // erase at index 2 — O(n)
v.erase(v.begin(), v.begin() + 3);   // erase range [0, 3)
v.clear();                            // remove all elements
v.assign({10, 20, 30});              // replace contents

// Swap
v.swap(other);                        // O(1) swap contents
swap(v, other);                       // same via free function

// Comparison
v == other;                           // element-wise equality
v < other;                            // lexicographic comparison
```

---

## string

```cpp
#include <string>

// Construction
string s;                             // empty
string s(5, 'a');                     // "aaaaa"
string s = "hello";                   // from C-string
string s(other, 0, 3);               // substring: first 3 chars of other
string s(other.begin(), other.end()); // from range

// Size
s.size();  s.length();                // both return length
s.empty();
s.capacity();
s.reserve(100);

// Access
s[0];                                 // unchecked
s.at(0);                              // checked
s.front();  s.back();
s.c_str();                            // null-terminated C string
s.data();                             // same as c_str() since C++11

// Modification
s += " world";                        // append
s.append("!");                        // append
s.push_back('!');                     // append single char
s.pop_back();                         // remove last char
s.insert(5, " dear");                 // insert at position 5
s.erase(5, 5);                        // erase 5 chars starting at pos 5
s.replace(0, 5, "Hi");               // replace first 5 chars with "Hi"
s.clear();

// Search
size_t pos = s.find("world");         // first occurrence, or string::npos
s.rfind("o");                         // last occurrence
s.find_first_of("aeiou");            // first vowel position
s.find_last_of("aeiou");             // last vowel position
s.find_first_not_of(" \t\n");        // first non-whitespace

// Substring & comparison
s.substr(6, 5);                       // "world" — from pos 6, length 5
s.compare("hello");                   // <0, 0, >0
s.starts_with("he");                  // C++20
s.ends_with("ld");                    // C++20
s.contains("ell");                    // C++23

// Conversion
int n = stoi("42");                   // string → int
long l = stol("123456789");
double d = stod("3.14");
string ns = to_string(42);           // int → string
```

### String Complexity

| Operation | Complexity |
|---|:---:|
| `operator[]`, `at()` | O(1) |
| `push_back`, `pop_back` | O(1)* |
| `append`, `+=` | O(k)* where k = appended length |
| `insert`, `erase` | O(n) |
| `find`, `rfind` | O(n × m) worst case |
| `substr` | O(k) where k = substring length |
| `compare` | O(min(n, m)) |
| `c_str()`, `data()` | O(1) |

\* amortized

---

## array

```cpp
#include <array>

array<int, 5> a = {1, 2, 3, 4, 5};   // fixed size, stack-allocated
array<int, 5> a{};                     // zero-initialized

a.size();                              // 5 (constexpr)
a[0];  a.at(0);
a.front();  a.back();
a.data();                              // raw pointer
a.fill(0);                             // set all to 0
a.swap(other);

// Can be used in constexpr contexts
constexpr array<int, 3> arr = {1, 2, 3};
static_assert(arr[0] == 1);
```

---

## deque

```cpp
#include <deque>

deque<int> dq;
dq.push_back(1);                      // O(1) amortized
dq.push_front(0);                     // O(1) amortized
dq.pop_back();                        // O(1)
dq.pop_front();                       // O(1)
dq[0];  dq.at(0);                     // O(1) random access
dq.front();  dq.back();
dq.insert(dq.begin() + 2, 99);       // O(n)
dq.erase(dq.begin() + 2);            // O(n)
dq.size();  dq.empty();
```

---

## list / forward_list

```cpp
#include <list>
#include <forward_list>

// Doubly-linked list
list<int> lst = {3, 1, 4, 1, 5};
lst.push_back(9);                     // O(1)
lst.push_front(0);                    // O(1)
lst.pop_back();  lst.pop_front();     // O(1)
lst.front();  lst.back();             // O(1)
auto it = lst.begin();
advance(it, 2);
lst.insert(it, 99);                   // O(1) given iterator
lst.erase(it);                        // O(1) given iterator

// List-specific operations (in-place, no invalidation)
lst.sort();                           // O(n log n) merge sort
lst.reverse();                        // O(n)
lst.unique();                         // remove consecutive dups — O(n)
lst.remove(1);                        // remove all elements == 1 — O(n)
lst.remove_if([](int x) { return x > 3; });
lst.merge(other_sorted_list);         // O(n) merge two sorted lists
lst.splice(lst.end(), other);         // O(1) transfer entire other list

// Singly-linked list
forward_list<int> fl = {1, 2, 3};
fl.push_front(0);                     // O(1)
fl.pop_front();                       // O(1)
fl.front();                           // O(1)
fl.insert_after(fl.begin(), 99);      // O(1) given iterator
fl.erase_after(fl.begin());           // O(1)
fl.sort();  fl.reverse();  fl.unique();
```

---

## set / multiset

```cpp
#include <set>

set<int> s = {5, 3, 1, 4, 2};        // stored sorted: {1,2,3,4,5}

// Insert & erase
s.insert(6);                          // O(log n) — returns {iterator, bool}
auto [it, inserted] = s.insert(3);    // inserted = false (already exists)
s.emplace(7);                         // construct in-place
s.erase(3);                           // erase by value — O(log n)
s.erase(s.begin());                   // erase by iterator — O(1) amortized

// Lookup
s.find(3);                            // O(log n) — iterator or s.end()
s.count(3);                           // 0 or 1 for set
s.contains(3);                        // C++20 — true/false
s.lower_bound(3);                     // first >= 3
s.upper_bound(3);                     // first > 3
auto [lo, hi] = s.equal_range(3);     // [lower_bound, upper_bound)

// Iteration (always sorted)
for (int x : s) cout << x << " ";    // 1 2 3 4 5

// Size
s.size();  s.empty();  s.clear();

// Custom comparator
set<int, greater<int>> desc = {3, 1, 4};  // {4, 3, 1}

// Multiset — allows duplicates
multiset<int> ms = {1, 1, 2, 3, 3, 3};
ms.count(3);                          // 3
ms.erase(ms.find(3));                 // erase ONE occurrence of 3
ms.erase(3);                          // erase ALL occurrences of 3
```

---

## map / multimap

```cpp
#include <map>

map<string, int> m = {{"apple", 3}, {"banana", 1}, {"cherry", 5}};

// Insert & modify
m["date"] = 4;                        // insert or overwrite — O(log n)
m.insert({"elderberry", 2});          // insert only if key absent
m.insert_or_assign("apple", 10);     // insert or overwrite (C++17)
m.emplace("fig", 6);

auto [it, ok] = m.try_emplace("grape", 7);  // C++17 — no overwrite

// Access
m["apple"];                           // 3 — WARNING: inserts default if absent
m.at("apple");                        // 3 — throws if absent

// Lookup
m.find("apple");                      // iterator or m.end()
m.count("apple");                     // 0 or 1
m.contains("apple");                  // C++20
m.lower_bound("b");                   // first key >= "b"

// Erase
m.erase("apple");                     // by key — O(log n)
m.erase(m.begin());                   // by iterator

// Iteration (sorted by key)
for (auto& [key, val] : m)
    cout << key << ": " << val << "\n";

// Multimap — duplicate keys allowed
multimap<string, int> mm;
mm.insert({"a", 1});
mm.insert({"a", 2});                  // both stored
auto range = mm.equal_range("a");     // iterate all values for key "a"
for (auto it = range.first; it != range.second; ++it)
    cout << it->second << " ";
```

---

## unordered_set / unordered_map

```cpp
#include <unordered_set>
#include <unordered_map>

// Unordered set
unordered_set<int> us = {5, 3, 1, 4, 2};
us.insert(6);                         // O(1) average
us.erase(3);                          // O(1) average
us.find(4);                           // O(1) average
us.count(4);                          // 0 or 1
us.contains(4);                       // C++20
us.size();  us.empty();

// Bucket interface
us.bucket_count();                    // number of buckets
us.load_factor();                     // elements / buckets
us.max_load_factor(0.5);             // trigger rehash sooner
us.reserve(1000);                     // pre-allocate buckets

// Unordered map
unordered_map<string, int> um;
um["apple"] = 3;                      // O(1) average
um.insert({"banana", 1});
um.at("apple");                       // throws if absent
um.find("apple");                     // O(1) average
um.erase("apple");                    // O(1) average

for (auto& [k, v] : um)
    cout << k << ": " << v << "\n";

// Custom hash for pair / tuple
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<long long>()(((long long)p.first << 32) | p.second);
    }
};
unordered_set<pair<int,int>, PairHash> ps;
unordered_map<pair<int,int>, string, PairHash> pm;
```

---

## stack / queue / priority_queue

```cpp
#include <stack>
#include <queue>

// Stack — LIFO
stack<int> st;
st.push(1);  st.push(2);  st.push(3);
st.top();                             // 3
st.pop();                             // removes 3
st.size();  st.empty();

// Queue — FIFO
queue<int> q;
q.push(1);  q.push(2);  q.push(3);
q.front();                            // 1
q.back();                             // 3
q.pop();                              // removes 1

// Priority queue — max-heap by default
priority_queue<int> maxpq;
maxpq.push(3);  maxpq.push(1);  maxpq.push(4);
maxpq.top();                          // 4
maxpq.pop();                          // removes 4

// Min-heap
priority_queue<int, vector<int>, greater<int>> minpq;
minpq.push(3);  minpq.push(1);  minpq.push(4);
minpq.top();                          // 1

// Custom comparator
struct Task { int priority; string name; };
auto cmp = [](const Task& a, const Task& b) {
    return a.priority < b.priority;   // higher priority first
};
priority_queue<Task, vector<Task>, decltype(cmp)> pq(cmp);
pq.push({3, "low"});
pq.push({10, "high"});
pq.top().name;                        // "high"

// Build heap from vector
vector<int> v = {5, 3, 1, 4, 2};
priority_queue<int> pq2(v.begin(), v.end());  // O(n) heapify
```

---

## pair / tuple

```cpp
#include <utility>  // pair
#include <tuple>

// Pair
pair<int, string> p = {1, "one"};
p.first;  p.second;                   // access
auto p2 = make_pair(2, "two");
auto [num, name] = p;                 // structured binding (C++17)

// Pairs compare lexicographically (first, then second)
pair<int,int> a{1, 3}, b{1, 5};
a < b;                                // true (same first, 3 < 5)

// Tuple
auto t = make_tuple(1, "hello", 3.14);
get<0>(t);                            // 1
get<1>(t);                            // "hello"
tuple_size_v<decltype(t)>;            // 3

auto [x, y, z] = t;                   // structured binding (C++17)

// tie for comparison
int a1, a2;
string s;
tie(a1, s, ignore) = t;              // ignore discards third element

// Tuple as key (needs < operator — works with map, not unordered_map)
map<tuple<int,int,int>, string> lookup;
lookup[{1, 2, 3}] = "found";
```

---

## Iterators

### Iterator Categories

| Category | Operations | Containers |
|---|---|---|
| Input | `++`, `*`, `==`, `!=` | `istream_iterator` |
| Output | `++`, `*` (write) | `ostream_iterator`, `back_inserter` |
| Forward | Input + multi-pass | `forward_list`, `unordered_*` |
| Bidirectional | Forward + `--` | `list`, `set`, `map` |
| Random Access | Bidirectional + `[]`, `+`, `-`, `<` | `vector`, `deque`, `array`, `string` |
| Contiguous (C++20) | Random Access + adjacent in memory | `vector`, `array`, `string`, `span` |

### Iterator Utilities

```cpp
#include <iterator>

vector<int> v = {10, 20, 30, 40, 50};

// Advance and distance
auto it = v.begin();
advance(it, 3);                       // move forward 3 — O(1) for RA, O(n) for others
int d = distance(v.begin(), it);      // 3

// next / prev (non-mutating)
auto it2 = next(v.begin(), 2);        // iterator to index 2
auto it3 = prev(v.end());             // iterator to last element

// Inserter iterators
vector<int> dest;
copy(v.begin(), v.end(), back_inserter(dest));     // push_back each
list<int> lst;
copy(v.begin(), v.end(), front_inserter(lst));     // push_front each

// Reverse iterators
for (auto it = v.rbegin(); it != v.rend(); ++it)
    cout << *it << " ";                // 50 40 30 20 10
```

---

## Algorithms — Sorting & Ordering

| Algorithm | Description | Complexity | Stable | Header |
|---|---|:---:|:---:|---|
| `sort` | Introsort (quicksort + heapsort + insertion) | O(n log n) | No | `<algorithm>` |
| `stable_sort` | Merge sort | O(n log n) | Yes | `<algorithm>` |
| `partial_sort` | Sort first k elements | O(n log k) | No | `<algorithm>` |
| `nth_element` | Place nth element in sorted position | O(n) avg | No | `<algorithm>` |
| `is_sorted` | Check if sorted | O(n) | — | `<algorithm>` |
| `is_sorted_until` | First unsorted element | O(n) | — | `<algorithm>` |

```cpp
vector<int> v = {5, 3, 1, 4, 2};

// Full sort
sort(v.begin(), v.end());                          // ascending
sort(v.begin(), v.end(), greater<>());             // descending
sort(v.begin(), v.end(), [](int a, int b) {        // custom
    return abs(a) < abs(b);
});

// Stable sort — equal elements keep original order
stable_sort(v.begin(), v.end());

// Partial sort — only first k sorted
partial_sort(v.begin(), v.begin() + 3, v.end());  // smallest 3 in front, sorted

// nth_element — O(n) average, partition around nth
nth_element(v.begin(), v.begin() + 2, v.end());   // v[2] = median-ish
// v[0..1] <= v[2] <= v[3..n-1] but not sorted within halves

// Sort indices
vector<int> idx(v.size());
iota(idx.begin(), idx.end(), 0);
sort(idx.begin(), idx.end(), [&](int a, int b) {
    return v[a] < v[b];
});
```

---

## Algorithms — Binary Search (Sorted Data)

| Algorithm | Description | Returns | Complexity |
|---|---|---|:---:|
| `binary_search` | Check if value exists | `bool` | O(log n) |
| `lower_bound` | First element >= value | iterator | O(log n) |
| `upper_bound` | First element > value | iterator | O(log n) |
| `equal_range` | Range of elements == value | pair of iterators | O(log n) |

```cpp
vector<int> v = {1, 2, 3, 3, 3, 5, 7};  // must be sorted

bool found = binary_search(v.begin(), v.end(), 3);       // true

auto lo = lower_bound(v.begin(), v.end(), 3);             // → index 2
auto hi = upper_bound(v.begin(), v.end(), 3);             // → index 5
int count_of_3 = hi - lo;                                 // 3

auto [lb, ub] = equal_range(v.begin(), v.end(), 3);       // [2, 5)

// Insert position for maintaining sorted order
auto pos = lower_bound(v.begin(), v.end(), 4);
v.insert(pos, 4);                                         // insert 4 in sorted position

// Custom comparator
vector<pair<int,int>> vp = {{1,10}, {2,20}, {3,30}};
auto it = lower_bound(vp.begin(), vp.end(), 2,
    [](const pair<int,int>& p, int val) { return p.first < val; });
```

---

## Algorithms — Searching (Unsorted Data)

| Algorithm | Description | Returns | Complexity |
|---|---|---|:---:|
| `find` | Find first match by value | iterator | O(n) |
| `find_if` | Find first match by predicate | iterator | O(n) |
| `find_if_not` | Find first non-match | iterator | O(n) |
| `count` | Count matches by value | int | O(n) |
| `count_if` | Count matches by predicate | int | O(n) |
| `search` | Find subsequence | iterator | O(n×m) |
| `adjacent_find` | Find first equal adjacent pair | iterator | O(n) |

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

auto it = find(v.begin(), v.end(), 5);                    // → index 4
if (it != v.end()) cout << *it;

auto it2 = find_if(v.begin(), v.end(),
    [](int x) { return x > 7; });                         // → 9

int cnt = count(v.begin(), v.end(), 1);                   // 2
int even = count_if(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; });                    // 3

auto adj = adjacent_find(v.begin(), v.end());             // first a[i]==a[i+1]
```

---

## Algorithms — Predicates & Checks

| Algorithm | Description | Complexity |
|---|---|:---:|
| `all_of` | True if predicate holds for all | O(n) |
| `any_of` | True if predicate holds for any | O(n) |
| `none_of` | True if predicate holds for none | O(n) |
| `equal` | Two ranges element-wise equal | O(n) |
| `mismatch` | First differing pair | O(n) |
| `lexicographical_compare` | Dictionary-order comparison | O(n) |

```cpp
vector<int> v = {2, 4, 6, 8, 10};

bool allEven   = all_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });  // true
bool anyOdd    = any_of(v.begin(), v.end(), [](int x) { return x % 2 != 0; });  // false
bool noneNeg   = none_of(v.begin(), v.end(), [](int x) { return x < 0; });      // true

vector<int> a = {1, 2, 3}, b = {1, 2, 4};
bool eq = equal(a.begin(), a.end(), b.begin());                                  // false
auto [it1, it2] = mismatch(a.begin(), a.end(), b.begin());                       // *it1=3, *it2=4
```

---

## Algorithms — Transformations & Modifications

| Algorithm | Description | Complexity |
|---|---|:---:|
| `transform` | Apply function, store result | O(n) |
| `for_each` | Apply function (no output) | O(n) |
| `copy` / `copy_if` | Copy elements | O(n) |
| `move` | Move elements to output | O(n) |
| `fill` / `fill_n` | Assign value to range | O(n) |
| `generate` / `generate_n` | Assign from generator | O(n) |
| `replace` / `replace_if` | Replace matching values | O(n) |
| `reverse` | Reverse in-place | O(n) |
| `rotate` | Left-rotate around pivot | O(n) |
| `unique` | Remove consecutive duplicates | O(n) |
| `remove` / `remove_if` | Move unwanted to end | O(n) |
| `swap_ranges` | Swap two ranges | O(n) |

```cpp
vector<int> v = {1, 2, 3, 4, 5};
vector<int> out(v.size());

// Transform
transform(v.begin(), v.end(), out.begin(), [](int x) { return x * x; });
// out = {1, 4, 9, 16, 25}

// Two-range transform
vector<int> a = {1, 2, 3}, b = {4, 5, 6}, c(3);
transform(a.begin(), a.end(), b.begin(), c.begin(), plus<>());
// c = {5, 7, 9}

// for_each
for_each(v.begin(), v.end(), [](int& x) { x *= 2; });

// Copy with predicate
vector<int> evens;
copy_if(v.begin(), v.end(), back_inserter(evens),
    [](int x) { return x % 2 == 0; });

// Fill & generate
fill(v.begin(), v.end(), 0);
iota(v.begin(), v.end(), 1);                       // {1, 2, 3, ...}
generate(v.begin(), v.end(), [n=0]() mutable { return n++; });

// Replace
replace(v.begin(), v.end(), 3, 99);                // replace all 3s with 99
replace_if(v.begin(), v.end(),
    [](int x) { return x < 0; }, 0);               // replace negatives with 0

// Reverse & rotate
reverse(v.begin(), v.end());
rotate(v.begin(), v.begin() + 2, v.end());         // {3,4,5,1,2}

// Remove-erase idiom
v.erase(remove(v.begin(), v.end(), 3), v.end());   // erase all 3s
v.erase(remove_if(v.begin(), v.end(),
    [](int x) { return x < 0; }), v.end());

// C++20 simplified
erase(v, 3);                                        // erase all 3s
erase_if(v, [](int x) { return x < 0; });

// Unique (sort first to remove all duplicates, not just consecutive)
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
```

---

## Algorithms — Aggregation & Numeric

| Algorithm | Description | Header | Complexity |
|---|---|---|:---:|
| `accumulate` | Left fold with initial value | `<numeric>` | O(n) |
| `reduce` | Parallel-friendly accumulate (C++17) | `<numeric>` | O(n) |
| `inner_product` | Dot product of two ranges | `<numeric>` | O(n) |
| `partial_sum` | Running prefix sums | `<numeric>` | O(n) |
| `adjacent_difference` | Consecutive differences | `<numeric>` | O(n) |
| `inclusive_scan` | Prefix sums (C++17) | `<numeric>` | O(n) |
| `exclusive_scan` | Prefix sums excluding current (C++17) | `<numeric>` | O(n) |
| `transform_reduce` | Map + reduce in one pass (C++17) | `<numeric>` | O(n) |
| `iota` | Fill with incrementing sequence | `<numeric>` | O(n) |
| `min_element` / `max_element` | Find min/max | `<algorithm>` | O(n) |
| `minmax_element` | Find both min and max | `<algorithm>` | O(n) |

```cpp
#include <numeric>

vector<int> v = {1, 2, 3, 4, 5};

int sum = accumulate(v.begin(), v.end(), 0);                // 15
int product = accumulate(v.begin(), v.end(), 1, multiplies<>()); // 120
string joined = accumulate(next(v.begin()), v.end(),
    to_string(v[0]),
    [](string a, int b) { return a + "," + to_string(b); }); // "1,2,3,4,5"

// C++17 reduce (order-independent, parallelizable)
int sum2 = reduce(v.begin(), v.end(), 0);                   // 15

// Prefix sums
vector<int> prefix(v.size());
partial_sum(v.begin(), v.end(), prefix.begin());             // {1,3,6,10,15}
inclusive_scan(v.begin(), v.end(), prefix.begin());           // same as partial_sum
exclusive_scan(v.begin(), v.end(), prefix.begin(), 0);       // {0,1,3,6,10}

// Adjacent differences
vector<int> diff(v.size());
adjacent_difference(v.begin(), v.end(), diff.begin());       // {1,1,1,1,1}

// Inner product (dot product)
vector<int> a = {1, 2, 3}, b = {4, 5, 6};
int dot = inner_product(a.begin(), a.end(), b.begin(), 0);   // 32

// Transform + reduce in one pass (C++17)
int sumSquares = transform_reduce(v.begin(), v.end(), 0,
    plus<>(), [](int x) { return x * x; });                  // 55

// iota
iota(v.begin(), v.end(), 0);                                 // {0,1,2,3,4}

// Min, max
int mn = *min_element(v.begin(), v.end());
int mx = *max_element(v.begin(), v.end());
auto [minIt, maxIt] = minmax_element(v.begin(), v.end());
int clamped = clamp(15, 0, 10);                              // 10
```

---

## Algorithms — Permutations & Combinatorics

| Algorithm | Description | Complexity |
|---|---|:---:|
| `next_permutation` | Next lexicographic permutation | O(n) |
| `prev_permutation` | Previous lexicographic permutation | O(n) |
| `is_permutation` | Check if one range is permutation of another | O(n²) worst |

```cpp
vector<int> v = {1, 2, 3};

// Generate all permutations
sort(v.begin(), v.end());             // must start sorted for all perms
do {
    for (int x : v) cout << x << " ";
    cout << "\n";
} while (next_permutation(v.begin(), v.end()));
// 1 2 3 → 1 3 2 → 2 1 3 → 2 3 1 → 3 1 2 → 3 2 1

// Check permutation
vector<int> a = {1, 2, 3}, b = {3, 1, 2};
bool isPerm = is_permutation(a.begin(), a.end(), b.begin()); // true
```

---

## Algorithms — Set Operations (Sorted Ranges)

| Algorithm | Description | Complexity |
|---|---|:---:|
| `set_union` | Union of two sorted ranges | O(n + m) |
| `set_intersection` | Intersection | O(n + m) |
| `set_difference` | Elements in first but not second | O(n + m) |
| `set_symmetric_difference` | Elements in either but not both | O(n + m) |
| `includes` | Check if first contains all of second | O(n + m) |
| `merge` | Merge two sorted ranges | O(n + m) |
| `inplace_merge` | Merge two sorted halves in-place | O(n) with buffer |

```cpp
vector<int> a = {1, 2, 3, 4, 5};
vector<int> b = {3, 4, 5, 6, 7};
vector<int> result;

set_union(a.begin(), a.end(), b.begin(), b.end(),
    back_inserter(result));                                  // {1,2,3,4,5,6,7}

result.clear();
set_intersection(a.begin(), a.end(), b.begin(), b.end(),
    back_inserter(result));                                  // {3,4,5}

result.clear();
set_difference(a.begin(), a.end(), b.begin(), b.end(),
    back_inserter(result));                                  // {1,2}

bool containsAll = includes(a.begin(), a.end(),
    b.begin(), b.begin() + 2);                               // check a ⊇ {3,4}

// Merge two sorted ranges
vector<int> merged;
merge(a.begin(), a.end(), b.begin(), b.end(),
    back_inserter(merged));
```

---

## Algorithms — Heap Operations

| Algorithm | Description | Complexity | Header |
|---|---|:---:|---|
| `make_heap` | Build max-heap from range | O(n) | `<algorithm>` |
| `push_heap` | Insert element into heap | O(log n) | `<algorithm>` |
| `pop_heap` | Move max to end, re-heapify | O(log n) | `<algorithm>` |
| `sort_heap` | Sort a heap range | O(n log n) | `<algorithm>` |
| `is_heap` | Check if range is a heap | O(n) | `<algorithm>` |

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9};

make_heap(v.begin(), v.end());        // v is now a max-heap: [9,5,4,1,1,3]

v.push_back(7);
push_heap(v.begin(), v.end());        // re-heapify after adding element

pop_heap(v.begin(), v.end());         // move max to back
int maxVal = v.back();                // 9
v.pop_back();                         // remove it

sort_heap(v.begin(), v.end());        // sort the heap → ascending order
bool isH = is_heap(v.begin(), v.end());
```

---

## Algorithms — Partitioning

| Algorithm | Description | Complexity |
|---|---|:---:|
| `partition` | Reorder: true-elements before false-elements | O(n) |
| `stable_partition` | Same but preserves relative order | O(n) with buffer |
| `partition_point` | First element where predicate is false (sorted) | O(log n) |
| `is_partitioned` | Check if partitioned | O(n) |

```cpp
vector<int> v = {1, 4, 2, 5, 3, 6};

auto pivot = partition(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; });
// evens before pivot, odds after: [4, 6, 2 | 5, 3, 1] (order within halves unspecified)

// Stable — preserves original order within each group
stable_partition(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; });
// [4, 2, 6 | 1, 5, 3]
```

---

## Utility Classes

### std::optional (C++17)

```cpp
#include <optional>

optional<int> divide(int a, int b) {
    if (b == 0) return nullopt;
    return a / b;
}

auto result = divide(10, 3);
if (result)                           // or result.has_value()
    cout << *result;                  // or result.value()
int val = result.value_or(0);         // default if empty

// Chaining
optional<string> name = getName();
int len = name.transform([](auto& s) { return (int)s.size(); })
              .value_or(0);           // C++23
```

### std::variant (C++17)

```cpp
#include <variant>

variant<int, double, string> v = 42;
get<int>(v);                          // 42
v = 3.14;
get<double>(v);                       // 3.14

holds_alternative<int>(v);            // false (now holds double)

// Visitor
visit([](auto&& val) { cout << val; }, v);

// Index
v.index();                            // 1 (double is at index 1)
```

### std::any (C++17)

```cpp
#include <any>

any a = 42;
any_cast<int>(a);                     // 42
a = string("hello");
any_cast<string>(a);                  // "hello"
a.has_value();                        // true
a.type().name();                      // type info
a.reset();                            // clear
```

### std::string_view (C++17)

```cpp
#include <string_view>

void process(string_view sv) {
    sv.size();
    sv.substr(0, 5);                  // no allocation — returns another view
    sv.find("hello");
    sv.starts_with("he");             // C++20
    sv.remove_prefix(2);              // advance start
    sv.remove_suffix(3);              // shrink end
}
process("hello world");               // no string allocation
```

### std::span (C++20)

```cpp
#include <span>

void process(span<int> data) {
    for (int& x : data) x *= 2;
    data.size();
    data.front();  data.back();
    data.subspan(1, 3);               // sub-view
    data.first(2);                    // first 2 elements
    data.last(2);                     // last 2 elements
}

vector<int> v = {1, 2, 3, 4, 5};
process(v);                            // from vector
int arr[] = {1, 2, 3};
process(arr);                          // from C array
```

---

## Choosing the Right Container

| Need | Best Container | Why |
|---|---|---|
| Indexed access, append-heavy | `vector` | O(1) access, O(1)* append, cache-friendly |
| Frequent front & back insert | `deque` | O(1)* both ends |
| Frequent middle insert/remove | `list` | O(1) insert/erase given iterator |
| Sorted unique elements | `set` | O(log n) all ops, auto-sorted |
| Sorted key-value | `map` | O(log n) all ops, sorted by key |
| Fast lookup by key | `unordered_map` | O(1) average lookup |
| Fast membership check | `unordered_set` | O(1) average lookup |
| LIFO processing | `stack` | Clean interface, O(1) ops |
| FIFO processing | `queue` | Clean interface, O(1) ops |
| Top-k / priority | `priority_queue` | O(log n) insert, O(1) peek max |
| Fixed size, stack-allocated | `array` | Zero overhead, constexpr-friendly |
| Bit flags / compact booleans | `bitset` | Space-efficient, fast bitwise ops |

### When to Choose vector Over Everything Else

`vector` should be the default unless you have a specific reason not to use it:
- Cache-friendly (contiguous memory) outperforms `list`/`map` for small-to-medium data even when complexity is theoretically worse.
- Typically faster than `deque` for most use cases.
- Use `reserve()` to avoid reallocation overhead.
- Only switch to `list`/`set`/`map` when you genuinely need their guarantees (iterator stability, sorted order, O(log n) lookup).
