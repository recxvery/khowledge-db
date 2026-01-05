##### **JSON - текстовый формат обмена данными, основанный на JavaScript. При этом формат независим от JavaScript'a и может быть использован в любом другом языке программирования** 


## **encoding/json**

### Marshalling/unmarshalling

```go
	bytes, err := json.Marshal(pesron)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(bytes))
```
Супер базовый подход к упаковке файлов в json.
```json
{"Name":"Vov4iq","Age":19,"Married":false,"Hobbies":["Programming","Music","Psychology"],"Address":{"Country":"Russia","City":"Vladimir"}}
```
Вывод
	1
Распаковка
```go
	var resultPerson Person

	if err = json.Unmarshal(bytes, &resultPerson); err != nil {
		log.Fatal(err)
	}

	fmt.Println(person)
```
```go
{Vov4iq 19 false [Programming Music Psychology] {Russia Vladimir}}
```

#### Теги в json

Теги пишутся в бэктиках
```go
type PersonWithTags struct {
	Name    string   `json:"name"`
	Age     int8     `json:"age"`
	Married bool     `json:"married"`
	Hobbies []string `json:"hobbies,omitempty"`
	Address Address  `json:"-"`
}

```

**`omitempty`** - опускает пустые значения. То есть с этим тегом у нас будет вывод
omitempty не работает для пустого, но инициализированного slice.
```go
{"name":"Vov4iq","age":19,"married":false}
```
Без:
```go
{"name":"Vov4iq","age":19,"married":false,"hobbies":[]}
```

`-` - пропускает значение. Оно не попадёт в JSON файл

С этим тегом вывод:
```
{"name":"Vov4iq","age":19,"married":false}
```

### Парсим данные если не знаем, какой тип данный внутри 

```go
func unknownJson() {
	request := `{"first":1, "second": [1, 3, "wqeqwe"]}`

	result := make(map[string]any) //тут чисто решает any || пустой интерфейс 
	//который позволяет нам выводить абсолютно любой тип данных

	if err := json.Unmarshal([]byte(request), &result); err != nil {
		log.Fatal(err)
	} //преобразуем request в байты. передаём по указателю в нашу мапу

	for k, v := range result {
		fmt.Printf("Key: %s, Value: %v\n", k, v)
	}
}
>Key: first, Value: 1
>Key: second, Value: [1 3 wqeqwe]
```

### Custom'ные marshaling/unmarshaling

Нужны тогда когда есть конфликт между GO и JSON. Например у нас в структуре тип данных
`time.time` а в JSON подаётся строка. Следовательно нам нужно превратить это значение в необходимый нам тип данных. Для этого мы и делаем кастомные marshaling/unmarshaling

Пример с time.time:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"time"
)

type Person struct {
	Name      string
	BirthDate time.Time
}

func (p *Person) UnmarshalJSON(data []byte) error {
	// временная структура под JSON
	var temp struct {
		Name      string `json:"name"`
		BirthDate string `json:"birth_date"`
	}

	// обычный анмаршал
	if err := json.Unmarshal(data, &temp); err != nil {
		return err
	}

	// парсинг строки в дату
	date, err := time.Parse("2006-01-02", temp.BirthDate)
	if err != nil {
		return err
	}

	// запись в основную структуру
	p.Name = temp.Name
	p.BirthDate = date

	return nil
}

func main() {
	jsonData := []byte(`{
		"name": "Marcus",
		"birth_date": "1996-08-21"
	}`)

	var p Person
	if err := json.Unmarshal(jsonData, &p); err != nil {
		log.Fatal(err)
	}

	fmt.Println("Name:", p.Name)
	fmt.Println("BirthDate:", p.BirthDate)
	fmt.Println("Year:", p.BirthDate.Year())
}

```

Вот ещё практическая реализация с `float` типом данных
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"strconv"
)

type Person struct {
	Name    string
	Balance float64
}

func (p *Person) UnmarshalJSON(data []byte) error {
	var temp struct {
		Name    string `json:"name"`
		Balance string `json:"balance"`
	}

	if err := json.Unmarshal(data, &temp); err != nil {
		return err
	}

	bankBalance, err := strconv.ParseFloat(temp.Balance, 64)
	if err != nil {
		return err
	}

	p.Name = temp.Name
	p.Balance = bankBalance

	return nil
}

func main() {
	jsonData := []byte(`{
		"name" : "Marcus",
		"balance" : "82349.3242"
	}`)

	var p Person
	if err := json.Unmarshal(jsonData, &p); err != nil {
		log.Fatal(err)
	}

	fmt.Println(p.Balance + 81000)
}

```


### Как работают кастомные маршалинги/анмаршалинги под капотом?

Внутри метода json.Unmarshal(), мы смотрим есть ли у нас метод `UnmarshalJSON`, если он существует, то выполняется кастомный анмаршалинг. 
```go
type Unmarshaler interface {
	UnmarshalJSON([]byte) error
}
```
Внутри у нас UnmarshalJSON()

```go
	if err := json.Unmarshal(jsonData, &p); err != nil {
		log.Fatal(err)
	}
```
Unmarshal грубо говоря преобразует данные из `jsonData` в структуру через указатель `&p`
Без указателя:
```
2025/12/13 18:00:14 json: Unmarshal(non-pointer main.Person)
```


### Encoding/Decoding в Go

Вот такие дела сделали. Короче decoding/encoding использутся при больших данных в отличие от marshal/unmarshal(они читают полностью, а decoding/encoding потоково(через интерфейсы io.Reader/io.Writer)). Как по мне с этой технологией легче работать с исходным файлом(имхо)
```go
func encoderDecoder() {
	file, err := os.Open("practice.json")
	if err != nil {
		log.Fatal(err)
	}

	var person PersonWithTags

	decoder := json.NewDecoder(file)
	if err := decoder.Decode(&person); err != nil {
		log.Fatal(err)
	}
	fmt.Println(person)

	if err = file.Close(); err != nil {
		log.Fatal(err)
	}

	person.Name = "Petya"

	file, err = os.OpenFile("practice.json", os.O_WRONLY|os.O_TRUNC, 0666)
	if err != nil {
		log.Fatal(err)
	}

	defer func() {
		if err = file.Close(); err != nil {
			log.Fatal(err)
		}
	}()

	if err := json.NewEncoder(file).Encode(person); err != nil {
		log.Fatal(err)
	}
}
```

#### Отличия 

| Что                            | Marshal / Unmarshal | Encoder / Decoder       |
| ------------------------------ | ------------------- | ----------------------- |
| Работает с                     | `[]byte`            | `io.Reader / io.Writer` |
| Читает целиком                 | ✅                   | ❌                       |
| Поток                          | ❌                   | ✅                       |
| Подходит для больших данных    | ❌                   | ✅                       |
| Использует кастомный Unmarshal | ✅                   | ✅                       |
| Про файлы                      | ❌                   | ❌ (через Reader)        |
