# Variables, Addresses, and Memory

Every variable has:

1. A **value**
2. An **address** (memory location)

```cpp
int x = 10;
```

* `x` → stores value `10`
* `&x` → is the address of `x`

---

# Pointers

A pointer is a variable whose value is **an address**.

```cpp
int x = 10;
int* p = &x;   // p stores address of x
```

Meaning:

* `p` → the pointer itself (an address)
* `*p` → value at that address (10)

---

# Const Pointer Types

A key point: *const can apply to the pointer itself, or the data it points to, or both.*

### **Pointer to const data**

Pointer can change, data cannot.

```cpp
const int* p = &x;
```

* ❌ `*p = 5;`       (data is read-only)
* ✔ `p = &y;`       (pointer can point elsewhere)

### **Const pointer to non-const data**

Pointer cannot change, data can.

```cpp
int* const p = &x;
```

* ✔ `*p = 20;`     (data can change)
* ❌ `p = &y;`     (pointer cannot move)

### **Const pointer to const data**

```cpp
const int* const p = &x;
```

* ❌ `*p = 20;`
* ❌ `p = &y;`

---

# References

A reference is an alias for another variable.
After it’s initialized, it **cannot be reseated**.

```cpp
int x = 10;
int& r = x;
```

* `r` **is** `x` (not a pointer)
* `r = 20;` modifies `x`

### References vs Pointers

| Feature              | Pointer     | Reference   |
| -------------------- | ----------- | ----------- |
| Can be null?         | ✔ Yes       | ❌ No        |
| Must be initialized? | ❌ No        | ✔ Yes       |
| Can be reseated?     | ✔ Yes       | ❌ No        |
| Syntax               | `*p`, `p->` | `r`, `obj.` |

References are safer and more idiomatic in C++.

---

# Const Reference

Often used for efficient and safe passing of large objects:

```cpp
void print(const std::string& s);
```

Benefits:

* No copy
* Cannot modify
* Works with temporaries

---

# Double Pointers (`int**`)

A pointer that stores the address of another pointer.

```cpp
int x = 5;
int* p = &x;
int** pp = &p;
```

Useful for:

* Dynamic allocation APIs
* Returning a pointer from a function
* Modifying a pointer argument
* Multidimensional arrays

Example: modifying a pointer inside a function:

```cpp
void allocate(int** ptr)
{
    *ptr = new int(42);   // modifies caller's pointer
}

int* p = nullptr;
allocate(&p);
```

---

# Double References (`int&&`, `int& &`)

**C++ does NOT allow double lvalue references**, but it *does* allow **rvalue references (`&&`)**.

They are used for:

* Move semantics
* Perfect forwarding
* Optimizing away copies

Minimal example:

```cpp
void take(int&& x)
{
    int y = x; // ok
}
```

### Forwarding reference (T&&)

Used in templates:

```cpp
template<typename T>
void forward(T&& arg)
{
    f(std::forward<T>(arg));
}
```

If you want `double reference` meaning “reference to a reference”, C++ collapses references:

| T       | Incoming Ref | Result  |
| ------- | ------------ | ------- |
| `int&`  | `&`          | `int&`  |
| `int&&` | `&`          | `int&`  |
| `int&`  | `&&`         | `int&`  |
| `int&&` | `&&`         | `int&&` |

This rule ensures C++ references remain simple and consistent.

---

# Passing Variables to Functions

## **Pass by Value**

Copy of the variable.

```cpp
void f(int x);  // copy
```

---

## **Pass by Pointer**

Enables:

* modifying caller variable
* passing null
* dynamic allocation patterns

```cpp
void inc(int* p)
{
    (*p)++;
}

int x = 10;
inc(&x);     // x becomes 11
```

---

## **Pass by Reference**

Cleaner and safer than pointers:

```cpp
void inc(int& r)
{
    r++;
}

int x = 10;
inc(x);     // x becomes 11
```

---

## **Pointer to Pointer in Function**

Modify caller’s pointer:

```cpp
void create(int** ptr)
{
    *ptr = new int(99);
}

int* p = nullptr;
create(&p);
```

---

## **Reference to Pointer**

Simpler version of above:

```cpp
void create(int*& ptr)
{
    ptr = new int(99);
}

int* p = nullptr;
create(p);   // easier syntax
```

---

# Function Overloads Demonstrating Behavior

```cpp
void foo(int value);       // by value
void foo(int* ptr);        // by pointer
void foo(int& ref);        // by reference
void foo(const int& ref);  // by const reference
```

Which one is chosen depends on argument type.

---

# Practical Embedded/System Examples

## **I/O buffer provided by caller**

```cpp
void getSensorData(uint8_t* buffer, size_t len);
```

## **C-style handle allocation**

```cpp
bool initDevice(DeviceHandle** h)
{
    *h = new DeviceHandle;
    return (*h != nullptr);
}
```

## **Reference simplifies code**

```cpp
void scale(float& value, float factor)
{
    value *= factor;
}
```

---

# Summary Table

| Technique            | Can modify caller var? | Can be null? | Safe? | C style?      |
| -------------------- | ---------------------- | ------------ | ----- | ------------- |
| Value                | ❌                      | —            | ✔     | ✔             |
| Pointer              | ✔                      | ✔            | ⚠     | ✔             |
| Const Pointer        | ✔ (depending)          | ✔            | ✔     | ✔             |
| Reference            | ✔                      | ❌            | ✔✔✔   | C++ only      |
| Const Reference      | ❌                      | ❌            | ✔✔✔   | C++ idiomatic |
| Pointer-to-pointer   | ✔✔                     | ✔            | ⚠⚠    | ✔             |
| Reference-to-pointer | ✔✔                     | ❌            | ✔     | C++ only      |

---
