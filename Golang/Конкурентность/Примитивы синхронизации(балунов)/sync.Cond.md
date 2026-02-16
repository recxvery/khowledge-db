[[Waitgroup'ы]]
[[race condition, атомики,  мьютексы]]
[[Once()]]


**sync.Cond()** - примитив синхронизации, обеспечивающий блокирование одного или несколько потоков до момента поступления от другого потока о выполнении некоторого уровня.

![[Pasted image 20260216150857.png]]

### publisher/subsrciber

```go
package main

import (
	"log"
	"sync"
	"time"
)

func subscribe(name string, data map[string]string, c *sync.Cond) {
	c.L.Lock()

	for len(data) == 0 {
		c.Wait()
	}

	log.Printf("[%s] %s\n", name, data["key"])

	c.L.Unlock()
}

func publish(name string, data map[string]string, c *sync.Cond) {
	time.Sleep(time.Second)

	c.L.Lock()
	data["key"] = "value"
	c.L.Unlock()

	log.Printf("[%s] data publisher \n", name)
	c.Broadcast()
}

func main() {
	data := map[string]string{}
	cond := sync.NewCond(&sync.Mutex{})

	wg := sync.WaitGroup{}
	wg.Add(3)

	go func() {
		defer wg.Done()
		subscribe("subcriber_1", data, cond)
	}()

	go func() {
		defer wg.Done()
		subscribe("subcriber_2", data, cond)
	}()

	go func() {
		defer wg.Done()
		publish("publisher", data, cond)
	}()


	wg.Wait()
}

```

Что происходит? Иницииализируем пустую мапу, потом захватываем мьютекс в `subscribe()`, после чего проверяем условие, условие неверное ===> отпускаем мьютекс и засыпаем(всё это происходит в `Wait()`), когда его разбудят он снова возьмёт мьютекс.

Далее код переходит к `publisher()` он захватывает мьютекс, записывает данные и методом `c.Broadcast()` будит все горутины. Оба `subscriber'a` просыпаются, проверяют условие в `for`, выходят из цикла, мьютекс захваченный `Wait()` анлочится, значения выводятся на экран.


![[Pasted image 20260216154554.png]]

во второй строчке переходим в режим ожидания, в четвёртом будимся.

### Это на разбор

```go
func waitWithLock() {
	cond := sync.NewCond(&sync.Mutex{})
	cond.Wait()
}

func waitAfterSignal() {
	cond := sync.NewCond(&sync.Mutex{})

	cond.Signal()

	time.Sleep(100 * time.Millisecond)

	cond.L.Lock()
	cond.Wait()
	cond.L.Unlock()
}

func main() {
	waitWithLock()
}

```

я не в состоянии понять что балунов хотел показать здесь. Что то по типу отличия от каналов но энивэй непонятно.