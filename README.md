# carp-raymarching

A consolidated Raymarching Library suite for the Carp programming language.

## Modules Included
- **[Raymarch](#raymarch)**: See module documentation below.
- **[Raymarch-Renderer](#raymarch-renderer)**: See module documentation below.
- **[Render](#render)**: See module documentation below.
- **[Scene](#scene)**: See module documentation below.
- **[Sdf](#sdf)**: See module documentation below.
- **[Svo](#svo)**: See module documentation below.

## Examples

See [examples.md](examples.md) for module usage examples.

---

## Raymarch

A pure geometric traversal kernel for the Carp programming language. This library provides the core math and logic for raymarching signed distance fields (SDFs).

## Features

- **Ray Safety**: Enforced normalization of ray directions and automatic start-bias handling to prevent self-intersection.
- **Numerical Stability**: Advanced stagnation protection using split epsilon values for hit detection and minimum step increments.
- **Cross-platform Parity**: Canonical WGSL implementations included for seamless transitions between CPU (Carp) and GPU (WebGPU) rendering.
- **Modular Design**: Zero-dependency kernel that integrates with any SDF provider.

## Installation

```clojure
(load "https://github.com/sqrew/carp-raymarch@master")
```

## Usage

For detailed usage, see [examples.md](examples.md).

```clojure
(use Raymarch)
(use Vector3)

;; Define a simple SDF
(defn sphere [p]
  (- (Vector3.mag p) 1.0))

;; March a ray through the field
(let [ray (Ray.make (Vector3.init 0.0 0.0 5.0) (Vector3.init 0.0 0.0 -1.0))
      config (Raymarch.default-config)
      hit (Raymarch.march &sphere &ray &config)]
  (if @(RaymarchHit.hit &hit)
    (println* "Hit at: " (str (RaymarchHit.p &hit)))
    (println* "Miss!")))
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


---

## Raymarch-Renderer

The integration layer and orchestration pipeline for the Carp modular raymarching stack.

## Features

- Unified RenderConfig
- Shading pipeline (traversal + identity + visibility + shading)
- Example CPU PPM renderer

## Examples

See [examples.md](examples.md) for usage examples.


---

## Render

Visual utilities and shading equations for the Carp rendering stack.

## Features

- Physical visibility (Shadows/AO)
- Shading equations (Lambert, Blinn-Phong)
- Material definitions

## Examples

See [examples.md](examples.md) for usage examples.


---

## Scene

World composition and identity query layer for the Carp rendering stack.

## Features

- SceneObject ownership
- Functional world queries (SDF and Identity)
- Material sampling

## Examples

See [examples.md](examples.md) for usage examples.


---

## Sdf
A high-performance Signed Distance Field (SDF) module for the Carp programming language.

## Features
- **CPU + GPU Logic**: Provides pure Carp functions for distance queries and WGSL snippets for GPU shaders.
- **Primitives**: Sphere, Box, Torus.
- **Operations**: Union, Subtraction, Intersection, and the legendary **Smooth Union** (Gooey Merge).
- **Zero Dependencies**: Includes its own minimal math types for maximum portability.

## Installation
```clojure
(load "https://github.com/sqrew/carp-sdf@master")
```

## Examples

See [examples.md](examples.md) for usage examples.

## License
MIT


---

## Svo

A modular, high-performance 3D Sparse Voxel Octree (SVO) implementation for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library provides a spatial octree acceleration structure designed for memory-efficient 3D scalar/vector storage, hierarchical queries, and GPU-friendly buffer layouts.

## Features
- **Flat Node Pooling**: All nodes are allocated in a contiguous array, avoiding pointer fragmentation, minimizing memory overhead, and enabling direct GPU buffer streaming.
- **Hierarchical Traversals**: Implements $O(\log N)$ descent for insertion and sampling, enabling fast spatial queries.
- **Early Termination**: Sampling automatically returns parent values for unsubdivided uniform spaces, drastically reducing tree depth traversals.
- **Dynamic Subdivision**: Allocates contiguous 8-node blocks on-demand during point insertions.


## Examples

See [examples.md](examples.md) for usage examples.
## Architecture
Each parent node indexes its children as a contiguous block of 8 sibling elements in the pool:
```carp
(deftype SvoNode [
  child-ptr Int       ;; index of the start of the 8 children in the node pool (-1 if leaf)
  value Double        ;; scalar/field value stored at this node
  is-leaf Bool
])
```
This enables extremely fast relative indexing (`child-ptr + octant-index`) during tree descents.

## License
MIT


## License

MIT
