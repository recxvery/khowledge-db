[[race condition, атомики,  мьютексы]]

**`sync.Once()` - это примитив синхронизации из пакета `sync`, предназначенный для гарантированного однократного выполнения функции в конкурентной среде.**

Суть в том что мы можем хоть десять горутин запустить. Энивэй функция сработает один раз.

```go
func main() {
	var once sync.Once

	onceBody := func() {
		fmt.Println("Only once")
	}

	var wg sync.WaitGroup
	wg.Add(10)

	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			once.Do(onceBody)
		}()
	}

	wg.Wait()
}

//>>Only once
```

![[Pasted image 20260216044703.png]]

##### Lazy initialization

```go
type Map struct {
	once sync.Once
	values map[interface{}]interface{}
}

func NewMap() *Map {
	return &Map{}
}

func (m *Map) Add(key, value interface{}) {
	m.init()
	m.values[key] = value
}

func (m *Map) init()  {
	m.once.Do(func() {
		m.values = make(map[interface{}]interface{})
	})
}
```

Суть в том, что в этом коде делаем ленивую инициализацию через `init()`, мы не сразу аллоцируем и занимаем память, а по ходу дела занимаемся этим(без вызова `Add()` нам нет смысла инициаилизировать память.) Add не ипользуется = нет мапы.

##### Singleton case

```go
type Singleton struct {
}

var instance *Singleton
var once sync.Once

func GetInsistance() *Singleton {
	once.Do(func() {

		instance = &Singleton{}
	})
	//if instance == nil {
			//interruption если сдеодна горутина застрянет здесь, другая создаст первую струкктуру, первая вернётся и создаст ещё одну.
	//instance = &Singleton{}
	//}
	
	return instance
}
```

Суть в том Singleton - паттерн где нам нужно создать одну единственную структуру даннхых соответствующего типа. Поэтому при 1000 горутинах `sync.Once` классно заходит и позволяет реализовать паттерн очень легко.