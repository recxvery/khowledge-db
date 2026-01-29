**goroutine** - функция которая может запускаться конкурентно

```go
package main

import "fmt"

func foo() {
	fmt.Println("Hello from foo")
}

func main() {
	go func() {
		fmt.Println("Hello from goroutine")
	}()	//fork point

	go foo() //fork point

	fmt.Println("Hello from main")
}
```

При этом коде вывод будет:

```
Hello from main
```

Нам нужно решить 3 проблемы:

![[Pasted image 20260120095803.png]]

**fork** - точка объявления горутины
![[Pasted image 20260120101059.png]]

c помощью точки **join** мы должны выполнить синхронизацию горутин

![[Pasted image 20260120101221.png]]

### Самый простой способ реализации горутин:

```go
func foo(wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Println("Hello from foo")
}

func main() {
	wg := &sync.WaitGroup{} //значенние переддаются по указателю так как иначе при переносе в функции у нас появится ссылка на абсолютно другой waitGroup. 

	wg.Add(1)
	go func() {
		defer wg.Done()//добавляем это условие в коде
		fmt.Println("Hello from goroutine")
	}()	
	
	wg.Add(1)//добавляем горутины
	go foo(wg)

	wg.Wait()//join point - грубо говоря блокируем main горутину пока не выпонлятся остальные

	fmt.Println("Hello from main")
}
```

##### Всегда ли нужна  синхронизация горутин?

Короткий ответ: если горутина не нужна здесь и сейчас то она может быть не синхронизирована. Если можно не синхронизировать то не нужно синхронизировать

### Базовый случай race-conditions

```go
func main() {
	money := 0
	wg := &sync.WaitGroup{}

	wg.Add(1000)
	for range 1000 {
		go func() {
			defer wg.Done()
			money++
		}()
	}

	wg.Wait()
	fmt.Println(money)
}
//Никогда не выведется 1000. Будет близкие к этому числу значения
```

Так происходит потому что у нас `money++` - неатомарная операция.

![[Pasted image 20260120103223.png]]

![[Pasted image 20260120103346.png]]

Здесь прям наглядно показывается состояние гонки. Наши горутины выполняются бок-о-бок. Получается так, что мы принимаем два раза `money = 0` (горутины одновременно читают 0 из памяти) в сумме получается что после выполнения операций у нас `money = 1`. 

Атомарность в горутинах — это когда операция с данными выполняется **целиком за один шаг**, без возможности прерывания другими горутинами. Как видим у нас она отсутствует

![[Pasted image 20260120104403.png]]

Вот как должен по идее выполняться наш код схематично

Фикшенный код:  

```go
func main() {
	var money atomic.Int64
	wg := &sync.WaitGroup{}

	wg.Add(1000)
	for range 1000 {
		go func() {
			defer wg.Done()

			money.Add(1)
		}()
	}

	wg.Wait()
	fmt.Println(money.Load()) //чтобы вывести атомик нужен метод Load()
}
```

Порой datarace очень умело скрывается результат может быть правильным. Но если условно мы запустим один и тот же код 10000 раз. То он может вывести иное число. Здесь например может вывести 2, а не 3:

```go
func main() {
	money := 0
	wg := &sync.WaitGroup{}

	wg.Add(3)
	for range 3 {
		go func() {
			defer wg.Done()

			money++
		}()
	}

	wg.Wait()
	fmt.Println(money)
}
```

Чтобы определить есть `datarace` или нет используется команда

```go
go run -race .  
```

Вывод:

```
3
Found 2 data race(s)
exit status 66
```

Можем просто переделать с атомиками и datarace'ов не будет:

```go
func main() {
	var money atomic.Int64
	wg := &sync.WaitGroup{}

	wg.Add(3)
	for range 3 {
		go func() {
			defer wg.Done()

			money.Add(1)
		}()
	}

	wg.Wait()
	fmt.Println(money.Load())
}
```

Пример с мьютексами:

```go
func main() {
	var money int64
	var donationsCount int64

	mutex := &sync.RWMutex{}

	go func() {
		for {
			mutex.RLock()
			m := money
			dc := donationsCount
			mutex.RUnlock()
			if m != dc {
				fmt.Println("money=", m, "donations=", dc)
				break
			}
		}
	}()

	wg := &sync.WaitGroup{}

	wg.Add(1000)
	for range 1000 {
		go func() {
			defer wg.Done()

			mutex.Lock()
			money++
			donationsCount++
			mutex.Unlock()
		}()
	}

	wg.Wait()
	fmt.Println(money)
}
```

Без мьютекса горутины будут вмешиваться друг в друга:

```
mutex.Lock()        // только одна заходит
money.Add(1)        // +1 деньги  
donationsCount.Add(1)  // +1 донаты
mutex.Unlock()      // выход → следующая заходит

```

**Без мьютекса**: может прочитать `money=0, donations=1` (первая горутина добавила donation, вторая money ещё нет).

**С мьютексом**: читает **одновременно** — видит либо `0,0` либо `1,1` либо `2,2` или `3,3`.

**Атомики** - самые низкоуровневая структура
**Мьютексы** - выше атомиков
**Каналы** - самая высокоуровневая структура


