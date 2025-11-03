
## strings.Count().

```go
wordCount := strings.Count(string(dat), word)
```

Короче, грубо говоря это строка очень удобно работает и не нужно в ручную делать счётчик для каждого слова. 
вот более просто пример позволяющий помочь нам понять что да как:

```go
strings.Count("hello hello", "lo") // → 2
	```

## strings.Split()

```go
strokes := strings.Split(string(dat), "\n")
```
Ну как в питоне разделяет по разделителю строку, чтобы получился в итоге массив

## strings.Contains()

```go
			if strings.Contains(stroke, word) {
				result = append(result, fmt.Sprintf("%v, %s: %s\n", i+1, word, stroke))
			}
```

Тут метод contains просто возвращает булевое значение, содержится то или иное слово в нашей строчке или нет.

## strings.Join()

```go
resString := strings.Join(result, "\n")
```

Тупа конвертируем массив в строчку.

## strings.ContainsAny()

проверяют содержатся ли в строке те или иные символы. ![[Pasted image 20250629182002.png]]![[]]