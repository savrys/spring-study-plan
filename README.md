# 🌱 Spring за 3 дня: от JDBC к работающему CRUD

> Для кого: junior, Java SE + JDBC + ООП + коллекции. Цель — за 3 дня по 6–8 часов понять IoC/DI/MVC/Data JPA и написать работающее CRUD-приложение.
> Стек плана: **Java 17+**, **Spring Boot 3.5.x**, **Maven**, **H2** (потом можно PostgreSQL). Всё проверяется через `curl` и H2 Console.

---

## 🧭 1. Введение

Spring — это контейнер объектов и инфраструктура вокруг них. Ты уже писал `new UserDao(connection)`, сам открывал `Connection`, сам разбирал `ResultSet` и сам решал, когда вызвать `commit()`. Spring забирает у тебя **`new`** и **ручное управление жизненным циклом объектов**: он создаёт объекты (Bean), сам передаёт их друг другу (DI), сам поднимает Tomcat, сам превращает метод в HTTP-эндпоинт и сам генерирует SQL по имени метода репозитория.

Главная мысль: **ты пишешь классы и аннотации, а Spring решает, кто кого создаёт, кому что передать и когда открыть транзакцию.** Твой опыт с JDBC здесь — преимущество: ты понимаешь, что происходит «под капотом», и не будешь думать, что `save()` — это магия.

Что **не** учим за эти 3 дня и почему: XML-конфиги (legacy, в новых проектах не нужны), AOP (полезно, но не для старта), Spring Security (отдельная большая тема), WebFlux/реактивность (другой стек), микросервисы, Kotlin, GraalVM. Всё это — в разделе 9 «что учить после».

Три правила, которые сэкономят тебе часы: (1) не смотри видео больше 20–30 минут подряд без кода; (2) каждый шаг заканчивается работающим `curl`, а не «вроде должно работать»; (3) ошибку читай с **последней** строки стека — там настоящая причина, а не в первой.

---

## 📅 2. План по дням

### 🗓 День 1 — IoC, DI и первый REST-эндпоинт (без БД)

**🎯 Цель дня:** понять, что такое IoC-контейнер и Bean, и получить работающий `GET /api/hello`, который отвечает текстом.

**📚 Теория (1,5 ч, только эти темы):**
- IoC (Inversion of Control) и DI (Dependency Injection) — чем отличаются, зачем нужны.
- `ApplicationContext` — контейнер бинов; Bean = объект, которым управляет Spring.
- Стереотипы: `@Component`, `@Service`, `@Repository`, `@Controller` — в чём разница (семантическая).
- Инъекция зависимостей: через конструктор (правильно), через поле (плохо), через сеттер (редко).
- Spring Boot vs Spring Framework: starters, auto-configuration, встроенный Tomcat, `@SpringBootApplication`.
- Maven-минимум: `pom.xml`, GAV-координаты, `dependency`, `starter`, `mvnw`.
- Структура Boot-проекта: `src/main/java`, `src/main/resources`, `application.properties`.

**💻 Практика (5 ч):** проект `spring-day1` с нуля через start.spring.io → запуск → вывод списка бинов в консоль → `GreetingService` + `GreetingController` → `curl` → эксперименты «сломай DI и почини» → свой `@Bean`. Пошагово — в разделе **3.1**.

**✅ Чек-лист «я понял, если могу ответить»:**
1. Чем IoC отличается от DI?
2. Что делает `@SpringBootApplication` — какие три аннотации внутри?
3. Почему constructor injection лучше field injection? Назови минимум 2 причины.
4. Что будет, если на класс не повесить `@Service`, но попытаться его внедрить?
5. Что будет, если бинов одного типа два? Как выбрать нужный?
6. Чем `@Component` отличается от `@Bean`?
7. Что такое starter и почему добавление `spring-boot-starter-web` поднимает Tomcat?

**⏱ Тайминг:** теория 1,5 ч · практика 5 ч · повторение/конспект 1 ч · резерв 0,5 ч. Практика = **62%**.

**⚠️ Типичные ошибки:**
- Смотреть весь курс «Spring с нуля» до первой строчки кода — к вечеру будешь знать термины и не уметь запустить проект.
- Копировать код из туториала 2019 года: там `javax.*` и XML. В Boot 3 — `jakarta.*` и аннотации.
- Ставить `@Service` на интерфейс вместо реализации.
- Писать `@Autowired` над полем и радоваться — потом ловить NPE в тестах.
- Класть `main`-класс в `com.example.day1`, а контроллер — в соседний корень `com.example.controller`: component scan видит только свой пакет и вложенные, поэтому будет 404.
- Ждать JSON от метода, который возвращает `String`: `@RestController` вернёт `text/plain`.

---

### 🗓 День 2 — Spring Data JPA + полный CRUD через REST

**🎯 Цель дня:** заменить свой JDBC-DAO на `JpaRepository` и получить CRUD для сущности `Task` в H2, проверенный `curl`.

**📚 Теория (1 ч):**
- ORM и JPA: `@Entity`, `@Id`, `@GeneratedValue`, `@Column`, `@Table`; что такое persistence context (без глубокого разбора).
- `JpaRepository`: `save`, `findById`, `findAll`, `deleteById`, `existsById`; derived query methods (`findByDone`, `findByTitleContainingIgnoreCase`).
- `@Transactional`: зачем, где ставить, что такое dirty checking.
- `application.properties`: datasource URL, `ddl-auto`, `show-sql`, H2 Console.
- MVC-слой: `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PathVariable`, `@RequestParam`, `@RequestBody`.
- Коды ответов: 200 / 201 / 204 / 400 / 404 / 415.

**💻 Практика (5,5 ч):** проект `spring-day2` → H2 → `Task` → `TaskRepository` → `TaskService` → `TaskController` → 5 curl-запросов → проверка в H2 Console и по SQL-логам. Пошагово — в разделе **3.2**.

**✅ Чек-лист:**
1. Чем `JpaRepository` отличается от твоего `UserDao` на JDBC? Кто пишет SQL?
2. Откуда Spring знает, какой SQL выполнить для `findByDone(true)`?
3. Зачем `@Transactional` и что будет с двумя `save()` без неё?
4. Что делает `ddl-auto=update` и почему так нельзя в продакшене?
5. Почему entity нужен пустой конструктор?
6. Чем `@PathVariable` отличается от `@RequestParam`? Приведи пример URL для каждого.
7. Что вернёт сервер, если сделать POST без заголовка `Content-Type: application/json`?

**⏱ Тайминг:** теория 1 ч · практика 5,5 ч · повторение 1 ч · резерв 0,5 ч. Практика = **69%**.

**⚠️ Типичные ошибки:**
- Entity без пустого конструктора → `No default constructor for entity`.
- Нет геттеров → в JSON `{}` (Jackson сериализует только через геттеры/поля).
- `ddl-auto=create` на каждый запуск → «данные пропали» (это не баг, это create).
- `POST` без `Content-Type` → `415 Unsupported Media Type`.
- Путать `GET /api/tasks/1` (`@PathVariable`) и `GET /api/tasks?id=1` (`@RequestParam`).
- Возвращать `Task` со связью `@ManyToOne` и ловить `LazyInitializationException` (лечится DTO, см. День 3).
- Забыть про H2 in-memory: после перезапуска БД пустая — это нормально, не «сломалось».

---

### 🗓 День 3 — Валидация, ошибки, DTO, вторая сущность и финальный проект

**🎯 Цель дня:** довести CRUD до «как в настоящем проекте»: DTO, `@Valid`, единый обработчик ошибок, связь `Task → Category`, один тест и README. Финальный проект — в разделе **7.2**.

**📚 Теория (0,5 ч):**
- Зачем DTO и чем отличается от entity.
- Bean Validation: `@NotBlank`, `@Size`, `@Valid`, кто бросает `MethodArgumentNotValidException`.
- Глобальная обработка ошибок: `@RestControllerAdvice` + `@ExceptionHandler`, `ResponseEntity`.
- Связи: `@ManyToOne`, `@JoinColumn`, `FetchType.LAZY`, `optional=false`.
- Тесты: `@DataJpaTest`, `@SpringBootTest`, базовая проверка через AssertJ.

**💻 Практика (6 ч):** DTO + валидация → handler ошибок → `Category` + связь → тест репозитория и сервиса → 8–10 curl-проверок → README + git-коммиты. Пошагово — в разделе **3.3**.

