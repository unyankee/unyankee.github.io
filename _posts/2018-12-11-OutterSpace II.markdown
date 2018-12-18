---
layout: post
title:  "OutterSpace II"
date:   2018-12-18
categories: 
---
## My first realtime Raytracer  

![My helpful screenshot](/assets/OutterSpace/Screenshot_4.png)

For this part of my project, I am researching about the best way of dealing with raytracing models.  
As you can see, I am currently raytracing different models, it works, but the loop that needs to be done is quite a mess.  

The image shows a rasterized cube, showing its normals on the left side, and on the right side is the version of the same, but completely raytraced.

The most important thing I am trying to achieve is to avoid the maximum amount of loops on the shader, so I have different compute shaders.  

One is for the init of the rays, dealing with a shared buffer between shaders, that is populated with the corresponding data.  

It uses, the projection matrix of the camera, and the view matrix, to get the inverse, and using the NDC, get the corresponding world coordinates, this way I do not have to deal with sending any position to the shader, everything is on this 2 matrixes.  
Taking profit of the **gl_GlobalInvocationID**, that I use as a pixel Coordinate because there is a corresponding match, between image
resolution, and this thread identifier,

The other one deals with the whole process of raytracing, but, I am still researching information about compute shaders, because, to avoid 
the maximum number of loops in this shader, I am taking profit of the WorkGroups, so my compute shader call, looks something like this.

{% highlight c++ linenos %}
//...
// Init camera rays compute shader
glDispatchCompute((GLuint)(float(testureW)/ 8.0f), (GLuint)(float(testureH)/ 8.0f), samplesZ);
//...
// Raytracer compute shader
glDispatchCompute((GLuint)(float(testureW)/ 8.0f), (GLuint)(float(testureH)/ 8.0f), samplesZ * bounces);
{% endhighlight %}	

The **samplesZ** variable, is the number of samples the ray is going to do, and then being sampled, or is the intention, because, right now
is set all the time to 1, the same happens to the **bounces** variable.  

The idea is to take profit of a 3D call, used, on the camera init side, to generate a number of rays, that could be sampled, and get
the average of all the colors. And on the other side, each time the ray ends one execution, it writes itself on the shared buffer, so
when it's called again, it continues. This way, we have completely erased the for loop for averaging the final color, and the for loop for moving the ray once per each time it hits something.

On the other side, on the compute shader, I have this:

{% highlight glsl linenos %}
layout(local_size_x = 8, local_size_y = 8,local_size_z = 1) in;
{% endhighlight %}	

To complete the whole range of the buffer.

