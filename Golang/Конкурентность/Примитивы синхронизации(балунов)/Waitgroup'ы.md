[[Горутины и каналы]]
[[Планировщик(тузов)]]]


**Waitgroup'ы** - примитив синхронизации в GO, который позволяет одним или нескольким горутинам дождаться выполнения других горутин, используя внутренний счётчик активных задач. 

![[Pasted image 20260208144554.png]]

Супер базовый пример с `waitgroup'ой`:

```go
func main() {
	var wg sync.WaitGroup
	wg.Add(5)

	for i := 0; i < 5; i++ {
		go func() {
			defer wg.Done()
			log.Println("test")
		}()
	}

	time.Sleep(time.Second)
	wg.Wait()
}
```

счетчик `waitgroup'a` меньше 0 >>>:
```go
package main

import (
	"sync"
)

func negativeCounter() {
	wg := sync.WaitGroup{}
	wg.Add(-10)
}

func waitZeroCounter() {
	wg := sync.WaitGroup{}
	wg.Wait()
}

func main() {
	negativeCounter()
}


panic: sync: negative WaitGroup counter
```


**Когда мы используем любой примитив из пакета `sync`, не нужно его копировать.**

Хорошой практикой считается добавлять wg.Add() перед каждой горутиной, как например здесь, а не писать `wg.Add(3)`:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func postman(wg *sync.WaitGroup, text string) {
	defer wg.Done() //считается хорошей практикой.
	for i := 1; i <= 3; i++ {
		fmt.Println("Я почтальон,я отнёс газету", text, "в ", i, "раз")
		time.Sleep(250 * time.Millisecond)
	}

}

func main() {

	wg := &sync.WaitGroup{}

	wg.Add(1)
	go postman(wg, "News")

	wg.Add(1)
	go postman(wg, "Sport news")

	wg.Add(1)
	go postman(wg, "Auto news")

	wg.Wait()

	fmt.Println("main was done")
}

```


Сложный пример:

Здесь нужно использовать буферизованный канал + закрыть его.
```go
package main

import (
	"fmt"
	"sync"
)

var stringsArr = []string{
	"asddasdas",
	"dasdasads",
	"dassdaa",
	"hwt4",
}

func makeArray() ([]int, []int, []int, []int) {
	arr1, arr2, arr3, arr4 := make([]int, 0), make([]int, 0), make([]int, 0), make([]int, 0)
	for i := 1; i < 101; i++ {
		switch  {
		case  i <= 25:
			arr1 = append(arr1, i)
		case i > 25 && i <= 50:
			arr2 = append(arr2, i)
		case i > 50 && i <= 75:
			arr3 = append(arr3, i)
		case i > 75:
			arr4 = append(arr4, i)
		}
	}
	return arr1, arr2, arr3, arr4
}

func countSum(arr []int, wg *sync.WaitGroup, ch chan int )  {
	sum := 0
	for _, v := range arr {
		sum += v
	}
	ch <- sum
	wg.Done()

}

func main() {
	arr1, arr2, arr3, arr4 := makeArray()
	ch := make(chan int, 4)

	sum := 0
	wg := &sync.WaitGroup{}

	wg.Add(1)
	go countSum(arr1, wg, ch)

	wg.Add(1)
	go countSum(arr2, wg, ch)

	wg.Add(1)
	go countSum(arr3, wg, ch)

	wg.Add(1)
	go countSum(arr4, wg, ch)

	wg.Wait()
	close(ch)
	for v := range ch {
		sum += v
	}

	fmt.Println("main was done, sum:", sum)
}

```