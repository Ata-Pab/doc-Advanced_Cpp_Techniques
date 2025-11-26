# Modern C++ General Enhancements

## Variable Declarations

You can declare variables in `if` and `switch`

```cpp
if (auto m = getMonth(); m== 5)
    std::cout << "June" << std::endl;
else
    std::cout << "Not June" << std::endl;
```

```cpp
switch (auto m = getMonth(); m+1)
{
    case 6:
    case 7:
    case 8:
        std::cout << "Summer" << std::endl;
        break;
    default:
        std::cout << "Not Summer" << std::endl;
}
```

## Structured Binding

```cpp
int array [] = { 10, 20, 30 };

auto [a, b, c] = array; // Copies of array elements
auto & [d, e, f] = array; // References
const auto& [h, h, i] = array // Const references
```

```cpp
Point point = { 10, 20, 30};

auto [a, b, c] = point; // Copies of data members x, y, z
auto & [d, e, f] = point; // References
const auto& [h, h, i] = point // Const references
```
The number of identifiers must match:
- The number of elements in array
- The number of non-static data members in an object

## 3-way Comparison Operator <=> (spaceship operator)

Combines ==, !=, <, >, <=, >= into a single operator.

```sh
(A <=> B) < 0 is true if A < B
(A <=> B) > 0 is true if A > B
(A <=> B) == 0 is true if A and B are equal/equivalent.
```

```cpp
Struct Money {
    int value;
    Money (int v): value {v} {}

    std::strong_ordering operator <=> (const Money& m2) const {
        return value <=> m2.value;
    }
}
```

```cpp
Money m1(100), m2(200);

std::strong_ordering result = (m1 <=> m2);

// strong_ordering can be compared against 0 (via ==, <, >, etc.)
if (resutl == 0) {
    ...
}
```

OR just use:

```cpp
Money m1(100), m2(200);

if ((m1 <=> m2) == 0) {
    ...
}
```

## Designated Initializers

Allows you to set fields in an aggregate initialization.

```cpp
struct Point {
    int x = 0;
    int y = 0;
    int z = 0;
};

Point p1 {
    .x = 100,
    .y = 200,
    .z = 150
};
```

## Explicit Overrides

A subclass can qualify a method as `override`:
- The compiler verifies that it correctly overrides a `virtual` method from the superclass

```cpp
class MySuperClass {
    virtual void myFunc(int);
    ...
};

class MySubClass : public MySuperClass {
    void myFunc(int) override; // The override keyword provides to verify correct function is overridden
    ...
};
```

## Static Asserts

You can verify that a **compile-time expression** is true, via the `static_assert` keyword. Generally be used in template classes and template functions.

See the [Reference](https://en.cppreference.com/w/cpp/meta).
```cpp
template <typename T>
struct Check
{
    static_assert(sizeof(int) <= sizeof(T), "T is not big enough!");
}
```

```cpp
template <typename T>
T func(T x, T y)
{
    static_assert(std::is_integral<T>::value, "Must be an integral!"); // string variables will cause compiler error
}
```

## `using` vs `typedef`

You can use `using` statements as an alternative to `typedef` statements.

```cpp
typedef std::unordered_set<int> InSet;
// Equals
using InSet = std::unordered_set<int>;
```

```cpp
typedef void (*CallBackPtr) (int);
// Equals
using CallBackPtr = void (*)(int);
```

## Variadic Templates

Templates can take any number/type of parameters. 

```cpp
template <typename... Types>
class tuple; 

std::tuple <int, char, double> myTuple;
```

```cpp
template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... params); 

class Employer {
    string name;
    int age;
    ...
}

std::make_shared<Employer>("Atalay", 21, "Turkey");
```

A variadic template has at least **one parameter pack**. A parameter packs is a template parameter that accepts zero or more **template arguments**.
- In a vaiadic function, you can use a **pack expansion**
- Call another variadic function, and pass into it a pattern followed by `...` .

```cpp
template <typename... ArgsF1>
void f1(ArgsF1... args)
{
    // Do something with args...
}

template <typename... ArgsF2>
void f2(ArgsF2... args)
{
    f1 (&args...); // &args... is a pack expansion, &args is its pattern
}


f2(1, 0.2, "A"); // f2 (int p1, double p2, const char* p3)
// f1 (&args...) is like f1 (&p1, &p2, &p3);
// and f1(ArgsF1... args) is like f1 (int *p1, double * p2, const char ** p2)
```

### Fold Expressions in Variadic Templates

You can use `fold` expressions to unpack variadic args

```cpp
template <typename... T>
void myFunc (T... args)
{
    // Do something with args here
}

// myFunc(arg0, arg1, arg2)

(args* ...) // Becomes (args0 * (args1 * args2))
(... * args) // Becomes ((args0 * args1) * args2)
(args * ... * 1) // Becomes (args0 * (args1 * (args2 * 1)))
(1 * ... * args) // Becomes (((1 * args0) * args1) * args2)
```

```cpp
template <typename... T>
auto product (T... nums)
{
    return (nums * ...);
}

int n = product(10, 20, 30, 40);
```

```cpp
template <typename... T>
void printArgs(T... args)
{
    (std::cout << ... << args);
    // Equals: ((((((std::cout << "Swans ") << 3) << " Cardiff ") << 0) << "!!!") << "\n"); 
    // args on the right hand side — start from the right and move to the left
}

echo ("Swans ", 3, " Cardiff ", 0, " !!!", "\n");
```

## Concepts

A concept is a compile-time named set of requirements for some data type. A standard concept defined in `<concepts>`. 
- A concept can specify a requires expression. It enables you to define requirements for type parameters

```cpp
template <typename T>
concept std::integral = std::is_integral<T>::value;

// You can use a concept as follows
template <std::integral T>
void thisFuncTakesIntegralOnly (T val) {
    ...
}
```

- Concepts are higher level `type` specifications for templates. Specify a type parameter must support `+`:

```cpp
template <typename T>
concept Addable = requires (T x) { x + x; };
// For example, you have a class and want to use Addable concept, you should have operator+ overloading. Otherwise it fails. It places the requirements for the incoming type.

template <Addable T>
void thisFuncTakesAddablesOnly(T a, T b)
{
    T result = a + b;
}
```