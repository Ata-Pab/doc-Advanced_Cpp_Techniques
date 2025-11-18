## Overview of `nullptr` and `nullptr_t`

`nullptr` is used to represent a null pointer rather than using `NULL` or `0`.

```cpp
const char *p1 = nullptr;

if (p1) { ... } // Valid: Conversion from nullptr to bool

int i = nullptr // Invalid: No conversion from nullptr to int
```

```cpp
int *p1 = nullptr;

int *p2 = NULL;

int *p3 = 0;

if (p1 == p2 && p1 == p2) { ... } // Valid (The test succeeds, passes the if condition) 
```

## Overview of Constant-ness

- `constexpr`
- `consteval` (C++20)
- `constinit` (C++20)

### `constexpr` functions

Indicates a function return value can be computed at compile time, if the calling code requires it. It provides to run code more quickly because it already knows the result (calculated it in compile time).

```cpp
constexpr int multiply (int x, int y)
{
    return x * y;
}
```

### `constexpr` variables

Indicates the variable must be initialized at compile-time. `constexpr` variables are implicitly `const`.

```cpp
constexpr int val = multiply(5 , 10);
```