---
layout: page
title: Collections
category: golang
permalink: /golang/collections
chapter: 3
---

* ***Inbuilt types***:
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
* They are built on top of Arrays.

```
package main

import (
	"fmt"
)


func main() {
	
	slice := []int{1,2,3}
	fmt.Println(slice)
	
	// equivalent code underhood that the Go compiler is doing
	/*
	arr2 := [3]int{1,2,3}
	fmt.Println(arr2)
	slice := arr2[:]
	fmt.Println(slice)
	*/
}

```

* Adding to slice using ***append***

```
package main

import (
	"fmt"
)

func main() {
	
	slice := []int{1,2,3}
	fmt.Println(slice)
	
	slice = append(slice, 4)
	slice = append(slice, 5,6,7)
	fmt.Println(slice)
}
``` 

* Other uasage with colon operator:

```
package main
import (
	"fmt"
)
func main() {
	slice := []int{1,2,3}
	fmt.Println(slice)
	
	slice = append(slice, 4)
	slice = append(slice, 5,6,7)
	fmt.Println(slice)
	
	s2 := slice[1:] // starting with first index till the end
	fmt.Println(s2)
	s3 := slice[2:] // starting with second index till the end
	fmt.Println(s3)
	s4 := slice[:3] // starting from first index till third index
	fmt.Println(s4)
    s5 := slice[2:5] // starting from first index till third index
	fmt.Println(s5)  
}
```

### Maps

* Simple declaration:

```
package main
import (
	"fmt"
)
func main() {
	m := map[string]int{"foo":25}
	fmt.Println(m["foo"])
	m["foo"] = 20	// re-assgining value to the key
	fmt.Println(m["foo"])

    delete(m, "foo") // deleting an element
    fmt.Println(m)
}
```


### Struct
