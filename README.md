# AdvancedScript

just a try to add more feature's to x64dbg script system

////////////////////////////////////////////////////////////////////////////
## History Section:
```
- version 2.0:
      1-all numbers are hex numbers.
      2-more nested in arguments.
      3-Build bridge to make plugin system Compatible with x64dbg script system.
      4-create parallel Functions to x64dbg Functions, like ( cmp >> cmpx ).
      5-rename new name (Varx Getx Setx) and fix array index entry.
      6-add VarxClear ( clear all variable to help user in test's ) , memdump with print style.
- version 1.6:
      1- add Parser system to recognize arguments.
      2- begin build Script system.
      3- add more Helper Functions.
- version 1.4:
      1- make StrCompx in separate Thread and add Sleep time to wait x64dbg to finish process.
      2- Fix Hex2duint function add length check in case it less than 2 . 
- version 1.3:
      1- Add another argument to cbLogxJustAtBP for printing on LogxWindow.
      2- now it accept bool argument like this (true/false-on/off-1/0).
      3- add StrComp_BP function for compare string in memory at BP.
      4- compiled x32.
 ```

////////////////////////////////////////////////////////////////////////////
## Script Section:
```
-arguments value system (AVS): all argument pass through Parser system recognizer,how it work:
      1- all numbers are in hex shape ( setx $x,50  == setx $x,0x50). 
      2- search for all {} which is releated x64dbg system and try to resolve it.
      3- search for all $ which it's related to defined variables of AdvancedScript and resolve it. 
      4- search for all tokens by order ( * / + -) and resolve it.
      5- all numbers in int variables are saved as decimal, put when it printed it print with 
            two way hex and decimal.
      6- all numbers which stored in str or array var are heximal.
      7- when define variables no need to use $, but wher ever you use this var you should add $ first.
      8- when calulate int value all arguments will converted to int and make the formula exist.
      9- when calulate str or array all arguments are set to gather after it calculate it's value's.
      10- when we define array it will create array with 0x500 elements.
```
### 1- Varx: 
it's Like Var in x64dbg system, for defining variable's which can used in Script commands.
```
Parameter:
Varx P1, P2 , P3(optional) 
      P1: variable type it holde ( str , int , array )  /// array is defined with 500 elements.
      P2: variable name it should not have spaces or begin with $ , 
        >>>>>>but when resolve it's value it should add $ before it.<<<< like this $x  >> value of x 
      P3: the value of the variable it's optional for str and array only and the value will be null
            for int type it should be have a value and must be int 
            note : you can use variable for this arguments like $x or {rax}.
            no need to use ""
 Variable type is :
 - int: all value will saved as int value.
 - str.
 - array with 0x500 string elements ( it's just string).
  sample :
         - varx int, x, 90
         - varx array, y, 1
         - varx str, x, {rax}
         - varx array, y
         - varx str, x
         //////////
         - varx int, x, 0x45fa
         - varx int,x1,25+30     /// 0x55 /85
         - varx array, z,30
         - setx $z[10],test
         - varx int,x2,$x +$x1+$z[0]
```
### 2- Setx : 
set value to the virables in AdvancedScript vriable system or x64dbg system.
you can make add sub or multi or divide and you can nested arguments as you like.

