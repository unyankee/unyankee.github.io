---
layout: post
title:  "OutterSpace"
date:   2018-12-11
categories: 
date: 2018-07-10 
---
## WIP

## New Begining

This is my new big project, developing my own framework to be able to work, or test any new feature or simply test 
whatever I'd like ot test.


This new framework is currently capable of several things, and it is getting bigger, because I really want to test everything I could.

But keeping in mind one simple thing, it must be simple to use,not just for me, but everyone who wanted to use it.
At the same time needs to be flexible, because is a framework, but I really would like to keep it updated, so I could keep using it
in the future, to keep testing everything that needs to be tested.

So, two modes of working are avaliable, the most simple one, for just test how it works, for example, creating a simple GameObject to be rendered.

For that task we need several components, to let the system know everything is okey, and can be used.
{% highlight ruby linenos %}
//Main class, it could be a 'Node'.
GameObject* testingCube = GameObjectSystem::get().createGameObject();
//It needs to let the system know this entity can be rendered
testingCube->addComponent<MeshRenderer>();
//The system needs to know the mesh it has, is ok, and exists
testingCube->addComponent<MeshFilter>();
//A material to render the GameObject, so we could see it
testingCube->addComponent<Material>();
//A representation in the world
testingCube->addComponent<Transform>();
{% endhighlight %}

Then we need a mesh, to deal with it, the framework provides a simple function to load any wavefront/ obj.

{% highlight ruby linenos %}
Mesh* newMesh = MeshSystem::get().generateBasicMesh("../../../assets/Cube.obj", "../../../assets");
//Pass this mesh to our mesh filter, to apply it
testingCube->getComponent<MeshFilter>()->appliedNewMesh(newMesh);
{% endhighlight %}	

We just need to init out material, and everything would be correctly configured.
{% highlight ruby linenos %}
testingCube->getComponent<Material>()->init();
testingCube->getComponent<Material>()->shader_->setNewVertexShader(readFile("../../../Engine/shaders/GLSL/SimpleVertex.vs", "r"));
testingCube->getComponent<Material>()->shader_->setNewFragmentShader(readFile("../../../Engine/shaders/GLSL/SimplePixel.ps", "r"));
{% endhighlight %}	

* The framework creates components, using the addComponent<>() function
* Any component can be accesed by calling the getComponent<>() function
* And lastly, any component can be cloned using the setComponent<>() function


But if more flexible is needed, the buffer can be completely customized using a HighBuffer instance, to generate a custom mesh.
Thats it, for example:

{% highlight ruby linenos %}
// A custom buffer to keep tracking of the mesh information
HighBuffer* buffNoInst = BufferSystem::get().createNewBuffer();
//An empty mesh, to fill with custom data
Mesh* m2 = MeshSystem::get().createMesh();
//Couple of vectors to handle with the custom vertex layout this frameworkds operate with
//Real position of the vertices
std::vector<Vertex> vertices;
//Indexes of the vertices, avoiding to upload repeated vertices to GPU
std::vector<uint32_t> indices;
//Ask the system to fill our vertices / indices vectors
MeshSystem::get().loadOBJ("../../../assets/test.obj", "../../../assets", vertices, indices);
// Create a cutom order to send this information layout, using the data we want
std::vector<DataOrder> order2;
//we want a position layout
//using StaticDraw
//with a given offset
//it fits the size of a Vertex struct
//0 of stride, the next element is right the next one
//There are 3 per group
//On the shader, is the layout 0
//There are vertices.size()
order2.push_back(DataOrder(DataOrderEnum::kDataOrderEnum_Position, DataUsage::kDataUsage_StaticDraw,
	offsetof(Vertex, pos), sizeof(Vertex), 0, 3, 0, (uint32)vertices.size()));
order2.push_back(DataOrder(DataOrderEnum::kDataOrderEnum_UVs, DataUsage::kDataUsage_StaticDraw,
	offsetof(Vertex, texCoord), sizeof(Vertex), 0, 2, 1, (uint32)vertices.size()));
order2.push_back(DataOrder(DataOrderEnum::kDataOrderEnum_Indexes, DataUsage::kDataUsage_StaticDraw,
	0, sizeof(uint32_t), 0, 0, 0, (uint32)indices.size()));

buffNoInst->provideData(vertices, indices);
buffNoInst->provideDataOrder(order2);

m2->addSubmeshes(buffNoInst);

{% endhighlight %}	

It can load any obj geometry. Right now is using a simple material that displays the UV's.

![My helpful screenshot](/assets/OutterSpace/Screenshot_1.png)

As one of the things I am experimenting with is how to handle with a lot of entities, of course that is supports instancing. 
Here an example, of a 1000 dynamic entities, being updated and rendered at the same time.

Still an early version, but kinda good results. 1000 asteroids at about 1600fps !!

![My helpful screenshot](/assets/OutterSpace/Screenshot_.png)

This instancing method, and compute shaders, are used to manage simple particle systems, like this 50.000 particles orbiting around a point, and 
bouncing when they hit each other.

![My helpful screenshot](/assets/OutterSpace/Screenshot_2.png)




I have been testing raytracing, my first approach thanks to Peter Shirley books(Raytracing In One Weekend, the next weekend, and the rest of your life).

![My helpful screenshot](/assets/OutterSpace/output.bmp)

An offline raytracing renderer, heavily multithreading, that can display the image that is being currently calculated, as well as
measure the time that it takes, to be able to have some measures to be able to optimize.


Right now is being used for my final year project, so I am experimenting with realtime raytracing.
But until now, I got this, shared resources between CPU-GPU, to be able to render at real time a completely raytraced scene, using compute shaders again.
The next step it would be to complete the second tutorial of Peter Shirley, but completely on the GPU.

![My helpful screenshot](/assets/OutterSpace/Screenshot_3.png)
