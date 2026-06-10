# Modern C++ Cheat Sheet (C++11 / 14 / 17 / 20)

Practical reference for modern C++ features with concise examples.

> **See also:** [STL Cheat Sheet](stl-cheatsheet.md) for containers, iterators, algorithms, and complexity tables.

---

## C++11

### Auto Type Deduction

```cpp
auto x = 42;                    // int
auto pi = 3.14;                 // double
auto name = string("hello");    // std::string
auto it = vec.begin();          // vector<int>::iterator

// with references and const
const auto& ref = vec[0];       // const int&
auto& mref = vec[0];            // int& (modifiable)
```

### Range-Based For Loop

```cpp
vector<int> nums = {1, 2, 3, 4, 5};

for (int n : nums)              // by value (copy)
    cout << n;

for (int& n : nums)             // by reference (modifiable)
    n *= 2;

for (const auto& n : nums)     // by const ref (read-only, no copy)
    cout << n;
```

### Lambda Expressions

```cpp
// Basic lambda
auto add = [](int a, int b) { return a + b; };
cout << add(3, 4);  // 7

// Capture by value
int x = 10;
auto fn = [x]() { return x * 2; };   // captures copy of x

// Capture by reference
auto fn2 = [&x]() { x += 5; };       // modifies original x

// Capture all by value / reference
auto fn3 = [=]() { return x; };      // all by value
auto fn4 = [&]() { x++; };           // all by reference

// Mutable lambda (modify captured-by-value)
auto counter = [count = 0]() mutable { return ++count; };
cout << counter();  // 1
cout << counter();  // 2

// Lambda with explicit return type
auto divide = [](double a, double b) -> double { return a / b; };

// Lambda in STL algorithms
vector<int> v = {3, 1, 4, 1, 5};
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });  // descending
```

### Move Semantics & Rvalue References

```cpp
// Move constructor and assignment
class Buffer {
    int* data;
    size_t size;
public:
    // Move constructor — steals resources
    Buffer(Buffer&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }

    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};

// std::move — cast to rvalue reference
vector<string> src = {"hello", "world"};
vector<string> dst = std::move(src);  // src is now in a valid but unspecified state

// Perfect forwarding
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));  // preserves value category
}
```

### Smart Pointers

```cpp
#include <memory>

// unique_ptr — exclusive ownership, zero overhead
auto p1 = make_unique<int>(42);       // C++14, use new in C++11
unique_ptr<int> p2(new int(42));      // C++11
cout << *p1;                           // 42
// auto p3 = p1;                       // ERROR: non-copyable
auto p3 = std::move(p1);              // OK: transfer ownership

// unique_ptr with arrays
auto arr = make_unique<int[]>(10);    // array of 10 ints
arr[0] = 42;

// unique_ptr with custom deleter
auto filePtr = unique_ptr<FILE, decltype(&fclose)>(fopen("f.txt", "r"), &fclose);

// shared_ptr — reference counted
auto sp1 = make_shared<int>(100);
auto sp2 = sp1;                        // ref count = 2
cout << sp1.use_count();               // 2

// weak_ptr — non-owning observer, breaks circular references
weak_ptr<int> wp = sp1;
if (auto locked = wp.lock()) {         // returns shared_ptr if alive
    cout << *locked;
}
```

### Initializer Lists & Uniform Initialization

```cpp
// Brace initialization
int x{42};
vector<int> v{1, 2, 3, 4};
map<string, int> m{{"a", 1}, {"b", 2}};
pair<int, string> p{1, "one"};

// std::initializer_list in custom classes
class Matrix {
public:
    Matrix(initializer_list<initializer_list<int>> data) {
        for (auto& row : data)
            for (int val : row)
                /* store val */;
    }
};
Matrix m = {{1, 2}, {3, 4}};

// Prevents narrowing conversions
// int x{3.14};  // ERROR: narrowing
```

### nullptr

