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

Indicates a function return value can be computed at **compile time**, if the calling code requires it. It provides to run code more quickly because it already knows the result (calculated it in compile time).

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

### `consteval` (C++20)

Specifies a function is an immediate function.

```cpp
consteval int getNumPlayers (int numTeams)
{
    return 11 + numTeams;
}

string players[getNumPlayers(3)]; // getNumPlayers is evaluated at compile-time, so this is valid declaration
```

❗You can't use `consteval` functions in a context that requires a runtime evaluation like variable initialization/declaration.

### `constinit` (C++20)

`constinit` pertains/related to static-storage variables. Mandates/denotes a variable must be initialized at compile-time.
- Avoids `static initialization order` problem of run-time init.
- `constinit` cannot be used together with `constexpr`. When the declared variable is a reference, constinit is equivalent to `constexpr`.
- `constinit` does not mandate **constant** destruction and const-qualification. **Its value can be changed** later at runtime.

```cpp
constinit int myGlobalVar = 42; // compile-time constant
```

```cpp
constexpr int square(int i)
{
    return i * i;
}
 
int twice(int i)
{
    return i + i;
}
 
constinit int sq = square(2);    // OK: initialization is done at compile time
// constinit int x_x = twice(2); // Error: compile time initializer required
```