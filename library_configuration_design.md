# 1. Configuration via `constexpr` options

This is the cleanest **C++ equivalent** of your C approach — but improved using strong typing and scoped enums.

### `ring_buffer_config.hpp`

```cpp
#pragma once

namespace rb::config {

inline constexpr bool use_static_allocation = true;
inline constexpr bool use_memory_functions  = false;
inline constexpr bool use_overwrite_mode    = true;
inline constexpr bool isr_safe_design       = true;

} // namespace rb::config
```

### Pros

* Type-safe, no macros.
* Works in embedded/no-RTTI/no-heap environments.
* Zero runtime cost (`constexpr`).
* No ABI impact for header-only libs.
* Users override by editing file or including their own before your header.

### Cons

* Requires rebuilding dependent code when config changes (but that’s usually desirable).

---

# 2. Template Policy / Policy Classes (Most professional for reusable C++ libraries)

Modern C++ libraries (e.g., EASTL, Folly, Boost) use **policy-based design**.

### `ring_buffer.hpp` (example)

```cpp
template<
    typename T,
    typename AllocPolicy,
    typename OverwritePolicy,
    typename SyncPolicy
>
class RingBuffer { ... };
```

### Policies:

```cpp
struct StaticAlloc {
    static constexpr bool use_static = true;
};

struct OverwriteEnabled {
    static constexpr bool overwrite = true;
};

struct IsrSafe {
    static void atomic_store(size_t& target, size_t value) {
        target = value; // or platform-specific
    }
};
```

### Pros

* **Most flexible and professional C++ approach.**
* Allows mixing different configurations **per instance** rather than global.
* No macros, no global config header.
* Enables compile-time specialization and inlining.

### Cons

* Slightly higher complexity.
* API becomes more template-heavy.

---

# 3. Traits-Based Configuration (Good for behaviour customisation)

Similar to policy classes but more implicit.

### `ring_buffer_traits.hpp`

```cpp
template<typename T>
struct RingBufferTraits {
    static constexpr bool use_static_allocation = true;
    static constexpr bool overwrite_on_full = false;
};
```

Users specialize for their types:

```cpp
template<>
struct RingBufferTraits<MyType> {
    static constexpr bool overwrite_on_full = true;
};
```

### Pros

* Useful when config depends on type.
* Cleaner instantiation than full-blown policies.

### Cons

* Only works when traits logically attach to `T`.

---

# 4. Acceptable but Less Ideal Options

## Preprocessor Macros (old C-style)

Still acceptable **for embedded** but not preferred for C++ libraries.

### Pros

* Simple.
* No runtime overhead.

### Cons

* Globally scoped, not type-safe.
* Debugging and code navigation worse.
* No per-instance configurability.

---

## Runtime Configuration Objects

Not recommended for low-level data structures like ring buffers, especially in embedded.

# Example Final Design

### `ring_buffer_config.hpp`

```cpp
#pragma once
#include <cstddef>

namespace rb::config {
    inline constexpr bool static_allocation = true;
    inline constexpr bool overwrite_mode = true;
    inline constexpr bool isr_safe = true;
    inline constexpr bool use_memcpy = false;
}
```

### `ring_buffer.hpp`

```cpp
#pragma once
#include "ring_buffer_config.hpp"

template<typename T, std::size_t N>
class RingBuffer {
public:
    RingBuffer() {
        if constexpr (!rb::config::static_allocation) {
            buffer = new T[N];
        }
    }

    bool push(const T& item) {
        // behavior switches at compile time
        if constexpr (rb::config::overwrite_mode) {
            // overwrite logic
        } else {
            // reject when full
        }
    }

private:
    T storage[N];
    T* buffer = storage;
};
```

---