```cpp
int* p = nullptr;            // replaces NULL and 0
if (p == nullptr) { /* ... */ }

void foo(int);
void foo(int*);
foo(nullptr);                // calls foo(int*), not foo(int)
```

### enum class (Scoped Enums)

```cpp
enum class Color { Red, Green, Blue };
enum class Size : uint8_t { Small = 1, Medium, Large };

Color c = Color::Red;
// int x = c;               // ERROR: no implicit conversion
int x = static_cast<int>(c); // OK: explicit
```

### constexpr

```cpp
constexpr int square(int n) { return n * n; }
constexpr int val = square(5);           // computed at compile time
static_assert(val == 25, "math broken");

constexpr int fib(int n) {
    return (n <= 1) ? n : fib(n - 1) + fib(n - 2);
}
```

### Variadic Templates

```cpp
// Base case
void print() {}

// Recursive variadic
template<typename T, typename... Args>
void print(T first, Args... rest) {
    cout << first << " ";
    print(rest...);
}
print(1, "hello", 3.14);  // 1 hello 3.14
```

### Type Aliases (using)

```cpp
using IntVec = vector<int>;
using StringMap = unordered_map<string, string>;
using Callback = function<void(int)>;

// Template alias
template<typename T>
using Vec2D = vector<vector<T>>;

Vec2D<int> grid(5, vector<int>(5, 0));
```

### static_assert

```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
static_assert(is_integral_v<int>, "need integral type");  // C++17
```

### Deleted and Defaulted Functions

```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    NonCopyable(NonCopyable&&) = default;
    NonCopyable& operator=(NonCopyable&&) = default;
};
```

### override and final

```cpp
class Base {
public:
    virtual void draw() const {}
    virtual void update() {}
};

class Derived : public Base {
public:
    void draw() const override {}   // compiler-checked override
    // void draw() override {}      // ERROR: signature mismatch (missing const)
};

class Final final : public Base {}; // cannot be inherited
```

### Concurrency Primitives

```cpp
#include <thread>
#include <mutex>
#include <future>
#include <atomic>

// Threads
thread t([]{ cout << "hello from thread\n"; });
t.join();

// Mutex
mutex mtx;
{
    lock_guard<mutex> lock(mtx);   // RAII lock, released at scope end
    // critical section
}

// Async + Future
auto fut = async(launch::async, [](int x) { return x * x; }, 5);
cout << fut.get();  // 25 (blocks until ready)

// Atomic
atomic<int> counter{0};
counter.fetch_add(1);              // thread-safe increment
```

---

## C++14

### Generic Lambdas

```cpp
auto add = [](auto a, auto b) { return a + b; };
cout << add(1, 2);        // 3 (int)
cout << add(1.5, 2.5);    // 4.0 (double)
cout << add(string("hi"), string("!")); // "hi!"
```

### Lambda Init Capture

```cpp
auto ptr = make_unique<int>(42);
auto fn = [p = std::move(ptr)]() { return *p; };  // move into lambda
cout << fn();  // 42

// Create new variable in capture
auto counter = [n = 0]() mutable { return ++n; };
```

### Return Type Deduction

```cpp
auto multiply(int a, int b) {
    return a * b;           // compiler deduces int
}

// Works with complex return types
auto makeVec() {
    return vector<int>{1, 2, 3};
}
```

### make_unique

```cpp
auto p = make_unique<int>(42);                  // single object
auto arr = make_unique<int[]>(100);             // array
auto widget = make_unique<Widget>(arg1, arg2);  // forwarded constructor args
```

### Binary Literals & Digit Separators

```cpp
int bin = 0b1010'1100;       // binary literal = 172
int big = 1'000'000;         // digit separator for readability
double pi = 3.141'592'653;
```

### deprecated Attribute

```cpp
[[deprecated("use newFunc() instead")]]
void oldFunc() { /* ... */ }
```

---

## C++17

### Structured Bindings

