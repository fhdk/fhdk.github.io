---
title: 'Killing process in MS Windows'
taxonomy:
    category:
        - docs
---

## Killing processes

Listing processes
```
PS C:\> tasklist 
```

Find process - by name
```
PS C:\> tasklist | findstr /C:cmd
```
Stop process by id
```
PS C:\> stop-process -Id 7628
```

Stop process by name
```
PS C:\> Stop-Process -name XYZ
```