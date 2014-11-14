CIS 565 Project 6: Deferred Shader
==================================

* Kai Ninomiya (Arch Linux, Intel i5-2410M)

### [Live Demo](https://kainino0x.github.io/Project6-DeferredShader/)

Demo image with bloom and SSAO enabled:

![](images/bloom_ssao.png)


Interface
---------

* WASDRF: Fly
* Mouse or arrow keys: rotate camera
* 1: View space positions
* 2: View space normals
* 3: Albedo color
* 4: Depth
* 0: Deferred render, reset to diffuse+specular & SSAO only
  * 6: Disable/enable diffuse+specular
  * 7: Enable/disable bloom
  * 8: Enable/disable toon
  * 9: Disable/enable SSAO


Effects
-------

### Diffuse & Specular lighting

This adds a lighting term which includes Lambert diffuse and Blinn-Phong
specular lighting.

![](images/diffuse.png)


### Bloom effect

Sum the entire image with a version blurred using a circular paraboloid.
The result of this is a bloom effect with circular highlights at configurable
radii and with a configurable factor.
(This kernel was chosen because it looks good, is fast to compute, and doesn't
require a convolution kernel texture).

![](images/bloom.png)


### Toon shading

Toon shading is performed using a ramp shading transformation on the diffuse
color output: mapping the diffuse factor range [0, 1] to a

Toon outlines are added by overwriting pixels with near-tangent view-space
normals. This method doesn't look very good, but avoids the need for additional
outputs to the post-processing stage and is considerably cheaper than a
post-processed effect which reads multiple input pixels.

![](images/toon.png)


### Screen-Space Ambient Occlusion

Screen-space ambient occlusion is calculated using a hemisphere method:
pseudorandom points in a center-weighted hemisphere are sampled to find out
whether they are occluded from the camera's perspective. I used John Chapman's
SSAO implementation notes to understand the general sampling methodology, then
implemented it from that understanding.

![](images/with_ssao.png)


Performance
-----------

Tests taken on the initial scene (without developer tools open).

As seen in the data below, removing features at compile time (using
preprocessor flags) gave slightly better performance than disabling at runtime.
The difference is fairly minimal, though (1.5 ms maximum difference). I suspect
this is due to block locality: since the flag is uniform, all threads in a
block will skip, and only a few comparison operations are introduced into the
machine instruction trace.


### Runtime Flags

| Effects               | Frame time |
|:--------------------- | ----------:|
| No shading            |    10.5 ms |
| Diffuse+specular only |    +0.0 ms |
| Bloom only            |   +69.5 ms |
| Toon only             |    +0.0 ms |
| SSAO only             |    +4.0 ms |
| Bloom & SSAO          |   +74.0 ms |


### Compile-Time Switches

| Effects               | Frame time |
|:--------------------- | ----------:|
| No shading            |     9.0 ms |
| Diffuse+specular only |    +1.5 ms |
| Bloom only            |   +69.5 ms |
| Toon only             |    +0.5 ms |
| SSAO only             |    +5.0 ms |
| Bloom & SSAO          |   +74.5 ms |


References
----------

[1] John Chapman's SSAO implementation notes.
    http://john-chapman-graphics.blogspot.com/2013/01/ssao-tutorial.html

[2] stats.js. Copyright 2009-2012 Mr.doob. Used under MIT License.
    https://github.com/mrdoob/stats.js


Starter Code Acknowledgements
-----------------------------

Many thanks to Cheng-Tso Lin, whose framework for CIS700 we used for this
assignment.

This project makes use of [three.js](http://www.threejs.org).
