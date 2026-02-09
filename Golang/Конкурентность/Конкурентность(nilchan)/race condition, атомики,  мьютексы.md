**race condition** - состояние когда две или более горутины, обращаются к одному участку памяти, и хотя бы одна из них пишет.

![[Pasted image 20260209031248.png | 700]]

Атомарность в горутинах — это когда операция с данными выполняется **целиком за один шаг**, без возможности прерывания другими горутинами.

```go

var number atomic.Int64

func increase(wg *sync.WaitGroup) {
	defer wg.Done()
	
	for i := 1; i <= 1000; i++ {
		number.Add(1)
	}
}

func main() {

	wg := &sync.WaitGroup{}
	wg.Add(10)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)

	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	
	wg.Wait()
	fmt.Println("Number:", number.Load())
}

//>_: 10000
//>_:Number: 8399(без атомарности)
```

Атомики занимают больше ресурсов чем простые операции, поэтому их используют не так часто.

### Мьютекс

![[Pasted image 20260209034212.png]]

Mutural exclusion - взаимное исключение(**в рамках какого-либо кода, может зайти только одна горутина**).

На этом примере мьютекс грубо говоря блокирует другие горутины, пока не справится с операцией(добавление элемента в слайс), мы сначала лочим критическую секцию, а потом разблокируем, таким образом, мы выполняем команды последовательно, избегая состояние гонок

```go
package main

import (
	"fmt"
	"sync"
)

// var number atomic.Int64
var slice []int

var mtx sync.Mutex

func increase(wg *sync.WaitGroup) {
	defer wg.Done()

	for i := 1; i <= 1000; i++ {
		mtx.Lock()// goroutine ---> waiting
		slice = append(slice, i)
		mtx.Unlock()
	}
}

func main() {

	wg := &sync.WaitGroup{}
	wg.Add(10)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)

	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)
	go increase(wg)

	wg.Wait()
	fmt.Println("Slice len:", len(slice))
}

```

![[Pasted image 20260209034702.png | 670]]
![[Pasted image 20260209034725.png]]