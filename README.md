# kdb-Default-Function-Args

### Who says kdb+ functions can't have default function arguments?

Library to provide default args functionalities when assigning Q functions. 

Note that this isn't meant to be quicker than how one would project kdb+ functions normally. Given the relatively lower costs of parsing the query versus the actual running of functions, this script serves to make it easy for those kdb+ end-users more comfortable with alternative languages such as Python etc... 
<br><br/>
### A) Installation instructions
```
1) Copy the F.k and F.q_ over to QHOME directory
2) Once completed, can run the following examples under the F) namespace
```

### B) Examples under the F) namespace

#### Defining and assigning a function

Default arguments are specified in the "args = val" format. One must make sure the various default arguments have been specified on its corresponding argument position, else the parser would flag an error and prematurely terminate!
```
q)F)b:a:{[a;b;c;d] a+b+c+d}[1][;3;d=4]
{[a;b;c;d] a+b+c+d}[1;;3;d=4]
```
Note that the parser has no issues handling multiple assignments within a single command line.
```
q)F)a:b:{x+y};c:`b[x=1] 
{x+y}[x=1;]

q)a
{x+y}
```

Note that the parser has no issues wrapping functions, i.e. those that haven't been set with default arguments BEFORE, with the default args facility. However, ONCE set with default arguments, the functions cannot be re-wrapped again, until resolved with `function[]`, else the `'nyi`error shown below would occur. This is __PRIMARILY__ for __CONSISTENCY__ reasons, especially since each projection is a logical extension of the values of each argument

```
q)F)a:b:{x+y};c:`b[x=1] 
{x+y}[x=1;]

q)d:c[1;]
q)d
locked[`u#``x!(();1);{x+y};`x`y]enlist[1;]

q)F)d:c[1;]       
'nyi
  [3]  (.F.chkFnType)

  [2]  (.F.chkFnAssign)

q)d:c[1;2] 
3
q)F)d:c[1;2] 
'nyi
  [3]  (.F.chkFnType)

  [2]  (.F.chkFnAssign)

q)F)d:c[x=1]
'nyi
  [3]  (.F.chkFnType)

  [2]  (.F.chkFnAssign)
```

Note the parser is unable to handle anything involving composite methods for now (as shown below), since the parser is looking specifically for `a:val`, and kdb existing parser doesnt support `a:'[{x+y}[x=1];enlist]` anyway
```
q)F)a:'[{x+y}[x=1];enlist]
'x
  [0]  (.F.e)
```

#### Example of specifying the wrong default positional argument

The parser recognises that the `a=1` position is incorrectly placed on the `b` argument position, prematurely terminating after flagging an error message 

```
q)F)c:{[a;b;c;d]a+b+c+d}[1][a=1;b=1][2]
'Default Args Wrong: `a should be `b
  [3]  (.F.genDefDict)

  [2]  (.F.newAssign)
```

#### Checking the function information upon assignment

To retrieve the original function definition in q, one can use the __.F.pprint__ function (when outside the F) namespace):

```
q)F)b:a:{[a;b;c;d] a+b+c+d}[1][;3;d=4]
{[a;b;c;d] a+b+c+d}[1;;3;d=4]

q).F.pprint `a     // Another acceptable format is ".F.pprint a"
{[a;b;c;d] a+b+c+d}[1;;3;d=4]
```

It has been set to __auto__ pretty-print by default under the F) namespace too:
```
q)F)a   
{[a;b;c;d] a+b+c+d}[1;;3;d=4]
```

Note that to set the output from the F.pprint as string, instead of the std-out being observed, one can change the variable `.F.stringMode` to `1b`. Observe the results below:

```
q)F)a:{[tab;no] select from tab where i>no}[tab=t]
{[tab;no] select from tab where i>no}[tab=+(,`x)!,0 1 2 3 4 5 6 7 8 9;]

q).F.stringMode: 1b

q)F)a:{[tab;no] select from tab where i>no}[tab=t]
"{[tab;no] select from tab where i>no}[tab=+(,`x)!,0 1 2 3 4 5 6 7 8 9;]"
```

#### Examples of how dynamic the functions are in interpreting the number of arguments against the default arguments 
```
q)F)b:a:{[a;b;c;d] a+b+c+d}[1][;3;d=4]
{[a;b;c;d] a+b+c+d}[1;;3;d=4]

q)a[1]      // Project b=1, Use default value d=4
9

q)a[1;2]    // Project values b=1 and d=2, don't use default values
7

q)a[]       // Use default value d=4, Await projection of b value (since no default b value)
{[a;b;c;d] a+b+c+d}[1;;3;][;4]
```

#### More complicated examples
```
q)F)c:{[a;b;c;d]a+b+c+d}[1][b=1;;d=4][2]
{[a;b;c;d]a+b+c+d}[1;b=1;2;d=4]

q)c[]        //Use default values of b=1 and d=4
8

q)c[::;1]    //Use default value b=1, Specify d=1 instead of default d=4
5

q)c[;1][]    //Project value d=1, Use default value b=1
5 

q)c[;1][3]   //Project value d=1, then project value b=3
7 

q)c[;1]      //Project value d=1, and it would await b value to be projected
locked[`u#``b`d!(();1;4);{[a;b;c;d]a+b+c+d}[1;;2;];`b`d]enlist[;1]

q)c[;1][2]   //Project value d=1, then project value b=2
6

q)c[;]       //To avoid triggering default values b=1 & d=4
locked[`u#``b`d!(();1;4);{[a;b;c;d]a+b+c+d}[1;;2;];`b`d]enlist[;]
q)c[;][4;5]
12
q)c[;][4][5]
12
q)c[;][;5][4]
12
```

An interesting case would be using select within functions itself, which illustrates the parser's ability to dynamically handle all kinds of default arguments within the function assignment itself
```
q)t:([] til 10)

q)F)a:{[tab;no] select from tab where i>no}[tab=t]
{[tab;no] select from tab where i>no}[tab=+(,`x)!,0 1 2 3 4 5 6 7 8 9;]

q)a[]  
{[tab;no] select from tab where i>no}[+(,`x)!,0 1 2 3 4 5 6 7 8 9;]

q)a[][1]
x
-
2
3
4
5
6
7
8
9

q)a[([] a: til 5);2]
a
-
3
4
```

#### To always use default function arguments, one can use [] first before projecting other required arguments in
```
q)fn[] 
```

### C) Work in Progress

```
a) Getting it to pretty-print by default for all kinds of parse trees
b) Thinking of a consistent logic by which one can re-wrap functions, that doesnt require discarding all prior default argument settings
```
