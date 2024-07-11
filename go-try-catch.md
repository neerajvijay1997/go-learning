## Use of defer and recover

https://gobyexample.com/recover
https://go.dev/blog/defer-panic-and-recover


```go
package main

import (
	"fmt"
	"time"
)

// functionD simulates a function that does not panic
func functionD() {
	fmt.Println("Running functionD")
	// Simulating some work
	time.Sleep(500 * time.Millisecond)
}

// functionC simulates a function that might panic
func functionC() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in functionC:", r)
		}
	}()

	fmt.Println("Running functionC")
	functionD()
	panic("error in functionC")
}

// functionB calls functionC in a goroutine and handles its errors
func functionB() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in functionB:", r)
		}
	}()

	fmt.Println("Running functionB")
	go functionC()
}

// functionA is the main function that starts the goroutines
func functionA() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in functionA:", r)
		}
	}()

	go functionB()
}

func main() {
	functionA()
	// Sleep to allow all goroutines to finish before main exits
	time.Sleep(10 * time.Second)
	fmt.Println("Main still executing")
}


```

Error handling with channels

```go

package main

import (
	"fmt"
	"time"
)

// functionD simulates a function that does not panic
func functionD() {
	fmt.Println("Running functionD")
	// Simulating some work
	time.Sleep(500 * time.Millisecond)
}

// functionC simulates a function that might panic
func functionC(errCh chan<- error) {
	defer func() {
		if r := recover(); r != nil {
			errCh <- fmt.Errorf("panic in functionC: %v", r)
		}
	}()

	fmt.Println("Running functionC")
	functionD()
	panic("error in functionC")
}

// functionB calls functionC in a goroutine and handles its errors
func functionB(errCh chan<- error) {
	defer func() {
		if r := recover(); r != nil {
			errCh <- fmt.Errorf("panic in functionB: %v", r)
		}
	}()

	fmt.Println("Running functionB")
	cErrCh := make(chan error)
	go functionC(cErrCh)

	if err := <-cErrCh; err != nil {
		errCh <- err
	}
}

// functionA is the main function that starts the goroutines
func functionA() {
	errCh := make(chan error)

	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in functionA:", r)
		}
	}()

	go functionB(errCh)

	if err := <-errCh; err != nil {
		fmt.Println("Error received in functionA:", err)
	}
}

func main() {
	functionA()
	// Sleep to allow all goroutines to finish before main exits
	time.Sleep(10 * time.Second)
	fmt.Println("Main still executing")
}
```
