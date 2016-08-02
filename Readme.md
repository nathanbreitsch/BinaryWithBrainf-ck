
# Binary to Decimal Conversion with Brainf*ck

The following is a program, written in brainfuck, to translate binary numbers into decimal numbers.  If you are unfamiliar with brainfuck, check out the following [tutorial](https://learnxinyminutes.com/docs/brainfuck/).  

The input is limited to numbers which have single-digit decimal representation since the output is contained in a single brainfuck cell and is interpreted as an ascii character.  If you have an ascii table and are willing to subtract 48 from something, then the program will work for inputs up to 255 decimal, or the maximum value of a brainfuck cell.

``` brainfuck
-+ [ - < + ] ->, [ + [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>++ [ - < + ] ->>]+ [ - < + ] ->>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>]+ [ - < + ] ->------------------------------------------------+ [ - < + ] ->>>[-]+ [ - < + ] ->[-+ [ - < + ] ->>++ [ - < + ] ->>>++ [ - < + ] ->]+ [ - < + ] ->[-]+ [ - < + ] ->>>[-+ [ - < + ] ->++ [ - < + ] ->>>]+ [ - < + ] ->,]+ [ - < + ] ->>+ [ - < + ] ->>++++++++++++++++++++++++++++++++++++++++++++++++.
```

The code you see above works!  If you don't believe me, you can test it in this [interpreter](https://www.nayuki.io/page/brainfuck-interpreter-javascript).

It is a well accepted best practice not to write brainfuck directly (or at all, for that matter).  Therefore, we build up a bunch of reusable strings with haskell.

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


```haskell
readDoWrite action readIndex outIndex finalTransform = 
    goto readIndex ++ ", [ " ++ action ++ goto readIndex ++ "," ++ "]" ++ 
    goto outIndex ++ finalTransform ++ "."
```


```haskell
double addr ss1 ss2 = copyTo addr ss1 ss2 ++ addTo ss1 addr ss2
```


```haskell
fromAscii x = goto x ++ replicate 48 '-'
toAscii x = goto x ++ replicate 48 '+'
```


```haskell
home = 0
inputCell = 1
outputCell = 2
safeSpace1 = 3
safeSpace2 = 4
```

## The Final Program:


```haskell
"-" ++ readDoWrite (
        double outputCell safeSpace1 safeSpace2 ++
        fromAscii inputCell ++ 
        addTo inputCell outputCell safeSpace1
    ) 1 2 (toAscii outputCell)
```


    "-+ [ - < + ] ->, [ + [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>++ [ - < + ] ->>]+ [ - < + ] ->>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>]+ [ - < + ] ->>>>[-]+ [ - < + ] ->>>[-+ [ - < + ] ->>++ [ - < + ] ->>>>++ [ - < + ] ->>>]+ [ - < + ] ->>>[-]+ [ - < + ] ->>>>[-+ [ - < + ] ->>>++ [ - < + ] ->>>>]+ [ - < + ] ->------------------------------------------------+ [ - < + ] ->>>[-]+ [ - < + ] ->[-+ [ - < + ] ->>++ [ - < + ] ->>>++ [ - < + ] ->]+ [ - < + ] ->[-]+ [ - < + ] ->>>[-+ [ - < + ] ->++ [ - < + ] ->>>]+ [ - < + ] ->,]+ [ - < + ] ->>+ [ - < + ] ->>++++++++++++++++++++++++++++++++++++++++++++++++."

