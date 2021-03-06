# 📦 gltfpack

gltfpack is a tool that can automatically optimize glTF files to reduce the download size and improve loading and rendering speed.

## Installation

You can download a pre-built binary for gltfpack on [Releases page](https://github.com/zeux/meshoptimizer/releases), or install [npm package](https://www.npmjs.com/package/gltfpack) as follows:

```
npm install -g gltfpack
```

## Usage

To convert a glTF file using gltfpack, run the command-line binary like this on an input `.gltf`/`.glb`/`.obj` file (run it without arguments for a list of options):

```
gltfpack -i scene.gltf -o scene.glb
```

gltfpack substantially changes the glTF data by optimizing the meshes for vertex fetch and transform cache, quantizing the geometry to reduce the memory consumption and size, merging meshes to reduce the draw call count, quantizing and resampling animations to reduce animation size and simplify playback, and pruning the node tree by removing or collapsing redundant nodes. It will also simplify the meshes when requested to do so.

By default gltfpack outputs regular `.glb`/`.gltf` files that have been optimized for GPU consumption using various cache optimizers and quantization. These files can be loaded by GLTF loaders that support `KHR_mesh_quantization` extension such as [three.js](https://threejs.org/) (r111+) and [Babylon.js](https://www.babylonjs.com/) (4.1+).

When using `-c` option, gltfpack outputs compressed `.glb`/`.gltf` files that use meshoptimizer codecs to reduce the download size further. Loading these files requires extending GLTF loaders with support for `MESHOPT_compression` extension; [demo/GLTFLoader.js](https://github.com/zeux/meshoptimizer/blob/master/demo/GLTFLoader.js) contains a custom version of three.js loader that can be used to load them.

For better compression, you can use `-cc` option which applies additional compression; additionally make sure that your content delivery method is configured to use deflate (gzip) - meshoptimizer codecs are designed to produce output that can be compressed further with general purpose compressors.

gltfpack can also compress textures using Basis Universal format, either storing .basis images directly (`-tb` flag, supported by three.js) or using KTX2 container (`-tc` flag, requires support for `KHR_image_ktx2`). Compression is performed using `basisu` executable that must be available in `PATH`; alternatively the path to the executable can be specified via `BASISU_PATH` environment variable. Textures can also be embedded into `.bin`/`.glb` output using `-te` flag.

When using compressed files, [js/meshopt_decoder.js](https://github.com/zeux/meshoptimizer/blob/master/js/meshopt_decoder.js) needs to be loaded to provide the WebAssembly decoder module like this:

```js
import './meshopt_decoder.js'; // imports MeshoptDecoder using ES6

...

var loader = new THREE.GLTFLoader();
loader.setMeshoptDecoder(MeshoptDecoder);
loader.load('pirate.glb', function (gltf) { scene.add(gltf.scene); });
```

## Options

By default gltfpack makes certain assumptions when optimizing the scenes, for example meshes that belong to nodes that aren't animated can be merged together, and has some defaults that represent a tradeoff between precision and size that are picked to fit most use cases. However, in some cases the resulting `.gltf` file needs to retain some way for the application to manipulate individual scene elements, and in other cases precision or size are more important to optimize for. gltfpack has a rich set of command line options to control various aspects of its behavior, with the full list available via `gltfpack -h`.

The following settings are frequently used to reduce the resulting data size:

* `-cc`: produce compressed gltf/glb files (requires `EXT_meshopt_compression`)
* `-tc`: convert all textures to KTX2 with BasisU supercompression (using basisu executable; requires `KHR_image_basisu`)
* `-mi`: use mesh instancing when serializing references to the same meshes (requires `EXT_mesh_gpu_instancing`)
* `-si R`: simplify meshes to achieve the ratio R (default: 1; R should be between 0 and 1)

The following settings are frequently used to restrict some optimizations:

* `-kn`: keep named nodes and meshes attached to named nodes so that named nodes can be transformed externally
* `-km`: keep named materials and disable named material merging
* `-ke`: keep extras data
