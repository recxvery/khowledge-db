![[Pasted image 20250807040418.png]]

```go
func reversed(n int) int {
	reversedInt := 0
	for n > 0 {
		remainder := n % 10 //1
		reversedInt *= 10//0
		reversedInt += remainder//1
		n /= 10// 
	}
	return reversedInt
}

func isPalindrome(x int) bool {
    if x < 0 {
		return false
	} else if x == reversed(x) {
			return true
	} else {
		return false
	}
}
```

Зачем нужно: 		reversedInt *= 10//0

![[Pasted image 20250807040442.png]]


## Палиндром в случае со строкой (без рекурсии лол)

![[Pasted image 20250814070740.png]]

```go
package main

import (
	"bufio"
	"fmt"
	"os"
    "strings"
)

func isPalindrome(s string) bool {
	runes := []rune(s)
	length := len(runes)
	
	if length <= 1 {
		return true
	}


	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		if runes[i] != runes[j] {
			return false
		} 
	}
	return true
}

func main() {
	// не изменяйте функцию main!!

	input, _ := bufio.NewReader(os.Stdin).ReadString('\n')
	input = strings.TrimSpace(input)

	if isPalindrome(input) {
		fmt.Println("Палиндром")
	} else {
		fmt.Println("Не палиндром")
	}
}
```

## TRUE ANSWER

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// Убираем пробелы и приводим строку к одному регистру для корректной проверки
	text := "город дорог"
	text = strings.ReplaceAll(text, " ", "") // Убираем пробелы
	text = strings.ToLower(text)             // Приводим к нижнему регистру

	// Проверяем, является ли строка палиндромом
	isPalindrome := checkPalindrome(text)
	fmt.Println(isPalindrome) // Вывод: true
}

func checkPalindrome(text string) bool {
	//based: turn string into rune
	word := []rune(text)

	if len(word) == 0 || len(word) == 1 {
		return true
	}

	//check on is last symbol equal for da first one
	if word[0] == word[len(word) - 1] {
		return checkPalindrome(string(word[1 : len(word) - 1]))
	}

	return false
}
```