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

### Comments in Go
Single line using ***//***
Multi-line using ***/**/***

### Code formnatting
Standard formatting. Like Tabs.
However here are some that are strongly enforced.
```
func main() 
{
	fmt.Println("Hello, playground")
}

Error: ./prog.go:7:6: missing function body
./prog.go:8:1: syntax error: unexpected semicolon or newline before {

```

### Starting a project

* Downloading Go on Mac: https://golang.org/doc/install?download=go1.14.2.darwin-amd64.pkg
* Verification: go version
```
go version
go version go1.14.2 darwin/amd64
```

* Primary go commands can be listed by:
```
go
Go is a tool for managing Go source code.

Usage:

	go <command> [arguments]

The commands are:

	bug         start a bug report
	build       compile packages and dependencies
	clean       remove object files and cached files
	doc         show documentation for package or symbol
	env         print Go environment information
	fix         update packages to use new APIs
	fmt         gofmt (reformat) package sources
	generate    generate Go files by processing source
	get         add dependencies to current module and install them
	install     compile and install packages and dependencies
	list        list packages or modules
	mod         module maintenance
	run         compile and run Go program
	test        test packages
	tool        run specified go tool
	version     print Go version
	vet         report likely mistakes in packages
```

* Use ***go help <cmd>*** to know about a subcommand.

* Using the ***doc*** command
```
go doc json
go doc json.Decoder
// we can keeping digging deeper to sub-objects
```

* comments in source code will be pulled up for display when we use the ***doc*** command.


#### Setting up Visual Studio code for Go Development
1. Install Go from package manager :  ms-vscode.go
2. Mac -> cmd+shift+p -> Go:Install/Update -> check all and install
3. succesful installation gives a message -> All tools successfully installed. You are ready to Go :).

Sample project to test our setup:

```
package main

import "fmt"

func main() {
	fmt.Println("Hello !! Gophers....")
}

> go run main.go
```
Note: The compiled binary was not left behind.


### Creating a Project
Modules are how projects are organized. Earlier these were called workspaces.

* Initializing a module:
- Module is a directory having go.mod file.
- mod file contains configuration and information about the module itself.

```
go mod init github.com/sample/webservice
```

Now lets add the sample main.go we had used earlier.
Since we now have module setup we don't need use the ***run file.go***

Instead:

```
go run github.com/sample/webservice
Hello!! Gophers....
```

This would be helpful when we have multiple folders and save us from navigating.


