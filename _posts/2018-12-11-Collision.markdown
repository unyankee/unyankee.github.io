---
layout: post
title:  "Physics"
description:  "Collision Demo"
date:   2019-2-18
categories: 
image: /assets/Collision/Screenshot_1.png
---

All references links are still pending to add. 

<iframe width="100%" height="315" src="https://www.youtube.com/embed/EDhVRpYAZzA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Used CPU: Ryzen 7 2700x


## Pre-calculating potential expensive operations.

An specific struct  has been added to store all the possible expensive values such as all the normals of the triangles, to avoid calculate them on each iteration.  
Also, when the heightmap is loaded, a Bounding Volume Hierarchy is generated, to speed up the triangle detection with all the spheres. It contains all the triangles of the heightmap, and, as it is static, is generated once, and prepared using a specific order to make the checking as fast as possible.  

The order is a depth-first search order, but per each node, add the next node in case checking the current one, results in a fail, and create a vector with all this information.  

![My helpful screenshot](/assets/Collision/Screenshot_2.png)

This way, if we test the node number 1 and its checking is right, if we increment the variable used as an index, we access the number 2, but if node number 1 fails, we can access directly to the number 4. That way, all the information is contiguous in memory, and any single jump with pointers is performed.

The iteration method is a while loop with no conditions, increment an index in case we hit a right value, and ask for the next one if we fail if the current index is 0xffffffff we break the whole tree and exit.  

![My helpful screenshot](/assets/Collision/Screenshot_3.png)


The same patterns have been used to speed up the sphere against sphere collision check, regenerating a BVH for all the spheres being updated, but it takes more time to use the BVH hierarchy for this purpose, if all the spheres are close to each other’s, only works if the geometry is split up around the world, because it will discard unnecessary collisions faster. Even adding the cost of regenerating the tree and transforming it into a vector.  

Another algorithm, using a stack to check collisions is also provided, but as it is slower, is not been used, and the default enabled is the brute force method, checking every single sphere against the rest of them.


All this was pre-computed following some raytracing optimizations, as said in (Wald, Boulos, and Shirley es) paper, considering the percentage of geometry that is completely static, can be completely prepared just once on load time, and reused each needed time.  

The iteration algorithm is inspired by the (Laine,Samuli), on the paper “Restart Trail for stackless BVH traversal”, where a stackless algorithm is proposed, and the main ideas are implemented.  

The main collision algorithms that are used, are from the book “Realtime Collision Detection” of (Ericson ), including sphere triangle, and capsule sphere.  

Also, the BVH construction algorithm is also inspired by the (Wald et al. es) BVH explanation on the paper “Ray Tracing Deformable Scenes using Dynamic Bounding Volume Hierarchies.”  

## Simple Class Structure




The main class is called SimpleSphere, it represents what it might be called GameObject.  

Is the Node that joins all the components that have been created for this demo purpose, following the Entity/System/Component structure. Meaning we have containers of data, all the systems created, and the executors of this data, the classes that these systems contain.  

![My helpful screenshot](/assets/Collision/Screenshot_4.png)

All systems have a templated parent class created to make a new system easier, and all the components that contain derive from the same class, so all of them have a proper ID variable and access to the owner of themselves. Following the design proposed by (Mike Acton ). All the systems deal with each update for their components.  

![My helpful screenshot](/assets/Collision/Screenshot_5.png)

##### Transform System and Transform: 
Deals with all the information and operations regarding position, rotation, and scaling. Although the only scale and position are being implemented in the collision system.  
##### Physics System and Physics: 
Deals with all the collision detection and collision resolution methods, following a pipeline, step by step order.
##### Collider System and Collider:
Represents the collider of each entity, and is the component that adjusts the position, scale and rotation of all the AABB used, and the Capsule information used.  

A couple of custom classes that act like helpers have been also created, just to deal with all the BVH information, that way we also have **BasicBVHNode** that deals with all the pointers of the tree, and the class used in the vector called GPUBasicBVHNode, making sure is rightly padded.  

The default functions for dealing with the ray-triangle, have been replaced. In the case the sphere is moving below a certain speed,it makes use of a more simple triangle – Sphere detection.
So the heightmap class allow us to choose with comparison method used. (ray - triangle or Sphere - triangle)
Also, some helper classes to store the collision impact data has been used.

## Physics System.

The main task, it is almost performed on the Physics System class, and on the Physics class. 
As said early, it follows a step by step division, that means, instead of detecting and solving possible collisions, one by one, All the work is divided into 4 steps:

