# Multithreading

## Overview

To create a new thread and await its completion:

```cpp
#include <thread>
#include <iostream>

void myFunc1 () {
    std:: cout << "Doing some task" << std:: endl;
}

int main() {
    std::thread task1 (myFunc1);
    ...
    ...
    task1.join(); // If the main thread is going to potentially finish before the other thread (task1), (means program come here) wait to task1's complete (await). The main thread will then wait for that other thread to finish.
    ... // Program does not come here until the task1 is completed
}
```

## Using Lambdas

The ``std::thread` constructor allows you to use a lambda to specify the code to run in the thread:

```cpp
#include <thread>
#include <iostream>

int main() {
    std::thread task1 ([] {
        std:: cout << "Doing some task" << std:: endl;
    });
    ...
    task1.join();
}
```

Beware of object lifetimes — pass-by-value is safest. If your thread is going to run for any non-trivial amount of time, **don't** capture variables by **reference**, capture by **value** instead (Use `[=]` for lambda functions — will have copied values into the lambda object, they will live long enough for the lifetime of the lambda function).

## Working with the Current Thread

The `std::this_thread` namspace has functions that enable you to work with the current thread.

```cpp
std::thread::id id = std::this_thread::get_id(); // 0, 1, 2,... (generally for logging/diagnostic purposes)
```

```cpp
std::this_thread::yield(); // It tells the current thread to stop utilizing the CPU.
// Its useful if you have lots of threads, but not so many CPUs. Example scenario, you have 1 CPU and have many threads. It means only one task/therad can executes at a time. If the currently executed thread is quite greedy (is like in a tight loop with algorithmic calculation) then this thread here could hog the CPU intensively. Then the no other threads get a chance to execute. You can call `yield` function periodically in this thread to pause itself and allow other threads to be executed.
```

```cpp
std::this_thread::sleep_for(aDuration); // Schedule cthe current thread (ms, s, m, h)
// The thread is kind of put to the back of the queue and will wait. It will be re-awoken after the specified duration time.
```

```cpp
std::this_thread::sleep_until(aTime); // A strict time to be executed
```

## Calling Function Asynchronously

Use `std::async()` to call a function asynchronously. Define in the `#include <future>` header file. To use `std::async()`:
- Specify the function you want to invoke with arguments
- Returns a `std::future` that will hold the function result

```cpp
std::future<T> std::async(function, argument_list)
```
- `T` specifies the result type of the asynchronous call. You can specify it `<void>`.

### Getting the Result from a Future

To get the result from future: Call `get()`. Blocks until a return is available, then grabs it — It's dangerous to be used because you don't know how long you're going to have to wait.
- Alternatively (better approach) call `wait_for()` — specify a timeout (or 0 for peek). It will always return within the specified timeout. Its like a pulling mechanism. You pull the thread to say "have you nearly finished yet?"

⚠ If the asynchronous function threw an exception:
- If you are waiting on the thread, the wait will terminate
- You'll get the exception when you call `get()` 

```cpp
double myFunc(int x, int y);
...

// Run lambda asynchronously 
std::future<double> f = std::async(myFunc, 10, 20);

// Do other work here
...

while (f.wait_for(std::chrono::seconds(0)) != std::future_status::ready)
{
    // Isn't ready, do some more work here
    ...
}

// Now get the result (or exception) from the future
double val = f.get();
```

`std::future<T>`:
- Results can only be accessed once
- Suitable for most use cases
- Moveable, not copyable
- Exactly one future has right to access result

`std::shared_future<T>`:
- Creatable from std::future
- Result can be accessed multiple times
- Appropriate when many threads access a single result
- Moveable and copyable