**✅ Чек-лист:**
1. Зачем DTO, если есть entity? Назови 3 причины.
2. Кто и когда бросает `MethodArgumentNotValidException`?
3. Чем 400 отличается от 404 и 415?
4. Что делает `@RestControllerAdvice` и почему он должен лежать в сканируемом пакете?
5. Какой `FetchType` у `@ManyToOne` по умолчанию и почему лучше `LAZY`?
6. Почему двусторонняя связь ломает JSON и как это чинится?
7. Зачем `@Transactional(readOnly = true)` на читающих методах?

**⏱ Тайминг:** теория 0,5 ч · практика 6 ч · повторение/README 1 ч · резерв 0,5 ч. Практика = **75%**.

**⚠️ Типичные ошибки:**
- Добавить `@Valid`, но не добавить зависимость `spring-boot-starter-validation` — валидация молча не работает.
- Ловить `Exception` вместо конкретного — тогда 500 на всё и не видно причины.
- Ставить `@Transactional` на `private` метод или вызывать его из того же класса (self-invocation) — прокси не сработает.
- Оставить `@ManyToOne` без `fetch = LAZY` (по умолчанию EAGER) и тащить пол-БД на каждый запрос.
- Двусторонняя связь + Jackson → `StackOverflowError`. Решение: DTO (правильно) или `@JsonIgnore` (быстро).
- Отдавать 200 на создание и 200 на удаление. Правильно: **201** + `Location`, **204**.
- Писать тест `new TaskService()` — зависимость `null`, тест падает не по той причине. Только внедрение через `@SpringBootTest`/`@DataJpaTest`.

---

## 🔬 3. Детальный разбор практики

### 3.1 День 1. Проект `spring-day1`: контейнер и первый эндпоинт

#### Шаг 0. Проверить окружение (10 мин)

```bash
java -version
# ожидаемо: openjdk version "17.x" или "21.x"
```

Нужен **JDK 17+** (Boot 3 не работает на 11). Если стоит 8/11 — поставь 21 (Temurin/Oracle) и проверь `JAVA_HOME`. IDE: IntelliJ IDEA Community подойдёт (Ultimate удобнее, но не обязательна).

#### Шаг 1. Создать проект (10 мин)

1. Открой **https://start.spring.io**.
2. Project: **Maven**; Language: **Java**; Spring Boot: **3.5.x** (если предлагают 4.x — можно взять 3.5.x, чтобы совпадало с большинством туториалов; код из плана работает и там, и там).
3. Group: `com.example`, Artifact: `spring-day1`, Packaging: **Jar**, Java: **17** или **21**.
4. Dependencies: **Spring Web**, **Spring Boot DevTools**.
5. Generate → распаковать → открыть в IDE.

Структура (запомни её, она одинаковая во всех Boot-проектах):

```text
spring-day1/
├─ pom.xml
├─ mvnw, mvnw.cmd
└─ src/main/
   ├─ java/com/example/day1/
   │  ├─ Day1Application.java
   │  ├─ config/AppConfig.java
   │  ├─ service/GreetingService.java
   │  └─ web/GreetingController.java
   └─ resources/application.properties
```

Запуск (Windows):

```bash
.\mvnw.cmd spring-boot:run
```

Ожидаемый результат в консоли:

```text
Tomcat started on port 8080 (http)
Started Day1Application in 1.8 seconds
```

Открой `http://localhost:8080` — увидишь **Whitelabel Error Page**. Это **нормально**: сервер работает, но `/` не описан. Не пугайся.

#### Шаг 2. Точка входа и что такое `@SpringBootApplication`

```java
package com.example.day1;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Day1Application {
    public static void main(String[] args) {
        SpringApplication.run(Day1Application.class, args);
    }
}
```

`@SpringBootApplication` = три аннотации:
- `@SpringBootConfiguration` — это конфигурация;
- `@EnableAutoConfiguration` — Boot смотрит на зависимости в classpath и сам настраивает Tomcat, Jackson, DataSource и т.д.;
- `@ComponentScan` — сканирует **этот пакет и все вложенные** на предмет `@Component`/`@Service`/`@Repository`/`@Controller`.

Отсюда правило: **все свои классы клади в `com.example.day1.*` или ниже**, иначе их не найдут (частая причина 404 и `NoSuchBeanDefinitionException`).

#### Шаг 3. Посмотреть на контейнер руками (20 мин)

Добавь временный класс — он докажет, что бины реально создаются контейнером, а не тобой:

```java
package com.example.day1.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class BeanInspector implements CommandLineRunner {

    private static final Logger log = LoggerFactory.getLogger(BeanInspector.class);
    private final ApplicationContext ctx;

    public BeanInspector(ApplicationContext ctx) { this.ctx = ctx; }

    @Override
    public void run(String... args) {
        log.info("Всего бинов: {}", ctx.getBeanDefinitionCount());
        log.info("Контекст: {}", ctx.getId());
    }
}
```

Результат: в консоли `Всего бинов: ~150` (точное число зависит от версии). Это и есть IoC-контейнер: ~150 объектов уже созданы и связаны без единого твоего `new`.

#### Шаг 4. `@Service` + `@RestController` + конструкторная инъекция (1 ч)

`service/GreetingService.java`:

```java
package com.example.day1.service;

import org.springframework.stereotype.Service;
import java.time.Clock;
import java.time.LocalTime;

@Service
public class GreetingService {

    private final Clock clock;

    // зависимость приходит из контейнера, а не создаётся через new
    public GreetingService(Clock clock) { this.clock = clock; }

    public String greet(String name) {
        int hour = LocalTime.now(clock).getHour();
        String part = hour < 12 ? "Доброе утро" : hour < 18 ? "Добрый день" : "Добрый вечер";
        return part + ", " + name + "!";
    }
}
```

`config/AppConfig.java` — свой бин через `@Bean`:

```java
package com.example.day1.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.time.Clock;

@Configuration
public class AppConfig {

    @Bean
    public Clock clock() {
        return Clock.systemDefaultZone();
    }
}
```

`web/GreetingController.java`:

```java
package com.example.day1.web;

import com.example.day1.service.GreetingService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class GreetingController {

    private final GreetingService service;

    public GreetingController(GreetingService service) { this.service = service; }

    @GetMapping("/hello")
    public String hello(@RequestParam(defaultValue = "мир") String name) {
        return service.greet(name);
    }

    @GetMapping("/hello/{name}")
    public String helloPath(@PathVariable String name) {
        return service.greet(name);
    }
}
```

**Почему именно эти аннотации:**
- `@Service` — говорит контейнеру «создай этот класс и управляй им». Семантика: бизнес-логика. Технически то же самое, что `@Component`, но читаемее.
- `@Configuration` + `@Bean` — когда класс из чужой библиотеки (`Clock`) нельзя пометить `@Component`, ты создаёшь бин вручную, но всё равно отдаёшь его контейнеру.
- `@RestController` — это `@Controller` + `@ResponseBody`: возвращаемое значение пишется прямо в тело ответа (для `String` — как `text/plain`, для объекта — как JSON через Jackson).
- `@RequestMapping("/api")` — общий префикс, чтобы не дублировать `/api` в каждом методе.
- `@GetMapping` — HTTP GET. `@RequestParam` — параметр из `?name=...`, `@PathVariable` — кусок пути `/hello/Иван`.

**Проверка:**

```bash
curl.exe -i "http://localhost:8080/api/hello?name=Иван"
# HTTP/1.1 200
# Content-Type: text/plain;charset=UTF-8
# Добрый вечер, Иван!

curl.exe -i "http://localhost:8080/api/hello/Мария"
# Добрый вечер, Мария!
```

Если вместо текста — 404: проверь пакет контроллера и что `@RestController` на месте. Если 500 — смотри последнюю строку стека.

#### Шаг 5. Эксперимент: сломать DI и починить (40 мин)

Это самая полезная часть дня — ты увидишь, как контейнер реагирует на ошибки.

1. **Убери `@Service`** с `GreetingService` и перезапустись. Получишь:
   `UnsatisfiedDependencyException` → `NoSuchBeanDefinitionException: No qualifying bean of type 'GreetingService'`.
   Вывод: бин создаётся только если класс — кандидат в компоненты (аннотация + сканируемый пакет).
