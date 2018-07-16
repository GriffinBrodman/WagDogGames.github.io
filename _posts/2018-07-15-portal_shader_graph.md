---
title: Making a Portal Effect with the Shader Graph
layout: blog_post
description: I've been really excited about the launch of the ShaderGraph, and I wanted to share a small tutorial of how I made a cool, simple shader for the Hamster Portal.
leftsidetitle: ShaderGraph Portal Intro Level Tutorial
leftside: I used the new Unity ShaderGraph to create a procedural portal texture, using screenspace coordinates and noise functions.
tags: LWRP ShaderGraph Unity
postimage: 2018posts/PortalShader/overview.PNG
previewimage: 2018posts/PortalShader/overview.PNG
authorlink: https://twitter.com/GriffinBrodman
authorname: Griffin Brodman
---

One thing I've been really excited about for a long time has been the release of the ShaderGraph in Unity! I've spent a lot of time in Blender, and having a Node Graph system for creating shaders just is a lot more intuitive to me.

For Hamster Party, we're using the Lightweight Render Pipeline. Our game is for mobile, and we don't plan on using any tricky lighting or shaders, so it seemed like a great thing to try out. So far my experience with it has been great. I enjoy having all of my graphics settings preset for the type of project I'm making but also being fully customizable at the same time time. The ShaderGraph is quite powerful, and while I do plan to write my own shaders as well, it can just achieve complex effects quickly.

In Hamster Party, you find yourself having to go through Hamster Tubes, through a portal across space and time, and having to rescue hamsters by rotating their tubes so they can make their way back to the party.

For this portal, I knew I wanted some cool swirling effect, and I also wanted to make it look like a tear in space. Immediately I knew I probably wanted a shader in terms of screen space, where the color of a pixel is determined by its position on the screen, as compared to a shader terms of object space or the UVs, determined by the pixel's position on the object. A good, simplified illustration is that if you had a sphere with some design, and you rotated the sphere, if the shader was in terms of the UVs or Object Space, the image would rotate with the object If the shader was in terms of Screen Space, the image would not change at all. I like to think of it as if you had an image behind your screen, and the mesh that you put your shader on was like a hole in the screen, if that makes sense. For more about shaders, I highly recommend "Makin' Stuff Look Good" and his [series on Shaders](https://www.youtube.com/watch?v=T-HXmQAMhG0).

I certainly don't think I've achieved anything too special with my effect, but I know the ShaderGraph is relatively new and maybe this could give someone out there some tips or inspiration. To start off, I wanted my Shader to have some swirling effect, to look cool and ethereal and purple. I looked into the three types of noise that the ShaderGraph currently supports, Simple Noise, Gradient Noise, and Voronoi. They all have varied and interesting uses, but the Simple Noise definitely looked the most like what I was going for. I've used Voronoi noise before for things like leather or cracks in an egg, and Gradient Noise might be good for a bumpy surface, and there's lots of interesting reading behind the generation and uses of both of them.

![noise image](http://WagDogGames.com/img/2018posts/PortalShader/noise.PNG)

I then used the Lerp Node (Linear Interpolation) to color it. If you're unfamiliar, Linear Interpolation is when you have two input values, and a factor to interpolate between them (T in the Lerp Node).If you had the numbers 3 and 6, and you wanted to interpolate to find the value 40% betweem them, you'd calculate

```
(1-T) * A + T * B 
60% * 3 + 40% * 6, or 4.2.
```

In Unity, Lerp can be thought of as a "combine" node, where it takes some factor to combine them. This gets interesting when you supply T as something more complex than a simple value. In this case, we're supplying T as the image produced by the Gradient Noise. The conversion between an image to a value can be thought of as "How white is this pixel?" A pure white pixel would be a 1, a pure black pixel would be a 0. Using the Simple Noise as an input, every pixel is Lerped between the two colors I've selected, and you can see the results in the image below. (Note: For most shaders, I would use an input here so that these colors could be independently set per material, but I wanted to keep things simple for this tutorial. A Unity Material is basically an instance of a shader, so if you have inputs to your shader, different materials could be providing different inputs to the same shader for a different effect.)

![image of colored lerp](http://WagDogGames.com/img/2018posts/PortalShader/lerp.PNG)

Now that I had the color I wanted, I figured that all good portals are swirling and moving. Since I knew I wanted to move my texture, I hooked up the UV input from the Gradient Noise to the Tiling and Offset Node. If you've never heard of a UV, it's the mapping for a mesh to get from 2D inputs to 3D inputs. When people create textures, they're created in 2D, so graphics programs need a way to wrap that 2D image onto the 3D mesh. The name is really cute actually, if an object is made up of vertices with XYZ coordinates, it also has a 2D UV map made up of U and V. Like letters, get it? UVs are a big part of working with 3D models, and definitely worth learning about.

By passing in a Tiling and Offset Node, I can tile the image, or I can shift (offset) the image. By moving the offset, I can make it look like the portal has some swirling magical energies of old. But I wanted it to continually swirl, so I needed to use time as my input. I split time into two multiply Nodes, and then combine them, so that I could control the speed at which the portal moves up/down and left/right independently, (along the U and V axes). Putting this all together, I get an effect that looks like this. (Note: this is the first GIF I've ever taken and I was still learning, ignore the mouse and random boxes here and in other gifs)

![gif of portal](http://WagDogGames.com/img/2018posts/PortalShader/movement.gif)

At this point, I was almost done, but like I said before, I was interested in a more otherworldly look. I added a Screen Position Node here, and used it for the Tiling and Offset Node. You can see the effect of this below, when I applied the shader to a model I threw together.

![gif of screenspace portal](http://WagDogGames.com/img/2018posts/PortalShader/before.gif)

This was pretty good, but I thought I could do better. I wasn't happy with two things - I wanted my portal to look like there were more layers moving inside it, and I didn't love how the portal was being lit.

At first, I was using the PBR Master Shader, PBR standing for Physically Based Rendering. This is a workflow for making physically accurate materials, being able to adjust things like smoothness, metalness, etc. It's a great feature and it can make some amazingly realistic looking materials. That being said, I didn't love how my scene lighting was interacting with the portal, it made it hard to see the swirling. I decided to use the Unlit Master shader instead, meaning it would not interact with lighting in the scene. With some Emission on a seperate material on the rim of the portal, and enabling some post processing bloom, I began to get satisfied with how the portal was looking.

The last thing I added was two more colors, another Lerp Node to combine them, different time multipliers so it would look like a different layer,  and then a final Lerp Node to combine both of my colored Lerps. 

![gif of finished portal](http://WagDogGames.com/img/2018posts/PortalShader/after.gif)

That's about it! I totally might have done things wrong, and it's still a basic effect but I hope this helps. If you have any questions, or just want to show off your really cool portal shader, feel free to tweet us at [@WagDogGames](https://twitter.com/WagDogGames), or send an email. The ShaderGraph is a very powerful tool, and I hope you get inspired to make something cool!