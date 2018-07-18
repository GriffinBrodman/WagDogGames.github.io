---
title: Making a Portal Effect with the Shader Graph
layout: blog_post
description: I've been really excited about the launch of the ShaderGraph, and I wanted to share a small tutorial of how I made a cool, simple shader for the Hamster Portal.
leftsidetitle: ShaderGraph Portal Intro Level Tutorial
leftside: I used the new Unity ShaderGraph to create a procedural Portal texture, using screenspace coordinates and noise functions.
tags: LWRP ShaderGraph Unity
postimage: 2018posts/PortalShader/overview.PNG
previewimage: 2018posts/PortalShader/overview.PNG
authorlink: https://twitter.com/GriffinBrodman
authorname: Griffin Brodman
---

In Hamster Party, your goal will be to rotate tubes in order to provide a path for a hamster to travel through a Portal across space and time, finally arriving at the Hamster Party. I want to talk a little about the Portal I designed, and give a brief intro tutorial / overview of different aspects of the ShaderGraph to create an effect like I did.

One thing I've been really excited about for a long time has been the release of the ShaderGraph in Unity! I've spent a lot of time in Blender, and having a similar Node Graph system for creating shaders is a lot more intuitive to me.

For Hamster Party, we're using the Lightweight Render Pipeline. Our game is for mobile, and we don't plan on using any tricky lighting or shaders, so it suited our needs appropriately. So far my experience with it has been great. I enjoy having all of my graphics settings preset but fully customizable. The ShaderGraph is quite powerful, and while I do plan to write my own shaders as well, it can achieve complex effects quickly.

I certainly don't think I've achieved anything too special with my effect, but I know the ShaderGraph is relatively new and maybe this could give someone out there some tips or inspiration. To start off, I wanted my Portal Shader to have some swirling effect, to look cool and ethereal and purple. I looked into the three types of noise that the ShaderGraph currently supports: Simple Noise, Gradient Noise, and Voronoi. They all have varied and interesting uses, but the Simple Noise definitely looked the most like what I was going for - smooth transitions between random areas of white and black. I've used Voronoi noise before for things like leather or cracks in an egg, and Gradient Noise might be good for a bumpy surface, and there's lots of interesting reading behind the generation and uses of both of them.

![noise image](http://WagDogGames.com/img/2018posts/PortalShader/noise.PNG)

I then used the Lerp Node (Linear Interpolation) to color it. If you're unfamiliar, Linear Interpolation is when you have two input values, and a factor to interpolate (calculate some middle value) between them. The factor is known as T, and it's a percentage. The inputs can be expressed as A and B. For example, if you had the numbers 3 and 6, and you wanted to interpolate to find the value 40% between them, you'd calculate

```
(1-T) * A + T * B 
60% * 3 + 40% * 6, or 4.2.
```

In Unity, Lerp can be thought of as a "combine" node, where it takes some factor to combine two inputs. This gets interesting when T is something more complex than a simple value. In this case, we're supplying T as the image produced by the Simple Noise, so it has a different value for every pixel. The conversion between an image to a value can be thought of as "How white is this pixel?" A pure white pixel would be equal to 1, a pure black pixel would be 0. Using the Simple Noise as an input, every pixel is Lerped between the two colors I've selected for my shader (as seen below, purple and pink).

![image of colored lerp](http://WagDogGames.com/img/2018posts/PortalShader/lerp.PNG)

Now that I had color as a baseline, I wanted to tackle the dynamic aspect of the Portal: a swirling, churning movement. Since I knew I wanted to move my image, I hooked up the UV input from the Simple Noise to the Tiling and Offset Node. If you've never heard of a UV, it's the mapping for a mesh to get from 2D inputs to 3D inputs. When people create textures, they're created in 2D, so graphics programs need a way to wrap that 2D image onto the 3D mesh. The name is really logical actually - if a 3D object is made up of vertices with XYZ coordinates, it also has an associated 2D UV map with the axes U and V. UVs are a big part of working with 3D models, and are definitely worth learning about.

By passing a Tiling and Offset Node to the Simple Noise as an input, I can tile the image (make it repeat), and I can offset the image (move or shift it). By continually increasing the offset, I can create the impression that the Portal has a swirling magical energy. In order to get my desired effect, I needed to use time as my modifier of the offset. I split time into two Multiply Nodes to achieve separate movement speeds along the U and V axes (left/right and up/down).

![gif of portal](http://WagDogGames.com/img/2018posts/PortalShader/movement.gif)

I wanted to make the Portal Shader appear to be a tear in space. Immediately I knew I wanted the shader to be in terms of screen space, as opposed to the standard object space. In object space, the color of a point on an object is linked to the position of that point on that object. Basically, if you put a design on an object, and you rotated the object, you'd see the design rotate in equal measure. In screen space, however, the color of a point on an object is determined by its position on the screen. If you rotated an object with respect to the screen, the design would remain fixed on the screen despite the movement of the object. Imagine that the object is essentially a hole in your screen, through which you see the design plastered on the screen, immovable. For more about shaders and different spaces, I highly recommend "Makin' Stuff Look Good" and his [series on shaders](https://www.youtube.com/watch?v=T-HXmQAMhG0).


To accomplish the screen space effect, I added a Screen Position Node as an input to the Tiling and Offset Node. You can see the combined effect of these components of the Portal Shader below, when I applied the shader to a model I threw together.

![gif of screenspace portal](http://WagDogGames.com/img/2018posts/PortalShader/before.gif)

I was happy with these results, but I wanted to refine the effect further. I wasn't happy with two things - I wanted my Portal to look like there were more layers moving inside of it (for the sake of depth), and I didn't love how the Portal was being lit (current lighting was making the swirling effect hard to see).

To address adding a second moving layer, I added two more colors, another Lerp Node to combine them, different time multipliers so it would look like a different layer moving at a different speed, and then a final Lerp Node to combine both of my colored Lerps. That was a little more complex, and I'm not going to walk through that step by step in this tutorial.

To address the lighting issue, I took a look at the Master Shader I was using. In the ShaderGraph, there are two types of Master Shaders that you can work with: PBR and Unlit. At first, I was using the PBR Master Shader (Physically Based Rendering). This is a workflow for making physically accurate materials, giving you the ability to adjust things like smoothness, metalness, etc. It's a great feature and it can make some amazingly realistic looking materials that interact with light in a way that replicates real world objects. However, I didn't like that the lighting in my scene had a visual effect on this Portal; it made it hard to see the swirling, and conceptually I imagined that portals would make their own light. I decided to use the Unlit Master Shader instead, meaning my Portal would not interact with the lighting of the scene.

I added some Emission (glow) on a separate material on the rim of the Portal, as well as some post processing bloom. With these finishing touches, I began to be satisfied with how the Portal was looking. Below is my finished result.

![gif of finished portal](http://WagDogGames.com/img/2018posts/PortalShader/after.gif)

That's about it! I'm not a professional artist or game developer, so take this with a grain of salt. This is a pretty simple effect, but I hope this helps. If you have any questions, or just want to show off your really cool Portal Shader, feel free to tweet us at [@WagDogGames](https://twitter.com/WagDogGames), or send an email. The ShaderGraph is a very powerful tool, and I hope you get inspired to make something cool!