2. **Верни `@Service` и добавь второй `Clock`** в `AppConfig`:
   ```java
   @Bean
   public Clock utcClock() { return Clock.systemUTC(); }
   ```
   Чтобы гарантированно получить `NoUniqueBeanDefinitionException`, переименуй параметр конструктора в `anyClock`: `public GreetingService(Clock anyClock)`. Если оставить имя `clock`, Spring может выбрать бин по имени параметра — это тоже способ разрешения неоднозначности, но полагаться на него не надо. Почини двумя способами: `@Primary` на `clock()` **или** `@Qualifier("utcClock")` в конструкторе. Запомни разницу: `@Primary` — выбор по умолчанию, `@Qualifier` — точечный выбор.
3. **Сравни с field injection** (плохой вариант):
   ```java
   @Autowired private GreetingService service; // работает, но зависимость скрыта
   ```
   Почему плохо: нельзя `final`, объект нельзя создать вне контейнера (в тесте будет `null`), зависимости не видны в конструкторе, циклические зависимости всплывают позже. **Правило: всегда constructor injection.**

#### Шаг 6. Maven-минимум (30 мин)

В `pom.xml` важны три вещи:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.x</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

- `parent` задаёт версии всех библиотек — тебе не нужно указывать версии вручную.
- `starter` — набор зависимостей под задачу: `starter-web` = Spring MVC + Tomcat + Jackson + validation-api.
- `spring-boot-maven-plugin` собирает исполняемый jar: `.\mvnw.cmd package` → `java -jar target/spring-day1-0.0.1-SNAPSHOT.jar`.

Проверка дерева зависимостей:

```bash
.\mvnw.cmd dependency:tree -Dincludes=org.springframework.boot
```

#### Шаг 7. Самостоятельное задание (45 мин)

Сделай без подсказок, подсматривая только имена аннотаций:

1. `GET /api/info` → JSON `{"app":"spring-day1","time":"19:42"}`. Создай `record AppInfo(String app, String time)` и верни его из контроллера — Jackson сам сделает JSON.
2. Добавь `@Bean DateTimeFormatter formatter()` в `AppConfig` и внедри его в `GreetingService`, чтобы время форматировалось через него.
3. `GET /api/hello?name=иван&caps=true` — вернуть имя в верхнем регистре. Параметр `caps` — `@RequestParam(defaultValue = "false") boolean caps`.

```java
public record AppInfo(String app, String time) { }

@GetMapping("/info")
public AppInfo info() {
    return new AppInfo("spring-day1", LocalTime.now().toString());
}
```

Проверь оба URL через `curl.exe`. Если JSON пустой (`{}`) — проблема в отсутствии геттеров/компонентов record.

#### Шаг 8. Конспект и git (30 мин)

- `git init`, `.gitignore` (`target/`, `.idea/`), первый коммит.
- README на 5 строк: как запустить, какие URL работают.
- Выпиши своими словами: IoC, DI, Bean, `@SpringBootApplication`, constructor injection, `@Bean` vs `@Component`. Не копипастой — иначе на собеседовании не вспомнишь.

**Результат дня:** приложение запускается, `GET /api/hello` и `GET /api/hello/{name}` отвечают, ты можешь объяснить, откуда взялся `Clock` в конструкторе и почему без `@Service` всё падает.

---

### 3.2 День 2. Проект `spring-day2`: `Task` + H2 + Spring Data JPA

#### Шаг 0. Новый проект (10 мин)

start.spring.io → Artifact `spring-day2`, зависимости: **Spring Web**, **Spring Data JPA**, **H2 Database**, **Spring Boot DevTools**.

Структура:

```text
src/main/java/com/example/day2/
├─ Day2Application.java
└─ task/
   ├─ Task.java
   ├─ TaskRepository.java
   ├─ TaskService.java
   └─ TaskController.java
src/main/resources/application.properties
```

#### Шаг 1. `application.properties` (10 мин)

```properties
spring.datasource.url=jdbc:h2:mem:taskdb;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

- `jdbc:h2:mem:...` — БД в памяти, живёт пока живо приложение. `DB_CLOSE_DELAY=-1` не даёт ей умереть между соединениями.
- `ddl-auto=update` — Hibernate сам создаст/обновит таблицы по entity. Удобно для учёбы, **опасно в продакшене** (там Flyway/Liquibase).
- `show-sql=true` — ты увидишь реальные SQL-запросы. Обязательно включи: это лучший способ понять, что делает JPA.
- H2 Console — веб-интерфейс к БД: `http://localhost:8080/h2-console`.

#### Шаг 2. Entity `Task` (30 мин)

```java
package com.example.day2.task;

import jakarta.persistence.*;

@Entity
@Table(name = "tasks")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String title;

    @Column(nullable = false)
    private boolean done;

    protected Task() { } // нужен JPA/Hibernate

    public Task(String title) {
        this.title = title;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public boolean isDone() { return done; }
    public void setDone(boolean done) { this.done = done; }
}
```

**Почему аннотации:**
- `@Entity` — класс отображается на таблицу; Hibernate строит по нему метаданные.
- `@Table(name = "tasks")` — явное имя таблицы (иначе будет `TASK`; явное имя спасает от зарезервированных слов).
- `@Id` — первичный ключ; `@GeneratedValue(IDENTITY)` — автоинкремент, аналог `SERIAL`/`AUTO_INCREMENT`.
- `@Column(nullable = false, length = 100)` — ограничения колонки, аналог того, что ты писал в `CREATE TABLE`.
- Пустой конструктор обязателен: Hibernate создаёт объект без вызова твоего конструктора с аргументами.

#### Шаг 3. Repository — весь DAO в одну строку (10 мин)

```java
package com.example.day2.task;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface TaskRepository extends JpaRepository<Task, Long> {

    List<Task> findByDone(boolean done);

    List<Task> findByTitleContainingIgnoreCase(String part);
}
```

**Что произошло:** ты не написал ни одной реализации. Spring Data создаёт прокси-класс, читает имя метода `findByDone` и генерирует `select * from tasks where done = ?`. `JpaRepository<Task, Long>` — тип сущности и тип PK; оттуда бесплатно приходят `save`, `findById`, `findAll`, `deleteById`, `existsById`, `count`.

Сравни с JDBC: раньше на каждый запрос — `PreparedStatement`, `ResultSet`, маппинг. Теперь — имя метода. Если имя нестандартное — `@Query("select t from Task t where t.title like %:part%")`.

#### Шаг 4. Service + `@Transactional` (30 мин)

```java
package com.example.day2.task;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.http.HttpStatus;

import java.util.List;

@Service
public class TaskService {

    private final TaskRepository repo;

    public TaskService(TaskRepository repo) { this.repo = repo; }

    @Transactional(readOnly = true)
    public List<Task> findAll(Boolean done) {
        return done == null ? repo.findAll() : repo.findByDone(done);
    }

    @Transactional(readOnly = true)
    public Task findById(Long id) {
        return repo.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "task " + id + " not found"));
    }

    @Transactional
    public Task create(Task task) {
        task.setId(null); // защита от "создания" с чужим id
        return repo.save(task);
    }

    @Transactional
    public Task update(Long id, Task patch) {
        Task task = findById(id);          // объект уже в persistence context
        task.setTitle(patch.getTitle());
        task.setDone(patch.isDone());
        return task;                        // save() не нужен: dirty checking
    }

    @Transactional
    public void delete(Long id) {
        if (!repo.existsById(id)) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "task " + id + " not found");
        }
        repo.deleteById(id);
    }
}
```

`@Transactional` — аналог твоего `connection.setAutoCommit(false)` + `commit()` в конце + `rollback()` в `catch`. Ставится на **public** метод бина; Spring оборачивает его в прокси. `readOnly = true` — подсказка драйверу и Hibernate, что менять ничего не будем.

Dirty checking: `update` меняет поля уже загруженной сущности, и Hibernate сам сделает `UPDATE` при коммите. `save()` не обязателен — это неочевидный, но важный момент.

#### Шаг 5. Controller — 5 эндпоинтов (40 мин)

```java
package com.example.day2.task;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;
import java.util.List;

@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    private final TaskService service;

    public TaskController(TaskService service) { this.service = service; }

    @GetMapping
    public List<Task> all(@RequestParam(required = false) Boolean done) {
        return service.findAll(done);
    }

    @GetMapping("/{id}")
    public Task one(@PathVariable Long id) {
        return service.findById(id);
    }

    @PostMapping
    public ResponseEntity<Task> create(@RequestBody Task task) {
        Task saved = service.create(task);
        return ResponseEntity
                .created(URI.create("/api/tasks/" + saved.getId()))
                .body(saved);
    }

    @PutMapping("/{id}")
    public Task update(@PathVariable Long id, @RequestBody Task task) {
        return service.update(id, task);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}
```

