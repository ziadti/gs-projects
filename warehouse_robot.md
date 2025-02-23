# [Warehouse Robot](https://docs.google.com/spreadsheets/d/1jW-FtCmz-W50WTalcnl8eSsYHx3QgwzC7LuZvQJ9OAE/)

Simulates a robot moving across a grid, pushing boxes and running into obstacles.

It's a simulation of the first part of the [15th puzzle of Advent of Code 2024](https://adventofcode.com/2024/day/15).

## Demo

https://github.com/user-attachments/assets/a53f906b-07b3-44d6-8708-129720c2b950

## How I made it

I mostly used regular expressions to simulate the movements of the robot.

- To move left, we find: `[^#O]((?:O+)?@)(.*)` and replace it with `$1.$2`. (See [here](<https://regex101.com/r/0Lo6yG/1>))
- To move right, we find: `(.*)(@(?:O+)?)[^#O]` and replace it with `$1.$2`. (See  [here](<https://regex101.com/r/a4qKKI/2>))
- To move up and down, we rotate the grid, move left/right using the patterns above and rotate it back.

The formula that does this is the following:

```py
=ARRAYFORMULA(IFERROR(LET(
   R,LAMBDA(g,LET(
      s,TOCOL(SPLIT(g,CHAR(10))),
      x,MID(s,SEQUENCE(1,LEN(SINGLE(s))),1),
      BYCOL(
        BYROW(TRANSPOSE(SORT(x,SEQUENCE(ROWS(x)),)),LAMBDA(r,JOIN(,r))),
        LAMBDA(c,JOIN(CHAR(10),c))
      )
   )),
   RN,LAMBDA(g,LET(
       s,TOCOL(SPLIT(g,CHAR(10))),
       x,MID(s,SEQUENCE(1,LEN(SINGLE(s))),1),
       BYCOL(
         BYROW(SORT(TRANSPOSE(x),SEQUENCE(COLUMNS(x)),),LAMBDA(r,JOIN(,r))),
         LAMBDA(c,JOIN(CHAR(10),c))
       )
  )),
  ML,LAMBDA(x,REGEXREPLACE(x,"[^#O]((?:O+)?@)(.*)","$1.$2")),
  MR,LAMBDA(x,REGEXREPLACE(x,"(.*)(@(?:O+)?)[^#O]","$1.$2")),
  MD,LAMBDA(x,RN(ML(R(x)))),
  MU,LAMBDA(x,RN(MR(R(x)))),
  g,INDEX(s,1),
  m_,INDEX(s,2),
  m,MID(m_,SEQUENCE(LEN(m_)),1),
  SWITCH(B3,"←",ML(A2),"→",MR(A2),"↓",MD(A2),"↑",MU(A2))
 )))
```

This is the iterative calculation formula that allows us to keep a history of the movements:

```py
=LET(
   triggers, {simulator!P15; simulator!R15; simulator!Q14; simulator!Q15; simulator!U15},
   triggers_str, {"←"; "→"; "↑"; "↓"; "RST"},
   triggers_num, SEQUENCE(ROWS(triggers_str)),
   history, INDIRECT("R[1]C",),
   prev_triggers_state, INDIRECT("RC",),
   cur_triggers_state, SUMPRODUCT(triggers, triggers_num),
   cur_trigger_num, ABS(cur_triggers_state - prev_triggers_state),
   input, XLOOKUP(cur_trigger_num, triggers_num, triggers_str, ),
   output, VSTACK(cur_triggers_state, SWITCH(input, "RST", , history & input)),
   output
 )
```
