# [Rubik's Cube Solver](https://docs.google.com/spreadsheets/d/10OTPkxLJdIL_NgJDqd0XcLkZhfuBC6HyFQo6GIUBLqs/)

A Rubik's cube solver that uses the [Old Pochmann method](https://ruwix.com/the-rubiks-cube/how-to-solve-the-rubiks-cube-blindfolded-tutorial/), a purely algorithmic intuitionless method that only solves one piece at a time designed for blindfolded solving. The lack of reliance on intuition makes this method easy to implement algorithmically with the drawback that the solutions result in a very long sequence of moves with an average that's over 300. (For reference, the [beginner's method](https://ruwix.com/the-rubiks-cube/how-to-solve-the-rubiks-cube-beginners-method/) has an aaverage number of moves of ~110, [CFOP](https://ruwix.com/the-rubiks-cube/advanced-cfop-fridrich/) has an average number of moves of ~55, [Roux](https://ruwix.com/the-rubiks-cube/different-rubiks-cube-solving-methods/roux-method/) has an average number of moves of ~45.)

## Demo

https://github.com/user-attachments/assets/2a609437-b373-4a3f-bd87-1fb1759f898a

## How I made it

There are two major steps involved in this project. First, we map the colors of the input scramble to their numeric value while adjusting the orientation so that the white face is in front and the green face is on top.

![Untitled](https://github.com/user-attachments/assets/ca4f0856-6120-4b75-9aa5-25294d2568cf)

Where this is the reference cube

![image](https://github.com/user-attachments/assets/7cf0ad03-6939-4ef0-a2c4-394632102c8d)

This is the formula that does this:

```py
=ARRAYFORMULA(LET(
  ref,D1:O9,
  scrambled,D36:O44,
  SLOOKUP,LAMBDA(keys,lookup,result,MAP(keys,LAMBDA(key,FILTER(result,EXACT(key,lookup))))),
  UPDATE_CUBE,LAMBDA(cube,move,REDUCE(cube,SPLIT(move,", "),LAMBDA(cube,move,
        LET(moves_,WRAPROWS(SPLIT(SLOOKUP(move,A1:A54,B1:B54),"/ "),2),
            moves,VLOOKUP(moves_,{TOCOL(ref),TOCOL(cube)},2,0),
            IFERROR(1/SUBSTITUTE(IFERROR(REDUCE(cube,SEQUENCE(ROWS(moves)),LAMBDA(cube,i,IF(cube=INDEX(moves,i,1),INDEX(moves,i,2)&"#",cube))),cube),"#",)^-1))))),
  iscenter,REGEXMATCH(ref&"","^(5|14|23|32|41|50)$"),
  isedge,REGEXMATCH(ref&"","^(2|4|6|8|11|13|15|17|20|22|24|26|29|31|33|35|38|40|42|44|47|49|51|53)$"),
  iscorner,REGEXMATCH(ref&"","^(1|3|7|9|10|12|16|18|19|21|25|27|28|30|34|36|37|39|43|45|46|48|52|54)$"),
  fixcenters,IF(iscenter,IF(scrambled="W",23,IF(scrambled="O",32,IF(scrambled="G",5,IF(scrambled="R",14,IF(scrambled="B",50,41))))),scrambled),
  edges_xprod,{TOCOL(Q2:Q25&TOROW(S2:S25)),TOCOL(R2:R25&TOROW(T2:T25))},
  pick_edge,BYROW(edges_xprod,LAMBDA(r,AND(COUNTIF(ref&fixcenters,r)))),
  edge_lookup,FILTER(INDEX(edges_xprod,,1),pick_edge),
  edge_result,VLOOKUP(FILTER(BYROW(REGEXEXTRACT(edges_xprod,"\D"),LAMBDA(r,JOIN(,r))),pick_edge),{S2:S25&T2:T25,Q2:Q25},2,0),
  fixedges,XLOOKUP(ref&fixcenters,edge_lookup,edge_result,fixcenters),
  corns_xprod,{TOCOL(V2:V49&TOROW(Y2:Y49)),TOCOL(W2:W49&TOROW(Z2:Z49)),TOCOL(X2:X49&TOROW(AA2:AA49))},
  pick_corn,BYROW(corns_xprod,LAMBDA(r,AND(COUNTIF(ref&fixcenters,r)))),
  corn_lookup,FILTER(INDEX(corns_xprod,,1),pick_corn),
  corn_result,VLOOKUP(FILTER(BYROW(REGEXEXTRACT(corns_xprod,"\D"),LAMBDA(r,JOIN(,r))),pick_corn),{Y2:Y49&Z2:Z49&AA2:AA49,V2:V49},2,0),
  fixcorners,XLOOKUP(ref&fixcenters,corn_lookup,corn_result,fixedges),
  front1,CHOOSECOLS(CHOOSEROWS(fixcorners,4,5,6),4,5,6),top1_,CHOOSEROWS(CHOOSECOLS(fixcorners,4,5,6),1,2,3),bottom1,CHOOSEROWS(CHOOSECOLS(fixcorners,4,5,6),7,8,9),
  left1,CHOOSECOLS(CHOOSEROWS(fixcorners,4,5,6),1,2,3),right1,CHOOSECOLS(CHOOSEROWS(fixcorners,4,5,6),7,8,9),back1,CHOOSECOLS(CHOOSEROWS(fixcorners,4,5,6),10,11,12),
  move1,SPLIT(IF(COUNTIF(top1_,23),"x'",IF(COUNTIF(bottom1,23),"x",IF(COUNTIF(left1,23),"y'",IF(COUNTIF(right1,23),"y",IF(COUNTIF(back1,23),"x2",))))),", "),
  fixorientation1,UPDATE_CUBE(fixcorners,move1),
  front2,CHOOSECOLS(CHOOSEROWS(fixorientation1,4,5,6),4,5,6),top2_,CHOOSEROWS(CHOOSECOLS(fixorientation1,4,5,6),1,2,3),bottom2,CHOOSEROWS(CHOOSECOLS(fixorientation1,4,5,6),7,8,9),
  left2,CHOOSECOLS(CHOOSEROWS(fixorientation1,4,5,6),1,2,3),right2,CHOOSECOLS(CHOOSEROWS(fixorientation1,4,5,6),7,8,9),back2,CHOOSECOLS(CHOOSEROWS(fixorientation1,4,5,6),10,11,12),
  move2,SPLIT(IF(COUNTIF(bottom2,5),"z2",IF(COUNTIF(left2,5),"z",IF(COUNTIF(right2,5),"z'",))),", "),
  fixorientation2,UPDATE_CUBE(fixorientation1,move2),
  fixorientation2
))
```

This formula uses these two lookup tables:

![image](https://github.com/user-attachments/assets/aa4f247b-e25a-4f16-9423-a99ae82991b8)

Once we have done this, we apply the actual algorithm to solve the cube:

```py
=ARRAYFORMULA(LET(
  ref,D1:O9,
  scrambled,D48:O56,
  SLOOKUP,LAMBDA(keys,lookup,result,MAP(keys,LAMBDA(key,FILTER(result,EXACT(key,lookup))))),
  UPDATE_CUBE,LAMBDA(cube,move,REDUCE(cube,SPLIT(move,", "),LAMBDA(cube,move,
        LET(moves_,WRAPROWS(SPLIT(SLOOKUP(move,A1:A54,B1:B54),"/ "),2),
            moves,VLOOKUP(moves_,{TOCOL(ref),TOCOL(cube)},2,0),
            IFERROR(1/SUBSTITUTE(IFERROR(REDUCE(cube,SEQUENCE(ROWS(moves)),LAMBDA(cube,i,IF(cube=INDEX(moves,i,1),INDEX(moves,i,2)&"#",cube))),cube),"#",)^-1))))),
  IS_EDGE_SOLVED,LAMBDA(edge,cube,MAP(edge,LAMBDA(edge_,OR((edge_=ref)*(edge_=cube))))),
  IS_CORNER_SOLVED,LAMBDA(corner,cube,MAP(corner,LAMBDA(corner_,OR((corner_=ref)*(corner_=cube))))),
  iscenter,REGEXMATCH(ref&"","^(5|14|23|32|41|50)$"),
  isedge,REGEXMATCH(ref&"","^(2|4|6|8|11|13|15|17|20|22|24|26|29|31|33|35|38|40|42|44|47|49|51|53)$"),
  iscorner,REGEXMATCH(ref&"","^(1|3|7|9|10|12|16|18|19|21|25|27|28|30|34|36|37|39|43|45|46|48|52|54)$"),
  isbuffer,REGEXMATCH(ref&"","^(10|6)$"),
  all_edges,TOCOL(1/(isedge*ref)^-1,2),
  unsolved_edges,FILTER(all_edges,IS_EDGE_SOLVED(all_edges,scrambled)-1),
  initial_edge,TOCOL(1/(isbuffer*isedge*scrambled)^-1,2),
  edge_buf1,29,edge_buf2,6,
  solve_edges_,ARRAY_CONSTRAIN(REDUCE(VSTACK(HSTACK(scrambled,),HSTACK(unsolved_edges,initial_edge)),SEQUENCE(20),LAMBDA(a,i,LET(
       cube,ARRAY_CONSTRAIN(a,9,12),
       cur_edge_,--TEXTJOIN(,,IF(ref*isedge*isbuffer,cube,)),
       cur_edge,IF(OR(cur_edge_=edge_buf1,cur_edge_=edge_buf2),INDEX(a,10,1),cur_edge_),
       edge_alg,SUBSTITUTE(VLOOKUP(cur_edge,AC1:AD50,2,0),AJ2,AK2),
       updated_cube,UPDATE_CUBE(cube,edge_alg),
       solved_edges,FILTER(all_edges,IS_EDGE_SOLVED(all_edges,updated_cube)),
       unsolved_edges_,FILTER(all_edges,0=COUNTIF(solved_edges,all_edges)),
       unsolved_edges,SORT(unsolved_edges_,--REGEXMATCH(unsolved_edges_&"","^(6|29)$"),1),
       VSTACK(HSTACK(updated_cube,IFNA(INDEX(a,1,13)&"Edge "&VLOOKUP(cur_edge,AF1:AG49,2,0)&": "&edge_alg&CHAR(10),INDEX(a,1,13))),HSTACK(unsolved_edges,solved_edges)))
       )),9,13),
  solve_edges,ARRAY_CONSTRAIN(solve_edges_,9,12),
  all_corns,TOCOL(1/(iscorner*ref)^-1,2),
  unsolved_corns,FILTER(all_corns,IS_CORNER_SOLVED(all_corns,solve_edges)-1),
  initial_corn,TOCOL(1/(isbuffer*iscorn*solve_edges)^-1,2),
  solve_corns,ARRAY_CONSTRAIN(REDUCE(VSTACK(solve_edges_,HSTACK(unsolved_corns,initial_corn)),SEQUENCE(20),LAMBDA(a,i,LET(
       cube,ARRAY_CONSTRAIN(a,9,12),
       cur_corn_,--TEXTJOIN(,,IF(ref*iscorner*isbuffer,cube,)),
       cur_corn,IF(OR(cur_corn_=10,cur_corn_=39,cur_corn_=1),INDEX(a,10,1),cur_corn_),
       corn_alg,SUBSTITUTE(VLOOKUP(cur_corn,AC1:AD50,2,0),AJ3,AK3),
       updated_cube,UPDATE_CUBE(cube,corn_alg),
       solved_corns,FILTER(all_corns,IS_CORNER_SOLVED(all_corns,updated_cube)),
       unsolved_corns_,FILTER(all_corns,0=COUNTIF(solved_corns,all_corns)),
       unsolved_corns,SORT(unsolved_corns_,--REGEXMATCH(unsolved_corns_&"","^(1|10|39)$"),1),
       VSTACK(HSTACK(updated_cube,IFNA(INDEX(a,1,13)&"Corner "&VLOOKUP(cur_corn,AF1:AG49,2,0)&": "&corn_alg&CHAR(10),INDEX(a,1,13))),HSTACK(unsolved_corns,solved_corns)))
       )),9,13),
  solution,INDEX(solve_corns,1,13),
  solve_corns
))
```

The formula above uses this lookup table which maps each edge to the corresponding algorithm:

![image](https://github.com/user-attachments/assets/608a2639-668b-4ab4-af4e-63dcde9d5547)

And this lookup table which maps each corner to the corresponding algorithm:

![image](https://github.com/user-attachments/assets/535134d7-f6fe-4c8d-92c8-0ebd1bb5f7f8)




