# Smart Pointers in Modern C++

Typical use of smart pointers:
- Exception safety
- Ensures client conde releases dynamic resoruces, without a proliferation of try/catch statements in every function
- Smart pointers are like objects that behave like pointers with exception safety and comply **RAII** (Resource Acquisition Is Initialization)

```cpp
void myFunc() {
    ,,,
    MyClass *p = new MyClass;
    ...
    // ⚠ An exception occurs here and program exits from this function
    ...
    delete p; // ! The memory free command was never called due to exception (Memory Leakage)
}
```

C++ originally had a `std::auto_ptr` smart pointers (old version C++):
- Supports normal pointer operations on dynamic objects
- Deletes object when `std::auto_ptr` goes out of scope
- Avoids possible memory  leaks
- Deprecated in C++11, removed in C++17

```cpp
void myFunc() {
    ,,,
    std::auto_ptr *p = new MyClass;
    ...
    // ⚠ An exception occurs here and program exits from this function with deleting the created object from the memory
    ...
}
```

C++11 onwards has a smart pointer library that provides many more types of "smart" pointer semantics. Specifically, the smart pointer library offers the following types of smart pointers in `<memory>`.
- `std::shared_ptr`: Object ownership **shared** among multiple pointers
- `std::weak_ptr:` Non-owning observers of shared object
- `std::unique_ptr`: Single owner of shared resource

## Shared Pointers — `std::shared_ptr`

`std::shared_ptr` is a non-exclusive pointer to a dynamically allocated object
- Uses a **reference counter (RC)** internally (assumes all of the created object are on the **heap**)
- Ownership of an object can be shared by any number of other shared pointers
- The shared object is not released until the last shared pointer referencing the object is destroyed
- Use `std::shared_ptr` whenever you want reference counting
- You can use a shared pointer with the STL containers because `std::shared_ptr` supports copying semantics

### Modifier Methods:

- **operator=()** : You can change/assign a new point between two shared pointers. [Ref](https://cplusplus.com/reference/memory/shared_ptr/operator=/)
- **reset()** : Stop pointing anywhere. [Ref](https://cplusplus.com/reference/memory/shared_ptr/reset/)
- **swap()** : Swapping pointing references between two shared pointers [Ref] (https://cplusplus.com/reference/memory/shared_ptr/swap/)


### Query Methods:
- **use_count():** : It gives the current reference count for the given point 
- **unique():** : It gives the boolean value/result for the answer of "Am I the only owner of this point"
- **operator bool ()** : It gives the boolean value/result for the answer of "Is this shared pointer whether null"

### Object-access Methods:
- **get()** : Return back a pointer to the object 
- **operator\* ()** : Reference of the shared pointer
- **operator-> ()** : To invoke an element of the object (functions, variables)

```cpp
#include <memory>

int main ()
{
    std::shared_ptr<int> p1 (new int(100)); // Points for the new variable on the heap (int(100))
    std::shared_ptr<int> p2(p1); // Copy construct -> p1, ICREMENTs the RC
    p1.reset(new int (200)); // Point different variable on the heap, DECREMENTs the RC of the p2
}
```

### `std::make_shared()` Method

`std::make_shared()` allocates the dynamic object and the RC in a single allocation. More efficient than allocating them separately. Always preferring to use `std::make_shared()` is more efficient and performant.

- Allocating object and RC separately:
```cpp
std::shared_ptr<int> p1 (new int(100));
```

- Allocating object and RC all at once:
```cpp
auto p1 = std::make_shared<int>(100);
```

## Weak Pointers — `std::weak_ptr`

`std::weak_ptr` is a special-case smart pointer used in conjunction with `std::shared_ptr` and doesn't participate in reference counting.
- A `std::weak_ptr` knows if the object has gone when a resource's reference count becomes 0 (when all of other shared pointers are disposed/distroyed) its `std::weak_ptr expire
- Use `std::weak_ptr` to observe an object, where you don't absolutely require it to remain alive
- Also `std::weak_ptr` can be used to prevent circular references between `std::shared_ptr`s. (To break the link between shared pointers. E.g. Two classes like Car and Person have their own shared pointers that points to each other. One of them is distroyed, the RC decrements to 1. However the other one is never destroyed — circular reference problem. To solve it, use shared_pointer in Person and use weak_pointer in Car, when the person is destroyed, the Car object will automaitcally be destroyed.)


### Important Methods:

- **operator=()** : Constructor and the equal operator makes the same thing. Once, the shared_pointer should be created. Then a weak_pointer can be assigned to this shared_pointer. [Ref](https://en.cppreference.com/w/cpp/memory/weak_ptr.html)
- **reset()** : Stopping to observe a shared_ptr. Turns to Nullptr. [Ref](https://en.cppreference.com/w/cpp/memory/weak_ptr/reset.html)
- **lock()** : Converts a weak_ptr to a shared_ptr. (E.g. sp2 = wp.lock(); ) [Ref](https://en.cppreference.com/w/cpp/memory/weak_ptr/lock.html)


### Query Methods:

- **use_count()** : What is the use count of the object that referenced by the observed shared_ptr
- **expired()** : Is there reference count (RC) zero?

⚠ There are no object-access methods (like operator*(), operator->()) for weak_ptr's. **weak_ptr** does not provide any direct access to the underlying object. You can't directly dereference the weak pointer because the object that it refers to could have been deleted. The only way to use weak_ptr is using the `lock` function and get back a properly shared pointer to lock the object against future deletion.. 

```cpp
auto sp1 = std::make_shared<int> (100);

// Create a weak_ptr to observe object maintained by a shared_ptr.
std::weak_ptr<int> wp = sp1; // std::weak_ptr<int> wp(sp1);

// Some code, might cause the observed object to be released
// In this case, wp will know the object has been released

...

// When ready to access observed object, upgrade to a shared_ptr
// The shared_ptr will be empty if the object has actually expired
std::shared_ptr<int> sp2 = wp.lock();

if (sp2)
{
    // Comes here if only the observed object is not released
    // Here, sp2 points to the object originally pointed by wp
    // We own the object now, so it won't disappear hereafter
    ...
}
```

## Unique Pointers — `std::unique_ptr`

`std::unique_ptr` retains sole ownership of an object through a pointer. 
- Destroys object when `std::unique_ptr` goes out of scope
- No two `std::unique_ptr`s can manage the same object
- Can be moved to a new owner, but not copied or shared
- Can't be used in STL algorithm that requires a copy
- There are 2 version of `std::unique_ptr`: 
    - One takes a pointer, the other takes an array
    - Slightly different semantics for each


### Key Methods
- **operator=()** : No copy behaviour, only move semantics (up2 = std::move(up1)), you can just move ownership
- **release()** : Releases the ownership of the managed object, if any. [Ref](https://en.cppreference.com/w/cpp/memory/unique_ptr/release.html)

```cpp
MyClass * mc = up1.release();
...
delete mc; // The caller is responsible for cleaning up the object 
```

- **reset()** : Replaces the managed object with another on (up1.reset(new MyClass))
- **swap()** : Swapping/exchanging two unique pointers 
- **get()**: Returns the object that's referenced 
- **get_deleter()** : Returns the deleter object which would be used for destruction of the managed object. [Ref](https://en.cppreference.com/w/cpp/memory/unique_ptr/get_deleter.html)
- **operator bool()** : Is the pointer null?

### Single-Item version:
- **operator\*** : Reference to the actual object
- **operator->** : Access to the actual object (functions, variables)
- **operator[]** : Points to an array of items (up1[i]). You can index into the item you want to

```cpp
struct Data {
    int state;
    Data(int state) : state(state) {}
    ~Data () { std::cout << "Goodbye" << state << std::endl; }
};

std::unique_ptr<Data> p1(new Data(100)); // auto p1 = std::make_unique<Data>(100);
std::cout << p1->state << std::endl; // Use
std::unique_ptr<Data> p2 = std::move(p1); // Move, p1 is released, nullptr
```

# Using `std::enable_shared_from_this` class

`std::enable_shared_from_this` defines a method name `shared_from_this()` and returns a `std::shared_ptr` to the current object.

**Purpose**:

Imagine you have an object that is currently managed by a `std::shared_ptr`, you can ask the object to generate an additional `std::shared_ptr` that shares the RC

```cpp
class Widge : public std::enable_shared_from_this<Widget> {
    public:
        void some_method()
        {
            some_api(shared_from_this());
        }
};

void some_api(const std::shared_ptr<Widget>& sp);
```

`std::enable_shared_from_this` has a `std::weak_ptr` field inside it:

```cpp
template <typename T>
class std::enable_shared_from_this {
    private:
        weak_ptr<T> self;

    public:
        std::shared_ptr<T> shared_from_this()
        {
            return self.lock(); // Creates shared_ptr from weak_ptr
        }
}
```

If you create a `std::shared_ptr<Widget>`: The shared_ptr constructor has some code that interanlly hooks self to the shared_ptr.

`std::shared_ptr<Widget> sp1(new Widget);`

If your object inherits from `std::enable_shared_from_this` then it will have a weak_ptr called self.