**Почему так:**
- `@RequestBody` — взять JSON из тела и превратить в объект через Jackson. Без него параметр будет `null`.
- `ResponseEntity.created(...)` — вернуть **201** и заголовок `Location` на созданный ресурс (правильный REST-стиль).
- `@ResponseStatus(NO_CONTENT)` — **204** на удаление, без тела.
- `Boolean` (обёртка), а не `boolean`, для `done` — чтобы отличить «параметр не передан» (`null`) от `false`.

#### Шаг 6. Проверка curl-ами (40 мин)

```bat
# 1. Пустой список
curl.exe -i http://localhost:8080/api/tasks
# 200, []

# 2. Создание
curl.exe -i -X POST http://localhost:8080/api/tasks ^
  -H "Content-Type: application/json" ^
  -d "{\"title\":\"Купить молоко\",\"done\":false}"
# 201, Location: /api/tasks/1
# {"id":1,"title":"Купить молоко","done":false}

# 3. Чтение
curl.exe -i http://localhost:8080/api/tasks/1

# 4. Фильтр
curl.exe "http://localhost:8080/api/tasks?done=false"

# 5. Обновление
curl.exe -i -X PUT http://localhost:8080/api/tasks/1 ^
  -H "Content-Type: application/json" ^
  -d "{\"title\":\"Купить молоко и хлеб\",\"done\":true}"

# 6. Удаление
curl.exe -i -X DELETE http://localhost:8080/api/tasks/1
# 204

# 7. Проверка 404
curl.exe -i http://localhost:8080/api/tasks/1
```

> В PowerShell вместо `^` используй обратный апостроф `` ` `` для переноса строки, либо пиши команду в одну строку. В Git Bash/cmd — `^`. JSON в PowerShell удобнее передавать в одинарных кавычках: `-d '{"title":"Тест"}'`.

#### Шаг 7. Проверка в H2 Console (15 мин)

1. Открой `http://localhost:8080/h2-console`.
2. JDBC URL: **точно** `jdbc:h2:mem:taskdb` (как в properties, иначе увидишь пустую БД).
3. User: `sa`, пароль пустой → Connect.
4. Выполни:

```sql
SELECT * FROM TASKS;
SELECT ID, TITLE, DONE FROM TASKS WHERE DONE = TRUE;
```

Дополнительно посмотри в консоли приложения SQL, который Hibernate выполнил при `POST` — ты увидишь `insert into tasks ...`. Это и есть доказательство, что JPA работает, а не «магия».

#### Шаг 8. Самостоятельное задание (1 ч)

1. `GET /api/tasks/search?q=мол` — поиск по части title через `findByTitleContainingIgnoreCase`. Эндпоинта пока нет, метод в репозитории есть — добавь.
2. `GET /api/tasks/count` → `{"count":3}`. Используй `repo.count()` и `record CountResponse(long count)`.
3. Проверь «плохие» запросы и запиши результат в конспект:
   - POST без `Content-Type: application/json` → **415**;
   - POST с телом `{}` → **500** (`title = null`, а колонка `nullable = false`) — на Дне 3 это станет **400** с валидацией;
   - GET `/api/tasks/999` → **404** благодаря `ResponseStatusException`.
4. Перезапусти приложение: данные исчезли? Объясни, почему H2 in-memory так себя ведёт. Затем поменяй URL на `jdbc:h2:file:./taskdb` и проверь, что данные выжили после рестарта.

#### Шаг 9. Конспект и git (30 мин)

- Коммит: `feat: task crud with spring data jpa`.
- В конспект: `JpaRepository`, derived queries, `@Transactional`, `ddl-auto`, 201 vs 200, dirty checking — по 2–3 предложения своими словами.

**Результат дня:** CRUD работает целиком, данные видны в H2, в логах — реальные SQL.

---

### 3.3 День 3. Валидация, ошибки, DTO, связи, тесты

#### Шаг 0. Добавить зависимости (5 мин)

В `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Важно: начиная с Boot 2.3 `starter-web` **не** тянет Hibernate Validator. Без этой зависимости `@Valid` не делает ничего.

#### Шаг 1. DTO вместо entity в API (40 мин)

```java
package com.example.day2.task;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record TaskRequest(
        @NotBlank(message = "title не должен быть пустым")
        @Size(max = 100, message = "title длиннее 100 символов")
        String title,

        boolean done,

        Long categoryId
) { }
```

```java
package com.example.day2.task;

public record TaskResponse(Long id, String title, boolean done, String category) {

    public static TaskResponse from(Task t) {
        return new TaskResponse(
                t.getId(),
                t.getTitle(),
                t.isDone(),
                t.getCategory() == null ? null : t.getCategory().getName()
        );
    }
}
```

`record` — компактный неизменяемый класс (Java 16+). Если ещё не проходил — замени на обычный класс с `final` полями и геттерами, смысл тот же.

**Зачем DTO:** (1) не отдавать наружу внутреннюю структуру БД; (2) не давать клиенту менять `id` и служебные поля; (3) развязать API и entity, чтобы `@ManyToOne` не ломал JSON; (4) валидация — на входном DTO, а не на сущности.

#### Шаг 2. Валидация в контроллере (20 мин)

```java
@PostMapping
public ResponseEntity<TaskResponse> create(@Valid @RequestBody TaskRequest req) {
    TaskResponse saved = service.create(req);
    return ResponseEntity.created(URI.create("/api/tasks/" + saved.id())).body(saved);
}
```

`@Valid` запускает проверку ограничений. Если есть нарушения — Spring бросает `MethodArgumentNotValidException` **до** входа в метод. Без обработчика клиент получит неинформативный 400; с обработчиком — понятный JSON.

Проверка:

```bat
curl.exe -i -X POST http://localhost:8080/api/tasks ^
  -H "Content-Type: application/json" ^
  -d "{\"title\":\"\"}"
# 400, в теле — сообщение про title
```

#### Шаг 3. Единый обработчик ошибок (40 мин)

```java
package com.example.day2.common;

public class TaskNotFoundException extends RuntimeException {
    public TaskNotFoundException(Long id) {
        super("task " + id + " not found");
    }
}
```

```java
package com.example.day2.common;

import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.*;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;

import java.util.stream.Collectors;

@RestControllerAdvice
public class ApiExceptionHandler {

    public record ApiError(int status, String error, String message, String path) { }

    @ExceptionHandler(TaskNotFoundException.class)
    public ResponseEntity<ApiError> notFound(TaskNotFoundException e, HttpServletRequest req) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ApiError(404, "Not Found", e.getMessage(), req.getRequestURI()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiError> validation(MethodArgumentNotValidException e, HttpServletRequest req) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(fe -> fe.getField() + ": " + fe.getDefaultMessage())
                .collect(Collectors.joining("; "));
        return ResponseEntity.badRequest()
                .body(new ApiError(400, "Bad Request", message, req.getRequestURI()));
    }
}
```

Теперь в сервисе вместо `ResponseStatusException` бросай `TaskNotFoundException` — и все ошибки будут в одном формате:

```json
{"status":404,"error":"Not Found","message":"task 42 not found","path":"/api/tasks/42"}
```

`@RestControllerAdvice` = глобальный `try/catch` вокруг всех контроллеров. Обязательное условие: класс должен лежать в сканируемом пакете (рядом с `main` или ниже). Для `CategoryNotFoundException` добавь такой же метод (или сделай общий базовый `NotFoundException`), иначе он превратится в 500.

#### Шаг 4. Вторая сущность и связь (50 мин)

```java
package com.example.day2.category;

import jakarta.persistence.*;

@Entity
@Table(name = "categories")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    protected Category() { }

    public Category(String name) { this.name = name; }

    public Long getId() { return id; }
    public String getName() { return name; }
}
```

В `Task` добавь:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "category_id")
private Category category;

public Category getCategory() { return category; }
public void setCategory(Category category) { this.category = category; }
```

`@ManyToOne` = внешний ключ на одну категорию (аналог `FOREIGN KEY (category_id) REFERENCES categories(id)`). `LAZY` — категория загрузится только при обращении; `@JoinColumn` задаёт имя колонки.

`CategoryRepository`:

```java
public interface CategoryRepository extends JpaRepository<Category, Long> {
    Optional<Category> findByNameIgnoreCase(String name);
}
```

Исключение для категории (по аналогии с `TaskNotFoundException`):

