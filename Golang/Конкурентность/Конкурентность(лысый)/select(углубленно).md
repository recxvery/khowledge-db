В первую очередь - блокирующий оператор.

Вот например в этом коде:
```go
package main

func main() {
	select{}
}
```
У `select'а` нету никаких условий и прочего. Он просто вечно будет заставлять нашу горутину ждать, GO не дурак и просто приведёт это к `deadlock'у`

Можно применять `time.After()`, чтобы подождать какое-то время горутинку
```go
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- 1
	}()

	select {
	case v := <-ch1:
		fmt.Println("v := ", v, "first chanel")
	case v := <-ch2:
		fmt.Println("v := ", v, "second chanel")
	case <- time.After(1 * time.Second):
		fmt.Println("time out")
	}
```