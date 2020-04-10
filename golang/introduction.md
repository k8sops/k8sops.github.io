---
layout: page
title: Introduction
category: golang
permalink: /golang/intro
chapter: 1
---

## Basics

play.golang.org --> test website to run small snippets

```
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello, playground")
}

```

Unused package throws compilation errors

```
package main

import (
	"fmt"
    "os"
)

func main() {
	fmt.Println("Hello, playground")
}

./prog.go:5:2: imported and not used: "os"
Since "os" package is not being used.
```