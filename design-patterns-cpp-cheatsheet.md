# Design Patterns — Modern C++ Cheat Sheet

Modern C++ (C++17/20) equivalents of the 21 GoF patterns from the
[Geekific design-patterns playlist](https://youtube.com/playlist?list=PLlsmxlJgn1HJpa28yHzkBmUY-Ty71ZUGc)
and its [GitHub repo](https://github.com/geekific-official/geekific-youtube/tree/main/design-patterns)
(originally Java). Each entry gives the intent, a compact idiomatic C++ snippet, and the modern-C++ angle.

Conventions used throughout:
- `std::unique_ptr` for ownership, `std::shared_ptr` only when sharing is required.
- `= default` / `override` / `[[nodiscard]]` where it helps.
- Prefer `std::function`, `std::variant`, lambdas, and value semantics over hand-rolled hierarchies when they read better.

---

## Index

**Creational:** [Factory](#1-factory-method) · [Abstract Factory](#2-abstract-factory) · [Builder](#3-builder) · [Prototype](#4-prototype) · [Singleton](#5-singleton)
**Structural:** [Adapter](#6-adapter) · [Bridge](#7-bridge) · [Composite](#8-composite) · [Decorator](#9-decorator) · [Facade](#10-facade) · [Flyweight](#11-flyweight) · [Proxy](#12-proxy)
**Behavioral:** [Chain of Responsibility](#13-chain-of-responsibility) · [Command](#14-command) · [Iterator](#15-iterator) · [Mediator](#16-mediator) · [Memento](#17-memento) · [Observer](#18-observer) · [State](#19-state) · [Strategy](#20-strategy) · [Template Method](#21-template-method) · [Visitor](#22-visitor)

---

# Creational

## 1. Factory Method
**Intent:** Defer instantiation to subclasses / a factory function; callers depend on the interface, not the concrete type.

```cpp
#include <memory>
#include <string>

struct Notification {
    virtual ~Notification() = default;
    virtual std::string notifyUser() const = 0;
};

struct SMSNotification : Notification {
    std::string notifyUser() const override { return "Sending an SMS"; }
};
struct EmailNotification : Notification {
    std::string notifyUser() const override { return "Sending an e-mail"; }
};

enum class Channel { SMS, Email };

// Factory method
std::unique_ptr<Notification> createNotification(Channel c) {
    switch (c) {
        case Channel::SMS:   return std::make_unique<SMSNotification>();
        case Channel::Email: return std::make_unique<EmailNotification>();
    }
    return nullptr;
}
```
**Modern angle:** A free function returning `unique_ptr` replaces a `NotificationFactory` class. Consider registering creators in a `std::unordered_map<Key, std::function<std::unique_ptr<Notification>()>>` for open/closed extensibility.

---

## 2. Abstract Factory
**Intent:** Create *families* of related objects without binding to concrete classes.

```cpp
#include <memory>
#include <string>

struct Button   { virtual ~Button() = default;   virtual std::string paint() const = 0; };
struct Checkbox { virtual ~Checkbox() = default; virtual std::string paint() const = 0; };

struct WinButton   : Button   { std::string paint() const override { return "Windows button"; } };
struct WinCheckbox : Checkbox { std::string paint() const override { return "Windows checkbox"; } };
struct MacButton   : Button   { std::string paint() const override { return "macOS button"; } };
struct MacCheckbox : Checkbox { std::string paint() const override { return "macOS checkbox"; } };

struct GUIFactory {
    virtual ~GUIFactory() = default;
    virtual std::unique_ptr<Button>   createButton()   const = 0;
    virtual std::unique_ptr<Checkbox> createCheckbox() const = 0;
};

struct WinFactory : GUIFactory {
    std::unique_ptr<Button>   createButton()   const override { return std::make_unique<WinButton>(); }
    std::unique_ptr<Checkbox> createCheckbox() const override { return std::make_unique<WinCheckbox>(); }
};
struct MacFactory : GUIFactory {
    std::unique_ptr<Button>   createButton()   const override { return std::make_unique<MacButton>(); }
    std::unique_ptr<Checkbox> createCheckbox() const override { return std::make_unique<MacCheckbox>(); }
};
```
**Modern angle:** One concrete factory per platform guarantees the produced widgets are consistent. Client code holds a `GUIFactory&` and never sees `Win*`/`Mac*`.

---

## 3. Builder
**Intent:** Construct a complex object step by step; separate construction from representation.

```cpp
#include <string>
#include <utility>

class BankAccount {
public:
    class Builder; // fluent builder

    std::string toString() const {
        return name_ + " / " + std::to_string(accountNumber_) +
               " bal=" + std::to_string(balance_);
    }
private:
    friend class Builder;
    BankAccount() = default;
    std::string name_;
    long accountNumber_ = 0;
    double balance_ = 0.0;
    std::string email_;
};

class BankAccount::Builder {
public:
    explicit Builder(std::string name, long acct) {
        acc_.name_ = std::move(name);
        acc_.accountNumber_ = acct;
    }
    Builder& withBalance(double b) & { acc_.balance_ = b; return *this; }
    Builder& withEmail(std::string e) & { acc_.email_ = std::move(e); return *this; }
    BankAccount build() { return std::move(acc_); }
private:
    BankAccount acc_;
};

// auto acc = BankAccount::Builder("Ada", 1001).withBalance(500).withEmail("a@x.io").build();
```
**Modern angle:** For simple cases, designated initializers (C++20) on an aggregate config struct often replace a builder entirely:
```cpp
struct AccountConfig { std::string name; long number{}; double balance{}; };
AccountConfig cfg{ .name = "Ada", .number = 1001, .balance = 500 };
```

---

## 4. Prototype
**Intent:** Create new objects by cloning an existing instance instead of constructing from scratch.

```cpp
#include <memory>
#include <string>

struct Shape {
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0; // polymorphic copy
    virtual std::string draw() const = 0;
};

struct Circle : Shape {
    int radius;
    explicit Circle(int r) : radius(r) {}
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this); // deep copy via copy-ctor
    }
    std::string draw() const override { return "Circle r=" + std::to_string(radius); }
};
```
**Modern angle:** The `clone()` idiom (a.k.a. *virtual constructor*) is the canonical way to copy through a base pointer. CRTP can remove the boilerplate:
```cpp
template <class Derived, class Base>
struct Cloneable : Base {
    std::unique_ptr<Base> clone() const override {
        return std::make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};
```

---

## 5. Singleton
**Intent:** Exactly one instance, with global access.

```cpp
class Config {
public:
    static Config& instance() {     // Meyers' singleton: thread-safe since C++11
        static Config inst;
        return inst;
    }
    Config(const Config&) = delete;
    Config& operator=(const Config&) = delete;

    int port() const { return port_; }
    void setPort(int p) { port_ = p; }
private:
    Config() = default;
    int port_ = 8080;
};

// Config::instance().setPort(9090);
```
**Modern angle:** The function-local `static` is the correct, lazy, thread-safe singleton — no locks or double-checked locking needed. Use sparingly; prefer dependency injection where testability matters.

---

# Structural

## 6. Adapter
**Intent:** Convert one interface into another the client expects.

```cpp
#include <memory>
#include <string>

// Target the client wants
struct JsonPrinter { virtual ~JsonPrinter() = default; virtual std::string toJson() const = 0; };

// Adaptee with an incompatible interface
struct LegacyXml { std::string toXml() const { return "<data/>"; } };

class XmlToJsonAdapter : public JsonPrinter {
public:
    explicit XmlToJsonAdapter(LegacyXml xml) : xml_(std::move(xml)) {}
    std::string toJson() const override {
        return R"({"converted_from":")" + xml_.toXml() + R"("})";
    }
private:
    LegacyXml xml_;
};
```
**Modern angle:** Object adapter via composition (shown) is preferred over class adapter via multiple inheritance. Wrap by value or `unique_ptr` depending on ownership.

---

## 7. Bridge
**Intent:** Decouple an abstraction from its implementation so both vary independently.

```cpp
#include <memory>
#include <string>

// Implementor
struct Device {
    virtual ~Device() = default;
    virtual void setVolume(int) = 0;
    virtual std::string name() const = 0;
};
struct TV    : Device { void setVolume(int v) override {} std::string name() const override { return "TV"; } };
struct Radio : Device { void setVolume(int v) override {} std::string name() const override { return "Radio"; } };

// Abstraction holds a pointer to the implementor
class RemoteControl {
public:
    explicit RemoteControl(std::unique_ptr<Device> d) : device_(std::move(d)) {}
    virtual ~RemoteControl() = default;
    virtual void volumeUp() { device_->setVolume(+1); }
protected:
    std::unique_ptr<Device> device_;
};

class AdvancedRemote : public RemoteControl {
public:
    using RemoteControl::RemoteControl;
    void mute() { device_->setVolume(0); }
};
```
**Modern angle:** Bridge = "prefer composition over inheritance" made explicit. The abstraction hierarchy (`RemoteControl`) and implementation hierarchy (`Device`) evolve separately.

---

## 8. Composite
**Intent:** Treat individual objects and compositions uniformly via a common interface (trees / part-whole).

```cpp
#include <memory>
#include <string>
#include <vector>

struct FileSystemNode {
    virtual ~FileSystemNode() = default;
    virtual long size() const = 0;
    virtual std::string name() const = 0;
};

class File : public FileSystemNode {
public:
    File(std::string n, long s) : name_(std::move(n)), size_(s) {}
    long size() const override { return size_; }
    std::string name() const override { return name_; }
private:
    std::string name_; long size_;
};

class Directory : public FileSystemNode {
public:
    explicit Directory(std::string n) : name_(std::move(n)) {}
    void add(std::unique_ptr<FileSystemNode> child) { children_.push_back(std::move(child)); }
    long size() const override {
        long total = 0;
        for (const auto& c : children_) total += c->size();
        return total;
    }
    std::string name() const override { return name_; }
private:
    std::string name_;
    std::vector<std::unique_ptr<FileSystemNode>> children_;
};
```
**Modern angle:** `vector<unique_ptr<...>>` owns the subtree; recursion over `size()` works identically for leaves and composites.

---

## 9. Decorator
**Intent:** Attach responsibilities to an object dynamically by wrapping it.

```cpp
#include <memory>
#include <string>

struct Coffee {
    virtual ~Coffee() = default;
    virtual double cost() const = 0;
    virtual std::string desc() const = 0;
};

struct SimpleCoffee : Coffee {
    double cost() const override { return 2.0; }
    std::string desc() const override { return "Coffee"; }
};

// Base decorator wraps another Coffee
class CoffeeDecorator : public Coffee {
public:
    explicit CoffeeDecorator(std::unique_ptr<Coffee> inner) : inner_(std::move(inner)) {}
protected:
    std::unique_ptr<Coffee> inner_;
};

struct Milk : CoffeeDecorator {
    using CoffeeDecorator::CoffeeDecorator;
    double cost() const override { return inner_->cost() + 0.5; }
    std::string desc() const override { return inner_->desc() + " + Milk"; }
};

// auto c = std::make_unique<Milk>(std::make_unique<SimpleCoffee>());
```
**Modern angle:** Each decorator owns its wrapped object via `unique_ptr`, letting you stack `Milk(Sugar(SimpleCoffee))` at runtime without subclass explosion.

---

## 10. Facade
**Intent:** Provide a single simplified entry point to a complex subsystem.

```cpp
#include <string>

// Complex subsystem
struct CPU    { void freeze() {} void jump(long) {} void execute() {} };
struct Memory { void load(long, const std::string&) {} };
struct Drive  { std::string read(long, int) { return "boot-sector"; } };

class ComputerFacade {
public:
    void start() {
        cpu_.freeze();
        memory_.load(0, drive_.read(0, 512));
        cpu_.jump(0);
        cpu_.execute();
    }
private:
    CPU cpu_; Memory memory_; Drive drive_;
};

// ComputerFacade{}.start();
```
**Modern angle:** The facade owns the subsystem by value and exposes one verb (`start()`). Clients are shielded from ordering and wiring details.

---

## 11. Flyweight
**Intent:** Share immutable intrinsic state across many objects to save memory; pass extrinsic state in.

```cpp
#include <memory>
#include <string>
#include <unordered_map>

// Intrinsic, shared state
struct TreeType {
    std::string name, color, texture;
    void draw(int x, int y) const { /* render at (x,y) using shared data */ }
};

class TreeFactory {
public:
    std::shared_ptr<const TreeType> get(const std::string& name,
                                        const std::string& color,
                                        const std::string& texture) {
        std::string key = name + color + texture;
        auto it = pool_.find(key);
        if (it != pool_.end()) return it->second;
        auto t = std::make_shared<const TreeType>(TreeType{name, color, texture});
        pool_.emplace(key, t);
        return t;
    }
private:
    std::unordered_map<std::string, std::shared_ptr<const TreeType>> pool_;
};

// Extrinsic state lives in the lightweight object
struct Tree { int x, y; std::shared_ptr<const TreeType> type; };
```
**Modern angle:** `shared_ptr<const T>` in an `unordered_map` pool gives natural deduplication; `const` keeps the shared intrinsic state immutable.

---

## 12. Proxy
**Intent:** Stand in for another object to control access (lazy init, caching, access control, logging).

```cpp
#include <iostream>
#include <memory>
#include <string>

struct Image {
    virtual ~Image() = default;
    virtual void display() = 0;
};

class RealImage : public Image {
public:
    explicit RealImage(std::string file) : file_(std::move(file)) {
        std::cout << "Loading " << file_ << "\n"; // expensive
    }
    void display() override { std::cout << "Displaying " << file_ << "\n"; }
private:
    std::string file_;
};

class ProxyImage : public Image {       // virtual (lazy-loading) proxy
public:
    explicit ProxyImage(std::string file) : file_(std::move(file)) {}
    void display() override {
        if (!real_) real_ = std::make_unique<RealImage>(file_);
        real_->display();
    }
private:
    std::string file_;
    std::unique_ptr<RealImage> real_;
};
```
**Modern angle:** Same interface as the real subject; the proxy defers the heavy `RealImage` construction until first `display()`.

---

# Behavioral

## 13. Chain of Responsibility
**Intent:** Pass a request along a chain of handlers until one handles it.

```cpp
#include <memory>
#include <string>

enum class Level { Info, Debug, Error };

class Logger {
public:
    explicit Logger(Level lvl) : level_(lvl) {}
    virtual ~Logger() = default;
    Logger* setNext(std::unique_ptr<Logger> next) {
        next_ = std::move(next);
        return next_.get();
    }
    void log(Level lvl, const std::string& msg) {
        if (lvl == level_) write(msg);
        if (next_) next_->log(lvl, msg);
    }
protected:
    virtual void write(const std::string&) = 0;
private:
    Level level_;
    std::unique_ptr<Logger> next_;
};
```
**Modern angle:** Each link owns the next via `unique_ptr`. For simple cases, a `std::vector<std::function<bool(Request&)>>` iterated until one returns `true` is even leaner.

---

## 14. Command
**Intent:** Encapsulate a request as an object (parameterize, queue, undo).

```cpp
#include <functional>
#include <memory>
#include <vector>

struct Command {
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

struct Light { bool on = false; };

class LightOnCommand : public Command {
public:
    explicit LightOnCommand(Light& l) : light_(l) {}
    void execute() override { light_.on = true; }
    void undo() override    { light_.on = false; }
private:
    Light& light_;
};

class Remote {
public:
    void submit(std::unique_ptr<Command> c) { c->execute(); history_.push_back(std::move(c)); }
    void undoLast() { if (!history_.empty()) { history_.back()->undo(); history_.pop_back(); } }
private:
    std::vector<std::unique_ptr<Command>> history_;
};
```
**Modern angle:** When you don't need `undo`, a command *is* just `std::function<void()>` — store `std::vector<std::function<void()>>` and skip the class hierarchy.

---

## 15. Iterator
**Intent:** Traverse a collection without exposing its internals.

```cpp
#include <cstddef>
#include <vector>

template <class T>
class RingBuffer {
public:
    void push(T v) { data_.push_back(std::move(v)); }

    // Modern C++: model the standard iterator interface so range-for works
    using iterator = typename std::vector<T>::iterator;
    using const_iterator = typename std::vector<T>::const_iterator;
    iterator begin() { return data_.begin(); }
    iterator end()   { return data_.end(); }
    const_iterator begin() const { return data_.begin(); }
    const_iterator end()   const { return data_.end(); }
private:
    std::vector<T> data_;
};

// RingBuffer<int> rb; rb.push(1); for (int x : rb) { ... }
```
**Modern angle:** Don't hand-roll a GoF `Iterator` interface — expose `begin()/end()` so the type plugs into range-`for`, `<algorithm>`, and C++20 ranges. For lazy sequences, write a `std::input_iterator` or a coroutine `generator` (C++23 `std::generator`).

---

## 16. Mediator
**Intent:** Centralize complex communication between objects so they don't refer to each other directly.

```cpp
#include <string>
#include <vector>

class User;

struct ChatRoom {            // Mediator interface
    virtual ~ChatRoom() = default;
    virtual void broadcast(const std::string& from, const std::string& msg) = 0;
    virtual void join(User* u) = 0;
};

class User {
public:
    User(std::string name, ChatRoom& room) : name_(std::move(name)), room_(room) {}
    const std::string& name() const { return name_; }
    void send(const std::string& msg) { room_.broadcast(name_, msg); }
    void receive(const std::string& from, const std::string& msg) { /* show */ }
private:
    std::string name_;
    ChatRoom& room_;
};

class ConcreteChatRoom : public ChatRoom {
public:
    void join(User* u) override { users_.push_back(u); }
    void broadcast(const std::string& from, const std::string& msg) override {
        for (auto* u : users_) if (u->name() != from) u->receive(from, msg);
    }
private:
    std::vector<User*> users_; // non-owning
};
```
**Modern angle:** Colleagues hold a reference to the mediator, not to each other. Use raw/observer pointers for the non-owning back-references.

---

## 17. Memento
**Intent:** Capture and restore an object's internal state without violating encapsulation.

```cpp
#include <string>
#include <vector>

class Editor {
public:
    class Memento {                      // opaque snapshot
        friend class Editor;
        explicit Memento(std::string s) : state_(std::move(s)) {}
        std::string state_;
    };

    void type(const std::string& words) { content_ += words; }
    const std::string& content() const { return content_; }

    Memento save() const { return Memento{content_}; }
    void restore(const Memento& m) { content_ = m.state_; }
private:
    std::string content_;
};

// Caretaker keeps history
class History {
public:
    void push(Editor::Memento m) { stack_.push_back(std::move(m)); }
    Editor::Memento pop() { auto m = stack_.back(); stack_.pop_back(); return m; }
private:
    std::vector<Editor::Memento> stack_;
};
```
**Modern angle:** A nested `Memento` with `private` members + `friend` originator keeps state hidden from the caretaker, which only shuttles snapshots.

---

## 18. Observer
**Intent:** Notify dependents automatically when a subject changes (publish/subscribe).

```cpp
#include <functional>
#include <string>
#include <vector>

class NewsAgency {
public:
    using Observer = std::function<void(const std::string&)>;

    int subscribe(Observer obs) {
        observers_.push_back(std::move(obs));
        return static_cast<int>(observers_.size()) - 1; // token to unsubscribe
    }
    void publish(const std::string& news) {
        for (auto& o : observers_) if (o) o(news);
    }
private:
    std::vector<Observer> observers_;
};

// NewsAgency a;
// a.subscribe([](const std::string& n){ /* channel 1 */ });
// a.publish("Breaking!");
```
**Modern angle:** `std::vector<std::function<...>>` removes the need for an `Observer` interface and `update()` override. If observers have shorter lifetimes than the subject, store `std::weak_ptr` and prune expired ones on publish.

---

## 19. State
**Intent:** Let an object alter its behavior when its internal state changes — it appears to change class.

```cpp
#include <iostream>
#include <memory>

class Document;

struct State {
    virtual ~State() = default;
    virtual void publish(Document& doc) = 0;
};

class Document {
public:
    explicit Document(std::unique_ptr<State> s) : state_(std::move(s)) {}
    void setState(std::unique_ptr<State> s) { state_ = std::move(s); }
    void publish() { state_->publish(*this); }
private:
    std::unique_ptr<State> state_;
};

struct Published : State { void publish(Document&) override { std::cout << "Already live\n"; } };
struct Draft : State {
    void publish(Document& doc) override {
        std::cout << "Moderation...\n";
        doc.setState(std::make_unique<Published>());
    }
};
```
**Modern angle:** Each state transition swaps the `unique_ptr<State>`. For small, closed state sets, `std::variant<Draft, Moderation, Published>` + `std::visit` is a zero-allocation alternative.

---

## 20. Strategy
**Intent:** Define a family of interchangeable algorithms and select one at runtime.

```cpp
#include <functional>
#include <vector>

class ShoppingCart {
public:
    using PaymentStrategy = std::function<void(double)>;

    void setPayment(PaymentStrategy s) { pay_ = std::move(s); }
    void checkout(double amount) { if (pay_) pay_(amount); }
private:
    PaymentStrategy pay_;
};

// ShoppingCart cart;
// cart.setPayment([](double amt){ /* pay by card */ });
// cart.checkout(42.0);
```
**Modern angle:** `std::function` (or a template policy parameter for zero overhead) replaces a `Strategy` interface hierarchy. Strategy and Command/Observer all collapse to "store a callable" in modern C++.

---

## 21. Template Method
**Intent:** Define the skeleton of an algorithm in a base class; let subclasses override specific steps.

```cpp
#include <iostream>

class DataProcessor {
public:
    void run() {                 // the template method — fixed skeleton, non-virtual
        readData();
        process();               // customizable step
        writeData();
    }
    virtual ~DataProcessor() = default;
protected:
    void readData()  { std::cout << "read\n"; }
    void writeData() { std::cout << "write\n"; }
    virtual void process() = 0;  // the hook
};

struct CsvProcessor : DataProcessor {
    void process() override { std::cout << "process CSV\n"; }
};
```
**Modern angle:** `run()` stays non-virtual (the "non-virtual interface" idiom); only the hooks are `virtual`. CRTP can do this at compile time when you don't need runtime polymorphism.

---

## 22. Visitor
**Intent:** Add new operations to a class hierarchy without modifying the elements (double dispatch).

```cpp
#include <memory>
#include <vector>

struct Circle; struct Square;

struct Visitor {
    virtual ~Visitor() = default;
    virtual void visit(Circle&) = 0;
    virtual void visit(Square&) = 0;
};

struct Shape {
    virtual ~Shape() = default;
    virtual void accept(Visitor& v) = 0;
};
struct Circle : Shape { double r{}; void accept(Visitor& v) override { v.visit(*this); } };
struct Square : Shape { double s{}; void accept(Visitor& v) override { v.visit(*this); } };

struct AreaVisitor : Visitor {
    double total = 0;
    void visit(Circle& c) override { total += 3.14159 * c.r * c.r; }
    void visit(Square& s) override { total += s.s * s.s; }
};
```
**Modern angle:** Classic double-dispatch via `accept`/`visit`. The modern alternative for a *closed* set of types is `std::variant` + `std::visit` with an overload set — no `accept()` boilerplate:
```cpp
#include <variant>
using Shape = std::variant<Circle, Square>;
struct Area { /* operator() per type */ };
// std::visit(Area{}, shape);
```

---

## Quick decision guide

| You want to… | Pattern | Modern C++ shortcut |
|---|---|---|
| Pick a concrete type at runtime | Factory / Abstract Factory | map of `function<unique_ptr<T>()>` |
| Build an object with many options | Builder | aggregate + designated initializers (C++20) |
| Copy through a base pointer | Prototype | virtual `clone()` |
| One global instance | Singleton | function-local `static` |
| Make interfaces compatible | Adapter | wrapper class by composition |
| Split abstraction from impl | Bridge | hold `unique_ptr<Impl>` |
| Tree of objects | Composite | `vector<unique_ptr<Node>>` |
| Add behavior dynamically | Decorator | wrap `unique_ptr<Base>` |
| Hide a subsystem | Facade | one class owning members by value |
| Share heavy immutable data | Flyweight | pool of `shared_ptr<const T>` |
| Control access to an object | Proxy | same interface, lazy `unique_ptr` |
| Try handlers in order | Chain of Responsibility | `vector<function<bool(Req&)>>` |
| Encapsulate an action | Command | `std::function<void()>` (+ undo class) |
| Traverse a collection | Iterator | `begin()/end()`, ranges, `std::generator` |
| Decouple many-to-many comms | Mediator | hub holding non-owning pointers |
| Snapshot/restore state | Memento | nested `friend` Memento |
| Notify on change | Observer | `vector<function<...>>` / `weak_ptr` |
| Behavior depends on state | State | swap `unique_ptr<State>` / `variant` |
| Swap algorithms | Strategy | `std::function` / template policy |
| Fixed skeleton, variable steps | Template Method | NVI (non-virtual interface) |
| New ops over a hierarchy | Visitor | `std::variant` + `std::visit` |
```
