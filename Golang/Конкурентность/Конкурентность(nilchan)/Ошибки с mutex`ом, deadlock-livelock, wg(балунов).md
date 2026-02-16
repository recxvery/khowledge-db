[[race condition, атомики,  мьютексы]]

[[Waitgroup'ы]]

## Оглавление
- [[#race condition + deadlock]]
- [[#Локальный мьютекс]]
- [[#defer + mutex]]
- [[#defer с мьютексом]]
- [[#Incorrect defer + mutex]]
- [[#Incorrect defer + loop]]
- [[#Мьютекс + структуры данных]]
- [[#incorret struct design]]
- [[#lockable struct]]
- [[#Recursive lock]]
- [[#deadlock]]
- [[#Unlock другой горутины]]
- [[#livelock]]
- [[#Starvation]]

**Data race** - несинхронизированное обращение к одному и тому же участку памяти разных потоков(горутин), как минимум один из которых осуществляет запись.

**Нельзя допускать Data race'ы в программах.**

**race condition** - ошибка проектирования приложения, при которой работа программы зависит от того в каком порядке выполнятся части кода.

Используя больше одного примитива синхронизации, нужно хорошенько думать над кодом.

**Deadlock** - Ситуация в многозадачной среде, при которой несколько потоков(горутин) находятся в состоянии ожидании ресурсов, занятых друг-другом, и ни один из них не может продолжить выполнение.

Deadlock = **циклическое ожидание**.

Каждый участник ждёт ресурс,  
который удерживает другой участник.

Ресурсы - мьютексы

Процессы - горутины

![[Pasted image 20260215044040.png]]

![[Pasted image 20260215044119.png | 500]]

**Livelock** - ситуация в которой система не застревает, а занимается бесполезной работой. Её состояние постоянно меняется, но тем не менее, она не производит полезной работы

**Starvation** - ситуация где поток не может получить все необходимые ресурсы, для выполнения работы и начинает голодать.
### race condition + deadlock

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	text := ""

	wg := sync.WaitGroup{}
	wg.Add(1)

	go func() {
		defer wg.Done()
		//mtx.Lock()
		text = "hello world"
		//mtx.Unlock()
	}()
	wg.Wait()

	wg.Add(1)
	go func() {
		defer wg.Done()
		//mtx.Lock()
		fmt.Println(text)
		//mtx.Unlock()
	}()

	wg.Wait()
}

```

`data race + race condition` - так как горутины не синхронизированы и у нас может вывести пустую строчку, а может `hello world`.  Data race фиксим мьютексами, race condition - wg'шками.

![[Pasted image 20260212154434.png | 400]]

##### Локальный мьютекс:

```go
package main

import (
	"fmt"
	"sync"
)

var value int
var mtx sync.Mutex

func inc() {
	//нельзя делать мьютекс локальным! data race
	var mtx sync.Mutex

	mtx.Lock()
	value++
	mtx.Unlock()
}

func main() {

	wg := sync.WaitGroup{}
	wg.Add(1000)

	for i := 0; i < 1000; i++ {
		go func() {
			defer wg.Done()
			inc()
		}()
	}

	wg.Wait()
	fmt.Print(value)
}

```

Почему этот код хуже:

```go
var cache map[string]string
var mtx sync.Mutex

func doSomething() {
	mtx.Lock()
	item := cache["key"]
	fmt.Println(item)
	mtx.Unlock()
}
```

чем этот?:

```go
var cache map[string]string
var mtx sync.Mutex

func doSomething() {
	mtx.Lock()
	item := cache["key"]
	mtx.Unlock()
	fmt.Println(item)
```

Чем больше операций после блокировки мы положим тем больше горутин будут ожидать соответственно хуже будет пропускная способность.

Критическая секция должна быть минимальной, но достаточной для сохранения инварианта.

##### defer + mutex

```go
var mtx sync.Mutex

func operation()



func doSomething() {
	mtx.Lock()
	operation()
	mtx.Unlock()

	//long operation

	mtx.Lock()
	operation()
	mtx.Unlock()
}
```

можем забыть написать unlock мьютекса и т.д. короче дляб езопасности лучше использовать вариант с обёрткой.

