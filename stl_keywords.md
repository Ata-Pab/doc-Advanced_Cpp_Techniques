# `std::pair`

`std::pair` is a **lightweight struct** holding **exactly two values** (possibly of different types).

```cpp
template <class T1, class T2>
struct pair {
    T1 first;
    T2 second;
};
```

## Use it when:

* Returning multiple values from a function (without struct definition)
* Associative containers (`std::map`, `std::unordered_map`)
* Utility functions (`std::make_pair`)
* Temporary grouped values
* Embedded code: returning status + data, coordinates, min/max, etc.

```cpp
std::pair<int, float> p = {10, 3.14f};

int a = p.first;
float b = p.second;
```

### With `auto`

```cpp
auto p = std::make_pair("ESP32", 115200);
```

### Function return

```cpp
std::pair<bool, uint8_t> uart_read_byte()
{
    if (data_ready()) return {true, read_register()};
    return {false, 0};
}
```

# `std::forward`

It’s core to **perfect forwarding** and **move semantics**.

- `std::forward<T>(arg)` conditionally performs **move** or **copy** depending on whether the original value was an **lvalue** or **rvalue**.

Used almost exclusively in:

- Generic code
- Templates
- Wrapper functions
- Factory functions
- Variadic templates (`std::forward<Args>(args)...`)

## Why do we need `std::forward`?

Consider a wrapper function that forwards parameters to another function:

```cpp
template<typename T>
void wrapper(T&& x) {
    foo(x);
}
```

Here, **x is always an lvalue**, because it has a name.
So even if you pass an rvalue:

```cpp
wrapper(10);
```

Inside wrapper:

* `x` becomes an lvalue
* `foo(x)` will call `foo(const int&)` instead of `foo(int&&)`

This is **wrong** if we want to preserve the value category.

## Solution: `std::forward`

```cpp
template<typename T>
void wrapper(T&& x) {
    foo(std::forward<T>(x));
}
```

Now:

* If caller passed an **lvalue**, `forward` returns lvalue.
* If caller passed an **rvalue**, `forward` returns rvalue.

This is **perfect forwarding**.

```cpp
void foo(const std::string& s) {
    std::cout << "Lvalue\n";
}

void foo(std::string&& s) {
    std::cout << "Rvalue\n";
}

template<typename T>
void wrapper(T&& arg) {
    foo(std::forward<T>(arg));
}
```

```cpp
std::string a = "Hello";
wrapper(a);          // Lvalue version
wrapper("World");    // Rvalue version
```

## Embedded/Practical Example

Imagine you have a message builder API:

```cpp
template<class T>
void send_msg(T&& payload) {
    uart_send( prepare_buffer(std::forward<T>(payload)) );
}
```

If `payload` is:

* a temporary buffer → moved
* a static/global buffer → copied

This keeps code efficient without duplicating overloads.


# `std::optional`

`std::optional` is a way of specifying a possibly empty value, similar to a **nullable** column in a database table.

```cpp
#include <optional>

std::optional<int> getQuotient (int n, int d)
{
    if (d != 0)
        return n / d;
    else
        return {}; // No value is returned
}

std::optional<int> result = getQuotient (10, 2);

if (result)
    std::cout << "Value: " << *result << std::endl;
else
    std::cout << "No value" << std::endl;
```

# `std::string_view`

`std::string_view` is a non-owning view into the actual text of a `std::string`. Internally, it just contains a text pointer and a length. Cheaper to use than `std::string`.

```cpp
#include <string_view>

std::string msg = "This is the code example";
const char* p = &msg.at(msg.find_first_of('c'));

std::string s(p); // Copy/Clone string
std::cout << s << std::endl; // "code example"

std::string_view sv(p); // Points to the p
std::cout << sv << std::endl; // "code example"
```

# `std::any`

`std::any` is a container of any type of item

```cpp
#include <any>

auto a = std::any(43);

a = std::string("John");
a = 3.16;
```

To access the value, you must use `std::any_cast<T>`

```cpp
#include <any>

auto a = std::any(43);

std::cout << std::any_cast<int>(a) << std::endl;

try {
    std::cout << std::any_cast<std::string>(a) << std::endl;
} catch (std::bad_any_cast& e) {
    std::cout << e.what() << std::endl;
}
```

# `std::variant`

`std::variant` is a type-safe union. A better way to overlay various data types on same space. You must declare the types allowable in the space.

```cpp
#include <variant>

std::variant<int, double> num;
```

You can get the value out of the variant via `std::get()`
- `std::get()<type>(aVariant)`
- `std::get()<index>(aVariant)`

```cpp
#include <variant>

std::variant<int, double> num; // type-0 is `int` and the type-1 is `double`

num = 42;

std::cout << std::get<int>(num) << std::endl;
std::cout << std::get<0>(num) << std::endl;

num = 3.16;

std::cout << std::get<double>(num) << std::endl;
std::cout << std::get<1>(num) << std::endl;
```

You can get a pointer to the value via `std::get_if()`. It returns `nullptr` if the value isn't that type.


```cpp
std::variant<int, double> num;

num = 3.16;

if (const auto pInt = std::get_if<int>(&num); pInt) {
    std::cout << "Value: " << *pInt << std::endl;
} else if (const auto pDouble = std::get_if<double>(&num); pDouble) {
    std::cout << "Value: " << *pDouble << std::endl;
}
```

---