```cpp
// Pairs and tuples
auto [x, y] = make_pair(1, 2);
auto [a, b, c] = make_tuple(1, "hi", 3.14);

// Maps
unordered_map<string, int> m = {{"a", 1}, {"b", 2}};
for (auto& [key, val] : m)
    cout << key << ": " << val << "\n";

// Structs
struct Point { int x, y; };
auto [px, py] = Point{3, 4};

// Arrays
int arr[] = {1, 2, 3};
auto [a1, a2, a3] = arr;
```

### if / switch with Initializer

```cpp
// if with init
if (auto it = m.find("key"); it != m.end()) {
    cout << it->second;
}
// 'it' is scoped to the if-else block

// switch with init
switch (auto val = compute(); val) {
    case 0:  /* ... */ break;
    case 1:  /* ... */ break;
    default: break;
}
```

### std::optional

```cpp
#include <optional>

optional<int> findIndex(vector<int>& v, int target) {
    for (int i = 0; i < (int)v.size(); i++)
        if (v[i] == target) return i;
    return nullopt;
}

auto result = findIndex(v, 42);
if (result.has_value())           // or: if (result)
    cout << result.value();       // or: *result

int val = result.value_or(-1);    // default if empty
```

### std::variant

```cpp
#include <variant>

variant<int, double, string> v;
v = 42;
cout << get<int>(v);             // 42
v = "hello"s;
cout << get<string>(v);          // hello

// Safe access
if (holds_alternative<int>(v))
    cout << get<int>(v);

// Visitor pattern
auto visitor = [](auto&& arg) {
    using T = decay_t<decltype(arg)>;
    if constexpr (is_same_v<T, int>)
        cout << "int: " << arg;
    else if constexpr (is_same_v<T, string>)
        cout << "string: " << arg;
};
visit(visitor, v);
```

### std::any

```cpp
#include <any>

any a = 42;
cout << any_cast<int>(a);       // 42
a = string("hello");
cout << any_cast<string>(a);    // hello
cout << a.type().name();        // type info

// Throws bad_any_cast on type mismatch
try { any_cast<int>(a); }
catch (bad_any_cast&) { /* wrong type */ }
```

### std::string_view

```cpp
#include <string_view>

void process(string_view sv) {   // non-owning, no allocation
    cout << sv.substr(0, 5);
    cout << sv.size();
}

process("hello world");          // works with const char*
string s = "hello world";
process(s);                      // works with std::string
process(string_view(s).substr(6)); // "world" — no copy
```

### if constexpr (Compile-Time Branching)

```cpp
template<typename T>
auto process(T val) {
    if constexpr (is_integral_v<T>) {
        return val * 2;
    } else if constexpr (is_floating_point_v<T>) {
        return val + 0.5;
    } else {
        return val;
    }
}
// Dead branches are discarded — no compilation error for invalid code in them
```

### Fold Expressions

```cpp
// Sum all arguments
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);            // right fold
}
cout << sum(1, 2, 3, 4);  // 10

// Print all with separator
template<typename... Args>
void printAll(Args... args) {
    ((cout << args << " "), ...);   // comma fold
}
printAll(1, "hello", 3.14);  // 1 hello 3.14

// All true?
template<typename... Args>
bool allTrue(Args... args) {
    return (args && ...);
}
```

### Class Template Argument Deduction (CTAD)

```cpp
pair p{1, 2.5};                  // pair<int, double>
tuple t{1, "hello"s, 3.14};     // tuple<int, string, double>
vector v{1, 2, 3};              // vector<int>
optional o{42};                  // optional<int>

lock_guard lock(mtx);           // lock_guard<mutex>
```

### Nested Namespaces

```cpp
// Before C++17
namespace A { namespace B { namespace C { /* ... */ } } }

// C++17
namespace A::B::C {
    void foo() {}
}
```

### std::filesystem

