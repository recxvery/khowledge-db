Как по идее решается fixed window: 

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	nums := []int{-1}
	k := 1
	fmt.Println(findMaxAverage(nums, k)) //вводим input'ы
}

func findMaxAverage(nums []int, k int) float64 {

	MaxSum := -math.MaxFloat64 //берём минимально возможное значение. ВОВА БЛЯТЬ ЭТО НЕ ЕГЭ ГДЕ МИНИМУМ = 0 ЭТО ЛИТКОД БЛЯТь
	n := len(nums)

	sum := 0	
	for i := 0; i < k; i++ {
		sum += nums
	}

	for i := k; i < n; i++ { // n - k, т.к мы находим грубо говоря сколько вообще у нас подслайсов будет, в случае с 6 элементами их будет три
		sum = 0
		for j := i; j < i; j++ { //j := i, j < i + K? ---> j := i, даёт нам исходный индекс с которого начинается подслайс, j < i + k, прикол в том что таким образом у нас
			//у нас всегда будет фиксированная длина, типо сначала будет i + k == 4, потом i + k == 5, но
			//так как i = 1, то выполнится также 4 раза типо i = 1,i = 2, i =3, i = 4
			sum += nums[j]
			fmt.Println(sum)
		}

		MaxSum = math.Max(MaxSum, float64(sum))
	}
	return MaxSum / float64(k)
}


```
Итог timed out - leetcode ахуенна

По факту:
```go
package main

import (
	"fmt"
	"math"
)

func main() {
	nums := []int{5}
	k := 1
	fmt.Println(findMaxAverage(nums, k)) //вводим input'ы
}

func findMaxAverage(nums []int, k int) float64 {
	MaxSum := -math.MaxFloat64
	n := len(nums)
	sum := 0

	for i := 0; i < k; i++ {
		sum += nums[i] //считаем первое окно. для того чтобы не создавать O(n^2)
	}
	MaxSum = math.Max(MaxSum, float64(sum)) // а это нужно для того что если у нас будет один элемент. и цикл снизу не выполнится. мы смогли все равно найти максимум.

	fmt.Println(sum)
	for i := k; i < n; i++ { // берём за первый элемент k(k = 4, n = 6), получается что 3 раза наше окошко двинется
		sum += nums[i] // + элемент входящий в окно
		sum -= nums[i-k] //- элемент выпадающий из окна. допустим 5 - 4, выйдет первый элемент и т.д.
		MaxSum = math.Max(MaxSum, float64(sum))
	}
	return MaxSum / float64(k)
}

```


![[Pasted image 20250814042045.png]]

```go
package main

import "fmt"

func main() {
    // Исходная строка для анализа
    s := "abcdaabcde"
    
    // Вызываем функцию и получаем длину и саму подстроку
    length, substr := lengthOfLongestSubstring(s)
    
    // Выводим результат в формате "длина подстрока"
    fmt.Printf("%d %s\n", length, substr) // Ожидаемый вывод: 5 abcde
}

// Функция для нахождения самой длинной подстроки без повторяющихся символов
func lengthOfLongestSubstring(s string) (int, string) {
    // Создаем map для хранения последних позиций символов
    // Ключ: символ (тип rune для поддержки Unicode)
    // Значение: последний индекс где встретился этот символ
    charIndexMap := make(map[rune]int)
    
    // Переменная для хранения максимальной длины подстроки
    maxLength := 0
    
    // Начальный индекс текущего окна подстроки
    start := 0
    
    // Начальный индекс самой длинной найденной подстроки
    longestStart := 0

    // Итерируемся по строке
    // end - текущий индекс, char - текущий символ
    for end, char := range s {
        // Проверяем, встречался ли символ char ранее
        // lastIndex - индекс предыдущего вхождения, exists - флаг наличия
        if lastIndex, exists := charIndexMap[char]; exists && lastIndex >= start {
            // Если символ уже был в текущем окне, сдвигаем start
            // за предыдущее вхождение этого символа
            start = lastIndex + 1
        }
        
        // Обновляем в map последнюю позицию текущего символа
        charIndexMap[char] = end

        // Вычисляем длину текущей подстроки
        currentLength := end - start + 1
        
        // Если текущая подстрока длиннее максимальной найденной
        if currentLength > maxLength {
            // Обновляем максимальную длину
            maxLength = currentLength
            // Запоминаем начало самой длинной подстроки
            longestStart = start
        }
    }

    // Возвращаем длину и саму подстроку
    // Подстрока вычисляется от longestStart до longestStart+maxLength
    return maxLength, s[longestStart : longestStart+maxLength]
}

```