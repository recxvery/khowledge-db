CRUD'а для фильмов. Сюда буду писать что походу дела нового я узнал

## Теги в структуре

```go
	type movie struct {
	ID   string `json:"id"`
	Isbn string `json:"isbn"`
}
```
Нужны они нам для того чтобы в будущем отсылать данные в формате JSON у нас они отсылались строго по ключу "id", а не по ID. Проще говоря теги позволяют нам форматировать данные при их отправке в файл. Насколько я понял. Например в json  у нас не будет ключей с заглавынми буквами. А для того данные по структуре, поле должно быть с заглавной буквой(Публичным)

## **Схема проекта**
![[Pasted image 20250831043153.png]]


```go
	r := mux.NewRouter()

	r.HandleFunc("/movies", getMovies).Methods("GET")
	r.HandleFunc("/movies/{id}", getMovie).Methods("GET")
	r.HandleFunc("/movies/", createMovie).Methods("POST")
	r.HandleFunc("/movies/{id}", updateMovie).Methods("PUT")
	r.HandleFunc("/movies/{id}", deleteMovie).Methods("DELETE")
```

==GET== - получаем данные с сервера.
==POST== - Создаём файл/данные на  сервере.
==PUT== - обновляем данные на сервере. в случае если этих данных нет они добавляются
==DELETE== - удаляем данные на сервере

```go
r.Handlefunc()
```
нужна для того чтобы url мы связывали с функцией когда обрабатываем запрос 

Пример:
```go
r.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello, world!"))
}) // hello -->  "Hello, world!"
```

go

```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Привет! Это простой сервер на Go.")
}

func main() {
    http.HandleFunc("/hello", helloHandler) // Связываем адрес "/hello" с функцией helloHandler
    http.ListenAndServe(":8080", nil)       // Запускаем сервер на порту 8080
}

```
---

## Что здесь происходит?

- `http.HandleFunc("/hello", helloHandler)` — говорит серверу: "Если кто-то придёт на адрес `/hello`, вызови функцию `helloHandler`".
    
- `helloHandler` — функция, которая отвечает клиенту: просто отправляет текст "Привет! Это простой сервер на Go."
    
- `http.ListenAndServe(":8080", nil)` — запускает сервер на порту 8080, где `nil` значит использовать стандартный роутер.
    

Теперь, если открыть в браузере `http://localhost:8080/hello`, вы увидите строку приветствия.


```go
	fmt.Printf("Starting server at port 8000\n")
	log.Fatal(http.ListenAndServe(":8000", r))
```

- (http.ListenAndServe(":8000", r)) - создаём HTTP-сервер с портом - 8000 и передаём обработчик(тот куда будут передаваться запросы), наш роутер  в данном случае.
- log.Fatal() - вывод ошибки в случае, если с функцией че то не так.

```go
type Movie struct {
	ID       string    `json:"id"`
	Isbn     string    `json:"isbn"`
	Title    string    `json:"title"`
	Director *Director `json:"director"`
}

type Director struct {
	Firstname string `json:"firstname"`
	Lastname  string `json:"lastname"`
}

	movies = append(movies, Movie{ID: "1", Isbn: "438227", Title: "Movie 1", Director: &Director{Firstname: "John", Lastname: "Doe"}})
	movies = append(movie, Movie{ID: "1", Isbn: "45455", Title: "Movie 2", Director: &Director{Firstname: "Steve", Lastname: "Smith"}})

```
Почему здесь используется именно указатель?

1. В случае если поле с режиссёром будет пустое, то выведется просто nil. Это легче и для json'a и не спровоцирует **panic: runtime error: invalid memory address or nil pointer dereference**. Ещё это лучше для сериализации(форматировании в json) т.к. в случае с указателем поле просто будет опущено, по объекту - будут пустые поля что непонятно как работает с JSON'oм
2. Мы экономим память. В данном случае это не сильно важно, но в случае если бы структура имела много полей, мы бы раз за разом копировали её полностью, в случае с указателем мы просто копируем адрес на оригинал что имеет фиксированный размер(8 байт) Наглядные примеры ниже:

```go
type Point struct {
    X int // 8 байт (int64)
    Y int // 8 байт (int64)
}
// Общий вес: 16 байт
// Указатель на неё: 8 байт
// Копировать дешево. Указатель не даёт выгоды.



//ANOTHER CASE

type BigData struct {
    Data [1000]int64 // Массив из 1000 чисел int64
}

// Вес массива: 1000 элементов * 8 байт = 8000 байт (8 КБ).
// Указатель на эту структуру: 8 байт.

// Функция, принимающая значение:
func processByValue(b BigData) { // Создается копия 8 КБ!
    // ...
}

// Функция, принимающая указатель:
func processByPointer(b *BigData) { // Передается только 8 байт!
    // ...
}
```

---
Стоящий материал на тему JSON - https://habr.com/ru/articles/797019/

