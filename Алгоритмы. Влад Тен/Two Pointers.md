### Суть паттерна в том что мы ставим два не прямых указателя(не в плане указателя ГО), а вплане создаём переменные которые будут указывать на допустим первый и последний элемент массива. Применение в разных задачах:
Схема для двух указателей:

![[{D95EF7B3-A0D6-423F-B252-D2E3117D7D97}.png]]

### reverseString()

```go
func reverseString(s []byte) []byte {
	j := len(s) - 1
	for i := 0; i < len(s) / 2; i++ {
		
		s[i],s[j] = s[j], s[i] 
		j--
	}

	return s
}
```
Здесь **j** и **i** как раз те самые два указателя

### isPalindrome()
 
```go
package main

import (
	"fmt"
)

func main() {
	s := "A man, a plan, a canal: Panama"
	fmt.Println(isAlphaNumerical('A'))
	fmt.Println(isPalindrome(s))
	// b := "race a car"
	// d := "0P"
	// v := "0z;z   ; 0"
	// fmt.Println(isPalindrome(b))
	// fmt.Println(isPalindrome(d))
	// fmt.Println(isPalindrome(v))
}

func isPalindrome(s string) bool {
	left, right := 0, len(s)-1
	for left < right {
		for left < right && !isAlphaNumerical(s[left]) {
			fmt.Println("Слева не буквенно-численный символ")
			left++
		}

		for left < right && !isAlphaNumerical(s[right]) { //зачем здесь for? чтобы пропустить все символы до первой буквы или цифры
			fmt.Println("Слева не буквенно-численный символ")
			right--
		}

		if ToLower(s[left]) != ToLower(s[right]) {
			fmt.Println(string(s[left]), string(s[right]))

			return false
		}

		left++
		right--
	}
	return true
}

func isAlphaNumerical(b byte) bool {
	return 	(b >= 'a' && b <= 'z') ||
		(b >= 'A' && b <= 'Z') ||
		(b >= '0' && b <= '9')
}

func ToLower(b byte) byte {
	if b >= 'A' && b <= 'Z' {
		return b + 32
	}
	return b
}
 
```
Самая быстрая функция для нахождения палиндрома и да со своими функциями получается быстрее чем через встраивание либы strings

### Two Sum II

```go
func twoSum(numbers []int, target int) []int {
	right, left := len(numbers) - 1, 0 //создаем два указателя начало/конец массива
	for left < right { //цикл будет выполняться пока указатели не пересекутся
		sum := numbers[right] + numbers[left]
		switch  {
		case sum > target: 
			right-- //так как у нас сортированный массив то если сумма больше то мы двигаем правый указатель влево
		case sum < target: 
			left++ //если сумма наоборот меньше то двигаемся к следующему числу по возрастанию. левый указатель вправо
		default:
			return []int{left+1, right+1}//leetcode requirements
		}
	}
	return nil
}
```

Здесь я понял что можно не начинать цикл проходки по массиву а просто задать условие которое будет наиболее эффективно и давать нам как можно больше пространства. условие подобрано идеально. И сам метод вычисления (сумма больше/меньше) мне нравится + не забываем про switch. насколько помню он быстрее.

Ещё сюда же к Two Sum II. Мы не можем решить это через мапу потому что таким образом у нас будет образовывать сложность **О(n)**, 1 000 000 элементов в массиве = 1 000 000 в мапе. Два указателя будут намного эффективнее. Наглядность ниже:
```
Мапа на 1000 элементов:
- ~1000 записей (ключ + значение + хэш + указатели)
- ~40-80 байт на запись в Go
- Итого: 40-80 КБ + оверхед бакетов
```

```
Два указателя:
- left: 8 байт
- right: 8 байт  
- Итого: 16 байт в любом случае
```


