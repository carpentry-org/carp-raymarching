# Examples for carp-raymarching

This file contains usage examples for all sub-modules in the library.

## Raymarch

## Basic Traversal

This example shows how to perform a basic raymarch against a sphere.

```clojure
(load "raymarch.carp")

(use Raymarch)
(use Vector3)

(defn scene [p]
  (let [sphere-dist (- (Vector3.mag p) 1.0)]
    sphere-dist))

(defn main []
  (let [ro (Vector3.init 0.0 0.0 5.0)
        rd (Vector3.init 0.0 0.0 -1.0)
        ray (Ray.make ro rd)
        config (Raymarch.default-config)
        hit (Raymarch.march &scene &ray &config)]
    (if @(RaymarchHit.hit &hit)
      (do
        (println* "Hit surface at distance: " @(RaymarchHit.t &hit))
        (println* "Normal: " (str (RaymarchHit.normal &hit)))
        (println* "Steps taken: " @(RaymarchHit.steps &hit)))
      (println* "Traversal missed or exceeded max distance."))))
```


---

## Raymarch-Renderer
This directory contains practical examples of how to use the Carp modular raymarching stack.

### 1. Simple Render (`red_sphere.carp`)
Shows the "Hello World" of raymarching: a shaded, reflective sphere on a floor.
- **Key Concepts**: Basic pipeline setup, recursive reflections, simple material definitions.

### 2. CSG Gallery (`csg_gallery.carp`)
Demonstrates complex geometry using Boolean operations from `carp-sdf`.
- **Key Concepts**: `Sdf.subtract`, `Sdf.union`, mapping multiple objects to different materials.

### 3. Multi-Light Scene (`multi_light.carp`)
Shows how to accumulate energy from multiple light sources for more dynamic lighting.
- **Key Concepts**: Accumulating results from `Renderer.shade-hit`, using colored lights.

### 4. Neon Infinite Grid (`neon_infinite.carp`)
A high-fidelity showcase of the entire stack's integration.
- **Key Concepts**: Infinite domain repetition (`Sdf.repeat`), material emission (glow), atmospheric fog, and recursive mirrored floor reflections.

## Running the Examples
You can run any example using the Carp REPL:
```bash
carp -x examples/red_sphere.carp
```
Each example generates a `.ppm` image file in the `examples/` directory.


---

## Render

## Defining a Material

```carp
(use Material)
(use Vector3)

(let [iron (Material.make (Vector3.init 0.56 0.57 0.58) 0.1 1.0)]
  (do-something-with iron))
```

## Using Shading.blinn-phong

```carp
(use Shading)
(use Vector3)

(let [n (Vector3.init 0.0 1.0 0.0)
      l (Vector3.normalize (Vector3.init 1.0 1.0 1.0))
      v (Vector3.init 0.0 0.0 1.0)
      albedo (Vector3.init 1.0 0.0 0.0)
      roughness 0.2
      metallic 0.0]
  (Shading.blinn-phong n l v albedo roughness metallic))
```


---

## Scene

How to define a list of `SceneObject` and query them using `Scene.min-hit`:

```carp
(def objects
  [(SceneObject.init (Vector3.init 0.0f 0.0f 0.0f) (Material.init ...))
   (SceneObject.init (Vector3.init 2.0f 0.0f 0.0f) (Material.init ...))])

(def hit (Scene.min-hit &objects ray))
```


---

## Sdf

## CPU Usage

Calculating the signed distance from a point to a sphere using pure Carp logic:

```clojure
(use Sdf)
(use Vec3)

(defn main []
  (let [p (Vec3.new 0.0 5.0 0.0)
        d (Sdf.sphere &p 1.0)]
    (println* "Distance to sphere: " (str d))))
```

## GPU Shader Usage

Extracting the WebGPU Shading Language (WGSL) snippets for compile-time GPU shader generation:

```clojure
(use Sdf)

(def my-shader 
  (str* Sdf.Wgsl.sphere 
        Sdf.Wgsl.smooth-union
        "// Your raymarching code here..."))
```


---

## Svo

## 1. Basic Set & Sample

```carp
(load "carp-svo/svo.carp")
(use Svo)

(defn main []
  (let [origin (Vector3.init -5.0 -5.0 -5.0)
        size 10.0
        max-depth 3
        tree (Svo.new origin size max-depth 0.0)]
    (do
      ;; Set temperature of a particle in space
      (Svo.insert! &tree (Vector3.init 0.0 0.0 0.0) 25.5)
      
      ;; Sample at that location
      (let [temp (Svo.sample &tree (Vector3.init 0.0 0.0 0.0))]
        (println* "Temperature at center: " temp)))))
```

## 2. Sparse Space & Early Sample Termination

```carp
(load "carp-svo/svo.carp")
(use Svo)

(defn main []
  (let [origin (Vector3.init 0.0 0.0 0.0)
        size 100.0
        max-depth 6
        tree (Svo.new origin size max-depth -9.9)]
    (do
      ;; Even though max-depth is 6, we have not inserted anything yet.
      ;; The root is a leaf node, so sample terminates instantly (O(1) complexity).
      (let [v1 (Svo.sample &tree (Vector3.init 50.0 50.0 50.0))
            v2 (Svo.sample &tree (Vector3.init 2.0 80.0 15.0))]
        (do
          (println* "V1: " v1) ;; -9.9
          (println* "V2: " v2)))))) ;; -9.9
```

## 3. Clearing and Re-using the Octree Pool

```carp
(load "carp-svo/svo.carp")
(use Svo)

(defn main []
  (let [origin (Vector3.init 0.0 0.0 0.0)
        size 10.0
        max-depth 4
        tree (Svo.new origin size max-depth 0.0)]
    (do
      ;; Splat values
      (Svo.insert! &tree (Vector3.init 1.0 1.0 1.0) 42.0)
      (println* "Value before clear: " (Svo.sample &tree (Vector3.init 1.0 1.0 1.0))) ;; 42.0
      
      ;; Reset the octree pool to default state
      (Svo.clear! &tree)
      (println* "Value after clear: " (Svo.sample &tree (Vector3.init 1.0 1.0 1.0)))))) ;; 0.0
```

