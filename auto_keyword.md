# The `auto` keyword in C++

`auto` **is a type-deduction keyword** (introduced in **C++11**) in C++ that tells the compiler:

> **“Deduce the variable’s type from the initializer.”**

```cpp
auto x = 10;      // int
auto y = 2.5;     // double
auto p = &x;      // int*
auto s = "abc";   // const char*
```

The compiler **infers the exact type** from the right-hand side.

## Where to use `auto` keyword, benefits

### 1. Avoid long/complex type names

```cpp
std::vector<std::pair<std::string, int>>::iterator it = v.begin();

// Equals

auto it = v.begin();
```

### 2. Easier working with templates, lambdas, iterators

```cpp
auto lambda = [](int a, int b) { return a + b; };
```

### 3. Avoids duplicating types (less error-prone)

```cpp
const std::vector<int> v{1, 2, 3};
auto it = v.begin();   // deduces vector<int>::const_iterator
```

### 4. Forces initialization

This fails:

```cpp
auto x;   // ERROR: auto requires initializer
```

It prevents uninitialized variables.

## Rules & Behaviors

### 1. `auto` strips references and cv-qualifiers unless you specify them

```cpp
int a = 5;
int& r = a;
auto x = r;    // x is int, NOT int&
auto& y = r;   // y is int& (correct)
```

```cpp
const int c = 10;
auto x = c;       // int
const auto y = c; // const int
```

```cpp
auto x = 42;
const auto *p = &x; // const int*
```

```cpp
std::map<int, std::string> mymap
const auto& refm = mymap; // const std::map<int std::string> & 
```

```cpp
// const iterators
auto i = mymap.cbegin(); // std::map<int, std::string>::cont_iterator
```

### 2. `auto` and `auto&`

| Declaration        | Meaning                                    |
| ------------------ | ------------------------------------------ |
| `auto x = expr;`   | copy (value)                               |
| `auto& x = expr;`  | reference                                  |
| `auto&& x = expr;` | universal reference (forwarding reference) |

---

### 3. `auto` with functions (C++14 return type deduction)

#### Return type deduction:

```cpp
auto add(int a, int b) {
    return a + b;   // compiler deduces “int”
}
```

#### But if return types conflict:

```cpp
auto foo(bool b) {
    if (b) return 1;     // int
    else   return 2.5;   // double => ERROR
}
```

---

### 4. `auto` with `decltype(auto)`

`decltype` lets you declare a variable to be the same type as another variable

```cpp
int x = 42;
decltype(x) y = x; // int
```

Sometimes you want to preserve reference/const:

```cpp
decltype(auto) f(int& x) {
    return (x); // returns int&
}
```

Whereas normal auto:

```cpp
auto f(int& x) {
    return (x); // returns int (copy)
}
```

---

## Advanced Use-cases

### 1. Range-based loops

```cpp
for (auto& elem : container)
```

### 2. Lambda parameters (C++20)

```cpp
auto f = [](auto a, auto b) { return a + b; };
```

### 3. Structured bindings (C++17)

```cpp
auto [x, y] = std::pair{10, 20};
```

### 4. Perfect forwarding

```cpp
template<typename T, typename... Args>
auto make(Args&&... args) {
    return T(std::forward<Args>(args)...);
}
```

```cpp
template <typename T, typename Data>
void func(const T & builder)
{
    Data data = builder.createObject();
}

/*
 Class MyData { ... }
 ...
 Class MyBuilder {
    ... 
    MyData createObject()
 }
*/
MyBuilder builder;
func<MyBuilder, MyData>(builder);

// Equals
template <typename T>
void func(const T & builder)
{
    auto data = builder.createObject();
}

MyBuilder builder;
func(builder);
```

### 5. CTAD (Class Template Argument Deduction)

```cpp
std::pair p(10, 20);  // C++17
auto [a, b] = p;
```


