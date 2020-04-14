---
layout: page
title: Primitive Data Types
category: golang
permalink: /golang/primitives
chapter: 2
---

## Variable Declaration

Sample code ilustrating different ways of initializing variables:

```
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello, playground")
	// 1. declare and assign value later
    var i int
	i = 42
	fmt.Println (i)
	
    // 2. declare and assign
	var f float32 = 3.14
	fmt.Println(f)

    // 3. := means aasume the date type based on the value assigned
	firstName := "anirbaan"
	fmt.Println(firstName)

    //4. boolean
    b = true
    fmt.Println(b)

    //5. complex data type
    c:= complex(3,4)
	fmt.Println(c)
	
    //6. real and imaginary numbers
	r,im:= real(c), imag(c)
	fmt.Println(r)
	fmt.Println(im)
}
```

## Pointers

Points to an address in memory. Can hold a value.

```
package main

import (
	"fmt"
)

func main() {
	var firstName *string
	fmt.Println(firstName)
}

> <nil>

package main

import (
	"fmt"
)

func main() {
	var firstName *string
    firstName = "arthur"
	fmt.Println(firstName)
}

> ./prog.go:9:14: cannot use "arthur" (type string) as type *string in assignment
```

deferencing a pointer: reaching through the pointer, grabbing the data and giving it back.

```
package main

import (
	"fmt"
)

func main() {
	var firstName *string
    *firstName = "arthur"
	fmt.Println(firstName)
}
> panic: runtime error: invalid memory address or nil pointer dereference
```
correcting the above by creating a new string object:

```
package main

import (
	"fmt"
)

func main() {
	var firstName *string = new(string)
	*firstName = "arthur"
	// address
	fmt.Println(firstName)
	// value
	fmt.Println(*firstName)
}
> 0x40c138
arthur
```

* Address of operator:
```
package main

import (
	"fmt"
)

func main() {
	firstName := "name"
	fmt.Println(firstName)
	
	// address of operator 
	ptr := &firstName
	fmt.Println(ptr, *ptr)
	
	//changing the value at the address
	firstName = "lastname"
	fmt.Println(ptr, *ptr)
}
> name
0x40c138 name
0x40c138 lastname
```
