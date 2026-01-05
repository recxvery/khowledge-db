
```go
go get  github.com/githubnemo/CompileDaemon@latest 
```
Этот пакет автоматически пересобирает приложения(меняет действующий экзешник на новый, если наши файлы обновляются.)

```go
go get github.com/joho/godotenvv
```

Этот пакет уже серьёзнее и он нам нужен для архитектуры. Я так понимаю здесь мы устанавливаем виртуальное окружение в котором будем хранить пароли и т.д.. Чтобы на сервере были одни значения, локальнно другие.
```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	// Define a simple GET endpoint
	r.GET("/", func(c *gin.Context) {
		//когда придёт get-запрос на сервер / вызови вот эту функцию, / - корень сайта и путь
		//func(c *gin.Context) - обработчик запроса(хендлер), каждый раз когда браузер, postman, curl делает запрос, Gin - cоздаёт объект
		//Context(он содержит: запрос, ответ, параметры, JSON, заголовки, статус) context = всё что связано с одним http - запросом
		c.JSON(http.StatusOK, gin.H{ //c.JSOn = отправь клиенту со статусом 200 объект который мы указали в gin.H
			//http.StatusOK = код 200, что всё окей. gin.H = map[string]interface{} 
			"message": "pong",
		})
	})

	// Start server on port 8080 (default)
	// Server will listen on 0.0.0.0:8080 (localhost:8080 on Windows)
	r.Run() //запуск сервера. Запущенный на localhost:8080
}


```

Создали файл `.env` - по сути просто текстовый файл с переменной порта и всё.
```go
	err := godotenv.Load()

	if err != nil {
		log.Fatal("Error loading .env file")
	}
	r := gin.Default()
```

```go
	err := godotenv.Load()
	//Под капотом:
	//and all the env vars declared in .env will be available through os.Getenv("SOME_ENV_VAR")
```
Суть в том, что мы кладём порт 3000 из файла в  

