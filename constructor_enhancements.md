# Improved Initialization

## Constructor Enchancements

### Initializing Members Inline

You can initialize members inline, in the declaration. Handy way to define common initialization values across multiple constructors

```cpp
class BankAccount {
    private:
        std::string accountHolder;
        double balance = 0.0;
        int id = generateId();

    public:
        BankAccount(const std::string & acch) : accountHolder(acch)
        {

        }
        ...
};
```

### Delegeting Constructors

You can define delegating constructors. One constructor calls another to reduce duplicate code.

```cpp
class BankAccount {
    private:
        std::string accountHolder;
        

    public:
        BankAccount(const std::string & acch) : BankAccount(acch, getDefaultOverdraft()) // Call the main constructor
        { } // Empty body
        // Rather than do the same things in each constructor, just call the main constructor

        // The main constructor
        BankAccount(const std::string & acch, double od) : accountHolder(acch), overdraft(od)
        {
            // Common behaviour for all constructs here
        }
        ...
};
```

### Inheriting Constructors

You can indicate a derived class should inherit all constructors from the base/parent class

```cpp
class B {
    public:
        B(int);
        B(const string &);
        ...
};

class D : public B 
{
    public:
        using B::B; // Inherit all B's constructors
        ... // You can define additional constructors
        // D (double);
}

// In .cpp file, constructor mapping like below is automatically have done with "using B::B"
D(int i)::B(i) { } // No need if using B::B is used
D(const string & s)::B(s) { } // No need if using B::B is used
```

## Uniform Initialization Syntax

You can use `{}` to initialize all kinds of values:

```cpp
const int val1 {5}; // equals const int val1 = 5;
const int val2 {15}; // equals const int val2 = 15;
```

```cpp
int a[] {1, 2, val1, val1 + val2}; // equals int a[] = {1, 2, val1, val1 + val2};
```

```cpp
struct PointStructure {
    int x, y;
};

const PointStructure p1 {10, 20}; // equals const PointStructure p1 = {10, 20};
```

```cpp
class PointClass {
    public:
        PointClass (int x, int y);
};

const PointClass p2 {30, 40};
```

```cpp
class Widget {
    private:
        const int data[3];
    public: 
        Widget () : data {100, 200, 300} { }
};
```

```cpp
void myFunc(const std::vector<int>& v); // std::vector(std::initializer_list<int> init);

myFunc ({100, 200, 300}); // It works even though its a rvalue
// Why this works without &&:
// const T& can bind to temporaries
// T&& is specifically for move semantics and explicit rvalue binding
// For const references, you don't need && because const references already accept rvalues
// The temporary vector binds to the const std::vector<int>& parameter
```

A common use of the uniform initialization syntax is to simplify STL container code:

```cpp
// Construction
std::vector<int> v { 100, 200, 300};

// Multi-element insertion
v.insert(v.end(), {3, 12, 19, 1, 2, 7}) // std::vecotr::insert(ITERATOR, std::initializer_list<int>)

// Assignment
v = { 25, 12 }; // std::vector::operator=(std::initializer_list<int>)
```

## Initializaer Lists

### `std::initializer_list`

C++ has a type named `std::initializer_list`

If a function has a `std::initializer_list` param:
- Client code can pass in a brace initializer list
- Compiler converts to `std::initializer_list`

```cpp
#include <initializer_list>

void f1 (std::initializer_list<int> nums)
{
    for (auto n: nums)
    {
        std::cout << n << std::endl;
    }
}

f1({ 10, 20, 30 }) // Compiler creates std::initializer_list
```

`std::initializer_list` stores the initializer values in an array internally. It offers these member functions, to allow you to access the values:
- size(): Number of elements in the array // for (auto n: nums)
- begin(): pointer to first array element // for (auto i = nums.begin(); i<nums.end(); ++i); 
- end(): pointer to one-beyond-last array element

You can use `std::initializer_list` with templates

```cpp
template<typename T>
void display_items (std::initializer_list<T> elements)
{
    for (auto e: elements)
        std::cout << e << std::endl;
} 

display_items( { 1, 3, 4 } );
display_items( { "1", "3", "4"} );
```

### Type Deduction with `auto`

If you use `{ }` with an `auto` variable, this is how the compiler deduces the type of the variable:

```cpp
auto x {10}; // Valid: dedecuse x as int
auto y { 10, 20, 30 }; // ! Invalid: can't provide multiple values (It can be one of the vector, list, etc. classes) — Ambiguous
auto z = { 10, 20, 30 }; // Valid (with equals operator): deduces z as std::initializer_list<int>
```

```cpp
auto i1 = 10; // int
auto i2(10); // int
auto i3{10}; // int
auto i4 = {10}; // std::initializer_list<int>

std::vector<int> vec1(i1); // A vector variable with 10 integer size with default (0) values
std::vector<int> vec2(i2); // A vector variable with 10 integer size with default (0) values
std::vector<int> vec3(i3); // A vector variable with 10 integer size with default (0) values
std::vector<int> vec4(i4); // A vector variable with 1 integer size with 10 (1 element size)
```



