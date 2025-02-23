# [Rubik's Cube Simulator](https://docs.google.com/spreadsheets/d/1znNMAntjSUPOPFdG3pLMWOUQ6GhkHyLCTgPaLriuYrA/)

An interactive 3D Rubik's cube simulator capable of simulating the basic face turns and slice moves with a reset button to reset the cube to its initial (solved) state. I have an [Excel version](https://onedrive.live.com/:x:/g/personal/956C2B4CF15D3B49/EQ0nzTycAYdJvFR-DOCw6TABSoDPTOKhgt3rRck_owBrgw?resid=956C2B4CF15D3B49!s3ccd270d019c4987bc547e0ce0b0e930&ithint=file%2Cxlsx&migratedtospo=true&redeem=aHR0cHM6Ly8xZHJ2Lm1zL3gvYy85NTZjMmI0Y2YxNWQzYjQ5L0VRMG56VHljQVlkSnZGUi1ET0N3NlRBQlNvRFBUT0toZ3QzclJja19vd0JyZ3c) capable of simulating all the [54 possible moves](https://solvethecube.com/notation), including direct double turns, wide turns and cube rotations but I'm too lazy to port it to Google Sheets.

## Demo


https://github.com/user-attachments/assets/5dbb55f2-1963-4a73-b65d-110c2837185f


## How I made it

This project was divided in mainly three parts.

First, I made a 2D representation of the cube and manually created a lookup table with the sequence of swaps associated with each movement

![2D cube and swaps](https://i.imgur.com/8igMFb6.png)

I used the lookup table and the 2D cube in the main formula that updates the state of the cube based on the given moves

```py
=ARRAYFORMULA(LET(
   ref, D1:O9,
   SLOOKUP, LAMBDA(keys, lookup, result, MAP(keys, LAMBDA(key, FILTER(result, EXACT(key, lookup))))),
   UPDATE_CUBE, LAMBDA(cube, move,
     REDUCE(cube, SPLIT(move,", "), LAMBDA(cube, move, LET(
       moves_, WRAPROWS(SPLIT(SLOOKUP(move, A1:A54, B1:B54), "/ "), 2),
       moves, VLOOKUP(moves_, {TOCOL(ref), TOCOL(cube)}, 2, 0),
       IFERROR(1 /
         SUBSTITUTE(IFERROR(
           REDUCE(cube, SEQUENCE(ROWS(moves)), LAMBDA(cube, i,
             IF(cube = INDEX(moves,i,1), INDEX(moves,i,2) & "#", cube))
           ), cube), "#", )^-1
       )
     )))
   ),
   UPDATE_CUBE(ref, D11)
 ))
```

Then, I created the actual 3D cube that mirrors the calculated values from the previous step. This part involved a lot of manual work since I was handling each pixel individually. I used the following conditional formatting rules to color each pixel based on its calculated value

<img src="https://i.imgur.com/rKoM3Dt.png" height="500">

Finally, I made the interface to interact with the cube which is just a set of checkboxes associated with each of the possible moves + the reset button to reset the cube to its initial state. I used iterative calculation to store the history of the movements.

This is the formula I used in this step:

```py
=LET(
   triggers, {cube!BS74; cube!AN29; cube!CH29; cube!BI74; cube!AI38; cube!CM38; cube!BF21; cube!CI65; cube!AL65;
              cube!BN21; cube!CM58; cube!AI58; cube!AX70; cube!BW24; cube!CM49; cube!AI49; cube!CA70; cube!AW25; cube!E100},
   triggers_str, {"F"; "F'"; "R"; "R'"; "U"; "U'"; "B"; "B'"; "L"; "L'"; "D"; "D'"; "M"; "M'"; "E"; "E'"; "S"; "S'"; "RESET"},
   triggers_num, SEQUENCE(ROWS(triggers_str)),
   history, INDIRECT("R[1]C",),
   prev_triggers_state, INDIRECT("RC",),
   cur_triggers_state, SUMPRODUCT(triggers, triggers_num),
   cur_trigger_num, ABS(cur_triggers_state - prev_triggers_state),
   input, XLOOKUP(cur_trigger_num, triggers_num, triggers_str, ),
   ans, VSTACK(cur_triggers_state, IF(input = "RESET", , TRIM(history & " " & input))),
   ans
 )
```

