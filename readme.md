# Three.js Performance Tips:

1.  Our goal should be to achieve 60 fps at least.
2.  Keep an eye on performance and test across multiple devices, kepp an eye on how heavy is our website.
3.  We need to measure the performance, the first thing we need to know is FPS.
    We can use a Javascript FPS meter like stats.js

4.  Monitoring Draw calls:
    Draw calls are actions of drawing by the GPU. The less draw calls we have the better is the performance.

    To monitor this we have a chrome extension named Spector.js

5.  Renderer information:
    console.log(renderer.info)

6.  Keep good javascript code:
    Keep a performant native javascript code especially in the tick function.
7.  Dispose of things:
    Once we are sure that we don't need a resource like a geometry or a material, dispose it.

    scene.remove(cube);

    // ==> The cube is removed from the scene but the geometry and material is still present.
    // ==> To remove geometry and material we need to dispose them.
    cube.geometry.dispose();
    cube.material.dispose();

8.  Avoid using Three.js lights.
    Use baked lights or use cheap lights like (AmbientLight, DirectionalLight, HemisphereLight)

9.  Avoid Adding and removing lights:
    When adding or removing light from the scene, all the materials supporting lights will have to be recompiled.

10. Shadows:
    Avoid Three.js shadows, use baked shadows.

11. Optimize Shadow Maps:
    Make sure the shadow maps fit perfectly with the scene.

    Try to use smallest possible resolution with a descent result for the mapSize
    directionalLight.shadow.mapSize.set(1024, 1024);

12. Use CastShadow and RecieveShadow wisely:

        cube.castShadow = true
        cube.receiveShadow = false

        torusKnot.castShadow = true
        torusKnot.receiveShadow = false

        sphere.castShadow = true
        sphere.receiveShadow = false

        floor.castShadow = false
        floor.receiveShadow = true

13. Deactive shadow Auto Update:

    Shadow maps get updated before each render.
    We can deactivate this auto-update and alert three.js that the shadow maps needs to be updated only when necessary

    renderer.shadowMap.autoUpdate = false
    renderer.shadowMap.needsUpdate = true

14. Textures :

    Textures take a lot of space in the GPU memory especially with the mipmaps.
    The Texture file weight has nothing to do with that, only the resolution matters. We should try to reduce the resolution to the minimum while keeping a decent result.

    Keep a power of 2 resolutions:

        While resizing, remember to keep a power of 2 resolution for mipmaps, The resolutions doesn't need to be square. If we don't do this three.js will try to fix it by resizing the image to the closest power of 2 resolution.

    Use Right format:
    Using the right format can reduce the loading time, We can use .jpg or .png according to the image and compression but also the alpha

        We can use online tools like TinyPNG to reduce the weight even more.

    We can also use basis format:
    Basis is format just like .jpg and .png but the compression is powerful, and the format can be read by the GPU more easily.
    But it is hard to generate and it's a lossy compression.

15. Geometries:

    All Geometries are now BufferGeometries, without having to use the word buffer.
    Thus there is no point in doing optimisation for geometries.

16. Updating the vertices of a geometry is terrible for the performance avoid doing it in the tick() function.
    If we want to animate the vertices, we should do it in the vertex shaders.

17. Mutualize Geometries:
    If we want to have multiples Meshes using the same geometry shape, create only one geometry, and use it on all the meshes.

18. Merge Geomtries:
    While creating multiple meshes even with mutual geometry there will be mulitple draw calls that is bad for performance, what we can do is to merge all the draw calls togethers so we draw the same amount of triangles but only in one draw call.

    If the geometries are not supposed to move, we can merge them by using the 'BufferGeometryUtils'.

    We don't need to instantiate it, and we can use its methods directly.

    Use the mergeBufferGeomtries(...) with an array of geometries. We can then use the merged geometry with a single Mesh.

19. Materials:

    Mutualize Materials:
    If we are using the same type of material for multiple meshes, create only one and use it multiple times.

    Cheap Materials:
    Also try to use cheap materials. MeshStandardMaterial or MeshPhysicalMaterial need more resources than materials such as MeshBasicMaterial, MeshLambertMaterial or MeshPhongMaterial.

20. Meshes:
    Use InstancedMesh:

    When we merge geometries together we cannot move each cube seperatly, If we need to move one cube than we have to update all the vertices of the geometry.

    we can use instanced mesh to over come this issue.

    It's like we are going to create only one InstancedMesh and we provide transformation matrix for each "instance" of that mesh.
    The matrix has to be Matrix4, and we can apply any transformation by using the various available methods.

    If we know we are going to update the matrices in the tick function, we have to add this code
    mesh.instanceMatrix.setUsage(THREE.DynamicDrawUsage);

21. Models:

    1. low-polygon:
       The fewer the polygons, the better. If we need details we can use normal maps.

    2. Draco compression:
       If the model has a lot of details with very complex geometries, use the DRACO compression. This will result in freeze in the beginning because it needs to be decompressed the geometries and we also need to load the draco library.

    3. Gzip:
       Gzip is the compression happening on the server side. Most of the servers do not support gzip for files like .glb, .gltf, .obj etc.

22. Camera:

    1. Field of View:
       When objects are not in the field of view, they won't be rendered (frustum culling). This seems to be not a good solution but we can use it to render less object.

    2. Near and Far:
       Like field of view, we can reduce the near and far properties of the camera.

23. Renderer:

    1. Pixel ratio:
       When we set the renderer pixel ratio, don't use the default pixel ratio as some devices can have large pixel ratio. 2 is suffiecient. Use the min value using Math.min(...)

       renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

    2. Power Preferences:
       Some devices are able to switch between different GPU or different GPD usage. We can tell the WebGLRendererer on what power is required when instanciating the WebGLRendererer by specifying a 'powerPreference' property

       renderer = new THREE.WebGLRendererer({
       canvas: canvas,
       powerPreference: 'high-performance'
       })

    3. Antialias:
       The default antialias is performant, but less performant than no antialias. Only add when we have some visible aliasing and no performance issue.

    4. Post processing:
       Limit passes:
       Each post-processing pass will take as many pixels as the render's resolution(including pixel ratio) to render.

       If we have a 1920x1080 resolution with 4 passes and pixel ratio of 2, that makes 1920*2*1080*2*4 pixels to render.

       Try to regroup our custom passes into one.

    5. Shaders:
       We can force the precision of the shaders in the materials by changing their precision property.

       const shaderMaterial = new THREE.ShaderMaterial({
       precision: 'lowp'
       })

       This won't work with RawShaderMaterial, and you'll have to add the 'precision' by yourself on the shaders like we did on the shader.

    6. Textures:
       Using perlin noise functions is cool, but it can affect our performance considerably. Sometimes we better use textures representing noise.

    7. Use defines in shaders.

    8. Do the calculations in the vertex shaders and send the result to the fragment shader.