### 3Sum
```go
func threeSum(nums []int) [][]int {
	sort.Ints(nums) //[-4, -1, -1, 0, 1, 2] O(n log n) //сортируем массив иначе у нас будут по кд повторяющиеся тройки
	result := [][]int{}
	for i := range len(nums) - 2 { //len(nums) - 2 потому что таким образом мы высобождаем место под left/right(если не написать то left == right при последней итерации i)
		//пропускаем дубликаты. каким образом? если у нас nums[i] == -1; nums[i + 1] == -1, цикл пропустит итерацию
		if i > 0 && nums[i] == nums[i-1] {
			continue
		}

		//classic 2 sum algo
		left, right := i+1, len(nums)-1

		for left < right {
			sum := nums[i] + nums[left] + nums[right] //складываем сумму. 1. -4 + (-1) + (2) == -3 < 0 / 2. -4 + (-1) + (2) == -3 < 0. / 3. -4 + 0 + 2 == -2
			// 4. -4 + 1 + 2 == -1
			//нашли сумму
			if sum == 0 {
				result = append(result, []int{nums[i], nums[left], nums[right]})

				//скипаем дубликаты(по сути тот же самый алгоритм что и выше только тут ещё проверка на пересечения указателей)
				for left < right && nums[left] == nums[left+1] { //						                                                          *(left = 4)
					left++ //для чего нужен этот цикл и следующий? Для того чтобы унас не возникло одинаковых троек. Например массив [-2, -1, -1, -1, 1, 1, 1] на итерации
					//i = 0, left = 4, смотрим и видим что у нас следующий элемент равен, значит пропускаем чтобы не было повторяющихся троек.(чтобы в результат попал только
					//один слайс)

				}

				for left < right && nums[right] == nums[right-1] {
					right--
				}

				left++ //если не указываем здесь ничего то указатели просто навечно застревают в цикле left < right
				right--
			} else if sum < 0 {
				left++ //увеличиваем сумму если она меньше 1. 1 + 1  == 2. / 2. 2 + 1 == 3. / 3. 3 + 1 == 4 // 4. 4 + 1 == 5 - выход из цикла
			} else {
				right-- //уменьшаем если большая
			}
		}
	}

	return result
}

```

### SortedSquares
```go
func sortedSquares(s []int) []int {
	res := make([]int, len(s))

	left, right, i := 0, len(s)-1, len(s) - 1
	for i > -1 {
		if s[left]*s[left] > s[right]*s[right] {
			// fmt.Println(s[right] * s[right], s[left] * s[left])
			res[i] = s[left] * s[left]
			left++
		} else  {
			// fmt.Println(s[right] * s[right], s[left] * s[left])
			res[i] = s[right] * s[right]
			right--
		} 

		i--
	}

	return res
}
```
Тут сугубо проблемы с додумыванием условия, но решил сам, единственное что была подсказка что заполнять массив здесь надо с конца.


### 3Sum Closest

```go
func threeSumClosest(n []int, target int) int {
	sort.Ints(n) //+РЕП мне что додумался отсортировать список
	minDiff := n[0] + n[1] + n[2] //здесь очень важная вещь. когда мы понимаем что нам нужно найти ближайшее число к таргетуя самый важный инсайт - сравнивать разницу 
	//модулей(minDiff - target) и модуля(sum - target) это очень важно. так как конкретно в этой задаче я ломал голову как не проебаться с модулем. потому что просто найти на
	//именьший сумму в мапе будет недостаточно

	for i := range len(n) - 2 {
		left, right := i + 1, len(n) - 1
		for left < right {
			sum := n[i] + n[left] + n[right]

			if sum == target {
				return sum
			}

			//если текущая сумма ближе к таргет обновляем её
			if abs(sum - target) < abs(minDiff - target) {
				minDiff = sum
			}

			if sum > target {
				right--
			} else {
				left++
			}
		}
	}
	return minDiff
}

func abs(x int) int {
	if x < 0 {
		return -x
	} else {
		return x
	}
}
```


### Container with the most water


![[Pasted image 20251113085006.png]]

Здесь не получится просто найти два максимума. Т.к. нам нужно узнать между какими столбами поместится больше всего воды то есть для: 
```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49
```
Самый большой резервуар будет между 8 и 7 да.

Но при таком

```
height = [9, 0, 8, 4, 0, 0, 0, 0, 0, 0, 0, 0, 4]
Output: h[4:-1]
```

```
Подсказка от влада тена:
result = max(result, ....something)
```

Итоговый код:
```go
func containerFullOFWater(s []int) int {
		res := 0
		left, right := 0, len(s) - 1
		for left < right {
			height := 0 //объявляем высоту
			width := right - left //ширина из конца вычитаем начало

			if s[left] < s[right] { //если у нас слева меньшая высотая перемещаем вправо чтобы находить максимально возможную длину
				height = s[left] 
				left++
			} else { //тоже самое с правой высотой.
				height = s[right] //тут есть важный момент перед тем как передвинуть указатель сначала фиксируем его в высоте. чтобы не перекинуть случайно следующее значение
				right--
			}

			area := height * width //объявляем площадь
			 
			if area > res {
				res = area
			}
		}

		return res
}
```
Тут на самом деле ничего сложного нет. Я сам это решил за 5 минут даже не понимая как. хоть и с подсказкой от Влада. Здесь принцип из динамического программирования что мы сравниваем.


