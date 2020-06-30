---
name: Conway's Game of Life
github_link: "https://github.com/mefchristiansen/Conways-Game-of-Life"
tags: ["Go", "Ebiten"]
date: 2020-06-22
summary: "Conway's Game of Life written in Go and visualized using Ebiten."
intro: "Conway's Game of Life written in Go and visualized using Ebiten. This game lets users interact with the board by enabling them to set the state of a cell and its neighboring cells to alive on a left mouse click."
---

{% include image.html url="/_images/cgol.gif" description="Conway's Game of Life" %}

I completed Golang's A Tour of Go (check out my solutions to this exercises [here](https://gist.github.com/mefchristiansen/c8b10dadf6f2ae20922bd3b283ee1941))

I also enabled the user to be able to interact with the board by being able to set the state of a cell and its neighboring cells to alive when left clicking the cell. This allows for some really interesting results.

{% include image.html url="/_images/cgol-interaction.gif" description="Setting cells to alive on a left mouse click" %}

{% highlight go %}
if alive && (numNeighbors == 2 || numNeighbors == 3) {
  // If a cell is alive and has 2 or 3 neighbors, it survives
  nextGeneration[row][col].state = true
} else if !alive && numNeighbors == 3 {
  // If a cell is dead and has 3 neighbors, it becomes alive
  nextGeneration[row][col].state = true
} else {
  // All other live cells die in the next generation.
  // Similarly, all other dead cells stay dead.
  nextGeneration[row][col].state = false
}
{% endhighlight %}