```cpp
#include <filesystem>
namespace fs = std::filesystem;

fs::path p = "/home/user/file.txt";
cout << p.filename();               // "file.txt"
cout << p.extension();              // ".txt"
cout << p.parent_path();            // "/home/user"

if (fs::exists(p))
    cout << fs::file_size(p);

for (auto& entry : fs::directory_iterator("/home/user"))
    cout << entry.path() << "\n";

fs::create_directories("/tmp/a/b/c");
fs::copy("src.txt", "dst.txt");
fs::remove("old.txt");
```

### Parallel Algorithms

```cpp
#include <algorithm>
#include <execution>

vector<int> v(1'000'000);
sort(execution::par, v.begin(), v.end());              // parallel sort
for_each(execution::par_unseq, v.begin(), v.end(),     // parallel + vectorized
         [](int& x) { x *= 2; });
auto it = find(execution::par, v.begin(), v.end(), 42);
int total = reduce(execution::par, v.begin(), v.end(), 0); // parallel sum
```

### std::apply

```cpp
auto args = make_tuple(1, 2.0, "hello"s);
auto result = apply([](int a, double b, string c) {
    return to_string(a) + " " + to_string(b) + " " + c;
}, args);
```

### std::clamp

```cpp
int val = clamp(15, 0, 10);   // 10 (clamped to max)
int val2 = clamp(-5, 0, 10);  // 0  (clamped to min)
int val3 = clamp(5, 0, 10);   // 5  (within range)
```

---

## C++20

### Concepts

```cpp
#include <concepts>

// Using standard concepts
template<integral T>
T gcd(T a, T b) {
    return b == 0 ? a : gcd(b, a % b);
}

// Custom concept
template<typename T>
concept Printable = requires(T t) {
    { cout << t } -> same_as<ostream&>;
};

template<Printable T>
void print(T val) { cout << val << "\n"; }

// Abbreviated function template with concept
void process(integral auto x) { cout << x * 2; }

// Concept with multiple requirements
template<typename T>
concept Sortable = requires(T a, T b) {
    { a < b } -> convertible_to<bool>;
    { a == b } -> convertible_to<bool>;
    requires is_copy_constructible_v<T>;
};
```

### Ranges

```cpp
#include <ranges>
#include <algorithm>
namespace rv = std::ranges::views;

vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Pipe syntax — lazy, composable
auto result = v
    | rv::filter([](int n) { return n % 2 == 0; })  // {2,4,6,8,10}
    | rv::transform([](int n) { return n * n; })     // {4,16,36,64,100}
    | rv::take(3);                                    // {4,16,36}

for (int n : result) cout << n << " ";

// Range algorithms (no begin/end needed)
ranges::sort(v);
auto it = ranges::find(v, 42);
bool has5 = ranges::any_of(v, [](int n) { return n == 5; });
auto [mn, mx] = ranges::minmax(v);

// Views
auto evens = v | rv::filter([](int n) { return n % 2 == 0; });
auto first5 = v | rv::take(5);
auto reversed = v | rv::reverse;
auto enumerated = v | rv::enumerate;  // C++23 (pairs of index, value)

// iota — generate sequences
for (int i : rv::iota(0, 10))        // 0..9
    cout << i << " ";

// split and join
string csv = "a,b,c,d";
for (auto word : csv | rv::split(','))
    cout << string_view(word) << "\n";
```

### Coroutines (co_await, co_yield, co_return)

```cpp
#include <coroutine>

// Generator — lazily yields values
template<typename T>
struct Generator {
    struct promise_type {
        T current_value;
        auto get_return_object() { return Generator{Handle::from_promise(*this)}; }
        auto initial_suspend() { return suspend_always{}; }
        auto final_suspend() noexcept { return suspend_always{}; }
        auto yield_value(T value) {
            current_value = value;
            return suspend_always{};
        }
        void return_void() {}
        void unhandled_exception() { terminate(); }
    };
    using Handle = coroutine_handle<promise_type>;
    Handle handle;

    explicit Generator(Handle h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    bool next() { handle.resume(); return !handle.done(); }
    T value() { return handle.promise().current_value; }
};

Generator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto next = a + b;
        a = b;
        b = next;
    }
}

auto gen = fibonacci();
for (int i = 0; i < 10 && gen.next(); i++)
    cout << gen.value() << " ";  // 0 1 1 2 3 5 8 13 21 34
```