```java
package com.example.day2.common;

public class CategoryNotFoundException extends RuntimeException {
    public CategoryNotFoundException(Long id) {
        super("category " + id + " not found");
    }
}
```

Минимальный контроллер категорий (нужен для финального проекта):

```java
package com.example.day2.category;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;
import java.util.List;

@RestController
@RequestMapping("/api/categories")
public class CategoryController {

    private final CategoryRepository repo;

    public CategoryController(CategoryRepository repo) { this.repo = repo; }

    @GetMapping
    public List<Category> all() { return repo.findAll(); }

    @PostMapping
    public ResponseEntity<Category> create(@RequestBody Category category) {
        Category saved = repo.save(category);
        return ResponseEntity.created(URI.create("/api/categories/" + saved.getId())).body(saved);
    }
}
```

В сервисе создания задачи:

```java
@Transactional
public TaskResponse create(TaskRequest req) {
    Task task = new Task();
    task.setTitle(req.title());
    task.setDone(req.done());
    if (req.categoryId() != null) {
        task.setCategory(categoryRepository.findById(req.categoryId())
                .orElseThrow(() -> new CategoryNotFoundException(req.categoryId())));
    }
    return TaskResponse.from(repo.save(task));
}
```

Не забудь добавить `CategoryRepository` вторым параметром конструктора `TaskService` — иначе IDE подсветит ошибку, а Spring скажет `NoSuchBeanDefinitionException` только если ты забыл аннотацию, а не параметр.

**Ловушка:** не делай двустороннюю связь (`Category` с `List<Task>`) без необходимости. Если сделал — Jackson уйдёт в бесконечную рекурсию. Лечение: DTO (правильно) или `@JsonIgnore` на обратной стороне (быстро).

**Не забудь перевести остальные методы контроллера на DTO:** `GET` списка → `List<TaskResponse>`, `GET /{id}` и `PUT` → `TaskResponse`. Маппинг `Task → TaskResponse` делай **внутри** `@Transactional`-метода сервиса: иначе обращение к `LAZY`-категории упадёт с `LazyInitializationException`.

#### Шаг 5. Один тест репозитория и один тест сервиса (40 мин)

```java
package com.example.day2.task;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class TaskRepositoryTest {

    @Autowired
    TaskRepository tasks;

    @Test
    void findsOnlyDoneTasks() {
        Task done = new Task("сделано");
        done.setDone(true);
        Task todo = new Task("не сделано");
        todo.setDone(false);
        tasks.saveAll(List.of(done, todo));

        List<Task> result = tasks.findByDone(true);

        assertThat(result).extracting(Task::getTitle).containsExactly("сделано");
    }
}
```

```java
package com.example.day2.task;

import com.example.day2.common.TaskNotFoundException;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.assertj.core.api.Assertions.assertThatThrownBy;

@SpringBootTest
@Transactional
class TaskServiceTest {

    @Autowired
    TaskService service;

    @Test
    void throwsWhenTaskMissing() {
        assertThatThrownBy(() -> service.findById(999L))
                .isInstanceOf(TaskNotFoundException.class);
    }
}
```

`@DataJpaTest` поднимает только JPA-часть и подменяет DataSource на встроенную H2 — быстро. `@SpringBootTest` поднимает весь контекст — медленнее, зато проверяет всё вместе. Запуск: `.\mvnw.cmd test`.

#### Шаг 6. Финальная проверка (30 мин)

Пройди сценарий целиком:

```powershell
# категория
curl.exe -i -X POST http://localhost:8080/api/categories -H "Content-Type: application/json" -d '{"name":"Работа"}'

# задача с категорией
curl.exe -i -X POST http://localhost:8080/api/tasks -H "Content-Type: application/json" -d '{"title":"Написать отчёт","done":false,"categoryId":1}'

# список
curl.exe http://localhost:8080/api/tasks

# невалидный запрос
curl.exe -i -X POST http://localhost:8080/api/tasks -H "Content-Type: application/json" -d '{"title":""}'

# несуществующая задача
curl.exe -i http://localhost:8080/api/tasks/999
```

Ожидаемо: 201 с `Location`, 200 со списком, 400 с текстом ошибки, 404 с JSON.

#### Шаг 7. Самостоятельное задание (1 ч)

1. Фильтр по категории: `GET /api/tasks?categoryId=1`. Добавь в репозиторий `List<Task> findByCategoryId(Long categoryId)` — имя метода снова само превратится в SQL.
2. Перепиши поиск из Дня 2 на `@Query` (URL тот же, `/api/tasks/search?q=отч`) — сравни, чем отличается от derived-метода:
   ```java
   @Query("select t from Task t where lower(t.title) like lower(concat('%', :q, '%'))")
   List<Task> search(@Param("q") String q);
   ```
3. Бонус: пагинация `GET /api/tasks?page=0&size=5` — добавь параметр `Pageable pageable` в метод репозитория и контроллера.
4. Сводка кодов: выпиши, какой запрос даёт 200, 201, 204, 400, 404, 415 — по одному примеру.

#### Шаг 8. README и финальный git (40 мин)

README минимум:

```markdown
# Task Tracker API

## Запуск
.\mvnw.cmd spring-boot:run

## Эндпоинты
POST /api/tasks  {"title":"...","categoryId":1}
GET  /api/tasks?done=false
GET  /api/tasks/{id}
PUT  /api/tasks/{id}
DELETE /api/tasks/{id}

## Проверка
curl.exe -i http://localhost:8080/api/tasks
```

Плюс раздел «Чему научился» на 5 пунктов и известные ограничения (in-memory H2, нет auth). Затем:

```bash
git add .
git commit -m "feat: validation, error handling, category relation"
git log --oneline
```



---

## 📋 4. Шпаргалка по аннотациям

| Аннотация | Где ставится | Что делает | Когда использовать | Аналог из моего опыта |
|---|---|---|---|---|
| `@Component` | класс | Регистрирует класс как Bean в контейнере | Свой служебный класс без чёткой роли | `new MyClass()` + запись в самодельный `Map<Class, Object>` |
| `@Service` | класс | То же, но семантически «бизнес-логика» | Сервисный слой | Твой класс логики, который ты создавал в `main` |
| `@Repository` | класс/интерфейс | Bean + перевод исключений БД в `DataAccessException` | Слой доступа к данным | `UserDao` на JDBC + `try/catch SQLException` |
| `@Controller` | класс | MVC-контроллер, возвращает View (HTML) | Если нужен Thymeleaf/JSP | Сервлет с `forward` на JSP |
| `@RestController` | класс | `@Controller` + `@ResponseBody`: возвращает данные (JSON) | REST API | Сервлет, который сам пишет JSON в `response` |
| `@Configuration` | класс | Источник определений бинов | Свои `@Bean`-методы | `main`, где ты вручную собирал объекты |
| `@Bean` | метод | Возвращаемое значение — Bean | Класс из чужой библиотеки (`Clock`, `ObjectMapper`) | Строка `new SomeService(...)` в `main` |
| `@Autowired` | конструктор/поле/сеттер | Просит контейнер внедрить зависимость | На конструкторе — всегда; на поле — никогда | Ручная передача `new UserDao(conn)` в конструктор |
| `@Qualifier("name")` | параметр/поле | Выбирает конкретный Bean по имени | Когда бинов одного типа несколько | Явно выбрать, какой из двух DAO передать |
| `@Primary` | класс/метод `@Bean` | Bean по умолчанию при неоднозначности | Один «основной» из нескольких | `if (usePostgres) dao = pgDao; else dao = h2Dao;` |
| `@SpringBootApplication` | main-класс | `@Configuration` + auto-config + component scan | Один раз на приложение | `main` + Tomcat + чтение конфига + сканирование пакетов |
| `@RequestMapping` | класс/метод | Общий префикс URL | Префикс `/api/tasks` | `@WebServlet("/api/*")` |
| `@GetMapping` | метод | HTTP GET | Чтение | `if (method.equals("GET"))` в сервлете |
| `@PostMapping` | метод | HTTP POST | Создание | `if (method.equals("POST"))` + чтение body |
| `@PutMapping` | метод | HTTP PUT | Полное обновление | `UPDATE` + разбор body |
| `@DeleteMapping` | метод | HTTP DELETE | Удаление | `DELETE` + разбор id |
| `@RequestParam` | параметр | Значение из `?key=value` | Фильтры, пагинация | `request.getParameter("done")` |
| `@PathVariable` | параметр | Значение из пути `/tasks/{id}` | ID ресурса | `path.substring(path.lastIndexOf('/')+1)` |
| `@RequestBody` | параметр | JSON из тела → объект (Jackson) | POST/PUT | `ObjectMapper.readValue(request.getInputStream(), Task.class)` |
| `@ResponseStatus` | метод/класс | Задаёт HTTP-статус | 204 на delete, 201 на create | `response.setStatus(204)` |
| `@Entity` | класс | Маппинг класса на таблицу | Любая JPA-сущность | Класс + ручной `ResultSet` → объект |
| `@Table(name=...)` | класс | Имя таблицы | Если имя отличается/зарезервировано | `CREATE TABLE tasks (...)` |
| `@Id` | поле | Первичный ключ | Всегда у entity | `PRIMARY KEY` |
| `@GeneratedValue` | поле | Автогенерация PK | `IDENTITY` для H2/Postgres/MySQL | `SERIAL` / `AUTO_INCREMENT` |
| `@Column` | поле | Имя, `nullable`, `length`, `unique` | Ограничения колонки | Описание колонки в `CREATE TABLE` |
| `@ManyToOne` | поле | Связь «много к одному» | `Task → Category` | `FOREIGN KEY` + `JOIN` |
| `@JoinColumn` | поле | Имя FK-колонки | Вместе с `@ManyToOne` | `category_id BIGINT REFERENCES ...` |
| `@Transactional` | метод/класс | Открывает транзакцию, коммитит/откатывает | Запись в БД, несколько операций | `setAutoCommit(false)` + `commit()`/`rollback()` |
| `@Valid` | параметр | Запускает Bean Validation | Входные DTO | Ручные `if (title == null || title.isBlank()) throw ...` |
| `@RestControllerAdvice` | класс | Глобальные `@ExceptionHandler` для всех контроллеров | Единый формат ошибок | Общий `try/catch` вокруг сервлетов |
| `@ExceptionHandler` | метод | Ловит конкретное исключение | 404, 400, 409 | `catch (SQLException e)` с выбором кода |
| `@Value("${...}")` | поле/параметр | Подставляет значение из properties | Одиночная настройка | `props.getProperty("app.name")` |
| `@ConfigurationProperties` | класс | Группа настроек в объект | Много связанных настроек | Свой класс-конфиг, читающий properties |
| `@SpringBootTest` | класс теста | Поднимает весь контекст | Интеграционные тесты | Ручная сборка всех объектов в тесте |
| `@DataJpaTest` | класс теста | Только JPA + встроенная H2 | Тесты репозиториев | Тест DAO с тестовой БД |

