# Variables

The scope and shadowing

```oxiida
x = 5;
print x;

para {
    seq {
        x = x + 1;
        print x;
    }    

    seq {
        x = x + 1;
        print x;
    }    
}

print x;
```
