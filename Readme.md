
# Binary to Decimal Conversion with Brainf*ck

The following is a program, written in brainfuck, to translate binary numbers into decimal numbers.  If you are unfamiliar with brainfuck, check out the following [tutorial](https://learnxinyminutes.com/docs/brainfuck/).  

The input is limited to numbers which have single-digit decimal representation since the output is contained in a single brainfuck cell and is interpreted as an ascii character.  If you have an ascii table and are willing to subtract 48 from something, then the program will work for inputs up to 255 decimal, or the maximum value of a brainfuck cell.

``` brainfuck
-+ [ - < + ] ->>>[-]++ [ - < + ] ->, [ + [ - < + ] ->------------------------------------------------[+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->[-]]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>>>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>>>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>]+ [ - < + ] ->,]+ [ - < + ] ->>+ [ - < + ] ->>++++++++++++++++++++++++++++++++++++++++++++++++.
```

The code you see above works!  If you don't believe me, you can test it in this [interpreter](https://www.nayuki.io/page/brainfuck-interpreter-javascript).

I do not recommend writing brainfuck directly (if at all) so we instead build up a bunch of reusable strings using haskell

![Brainfuck Best Practices](./generate.png)


```haskell
gohome = "+ [ - < + ] -"
```


```haskell
goto addr = gohome ++ replicate addr '>'
```


```haskell
zeroCurrentCell = "[-]"
```


```haskell
zeroCell addr = goto addr ++ zeroCurrentCell
```


```haskell
addToCurrentCell val = replicate val '+'
```


```haskell
setCell addr val = zeroCell addr ++ addToCurrentCell val
```


```haskell
-- warning: zeros cell at addr1
destrAddTo addr1 addr2 = goto addr1 ++ "[-" ++ goto addr2 ++ "+" ++ goto addr1 ++ "]"
```


```haskell
-- warning: zeros cell at addr1
destrMoveTo addr1 addr2 = zeroCell addr2 ++ destrAddTo addr1 addr2
```


```haskell
addTo addr1 addr2 safeSpace = 
    zeroCell safeSpace ++
    goto addr1 ++ "[-" ++ goto addr2 ++ "+" ++ goto safeSpace ++ "+" ++ goto addr1 ++ "]" ++
    destrMoveTo safeSpace addr1 --zeros cell at safe space
```


```haskell
copyTo addr1 addr2 safeSpace = zeroCell addr2 ++ addTo addr1 addr2 safeSpace
```

**multiply** and **power** turn out not to be nessesary, but were interesting to write.  Multiplication seemed to need two extra cells and exponentes seemed to require four.  It is probably possible to abstract the repeated application of a given operation.


```haskell
multiply addr1 addr2 addr3 ss1 ss2 = 
    zeroCell addr3 ++
    copyTo addr2 ss1 ss2 ++ zeroCell ss2 ++
    goto ss1 ++ "[-" ++ addTo addr1 addr3 ss2 ++ goto ss1 ++ "]"
```


```haskell
power addr1 addr2 addr3 ss1 ss2 ss3 ss4 = zeroCell addr3 ++ "+" ++
    copyTo addr2 ss1 ss2 ++ zeroCell ss2 ++
    goto ss1 ++ "[-" ++ multiply addr1 addr3 ss2 ss3 ss4 ++ 
        zeroCell ss3 ++ zeroCell ss4 ++
        destrMoveTo ss2 addr3 ++ goto ss1 ++ "]"
```


```haskell
readDoWrite action readIndex outIndex finalTransform = 
    goto readIndex ++ ", [ " ++ action ++ goto readIndex ++ "," ++ "]" ++ 
    goto outIndex ++ finalTransform ++ "."
```


```haskell
fromAscii x = goto x ++ replicate 48 '-'
toAscii x = goto x ++ replicate 48 '+'
```

- 0: Home
- 1: Read Register
- 2: Agg and Output
- 3: Current $2^n$


```haskell
"-" ++ setCell 3 1 ++ readDoWrite (fromAscii 1 ++ 
    "[" ++ addTo 3 2 safeSpace1 ++
        zeroCell 1 ++ "]" ++
        copyTo 3 4 safeSpace1 ++ addTo 3 4 safeSpace1 ++ destrMoveTo 4 3
    ) 1 2 (toAscii 2)
```


    "-+ [ - < + ] ->>>[-]++ [ - < + ] ->, [ + [ - < + ] ->------------------------------------------------[+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->[-]]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>>>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->>>>>>>>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>>>++ [ - < + ] ->>>>>>>>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>>>>>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>>>>>>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>]+ [ - < + ] ->,]+ [ - < + ] ->>+ [ - < + ] ->>++++++++++++++++++++++++++++++++++++++++++++++++."

