# student-client — JavaFX-фронтенд для fun-student

Десктопный клиент для REST API `fun-student` (Spring Boot, JWT).
Работает с сервером по HTTP, бэкенд не изменён.

## Требования

- JDK 21
- Maven 3.9+ (JavaFX скачивается Maven'ом автоматически, отдельно ставить не нужно)
- Запущенный бэкенд `fun-student` (по умолчанию `http://localhost:8080`) и PostgreSQL для него

## Запуск

```bash
# 1. Бэкенд (в папке fun-student-master)
./mvnw spring-boot:run

# 2. Клиент (в папке student-client)
mvn javafx:run
```

Или собрать запускаемый jar:

```bash
mvn package
java -jar target/student-client-1.0.0-all.jar
```

> jar содержит нативные библиотеки JavaFX только для той ОС, на которой собран.

В IntelliJ IDEA: открыть `pom.xml` как проект и запустить класс `com.example.studentclient.Launcher`.

## Учётные записи

Сервер при первом запуске создаёт:

| Email                    | Пароль       | Что доступно                                    |
|--------------------------|--------------|-------------------------------------------------|
| `superadmin@example.com` | `superAdmin` | вкладка «Пользователи»                          |
| `admin@example.com`      | `admin`      | студенты: просмотр, создание, изменение, удаление |
| `teacher@example.com`    | `teacher`    | студенты: только просмотр                       |

Права повторяют `@PreAuthorize` на сервере: SUPER_ADMIN **не** видит студентов, т.к. сервер ему этого не разрешает.

## Возможности

- **Вход** — email/пароль, адрес сервера (запоминается), понятные ошибки: неверный пароль, сервер недоступен.
- **Студенты** — таблица с сортировкой и поиском, карточка выбранного студента (телефон, email),
  создание, изменение, удаление с подтверждением. Двойной щелчок — изменить, `Delete` — удалить.
- **Пользователи** (SUPER_ADMIN) — список с ролями, создание, изменение ролей, удаление.
- Все запросы выполняются в фоне, окно не зависает.
- По истечении JWT (1 час) клиент возвращает на экран входа с сообщением.

## Ограничения, идущие от API (бэкенд не менялся)

| Что                              | Почему                                                                         | Как сделано в клиенте                         |
|----------------------------------|--------------------------------------------------------------------------------|-----------------------------------------------|
| У студента нельзя изменить имя/фамилию | `PUT /student/{id}` принимает только `phoneNumber` и `email`               | поля заблокированы, есть подсказка            |
| У пользователя меняются только роли | `updateUser` ищет пользователя по email, пароль не трогает, отвечает пустым телом | email заблокирован, пароль скрыт       |
| Дубликат email при создании пользователя | сервер не обрабатывает исключение и отвечает ошибкой без текста        | клиент проверяет email по списку заранее      |
| Неверный пароль → 403 без тела   | нет обработчика `BadCredentialsException`                                      | 401/403 на входе = «Неверный email или пароль» |
| Нельзя удалить себя / снять с себя SUPER_ADMIN | роли читаются из БД на каждом запросе — доступ пропадёт сразу      | кнопка и чекбокс заблокированы                 |

## Структура

```
src/main/java/com/example/studentclient/
├── Launcher.java              точка входа для jar/IDE
├── StudentClientApp.java      javafx.application.Application
├── AppContext.java            окно, API-клиент, переключение экранов
├── api/
│   ├── ApiClient.java         все запросы к REST API (java.net.http + Jackson)
│   ├── ApiException.java      ошибки с русскими сообщениями
│   ├── Session.java           JWT, email, роли
│   ├── Role.java
│   └── dto/Dto.java           record-DTO под JSON сервера
└── ui/
    ├── LoginController.java
    ├── MainController.java        вкладки по ролям
    ├── StudentsController.java
    ├── StudentDialogController.java
    ├── UsersController.java
    ├── UserDialogController.java
    ├── Async.java                 фоновые запросы (javafx.concurrent.Task)
    └── Ui.java                    ошибки, подтверждения, диалоги-формы
src/main/resources/com/example/studentclient/
├── login.fxml, main.fxml, students.fxml, users.fxml
├── student-dialog.fxml, user-dialog.fxml
└── styles.css
```

FXML-файлы можно открыть в Scene Builder.

Чтобы убрать ссылки с тестовыми учётками с экрана входа, удалите блок `demo-box` в `login.fxml`
и методы `fill*` в `LoginController`.
