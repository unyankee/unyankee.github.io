---
layout: post
title:  "Procedural City"
categories: 
description:  "First time using OpenGL"
date: 2017-06-10 
image: /assets/City/Screenshot_1.png
---


## Procedural City

In fact, there is not much to say about this project, was the first time I had to deal with OpenGL, so it was an interesting activity.
We had to reimplement several parts of a framework based on OpenGL before working on the real thing, so I guessed the idea was to 
take practice using this API, before anything.



![My helpful screenshot](/assets/City/Screenshot_1.png)



The city is generated following a Cellular automaton, its task is to set specific values on a 2D grid.
This grid will be used then to smooth or readjust the values, reconfiguring the roads positions, or applying trees where there 
is nothing special set.
The closer the center is, the higher the building is, and the probability of appearing a park structure increases.


![My helpful screenshot](/assets/City/Screenshot_2.png)

![My helpful screenshot](/assets/City/Screenshot_3.png)