### Второй тип с 2 pointers. Fast and slow pointer.

```go
func removeDuplicates(n []int) int {
	c := 0

	for i := range n {
		if n[c] != n[i] {
			c += 1
			n[c] = n[i]
		}
	}
	fmt.Println(n)
	return c + 1
}
```


## Move Zeros

```go
func moveZeros(n []int) {
	nullP := 0
	for i := range n {
		if n[i] != 0 { //тут  просто нужнозациклиться на том что мы сдвигаем не нули вправо. а ненулевые элементы влево. тогда задача ста
			//новится очень простой и понятной
			n[nullP], n[i] = n[i], n[nullP]
			nullP++
		}
	}
}
```


![[Pasted image 20251116150021.png]]
Интересный факт что здесь по сути behy является подпоследовательностью массива выше.

**Подпоследовательность — это последовательность символов, полученная из исходной строки** **удалением некоторых символов без изменения порядка оставшихся**.
**Ключевое:** порядок символов должен сохраняться, но они **не обязательно должны идти подряд**.


Можно ставить два указателя в разных массивах и сравнивать. допустим один указатель на массив снизу другой на массив сверху. Тогда ![[Pasted image 20251116150701.png]]

```
Грубо говоря логика следующая:
b := 0
b2 := 0
for b2 in s {
	if b == b2 {
		b++	
	}
	
	if b == len(bArray) {
	return True
	}
}
```

### IsSubsequence

```go
func isSubsequence(sub string, s string) bool {
	subIndex, i := 0, 0
	for subIndex < len(sub) && i < len(s) {
		if s[i] == sub[subIndex] {
			subIndex++
		}
		i++
	}
	if subIndex == len(sub) {
		return true
	} else {
		return false
	}
}

```
Как раз таки суть того что мы описывали выше в коде


### Merge Arrays
```go
func merge(nums1 []int, m int, nums2 []int, n int)  {
    result := []int{}
	p1, p2 := 0, 0

	for p1 < m && p2 < n {
		if nums1[p1] < nums2[p2] {
			result = append(result, nums1[p1])
			p1++
		} else {
			result = append(result, nums2[p2])
			p2++
		}
	}

	for p1 < m {
		result = append(result, nums1[p1])
	}
	for p2 < n {
		result = append(result, nums1[p2])
	}

	copy(nums1, result)
} // тут думаю и объяснять ничего не надо довольно простенькая задача. Единственное что если тебе нужно соединить два массива которые отсорированы начинай добавлять с меньше элемента.
```



```go
func backspaceCompare(s string, t string) bool {
	n1, n2 := len(s)-1, len(t)-1
	skip_s, skip_t := 0, 0
	for n1 >= 0 || n2 >= 0 {
		//TODO: implement skip(if # or skip_counter > 0)
		for n1 >= 0 { //t = "a##b". я понял суть условия короче тут мы просто ебашим до талого пока skip/n1 != 0. то есть когда не будет решётки и скипа уже.
			if s[n1] == '#' { //2. # вот решётка и мы не выходим из тела цикла а попадаем в него снова со скипом. 2. снова решётка скип снова +, пропускаем ее через
				//else if и пропускаем следующий элемент. тут срабатывают два условия. поэтому в итоге 3. n1 := -1, и мы выходим на этом из цикла
				skip_s += 1 //
				n1--
			} else if skip_s > 0 {
				skip_s--
				n1--
			} else {
				break //1. выходим из цикла если у нас не решётка и нету скип b - break. 3. n = -1
			}
		}

		for n2 >= 0 {
			if t[n2] == '#' {
				n2--
				skip_t += 1
			} else if skip_t > 0 {
				n2--
				skip_t--
			} else {
				break
			}
		}

		if n2 >= 0 && n1 >= 0 && s[n1] != t[n2] { //если н2 и н1 больше нуля то мы сравниваем их между собой если не равны то сразу выход
			return false
		}
		if (n1 >= 0) != (n2 >= 0) {
			return false //тут если один равен нулю другой нет то не равны между собой
		}

		n1--
		n2--
	}
	return n1 == n2
}

func count(s string) int {
	c := 0
	for _, v := range s {

	}
}
```
Конченная задача хотя кажется очень простой. Счётчик пропусков + 4 переменных + постоянные сравнения двух строк делают эту задачу довольно сложной  и в которой можно кучу тупых ошибок на внимательность сделать.