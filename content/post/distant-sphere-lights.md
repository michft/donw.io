+++
date = "2015-02-11T12:58:23+01:00"
draft = false
title = "Distant Spherical Area Light Sources"
tags = [ "pbr", "lighting" ]
+++

This is a small test post exercising math notation with [MathJax](http://www.mathjax.org/) in [Hugo](https://gohugo.io/).

<!--more-->

Epic use the following equation for a spherical light source  \[[1]\]:

$$\mathbf{C} = (\mathbf{L} \cdot \mathbf{r})\mathbf{r} - \mathbf{L}$$
$$\mathbf{P} = \mathbf{L} + \langle \frac{sourceRadius}{ &verbar; \mathbf{C} &verbar; } \rangle \mathbf{C}$$

where the point with the smallest distance to the reflected ray is then $&par;\mathbf{P}&par;$.

Implicit in this equation is that $\mathbf{L}$ is unnormalised. With a Sun/Earth distance of 149,600,000km and Sun radius of 695,800km these calculations are not really going to work in limited precision shaders. To get around this we need to separate the length of $\mathbf{L}$ from its direction and hope that the large value terms fall out at the end. Using this substitution:

$$d\mathbf{l} = \mathbf{L}$$

we start with:

$$\mathbf{C} = (d\mathbf{l} \cdot \mathbf{r})\mathbf{r} - d\mathbf{l}$$

As the dot product is homogeneous under scaling we can pull $d$ out and factor it:

$$\mathbf{C} = d((\mathbf{l} \cdot \mathbf{r})\mathbf{r} - \mathbf{l})$$

leaving $\mathbf{P}$ at:

$$\mathbf{P} = d\mathbf{l} + \langle \frac{sourceRadius}{ sqrt(\mathbf{C} \cdot \mathbf{C}) } \rangle \mathbf{C}$$

Things become a little easier if we make the simple substitution:

$$\mathbf{D} = (\mathbf{l} \cdot \mathbf{r})\mathbf{r} - \mathbf{l}$$
$$\mathbf{C} = d\mathbf{D}$$

After a little reduction, $\mathbf{P}$ simplifies to:

$$\mathbf{P} = d\mathbf{l} + \langle \frac{sourceRadius}{d} \frac{1}{sqrt(\mathbf{D} \cdot \mathbf{D})} \rangle d\mathbf{D}$$

This allows two things:

* The $\frac{sourceRadius}{d}$ term can be precalculated outside the shader at whatever precision you like. As long as the result is float-representable then you're good to go. As an example, the Sun/Earth ratio is roughly 0.00465.
* The remaining $d$ scalar can be factored and ignored as we're only interested in the direction of $\mathbf{P}$.

The shader code is just as simple:

~~~cpp
float3 r = reflect(surface_to_camera, normal);
float3 D = dot(l, r) * r - l;
float3 P = l + D * saturate(radius_over_distance * rsqrt(dot(D, D)));
float3 specular_l = normalize(P);
~~~

This is a little simpler than the area disk light presented in \[[2]\] and allows you to use Epic's energy normalisation constant.

References:

* [Real Shading in Unreal Engine 4][1]
* [Moving Frostbite to PBR][2]

[1]: http://blog.selfshadow.com/publications/s2013-shading-course/ "Real Shading in Unreal Engine 4"
[2]: http://www.frostbite.com/2014/11/moving-frostbite-to-pbr/ "Moving Frostbite to PBR"
