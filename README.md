# Physically Based Rendering (PBR)
The goal of this project is to implement a PBR renderer. You are asked to implement:
* A Lambertian diffuse BRDF
* A Cook-Torrance GGX specular BRDF


## Grading
* Group size: **1**
* Due date: TODO
* Format: TODO


## Installation
After cloning the repository, install the dependencies using:

```sh
yarn # Alternatively you can run `npm install`
```


## Running
You can start the development server using:

```sh
yarn dev # Alternatively you can run `npm run dev`
```

You can now reach [localhost:8080](http://localhost:8080) to try your code.
The development server supports Hot-Reload, which means that when saving your code, the page
will automatically auto-reload with the latest changes.


## Provided Code

### Index
The [index](./src/index.ts) is the entry point of your application. The game loop is started there
and resources are initialized in the `Application` class.

In this repository, I created a simple shader that sets a uniform color on a triangle. This sample
will help you start to implement your PBR shader.

### Context
The [context](./src/gl.ts) is one of the most important. It abstracts WebGL calls and resources management.
Without that, you would need to spend quite a bit of code to setup:
* Geometries (Vertex Buffers)
* Textures
* Shaders
* etc...

I didn't want you to spend your time writing an abstraction, so I made one for you. If you want fancier features, please feel free to update it with your own changes.

For the most curious ones, the file uses `WeakMap` (basically hash tables) to retrieve uploaded GL objects from your own instances (textures, geometries, etc...).

### Shader
The [Shader](./src/shader/shader.ts) just contains the two shaders (vertex and fragment) as strings.
It also contains a dictionnary (`defines`) that can allow you to conditionnally compile code or not.

When working on big rendering project, we often want several versions of a same shader with some differences.
Using `#define`, we take advantage of the preproccessor to compile different variants.

The values in the `defines` dictionnary will basically be preprended to your shader before compiling it.


## Tooling
[Spector.js](https://chrome.google.com/webstore/detail/spectorjs/denbgaamihkadbghdceggmchnflmhpmk?hl=en) is a chrome extension that will help you analyze data sent to webgl, visualize framebuffers, etc... The best debugging tool for WebGL!


## Assignment
You are free to proceed the way you want. However, I would advise you to
go step by step and ensure intermediate results are correct. Otherwise, you will spend time
trying to figure out why your specular is off, when the issue might actually comes from a different part of the pipeline.


### Warm up (0 points)
This short part will help you discover the code. It will be used in the next section, so don't skip it.
1. Draw the `normal` vector
2. Draw the `viewDirection` vector


### Direct Lighting (2 points)
Implement point lights, use at least 2 in your scene.
Send simple light data to the shader (color, intensity, position or direction), compute their attenuation and display them.
The file `light.ts` already contains some of the code needed.

In addition, you can also implement directional lights if you want.

### Diffuse BRDF (2 points)
Implement the Lambertian diffuse BRDF.

### Specular BRDF (6 points)
Implement the Cook-Torrance GGX specular BRDF.

This is the kind of results you should get with 4 point lights:
![Example of results you should obtain with point lights](./screenshots/pointlights.jpg)


### Image-Based Lighting Lookup
For the Image-Based Lighting, the textures are encoded in **RGBM**. In order to map **RGBM** to **RGB**, you will need to find the formula somewhere. It consists in a simple linear range remapping, with a constant range multiplier of 6.


#### Image-Based Lighting: Diffuse (1 points)
The tasks to accomplish to lit your objects with the diffuse IBL are:
1. Load one of the  `diffuse` files provied in the folder `assets/env`
2. Use the geometry normal to sample the texture. Be careful here, the texture
   is saved as an [equirectangular projection](https://en.wikipedia.org/wiki/Equirectangular_projection). Start by converting your cartesian coordinates to polar coordinates using the given function `cartesianToPolar`. Then remap these coordinates to use them as equirectangular UV coordinates.
4. Apply the texture contribution to the indirect lighting

This is the kind of results you should get with the diffuse texture `Alexs_Apt_2k-diffuse-RGBM.png`:
![Example of results you should obtain using only the diffuse IBL](./screenshots/ibl-diffuse.jpg)


#### Image-Based Lighting: Specular (2 points)
For the specular IBL, the texture encodes different version of the environment
for different roughness values. There is a total of **6** roughness levels, starting
at the bottom of the texture.

Each level is **half the size** of the previous one. Thus, you will need to
compute the good position of the UV from the roughness value.

In order to get proper blending, you are advised to sample two roughness levels
simultaneously, and to blend them together.

The tasks can be summed up as:
1. Load one of the  `specular` files provied in the folder `assets/env`
2. Load the texture `assets/ggx-brdf-integrated.png` containing the precomputed BRDF. This texture uses sRGB color space, don't forget to remap to linear color space.
3. Convert the reflected ray from cartesian to polar
4. Offset the polar coordinates according to the roughness level
5. Repeat step **2** and **3** for a second level
6. Fetch both levels and blend them together according to how far between the two the sample was
7. Apply the result to the rendering equation using the pre-computed BRDF

This is the kind of results you should get with the diffuse texture `Alexs_Apt_2k-specular-RGBM.png`:
![Example of results you should obtain using only the diffuse IBL](./screenshots/ibl-specular.jpg)

Now that you implemented both the diffuse and the specular IBL, take a look at the combined results:

![Example of results you should obtain with both the diffuse and specular IBL](./screenshots/ibl-total.jpg)


### Image-Based Lighting Generation
For this project, you have worked with pre-computed data. Instead of using the asses from the repository,
try to generate yourself the cached environment textures.


#### Image-Based Lighting: Diffuse (5 points)
> Careful: Compute shaders aren't available in WebGL.

In this steps, you are asked to write a compute / fragment shader to generate the convoluted diffuse. This should obviouly done only once. You can do it once when your application is starting up. A lag of a few milliseconds might occur, which is totally fine.

Steps:
1. Create a framebuffer
2. Create a texture
3. Attach the texture to the framebuffer
4. Create a shader that will convolute the environment diffuse and write it to the texture
5. Use this result in your PBR shader


## Bonus

### Other BRDF (0.5 points)
You can experiment with other BRDFs (diffuse or specular), such as:
* Burley
* Oren-Nayar
* Ward

Whatever you want to try!


### Textures (1.5 points)
PBR is meaningless without carefully authored textures bringing complexity to materials.

You can download some texture that would map well to a sphere, such as [those ones](http://freepbr.com/materials/rusted-iron-pbr-metal-material-alt/).


### Image-Based Lighting Generation

#### Specular (8 points)
Just like you did for the diffuse, you can generate the specular probe. This task is harder, but will definitely make you stronger.

Please refer to the paper [Unreal paper](https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf) for the implementation. Don't hesitate to come see me during the lesson so I can give you a detailed explanation about what you have to do.
