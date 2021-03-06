---
layout: page
title: Projects
permalink: /projects
---
Here you can find info about my pet projects.
You might be interested in something &#128512;.

![Portfolio](../assets/img/Portfolio.png)

<h3 align="center" id="engine">Elven Engine</h3>
*C++, OpenGL, ImGui, lia, cmake*

[Github repo](https://github.com/denyskryvytskyi/ElvenEngine)

Game engine development from scratch.
Main purpose is to learn more about game engine architecture, graphics programming and how things actually work under the hood.

I try to minimize third party libraries usage and develop all kind of subsystems by myself, including math lib, event system, renderer and other stuff.
##### Features:
- Renderer
    - Forward rendering
    - Orthographic/perspective camera
    - Material system
    - 2D sprite renderer
    - Shader manager (loading from files)
    - API abstraction layer to support different graphics api
- Event system with events queue
- Editor based on ImGui
- Custom math library
- Cmake configuring/building

![Elven engine](../assets/img/BatchRenderer.png)


<h3 align="center" id="lia">Lia</h3>
*Just C++*

[Github repo](https://github.com/denyskryvytskyi/lia)

Custom math library to use in my projects, including game engine (Elven Engine).
##### Features:
- Linear algebra:
    - n-dimensional vectors
    - 4x4-dimensional matrices
    - transformations (translations, rotation, scale)
    - utility (matrix transposition, inversion)
    - orthographic and perpective projections matrices
- Trigonemetry
    - radian/degree conversions.


<h3 align="center" id="bulletshot">Bulletshot</h3>
*C++, OpenGL, ImGui, lia*

[Github repo](https://github.com/denyskryvytskyi/Bulletshot)

2D multhithreaded simulation of simple aabb collision.
Reimplemented [Bullets](#bullets) project in OpenGL.
Also one of the purposes was to create small custom OpenGL 2D renderer.

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442368-d3297b43-6644-4074-9c87-64761ba3a0ac.mp4" type="video/mp4">
</video>

<h3 align="center" id="snake">Snake</h3>
*Just C++*

[Github repo](https://github.com/denyskryvytskyi/Snake)

Console snake game developed using some game programming patters and pure C++.

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442450-54785ec4-c6a4-45e9-ae93-47d4f28776b5.mp4" type="video/mp4">
</video>

<h3 align="center" id="bullets">Bullets</h3>
*C++, SFML*

[Github repo](https://github.com/denyskryvytskyi/Bullets)

Main purpose is to implement aabb collision and multithreading to optimize bullets creation.
My first attempt to use multithreading to optimize many objects process in one scene.

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442330-23624317-b263-4b30-b8bd-060281360152.mp4" type="video/mp4">
</video>

<h3 align="center" id="space">Project Space</h3>
*C#, Unity*

[Github repo](https://github.com/denyskryvytskyi/Project-Space)

Attampt to create more complex RPG-like game based on my Asteroids game.
Also I used MLAgents toolkit to create AI enemies through reinforcement learning.
Actually my first attempt to use AI for game engines.
You can look a little bit fake trailer for this game:

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163449028-ce7245ac-590e-4d24-a3cd-216ee6a4e772.mp4" type="video/mp4">
</video>

<h3 align="center" id="asteroids">Asteroids</h3>
*C#, Unity*

[Github repo](https://github.com/denyskryvytskyi/Asteroids)

Complete 2D game developed in Unity engine.

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442211-6a777966-f63b-4d9e-8ac4-8fafe926a475.mp4" type="video/mp4">
</video>

<h3 align="center" id="unity">Unity small games</h3>
*C#, Unity*

Small demo games where a learned how to use Unity, including 2D and 3D projects.
There are some of them:

[Cubetron](https://github.com/denyskryvytskyi/Cubetron)
[Ballgame](https://github.com/denyskryvytskyi/BallGame)

<h3 align="center" id="flowerfarm">Flower farm</h3>
*C++, Cocos2d-x*

[Github repo](https://github.com/denyskryvytskyi/Flower-farm)

2D farm game developed with C++ and Cocos2d-x framework.
Developed in 5 days without previous knowledge of cocos2dx library.
Also I've tried to use component oriented classes structure instead classic inheritance.

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442406-036cb803-de4e-4647-92d8-7eb7e3240bb7.mp4" type="video/mp4">
</video>

<h3 align="center" id="taskmanager">Task manager</h3>
*Java, Spring (MVC, Data, Security), Hibernate, PostgreSQL, Bootstrap*

[Github repo](https://github.com/denyskryvytskyi/TaskManager)

There are a bunch of different web pet projects developed in java and php.
But this one, Task Manager, is the most complete and favorite one.
It was developed using Spring Framework (MVC/Data/Security).
It has classic admin panel, login/logout with credential validation and of course tasks manager with add/edit/delete functions.

![Task manager main page](../assets/img/Taskmanager.png)

<h3 align="center" id="battleship">Battleship</h3>
*C++, SFML*

[Github repo](https://github.com/denyskryvytskyi/Battleship)

My first complete programming project and game developed in 2015 year.
I have been developed it by two months knowing only basic of C++ from Herbert Schildt book.
But code was completly refactored maybe 3 times since that and it looks more elegant for now:)

<video width="100%" height="100%" controls="controls">
  <source src="https://user-images.githubusercontent.com/25298585/163442276-ade2a688-689f-4cf4-9e08-e880599159a1.mp4" type="video/mp4">
</video>