```go
package main

import (
	"sync"
)

var mtx sync.Mutex

func operation()

func withLock(mutex *sync.Mutex, action func()) {
	if action == nil {
		return
	}

	mutex.Lock()
	defer mutex.Unlock() //может быть паника

	action()
}

func doSomething() {
	withLock(&mtx, func() {
		operation()
	})

	//long operation

	withLock(&mtx, func() {
		operation()
	})
}

```

### defer с мьютексом

```go
func functionWithPanic() {
	panic("error")
}

func handle1() {
	defer func() {
		if err := recover(); err != nil {
			log.Println("recovered")
		}
	}()

	mtx.Lock()
	defer mtx.Unlock()

	functionWithPanic()
}

func handle2() {
	defer func() {
		if err := recover(); err != nil {
			log.Println("recovered")
		}
	}()

	mtx.Lock()

	functionWithPanic()

	mtx.Unlock()
}

func main() {
	handle1()
	handle2()
}

```

Казалось бы две одинаковые функции, но в случае со второй функцией наш мьютекс может не разлочиться. Почему так?

Что происходит в первой функции:

- panic
    
- выполняется `mtx.Unlock()` (из defer)
    
- выполняется `recover`
    
- функция завершается нормально

Что во второй?

1️⃣ `mtx.Lock()` — мьютекс захвачен  
2️⃣ вызывается `functionWithPanic()`  
3️⃣ происходит `panic`  
4️⃣ стек раскручивается  
5️⃣ выполняется `recover()`

Дальше паника и мьютекс остаётся заблокированным навсегда.

В `handle2()`:

> panic прерывает выполнение до `mtx.Unlock()`,  
> а `recover()` не откатывает уже пропущенные строки.

А если второй раз вызовем эту же фунцию - мьютекс уже заблокирован, настанет дедлок. 
#### Incorrect defer +  mutex

```go
package main

import (
	"fmt"
	"sync"
)

var mtx sync.Mutex
var cache map[string]string

func doSomething() {
	var value string

	{
		mtx.Lock() //блокировка мьютекса
		defer mtx.Unlock()
		value = cache["key"]
	}

	fmt.Println(value)

	{
		mtx.Lock()//дедлок из-за того что не разлочили прошлый мьютекс
		defer mtx.Unlock()
		value = cache["key"]
	}
}

```

Тут уже пошёл откровенный мудрёж от балунова. Но суть в том что `defer` **работает только ПЕРЕД ВЫХОДОМ ИЗ ФУНКЦИИ, `defer` НЕ СРАБОТАЕТ ПРИ ВЫХОДЕ ИЗ ОБЛАСТИ ВИДИМОСТИ.**

###### Incorrect defer + loop

```go
func doSomething() {
	for number := 0; number < 10; number++ {
		mtx.Lock()
		defer mtx.Unlock()

		values = append(values, number)//следующая итерация, получаем deadlock
	}
}
```

В этом коде, также суть в том что сработает `defer` только при выходе из функции, 1ая итерация - всё окей, на второй получаем дедлок из-за того что мьютекс разблокируется только после цила.

### Мьютекс + структуры данных

Инкапсулировать мьютекс лучше внутри структуры данных, скрывая сложность.

##### stack sync

```go
type Stack struct {
	mutex sync.Mutex
	data  []string
}

func newStack() *Stack {
	return &Stack{}
}

func (b *Stack) Push(value string) {
	b.mutex.Lock()
	defer b.mutex.Unlock()

	b.data = append(b.data, value)
}

func (b *Stack) Pop() {

	b.mutex.Lock()
	defer b.mutex.Unlock()

	if len(b.data) == 0 {
		panic("pop: stack is empty")
	} //len - под блокировкой

	b.data = b.data[:len(b.data)-1]
}

func (b *Stack) Top() string {
	b.mutex.Lock()
	defer b.mutex.Unlock()

	if len(b.data) == 0 {
		panic("top: stack is empty")
	} //len - под блокировкой

	return b.data[len(b.data)-1]
}

var stack Stack

func producer() {
	for i := 0; i < 1000; i++ {
		stack.Push("message")
	}
}

func consumer() {
	for i := 0; i < 10; i++ {
		_ = stack.Top()
		stack.Pop()
	}
}

func main() {
	producer()

	wg := sync.WaitGroup{}
	wg.Add(100)

	for i := 0; i < 100; i++ {
		go func() {
			defer wg.Done()
			consumer()
		}()
	}

	wg.Wait()
}

```

