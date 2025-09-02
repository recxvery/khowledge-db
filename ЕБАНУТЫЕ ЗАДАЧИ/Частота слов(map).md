
Не то чтобы прям трудная задача, но всё же, потратил на неё около 40 минут.

![[Pasted image 20250805225113.png]]

Код:

```go
package main

import (
	"bufio"
	"fmt"
	"os"

	// "testing/quick"
	// "slices"
	"strings"
)

func main() {

	reader := bufio.NewReader(os.Stdin)
	userInput, _ := reader.ReadString('\n')
	wordInput := strings.Fields(strings.TrimSpace(userInput))

	// fmt.Println(wordInput)

	wordsCount := make(map[string]int)

	for _, v := range wordInput {
			wordsCount[v]++
	}


	keys := make([]string, 0)

	for k := range wordsCount {
		keys = append(keys, k)
	}

	// fmt.Println(keys)

	for i := 0; i < len(keys)-1; i++ {
		for j := 0; j < len(keys)-1; j++ {
			if keys[j+1] < keys[j] {
				keys[j], keys[j+1] = keys[j+1], keys[j]
			}
		}
	}

	for _, v := range keys {
		fmt.Printf("%s: %v\n", v, wordsCount[v])
	}
}
//apple banana Apple banana cherry apple
//Aaa Bbb Ccc Aaa Ccc Bbb b a a a a
```