**Про `@Autowired` и конструктор:** если у класса один конструктор, `@Autowired` можно вообще не писать — Spring внедрит автоматически. Пиши конструктор с `final` полями: зависимости видны, объект всегда в валидном состоянии, тесты делаются через `new` без контейнера (для юнит-тестов сервиса с моками это важно).

---

## 🔄 5. Сравнение «как было / как стало»

| Задача | Как ты делал вручную (Java/JDBC) | Как это делает Spring |
|---|---|---|
| Создать объект | `new UserService(new UserDao(conn))` | Контейнер создаёт Bean и внедряет зависимости |
| Связать объекты | Передавал в конструктор руками | DI по типу (или `@Qualifier` по имени) |
| Подключиться к БД | `DriverManager.getConnection(url, user, pass)` | `DataSource` из `application.properties`, HikariCP-пул |
| Создать таблицу | `CREATE TABLE` вручную или SQL-скриптом | `ddl-auto` + `@Entity` (в учебном проекте) |
| Сохранить объект | `INSERT` + `PreparedStatement` + `setString` | `repository.save(task)` |
| Найти по id | `SELECT * WHERE id=?` + разбор `ResultSet` | `repository.findById(id)` → `Optional<Task>` |
| Выбрать с фильтром | `WHERE done=?` + ветвление в коде | `findByDone(true)` — SQL генерируется по имени |
| Транзакция | `setAutoCommit(false)`, `commit()`, `rollback()` в `catch` | `@Transactional` на методе |
| Принять HTTP-запрос | `HttpServlet.doGet`, `HttpServletRequest` | `@GetMapping` на методе контроллера |
| Достать параметр | `request.getParameter("name")` | `@RequestParam String name` |
| Достать id из пути | `substring` по `/` | `@PathVariable Long id` |
| Прочитать JSON | `ObjectMapper.readValue(inputStream, ...)` | `@RequestBody TaskRequest req` |
| Вернуть JSON | `response.setContentType("application/json")` + `writeValue` | Вернуть объект/DTO из `@RestController` |
| Обработать ошибку | `try/catch` в каждом сервлете, `sendError(404)` | `@RestControllerAdvice` + `@ExceptionHandler` |
| Валидация | `if (title == null \|\| title.isBlank())` | `@NotBlank` + `@Valid` |
| Запустить сервер | Скачать Tomcat, `web.xml`, war | `main` + встроенный Tomcat, `java -jar` |
| Конфигурация | Свой `config.properties` + чтение | `application.properties` + `@Value`/`@ConfigurationProperties` |

---

## 📚 6. Источники для изучения

> ⚠️ Честно о ссылках: я не даю URL там, где не уверен в актуальности — вместо этого даю канал/сайт + точный поисковый запрос. Проверяй дату (лучше 2023+) и версию Boot (3.x, `jakarta.*`). Всё основное — **на русском**; английский только для официальной документации.

### 📖 Официальная документация (английский, второй вариант)

| Источник | Язык | Формат | Что брать | Время |
|---|---|---|---|---|
| Spring Boot Reference — `docs.spring.io/spring-boot/reference/` | EN | docs | Разделы Core Features (IoC, auto-config), Web (MVC), Data (SQL/JPA) | 30–40 мин выборочно |
| Spring Framework Reference — `docs.spring.io/spring-framework/reference/` | EN | docs | Глава «The IoC Container», только первые 2 раздела | 20 мин |
| Guide «Building a RESTful Web Service» — `spring.io/guides/gs/rest-service/` | EN | туториал | Повторить то, что делаешь в День 1 | 20 мин |
| Guide «Accessing Data with JPA» — `spring.io/guides/gs/accessing-data-jpa/` | EN | туториал | День 2, если JPA не зашла | 25 мин |
| Guide «Validating Form Input» — `spring.io/guides/gs/validating-form-input/` | EN | туториал | День 3, валидация | 20 мин |
| Guide «Building REST services with Spring» — `spring.io/guides/tutorials/rest/` | EN | туториал | Правильные коды ответов и HATEOAS (можно пропустить) | 30 мин |

Это короткие гайды «сделай и запусти», а не справочники. Читай **после** того, как сделал сам, — иначе они заменят практику.

### 🎥 Видео на русском

Формат: видео и плейлисты YouTube. Язык: русский. Правило: смотри только то, что нужно для текущего шага. Один плейлист, а не пять.

- 🔥 **Sergey Nemchinskiy** (YouTube) — цикл по Spring Boot / Spring Framework на русском. Что смотреть: создание проекта, IoC/DI, контроллеры, Spring Data JPA. Зачем: самый ровный русскоязычный вход в Boot. Время: 3–5 ч выборочно (не весь плейлист). Уровень: новичок. Проверь, что плейлист свежий (2022+), и не пугайся, если часть видео про Boot 2 — концепции те же.
- **Евгений Борисов**, доклад/серия «Spring потрошитель» — как Spring устроен внутри (прокси, DI, транзакции). Зачем: понять, почему аннотации работают. Время: 1,5–2 ч. Уровень: **после** 3-го дня, не в первый.
- **«Аве Кодер»** — есть материалы по Java и фрагментарно по Spring/Hibernate. Зачем: альтернативное объяснение DI. Время: 1–2 ч. Уровень: новичок. Проверь актуальность по названиям видео.
- **«Хауди Хо»** — Spring **нет**: канал про Java SE, JavaFX, Android. Используй только если нужно повторить Java/JavaFX, для Spring не трать время.
- **Ulbi TV** — основной фокус JS/React/Node, Spring встречается фрагментарно. Как основной источник не рассчитывай; проверь поиском по каналу.
- **IT-KAMASUTRA** — фронтенд/JS, Spring нет. Не подходит.
- **Дмитрий Афанасьев** — есть Java-материалы; наличие свежего цикла именно по Spring проверь поиском по каналу, заранее не гарантирую.