Возникает следующая проблема в коде:

![[Pasted image 20260215025958.png | 300]]

К коду с горутинами надо относиться так будто между любой строчкой может встроиться 1000 горутин. Решается проблема следующим образом мы просто снимаем функцию `top()` и переносим её в `pop()`

```go
func (b *Stack) Pop() string {

	b.mutex.Lock()
	defer b.mutex.Unlock()

	if len(b.data) == 0 {
		panic("pop: stack is empty")
	}
	value := b.data[len(b.data)-1]
	b.data = b.data[:len(b.data)-1]
	return value 
}

```

ещё один важный момент здесь же. Это то что если бы len() не был бы под блокировкой произошла бы следующая ситуация:

```go
G1: проверяет len == 1
G2: делает pop → len становится 0
G1: продолжает и делает pop
```

race condition на выходе. len() в подобных случаях тоже должен быть под блокировкой мьютекса, так как происходит чтение `shared memory`, другая горутина может менять длину

#### buffer incorrect

```go
func (b *Buffer) Add(value int) {
	b.mtx.Lock()
	defer b.mtx.Unlock()

	b.data = append(b.data, value)
}

func (b *Buffer) Data() []int {
	b.mtx.Lock()
	defer b.mtx.Unlock()

	return b.data
}

```

Всё под блокировкой вроде но в 8 строчке просто отдаём пользователю контроль над структурой данных. По любому получим `data race`, либо делаем отдаём пользователю deep copy, либо делаем следующее:

```go
	func (b *Buffer) ForEach(action func(int))  {
		if action == nil {
			return
		}

		b.mtx.Lock()
		defer b.mtx.Unlock()

		for _, v := range b.data {
			action(v)
		}
	}

```

Даём пользователю возможность взаимодействовать с данными, под мьютексом.

##### incorret struct design

```go
package main

import (
	"sync"
)

type Data struct {
	sync.Mutex
	values []int
}

func (d *Data) Add(value int) {
	d.Lock()
	defer d.Unlock()

	d.values = append(d.values, value)
}

func main() {
	data := Data{}
	data.Add(100)

	// data.Unlock()
}
```

Так лучше не писать(без промежуточного поля) потому что даём возможность пользователю лочить/анлочить мьютекс что скорее всего приведёт к ошибке

##### lockable struct

```go
type Lockable[T any] struct {
	sync.Mutex
	Data T
}

func main() {
	var l1 Lockable[int32]
	l1.Lock()
	l1.Data = 100
	l1.Unlock()

	var l2 Lockable[string]
	l2.Lock()
	l2.Data = "test"
	l2.Unlock()
}
```

Суть в том, что казалось бы и там и там блокировки торчат наружу так? Разница в том что с джернериком мы как бы сообщаем сами себе или пользователю который будет работать с этим контейнером - бро, вся ответственность на тебе. Хочешь лочь, хочешь не лочь. А варианте предыдущем мы типо делаем всё инкапсулированно, но на самом деле нет. Мы можем не понимать, что у нас небезопасная структура используется.

|Lockable[T]|Data с публичным Mutex|
|---|---|
|Честный low-level primitive|Полу-инкапсулированная структура|
|Ответственность на пользователе|Ответственность размазана|
|Не притворяется безопасной|Может вводить в заблуждение|

##### Recursive lock

```go
type Cache struct {
	mtx sync.Mutex
	data map[string]string
}

func NewCache() *Cache {
	return &Cache{
		data: make(map[string]string),
	}
}

func (c *Cache) Set(key, value string) {
	c.mtx.Lock()
	defer c.mtx.Unlock()

	c.data[key] = value
}

func (c *Cache) Get(key string) (value string) {
	c.mtx.Lock()
	defer c.mtx.Unlock()

	if c.Size() > 0 {
		return c.data[key]
	}

	return ""
} 

func (c *Cache) Size() int {
	c.mtx.Lock()
	c.mtx.Unlock()

	return len(c.data)
}
```