### Modules (replacing headers)

```cpp
// math.cppm — module interface
export module math;

export int add(int a, int b) { return a + b; }
export int multiply(int a, int b) { return a * b; }

// main.cpp — import the module
import math;
int main() {
    cout << add(3, 4);        // 7
    cout << multiply(3, 4);   // 12
}
```

### Three-Way Comparison (Spaceship Operator)

```cpp
#include <compare>

struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;  // all 6 operators generated
};

Point a{1, 2}, b{3, 4};
if (a < b) cout << "a < b";
if (a == b) cout << "equal";

// Custom spaceship
struct Version {
    int major, minor, patch;
    auto operator<=>(const Version& o) const {
        if (auto cmp = major <=> o.major; cmp != 0) return cmp;
        if (auto cmp = minor <=> o.minor; cmp != 0) return cmp;
        return patch <=> o.patch;
    }
    bool operator==(const Version&) const = default;
};
```

### std::span

```cpp
#include <span>

void process(span<int> data) {     // non-owning view of contiguous data
    for (int& x : data) x *= 2;
    auto first3 = data.subspan(0, 3);
    cout << data.size();
}

vector<int> v = {1, 2, 3, 4, 5};
process(v);                         // works with vector
int arr[] = {1, 2, 3};
process(arr);                       // works with C array
```

### std::format

```cpp
#include <format>

string s = format("Hello, {}!", "world");              // "Hello, world!"
string s2 = format("{:>10}", 42);                       // "        42"
string s3 = format("{:.2f}", 3.14159);                  // "3.14"
string s4 = format("{0} + {0} = {1}", 3, 6);           // "3 + 3 = 6"
string s5 = format("{:08b}", 42);                       // "00101010"

// Width, alignment, fill
string s6 = format("{:<10}", "left");                   // "left      "
string s7 = format("{:>10}", "right");                  // "     right"
string s8 = format("{:^10}", "center");                 // "  center  "
string s9 = format("{:*^10}", "hi");                    // "****hi****"

// Use with print (C++23)
// print("x = {}, y = {}\n", 1, 2);
```

### consteval and constinit

```cpp
// consteval — must be evaluated at compile time
consteval int compiletimeSquare(int n) { return n * n; }
constexpr int a = compiletimeSquare(5);  // OK: compile time
// int b = compiletimeSquare(runtime_var); // ERROR: not compile-time

// constinit — compile-time initialization, but mutable at runtime
constinit int globalVal = 42;   // initialized at compile time
// Prevents static initialization order fiasco
```

### Designated Initializers

```cpp
struct Config {
    int width = 800;
    int height = 600;
    bool fullscreen = false;
    string title = "App";
};

Config cfg{
    .width = 1920,
    .height = 1080,
    .fullscreen = true,
    // .title uses default "App"
};
```

### std::jthread (Joinable Thread)

```cpp
#include <thread>
#include <stop_token>

// Auto-joins on destruction, supports cooperative cancellation
jthread worker([](stop_token st) {
    while (!st.stop_requested()) {
        // do work
    }
});
// worker.request_stop();  // signal cancellation
// destructor auto-joins
```

### Templated Lambdas

```cpp
auto lambda = []<typename T>(vector<T>& v) {
    for (auto& elem : v) cout << elem << " ";
};

vector<int> vi = {1, 2, 3};
vector<string> vs = {"a", "b"};
lambda(vi);  // 1 2 3
lambda(vs);  // a b
```

### contains() for Associative Containers