```
Parameter:
Setx P1, P2 
      P1: variable name, it must (not) begin with $, because we need to assigned new value .
      for the array variable we use [Array_index] , no need this for int and str or Array_index=0
       array_index can accept variable's value
      P2: the value of the variable can used AVS, no need to use "".
      
   sample :
         - varx int, x, 90                x=0x90
           Setx x,10                      x=0x10
           
         - varx str, x, {rax}             x=rax value
           Setx $x,test                   x=test
           varx array,z,10                z[0]=10     because all elements are string       
           setx $z[5],$x$z[0]             z[5]=test10
           setx $z[5],$x $z[0]            z[5]=test 10
           setx $z[5],$x+$z[0]            z[5]=test+10
           setx $x,$z[0]                  x=0x10
           setx $x,$z[5]                  wrong value can't converted to int
           
         - varx int, x, 0x45fa            int x= 0x45FA\17914 :has been added
           varx str, z, 0xaa              str z= 0xaa :has been added
           setx $x, $z + 0x33 - 25        x= 0xB8\184
           varx array, y, 0x10            array y[0]= 0x10 :has been added
           setx $x, $x + $y[0]            x= 0xC8\200
```
### 3- Getx / Printx : 
print the value of the virables .
```
Parameter:
SetVarx P1, P2 
      P1: variable name, it must begin with $, because we need to get the value .
            for the array we put [array_index] if not used then it will print first 
            element in the array.
            array_index can accept variable's value
          the value of the variable can used AVS, no need to use comma "".
      
   sample :
         - varx int, x, 90          x=0x90
           SetVarx $x,10            x=0x10
           getx $x                  x=0x10
         - varx str, x, {rax}       x=rax value
           getx $x                  x=rax value
         
         -varx int, x, 10           int x= 0x10\16 :has been added
          varx array, y, 1          array y[0]= 1 :has been added
          setx $y[$x], 110          y[10]= 110
          getx $y                   y[0]= 1
          getx $y[$x]               y[10]= 110
```
### 3- commands parallel of x64dbg  : 
it's collection of edit functions from x64dbg system, but it accept variables in the parameter.
- (Movx , addx , subx , mulx , divx , andx , orx , xorx , shlx ) : first parameter will not analyzed, just the second one.
- (pushx , popx):parameter will analyzed
- cmpx : both parameter will analyzed
```
sample :
     - again:
      varx str,base,{rax}
      cmpx {rax},$base   	>>> cmpx analyzed both parameters so we can write like this
      jne done
      jmp again 
      done:
     - varx int,x,20
       varx int,y,0xFF
       var z     		>>> this var from x64dbg system
       addx rax,$x+$y               rax =rax + 0x32+ 0x255 = rax + 0x287 
       addx z,$x+$y                 z=z+0x287

         
```
### 4- findallmemx : 
it's same findallmem in x64dbg system, but it accept variables in the parameter.
```
Parameter:
SetVarx P1, P2, P3 
      P1: The address to start searching from.
      P2: The byte pattern to search for. This byte pattern can contain wildcards (?)
            for example: EB0?90??8D
      P3: The size of the data to search in. Default is the entire memory map..
                  >>all variable can used AVS<<
   sample :
         - varx str, search, "4533C94533C033"  << or >> varx str, search, 4533C94533C033
           varx str, base, { rdx }
           findallmemx $base, $search
           mov rdi, ref.addr(0)
         
```
### 5- memdump : 
dump memory to log window like windbg style
```
	//00007ff8`02f42280  cc c3 cc cc cc cc cc cc - 0f 1f 84 00 00 00 00 00  ................
	//00007ff8`02f42290  cc c3 cc cc cc cc cc cc - 0f 1f 84 00 00 00 00 00  ................