В методе `c.Get()` получаем двойную блокировку мьютекса когда переходим в `c.Size()` . 

Чтобы это пофиксить просто создаём скрытый метод, без каких либо блокировок + как рекомендацию можно прописывать названия методов где мьютекс блокируется прописывать `actionLocked()`

```go
func (c *Cache) Get(key string) (value string) {
	c.mtx.Lock()
	defer c.mtx.Unlock()

	if c.sizeLocked() > 0 {
		return  c.data[key]
	}

	return ""
} 

func (c *Cache) Size() int {
	c.mtx.Lock()
	c.mtx.Unlock()

	return c.sizeLocked()
}


func (c *Cache) sizeLocked() int {
	return len(c.data)
}

```

![[Pasted image 20260215040007.png]]

```go
type Data struct {
	X int //один участок памяти
	Y int //другой участок памяти
}

func main() {
	var data Data
	values := make([]int, 2)

	wg := sync.WaitGroup{}
	wg.Add(2)

	go func() {
		defer wg.Done()

		data.X = 5
		values[0] = 5
	}()

		go func() {
		defer wg.Done()

		data.Y = 10
		values[1] = 10
	}()

	wg.Wait()
}

```

Несмотря на то что d.X и d.Y в одной структуре это разные участки памяти соответственно `data race'a` здесь нету.

### deadlock

```go
package main

import (
	"sync"
)

var resource1 int
var resource2 int

func normalizeResources(lhs, rhs *sync.Mutex) {
	lhs.Lock()
	rhs.Lock()

	//normalization

	rhs.Unlock()
	lhs.Unlock()
}

func main() {
	var mtx1 sync.Mutex
	var mtx2 sync.Mutex

	wg := sync.WaitGroup{}
	wg.Add(1000)

	for i := 0; i < 500; i++ {
		go func() {
			defer wg.Done()
			normalizeResources(&mtx1, &mtx2)
		}()
	}

	for i := 0; i < 500; i++ {
		go func() {
			defer wg.Done()
			normalizeResources(&mtx2, &mtx1)
		}()
	}
	
	wg.Wait()
}

```

Классическая реализация циклического deadlock'а. Почему? G1 из первый группы допустим захватывает mtx1, G2(из второй) берёт mtx2, далее G1 делает `mtx2.Lock()`, но мьютекс уже захвачен G2, далее G2 берёт `mtx1.Lock()` и ВСЁ НАХУЙ ПРОИСХОДИТ ВЗРЫВ МЕЖБАЛЛИСТИЧЕСКОЙ РАКЕТЫ В НАШЕМ КОДЕ.

```
G1 → ждёт mtx2 (у G2)
G2 → ждёт mtx1 (у G1)
никто не может продолжить.

это и есть deadlock.
```

Дополним код ниже этой функцией:

```go
	go func() {
		for {
			time.Sleep(time.Second)
			log.Println("tick")
		}
	}()
	wg.Wait()
}
```

И мы никогда не получим дедлока. ЕСТЬ ХОТЬ ОДНА ГОРУТИНА  ДЕДЛОКА НЕ БУДЕТ, ДАЖЕ ЕСЛИ 1000 других вечно будут ждать.

![[Pasted image 20260215043852.png]]

Ресурсы - мьютексы

Процессы - горутины

![[Pasted image 20260215044040.png]]

### Unlock другой горутины

```go
func unlockFromAnotherGoroutine() {
	mtx := sync.Mutex{}
	mtx.Lock()//захват мейн горутиы мьютексом

	wg := sync.WaitGroup{}
	wg.Add(1)

	go func() {
		defer wg.Done()
		mtx.Unlock()//анлочим мейн горутину
	}()

	wg.Wait()

	mtx.Lock() //если мы не разлочим мейн горутину будет дедлок просто проврека состояния грубо говоря
	mtx.Unlock()
}

func main() {
	// lockAnyTimes()
	// unlockWithoutLock()
	unlockFromAnotherGoroutine()
}

```

Мы можем анлочить мьютекс которая захватила горутина через другую горутину.

### livelock

