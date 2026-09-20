# highload-2026
Курсовой проект 2026 года [курса «Разработка high-load систем»](https://education.vk.company/curriculum/program/discipline/2007/) [Корпоративной магистерской программы «Распределённые веб-сервисы / Web scale systems»](https://dws.itmo.ru/).

## Начало работы

1. Откройте template-репозиторий курса и нажмите **Use this template → Create a new repository**.
2. В поле **Owner** выберите организацию `VK-Highload-Course-2026`.
3. Создайте **Private**-репозиторий с именем `<githubUserName>-highload-2026-hw`, например:

   ```text
   VK-Highload-Course-2026/ivanov-highload-2026-hw
   ```

4. Клонируйте созданный репозиторий на компьютер:

   ```bash
   git clone git@github.com:VK-Highload-Course-2026/<githubUserName>-highload-2026-hw.git
   cd <githubUserName>-highload-2026-hw
   ```

5. Убедитесь, что GitHub Actions запускается после первого push. Результат проверки можно увидеть во вкладках **Actions** и **Checks**.

Не создавайте fork общего template и не отправляйте решения в репозиторий template. Для каждого студента должен быть отдельный приватный репозиторий внутри организации.

## Этап 1. HTTP + storage (deadline 29 сентября 2026, 23:59 МСК)
### Make
Так можно запустить тесты:
```
$ ./gradlew test
```

А вот так -- сервер:
```
$ ./gradlew run
```

### Develop
Откройте в IDE -- [IntelliJ IDEA Community Edition](https://www.jetbrains.com/idea/) нам будет достаточно.

**ВНИМАНИЕ!** При запуске тестов или сервера в IDE **необходимо** передавать Java опцию `-Xmx128m`.

### Структура решения

Всю реализацию первого этапа размещайте в пакете:

```text
ru.vk.itmo.test.solution
```

Рекомендуемая структура:

```text
ru.vk.itmo.test.solution
├── MyService.java
├── MyHttpServer.java
└── Factory.java
```

`MyService` должен реализовывать интерфейс [`Service`](src/main/java/ru/vk/itmo/Service.java) и управлять жизненным циклом HTTP-сервера.

`MyHttpServer` должен наследоваться от `one.nio.http.HttpServer` и обрабатывать HTTP-запросы. `MyService` должен содержать экземпляр `MyHttpServer`; наследоваться одновременно от `HttpServer` и `Service` не требуется.

`Factory` должна реализовывать [`ServiceFactory.Factory`](src/main/java/ru/vk/itmo/test/ServiceFactory.java), создавать `MyService` из переданного `ServiceConfig` и быть помечена аннотацией:

```java
@ServiceFactory(stage = 1)
```

Ожидаемые имена классов не обязательны, но фабрика и все классы решения должны находиться внутри пакета `ru.vk.itmo.test.solution`.

Реализуйте интерфейсы `Service` и `ServiceFactory.Factory` и поддержите следующий HTTP REST API протокол:
* HTTP `GET /v0/entity?id=<ID>` -- получить данные по ключу `<ID>`. Возвращает `200 OK` и данные или `404 Not Found`.
* HTTP `PUT /v0/entity?id=<ID>` -- создать/перезаписать (upsert) данные по ключу `<ID>`. Возвращает `201 Created`.
* HTTP `DELETE /v0/entity?id=<ID>` -- удалить данные по ключу `<ID>`. Возвращает `202 Accepted`.

Используйте свою реализацию `Dao` из предыдущего курса `2026-nosql-lsm` или референсную реализацию, если своей нет.

Проведите нагрузочное тестирование с помощью [wrk2](https://github.com/giltene/wrk2) в **одно соединение**:
* `PUT` запросами на **стабильной** нагрузке (`wrk2` должен обеспечивать заданный с помощью `-R` rate запросов) **ниже точки разладки**
* `GET` запросами на **стабильной** нагрузке по **наполненной** БД **ниже точки разладки**

Нагрузочное тестирование и профилирование должны **проводиться в одинаковых условиях** (при одинаковой нагрузке на CPU). А почему не `curl`/F5, можно узнать [здесь](http://highscalability.com/blog/2015/10/5/your-load-generator-is-probably-lying-to-you-take-the-red-pi.html) и [здесь](https://www.youtube.com/watch?v=lJ8ydIuPFeU).

Приложите полученный консольный вывод `wrk2` для обоих видов нагрузки.

Отпрофилируйте приложение (CPU и alloc) под `PUT` и `GET` нагрузкой с помощью [async-profiler](https://github.com/async-profiler/async-profiler/).
Приложите SVG-файлы FlameGraph `cpu`/`alloc` для `PUT`/`GET` нагрузки.

**Объясните** результаты нагрузочного тестирования и профилирования и приложите **текстовый отчёт** (в Markdown). Все используемые инструменты были рассмотрены на лекции -- смотрите видео запись.

### Report
Когда всё будет готово, создайте отдельную ветку и отправьте её в свой репозиторий:

```bash
git checkout -b homework-1
git add .
git commit -m "Implement homework 1"
git push -u origin homework-1
```

После этого создайте Pull Request из `homework-1` в `main` в своём репозитории и назначьте преподавателей или team `Teachers` в качестве reviewers. Не пушьте изменения напрямую в `main`.

В Pull Request должны быть реализация, результаты профилирования и Markdown-отчёт с их анализом. На всех этапах **оценивается и код, и анализ (отчёт)** — без анализа полученных результатов работа оценивается минимальным количеством баллов.

Не забывайте **отвечать на комментарии в PR** (в том числе автоматизированные) и **исправлять замечания**.
