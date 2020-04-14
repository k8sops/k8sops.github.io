---
layout: page
title: Introduction
category: golang
permalink: /golang/collections
chapter: 3
---

* Inbuilt types:
1. Arrays: Fixed and indexed. And fixed type. All elements needs to be of same data type.
2. slices: dynamic size, indexed
3. Maps: Like slices but with key/value
4. structs: No classes in Go. Structs are similar to classes. They encapsulate an idea.


## Arrays
* Simple array declaration:
```
package main

import (
	"fmt"
)


func main() {
	var arr [3]int
	arr[0]= 100
	arr[1] = 300
	arr[2] = 500
	fmt.Println(arr)
    fmt.Println(arr[2])
}
> [100 300 500]
500

```

* ***Implicit*** Declaration:
```
arr2 := [3]int{1,2,3}	
fmt.Println(arr2)
```

### Slice
