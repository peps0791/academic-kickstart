---
title: Way Finder
subtitle: Building an indoor mapping solution using Nodejs and A-star algorithm. 
summary: Building an indoor mapping solution using Nodejs and A-star algorithm. 
date: 2019-02-02
math: true
diagram: true
markup: mmark
tags:
- NodeJS
- Javascript
- MongoDB
categories:
- Web
image:
  placement: 1
  caption: 'Image credit: [**Speedtest.net.in**](http://speedtest.net.in">)'
---

*“Necessity is the mother of all inventions.”*


I am navigationally challenged. My friends would vouch for that I imagine. After all, they have often found themselves in the predicament of trying to make me understand how to reach someplace. I can’t seem to grasp the concept of orientation. *"Where Am I"*? Or even worse, *"where am I with respect to this location"*? I don’t get it! The concept of *going north* from somewhere. So I follow what the Google navigator says. Unfortunately, it can’t be everywhere. On the other hand, I can get lost anywhere, everywhere.

Six years ago, when I was working at my first job, I would often find myself having a hard time in trying to find seats and cubicles.

*"Person A has been assigned to your issue. Good luck finding his seat!"*

It was a three-floor building with a staff size of around 500. We had a floor map, but that didn’t work for me. It never does. I needed something like Google maps. So I decided to build it.

## The Strategy

These were the components I had at my disposal:

1. Floor map
2. Employee list with seat number information

**The first question we have is how to formulate this problem statement?**

A navigation-based problem can easily be formulated as a graph-based problem, where each node is a possible source/destination seat and edges connecting the nodes are the paths. The goal is to find connecting edges between the source and destination nodes. Graph-based algorithms such as [Breadth First Search](https://en.wikipedia.org/wiki/Breadth-first_search), [Depth First Search](https://en.wikipedia.org/wiki/Depth-first_search) can be used to find a path from the source node to the destination node. However, these are not very efficient as they tend to search over the whole space without any "intelligence". To remedy this problem, algorithms such as [Greedy](https://en.wikipedia.org/wiki/Greedy_algorithm), [Dijkstra’s algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), and [A star search](https://en.wikipedia.org/wiki/A*_search_algorithm) use some sort of educated guessing(heuristics) to find the optimal path.


<figure>
    <img src="/assets/images/way-finder/map-graph.jpg">
    <figcaption style="margin-left: 30%;">Floor map to graph translation</figcaption>
</figure>


**The next problem is to solve for how to translate the available components in this form?**

"*How can the floor map be translated into a graph?"*

A grid comes to mind, where the dimensions of each square of the grid correspond to the dimensions of the seats in the floor map. Each square in the grid should correspond to a unit area in the map. It can either be a seat, walking space or blocked space. By doing that, we can convert our floor map into a graph.

There's one more problem to solve. **How do we add the grid to the floor map while keeping it feasible (or even convenient) for the office administrator to set up the map? How do we set up the UX workflow?**

We can create the grid overlayed on top of the uploaded floor map, and the admin user can mark the seats and hallways by clicking on them. Unmarked squares become blocked by default.

<figure>
    <img src="/assets/images/way-finder/grid.png">
    <figcaption style="margin-left: 30%;">Grid overlayed on top of floor map image</figcaption>
</figure>

<figure>
    <img src="/assets/images/way-finder/markseat.gif">
    <figcaption>Initial approach for marking seats and hallways. Time consuming and tiring! Definite improvements needed</figcaption>
</figure>


Neat! Except, there’s one more problem. It takes too much time to complete the marking process. Two things come to mind:

1. Hallways are mostly symmetrical (and straight lined). Unless you happen to work at one of those hippie-dippie places with zig-zag hallways. So instead of manually marking all the squares representing a hallway, we can just specify the endpoints and mark all the points lying between these two nodes automatically.

<figure>
    <img src="/assets/images/way-finder/assistant.gif">
    <figcaption style="margin-left: 20%;">Marking hallways by specifying the end-point coordinates.</figcaption>
</figure>

2. For those hippie-dippie offices, the hallway squares can be marked using the “drag-and-mark” approach. 

<figure>
    <img src="/assets/images/way-finder/markseat-fast.gif">
    <figcaption style="margin-left: 30%;">Drag-and-Mark approach for marking hallways</figcaption>
</figure>

And that’s it! So how did I set up the whole app?

I created the web app using NodeJS framework and MongoDB database. Nothing fancy for front-end. Just plain HTML and Jquery.

There are two modes in the app:

1. Admin Mode
2. Employee Mode

# Admin Mode

The admin mode is for setting up the map. Additional operations such as renaming map, deleting map can also be performed. It has the **Lane Marking Assistance section** which contains the first option I talked about earlier for lane marking. The second option, *"drag-and-mark"* is enabled by default. The admin mode also has a dashboard with the list of employees, searchable by employee information (name, team, phone, seat number etc).

<figure>
    <img src="/assets/images/way-finder/dashboard.png">
    <figcaption style="margin-left: 30%;">Admin Dashboard</figcaption>
</figure>


# Employee Mode

Once the map has been set up, this mode can be used to look up other employees using seat number, and get the navigation path between the source and destination seats. 

# Search

When the search is made for a destination seat:

1. Floor information for the target employee is fetched from the database to search the appropriate map.
2. List of all the marked hallways and seats for the floor are fetched from the database.
3. The map image is sent to the frontend along with the list of marked nodes.
4. A grid is created on top of the map.
5. Once the graph is created for the map, I use this [A star JS implementation](https://github.com/bgrins/javascript-astar/tree/0.0.1) on the front end to calculate the path between the nodes.
6. Once the path is calculated, it's highlighted in yellow.

<figure>
    <img src="/assets/images/way-finder/search.gif">
    <figcaption style="margin-left: 30%;">Search Workflow</figcaption>
</figure>


That works out all right! There can be a few improvements made to this approach:

1. Automating the marking process, even further, using the floor map pdf properties to detect lanes and seats.
2. Integrating it with GPS-based navigation.

Those are certainly interesting problems, well worth returning to! Will give them a shot soon. Until then, adios! 


P.S - Here's the link to the [Github Repo](https://github.com/peps0791/way-finder).