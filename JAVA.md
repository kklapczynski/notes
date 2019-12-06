### Strings
- for string comparisions use only .equals or .equalsIgnoreCase; "==" checks memory location and separate strings are always separate locations in memory
    - "Hello".substring(0,3) == "Hel"  => returns false
- .length() returns number of code units ( not Unicode characters - some like emotikons are build from 2 code units)
- there is no method that mutates string - they always return a new one

### Numbers
- int and long for normal use
- BigInteger or BigDecimal objects for bigger numbers

### Structures
- "for each" loop:
```JAVA
    int[] a = new int[100];

    for (int element : a) {}
```

### Arrays
- array initialazers
```JAVA
int[] sP = {2,3,5,7,11,13};
```
- normal assignment copies reference not an array itself
```JAVA
int[] luckyNumbers = sP;    // changing value in one array chnages in the other as well
```
- to do true copy of array:
```JAVA
int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);
```