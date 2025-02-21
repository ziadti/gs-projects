# Google Sheets Projects

This is a repo with a collection of my formula only projects made entirely in Google Sheets.

## [3D Rubik's Cube Simulator](https://github.com/ziadti/gs-projects/blob/main/rubiks_cube_simulator.md)

An interactive 3D Rubik's cube simulator capable of simulating the basic face turns and slice moves with a reset button to reset the cube to its initial (solved) state. I have an [Excel version](https://onedrive.live.com/:x:/g/personal/956C2B4CF15D3B49/EQ0nzTycAYdJvFR-DOCw6TABSoDPTOKhgt3rRck_owBrgw?resid=956C2B4CF15D3B49!s3ccd270d019c4987bc547e0ce0b0e930&ithint=file%2Cxlsx&migratedtospo=true&redeem=aHR0cHM6Ly8xZHJ2Lm1zL3gvYy85NTZjMmI0Y2YxNWQzYjQ5L0VRMG56VHljQVlkSnZGUi1ET0N3NlRBQlNvRFBUT0toZ3QzclJja19vd0JyZ3c) capable of simulating all the [54 possible moves](https://solvethecube.com/notation), including direct double moves, wide moves and cube rotations (see 'main') but I'm too lazy to port it to Google Sheets.

<img src="https://i.imgur.com/4tnKD4i.gif" width="600">

## [Rubik's Cube Solver](https://github.com/ziadti/gs-projects/blob/main/rubiks_cube_solver.md)

A Rubik's cube solver that uses the [Old Pochmann method](https://ruwix.com/the-rubiks-cube/how-to-solve-the-rubiks-cube-blindfolded-tutorial/), a purely algorithmic intuitionless method that only solves one piece at a time designed for blindfolded solving. The lack of reliance on intuition makes this method easy to implement algorithmically with the drawback that the solutions result in a very long sequence of moves with an average that's over 300. (For reference, the [beginner's method](https://ruwix.com/the-rubiks-cube/how-to-solve-the-rubiks-cube-beginners-method/) has an aaverage number of moves of ~110, [CFOP](https://ruwix.com/the-rubiks-cube/advanced-cfop-fridrich/) has an average number of moves of ~55, [Roux](https://ruwix.com/the-rubiks-cube/different-rubiks-cube-solving-methods/roux-method/) has an average number of moves of ~45.)

<img src="https://i.sstatic.net/iR5Fk.gif" width="700">


## [Chess Board Editor](https://github.com/ziadti/gs-projects/blob/main/chessboard_editor.md)

A simple chess board editor that allows to move pieces across the board. It doesn’t validate the moves, it's just a basic drag-and-drop interface for setting up positions.

## [Universal Turing Machine (UTM)](https://github.com/ziadti/gs-projects/blob/main/utm_simulator.md)

A [Universal Turing Machine](https://en.wikipedia.org/wiki/Universal_Turing_machine) simulator that takes a list of transitions describing the behavior of a Turing machine, an input string, and up to three tapes, then processes it step by step. It allows stepping forward and backward, as well as resetting to the initial state. It also supports the wildcard symbol `*`, which can be used to match any symbol when reading from the tape or to write a symbol that was read in the current transition.

## [Random Access Machine (RAM)](https://github.com/ziadti/gs-projects/blob/main/ram_simulator.md)

A [RA-machine](https://en.wikipedia.org/wiki/Random-access_machine) simulator. It supports the instructions: 𝚁𝙴𝙰𝙳, 𝙻𝙾𝙰𝙳, 𝚂𝚃𝙾𝚁𝙴, 𝙰𝙳𝙳, 𝚂𝚄𝙱, 𝙼𝚄𝙻, 𝙳𝙸𝚅, 𝚆𝚁𝙸𝚃𝙴, 𝙹𝚄𝙼𝙿, 𝙹𝙶𝚃, 𝙹𝙶𝚉, 𝙹𝙻𝚃, 𝙹𝙻𝚉, 𝙷𝙰𝙻𝚃. It also supports direct, indirect and immediate addressing for the instructions for which it makes sense.



## [Light Beam Simulator (AoC 2023 Day 16)](https://github.com/ziadti/gs-projects/blob/main/light_beam_simulator.md)

Simulates a beam of light being reflected across a 20x20 grid. It's a simulation of the first part of the [16th puzzle](https://adventofcode.com/2023/day/16) of Advent of Code 2023.

## [Warehouse Robot (AoC 2024 Day 15)](https://github.com/ziadti/gs-projects/blob/main/warehouse_robot.md)

Simulates a robot moving across a grid, pushing boxes and running into obstacles. It's a simulation of the first part of the [15th puzzle](https://adventofcode.com/2024/day/15) of Advent of Code 2024.


<hr>

<em><sup>Shout out to <a href="https://docs.google.com/spreadsheets/d/1JoUVSSEYQUJrUvvszhElaD2T7PuZcC9GOU7uc3zp178/">Astral</a>, 
<a href="https://www.reddit.com/user/AdministrativeGift15/">Shay</a>, 
<a href="https://aliafriend.com/">Ali</a>, 
<a href="https://www.cooltables.online/">Max</a>, and 
<a href="https://www.reddit.com/user/mommasaidmommasaid/">Momma</a> for their insights.</sup></em>
