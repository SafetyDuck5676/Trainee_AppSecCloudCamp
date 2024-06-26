# Trainee_AppSecCloudCamp
Задание для AppSecCloudCamp
## 1. Вопросы для разогрева

1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались? 

В данный момент изучаю безопасную разработку для своей дипломной работы.

2. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было? 

Пока не было конкретно SECURITY Code Review, обычный Code Review.

3. Если у вас был опыт поиска уязвимостей, расскажите, как это было? 

В данный момент изучаю безопасную разработку для своей дипломной работы.

4. Почему вы хотите участвовать в стажировке?

Интересное направление, сам обучаюсь на информационной безопасности, и работаю программистом.

## 2. Security code review

### Часть 1. Security code review: GO

Анализ кода на GO с точки зрения безопасности выявляет несколько потенциальных уязвимостей:

1. Использование конкатенации строк для формирования SQL-запроса:
```go
query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
```
Это открытое для инъекций SQL, что может привести к SQL-инъекции.

2. Отсутствие обработки ошибок при сканировании результата запроса:
```go
err := rows.Scan(&name)
if err != nil {
    log.Fatal(err)
}
```
Это может привести к отказу в обслуживании при некорректных данных в таблице.

3. Отсутствие ограничения на количество возвращаемых записей:
```go
var products []string
for rows.Next() {
    var name string
    err := rows.Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    products = append(products, name)
}
```
Это может привести к переполнению памяти или длительному времени выполнения при большом объеме данных.

4. Отсутствие защиты базы данных с использованием защищенного пароля:
```go
db, err = sql.Open("mysql", "user:password@/dbname")
```
Это может привести к компрометации базы данных в случае утечки пароля.

### Последствия эксплуатации:

- SQL-инъекция может позволить злоумышленнику выполнить произвольные SQL-запросы к базе данных, включая удаление, изменение или чтение данных.
- Отказ в обслуживании может привести к недоступности веб-приложения.
- Переполнение памяти может привести к отказу сервера или замедлению его работы.
- Компрометация базы данных может привести к утечке конфиденциальной информации или изменению данных.

### Способы исправления уязвимостей:

1. Замена конкатенации строк на параметризованные запросы для предотвращения SQL-инъекций:
```go
query := "SELECT * FROM products WHERE name LIKE ?"
rows, err := db.Query(query, "%"+searchQuery+"%")
```

2. Обработка ошибок сканирования результата запроса и возврата корректного HTTP-статуса:
```go
if err != nil {
    http.Error(w, "Internal server error", http.StatusInternalServerError)
    log.Println(err)
    return
}
```

3. Ограничение количества возвращаемых записей с помощью использования пагинации или ограничения в запросе:
```go
query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%' LIMIT 100", searchQuery)
```

4. Использование безопасного способа хранения паролей, такого как чтение их из переменных среды:
```go
db, err = sql.Open("mysql", "user:"+os.Getenv("DB_PASSWORD")+"@/dbname")
```

### Рекомендация:

Наилучшим способом исправления этих уязвимостей является использование параметризованных запросов для предотвращения SQL-инъекций, обработка ошибок и ограничение количества возвращаемых записей. Также важно хранить конфиденциальные данные, такие как пароли, в безопасном месте, например, в переменных среды.

## 3. Моделирование Угроз

Потенциальные проблемы безопасности:

1. **Недостаточная аутентификация и авторизация (Auth Layer)**:
   - Если механизмы аутентификации и авторизации недостаточно защищены, злоумышленники могут получить доступ к конфиденциальной информации или даже модифицировать ее.
   - Результаты: компрометация личных данных пользователей, несанкционированные действия от имени пользователей.

2. **Недостаточная защита хранилища S3 (S3)**:
   - Если доступ к хранилищу S3 не настроен должным образом, злоумышленники могут получить доступ к хранимым данным, включая изображения, загруженные пользователями.
   - Результаты: утечка конфиденциальных изображений, возможное использование этих изображений в мошеннических целях.

3. **Инъекции SQL в PostgreSQL (PostgreSQL)**:
   - Если приложение не обрабатывает данные пользователей правильно перед выполнением SQL-запросов, атакующие могут выполнить SQL-инъекции и получить несанкционированный доступ к базе данных.
   - Результаты: утечка, модификация или удаление конфиденциальной информации в базе данных.

Последствия эксплуатации проблем безопасности:

1. Компрометация личных данных пользователей, что может привести к утечкам конфиденциальной информации и нарушению приватности.
2. Утрата доверия пользователей к сервису и его бренду.
3. Юридические последствия в виде штрафов и санкций за нарушение законов о защите данных.

Способы исправления уязвимостей и смягчения рисков:

1. **Аутентификация и авторизация**:
   - Внедрение механизмов двухфакторной аутентификации.
   - Использование сильных и уникальных паролей, хранение их в хэшированном виде.
   - Регулярное обновление секретов аутентификации.

2. **Защита хранилища S3**:
   - Настройка корректных политик доступа к хранилищу S3, ограничивающих доступ только для авторизованных пользователей.
   - Шифрование данных перед загрузкой в хранилище S3 и использование HTTPS при передаче данных.

3. **Защита от инъекций SQL**:
   - Использование параметризованных запросов вместо конкатенации строк для формирования SQL-запросов.
   - Внедрение механизмов валидации и фильтрации входных данных, чтобы предотвратить внедрение зловредного кода.

Уточняющие вопросы для разработчиков:

1. Как реализованы механизмы аутентификации и авторизации в сервисе?
2. Какие политики безопасности применяются к хранилищу S3 для защиты данных?
3. Какие меры безопасности применяются к базе данных PostgreSQL для защиты от инъекций SQL?
4. Есть ли аудит безопасности, чтобы обнаруживать несанкционированные действия и аномалии в системе?
5. Каковы планы по обновлению и мониторингу безопасности приложения?
