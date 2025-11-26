# Value Categories and Movability

Prior to C++11

## lvalue (left side value -  traditional variable)
Has identity and is addressable.

```cpp
ìnt i = 21;
i = 42;
int *pl = &i;
```
```cpp
/*
int & f1() {
    static int a[5];
    ...
    return a[0]; // &a[0]
}
*/

int &f1();
f1 () = 42;
int* p2 = &f1();
```

## rvalue (right side value - temporary value)
Does not have identity and is not addressable.

```cpp
21 = 42; // ! Invalid syntax
int * pl = &42; // ! Invalid syntax
```

```cpp
/*
int  f1() {
    static int a[5];
    ...
    return a[0];
}  // returns value, not the reference
*/

int f1();
f1() = 42; // ! Invalid syntax
int * p2 = &f1();  // ! Invalid syntax
```

> Since C++11 there are 3 value categories

- **lvalue**: same as discussed lvalue above - has identity and is addressable
- **xvalue**: an `expiring value` - has identity but is not addressable
- **prvalue**: a `pure rvalue`, same as rvalue - no identity and not addressable

Also there are 2 mixed value categories
- **glvalue**: generalized lvalue - either an `lvalue` or an `xvalue` - has identity
- **rvalue**: either an `xvalue` or a `prvalue`

## rvalue References

> C++ introduced the concept of `rvalue references`
- An rvalue reference can only refer to an rvalue
- Use the syntax: `Type&&`
- `xvalue` or a `prvalue` => `Type&&`

Typical usage of rvalue references:
- To declare a parameter that can only refer to an rvalue

```cpp
class Widget { ... }

void myFunc(Widget && rw) { ... }
```

It is valid to pass rvalue into functions which gets rvalue reference.

```cpp
Widget createWidget() {
    Widget w;
    ...
    return w;
}

myFunc(createWidget()); // Valid
```

```cpp
Widget w; // lvalue (has address)

myFunc(w); // ! Invalid - Syntax Error
```

# Copy and Move Semantics

Imagine a class that holds resources (e.g., buffer)
- You should define a `copy ctor` and `assignment operator`
- These functions should perform a deep copy

```cpp
class Widget {
    private:
        SomeDataStructure * data;
    public:
        Widget (const Widget&);
        Widget& operator=(const Widget&);
        ...
};
```

```cpp
Widget createWidget() {
    Widget w;
    ...
    return w;
}

...


Widget myWidget;

myWidget = createWidget(); // createWidget returns a temporary Widget
// The assignment operator clones it into myWidget then The temporary Widget is then discarded (inefficient) — The whole created big class object discarded
```

How could this be improved?
- A class can implement both copy and move semantics

**widget.hpp**
```cpp
class Widget {
    private:
        SomeDataStructure * data;
        ...
    public:
        // Copy Semantics — Clone
        Widget(const Widget&);
        Widget& operator=(const Widget&);

        // Move Semantics — Receive rvalue (i.e. object can be moved)
        // Moves ownership the data — "steal" the value of incoming object
        Widget(Widget&&) noexcept;
        Widget& operator=(Widget&&) noexcept;
        ...
}
```
**widget.cpp**
```cpp
class Widget {
    private:
        SomeDataStructure * data;
        ...
    public:
        // Implementing Move Semantics
        Widget(Widget && other) noexcept
        {
            data = other.data;    // assign to the same reference/address
            other.data = nullptr; // now we can set it to nullptr
        }
        // Usage: Widget w1(tempWidget); // tempWidget is an rvalue

        Widget& operator=(Widget && other) noexcept
        {
            if (this != &other)
            {
                delete data;
                data = other.data;
                other.data = nullptr;
            }
            return *this;
        }
        /* Usage: 
        Widget w1;
        w1 = tempWidget; // tempWidget is an rvalue
        */
        ...
}
```

### Tip:
```cpp
data = other.data;
other.data = nullptr;
// Equals
data = std::exchange(other.data, nullptr);
```

## Copying `lvalues`

If the source is an `lvalue`: 
- The `lvalue` continues to exist afterwards
- So, it must be copied (to preserve the original value) — You can't "steal" the data for the `lvalue`

```cpp
Widget w1;
...

// w1 is an lvalue, so this statement must use "copy semantics".

Widget w2 (w1); // Ok to keep using w1 here
```

## Moving `rvalues`

If the source is an `rvalue`:
- The value can be moved, rather than copied.
- More **efficient**

```cpp
Widget w1;
...

// createWidget() is rvalue, so this statement can use move semantics.
w1 = createWidget();
```

# Reference Binding Rules

## `lvalue` Reference Rules

If you declare an `lvalue` reference:
- It can bind to an `lvalue` (and change it)
- It can not bind to an rvalue (changes would be lost)

```cpp
void f1 (Widget &); // Takes an lvalue reference

Widget w;
f1(w); // Valid — can bind to an lvalue

f1(createWidget()); // Invalid — can't bind to an rvalue
```

## Const `lvalue` Reference Rules

If you declare a const `lvalue` reference:
- It can bind to an `lvalue`
- It can bind to an `rvalue` (because it won't change the value)

```cpp
void f2 (const Widget &); // Takes const lvalue reference

Widget w;
f2(w); // Valid — can bind to an lvalue

f2(createWidget()); // Valid — can bind to an rvalue due to its constantness
```

## `rvalue` Reference Rules

If you declare a const `rvalue` reference:
- It can not bind to an `lvalue` (because it vacates the object)
- It can bind to an `rvalue` 

```cpp
void f3 (Widget &&); // Takes rvalue reference

Widget w;
f3(w);  // ! Invalid — Can not bind to an lvalue

f3(createWidget()); // Valid — Can bind to an rvalue
```

## Const `rvalue` Reference Rules

Const `rvalue` reference can never be declared. 
- `rvalue` references typically need to vacate the object
- They can't be const


# AdditionaL Considerations

# What is `xvalue` ? 

An `xvalue` has identity, but it behaves like an `rvalue`.


<img src="./assets/images/xvalue_representation.png" width=640>

```cpp
void func (Type && arg)
{
    // arg value is rvalue ref, so it can utilize "move semantics"
}
```

You can obtain an `xvalue` via the `std::move()` function
- Receives an object as a parameter, returns it as an `xvalue`
- The `xvalue` will thereafter benefit from move semantics
- `std::move()` -> **convert lvalue into xvalue**

**Swap Function Example**
```cpp
#include <utility>

template <typename T>
void myswap(T & a, T & b)
{
    T tmp(std::move(a)); // Move a's data to tmp

    a = std::move(b); // Move b's data to a
    b = std::move(tmp) // Move tmp's data to be
}
```

## Universal References

If you use `&&` on a `template` type parameter:
- It's known as a universal reference
- Can bind to either an `lvalue` or `rvalue`

```cpp
#include <utility>

template <typename T>
void tf(T && arg); // universal reference (only in templates)

Widget w;

tf(w); // Pass an lvalue, so arg is Widget &

tf(Widget()); // Pass an rvalue, so arg is Widget &&
```

## Summary of the value types
- An `lvalue` ([...]) designates a function or an object. [...]
- An `xvalue` (an “eXpiring” value) also refers to an object [...]
- A `glvalue` (“generalized” lvalue) is an lvalue or an xvalue.
- An `rvalue` ([...]) is an xvalue, a temporary object or subobject thereof, or a value that is not associated with an object.
- A `prvalue` (“pure” rvalue) is an rvalue that is not an xvalue. [...]

https://stackoverflow.com/questions/20850536/when-does-lvalue-to-rvalue-conversion-happen-how-does-it-work-and-can-it-fail