1.	Checking for possible collisions:  
	*	Here the order does not really matter, as we are not updating any variable like position, radius… We are checking if any sphere is colliding with the heightmap and storing the data of the collision.  
	*	Then we perform the same operation, but with all the spheres.    

2.	All possible collisions are resolved, first spheres against spheres, and then spheres against heightmap.  

3.	All physics objects are updated, meaning that all positions are calculated using a simple Uniform Accelerated Rectilinear Movement equation.  

4.	Finally, we perform a test to solve the penetration of the spheres and avoid the sinking effect. Any sphere overlapping the plane or another 
sphere. Overwriting the last position the sphere had.		


For the collision detection we are using the BVH of the heightmap for the sphere against the plane, and depending on the selected option, the brute force or the BVH of the spheres, to solve the sphere against sphere.  
The collision resolution is performed as precise as possible, meaning we are using the radius of the sphere, mass of each object, friction, and Coefficient of Restitution.  

![My helpful screenshot](/assets/Collision/Screenshot_6.png)

The Updating step is performed using this formula, being V the current velocity of the sphere, T the delta time multiplied by the dilated time, A the acceleration of the sphere.  

The main problem is how to avoid the sphere going through another sphere or the heightmap, because of the speed, to avoid this, the capsule against AABB and capsule against capsule have been implemented, meaning that if any sphere is going fast, its collision method detection is going to be replaced, depending on the velocity of each object.

Currently supports: 
* A slow sphere moving towards a fast-moving sphere.
* Both spheres moving slow. 
* Slow Sphere against triangle of the heightmap. 
* A fast sphere against the plane.



When the sphere is going at a high speed, and it needs the capsule to detect collisions, one AABB is generated containing the capsule, and is sent to the heightmap BVH to be tested, if any triangle success, a ray is sent, following the segment of the capsule, and is checked the impact point with the radius of the capsule, resulting or not on a collision.  

As  (Szauer ) stated in the book “Game Physics Cookbook”, the first thing we do is find the relative velocity between the two rigid bodies. If the rigid bodies are moving apart from each other, we stop the function, as no collision occurs.
This is performed using a simple dot product between the normal of the impact, being the vector obtained with the positions of both spheres, and the relative velocity of the spheres. Pointing to the same direction.  

Then (Szauer ) propose a method to calculate the magnitude of the impulse vector, to resolve the collision. Using the minimum coefficient of restitution of both spheres. And apply it to both spheres, one being an addition, and the other one the opposite vector.   
After applying the linear impulse, (Szauer )propose to implement a friction calculation, as said on its book “Game Physics Cookbook”, To apply friction, we first find a vector tangential to the collision normal, once the tangential vector is found, we have to find ‘it’, the magnitude of the friction we are applying to this collision. We need to clamp the magnitude between ‘-j * friction’ and ‘j * friction’. This property is called Coulomb’s Law.  

With this vector we can overwrite the velocity we already write to each sphere, again adding it to our velocity vector.  
The same method is used to solve the collision between the plane and the sphere, but assuming the heightmap is static, its velocity is zero, and it has infinite mass, its mass is zero.  

Also, is important to consider, that every step is parallelized. What means that for example, when we are calculating all possible collisions, we are reading information, but not writting on the used Physics component, what means that we can perform the detection between spheres and the plane, and the spheres against other spheres, at the same time.
That way, if processing the plane takes 1 millisecond, and processing the spheres 2 milliseconds, we are not waiting 3 milliseconds, we saved 1 millisecond, and spent 2 calculating everything.
Another key point, is that the collision resolution between spheres and the plane is stored on a vector of possible hitresults, so this step can be also performed across all possible threads, what allows this demo to easily handle 4000 spheres at the same time, and colliding all of them against the plane. Making use of the BVH iteration across all threads again, as is static can also increase the perfomance.

  

## Works Cited  

Ericson, Christer. Real-Time Collision Detection. 1st ed. CRC Press, 2004. Web.   

Unity at GDC - A Data-Oriented Approach to using Component Systems. Dir. Mike Acton. Perf. Mike Acton. Unity, 2018. Youtube.  

Szauer, Gabor. Game Physics Cookbook. 1st ed. Packt Publishing, 2017. Web.   

Wald, Ingo, Solomon Boulos, and Peter Shirley. "Ray Tracing Deformable Scenes using Dynamic Bounding Volume Hierarchies." ACM Transactions on Graphics (TOG) 26.1 (2007): es. Web.  

Samuli Laine, NVIDIA Research, “Restart Trail for stackless BVH traversal”, Web  




