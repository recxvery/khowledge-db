[[Массивы в GO]]
[[Структуры]]


в GO map'a это тип данных основанный на hash-таблицах(парах ключ-значение). Или же абстрактная структура данных которая умеет выполнять три операции(вставка, удаление, поиск элемента по ключу)
![[Pasted image 20251224145116.png]]
Операции должны выполняться за константное время(скорость выполнения операций не должна зависеть от количества имеющихся данных).



## Структура мапы под капотом:




Значение мапы может быть любым, а ключ **comparable/uncomparable** 


**Uncomparable data types**:
- struct с incomparable типами полями
- slices
- maps
- functions



![[Pasted image 20250719205122.png]]
## Объявление

```go
var defaultMap map[int64]string
```

```go
mapByMake := make(map[string]string, 3)
```
здесь в конце ставим кол-во элементов которые мы хотим чтобы были в map'e. и это по сути параметр **capacity**

**Литеральный способ**

```go
mapByLiteral := map[string]int64{"Сон Ки Хун": 456, "Ин Хо": 1}
```

С помощью **new()**
```go
mapByNew = *new(map[string]string)
```
new если что это просто мы создаём объект и сразу же возвращаем указатель под него, но здесь мы инста его разыменовали

## Вставка/обновление значений и в целом работа с map'ами

Вставка/обновление:
```go
mapByMake["Winner"] = "222"
fmt.Printf("Type: %T, Value: %v\nLength: %d\n", mapByMake, mapByMake,len(mapByMake))
```

Получаем значения:
```go
fmt.Println(mapByLiteral["Сон Ки Хун"]) //456
```

Получаем значения которого не существует:
```go
fmt.Println(mapByLiteral["Чжун Хо"]) //0. т.к. для int типа данных дефолтным будет 0
```

Проверка на существование значения:
```go
number, ok := mapByLiteral["Мин Су"]
fmt.Printf("Type: %T, IsExist: %t\n", number, ok) //int64, false т.к. ключа такого нету 
```

Удаление пар:
```go
delete(mapByMake, "Winner")
fmt.Printf("Type: %T, Value: %v\nLength: %d\n", mapByMake, mapByMake, len(mapByMake))
//Type: map[string]string, Value: map[]
//Length: 0
```
если захочет удалить/напечатать элемент которого нету то ничего не произойдёт

**Итерация**
```go
mapForIteration := map[string]int{"First" : 1, "Second" : 2, "Third": 3, "Fourth": 4}

for key, value := range mapForIteration {
		fmt.Printf("Key: %s, Value: %d\n", key, value)
}
//Key: Third, Value: 3
//Key: Fourth, Value: 4
//Key: First, Value: 1
//Key: Second, Value: 2
```
Последовательность будет рандомная, всмысле выводится всегда будет в странном порядке.

## Используем map'ы


## Считаем кол-во слов

```go
	text := "мир мир дружба мир"
	words := strings.Split(text, " ")

	frequency := make(map[string]int)

	for _, word := range words {
		frequency[word]++
	}

	fmt.Println(frequency)
```

В данном примере мы:
1. Использовали функцию `Split` из пакета `strings`, с которой, скорее всего, вы еще не знакомы. Она разделяет строку (первый аргумент) по строчному разделителю (второй аргумент), и возвращает слайс строк. В нашем случае мы получили список всех слов из оригинальной строки разделенной по пробелам.
2. Мы прошлись по словам и использовали каждое из них как ключ мапы, увеличивая значение, связанное с этим ключом, каждый раз, когда встречали слово.

## юзаем как set(набор уникальных сущностей)

#### Изначальный порядок не важен:
```go
package main

import (
	"fmt"
)

func main() {
	nums := []int{1, 2, 2, 3, 3, 3, 4, 5, 5, 6, 6, 6}
	unique := make(map[int]bool)

	for _, num := range nums {
		unique[num] = true
	}

	for key := range unique {
		fmt.Print(key, " ")
	}
}
```
	
#### Изначальный порядок важен:

```go
package main

import (
	"fmt"
)

func main() {
	nums := []int{1, 2, 2, 3, 3, 3, 4, 5, 5, 6, 6, 6}

	unique := make(map[int]bool)
	result := make([]int, 0)

	for _, num := range nums {
		if !unique[num] {
			result = append(result, num)
		}
		unique[num] = true
	}

	for _, num := range result {
		fmt.Print(num, " ")
	}
}
```