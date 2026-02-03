[[Работа с файлами]]
[[Обработка ошибок в Go]]
## Что будет если исключить обработку ошибок?

```go
func main() {
	file, _ := os.Open("somefile.txt")
	// if err != nil {
	// 	fmt.Println("Ошибка в открытии файла")
	// } //представим что у нас нет этого кода
	fmt.Println(file) // - value: nil

	file.Read("somefile.txt") //panic'a так как пытаемся вызвать метод на nil-pointer
}
```

###  Примитивная обработка

```go
func Pay(u *User, usd int) string {
	if (u.Balance - usd) < 0 {
		return "Недостаточно средств"
	}

	u.Balance -= usd
	return ""
}
```


### **Правильная обработка** 

```go
func Pay(u *User, usd int) error {
	if (u.Balance - usd) < 0 {
		return errors.New("Недостаточно средств")
	}

	u.Balance -= usd
	return nil
}

	if err != nil {
		fmt.Println("Оплата не прошла. Ошибка:", err.Error())
	} else {
		fmt.Println("Оплата прошла")
	}
```

Наглядное пояснение того как выглядит ошибка под капотом. Здесь мы видим, что можно инициализировать ошибку со строкой внутри. А позже использовать её метод. Всё просто на самом деле

```go
package errors

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}

```
	
### **Супер очевидный и простой пример**

```go
func (c *Car) Gas() (int, error) {
	c.Armor -= 10

	if c.Armor <= 0 {
		return 0, errors.New("Машина не выдержит разгона")
	}

	return rand.Intn(150), nil
}

func main() {
	car := Car{Armor: 25}

	pp.Println("Car before", car)

	for {
		accelerate, err := car.Gas()
	
		if err != nil {
			fmt.Println(err.Error())
			break
		} else {
			fmt.Println("Car acceleration:", accelerate)
			pp.Println("Car before", car)
		}
	}
}

```

**Минус этого подхода следующий:** **недостаток контекста в обработке ошибок**, то есть простое `Errors.New("не найдено")` - яв-ся плохим примером так как когда мы смотрим в логи нам абсолютно ни о чём не говорит эта ошибка, что-куда-где пытались найти и будет труднее её отладить.

### Добавление контекста с fmt.Errorf()

Простые ошибки обычно несут мало информации, где конкретно в сложной системе они произошли. Чтобы добавить контекст к ошибке не теряя при этом исходную используется 
`fmt.Errorf()`, с глаголом форматирования: `%w` .

```go
func loadConfig() ([]byte, error) {
	data, err := os.ReadFile("app.config")
	if err != nil {
		//здесь оборачиваем исходную ошибку ERR
		return nil, fmt.Errorf("Не удалось запустить конфигурацию: %w", err)
	}
	return data, nil
}

func setup() error {
	cfgData, err := loadConfig()
	if err != nil {
		//еще один уровень обёртки. здесь мы понимаем что ошибка произошла конкретно во время setup
		return fmt.Errorf("Ошибка настройки приложения: %w", err)
	}
	//,,
	return nil
}
```

Из интересного. Чтобы `w` форматировал нашу ошибку правильно надо сделать её ошибкой. В **Go** считается ошибкой то что имеет метод `Error()` так что чтобы сделать ошибку и вывести через неё надо инициализировать данный метод. 

Пример без инициализации:
![[Pasted image 20251128174037.png]]


С: 
![[Pasted image 20251128174126.png]]

## **Error is/Errror as**
**Error is** -            


### Ловим панику

Нужно это чтобы не полетел прод
```go
defer func() {
		p := recover()
		if p != nil {
			fmt.Println("Паника", p)
		}
	}()
	a, b := 0, 8
	b /= a
	fmt.Println(b)
```
**Важное уточнение**. Ловить панику это по сути анти-паттерн в ГО. Она не предназначена для простых очевидных ошибок по типу(ошибка валидации ввода, файл не найден, запись в БД не удалась). Panic предназначен для исключительных ошибок, которые могут привести к невосстановимым состояниям и используются они очень редко, как правило в горутинах.

