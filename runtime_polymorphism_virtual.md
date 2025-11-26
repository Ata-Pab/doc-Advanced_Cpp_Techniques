# `virtual` in C++

**`virtual` enables run-time polymorphism.**
It allows C++ to choose which function to call **based on the actual object type**, not the pointer/reference type.

- Without `virtual`: C++ uses **static (compile-time) dispatch**.
- With `virtual`: C++ uses **dynamic dispatch** through **vtable** (virtual table).

# `virtual` Functions

A member function declared with `virtual`:

```cpp
class Base {
public:
    virtual void print() {
        std::cout << "Base\n";
    }
};
```

If a derived class overrides it:

```cpp
class Derived : public Base {
public:
    void print() override {
        std::cout << "Derived\n";
    }
};
```

### Example:

```cpp
Base* p = new Derived();
p->print();   // output: Derived   (because of virtual!)
```

## How it works internally: VTABLE ? 

With virtual functions:

* Compiler builds a **vtable** for each class.
* Each object contains a hidden pointer: **vptr**.
* `vptr` points to the vtable of the actual class.
* Function call: `p->print()` -> vtable lookup -> correct function.

Without virtual -> no vtable -> no runtime dispatch.

### Embedded consideration:

* A vptr costs typically **4 bytes** (32-bit target).
* Each virtual function call costs a small indirection.

But still used in many embedded C++ projects (drivers/HAL interfaces).

# Pure Virtual Functions (Abstract Interfaces)

A function can be marked **pure virtual**, making the class **abstract**:

```cpp
class Device {
public:
    virtual void init() = 0;
    virtual void deinit() = 0;
};
```

This class **cannot be instantiated**. It is effectively an **interface**.

### Derived class must implement the pure virtual methods:

```cpp
class UartDevice : public Device {
public:
    void init() override { /* ... */ }
    void deinit() override { /* ... */ }
};
```

# Abstract Classes (Interfaces)

A class with at least one pure virtual function is an **abstract class**.

Typical embedded pattern (HAL abstraction):

```cpp
class Uart {
public:
    virtual bool init(int baud) = 0;
    virtual bool write(const uint8_t* data, size_t len) = 0;
    virtual bool read(uint8_t* data, size_t len) = 0;
};
```

Derived implementations:

```cpp
class Esp32Uart : public Uart {
public:
    bool init(int baud) override { /*...*/ }
    bool write(const uint8_t* data, size_t len) override { /*...*/ }
    bool read(uint8_t* data, size_t len) override { /*...*/ }
};
```

Usage:

```cpp
Uart* uart = new Esp32Uart();
uart->init(115200);
```
# The `override` keyword

Always use it.

```cpp
void print() override;
```

* Compiler ensures you are truly overriding a **virtual** function.
* Catches mistakes:

```cpp
void pritn() override;  // error: no function to override
void print(); // Valid but the compiler does not know which function do you exactly override, acts like a new function declaration
```

# Virtual Destructor

If a class is intended for polymorphic deletion, its destructor should be virtual:

```cpp
class Base {
public:
    virtual ~Base() {}
};
```

Deleting via base pointer:

```cpp
Base* p = new Derived();
delete p;
```

Without virtual destructor -> **undefined behavior** (derived destructor won't run).

# Virtual Base Classes (Virtual Inheritance)

This is different from virtual functions. Used to solve the **diamond problem**.

### Diamond problem:

```
       Base
      /    \
     /      \
    /        \
Derived1   Derived2
    \        /
     \      /
      \    /
       Final
```

Both Derived1 and Derived2 include their own copy of `Base`.

### Solution: **virtual inheritance**

```cpp
class Base { ... };

class Derived1 : virtual public Base { ... };
class Derived2 : virtual public Base { ... };

class Final : public Derived1, public Derived2 { ... };
```

- Only **one** Base subobject exists.

This is used in some complex architectures (compilers, plugin frameworks), but rare in embedded/HAL designs.

# Non-virtual functions cannot be overridden

If a function is non-virtual, the type used at compile time decides the call:

```cpp
class A {
public:
    void foo() { std::cout << "A\n"; }
};

class B : public A {
public:
    void foo() { std::cout << "B\n"; }
    // void foo() override { std::cout << "B\n"; }  // ERROR!
    /* The override keyword is specifically designed to enforce that you're overriding a virtual function. If there's no virtual function to override, the compiler catches this as a mistake.
    */
};

A* p = new B();
p->foo();   // prints "A"
```

Because foo is not virtual.

# `final` keyword

To prevent further overrides:

```cpp
class Derived : public Base {
public:
    void print() override final;
};
```

You can also finalise a whole class:

```cpp
class FinalClass final { };
```

# Embedded Example

UART driver:

```cpp
class Uart {
public:
    virtual bool init() = 0;
    virtual void write_byte(uint8_t c) = 0;
    virtual uint8_t read_byte() = 0;
    virtual ~Uart() = default;
};
```

Hardware implementation:

```cpp
class STM32Uart : public Uart {
public:
    bool init() override { /* HAL_UART_Init */ }
    void write_byte(uint8_t c) override { /* ... */ }
    uint8_t read_byte() override { /* ... */ }
};
```

Mock implementation for unit tests:

```cpp
class MockUart : public Uart {
public:
    bool init() override { return true; }
    void write_byte(uint8_t) override { recorded++; }
    uint8_t read_byte() override { return 42; }
};
```

Code using polymorphism:

```cpp
void uart_protocol(Uart& uart) {
    uart.write_byte(0xAA);
    uint8_t x = uart.read_byte();
}
```

---

