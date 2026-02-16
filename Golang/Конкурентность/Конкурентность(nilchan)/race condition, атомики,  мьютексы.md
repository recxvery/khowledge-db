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

### Waitqueue
**Wait queue** — это внутренняя очередь горутин, которые не смогли захватить mutex и ждут его освобождения.

![[Pasted image 20260209034725.png]]

Важное уточнение мьютекс не блокирует поток, горутины паркуются runtime'м и снимаются с выполнения. Куда снимаются горутины? Наглядно:

![[Pasted image 20260211155936.png]]

То есть мьютекс содержит очередь ждущих горутин.
Грубо говоря структура мьютекса ниже:

```
struct {
    state
    waiters
}
```
##### Normal mode

![[Pasted image 20260211163911.png]]

G1 - разблокируется мьютекс. Первая горутина в очереди(FIFO) пойдёт захватывать мьютекс, пока она будет менять состояние `ready`/`running` и т.д. G2 может прийти и успеть захватить этот мьютекс. и наша горутина которая хотела захватить мьютекс отправится в конец очереди.
В normal mode mutex не гарантирует строгий FIFO.  
После Unlock новый goroutine может захватить lock быстрее ожидающих.
#### Starvation mode

![[Pasted image 20260212152041.png]]

Когда первая горутина разблокировывает мьютекс она передаёт захват первой горутине в Waitqueue чтобы горутины не голодали.
### RWMutex

RWMutex — это расширение обычного Mutex, которое позволяет нескольким горутинам одновременно читать данные, но сохраняет эксклюзивность при записи.

Суть в том что он впускает сразу много читателей(обычный мьютекс блокирует после того как входит одна горутина неважно на чтение/запись). При этом если у нас один писатель, `RWMutex` в этом плане будет работать как обычный мьютекс.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var likes int = 0
var mtx sync.RWMutex

func setLike(wg *sync.WaitGroup) {
	defer wg.Done()
	mtx.Lock()
	likes++
	mtx.Unlock()
}

func getLike(wg *sync.WaitGroup) {
	defer wg.Done()

	for i := 1; i <= 100_000; i++ {
		mtx.RLock()
		_ = likes
		mtx.RUnlock()
	}
}

func main() {
	wg := &sync.WaitGroup{}

	initTime := time.Now()

	for i := 1; i <= 10; i++ {
		wg.Add(1)
		go setLike(wg)
	}

	for i := 1; i <= 10; i++ {
		wg.Add(1)
		go getLike(wg)
	}

	wg.Wait()

	fmt.Println(time.Since(initTime))

}

```

Без `Rlock'a` код будет работать - 124.8801ms, с - 40.1972ms.