```go
func goroutine() {
	mtx1.Lock()
	
	runtime.Gosched()
	for !mtx2.TryLock() {
		//active waiting
	}


	mtx2.Unlock()
	mtx1.Unlock()
	log.Println("goroutine finished")
}

func goroutine2() {
	mtx2.Lock()
	
	runtime.Gosched()
	for !mtx1.TryLock() {
		//active waiting
	}

	mtx1.Unlock()
	mtx2.Unlock()
	log.Println("goroutine2 finished")

}

func main() {
	wg := sync.WaitGroup{}
	wg.Add(2)

	go func() {
		defer wg.Done()
		goroutine()
	}()
	
	go func() {
		defer wg.Done()
		goroutine2()
	}()
	
	wg.Wait()
}
```

Дедлока не будет. 

```
runtime.Goshed - context switching на другую горутину
```

суть в том что они как бы бесконечно пытаютяс перехватить мьютекс друг-друга и поэтому работают вечно. А мейнгорутина не может закончится из-за того что ждёт их.

### Starvation

```go
package main

import (
	"log"
	"sync"
	"time"
)
// Starvation may be with processor, memory, file descriptors,
// connections with database and so on...

func main() {
	var wg sync.WaitGroup
	var mtx sync.Mutex
	const runtime = 1 * time.Second


	greedyWorker := func() {
		defer wg.Done()
		
		var count int
		for begin := time.Now(); time.Since(begin) <= runtime; {
			mtx.Lock()
			time.Sleep(3 * time.Nanosecond)
			mtx.Unlock()
			count++
		}

		log.Println("Greedy worker was able to execute %v work loops\n", count)
	}

	politeWorker := func() {
		defer wg.Done()

		var count int
		for begin := time.Now(); time.Since(begin) <= runtime; {
			mtx.Lock()
			time.Sleep(1 * time.Nanosecond)
			mtx.Unlock()

			mtx.Lock()
			time.Sleep(1 * time.Nanosecond)
			mtx.Unlock()

			mtx.Lock()
			time.Sleep(1 * time.Nanosecond)
			mtx.Unlock()
			count++
		}
		log.Println("Polite worker was able to execute %v work loops\n", count)
	}

	wg.Add(2)
	go greedyWorker()
	go politeWorker()

	wg.Wait()
}

```

```
2026/02/15 05:25:10 Greedy worker was able to execute %v work loops
 1049
2026/02/15 05:25:10 Polite worker was able to execute %v work loops
 342
```

Суть в том что у нас две горутины конкурируют за мьютекс. Здесь наглядно показано что наш жадный воркер будет больше выполняться, а вежливый воркер будет чаще выходить - заходить в критическую ситуацию. Поэтмоу жадный воркер сделает больше итераций он реже выходит а легкий воркер наоборот. Суть в том что вежливый воркер будет постоянно конкурировать за мьютекс с жадным. И проигрывать потому что он должен три раза захватить и при этом конкурировать ПОСТОЯННО. А жадный один раз. Мьютекс не  гарантирует честности.

```
Greedy: 3ns
Polite: 1ns + 1ns + 1ns = 3ns
```

Две горутины конкурируют за один mutex.

Но:

- Greedy за одну итерацию делает **1 захват**
    
- Polite за одну итерацию делает **3 захвата**
    

И это ключевой момент.

Поэтому каждый раз, когда polite делает Unlock:

- greedy может перехватить lock раньше
    
- потому что он уже исполняется на CPU
    
- и сразу снова вызывает Lock()

### Инверсия приоритетов

![[Pasted image 20260216042805.png]]

Суть в чём. У нас приоритетный поток(T1), не может выполняться потому что наш мьютекс перехватил менее приоритетный поток. И второй менее приоритетный поток берёт на себя выполнение.

**Инверсия потоков** - не так перераспределение ресурсов которое мы планировали заранее. 

Происходит из-за того, что поток, который имеет высокий приоритет, захватывает примитив синхронизации, который был до этого захвачен менее приоритетным потоком. 

Чтобы этого избежать, мы просто передаём высокий приоритет, потоку с меньшим приоритетом чтобы он как можно быстрее выполнил свою работу, и отдать поток на работу с изначально более высоким приоритетом.

P.S. Это случилось с какой-то ракетой и она не могла юзать свои функции из-за этой хуйни

![[Pasted image 20260216043206.png | 400]]