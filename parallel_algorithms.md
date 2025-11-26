# Parallel Algorithms

Algorithms support an execution policy for parallelization via classes defined in the namespace `std::execution`:

- `sequenced_policy`
- `parallel_policy`
- `parallel_unsequenced_policy`
- `unsequenced_policy`

Per policy class, there's a predefined instance:

- `seq` for sequenced_policy
- `par` for parallel_policy
- `par_unseq` for parallel_unsequenced_policy
- `unseq` for unsequenced_policy

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

std::vector<int> v;
...

std::for_each(std::execution::par, v.begin(), v.end(),
            [] (int i) { std::cout << i << std:: endl; })
```