Поисковые запросы на YouTube (сортировка по дате, фильтр «2023+»):
- `Spring Boot с нуля на русском`
- `Spring Framework для начинающих русский`
- `Spring Data JPA на русском`
- `REST API Spring Boot русский`
- `IoC и DI простыми словами`

### 📝 Статьи и туториалы на русском

Формат: статьи, текстовые туториалы, курсы. Язык: русский.

- **Habr** — хаб «Spring» / поиск по сайту. Рабочие запросы (они же темы статей): `Spring Boot с нуля пошагово`, `IoC и DI простыми словами`, `Spring Data JPA репозитории`, `@RestController и @RequestMapping`, `@Transactional как работает`. Что искать в статье: полный CRUD с curl, а не только теория. Проверяй, что примеры на Boot 3 (`jakarta.*`).
- **JavaRush** (`javarush.com`) — есть курс по Spring и много статей. Зачем: структурированный вход, если видео не заходит. Что брать: модуль про IoC/DI и Spring Boot. Время: 3–5 ч выборочно.
- **proglib.io** — статьи/переводы по Spring Boot, JPA, `@Transactional`. Зачем: быстрые объяснения с примерами. Ищи `Spring Boot`, `Spring Data JPA`.
- **tproger.ru** — обзорные статьи по IoC/DI и Spring для новичков. Зачем: понять термины за 15 минут.
- **Metanit** — раздела по Spring **нет**: `metanit.com/java/spring/` отдаёт 404 (проверено при составлении плана). Используй Metanit только для повторения JDBC/SQL, не ищи там Spring.

🔥 **Если времени в обрез — хватит этого (топ-4 на русском):**
1. Видео-цикл Sergey Nemchinskiy по Spring Boot (выборочно, 3–4 ч).
2. JavaRush, курс/модуль по Spring (структура и термины).
3. Одна статья на Habr с полным CRUD на Spring Boot (повторить руками).
4. Любой свежий плейлист по запросу `Spring Boot с нуля на русском` — как замена, если не зашёл пункт 1.

Сверху — официальный гайд «Accessing Data with JPA» (EN) как шпаргалка по JPA, но он не заменяет русские источники.

### 🛠 Инструменты

- **start.spring.io** — генератор проекта. Настройки: Maven, Java, Boot 3.5.x, Group `com.example`, нужные starters.
- **H2 Console** — `http://localhost:8080/h2-console`, JDBC URL как в properties, user `sa`.
- **curl** — минимум для проверки API; в PowerShell вызывай `curl.exe`, а не алиас `curl`.
- **Postman** / **IntelliJ HTTP Client** (`.http`-файлы) — удобнее для коллекции запросов, когда curl-ов станет много.
- **`.\mvnw.cmd`** — Maven Wrapper: не нужно ставить Maven глобально.
- **Git** — коммит после каждого рабочего шага. Это твоя страховка от «сломал и не помню, что менял».

---

## ✅ 7. Ресурсы для самопроверки

### 7.1 Вопросы по темам

**День 1 — IoC/DI/Boot:**
1. Чем IoC отличается от DI?
2. Что делает `@SpringBootApplication`? Назови три аннотации внутри.
3. Почему constructor injection лучше field injection?
4. Что будет, если класс не помечен стереотипом, но его пытаются внедрить?
5. Как выбрать бин, если их два?
6. Чем `@Component` отличается от `@Bean`?
7. Что такое starter?

**День 2 — Web + Data JPA:**
8. Чем `JpaRepository` отличается от JDBC-DAO?
9. Откуда берётся SQL для `findByTitleContainingIgnoreCase`?
10. Зачем `@Transactional`?
11. Что делает `ddl-auto=update`?
12. Почему entity нужен пустой конструктор?
13. `@PathVariable` vs `@RequestParam`?
14. Что вернётся на POST без `Content-Type: application/json`?

**День 3 — валидация/ошибки/связи:**
15. Зачем DTO?
16. Кто бросает `MethodArgumentNotValidException`?
17. Что делает `@RestControllerAdvice`?
18. Какой FetchType у `@ManyToOne` по умолчанию?
19. Почему двусторонняя связь ломает JSON?
20. Зачем `@Transactional(readOnly = true)`?

<details>
<summary>Показать ответы</summary>

1. IoC — принцип «управление созданием объектов отдано контейнеру»; DI — конкретный способ это сделать (внедрение зависимостей).
2. `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan`.
3. Зависимости видны и `final`; объект нельзя создать в невалидном состоянии; проще тестировать; циклические зависимости видны сразу.
4. `NoSuchBeanDefinitionException` (через `UnsatisfiedDependencyException`).
5. `@Primary` — по умолчанию, `@Qualifier("имя")` — точечно.
6. `@Component` — на своём классе, находится сканированием; `@Bean` — метод в `@Configuration`, для чужих классов и сложной инициализации.
7. Набор зависимостей под задачу; Boot по classpath понимает, что включить (web → Tomcat + MVC).
8. Интерфейс без реализации: Spring Data генерирует реализацию и SQL; твой DAO требовал ручного кода.
9. Из имени метода (derived query) через Spring Data JPA.
10. Открыть транзакцию, закоммитить при успехе, откатить при исключении.
11. Hibernate обновляет схему по entity при старте (учебно; в проде — миграции).
12. Hibernate создаёт экземпляр без вызова конструктора с аргументами.
13. `/tasks/1` — `@PathVariable`; `/tasks?done=true` — `@RequestParam`.
14. `415 Unsupported Media Type`.
15. Не отдавать entity наружу, скрыть поля, развязать API и БД, валидировать вход.
16. Spring MVC после `@Valid`, если есть нарушения ограничений.
17. Ловит исключения всех контроллеров и возвращает единый формат ответа.
18. `EAGER` — поэтому для `@ManyToOne` явно ставят `LAZY`.
19. Jackson сериализует объект A → B → A → ... → `StackOverflowError`; лечится DTO или `@JsonIgnore`.
20. Подсказка Hibernate не делать dirty checking и не держать лишнее в памяти; для чтения эффективнее.

</details>

### 7.2 Финальный мини-проект: «Трекер задач» (2 сущности)

**Сущности:** `Category` (id, name) и `Task` (id, title, done, category).

**Эндпоинты:**

| Метод | URL | Тело | Ответ |
|---|---|---|---|
| POST | `/api/categories` | `{"name":"Работа"}` | 201 + категория |
| GET | `/api/categories` | — | 200 + список |
| POST | `/api/tasks` | `{"title":"Отчёт","categoryId":1}` | 201 + `Location` |
| GET | `/api/tasks` | — | 200 + список |
| GET | `/api/tasks?done=false` | — | 200 + фильтр |
| GET | `/api/tasks/{id}` | — | 200 / 404 |
| PUT | `/api/tasks/{id}` | `{"title":"...","done":true}` | 200 / 404 |
| DELETE | `/api/tasks/{id}` | — | 204 / 404 |
| GET | `/api/tasks/search?q=отч` | — | 200 + поиск (бонус) |

**Правила:** title не пустой и ≤ 100 символов; `categoryId` может быть `null`; удаление несуществующей задачи — 404; ошибки — в едином JSON.

**Критерии приёмки:**
- [ ] Приложение стартует без ошибок, H2 Console открывается.
- [ ] Все 8 запросов из таблицы дают ожидаемые коды.
- [ ] Пустой title → 400 с понятным сообщением, а не 500.
- [ ] Несуществующий id → 404 с JSON, а не stack trace.
- [ ] `GET /api/tasks` не содержит лишних полей и не падает на `category`.
- [ ] Проходят 2 теста (`.\mvnw.cmd test`).
- [ ] README: как запустить + 6 примеров curl.
- [ ] Git: минимум 5 коммитов.

**Бонус (если осталось время):** пагинация (`Pageable`), сортировка (`Sort`), поиск через `@Query`, поле `createdAt` с `@CreationTimestamp`, Swagger через `springdoc-openapi`.

### 7.3 Если что-то не работает — таблица разбора

