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

[Lumen Presentation](https://advances.realtimerendering.com/s2022/SIGGRAPH2022-Advances-Lumen-Wright%20et%20al.pdf)

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


## Stochastic sampling
[Stochastic Substitute Trees for Real-Time Global Illumination](https://www.tugraz.at/fileadmin/user_upload/Institute/ICG/Downloads/team_steinberger/Publications/SST.pdf)

[Stochastic Lightcuts for Sampling Many Lights](http://www.cemyuksel.com/research/stochasticlightcuts/stochasticlightcuts_tvcg.pdf)

[Lightcuts: A Scalable Approach to Illumination](https://www.graphics.cornell.edu/~bjw/lightcuts.pdf)


## Blender Eevee Next / Cycles
Eevee Next, compared to Cycles, provides faster render times but less realism. Eevee Next is known for its real-time rendering capabilities, making it ideal for tasks that require quick feedback or animations with complex lighting and shaders. It provides fast previews and interactivity during the creative process, which can be beneficial for artists who need rapid iterations.

On the other hand, Cycles is a ray-tracing engine that produces high-quality, photorealistic renders with accurate light interactions and global illumination. While it may be slower than Eevee Next for real-time previews, Cycles excels in creating final output for projects that demand top-notch visuals.

### Eevee Next Raytracing

Virtual Shadow Mapping like Lumen.

The render engine can now use screen surface ray tracing for every BSDF shader in a scene. Done by generating a ray from each BSDF and finding their intersection with the scene individually.

Methods:
- Light Probe: Use light-probe spheres and planes to find scene intersection. This option has the lowest tracing cost but relies on manually placed light-probes.
- Screen-Trace: Trace ray against the screen depth buffer. Fallback to light-probes if ray exits the view.

Denoising: Spatial Reuse, Temporal Accumulation, Bilateral Filter.

Fast GI Approximation is a fallback to the raytracing pipeline for BSDF using either Ambient Occlusion or Global Illumination.

## SSAO

Indirect lighting approximation by occlusion from surrounding geometry.

[Learn OpenGL article](https://learnopengl.com/Advanced-Lighting/SSAO)

Most modern implementation: Horizon-Based Ambient Occlusion Plus [HBAO+](https://developer.nvidia.com/rendering-technologies/horizon-based-ambient-occlusion-plus).

HBAO+ generates higher quality SSAO by using a physically based algorithm that approximates an integral with depth buffering sampling. It is much faster than HBAO.

## Shadow Mapping
Generate a depth map, then render the scene as normal and use the generated depth map to calculate whether fragments are in shadow.

[Learn OpenGL article](https://learnopengl.com/Advanced-Lighting/Shadows/Shadow-Mapping)

### Alternatives to Shadow Mapping
[NVIDIA Hybrid Frustum Traced Shadows](https://www.nvidia.com/en-us/geforce/news/nvidia-hfts-hybrid-frustum-traced-shadows) creates shadows that are close to the quality of ones created by Ray Tracing, but at a fraction of the performance cost.

[Virtual Shadow Maps](https://dev.epicgames.com/documentation/en-us/unreal-engine/virtual-shadow-maps-in-unreal-engine) is used in Unreal Engine 5's Lumen. Uses Shadow Map Ray Tracing for soft shadows. Works well with highly detailed Nanite geometry.


