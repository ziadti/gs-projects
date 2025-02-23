# [RAM Simulator](https://docs.google.com/spreadsheets/d/1jW-FtCmz-W50WTalcnl8eSsYHx3QgwzC7LuZvQJ9OAE/)

_* Note that this was not extensively tested so it's likely that there are bugs._

This project was inspired by [this simulator](https://random-access-machine-emulator.netlify.app/). I made my own version to practice and because I didn't like that it didn't show the current state of the memory tape. 

It supports the following instructions:

<img src="https://github.com/user-attachments/assets/1a3366a2-5751-48df-aa2e-974c3db1372c" height="400">

## Demo

Example of a program that calculates sqrt(n) with n = 16.

https://github.com/user-attachments/assets/ef1062a4-54dd-436b-9b7b-e3f356bc67bf

## How I made it

The entire simulation is powered by this formula, which is just a translation of what the machine does.

```py
=ARRAYFORMULA(LET(
    i,SEQUENCE(COUNT(ram!B3:3),1,,1),pcs,--ram!A14:A,istr,ram!B14:B,
    RM,LAMBDA(ist,re,REGEXMATCH(ist,re)),
    RX,LAMBDA(ist,re,LET(x,REGEXEXTRACT(ist,re),IFERROR(--x,x))),
    VL,LAMBDA(k,t,i,VLOOKUP(k,t,i,0)),
    INS,LAMBDA(config,col,idx,val,IF(({-1;i}=idx)*(SEQUENCE(1,5)=col),val,config)),
    config,IFNA(HSTACK({-1;i},{0;TOCOL(ram!B2:2,1)},{"";IF(i^0," ")},{0;IF(i^0," ")},1)),
    x,REDUCE(config,SEQUENCE(A2),LAMBDA(conf,_,LET(
       pc,INDEX(conf,1,5),ist,RX(VL(pc,{pcs,istr},2),"(?:.+: *)?(.+)"),mz,INDEX(conf,2,3),rz,INDEX(conf,1,2),oz,INDEX(conf,1,4),
       x,IF(RM(ist,"^[^JH]"),RX(ist,".*[ =*](.+)"),),mx,IF(x="",,VL(x,conf,3)),mmx,IF(mx="",,VL(mx,conf,3)),
       new_conf_,
         IF(RM(ist,"READ *(\d|\*)"),INS(INS(INS(conf,3,IF(RM(ist,"\*"),mx,x),VL(rz,conf,2)),2,-1,rz+1),5,-1,pc+1),
           IF(RM(ist,"LOAD *(\d|\*|=)"),INS(INS(conf,3,0,IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx)),5,-1,pc+1),
             IF(RM(ist,"STORE *(\d|\*)"),INS(INS(conf,3,IF(RM(ist,"\*"),mx,x),mz),5,-1,pc+1),
               IF(RM(ist,"ADD *(\d|\*|=)"),INS(INS(conf,3,0,mz+IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx)),5,-1,pc+1),
                 IF(RM(ist,"SUB *(\d|\*|=)"),INS(INS(conf,3,0,mz-IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx)),5,-1,pc+1),
                   IF(RM(ist,"MUL *(\d|\*|=)"),INS(INS(conf,3,0,mz*IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx)),5,-1,pc+1),
                     IF(RM(ist,"DIV *(\d|\*|=)"),INS(INS(conf,3,0,INT(mz/IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx))),5,-1,pc+1),
                       IF(RM(ist,"WRITE *(\d|\*|=)"),INS(INS(INS(conf,4,INDEX(conf,1,4),IFS(RM(ist,"= *\d"),x,RM(ist,"\*"),mmx,1,mx)),4,-1,oz+1),5,-1,pc+1),
                         IF(RM(ist,"JUMP"),INS(conf,5,-1,VL(RX(ist,"JUMP *(\w+)")&"*",{istr,pcs},2)),
                           IF(RM(ist,"JZ|JGT|JGZ|JLT|JLZ"),INS(conf,5,-1,IF(IFS(RM(ist,"JZ"),mz=0,RM(ist,"JGT"),mz>0,RM(ist,"JGZ"),mz>=0,RM(ist,"JLT"),mz<0,1,mz<=0),VL(RX(ist,"(?:JZ|JGT|JGZ|JLT|JLZ) *(\w+)")&"*",{istr,pcs},2),pc+1)),
                             IF(RM(ist,"HALT"),INS(conf,5,-1,INDEX(conf,1,5)),)
                           )
                         )
                       )
                     )
                   )
                 )
               )
             )
           )
         ),
       new_conf,INS(new_conf_,3,-1,IF(AND(INDEX(conf,,3)=INDEX(new_conf_,,3)),,TOCOL(IF(INDEX(conf,,3)=INDEX(new_conf_,,3),,INDEX(conf,,1)),1))),
       fin,HSTACK(new_conf,pc,VL(INDEX(new_conf,1,5),{pcs,istr},2),ist,INDEX(new_conf,2,3),x,IF(x="",,VL(x,new_conf,3)),IF((x="")+(VL(x,new_conf,3)=""),,VL(VL(x,new_conf,3),new_conf,3))),
       IFNA(fin)))
      ),
    IF(A2,x,IFNA(HSTACK(config,,ram!B14)))))
```

To reset, step forward and step backward I used iterative calculation with the following formula:

```py
=LET(
   triggers, {ram!B11; ram!C11; ram!D11},
   triggers_str, {"Prev"; "Reset"; "Next"},
   triggers_num, SEQUENCE(ROWS(triggers_str)),
   counter, INDIRECT("R[1]C",),
   prev_triggers_state, INDIRECT("RC",),
   cur_triggers_state, SUMPRODUCT(triggers, triggers_num),
   cur_trigger_num, ABS(cur_triggers_state - prev_triggers_state),
   input, XLOOKUP(cur_trigger_num, triggers_num, triggers_str, ),
   output, VSTACK(cur_triggers_state, SWITCH(input, "Reset", 0, "Next", counter + 1, "Prev", counter - 1, counter)),
   output
 )
```
