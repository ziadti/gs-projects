# [Tower of Hanoi](https://docs.google.com/spreadsheets/d/1iNaOEQKcJIUR_groXezZ9reZbs2J5qOCrMnnYw3QtC4/)

A playable [Tower of Hanoi](https://en.wikipedia.org/wiki/Tower_of_Hanoi) that works with up to 8 discs. It also includes a solver that automatically steps through the optimal solution.

## Demo


https://github.com/user-attachments/assets/4e8e0447-013a-4059-bc9a-2b9a5b13e150


## How I made it

Formula for the optimal solution:

```py
=LET(
   n_discs, U4,
   TOH, LAMBDA(TOH, n, from, to, aux,
          IF(n=0,,
             TOH(TOH, n-1, from, aux, to)&
             from & to &
             TOH(TOH, n-1, aux, to, from)
           )),
   TOH(TOH, n_discs, "A", "C", "B")
 )
```

Formula for the logic of the stack movements:

```py
=ARRAYFORMULA(LET(
    A, E3:E10, B, F3:F10, C, G3:G10,
    swaps_, INDEX(SPLIT(K1,"|"),2),
    swaps, IF(swaps_="SOL",K2,swaps_),
    moves_, IFERROR(WRAPROWS(MID(swaps,SEQUENCE(LEN(swaps)),1),2)),
    moves, FILTER(moves_, BYROW(moves_, LAMBDA(r, COUNTA(r)=2))),
    from_, INDEX(moves,,1),to_,INDEX(moves,,2),
    state, {A, B, C},
    idx, SEQUENCE(8),
    emp, REPT(",", 16),
    GET_TOP, LAMBDA(x, IFERROR(INDEX(FILTER(SEQUENCE(8)-1, x<>emp),1), 8)),
    MOVE, LAMBDA(state, x, y, LET(
      A, INDEX(state,,1), B, INDEX(state,,2), C, INDEX(state,,3),
      idx, SEQUENCE(8), emp, REPT(",", 16),
      from_, XMATCH(x, {"A","B","C"}), to_, XMATCH(y, {"A","B","C"}),
      from, INDEX(state,,from_), to, INDEX(state,,to_),
      IF({"A","B","C"} = x,
        IF(idx = GET_TOP(from)+1, emp, from),
        IF({"A","B","C"} = y, 
          IF(idx = GET_TOP(to), INDEX(from, GET_TOP(from)+1), to),
          state
        )
      )
    )),
    ans, 
     REDUCE(state,SEQUENCE(ROWS(moves)),LAMBDA(state,i,LET(
       A,INDEX(state,,1),B,INDEX(state,,2),C,INDEX(state,,3),
       val_A,--IFERROR(SINGLE(TOCOL(IFNA(REGEXEXTRACT(A,"\d+")),1)),9),
       val_B,--IFERROR(SINGLE(TOCOL(IFNA(REGEXEXTRACT(B,"\d+")),1)),9),
       val_C,--IFERROR(SINGLE(TOCOL(IFNA(REGEXEXTRACT(C,"\d+")),1)),9),
       from,INDEX(from_,i),to,INDEX(to_,i),
       res, IF(from = "A", 
         IF(to = "B", 
           IF(val_A < val_B, MOVE(state, "A","B"), state),
           IF(val_A < val_C, MOVE(state, "A","C"), state)
         ),
         IF(from = "B", 
           IF(to = "A", 
             IF(val_B < val_A, MOVE(state, "B","A"), state),
             IF(val_B < val_C, MOVE(state, "B","C"), state)
           ),
           IF(from = "C", 
             IF(to = "A",
               IF(val_C < val_A, MOVE(state, "C","A"), state),
               IF(val_C < val_B, MOVE(state, "C","B"), state)
             ),
             state
           )
         )
      ), IF(from = to, state, res)))),
    IFNA(ans, state)
 ))
```


Formula for keeping track of the history of the movements:

```py
=ARRAYFORMULA(LET(
   s, SEQUENCE(ROWS(TOCOL(O19:AM38))),
   triggers, {RIGHT({TOCOL(O19:AM38); TOCOL(AN19:BL38); TOCOL(BM19:CK38); AH4; AR4 ; Z8})=""},
   triggers_str, {IF(s,"A"); IF(s,"B"); IF(s,"C"); "RST" ; "UNDO"; "SOL"},
   triggers_num, SEQUENCE(ROWS(triggers_str)),
   prev_state, INDIRECT("RC",),
   prev_state_splitted, SPLIT(prev_state, "|"),
   moves_history, IFERROR(INDEX(prev_state_splitted, 2)),
   prev_triggers_state, INDEX(prev_state_splitted, 1),
   cur_triggers_state, SUMPRODUCT(triggers, triggers_num),
   cur_trigger_num, ABS(cur_triggers_state - prev_triggers_state),
   input, XLOOKUP(cur_trigger_num, triggers_num, triggers_str, ),
   moves_no_dupes_, IFERROR(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(REGEXREPLACE(moves_history,"..","$0 "), "AA",), "BB",), "CC",)," ",)),
   moves_no_dupes, moves_no_dupes_,
   cur_state, cur_triggers_state & "|" & SWITCH(input, "RST", , "UNDO", IFERROR(LEFT(moves_no_dupes, LEN(moves_no_dupes)-2)), "SOL", "SOL", IF(REGEXMATCH(moves_no_dupes & input, "SOL.+"), INDIRECT("R[1]C",) & REGEXEXTRACT(moves_no_dupes & input, "SOL(.+)"), moves_no_dupes & input)),
   VSTACK(cur_state, IFERROR(LEFT(B6, B5)))
 ))
```

