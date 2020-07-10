---
name: Maze Solver
tags: ["Java", "JSwing"]
github_link: "https://github.com/mefchristiansen/Maze-Solver"
date: 2020-04-19
summary: "A visualized and interactive maze solver Java application that automatically generates a random maze, and then proceeds to traverse and solve the maze using a variety of graph traversal algorithms (BFS, DFS, A*)."
intro: "A visualized and interactive maze solver Java application that automatically generates a random maze, and then proceeds to traverse and solve the maze using a variety of graph traversal algorithms (BFS, DFS, A*). This generation and traversal is visualized using JSwing, and allows the user to interact with the maze, enabling them to choose the size of the maze, the maze start and endpoints, the solution algorithm, as well as vary the animation speed."
---

{% include image.html url="/_images/maze.gif" description="Generating and solving a maze" %}

# Intention

This project started with a desire to more fully understand, as well as code, a variety of graph traversal algorithms. Like every other CS student, I covered a multitude of graph traversal algorithms in my algorithms course, but I don't remember coding any of the algorithms. I wanted to actually code my own implementation of these algorithms, and through that more fully understand how they work, their benefits and drawbacks, and understand which algorithm is most suitable for different situations. After writing the algorithms, I then decided that I wanted to visualize them. I thought that a fun (and challenging) way to visualize them would be to build a maze solver (with a frontend view) where the algorithms would traverse and solve a maze. Essentially what I wanted to do was to animate the algorithms while they were running to visualize how they worked at a code level.

# Execution / Design Decisions

## Generator and Solvers

Two main components of this application are generators and solvers. Generators generate mazes, whereas solvers solve them. 

The generator that I decided to implement was a [Recursive Backtracker](https://en.wikipedia.org/wiki/Maze_generation_algorithm#Recursive_backtracker). A Recursive Backtracker is a DFS implementation that randomly picks an unvisited neighboring cell at each iteration to visit to construct the maze. After reaching a point where there are no unvisited cells neighboring the current cell, the algorithm backtracks along its path to find an unvisited neighbor. Once every cell has been visited, the algorithm terminates, and the maze has been constructed.

I implemented three algorithms to solve the mazes:
* BFS (Breadth-First Search)
* DFS (Depth-First Search)
* A*

[BFS](https://en.wikipedia.org/wiki/Breadth-first_search) traverses a graph by starting at the root node (in this case the starting cell) and explores all of the cells at the present depth before moving onto nodes at the next depth. This uses the opposite strategy compared to DFS. The algorithm uses a Queue to store the cells to be visited at each depth.

[DFS](https://en.wikipedia.org/wiki/Depth-first_search) traverses a graph by starting at the root node (in this case the starting cell) and explores as far as possible along a specific branch before backtracking. This uses the opposite strategy compared to BFS. If implemented iteratively, the algorithm uses a Stack to keep track of all the previously visited nodes which will pop off when backtracking.

[A\*](https://en.wikipedia.org/wiki/A*_search_algorithm) is graph traversal and pathfinding algorithm. A* aims to find the path to the endpoint with the smallest cost (in this case, the shortest distance). At each iteration, the algorithm chooses the path that minimizes a cost function: f(n) = g(n) + h(n), where g(n) is the cost to the current node from the start node and h(n) is the estimate of the remaining cost to the end node using a heuristic. In this case, the heuristic (i.e. estimated cost), is the Manhattan Distance between the current and end node. Unlike BFS and DFS, A* is an informed search algorithm, meaning that it knows where the end node is and uses this knowledge to make an informed guess as to which path is most efficient.

## Java / JSwing

I decided to write this project in Java (with a JSwing frontend) to challenge myself to learn new techniques in Java and to broaden my coding language capabilities (as I haven't used Java in a little while). This involved learning JSwing, a Java GUI toolkit, including learning how to build a GUI, handle user inputs, and how to update the GUI in response to those inputs. A large component of working with JSwing is working with events and event listeners. When a user interacts with the GUI, for example clicking a button or adjusting a slider, this creates an event object. Event listeners are registered to "listen" to these events and handle them as they occur.

One of the more challenging aspects of this project was thread management within the context of JSwing. In JSwing, any updates to GUI must happen on the Event Dispatch Thread (EDT), a special thread just for GUI updates. This is due to the fact that many Swing object's methods are not thread safe, and invoking them from different threads can cause issues. Handling an event and updating the GUI is done via the EDT, which makes calls to an application's event handlers. The design issue that this causes is that you should never block or delay the EDT, as this results in a non-responsive UI, as it is only the EDT that can handle events and update the UI. If it is blocked by a process, the UI won't be able to handle any other events while that process is running. A good GUI must always be responsive so that a user can interact with the GUI even though an underlying process may be running. This means that no long-running tasks can happen in the EDT, and instead need to happen in a separate thread. The main functionalities in this application are long-running processes (maze generation and solving) and needed to happen in a separate thread, but still needed to update the GUI intermittently.

I wanted to visualize the code as it was being executed. This was a problem however as GUI changes can only happen on the EDT. Fortunately, Swing provides a very useful method to get around this issue, the SwingWorker class. The SwingWorker class lets you send data chunks from a separate thread to a method that will run asynchronously on the EDT (when next available) where you can then update the GUI. This lets you run a long-running task on a separate thread but still, update the GUI periodically. I used this approach to run the processes of generating and solving the mazes but still be able to update the GUI while the processes were running. In my case, the data I sent was the state of the maze at each iteration during either generation or solving, so that the GUI would be updated accordingly.

## MVC

After deciding to create a view (and as the complexity and scale of the project grew), I decided to implement an MVC (Model, View, Controller) application structure. MVC is a software design pattern that separates the business logic from the UI logic. The business logic (or backend code) belongs in the model, the UI logic belongs in the view, and the controller acts as the main orchestrator, handling user inputs and facilitating the connection between the front and backend. I was comfortable with this architecture as I use Rails every day at Considdr, and Rails is MVC based. This design ultimately made the code much easier to maintain. The logical separation of the components reduced the complexity of individual classes and functions, and, due to the loose coupling nature of MVC, changes in code had a smaller area of effect and it was therefore easier to make changes without having to change a lot of code. The main difficulty that came along with using this pattern was the "gluing" of the front and back end and making sure that changes in the backend were reflected in the view.

# Future Work

There are plenty of extensions to this project, including but not limited to:
* Writing more maze generation algorithms, to be able to generate a variety of different mazes, and to explore how the different search algorithms would approach the different kinds of mazes.
* Writing more graph traversal algorithms (i.e. maze solvers), to expand my knowledge of these algorithms, to visualize how they all work, and to explore which algorithm is most suitable for different situations.
* Enable users to edit the maze (i.e. add or remove walls)