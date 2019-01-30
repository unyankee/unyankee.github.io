---
layout: post
title:  "OutterSpace III"
date:   2019-1-20
categories: 
---


# Speeding up the raytracer.

I have been testing several approaches to speed up this stage, because, just using a single model, with a lot of triangles
is a big performance issue. Iterate over all the triangles a huge mesh has nowadays is such a heavy operation.


So following what I was testing [on the last post]({% post_url 2018-12-11-OutterSpace II %}), and since this is aiming to be my last end year project, I took tips from my project supervisor. Who told me about some really interesting ways of handling this issue.

So after all, I end up with a more accurate idea of what I should try.


* [New Buffers Layout](#new-buffers-layout)
  *  [BB](#bb)
  *  [VB](#vb)
  *  [IB](#ib)
  *  [Meta Buffer](#meta-buffer)
  *  [Instance Buffer](#instance-buffer)
  *  [Materials Buffer](#materials-buffer)
* [Getting right values](#getting-right-values)
* [Some test will be added here](#some-test-will-be-added-here)



## New Buffers Layout

![My helpful screenshot](/assets/OutterSpace/BuffersLayout.png)


A general idea is to completely split up all the data, on separated buffers, one per each data type.

#### BB


Is the Boundings Buffer, this is a bit special because I am still working on it, it is not a simple
Bounding box, instead of, is an Octree mesh partition of the mesh, with all the indices that are inside each volume.

The way it performs is checking for a maximum amount of triangles within an area. If the amount is higher than the top, it generates
8 children, with the right side, and recalculate.   

If no points are on the area, stops, and the area is marked to not continue.

Can be expensive, but is a loading operation, it is not intended to do it on run-time.

Here are some of the results it generates.




![My helpful screenshot](/assets/OutterSpace/octree1.jpg)  

![My helpful screenshot](/assets/OutterSpace/octree2.jpg)


#### VB

Is the vertex buffer, stores the Vertex struct, for all the meshes loaded on the engine.  
So, each mesh loaded on the engine is stored on this vector, just one time, and sent to the GPU.


{% highlight c++ linenos %}
struct Vertex {
	glm::vec3 pos;
	float __padding1;
	glm::vec3 color;
	float __padding2;
	glm::vec3 normals;
	float __padding3;
	glm::vec2 texCoord;
	float __padding4[2];
};
{% endhighlight %}	



#### IB


It so similar to the VB, but instead of storing a Struct, it stores all indexes of all the meshes.

#### Meta Buffer

Is a special buffer, it contains, for each mesh, loaded on a specific order, the position on the others buffers seen above.
So let's say we need to draw a Cube, and the system has a Cylinder and a Cube loaded, on that specific order.
That means, when the meta buffer is filled, whoever had the cube, is going to store here, the offset of each component.
Where starts the Cube vertex data, on the VB, and how long it is. This is applicated for the rest of buffers.


{% highlight c++ linenos %}
struct MetaBuffer {
	uint32 VertexOffset;
	uint32 VertexSize;

	uint32 IndexOffset;
	uint32 IndexSize;

	uint32 BoundingOffset;
	uint32 BoundingSize;
	
	uint32 __padding[2];	
};
{% endhighlight %}	

As you can see, I still have to add some values, that is because I am splitting the process, and working on each stage
individually. I want to be completely sure, it is working, and is the method I am going to follow before completing it.


#### Instance Buffer

This is the smallest one and contains the world matrix for each object to draw, and the position on the meta buffer, where 
the information needs to be picked up.


> *There is one per ach object to draw.*

{% highlight c++ linenos %}
struct PerInstance {
	glm::mat4 worldMatrix;
	uint32 metaOffset;
	uint32 materialOffset;
	uint32 __Padding[2];
};
{% endhighlight %}	



#### Materials Buffer

I am still researching how to implement this buffer because I want to be able to perform different things, and maybe I will need to upload textures, and some extra values, to represent colors, 
roughness, metallicness.  So I am still considering how to implement this part, meanwhile I am researching.



# Getting right values

Some quick fix since the last update is that thanks to using barycentric coordinates to check if a Ray hit a triangle, I can use the Coordinates as Weights, and interpolate the values of each vertex to get the right one.


![My helpful screenshot](/assets/OutterSpace/RightImage.png)

# Some test will be added here


Right now I can only say, that using a simple Bounding Box, in this case, the first Octree value, it means a significant boost. Because if there is nothing the ray can collide with, is not going to be calculated.
If there is nothing to be calculated I am assuming is the sky color.

Once I finished the octree mesh split up implementation, I will add to this post, a comparison between all the different stages I am going through.




To Continue





