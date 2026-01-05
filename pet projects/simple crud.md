Самый просто запуск сервер. `ListenAndServe` -  говорим GO принимай запросы по порту 8080
```go
func main() {
	log.Println("Server starting on: 8080")

	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

**Роутер** - грубо говоря адресат. По нему на странице мы можем переходить по адресу `/users/123` и т.д. без роутинга у нас будет доступна одна единственная страница.
`nil` — **дефолтный роутер**. http.DefaultServeMux - встроенный роутер автоматически распределяющий HTTP запросы

### Ручки в GO

Ручка - функция которая вызывается через роутинг. То есть когда мы переходим по какому-то адресу. Срабатывает та или иная функция

Пример простой ручки
```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello from Go server")
}
```

```go
main.go:

http.HandleFunc("/hello", helloHandler)

```

`r *http.Request` - запрос. То есть метод, url и прочее.

`w http.ResponseWriter` - ОТВЕТ. через него пишем тело ответа, статус и прочее


### Делаем первую БД

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/mattn/go-sqlite3"
)
type TaskStorage struct {
	db *sql.DB //указатель на подключение бд
}

func NewTaskStorage(dbpath string) (*TaskStorage, error) {
	db, err := sql.Open("sqlite3", dbpath) //говорим GO с какой БД имеем дело
	if err != nil {
		return nil, err
	}

	createTableQuery := `
	CREATE TABLE IF NOT EXISTS tasks (
		id INTEGER PRIMARY KEY AUTOINCREMENT,
		title TEXT NOT NULL,
		done BOOLEAN NOT NULL
	);
	`
	_, err = db.Exec(createTableQuery)
	if err != nil {
		return nil, fmt.Errorf("create table error: %w", err)
	}
	return &TaskStorage{db : db}, err
}
func (s *TaskStorage) CreateTask(title string) (Task, error) {
	result, err := s.db.Exec( //выполняет sql без возврата строк
		"INSERT INTO tasks (title, done) VALUES (?, ?)", //? - для защиты от SQL иньекций
		title,
		false,
	)
	if err != nil {
		return Task{}, err
	}

	id, err := result.LastInsertId() //получаем ID с sqlite
	if err != nil {
		return Task{}, err
	}
	fmt.Println("TASK SAVED:", title)

	return Task{
		ID:    int(id),
		Title: title,
		Done:  false,
	}, nil
}

func (s *TaskStorage) GetAllTasks() ([]Task, error) {
	rows, err := s.db.Query("SELECT id, title, done FROM tasks")
	if err != nil {
		return nil, err
	}
	defer rows.Close()

	tasks := make([]Task, 0)

	for rows.Next() {
		var t Task
		err := rows.Scan(&t.ID, &t.Title, &t.Done)
		if err != nil {
			return nil, err
		}
		tasks = append(tasks, t)
	}

	return tasks, nil
}

```
Тут вроде итак всё понятно, а что не понятно расписано

```go
_ "github.com/mattn/go-sqlite3"
```
Вот это супер интересная строчка, это называется пустой import. То есть мы загружаем пакет, не используем его в коде. Без `_` нам должно выдать ошибку `imported and not used` , потому что мы не обращаемся на прямую к функциям пакета.  Грубо говоря этим пакетом, мы просто добавили драйвер `sqlite3` в  реестр `database/sql` чтобы применять те же самые команды(у го нет команд для sqlite3,postgres, sql в этом пакете, так что нужно указывать)

### Ручка с POST/CREATE 

```go
func tasksHandler(storage *TaskStorage) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {

		switch r.Method {

		case http.MethodPost:
			var req createTaskRequest
			err := json.NewDecoder(r.Body).Decode(&req)
			if err != nil || req.Title == "" {
				http.Error(w, "invalid request body", http.StatusBadRequest)
				return
			}

			task, err := storage.CreateTask(req.Title)
			if err != nil {
				http.Error(w, "failed to create task", http.StatusInternalServerError)
				return
			}

			w.Header().Set("Content-Type", "application/json")
			w.WriteHeader(http.StatusCreated)
			json.NewEncoder(w).Encode(task)

		case http.MethodGet:
			tasks, err := storage.GetAllTasks()
			if err != nil {
				http.Error(w, "failed to get tasks", http.StatusInternalServerError)
				return
			}

			w.Header().Set("Content-Type", "application/json")
			json.NewEncoder(w).Encode(tasks)

		default:
			http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
		}
	}
}
```