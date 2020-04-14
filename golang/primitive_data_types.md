---
layout: page
title: Introduction
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