All this has been done, thanks to the excellent explanation that can be found [here](https://medium.com/@jcowles/gpu-ray-tracing-in-one-weekend-3e7d874b3b0f).  
The owner is [Jeremy Cowles](https://medium.com/@jcowles), the ML Graphics Lead, on Unity Technologies.


![My helpful screenshot](/assets/OutterSpace/Screenshot_5.png)

Here what I try to do, is to find the triangle that hit the ray, but, as I am currently researching which is the best way of 
sending this info to GPU, I am only sending one single model, in this case, a Sphere.

{% highlight c++ linenos %}
bool hitTriangleObject(in Object obj, in Ray r, float tmin, float tmax, inout HitRecord hr) {  
  for(int i = 0; i < meshAmount;i+=3){
    vec3 a,b,c;
    a = vec3(meshPosition_[(meshIndexPosition_[i + 0]) * 11 + 0],meshPosition_[(meshIndexPosition_[i + 0]) * 11 + 1],meshPosition_[(meshIndexPosition_[i + 0]) * 11 + 2]);	
    b = vec3(meshPosition_[(meshIndexPosition_[i + 1]) * 11 + 0],meshPosition_[(meshIndexPosition_[i + 1]) * 11 + 1],meshPosition_[(meshIndexPosition_[i + 1]) * 11 + 2]);
    c = vec3(meshPosition_[(meshIndexPosition_[i + 2]) * 11 + 0],meshPosition_[(meshIndexPosition_[i + 2]) * 11 + 1],meshPosition_[(meshIndexPosition_[i + 2]) * 11 + 2]);
    
    a = vec3(worldMeshLocation * vec4(a,1.0f));
    b = vec3(worldMeshLocation * vec4(b,1.0f));
    c = vec3(worldMeshLocation * vec4(c,1.0f));
    
    vec3 ab = b - a;
    vec3 ac = c - a;
    
    vec2 hitCoords;
    bool n = IntersectTriangle(r, a, ab, ac, tmax, hitCoords);
    		
    if(n){		  
       hr.t = tmax;
       hr.p = rayPointAtParameter(r, tmax);
       hr.normal = vec4(cross(ab,ac),1.0f);
       hr.Color = vec4(hitCoords,1.0f,1.0f);
       hr.materialId = MaterialMetallic;
       return true;		   		  
    }								
  }       			
return false;
};
{% endhighlight %}	

The most reliable thing is that the process to find the different triangles is a bit of a mess because we need to index
the **GL_ARRAY_BUFFER** I am using to render the same model, and the **GL_ELEMENT_ARRAY_BUFFER**.
Following the struct, I am already using.

> I know that it would be better to use vec4, because of the alignment, but right now I am testing, and all this working is considered 
experimental.

{% highlight glsl linenos %}
//44 of offset, each 44 next element
struct Vertex{ 
	vec3 position; // Starts on 0  | 4 bytes * 3 = 12
	vec3 color;    // Starts on 12 | 4 bytes * 3 = 12
	vec3 normals;  // Starts on 24 | 4 bytes * 3 = 12
	vec2 uv;       // Starts on 36 | 4 bytes * 2 = 8
};
{% endhighlight %}	



{% highlight glsl linenos %}
layout(binding = 8) buffer meshPosition{
	float meshPosition_[];
};
layout(binding = 9) buffer meshIndexPosition{
	uint meshIndexPosition_[];
};
{% endhighlight %}	

That's why to access the first point of the triangle I'm doing this.

{% highlight c++ linenos %}
// x of the first triangle
meshPosition_[(meshIndexPosition_[i + 0]) * 11 + 0]
// x of the second triangle
meshPosition_[(meshIndexPosition_[i + 1]) * 11 + 0]
// x of the third triangle
meshPosition_[(meshIndexPosition_[i + 2]) * 11 + 0]
//for the y just replace + 0 to + 1
//and for the z just replace + 0 to + 2
//So we could access the next float element.
//Because the are consecutive in memory
{% endhighlight %}	

Being **meshPosition_** my Vertex struct, and **meshIndexPosition_** my indexed buffer.  

On the c++ side, this would be something quite easy, because of the indexed struct, that I have replicated.

{% highlight c++ linenos %}
//obj[i].position = getVertexData[index].pos;
float a = getVertexData[index].pos.x;
float b = getVertexData[index].pos.y;
float c = getVertexData[index].pos.z;
{% endhighlight %}	



The next thing I really need to know how to do is to generate and use AABB, because, a cube is simple, we have so few triangles to test, but the sphere, is a whole different thing, do not even think about any complex model right now.  

And once I had this, I really need a generic way of sending the whole data to the GPU, I was thinking about a struct, containing
the collection of all vertices, indexed, and all the relevant info I need.  

> But the problem is that I do not know if a can send 'X' number of buffers to the compute shader, because of a top limit, or because
slowing the whole process.  

What currently I am thinking is more like, gather all the vertex data I really need, and send only this triangle collection to GPU. But that will force me to make a new step each time a new geometry is loaded, to have this vector of points ready, extra memory will need to be sacrificed, but I think this is the first option I will try.

Maybe something like this on C++ side per each mesh:


{% highlight c++ linenos %}
struct RaytracingData{
	size_t amountOfVertices;
	glm::mat4 worldMatrix;
	std::vector<float> vertices;
	std::vector<float> AABBPoints;	
	//How many points this mesh have
	//The world matrix, to calculate correctly
	//The collection of points of this mesh
	//The Bounding Box of this single Mesh
}
{% endhighlight %}	

And something like this on glsl side.
{% highlight glsl linenos %}
layout(binding = X) buffer customData{
	float AllData_[];
};
//Being the first value, how many values until next value
//The next 4 * 4 floats, the world matrix
//From this point adding amountOfVertices
//we have all the vertices points
//From here now , adding 6 we got our min / max
//points, we will also have to multiply
//With each vertex, by the world matrix.
//the next geometry
{% endhighlight %}	


Also, I will need to consider if I should generate, Bounding Boxes between the same mesh data, because, I will need to have a Hierarchy of this.
Because I can avoid calculating a collection of 1.000.000 triangles, but if I hit its Bounding box, we all know, I am not going to hit all of them, so
narrowing the possible target triangle is always a must.


