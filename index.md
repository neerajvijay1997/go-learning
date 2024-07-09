### Abstract class in go
In Go, there's no direct support for abstract classes as found in some other object-oriented languages like Java or C#. However, you can achieve similar behavior using interfaces and embedding. Here’s how you can create an abstract class-like structure in Go:

Define an interface: This will act as the abstract class.
Create a struct: This struct can include common methods and fields.
Embed the struct in other structs: This allows other structs to inherit common behavior.
```
package main

import "fmt"

// Abstract interface
type Animal interface {
	Speak() string
}

// Base struct
type BaseAnimal struct {
	name string
}

// Common method
func (b *BaseAnimal) GetName() string {
	return b.name
}

// Concrete struct embedding the BaseAnimal
type Dog struct {
	BaseAnimal
}

// Implement the Speak method
func (d Dog) Speak() string {
	return "Woof!"
}

// Another concrete struct embedding the BaseAnimal
type Cat struct {
	BaseAnimal
}

// Implement the Speak method
func (c Cat) Speak() string {
	return "Meow!"
}

func main() {
	dog := Dog{BaseAnimal{name: "Buddy"}}
	cat := Cat{BaseAnimal{name: "Whiskers"}}

	animals := []Animal{dog, cat}

	for _, animal := range animals {
		fmt.Printf("%s says %s\n", animal.(*BaseAnimal).GetName(), animal.Speak())
	}
}

```


### try catch

```
func catchme() {
    panic("oh no!")
}

func main() {
    defer func() {
        if err := recover(); err != nil {
            log.Println("panic occurred:", err)
        }
    }()

    catchme()
}
```

Example handling error in go routine
```
package main

import (
	"fmt"
	"time"
)

func worker(id int, errChan chan<- error) {
	defer func() {
		if r := recover(); r != nil {
			errChan <- fmt.Errorf("worker %d: %v", id, r)
		}
	}()
	
	// Simulate work that may panic
	if id == 1 {
		panic("something went wrong")
	}
	
	// Simulate work
	time.Sleep(time.Second)
	fmt.Printf("Worker %d done\n", id)
	errChan <- nil
}

func main() {
	errChan := make(chan error, 2) // Buffered channel to prevent goroutine blocking

	for i := 0; i < 2; i++ {
		go worker(i, errChan)
	}

	for i := 0; i < 2; i++ {
		if err := <-errChan; err != nil {
			fmt.Printf("Error: %v\n", err)
		}
	}
	
	fmt.Println("Main thread continues running")
}

```





### Non blocking send to channel
```
select {
case n.completedObjectivesForRPC <- completed.Id():
default:
}
```
This pattern is useful when you want to send a notification or a value to another part of your program but don’t want to wait if the recipient is not ready to receive it. This can help avoid deadlocks and keep the program responsive.

Non-Blocking Send: The key feature here is the use of the default case. It ensures that if no goroutine is currently ready to receive from n.completedObjectivesForRPC, the send operation will not block the execution of the current goroutine. Instead, it will immediately proceed to the default case, effectively discarding the value to be sent.

<https://gobyexample.com/non-blocking-channel-operations>
<https://stackoverflow.com/questions/48953236/how-to-implement-non-blocking-write-to-an-unbuffered-channel>

Example code
```
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int)
	go func() {
		for {
			select {
			case val := <-ch:
				fmt.Println("Received:", val)
			default:
				// Do some work if there's nothing to receive
				fmt.Println("No value received")
				time.Sleep(500 * time.Millisecond)
			}
		}
	}()

	for i := 0; i < 5; i++ {
		select {
		case ch <- i:
			fmt.Println("Sent:", i)
		default:
			fmt.Println("Channel is full, discarding:", i)
		}
		time.Sleep(300 * time.Millisecond)
	}
}

```






### Understanding string()
if got is byte of some struct then does string(got) convert it to string representation in go 

In Go, the string() conversion function can be used to convert a byte slice (or an array of bytes) into a string. However, it's essential to understand how this conversion works and its limitations when dealing with structured data.
When you have a byte slice that represents some structured data (e.g., a struct that has been serialized into bytes), using string(got) will convert the raw bytes into a string directly. This means that the conversion will interpret the byte values as UTF-8 encoded characters and create a string based on those byte values.

```go
package main

import (
    "fmt"
)

func main() {
    // Example byte slice (could be from a serialized struct)
    got := []byte{72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100}
    
    // Convert byte slice to string
    str := string(got)
    
    // Print the result
    fmt.Println(str) // Output: "Hello World"
}

```
In this example, the byte slice got contains the ASCII values of the characters "Hello World". When we use string(got), it converts these byte values into the corresponding string representation.
However, if got represents a serialized struct (e.g., a struct converted to a byte slice using a binary encoding like Gob, JSON, or another format), converting it directly to a string will not give you a meaningful string representation of the original struct. Instead, you'll get a string with potentially non-printable characters that represent the binary data.
```go
package main

import (
    "bytes"
    "encoding/gob"
    "fmt"
)

// Define a struct
type MyStruct struct {
    Name string
    Age  int
}

func main() {
    // Create an instance of the struct
    original := MyStruct{Name: "Alice", Age: 30}
    
    // Serialize the struct to bytes
    var buffer bytes.Buffer
    encoder := gob.NewEncoder(&buffer)
    err := encoder.Encode(original)
    if err != nil {
        fmt.Println("Error encoding struct:", err)
        return
    }
    
    // Get the serialized byte slice
    got := buffer.Bytes()
    
    // Convert byte slice to string
    str := string(got)
    
    // Print the result
    fmt.Println(str) // Output: (garbled binary data, not human-readable)
}

```
In this example, the got byte slice contains the serialized form of MyStruct. Converting it to a string using string(got) will not produce a meaningful, human-readable string. Instead, it will be a string of garbled characters because the byte slice represents binary-encoded data, not UTF-8 encoded text.

Key Points:
Text Data: If the byte slice contains text data (e.g., ASCII or UTF-8 encoded characters), string(got) will convert it to a readable string.
Binary Data: If the byte slice contains binary data (e.g., serialized structs), string(got) will produce a string of garbled characters.
For structured data, you typically need to use the appropriate decoding method to convert the byte slice back to its original form, rather than converting it directly to a string.