| Симптом | Причина | Решение |
|---|---|---|
| `Port 8080 was already in use` | Порт занят другим процессом | `server.port=8081` в properties или убить процесс: `netstat -ano \| findstr :8080`, затем `taskkill /PID <pid> /F` |
| `NoSuchBeanDefinitionException` | Нет аннотации, класс вне пакета сканирования, внедряешь интерфейс без реализации | Добавить стереотип; перенести класс в `com.example.*`; проверить тип |
| `NoUniqueBeanDefinitionException` | Два бина одного типа | `@Primary` или `@Qualifier("имя")` |
| `Failed to configure a DataSource: 'url' attribute is not specified` | Нет H2/Postgres зависимости или URL | Добавить драйвер и `spring.datasource.url` |
| `Table "TASKS" not found` | `ddl-auto=none/validate`, не та схема | `ddl-auto=update` для учёбы; проверить URL H2 |
| H2 Console 404 / пустая | Не включена или другой JDBC URL | `spring.h2.console.enabled=true`; вводить URL **точно** как в properties |
| `415 Unsupported Media Type` | Нет `Content-Type: application/json` | Добавить заголовок |
| 400 без деталей | Валидация сработала, нет обработчика | Добавить `@RestControllerAdvice` |
| В JSON `{}` или `null` | Нет геттеров / неверные имена | Добавить геттеры, проверить Jackson |
| `StackOverflowError` в JSON | Двусторонняя связь сущностей | DTO (правильно) или `@JsonIgnore` |
| `LazyInitializationException` | Обращение к LAZY-связи вне транзакции | Маппинг в DTO внутри `@Transactional` или `JOIN FETCH` |
| `@Autowired` поле `null` в тесте | Объект создан через `new` | Использовать `@SpringBootTest`/`@DataJpaTest` |
| `@Valid` не работает | Нет `spring-boot-starter-validation` | Добавить зависимость |
| `mvnw` не распознан | Windows/PowerShell | `.\mvnw.cmd ...`; проверить `JAVA_HOME` |
| `Unsupported class file major version` | JDK старше 17 | Поставить JDK 17/21 |
| 404 на твоём эндпоинте | Контроллер вне сканируемого пакета или другой путь | Проверить пакет и `@RequestMapping` |
| `JdbcSQLSyntaxErrorException` про identifier | Зарезервированное слово в имени таблицы/колонки | Переименовать, `@Table(name="...")`, `@Column(name="...")` |
| Изменения не применяются | Приложение не перезапустилось | Перезапустить; DevTools помогает не всегда |
| `Circular dependency` | A зависит от B, B от A через конструктор | Перепроектировать; `@Lazy` — костыль |
| `Schema-validation: missing table` | `ddl-auto=validate` без миграций | `update` для учёбы, Flyway для прода |

---

## 🆘 8. План на случай, если отстаю

**Приоритеты:**

| Приоритет | Что | Если времени нет |
|---|---|---|
| **P0 — must have** | IoC/DI, `@RestController`, `JpaRepository`, CRUD, H2, curl-проверки | Без этого нет MVP |
| **P1 — should have** | DTO, валидация, `@RestControllerAdvice`, связь `Task→Category` | Можно урезать до одного 400 и одного 404 |
| **P2 — nice to have** | тесты, пагинация, Swagger, README, git-история | Выбросить первым |

**Что можно смело выкинуть:** второй проект (делай всё в одном), `@ConfigurationProperties`, `@Query`, пагинацию, Swagger, тесты (если совсем горит), H2 Console (хватит SQL-логов), теорию Maven глубже `dependency:tree`.

**MVP за 3 дня (минимум, который считается успехом):**
1. Одна сущность `Task` (id, title, done).
2. H2 + `JpaRepository`.
3. 5 эндпоинтов: list, get by id, create, update, delete.
4. Корректные коды: 200/201/204/404.
5. Валидация title (хотя бы `@NotBlank`).
6. Всё проверено curl-ами, есть README с командами.

**Аварийный вариант за 60 минут** (если JPA/Boot совсем не идёт): тот же CRUD, но `TaskService` хранит `List<Task>` в памяти, без БД и JPA. Ты всё равно получишь работающее Spring MVC-приложение, DI и REST — это лучше, чем ничего. Потом заменишь `List` на репозиторий.

**Правило отставания:** отстал на 2 часа — режь теорию и P2, но не практику. Лучше один работающий эндпоинт, чем три просмотренных плейлиста.

---

## 🚀 9. Что учить после (по порядку)

1. **Spring Security** — аутентификация/авторизация, JWT, роли; без этого нет реальных проектов.
2. **Тестирование** — `@WebMvcTest` + `MockMvc`, `@MockitoBean`, Testcontainers; пиши тесты до того, как проект вырастет.
3. **Flyway/Liquibase** — миграции схемы вместо `ddl-auto`; обязательный навык для продакшена.
4. **Docker + docker-compose** — упаковать приложение и Postgres в контейнеры; дальше — деплой.
5. **AOP** — `@Aspect`, логирование, аудит, замер времени; поймёшь, как устроен `@Transactional`.
6. **PostgreSQL + JPA в реальных схемах** — индексы, `JOIN FETCH`, N+1, `@EntityGraph`, кэширование.
7. **Микросервисы** (Spring Cloud, Feign, Gateway, Kafka/RabbitMQ) — только после уверенного монолита.

---

## 📈 Прогресс-трекер

### День 1
- [ ] Проверил JDK (17+) и `java -version`
- [ ] Сгенерировал `spring-day1` на start.spring.io
- [ ] Запустил `.\mvnw.cmd spring-boot:run`, увидел «Tomcat started on port 8080»
- [ ] Открыл `localhost:8080` и понял, что Whitelabel — это нормально
- [ ] Написал `BeanInspector`, увидел количество бинов
- [ ] Создал `GreetingService` с `@Service` и конструкторной инъекцией
- [ ] Создал `AppConfig` с `@Bean Clock`
- [ ] Создал `GreetingController` с `@RestController` и `@RequestMapping`
- [ ] Проверил `GET /api/hello?name=...` через curl
- [ ] Проверил `GET /api/hello/{name}` через curl
- [ ] Сломал DI (убрал `@Service`) и прочитал ошибку
- [ ] Сделал два бина и починил через `@Primary`/`@Qualifier`
- [ ] Объяснил себе, чем `@Component` отличается от `@Bean`
- [ ] Сделал самостоятельное задание (`/api/info`, `caps`)
- [ ] Сделал `git init` и README
- [ ] Записал 7 ответов на вопросы Дня 1

### День 2
- [ ] Создал `spring-day2` с Web + Data JPA + H2
- [ ] Настроил `application.properties` (datasource, ddl-auto, show-sql, H2 Console)
- [ ] Написал `Task` с `@Entity`, `@Id`, `@GeneratedValue`, `@Column`
- [ ] Написал `TaskRepository` с derived-методами
- [ ] Написал `TaskService` с `@Transactional`
- [ ] Написал `TaskController` на 5 эндпоинтов
- [ ] Проверил POST и увидел 201 + `Location`
- [ ] Проверил GET списка и GET по id
- [ ] Проверил PUT (dirty checking) и DELETE (204)
- [ ] Проверил 404 на несуществующий id
- [ ] Зашёл в H2 Console и выполнил `SELECT * FROM TASKS`
- [ ] Увидел в логах реальный `insert`/`select`
- [ ] Добавил `/search` и `/count`
- [ ] Проверил 415 / 500 на `{}` / 404 и поведение in-memory H2
- [ ] Записал 7 ответов на вопросы Дня 2

### День 3
- [ ] Добавил `spring-boot-starter-validation`
- [ ] Сделал `TaskRequest`/`TaskResponse` (DTO)
- [ ] Поставил `@Valid` и проверил 400 на пустой title
- [ ] Написал `TaskNotFoundException`
- [ ] Написал `@RestControllerAdvice` с двумя обработчиками
- [ ] Сделал `Category` + `@ManyToOne(fetch = LAZY)`
- [ ] Сделал `CategoryController` (POST/GET)
- [ ] Проверил создание задачи с `categoryId`
- [ ] Написал `@DataJpaTest` и `@SpringBootTest`-тест, оба зелёные
- [ ] Переписал поиск на `@Query`
- [ ] Прогнал финальный сценарий из 5 curl-ов
- [ ] Написал README с запуском и примерами
- [ ] Сделал минимум 5 git-коммитов
- [ ] Записал 6 ответов на вопросы Дня 3

### Финальный проект
- [ ] Все 8 эндпоинтов работают по таблице из 7.2
- [ ] Ошибки в едином JSON-формате
- [ ] Нет `StackOverflowError` и `LazyInitializationException`
- [ ] `.\mvnw.cmd test` проходит
- [ ] README и curl-примеры готовы
- [ ] Могу объяснить вслух, что делает каждая аннотация в проекте
