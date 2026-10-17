WebGPU Forward+ and Clustered Deferred Shading
======================

**University of Pennsylvania, CIS 5650: GPU Programming and Architecture, Project 4**

* Shanshan
* Tested on: **Google Chrome 155.0.8059.80 (Official Build) (64-bit)** on
 Windows 11, AMD Ryzen 9 270 @ ~4.0GHz, 32GB RAM, NVIDIA GeForce RTX 5070 Laptop GPU (personal laptop)


### Live Demo

[![](img/thumb.png)](https://popi-x.github.io/Project4-WebGPU-Forward-Plus-and-Clustered-Deferred)

### Demo Video/GIF

<!-- TODO: record 30+ seconds showing Naive / Forward+ / Clustered Deferred and changing the light count -->
![](img/demo.gif)

---

## Overview

This project renders the Sponza atrium lit by up to 5000 moving point lights in WebGPU, using three renderers that can be switched in the GUI:

| Renderer | Passes per frame | Lights evaluated per fragment |
|---|---|---|
| **Naive** | 1 render pass | all lights |
| **Forward+** | clustering compute + 1 render pass | only the lights in the fragment's cluster |
| **Clustered Deferred** | clustering compute + G-buffer pass + fullscreen lighting pass | only the lights in the pixel's cluster, once per pixel |

<!-- TODO: screenshot of the scene -->
![](img/screenshot.png)

---

## Features

### 1. Light Clustering (shared by Forward+ and Clustered Deferred)

The view frustum is divided into a **16 × 9 × 24** grid of clusters (3456 clusters). A compute shader (`clustering.cs.wgsl`, one thread per cluster) builds each cluster's light list every frame.

- **XY**: the screen is split evenly into 16 × 9 tiles in NDC.
- **Z**: depth is sliced **logarithmically** between near and far:
  `zNear_k = near · (far / near)^(k / Z)`. This keeps clusters roughly cube-shaped, since screen tiles get larger with distance.
- **Cluster bounds**: the 4 NDC corners of each tile are un-projected with `invProjMat` and scaled to the slice's near/far depth, giving 8 view-space points. Their min/max is the cluster's **AABB**.
- **Light assignment**: each light's position is transformed to view space and tested against the AABB with a **sphere–AABB test** (clamp the center into the box, compare squared distance with r²). Up to **256** light indices are stored per cluster.

<!-- TODO (optional): debug view, e.g. number of lights per cluster as a heatmap -->

### 2. Forward+

The scene is drawn once. In the fragment shader, each fragment:
1. finds its cluster: x, y from `@builtin(position)` (pixel coordinates, y flipped to match NDC), z from the view-space depth with the inverse of the log slicing formula;
2. loops only over that cluster's lights.

### 3. Clustered Deferred

1. **G-buffer pass**: draws the scene and writes, per pixel, the closest surface's data into 3 render targets. No lighting is done here.

   | Target | Content | Format |
   |---|---|---|
   | `@location(0)` | world position | `rgba16float` |
   | `@location(1)` | normal | `rgba16float` |
   | `@location(2)` | albedo | `rgba8unorm` |

2. **Fullscreen pass**: one screen-covering triangle (generated from `@builtin(vertex_index)`, no vertex buffer). Each pixel reads the G-buffer with `textureLoad`, finds its cluster, and shades with the cluster's lights. Every pixel is shaded exactly once.

<!-- TODO (optional): images of the 3 G-buffer targets -->

---

## Performance Analysis

**Setup:** (TODO: resolution, e.g. 1920 × 1080), cluster grid 16 × 9 × 24, max 256 lights per cluster, light radius 2. Frame times are in **milliseconds** (lower is better), read from the stats panel and averaged over (TODO) seconds with the camera at the default position.

> Note: Chrome limits the frame rate to the display refresh rate, so any frame time at ~(TODO: 16.7) ms may be capped.

### Frame time vs. number of lights

| Lights | Naive (ms) | Forward+ (ms) | Clustered Deferred (ms) |
|---:|---:|---:|---:|
| 500 | TODO | TODO | TODO |
| 1000 | TODO | TODO | TODO |
| 2000 | TODO | TODO | TODO |
| 3000 | TODO | TODO | TODO |
| 5000 | TODO | TODO | TODO |

<!-- TODO: line chart of the table above -->
![](img/perf_lights.png)

### Is one of them faster?

TODO: Which renderer is fastest at low / high light counts, and by how much.

### Why?

TODO. Points to cover:
- **Naive vs. clustered**: Naive evaluates every light for every fragment (cost ≈ fragments × lights). Clustering reduces this to the few lights near each fragment, so the cost grows much more slowly with the light count.
- **Forward+ vs. Clustered Deferred**: Forward+ shades every fragment that passes the depth test at the time it's drawn, including surfaces that are later covered (overdraw). The alpha-tested `discard` in the fragment shader also prevents early-z. Clustered Deferred only writes cheap G-buffer data in the geometry pass and shades each pixel once.
- **Cost of Deferred**: 3 extra screen-sized textures written and read every frame (memory and bandwidth), plus an extra pass.

### Better at certain workloads? Benefits and tradeoffs

| | Forward+ | Clustered Deferred |
|---|---|---|
| Many lights, lots of overdraw | TODO | TODO |
| Few lights / simple scenes | TODO | TODO |
| Memory / bandwidth | TODO | TODO |
| Transparency | TODO (works naturally) | TODO (G-buffer stores one surface per pixel) |
| MSAA | TODO | TODO |
| Material flexibility | TODO | TODO (limited to what the G-buffer stores) |

### (Optional) Cluster grid size

| Grid | Forward+ (ms) | Clustered Deferred (ms) |
|---|---:|---:|
| 8 × 4 × 12 | TODO | TODO |
| 16 × 9 × 24 | TODO | TODO |
| 32 × 18 × 48 | TODO | TODO |

TODO: Too few clusters → each holds many lights; too many → clustering compute and memory cost increase.

### Optimizations

<!-- TODO: for each optimization, before / after in ms -->

| Change | Before (ms) | After (ms) |
|---|---:|---:|
| TODO | TODO | TODO |

### Possible further optimizations

- TODO (e.g. pack the G-buffer into fewer / smaller textures, reconstruct position from depth, octahedral normal encoding, compute-shader lighting pass)

---

### Credits

- [Vite](https://vitejs.dev/)
- [loaders.gl](https://loaders.gl/)
- [dat.GUI](https://github.com/dataarts/dat.gui)
- [stats.js](https://github.com/mrdoob/stats.js)
- [wgpu-matrix](https://github.com/greggman/wgpu-matrix)