```cpp
map<string, int> m = {{"a", 1}, {"b", 2}};
if (m.contains("a"))           // cleaner than m.find("a") != m.end()
    cout << "found";

set<int> s = {1, 2, 3};
if (s.contains(2))
    cout << "has 2";
```

### starts_with / ends_with for Strings

```cpp
string filename = "report.pdf";
if (filename.starts_with("report"))  cout << "is report";
if (filename.ends_with(".pdf"))      cout << "is PDF";

string_view sv = "hello world";
sv.starts_with("hello");  // true
```

### Bit Manipulation Utilities

```cpp
#include <bit>

int x = 0b1010;
cout << popcount((unsigned)x);     // 2 (number of set bits)
cout << has_single_bit(8u);        // true (power of 2)
cout << bit_ceil(5u);              // 8 (next power of 2)
cout << bit_floor(5u);             // 4 (prev power of 2)
cout << countl_zero(8u);           // count leading zeros
cout << countr_zero(8u);           // count trailing zeros = 3
cout << rotr(0b1100u, 2);          // rotate right
```

---

## Useful Patterns

### RAII (Resource Acquisition Is Initialization)

```cpp
class FileHandle {
    FILE* fp;
public:
    FileHandle(const char* name) : fp(fopen(name, "r")) {
        if (!fp) throw runtime_error("open failed");
    }
    ~FileHandle() { if (fp) fclose(fp); }

    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
};
// File is closed automatically when object goes out of scope
```

### Type Traits & SFINAE

```cpp
#include <type_traits>

// Type checks
static_assert(is_integral_v<int>);
static_assert(is_floating_point_v<double>);
static_assert(is_same_v<int, int>);
static_assert(is_base_of_v<Base, Derived>);

// Conditional type
using Type = conditional_t<sizeof(int) == 4, int, long>;

// enable_if (pre-concepts SFINAE)
template<typename T>
enable_if_t<is_integral_v<T>, T> doubleIt(T val) {
    return val * 2;
}

// C++20: prefer concepts over SFINAE
template<integral T>
T doubleIt(T val) { return val * 2; }
```

### Compile-Time Computation

```cpp
// constexpr array generation
constexpr auto makeTable() {
    array<int, 256> table{};
    for (int i = 0; i < 256; i++)
        table[i] = i * i;
    return table;
}
constexpr auto squareTable = makeTable();  // computed at compile time
```

### Scoped Timer

```cpp
#include <chrono>

struct Timer {
    chrono::high_resolution_clock::time_point start;
    Timer() : start(chrono::high_resolution_clock::now()) {}
    ~Timer() {
        auto end = chrono::high_resolution_clock::now();
        auto ms = chrono::duration_cast<chrono::milliseconds>(end - start).count();
        cout << "Elapsed: " << ms << " ms\n";
    }
};

{
    Timer t;
    // code to benchmark
}  // prints elapsed time
```

---

## Version Feature Map

| Feature | C++11 | C++14 | C++17 | C++20 |
|---|:---:|:---:|:---:|:---:|
| `auto` | x | | | |
| Range-based for | x | | | |
| Lambdas | x | generic | | templated |
| Move semantics | x | | | |
| Smart pointers | x | `make_unique` | | |
| `constexpr` | basic | relaxed | if constexpr | consteval |
| Variadic templates | x | | fold expr | |
| Structured bindings | | | x | |
| `optional/variant/any` | | | x | |
| `string_view` | | | x | |
| Filesystem | | | x | |
| Parallel algorithms | | | x | |
| Concepts | | | | x |
| Ranges | | | | x |
| Coroutines | | | | x |
| Modules | | | | x |
| `<=>` spaceship | | | | x |
| `span` | | | | x |
| `format` | | | | x |
| `jthread` | | | | x |
| `contains()` | | | | x |
| `starts_with/ends_with` | | | | x |
| `<bit>` utilities | | | | x |
