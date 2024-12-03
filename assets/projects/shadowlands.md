# Shadowlands: A Unity Desktop Game Showcasing AI Pathfinding with A Algorithm

![Shadowlands](assets/images/sl1.png)

Shadowlands is a desktop game created in Unity. The purpose of this project was to program the AI agent to find the shortest path between a starting and 
an ending point, using the A* algorithm.  
The map contains a forest in 3D graphics with roads, bridges,grass, rivers, lakes and rocks and is called 
Shadowlands.  

![Map](assets/images/sl4.png)

Our main character is a robot named Kyle, which is programmed to move using the A* algorithm.

![Kyle](assets/images/sl3.png)

The map contains several portals, and Kyle moves from one portal to another, 
trying to find the active portal in order to leave Shadowlands.
He already knows all the portal coordinates and tries them in order.

![Portal locations](assets/images/sl5.png)

Before planning his route he takes into account the obstacles, the starting and ending locations, 
and the movement costs of each different element on the map. 
The movement costs are predetermined so as to represent real life conditions.

![Credits](assets/images/sl2.png)

>Kyle uses quotes from Rick and Morty because... well why not?


This game is basically a short story and the user has no options in moving or selecting destinations.
However, for debugging purposes, I have added cheetcodes for overriding destinations and altering the default speed.
This allows the developer to interfere with the basic progression of our story using specific keys. 


You can watch a Demo bellow. 

<iframe 
  width="800" 
  height="450" 
  src="https://www.youtube.com/embed/gP_xa6cY_tU" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
  allowfullscreen>
</iframe>

