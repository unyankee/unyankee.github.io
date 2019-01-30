---
layout: post
title:  "DX11-Animations"
date:   2018-11-18
categories: 
---
## My first Animation System on Dx11

<iframe width="560" height="315" src="https://www.youtube.com/embed/bAM5TKkDaoY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


![My helpful screenshot](/assets/Animations/Screenshot_1.png)





#### Hierarchical class.


![My helpful screenshot](/assets/Animations/Screenshot_4.png)


![My helpful screenshot](/assets/Animations/Screenshot_5.png)

This new class is called Node, it represents a container of components, and it has another important class, the Transform, to deal with location, rotation, scale and some extra features such us specific camera location and rotation to look at a given component.  

The component class is called Object, it deals with what mesh needs to be rendered and it has its own local transform, to allow local transformations.  

The transform update had been implemented using the flat tree model, that is, we have four vectors of different data types. One is for the component that needs a transform, one for its local matrix, one for its world matrix and the last one is a vector of parent indexes. Used to update correctly all objects.
I have implemented this model because having the data I need consecutively in memory allows me to update all the transforms using a simple for loop, so I can avoid recursive functions, that are useful, but I considered this approach is more stable and easier to read.  
 
As Noel said in its article “Data-Oriented Design now and in the future”: (Noel Llopis ), “Break Up and Batch”.
 The update of the transform is designed following this idea, first we need to update all our data, but in groups, we prepare all the data we need, for every instance, and then, again for every instance we update using what we prepare.
Even doing two full passes over, would not be worse from memory point, because in modern hardware platforms, this way of working is more cache friendly. (Noel Llopis )
The next step would be adding the real system, that deals with all the components of the node class.



#### Keyframe animation system.

![My helpful screenshot](/assets/Animations/Screenshot_6.png)

Animation class: The most important class, its Update function will calculate the corresponding location and rotation, for a given animation, on a given time.
It would be better having each bone completely separated, so I could update separated bones for different animations, that way I could add extra features such us specific bones blending, or more complex animation blending.
Animation Controller class: It is just an abstraction for the node if it has an animation controller it can manage animations and update them. Also, is responsible for an update any ‘blending to’ animation if it has one.  

Collada Parser class: This class deals with all the parse functions to be able to grab the animation data from a .dae fail, using tinyXml2 to make it easier and a great Collada explanation from (COLLADA; Sailing the Gulf of 3D Digital Content Creation/a).   

As Jason Gregory suggested, each clip has its own local clock, represented by a floating point number, starting from zero when the animation starts to play and scaled using a playback rate variable, I like to call ‘Dilated Time’ (Gregory ). Allowing us to manage the play rate of each component individually, even play reverse by setting this dilated time to -1. 


#### Animation blending.

![My helpful screenshot](/assets/Animations/Screenshot_7.png)

The animation blending has been added using the animation controller since is just a simple blending between 2 animations, and we are not using 3 or more animations or more complex systems like blending different poses on specific bones, I considered it could be more profitable for this task, to make it as simple as I could.  

All nodes playing animations, that means with any animation controllers, can receive a target animation used to blend to, with a given time to blend.
Once this happens, the animation controller will start to blend calculating the right position for both animations, and tween between both. And when it finishes it will replace the main animation using the ‘blend’ animation.

Blending two different animations means, that for each animation the right position, the rotation needs to be calculated, that means, find the previous and next frame, for the time this animation is. Then interpolate with an alpha, calculated using the time it has been playing. And repeat this process but with the ‘blending to’ animation. Finally, we can repeat once more to find the in-between pose, of these two animations.  
 
(Gregory )Time-scaling a clip makes it appear to play back more quickly or more slowly than originally animated. To accomplish this, we simply scale the image of the clip when is laid down onto the global timeline.
My implementation is quite orthopedic, considering that is heavy to process, it could be more efficient, by calculating the current keyframe, and storing the next one, instead of recalculating the keyframe for each animation, for each frame.


#### Something new.

I have added the ability to load Collada geometries, parsing the same .dae file, if there is, as an example the executable it is loading the mixamo skeleton.  
Also exist a very simple geometry manager, because if a geometry is loaded, we only want to load it once, and reuse it for every instance.  
I have also added ImGui UI to manage all the robots on the scene, and make it easier to control each blending animation, or just see what each robot is doing,
A simple FSM for the robots to manage its behavior for the assessment.  
Also, have added an extra state for the robots, once you hit one with a bullet, it will blend to the death animation, and within 5 seconds will blend to idle.  
To detect if a bullet hit a robot, I used a bounding box structure, that is, the local bounding box generated by the CommonMesh class we had, updated to world space, so I can easily check if a bullet is inside any bounding box.  

![My helpful screenshot](/assets/Animations/Screenshot_8.png)

Also, have added a Collada exporter and an animation editing mode, that can be used to add keyframes, to update keyframes, and with a given path it will generate a Collada file and export the animation you have just modified, ready to be read and played on the same executable. 
Just by clicking on the [] can be easily moved, is the time for this data.
Once any bracket is clicked, its information it will appear, so it could be modified.

![My helpful screenshot](/assets/Animations/Screenshot_9.png)


## Discussion

My animation system can be used easily, it works using different animations easily and can load from any Collada file, but it has some constraints.  
For example, you can just load Collada files that respect a determined .dae format, I mean, it can not load a .dae file that contains the matrix transformation. It will only work using the format we were supplied, that is rotations must be completely separated, one for each axis, it does not matter the number of times/keyframes. Also, any file animation generated by this executable will respect this order structure, so if you want to export to any other program, it will need to know in advance how to deal with it.  
With all that said, this animation system can load any animation that respects this hierarchy, so any other model can be easily loaded if the geometry is separated, because it will also parse the hierarchy of the skeleton and apply it to the animated model, if the bone has the exact same name on the file used to load it and on the hierarchy Collada file
In the other hand, as is updating all the animations each frame, it can be computational heavy having a lot of animations at the same time, but for now, on my testing computer, it runs 900 animations at 30fps, being 625 the amount to start frame dropping(45-50fps) on release configuration. One possible fix would be starting the whole hierarchy again, by following the Component Oriented Design Mike Acton proposed (Mike Acton ).   
Having a system that updates every single component by separate (executers), following a determined order, and each component having the relevant information they need, just containers.  
So, if the model is prepared in advance, and is following the same structure the robot is, it can be used with completely no problems, it will generate the skeleton hierarchy, and prepare the animation data for its use. But if there is any other structure, this system becomes completely useless, because it will fail at loading any animation, and there will not be any animation playing on the screen.  



## References.
Works Cited
COLLADA; Sailing the Gulf of 3D Digital Content Creation. 30 Vol. Portland: Ringgold Inc, 2006. Web. 
Gregory, Jason. Game Engine Architecture. Boca Raton: Taylor & Francis, CRC Press, 2019. Web. 
Unity at GDC - A Data Oriented Approach to using Component Systems. Dir. Mike Acton. Perf. Mike Acton. Unity, 2018. Youtube.
Noel Llopis. "Data-Oriented Design Now and in the Future." (2011)Web.
 





