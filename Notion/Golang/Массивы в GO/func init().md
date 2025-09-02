init() вызывается перед main() и обычно для инциализации пакетов. При инициализации каждого из пакетов перед этим выполняется init()
```go
package main

import "fmt"

var msg string


func init() {
	msg = "got from init"
}

func main() {
	fmt.Println(msg)
} 
//got from init
```