# What Does Private Inheritance Mean?

When you write:

```cpp
class MyClass : private OtherClass
{
};
```

It means:

* `MyClass` **inherits** from `OtherClass`
* BUT *all* public and protected members of `OtherClass` become **private members** of `MyClass`
* Outside users **cannot treat `MyClass` as an `OtherClass`**

### The key point:

**Private inheritance = “implemented in terms of”**, not “is a kind of”.

---

# Summary Table: Public vs Protected vs Private Inheritance

| Inheritance type | “Is-a” relationship? | Public base members become | User can upcast? |
| ---------------- | -------------------- | -------------------------- | ---------------- |
| **public**       | YES                  | public                     | ✔ allowed        |
| **protected**    | NO                   | protected                  | ❌ not allowed    |
| **private**      | NO                   | private                    | ❌ not allowed    |

Example:

```cpp
class A {};
class B : public A {};    // B is an A
class C : private A {};   // C uses A but C is NOT an A
```

With private inheritance:

```cpp
C c;
A* a = &c;  // ERROR: cannot convert
```

---

# Why Would Anyone Use Private Inheritance?

### Scenario 1: **You want to reuse the implementation of a class**

But not expose its interface to users.

This is common when:

* Need access to protected methods
* Composition isn’t enough
* Want to override some `virtual` methods without “being” that base class

Example:

```cpp
class Vehicle {
protected:
    void startEngine();
public:
    void drive();
};

class Car : private Vehicle {
public:
    void go() {
        startEngine();  // OK
        drive();        // OK
    }
};
```

But:

```cpp
Car c;
c.drive();  // ! Not accessible
Vehicle* v = &c; // ! Cannot upcast
```

The base functionality is re-used **internally only**.

---

### Scenario 2: **“Is-implemented-in-terms-of”, not “is-a”**

This is the philosophical reason.

Example: using `std::vector` as the base of a custom container:

```cpp
class IntStack : private std::vector<int> {
public:
    void push(int x) { push_back(x); }
    int pop() { int v = back(); pop_back(); return v; }
};
```

The user cannot treat `IntStack` as a `vector<int>`:

```cpp
IntStack s;
std::vector<int>& v = s;  // ERROR
```

This is good: the stack interface stays controlled.

---

# Composition vs Private Inheritance

A classic rule:

> If you want “**has-a**”, use composition.
>
> If you want “**is-implemented-in-terms-of**”, use private inheritance.

Comparison:

### Composition (recommended 90% of the time):

```cpp
class MyClass {
private:
    OtherClass o;
};
```

Simple, decoupled, stable.

### Private inheritance (special cases):

```cpp
class MyClass : private OtherClass {};
```

You get:

* Access to **protected members** of the base
* Ability to **override virtual methods**
* Convenience of calling base implementations directly

But you also get:

* Tighter coupling
* Strange behavior with conversions
* More maintenance overhead

---

# Practical Embedded System Use Case

### Example: Hardware Driver Wrappers.

Imagine you want to wrap a legacy C API provided as a struct with function pointers.

```cpp
struct HAL_Driver {
    void (*Init)();
    void (*Send)(int);
};
```

Using private inheritance:

```cpp
class UARTDriver : private HAL_Driver {
public:
    UARTDriver() { Init(); }

    void write(int x) {
        Send(x);
    }
};
```

All HAL functions become private implementation details.

---

# Example: Overriding Virtual Methods Privately

```cpp
class Base {
public:
    virtual void run() = 0;
};

class Derived : private Base {
private:
    void run() override { /* custom behavior */ }
public:
    void execute() { run(); }
};
```

Outside code cannot call `run()` or upcast to `Base`. This is sometimes useful in frameworks or callback systems.

---

# Summary: When Is It Meaningful?

### When you need access to:

* Protected members
* Base class implementation

### But you DO NOT want:

* An “is-a” relationship
* Public exposure of the base class API
* Implicit upcasting

### Good for:

* Adapters
* Wrappers
* Internal reuse
* Implementation hiding
* Customizing behavior while restricting interface

---
