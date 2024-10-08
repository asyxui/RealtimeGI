## NVIDIA
Tech demo: https://www.nvidia.com/en-us/on-demand/session/gtcspring22-s43047/

Note: https://developer.nvidia.com/rtx/path-tracing

NVIDIA use the open source shading language [Slang](https://github.com/shader-slang/slang) for their development.

### Path Tracing Pipeline
V-Buffer -> RTXDI (Direct illumination) + Path tracer (all other light paths) + RTXGI (Optional Irradiance cache) -> NRD Denoising + DLSS + Post (bloom, depth of field etc.)

### Light sampling for direct illumination
Direction illumination for diffuse/glossy reflection
using spation-temporal resampling based on [this paper](https://research.nvidia.com/sites/default/files/pubs/2020-07_Spatiotemporal-reservoir-resampling/ReSTIR.pdf).

### Denoising
NVIDIA Real-time Denoisers (NRD) library of spation-temporal denoising kernels.

NVIDIA Deep Learning Super Sampling (DLSS) applied at the end.

The Path Tracer outputs additional metadata to decompose into multiple components. The components coulde be: Diffuse, Specular, Delta reflection (glass, mirror), Delta transmission (glass), Residual.

The Denoiser needs noise-free guide and demodulation buffers. Easy for diffuse/specular paths as you can use the material properties at primary hits. For delta reflection/transmission it's harder because we don't have the equivalent of a v-buffer for those. Trace additional simple paths for recording material properties but not shading.

### Performance optimizations
Shader Divergence is a big problem for path tracing. Memory coherency is quickly lost because of viariable path length, rays scattering all over, and a large number of material shaders. These are some ways to address this problem.

#### Material complexity
Material Distillation to reduce the large amount of shaders to a smaller set based on MDL SDK. 

A standard PBR Material is used with 6 BSDF lobes (difuse/glossy/delta reflection & diffuse/glossy/delta transmission)
#### Cache optimization
Path tracing is inherently random access. To maximize L1 and L2 cache hit ratio:

1. Compress the data (fp16, packed normals, use BCx format)
2. Cacheline alignment (128B)
3. Avoid indirections (minimize # of cachelines per hit)
4. Reduce header thrashing (fewer, larger resources)
5. Use vectorized loads (16B aligned fields)
#### BVH optimization
Every geometry is encapsulated by a bounding box. Ray traversal checks if the bounding box is hit. For overlapping bounding boxes, one might merge them or split them up. You can rotate the bounding box to align to local frame to reduce the space of the bounding box. Further topology optimizations could be to find identical subgraphs, extract instances, collapse nodes etc.


## Unreal Engine Lumen

### Nanite Virtualized Geometry
Nanite is Unreal Engine 5's virtualized geometry system which uses a new internal mesh format and rendering technology to render pixel scale detail and high object counts. It intelligently does work on only the detail that can be perceived and no more. Nanite's data format is also highly compressed, and supports fine-grained streaming with automatic level of detail. Thanks to https://dev.epicgames.com/documentation/en-us/unreal-engine/nanite-virtualized-geometry-in-unreal-engine

During import — meshes are analyzed and broken down into hierarchical clusters of triangle groups.

During rendering — clusters are swapped on the fly at varying levels of detail based on the camera view and connect perfectly without cracks to neighboring clusters within the same object. Data is streamed in on demand so that only visible detail needs to reside in memory. Nanite runs in its own rendering pass that completely bypasses traditional draw calls. Visualization modes can be used to inspect the Nanite pipeline.

### Mesh Distance Fields
A Signed Distance Field (SDF) is used to represent the surface of a mesh.

### Lit Translucenvy

### Volumetric Fog

### Lighting Update Speed
Lumen uses a number of caches to achieve real-time performance. While local lighting changes propagate quickly, global lighting changes, like disabling the Sun, can take multiple seconds to propagate. Thanks to https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine

## Unreal Engine 5.5 MegaLights
MegaLights uses stochastic sampling to render many shadow casting lights efficiently.


## Blender Eevee Next / Cycles
