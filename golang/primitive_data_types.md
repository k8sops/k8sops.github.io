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
* de-refrence operator: *firstName
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

* ***constants***
1. Cannot change their value like variables do.
2. Has to be declared & initialized in the same line.
3. Value of constants has to be available at compile time. Cannot have functions return values for constants.

```
package main

import (
	"fmt"
)

func main() {
	const pi = 3.14
	fmt.Println(pi)
	pi = 300
}
> cannot assign to pi
```
* ***implicitly typed*** constants.
```
package main

import (
	"fmt"
)

func main() {
	const pi = 3
	fmt.Println(pi + 3) // using pi as integer
	// bunch of other code
	fmt.Println(pi + 1.2) // using pi as floating type
}
> 6
4.2

Above will fail when we declare:
const pi int = 3 // now addition to float will fail.

And this how it can be handled:
fmt.Println(float32(pi) + 1.2)
```

### IOTA's and Cosntant Expressions:

* Constant blocks: Seprate section for declaring constants just like ***import***

```
package main

import (
	"fmt"
)

const ( 
pi = 3
second = 4
)

func main() {
	fmt.Println(second)
	fmt.Println(pi)
}
```
* ***iota***
Every time a iota is re-used it's value is incremented by 1.

```
package main

import (
	"fmt"
)

const( 
first = iota
second = iota
)

func main() {
	fmt.Println(first)
	fmt.Println(second)
	fmt.Println(first)
	fmt.Println(second)
}
> 0
1
0
1
```

* iota's can also be used in constant expressions. you can evolve the value of the constant in the constant block

```
package main

import (
	"fmt"
)

const( 
first = iota + 6
second
)

func main() {
	fmt.Println(first)
	fmt.Println(second)
}
> 6
7
```

* families of constants can be grouped using multiple constant blocks.

```
package main

import (
	"fmt"
)

const( 
first = iota + 6
second
)

const (
third = iota // here iota goes back to zero
)

func main() {
	fmt.Println(first)
	fmt.Println(second)
	fmt.Println(third)
}
>0
1
0
```