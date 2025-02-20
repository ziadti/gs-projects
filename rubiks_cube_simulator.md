# [Rubik's Cube Simulator](https://docs.google.com/spreadsheets/d/1znNMAntjSUPOPFdG3pLMWOUQ6GhkHyLCTgPaLriuYrA/)

## Demo

<img src="https://i.imgur.com/4tnKD4i.gif" width="600">

## How I made it

This project was divided in mainly three parts.

First, I made a 2D representation of the cube and manually created a lookup table with the sequence of swaps associated with each movement

![2D Cube](https://i.imgur.com/rQMJXb4.png)

![Swaps](https://i.imgur.com/BQxOmmK.png)

which I then used in the formula.

Then, I created the actual 3D cube that mirrors the calculated values from the previous step. This part involved a lot of manual work since I was handling each pixel individually. I used these conditional formatting rules to color each pixel based on its calculated value.

Finally, I made the interface to interact with the cube which is just a set of checkboxes associated with each of the 54 possible moves + the reset button (RST) to reset the cube to its initial state. I used iterative calculation to store the history of the movements.