Parameter:
SetVarx P1, P2, P3 
      P1: The address to start dump.
      P2: size of data   
   sample :
         - memdump 00000000FF613570,50
                        
                  00000000FF613570         4883EC28E807FEFFFF4883C428EB1190        H?�(����H?�(�.
                  00000000FF613580         90909090909090909090909090909090        ................ 
                  00000000FF613590         488974240848897C241041544881ECB0        H?t$H?|$ATH��� 
                  00000000FF6135A0         0000008364242000488D4C2440FF157D        ...?d$ .H�L$@�} 
                  00000000FF6135B0         8C00009065488B042530000000488B78        ?...eH?%0...H?x
        
        - memdump {cip},50         {cip}= rip address
         
        -varx str,address,{cip}
         varx str,size,20
         memdump $address,$size
         
```
### 6- VarxClear : 
clear all variable's rest variables list , so we can add again same var 
Good for maintenance.
``` 
   sample :
         - VarxClear           
```
////////////////////////////////////////////////////////////////////////////
## Log Section:

### 1- LogxJustAtBP: 
```
Enable user to Log Just At BP or HW-BP so the Log window will contain the Value's you log, nothing else.
ofc you have to set the Log at Edit BreakPoint Dialog,Like this:
```
![bp_box](https://user-images.githubusercontent.com/7176580/49206425-787a6d00-f3cb-11e8-91fb-7b961eef9853.png)
```
using:
      LogxJustAtBP true,true  /// Log at BreakPoints on both Log window ( x64dbg and LogxWindow)
      LogxJustAtBP false,false /// Log as defaults 
      you can use another way to set the arguments Like :
      LogxJustAtBP on,on     or    LogxJustAtBP 1,1
      LogxJustAtBP off,off     or    LogxJustAtBP 0,0
```
**note: use the last version of x64dbg.**

//////////////////////////////////

### 2- LogxTemplateManager: 
```
Show Template window which enable u to define many template you want to used it later at BP or At tracer.
it anable you to save and rename and delete template edit Template.
note : Template Folder will be in the x64dbg folder root as LogTemplate.
No parameters needed.
```
![logxtemplatemanager](https://user-images.githubusercontent.com/7176580/49510131-c6e2ac80-f8a0-11e8-8594-421f880e829b.png)

//////////////////////////////////

### 3- logxwindow: 
```
Show Additional log window which log data from Logx Function it just for help to see log
when u want without extra info .
No parameters needed
```
![logxwindow](https://user-images.githubusercontent.com/7176580/49495151-7f95f500-f87b-11e8-867d-33458678005f.png)

//////////////////////////////////

### 4- logx: 
```
useding to log by defined Template so you can used at command line direct, or you can put it at BP dialog to 
log as defined Template .
usage:
      logx TemplateName     /// Template name should define in LogxTemplateManager      
```
![logx](https://user-images.githubusercontent.com/7176580/49495460-78231b80-f87c-11e8-9cef-6c66e118f0df.png)

//////////////////////////////////

### 5- logxTrace: 
```
this used as a Tracer to log as defined template using step triger ( F7 , F8 )
it will log direct to a file at LogTrace Folder as the name it define.
note : Trace Folder will be in the x64dbg folder root as LogTrace.
usege :
logxTrace P1,P2,P3
      P1 : on or off // (on:Enable tracer, need to enter (P2 & P3) , (off:disable Tracer , no need P2,P3).
      P2: Template Name should be define in LogxTemplateManager.
      P3: Trace file name ( it will created in LogTrace folder and it will be as Text file).
sample:
logxTrace off
logxTrace on,Template0,test
```
////////////////////////////////////////////////////////////////////////////
## Functions Section:

### 1- strcompx : 
```
used at the command of BP dialog to get the string at the address of memory , this Function use 
Contains compare so no need to write all string it's enough to write part of it and function will
find it.
```
![strcompx](https://user-images.githubusercontent.com/7176580/49727096-e73bae00-fc88-11e8-9548-d97e1057d692.png)
```
Parameter:
StrCompx P1, P2 , P3 , P4
      P1: resume after hit the BP (true/false - on/off - 1/0). 1: contiue , 0: stop
      P2: log Template name ,it should defined in the LogxTemplateManager.
      P3: memory Address which hold the string if it's number then no problem, if it's string 
            (like [rsp+4]) it should follow x64dbg string format 
 ```
![string Formatting](https://x64dbg.readthedocs.io/en/latest/introduction/Formatting.html?highlight=string)          
```
      P4: UserString2Compare u can put any string you want but it should not contain ,
sample:  
 StrCompx 0,t1,{[esp+4]},x32dbg   // t1 name of the template
 StrCompx 1,Template0,{rdx},MSCTFIME UI
```
//////////////////////////////////
