# [K-Tapes Universal Turing Machine](https://docs.google.com/spreadsheets/d/155Oy3Tj-aVHUmOCx8cMOMZB7qcaxSYQodoDwVqXcbgE/)

A [Universal Turing Machine](https://en.wikipedia.org/wiki/Universal_Turing_machine) simulator that takes the description of a Turing machine, an input string, and up to three tapes, then processes it step by step. It allows stepping forward and backward, as well as resetting to the initial state. It also supports the wildcard symbol `*`, which can be used to match any symbol when reading from the tape or to write a symbol that was read in the current transition.

<img src="https://github.com/user-attachments/assets/426e82d7-2393-412b-9f85-ab8519ab73e0" height="390">


## Demo

Example of a Turing machine that increments a decimal number by 1

https://github.com/user-attachments/assets/a8d4bbfa-3f29-4e4c-a164-9c45498dc5b2

Example of a Turing machine that swaps every two characters

https://github.com/user-attachments/assets/7b1b7580-d32d-46cb-8b93-a8489ae1c2a4


## How I made it

Below is the formula that simulates the machine. It's a terrible formula because I began this project with the idea of only simulating a 1 tape Turing machine but then I changed my mind and wanted to expand it to work with up to 3 tapes.

```py
=ARRAYFORMULA(LET(
   rw,ROWS(D3:D),n_cells,50,input,TO_TEXT(S10)&IF(S10=""," ",),ini_state,P4,fin_state,P5,trans_,FILTER(TOCOL(O10:O,1),0=COUNTIF(B3:B,TOCOL(N10:N,1))),k,3,s,SEQUENCE(1,k),steps,C3,c_,BYROW(SPLIT(trans_," "),LAMBDA(r,COUNTA(r))),dbg,--REGEXEXTRACT(J2,"\d+"),
   c,IF(AND(INDEX(c_,1)=c_),INDEX(c_,1),0/0),k_,(c-2)/3,n,CHAR(10),ll_,"[current_state] ["&JOIN("] [","read_symbol"&s)&"] [next_state] ["&TEXTJOIN("] [",,{"write_symbol";"move"}&s)&"]",
   trans,SPLIT(IF(OR(AND(k=1,k_=1),AND(k=2,k_=2),AND(k=3,k_=3)),trans_,IF(AND(k=3,k_=2),REGEXREPLACE(trans_,"(.+? .+? .+?) (.+? .+? .+?) (.+)","$1 _ $2 _ $3 -"),
      IF(AND(k=2,k_=1),REGEXREPLACE(trans_,"(.+? .+?) (.+? .+?) (.+)","$1 _ $2 _ $3 -"),IF(AND(k=3,k_=1),REGEXREPLACE(trans_,"(.+? .+?) (.+? .+?) (.+)","$1 _ _ $2 _ _ $3 - -"),"ERRORS"))))," "),
   ll,SPLIT(REGEXREPLACE(ll_,"(?:urrent|ext)_state|ead|rite|ove|_symbol|[\[\]]",)," "),
   idx,SEQUENCE(n_cells,1,-INT(n_cells/2)),s_input,MID(input,SEQUENCE(LEN(input)),1),
   J,LAMBDA(a,IFNA(n&BYCOL(BYROW(a,LAMBDA(r,JOIN(" ",r))),LAMBDA(c,JOIN(CHAR(10),c))),"NA")),
   FILL,LAMBDA(arr,rept,IF(ROWS(arr)=1,WRAPCOLS(arr,rept,arr),arr)),
   INS_,LAMBDA(arr,idx,col,val,rel,IF((SEQUENCE(1,COLUMNS(arr))=col)*(IF(rel,INDEX(arr,,1)=idx,SEQUENCE(n_cells)=idx)),val,arr)),
   INS,LAMBDA(arr,idx,col,val,rel,LET(r,ROWS(idx),res,REDUCE(arr,SEQUENCE(r),LAMBDA(a,i,
        INS_(a,INDEX(idx,i),INDEX(FILL(col,r),i),INDEX(FILL(val,r),i),INDEX(FILL(rel,r),i)))),IF(res=TRUE,,res))),
   ini_config_,IFNA(HSTACK(idx,IF(idx^{0,0,0},"_"),{0;ini_state;0;0;0}),TRUE),
   read_sym1,LEFT(input),read_sym2,"_",read_sym3,"_",
   lft_,REGEXEXTRACT(trans_,"("&REPT(".*? ",k_)&".*?) "),sp,SPLIT(lft_," "),
   lft,SUBSTITUTE(REGEXREPLACE(SUBSTITUTE(BYROW(sp,LAMBDA(sp,REGEXREPLACE(TEXTJOIN(" ",1,INDEX(sp,,1),SWITCH(INDEX(sp,,2),"*v",read_sym2,"*vv",read_sym3,INDEX(sp,,2)),
         IFERROR(SWITCH(INDEX(sp,,3),"*v",read_sym3,"*^",read_sym1,INDEX(sp,,3))),IFERROR(SWITCH(INDEX(sp,,4),"*^",read_sym2,"*^^",read_sym1,INDEX(sp,,4)))),
          "\*([^^v]|$)",".+?$1"))),".+?",CHAR(9)),"[()\[\]{}|\\^$.+?]","\\$0"),CHAR(9),".+?"),
   key,REGEXEXTRACT(ini_state&" "&LEFT(input)&" _ _","("&REPT(".+? ",k_)&".+?) ?"),ini,FILTER(SEQUENCE(ROWS(trans_)),REGEXMATCH(key,lft)),
   pri,BYROW(SPLIT(FILTER(lft,REGEXMATCH(key,lft))," "),LAMBDA(r,SUMPRODUCT(REGEXMATCH(TO_TEXT(r),"\.\+\?")))),nondet,--OR(COUNTIF(pri,MIN(pri))>1),
   ini_config,INS(IFNA(HSTACK(INS(INS(ini_config_,SEQUENCE(LEN(input))-1,2,s_input,TRUE),{6;7;8},5,{LEFT(input);"_";"_"},FALSE),,)),{2;3},6,{IF(nondet-1,SORTN(FILTER(SEQUENCE(ROWS(trans_)),REGEXMATCH(key,lft)),1,,pri,1),JOIN(" ",ini));nondet},FALSE),
   fin_config,REDUCE(ini_config,SEQUENCE(steps),LAMBDA(config,step,LET(
        cur_state,INDEX(config,2,5),read_syms,CHOOSEROWS(TOCOL(INDEX(config,,5),1),SEQUENCE(k,1,-k)),
        key__,cur_state&" "&JOIN(" ",read_syms),key_,SPLIT(key__," "),lft_,REGEXEXTRACT(trans_,"("&REPT(".*? ",k_)&".*?) "),sp,SPLIT(lft_," "),
        key,REGEXEXTRACT(key__,"("&REPT(".+? ",k_)&".+?) ?"),
        read_sym1,INDEX(key_,2),read_sym2,INDEX(key_,3),read_sym3,INDEX(key_,4),
        lft,SUBSTITUTE(REGEXREPLACE(SUBSTITUTE(BYROW(sp,LAMBDA(sp,REGEXREPLACE(TEXTJOIN(" ",1,INDEX(sp,,1),SWITCH(INDEX(sp,,2),"*v",read_sym2,"*vv",read_sym3,INDEX(sp,,2)),IFERROR(SWITCH(INDEX(sp,,3),"*v",read_sym3,"*^",read_sym1,INDEX(sp,,3))),IFERROR(SWITCH(INDEX(sp,,4),"*^",read_sym2,"*^^",read_sym1,INDEX(sp,,4)))),"\*([^^v]|$)",".+?$1"))),".+?",CHAR(9)),"[()\[\]{}|\\^$.+?]","\\$0"),CHAR(9),".+?"),
        cur_trans,FILTER(CHOOSECOLS(trans,1,2,3,4),REGEXMATCH(key,lft)),next_trans_,FILTER(CHOOSECOLS(trans,SEQUENCE(k*2+1,1,k+2)),REGEXMATCH(key,lft)),
        pri,BYROW(SPLIT(FILTER(lft,REGEXMATCH(key,lft))," "),LAMBDA(r,SUMPRODUCT(REGEXMATCH(TO_TEXT(r),"\.\+\?")))),next_trans,SORTN(next_trans_,1,,pri,1),
        new_state,INDEX(next_trans,1),
        write_sym1_,INDEX(next_trans,2),write_sym2_,INDEX(next_trans,3),write_sym3_,INDEX(next_trans,4),
        move1,INDEX(next_trans,5),move2,INDEX(next_trans,6),move3,INDEX(next_trans,7),
        pos1_,INDEX(config,3,5),pos2_,INDEX(config,4,5),pos3_,INDEX(config,5,5),
        new_pos1,pos1_+SWITCH(move1,">",1,"<",-1,),new_pos2,pos2_+SWITCH(move2,">",1,"<",-1,),new_pos3,pos3_+SWITCH(move3,">",1,"<",-1,),
        write_sym1,SWITCH(write_sym1_,"*",read_sym1,"*v",read_sym2,"*vv",read_sym3,write_sym1_),
        write_sym2,SWITCH(write_sym2_,"*",read_sym2,"*^",read_sym1,"*v",read_sym3,write_sym2_),
        write_sym3,SWITCH(write_sym3_,"*",read_sym3,"*^^",read_sym1,"*^",read_sym2,write_sym3_),
        new_config_,INS(config,{pos1_;pos2_;pos3_},{2;3;4},{write_sym1;write_sym2;write_sym3},TRUE),
        new_read1,VLOOKUP(new_pos1,new_config_,2,0),new_read2,VLOOKUP(new_pos2,new_config_,3,0),new_read3,VLOOKUP(new_pos3,new_config_,4,0),
        new_lft,SUBSTITUTE(REGEXREPLACE(SUBSTITUTE(BYROW(sp,LAMBDA(sp,REGEXREPLACE(TEXTJOIN(" ",1,INDEX(sp,,1),SWITCH(INDEX(sp,,2),"*v",new_read2,"*vv",new_read3,INDEX(sp,,2)),IFERROR(SWITCH(INDEX(sp,,3),"*v",new_read3,"*^",new_read1,INDEX(sp,,3))),IFERROR(SWITCH(INDEX(sp,,4),"*^",new_read2,"*^^",new_read1,INDEX(sp,,4)))),"\*([^^v]|$)",".+?$1"))),".+?",CHAR(9)),"[()\[\]{}|\\^$.+?]","\\$0"),CHAR(9),".+?"),
        new_read_syms,{new_read1;new_read2;new_read3},new_key__,new_state&" "&JOIN(" ",new_read_syms),
        new_key,REGEXEXTRACT(new_key__,"("&REPT(".+? ",k_)&".+?) ?"),new_pri,BYROW(SPLIT(FILTER(lft,REGEXMATCH(new_key,new_lft))," "),LAMBDA(r,SUMPRODUCT(REGEXMATCH(TO_TEXT(r),"\.\+\?")))),
        prev,SORTN(FILTER(SEQUENCE(ROWS(trans_)),REGEXMATCH(key,lft)),1,,pri,1),
        next_,FILTER(SEQUENCE(ROWS(trans_)),REGEXMATCH(new_key,new_lft)),next,IFERROR(JOIN(" ",next_),INDEX(config,2,6)),
        nondet,--OR(INDEX(config,3,6),IF(ISNA(new_pri),,COUNTIF(new_pri,MIN(new_pri))>1)),
        new_config,INS(new_config_,SEQUENCE(8),5,{step;new_state;new_pos1;new_pos2;new_pos3;new_read1;new_read2;new_read3},FALSE),
        res,INS(INS(IF(OR(ISNA(new_state),nondet,cur_state=fin_state),config,new_config),1,7,IF(dbg=0,,JOIN(n&n,
                 J(next_),J(prev),J(FILTER(SEQUENCE(ROWS(trans_)),REGEXMATCH(key,lft))),J(INDEX(config,1,6)),TOCOL({"key"&J(key),"matches"&J(FILTER(lft,REGEXMATCH(key,lft))),"next_trans_"&J(next_trans_),"pri"&J(pri),
                 "nondet"&J(OR(COUNTIF(pri,pri)>1)),"next_trans"&J(next_trans),"write_sym1"&n&write_sym1,"write_sym2"&n&write_sym2,"write_sym3"&n&write_sym3,"lft"&J(lft),
                 "regm"&J(REGEXMATCH(key,lft)),"trans"&J(CHOOSECOLS(trans,SEQUENCE(k*2+1,1,k+2)))},2))),FALSE),{1;2;3},6,{IF(IFNA(OR(nondet,new_state=fin_state,ISNA(next_))),INDEX(config,2,6),prev);
               IFNA(IF(nondet,next,SORTN(next_,1,,new_pri,1)));nondet},FALSE),
        IFERROR(res,config)))),
   fin,IF(OR(P6="",S10=""),,IF(OR(steps=0,nondet,ROWS(TOCOL(trans_,2))=0),ini_config,fin_config)),
   IFNA(fin)
))
```

This is the formula that keeps track of the current step and allow us to reset and step forward and backward. It uses iterative calculation:

```py
=LET(
   triggers, {S14; T14; U14},
   triggers_str, {"Prev"; "Reset"; "Next"},
   triggers_num, SEQUENCE(ROWS(triggers_str)),
   counter, INDIRECT("RC",),
   prev_triggers_state, INDIRECT("R[1]C",),
   cur_triggers_state, SUMPRODUCT(triggers, triggers_num),
   cur_trigger_num, ABS(cur_triggers_state - prev_triggers_state),
   input, XLOOKUP(cur_trigger_num, triggers_num, triggers_str, ),
   output, VSTACK(MAX(0,SWITCH(input, "Reset", 0, "Next", H3 + 1, "Prev", H3 - 1, counter)), cur_triggers_state),
   output
 )
```
