---
layout: post
title:  "Search for a star 2019 / FlagCapturer"
date:   2019-1-18
categories: 
---
## My Entry for Search for a star 2019




* [Main Idea](#main-idea)
* [Design Evolution](#design-evolution)   
* [Evolution of the game](#evolution-of-the-game)   
* [Work organization](#work-organization)   
* [Game feel](#game-feel)   
* [Input management](#input-management)   
* [Reflection](#reflection)   
* [Important Issues About the APP ](#important-issues-about-the-app )   
* [External Assets ](#external-assets)   
* [Gallery](#gallery)   



I have been busy these days, because of my last year project, and this Game I have been doing for the 
[searh for a star 2019](http://gradsingames.com/game-dev-challenges/search-for-a-star/), using Unity.

If you want to give it a try [click here](https://unyankee.itch.io/flag-capturer), but remember, it has one constraint,
you will need to play with someone, and with controllers. I hope it enjoys you playing, as much as I making it.

<iframe width="560" height="315" src="https://www.youtube.com/embed/WLGi92EjzQ4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>  



## Main Idea

The main idea was to reuse as much as I can of the framework, because I took this jam as a Real Development Environment, where I would have a given framework of piece of program, I will have to improve, iterating over and over, to make it fun to play, and create an enjoyable experience.
As I am not an artist and I cannot produce any kind of art, I limited my scenarios to Basic shapes that are built-in Unity, and the models we have from the framework, created using these basic shapes.
  
The Game Idea was focused on a fast game where two teams exist (Red and Blue), and each team needs to steal 5 flags, from the enemy base or fighting in the middle of the screen.
Each match will last 1 minute as much, with the possibility of a quick replay option, to keep playing. This is because short fast matches between friends can create extra emotions external at the game, that helps to enjoy the game the most.



## Design Evolution

The first Idea was a pretty simple, map with two bases, and just one flag on the middle, with some platforms.
This was because, at this moment, the player could only jump, and move horizontally, so is limited.

![My helpful screenshot](/assets/FlagCapturer/Design1.png)

This Main map was kind of awful, because the game was so slow, and to reach the flag, the player needs to struggle with the jump physics. So the first part was to replace the current player controller, with a custom player movement script.
Allowing me to code the exact movement I was looking, so, right now, the player is moved using a Rigid body, and adjusting the velocity of the component, depending on the input.
I mean, the player can choose how high can jump, like Mario games, and has a bit of levitation when the highest point is reached, like Yoshi’s jump. Now the player can jump as wanted.
The movement was replaced with a simple interpolation with the desired direction and the actual, so it creates this acceleration behavior. But still looks simple, so, I was tweaking and playing with rotations, to make the player rotate depending on the direction is trying to go.
Now the player is kind of funny just to see because it looked like a worm rolling to reach the flag.

So the map needed to be readapted to make use of all these new movement features.

Now the map has extra platforms to force the player to be jumping as much time as possible, and watching this worm rolling and jumping was funny to see.

![My helpful screenshot](/assets/FlagCapturer/Design2.png)

The next step is to modify the bullets, this time, the provided script is used almost entirely. Because I knew from the beginning, I wanted to have simple bullets, but that could ricochet on every single object.
So, the map will suffer extra modifications, to allow this feature to be funny as well, so some platform a needs to be added and rotated.

![My helpful screenshot](/assets/FlagCapturer/Design3.png)

This design allows the player to shoot and create a wall of bullets, avoiding the enemies to pick up the flag, but it has some unexpected behavior, to create this wall, you need to climb to the top first, what is not what I was expecting, so I decided to try another angle.

![My helpful screenshot](/assets/FlagCapturer/Design4.png)

And it worked perfectly, this new one was the one I was looking for, you do not even need to climb to annoy your enemies, instead of, you need to calculate when the enemy is going to be there and shoot from the ground.
With all tests, I was about build the last map, and the wan that was going to be the main map on the game, so We need high platforms, some separations between them, and rotated blocks to take profit of the ricochet bullet.

![My helpful screenshot](/assets/FlagCapturer/Design5.png)

This new layout was perfect, we can shoot from the ground, we can shoot from the highest platforms, and it has the separation I was looking for between block to force players to jump, but I realized that maybe I need an extra feature to make the movement faster and easy to move between places.
Here is where the rope takes place, a simple ray cast procedure, that moves a point on a given direction, and whenever detects a collision, it pulls the player to this new position, making the game faster.

![My helpful screenshot](/assets/FlagCapturer/Design6.png)


Now is time to play with the flags, with the movement almost completed.
I decided to be a bit tricky, so when 2 players are playing, only a flag spawns in the middle of the map, but, if there are more players, the flags will be on their bases, at the top of the map.
To avoid the players to be all the time stealing the flag and split the action, because every player shooting to the same point could be chaotic.

The last feature is the behavior of the camera, so this is a very important part, because it needs to be fast, but not too much, because it can cause dizziness, but not too slow, because it can occlude elements in the map.
So, following some tips from my last project, the best option was to be all the time Lerping positions, the same the players do, but with extra rotations, and some minimum and maximum values to respect height and depth.
So the camera is constantly Lerping between its current position and wherever needs to be positioned and looking, being one of the smoothest camera movement I have made.
The point the camera needs to be is always the average point between all the players on screen.



## Evolution of the game

The game has experienced a lot of changes during the development because of an exhaustive iteration of replaying different mechanics and asking people what they feel about a specific mechanic on the game. Getting a lot of useful feedback, that has altered the game completely.


## Work organization

The different tasks to complete has been done using pen and paper.
Writing what it needs to be done, what I am currently doing, and what needs to be tested to be considered as finished.
Like a Scrum table, but with one person.
So I could easily see the current status of the project just looking my board.



## Game feel

One of the most important parts of a game, it needs to be visually attractive and be able to tell the player what is happening but not annoying the screen or overflow with information on the screen. 
So, I decided to create a whole trail of flames, that surrounds the frame where the players are playing, whenever a point is scored, making use of a typical Coin received sound, while some fireworks are playing on the side a point has been scored.
The points are displayed on the screen all the time, on the top corners, but when a point is scored, a flame number appears, also to tell the punctuation to everyone without stop playing to look at the score table.
Also, the background of the map is affected by the score, being fully dark when no one scored a point and is colored with a custom shader, using a color gradient, and pulses of light, to display on the screen at all time, who is winning the match, but keeping playing.
This is a score of 5 red points, so is really easy to see that the red team is winning. 

![My helpful screenshot](/assets/FlagCapturer/Design7.png)  
 
![My helpful screenshot](/assets/FlagCapturer/Design8.png)


Every player whenever jumps it creates a simple particle effect on the floor, and produce a sound, to clarify that an action has been done.
Also, the flags, produce a sound, to make all the players realize that someone reaches a flag.
Bullets mark its collision creating some sparks when colliding with something.
And very player emits a sound when is killed.


## Input management

As each player is spawned by a GameMode, so there is no way of knowing any order of who is going to be each one. Each player Movement Script has some String variables, where its right input name is assigned by the GameMode.
That is, we got all the inputs ready, on the Unity Input Manager, like this ‘Fire1_p’ or ‘Horizontal_P’, so when the GameMode spawns a player, it gives an ID, that is appended at the end of a specific name. Since is a number range from 1 to 4, always is ok.


## Reflection

I had so few time to work on this project due to extra work for other assignments, but It was another fun experience to count, because it has been a long time since the last time I used Unity, and taking into account is a simulation of a real development environment, is an awesome opportunity to create a game based on a framework.
The most difficult part, it might be making the whole thing works together, because I decided to create a lot of scripts to encapsulate an specific behaviour, also the lack of keyboard control, because the kind of game it is right now, having a mouse on the screen all the time could be a distraction for the rest of the players using gamepads, so I decided to fully erase this controller mechanism.
One of the major errors was forgetting about setting Unity to work with non-binary files, because the repository size has suffered it.

## Important Issues About the APP 

There is a bug in this unity version, that mirrors just the RT axis, if some controllers are mirrored, the solution is to unplug them and re plug them, meanwhile the application is running, I still do not know why this is happening, but the report for this known issue is been open for at least 3 years.
If any error occurs, for an optimal experience, it is recommended to have only 2 controllers plugged to the machine, so it does not mirror any axis.



## External Assets

A list of all the external assets that has been used.

[https://www.fesliyanstudios.com/royalty-free-music/downloads-c/action-music/9](https://www.fesliyanstudios.com/royalty-free-music/downloads-c/action-music/9)  
[https://game-icons.net/lorc/originals/bottom-right-3d-arrow.html](https://game-icons.net/lorc/originals/bottom-right-3d-arrow.html) 
[https://game-icons.net/lorc/originals/supersonic-bullet.html](https://game-icons.net/lorc/originals/supersonic-bullet.html)  
[https://game-icons.net/delapouite/originals/crosshair.html](https://game-icons.net/delapouite/originals/crosshair.html)  
[https://game-icons.net/delapouite/originals/position-marker.html](https://game-icons.net/delapouite/originals/position-marker.html)  
[https://game-icons.net/lorc/originals/flying-flag.html](https://game-icons.net/lorc/originals/flying-flag.html)  
[https://freesound.org/people/toiletrolltube/sounds/184374/](https://freesound.org/people/toiletrolltube/sounds/184374/)  
[https://freesound.org/people/plasterbrain/sounds/399095/](https://freesound.org/people/plasterbrain/sounds/399095/)  
[https://freesound.org/people/NenadSimic/sounds/171756/](https://freesound.org/people/NenadSimic/sounds/171756/)  
[https://freesound.org/people/jomse/sounds/428650/](https://freesound.org/people/jomse/sounds/428650/)  
[https://freesound.org/people/pepingrillin/sounds/252167/](https://freesound.org/people/pepingrillin/sounds/252167/)  
[https://opengameart.org/content/smoke-platform](https://opengameart.org/content/smoke-platform)  
[https://opengameart.org/content/xbox-one-controller-icon](https://opengameart.org/content/xbox-one-controller-icon)  
[https://opengameart.org/content/xbox-one-controls-icons](https://opengameart.org/content/xbox-one-controls-icons)  
[https://www.reddit.com/r/gamedev/comments/1z0zid/free_keyboard_and_controllers_prompts_pack/](https://www.reddit.com/r/gamedev/comments/1z0zid/free_keyboard_and_controllers_prompts_pack/)  


## Gallery

![My helpful screenshot](/assets/FlagCapturer/Cover.png)  

![My helpful screenshot](/assets/FlagCapturer/Cover2.png)  

![My helpful screenshot](/assets/FlagCapturer/Cover3.png)  


![My helpful screenshot](/assets/FlagCapturer/Cover4.png)  

![My helpful screenshot](/assets/FlagCapturer/Cover5.png)  


![My helpful screenshot](/assets/FlagCapturer/Cover6.png)  


![My helpful screenshot](/assets/FlagCapturer/Cover8.png)  

![My helpful screenshot](/assets/FlagCapturer/Cover9.png)  



