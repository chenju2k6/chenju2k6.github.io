---
layout: post
title:  "GoTee, Glamdring, Asylo"
date:   2019-07-16 23:00:28 -0500
categories: jekyll update
---

GoTee is a work done in EPFL and published in ATC'19. GoTee is a language-based execution model for TEEs (e.g. Intel SGX). This is different from the previous works. 

There are genenelly two categories of existing approaches of using SGX. The first one is to run the un-modified applications directly in the enclave. Haven and Graphene, for example, enables a OS and a library OS repectively inside the SGX. SCONE enables a container inside the enclave. Another approach is to partition the application's code and data into trusted and untrusted portions, "following the Intel SDK model". Intel SDK requires developers to define the interfaces between the secure and unsecure component. The secure and unsecure component are compiled and linked independently, the secure component is in a shared library form (.so), the untrusted application is a regular executable. Google's Asylo can be seen as an extended version of Intel SGX SDK. Glamdring automates the application partitioning by using static analysis. Existing works explores the mechanism to reduce the world switches, generally by employing asynchronous system calls. 


## Syntax

First of all, GoTee introduces a new keyword - "gosecure",  as shown in the below example. Simply put, the goroutine marked by "gosecure" will be compiled and run in the enclave. The GoTee will handle all the works underhood for the developers. 

```go

func HelloWorld(done chan bool) {
        fmt.Println("Hello World!")
        done <- true
}

func main() {
        done := make(chan bool)

        // A regular goroutine
        fmt.Println("From an untrusted domain:")
        go hw.HelloWorld(done)
        <-done

        // Now a secured routine
        fmt.Println("From a trusted domain:")
        gosecure hw.HelloWorld(done)
        <- done
}
``` 

The GoTee extends the Go compiler to include the keyword of "gosecure", 

In the file of "src/cmd/compile/internal/syntax/tokens.go", the "gosecure" has a name "_Gosecure". The "_Gosecure" is then treated as an operation with the opcode of "OGOSECURE".

```
case syntax.Gosecure:
	op = OGOSECURE
```


If the opcode is Gosecure, the `Gosecload` function will be called, see below code in "src/cmd/compile/internal/gc/ssa.go"

```go
case OGOSECURE:
   s.call(n.Left, callGosecure)
```

```go
case k==callGosecure
   call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, Gosecload, s.mem())
```

## Init and Load

Files:

* src/cmd/go/internal/work/gosec.go

The `CreateEnclaveExec` function is called at the compile time. It creates the temporaries files necessary to create the enclave executableu

The `CreateEnclaveExec` will register `gosec`,  the `gosec` will call `generateMain` function to create the main go file for the enclave executable. The filename is hardcoded as `encl.go`. The `encl.go` file will contain the following content

```
const (
	gosectmpl = `
package main
import(
	"gosecu"
	//"runtime"
	{{range .Imports}}
	{{ printf "%q" . }}{{end}}
)
func main() {
	// Starting the functions.
	{{range .Functions}}
	gosecu.RegisterSecureFunction({{ . }})
	{{end}}
	gosecu.EcallServer()
}
`
)
``` 


## Data movement

Data movement between secure and unsecure world is done by deep copying the data on the interface. The code is at "gotee/src/goseccommon/copy.go".
 
## Cooperative runtime

The cooperative runtime is defined in "gotee/src/runtime/gosec.go"

```go
type CooperativeRuntime struct {
	EcallSrv chan *EcallServerReq
	Ocall    chan OcallReq

	argc int32
	argv **byte

	readyE slqueue //Ready to be rescheduled in the enclave
	readyO slqueue //Ready to be rescheduled outside of the enclave

	//pool of answer channels.
	sysPool []*poolSysChan

	membuf_head uintptr

	StartUnsafe uintptr
	SizeUnsafe  uintptr

	// The enclave heap region.
	// This is the equivalent of my previous preallocated regions.
	// TODO secure it somehow.
	eHeap            uintptr
	Tcss             []SgxTCSInfo // array of tcs infos used in spawnThread
	Notes            [10]note     // notes for futex calls.
	OEntry           uintptr
	ExceptionHandler uint64

	Uach chan uintptr
}
```
The cooperative runtime is initialized in `InitCooperativeRuntime()`, which is called in `Gosecload`.
