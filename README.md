# vg-number

`vg-number` is a high-performance Python library designed for 2D numerical operations, advanced noise generation, and efficient pathfinding. It leverages NumPy for core operations to ensure maximum performance while providing a clean, intuitive API.

## Installation

You can install the library directly from the source directory.

```bash
# If using standard pip
pip install .

# If using uv (recommended for faster dependency resolution)
uv pip install .
```

## Core Components

The library is divided into four main modules:
1. `matrix`: Efficient 2D matrix data structures and convolution filters.
2. `noise`: Advanced 2D noise generation algorithms.
3. `pathfinding`: Flexible and efficient A* algorithm implementations.
4. `utils`: Core mathematical utilities.

---

### 1. Matrix (`vg_number.matrix`)

The `matrix` module provides `Matrix2D`, a highly optimized 2D matrix class built on top of NumPy. It supports tracking "unassigned" values efficiently using boolean masks, allowing for sparse-like operations without the overhead of standard dictionaries or lists.

#### Features:
- NumPy-backed high performance.
- Built-in support for unassigned/missing values.
- Standard arithmetic operations (`+`, `-`, `*`, `/`, `@`).
- Convolution and filtering operations.
- Direct interoperability with the `noise` module.

#### Basic Usage:

```python
from vg_number.matrix import Matrix2D, MatrixFilters, BlurType

# Create a 512x512 matrix filled with 0.0
matrix = Matrix2D((512, 512), default_value=0.0)

# Set and get values
matrix.set_value_at(10, 10, 5.5)
value = matrix.get_value_at(10, 10)  # Returns 5.5
matrix.set_value_at(10, 10, None)    # Mark as unassigned

# Arithmetic operations
matrix += 2.0
matrix_b = Matrix2D((512, 512), default_value=1.5)
result_matrix = matrix + matrix_b

# Convolution Filtering (e.g., Gaussian Blur)
gaussian_kernel = MatrixFilters.create_blur_kernel(BlurType.GAUSSIAN, radius=2)
blurred_matrix = result_matrix.apply_kernel(gaussian_kernel, normalize=True)
```

---

### 2. Noise Generation (`vg_number.noise`)

The `noise` module provides robust tools to generate various types of 2D noise, essential for procedural generation, textures, and terrain maps.

#### Features:
- Diverse algorithms: Perlin, Simplex, Cellular, Value noise, etc.
- `FastNoise2D` wrapper for rapid, optimized generation.
- Domain Warping to create organic, distorted noise patterns.
- Vectorized integration with `Matrix2D` for blazing-fast matrix population.

#### Basic Usage:

```python
from vg_number.noise.generators import FastNoise2D, DomainWarp2D
from vg_number.matrix import Matrix2D

# Initialize a noise generator
noise_gen = FastNoise2D(seed=42)

# Retrieve a single value
val = noise_gen.get_value_at((10.0, 20.0))

# Best Practice: Fill a Matrix2D efficiently using vectorized operations
matrix = Matrix2D.create_from_noise(noise_gen, size_x=256, size_y=256)

# Applying Domain Warp
warp = DomainWarp2D(seed=1337)
warped_x, warped_y = warp.warp(10.0, 20.0)
warped_val = noise_gen.get_value_at((warped_x, warped_y))
```

---

### 3. Pathfinding (`vg_number.pathfinding`)

A flexible implementation of the A* (A-Star) algorithm. It works with arbitrary graphs and provides specialized, optimized functions for generic 2D grids.

#### Features:
- Generic A* for any graph-like structure (`astar`).
- Optimized `astar_grid_2d` for quick setup on 2D maps.
- Pluggable heuristics (Manhattan and Zero included out of the box).
- Extensible `Heuristic` base class for custom estimators.
- Configurable options: diagonal movement, dynamic costs, iteration limits, and debugging callbacks.
- Weighted A* support for faster, sub-optimal searches.

#### Basic Usage:

```python
from vg_number.pathfinding import astar_grid_2d, Manhattan

# Define a walkability function
def is_walkable(pos):
    x, y = pos
    # Example: restrict movement to a 10x10 grid and place an obstacle at (5, 5)
    if pos == (5, 5):
        return False
    return 0 <= x < 10 and 0 <= y < 10

# Execute A* search
result = astar_grid_2d(
    start=(0, 0),
    goal=(9, 9),
    is_walkable_fn=is_walkable,
    heuristic=Manhattan(),
    allow_diagonal=True,
    diagonal_cost=1.414  # sqrt(2)
)

if result.found:
    print(f"Path found in {result.path_length} steps!")
    for step in result.path:
        print(f" -> {step}")
else:
    print("No path could be found.")
```

#### Generic Graph A*

For graphs that are not regular grids, use the low-level `astar` function with custom neighbor and cost functions:

```python
from vg_number.pathfinding import astar, Manhattan

result = astar(
    start=start_node,
    goal=goal_node,
    neighbors_fn=get_neighbors,
    cost_fn=get_cost,
    heuristic=Manhattan()
)
```

#### Available Heuristics

| Heuristic | Status | Best for |
|-----------|--------|----------|
| `Manhattan` | Implemented | 4-directional grid movement (L1 norm) |
| `Zero` | Implemented | Dijkstra-like search (always returns 0) |
| `Euclidean` | Placeholder | Free movement in any direction (L2 norm) |
| `Chebyshev` | Placeholder | 8-directional movement with equal diagonal cost |
| `Octile` | Placeholder | 8-directional movement with costly diagonals |

All heuristic classes accept a `weight` parameter. Weights greater than 1.0 turn A* into weighted A*, trading optimality for speed.

#### Advanced Examples

**Obstacles**

```python
obstacles = {(1, 1), (1, 2), (1, 3), (2, 2)}

def is_walkable(pos):
    x, y = pos
    return 0 <= x < 10 and 0 <= y < 10 and pos not in obstacles

result = astar_grid_2d((0, 0), (4, 4), is_walkable, Manhattan())
```

**Iteration Limit**

```python
result = astar_grid_2d(
    start=(0, 0),
    goal=(100, 100),
    is_walkable_fn=is_walkable,
    heuristic=Manhattan(),
    max_iterations=1000
)
```

**Search Callbacks**

```python
from vg_number.pathfinding import astar_with_callbacks, Manhattan

result = astar_with_callbacks(
    start=(0, 0),
    goal=(5, 5),
    neighbors_fn=get_neighbors,
    cost_fn=get_cost,
    heuristic=Manhattan(),
    on_node_explored=on_explored,
    on_node_added=on_added
)
```

**Weighted A***

```python
weighted = Manhattan(weight=2.0)
result = astar_grid_2d((0, 0), (50, 50), is_walkable, weighted)
```

#### PathResult

Every search returns a `PathResult` object with the following attributes:

- `path`: List of nodes from start to goal, or `None` if no path was found.
- `cost`: Total cost of the path, or `None` if no path was found.
- `path_length`: Number of nodes in the path (`0` if not found).
- `nodes_explored`: Number of nodes expanded during the search.
- `found`: `True` if a path was found.

`PathResult` also implements `__bool__`, so it can be used directly in conditions:

```python
if result:
    print(result.path)
```

#### Complexity

- **Time:** O(b^d), where `b` is the branching factor and `d` is the search depth.
- **Space:** O(b^d) for the open and closed sets.

---

### 4. Utilities (`vg_number.utils`)

Contains general mathematical and geometric utility functions used internally by the library, which can also be utilized independently for operations like interpolation and clamping.

```python
from vg_number.utils.math_utils import clamp, lerp

# Linear interpolation
value = lerp(0.0, 10.0, 0.5)  # Returns 5.0
```

## Development

To set up the project for local development:

1. Clone the repository.
2. Create and activate a virtual environment.
3. Install the package in editable mode:
   ```bash
   pip install -e .
   ```
