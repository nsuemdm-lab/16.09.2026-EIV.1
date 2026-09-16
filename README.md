**9-ИС302** и **9-ИС304**, все индивидуальные проекты можно разделить на **4 основных кластера** по типу бизнес-логики. 

---

# МЕТОДИЧЕСКИЕ УКАЗАНИЯ: Расширение структуры Базы Данных (Этап 4.1)

**Цель этапа:** Переход от плоской структуры (одна таблица) к реляционной базе данных (3-5 связанных таблиц). Освоение связей «Один-ко-многим» (1:M) и «Многие-ко-многим» (M:M).

**Инструкция:** Найдите свою фамилию в одном из кластеров ниже. Изучите типовую структуру БД для вашей бизнес-модели и реализуйте её в phpMyAdmin на хостинге Beget.

---

## КЛАСТЕР 1: E-commerce, Каталоги и Доставка
**Студенты:** Вахитов, Боровинский, Рязанова, Куропат, Никольский, Полина, Рашидов, Кириченко, Сайдулин.
*(Темы: Интернет-магазины, доставка еды/воды/продуктов, аптеки, оптовые закупки).*

**Бизнес-логика:** У вас есть товары, которые принадлежат категориям. Пользователи собирают товары в корзину и оформляют заказы. Один заказ может содержать много товаров.

**Необходимые таблицы:**
1. `categories` (Категории товаров).
2. `products` (Товары — *вы уже создали её на прошлом этапе*).
3. `orders` (Заказы — кто заказал, статус, итоговая сумма).
4. `order_items` (Состав заказа — сводная таблица для связи M:M).

**SQL-шаблон для внедрения (Связь Заказа и Товаров):**
```sql
-- Таблица заказов
CREATE TABLE `orders` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `user_id` INT(11) NOT NULL,
  `total_price` DECIMAL(10,2) NOT NULL,
  `status` ENUM('new', 'processing', 'delivered', 'cancelled') DEFAULT 'new',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Сводная таблица (Что именно лежит в заказе)
CREATE TABLE `order_items` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `order_id` INT(11) NOT NULL,
  `product_id` INT(11) NOT NULL,
  `quantity` INT(11) NOT NULL DEFAULT 1,
  `price_at_moment` DECIMAL(10,2) NOT NULL COMMENT 'Цена на момент покупки',
  PRIMARY KEY (`id`),
  FOREIGN KEY (`order_id`) REFERENCES `orders`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`product_id`) REFERENCES `products`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;
```

---

## КЛАСТЕР 2: Бронирование, Услуги и Расписания
**Студенты:** Беляева, Гурбатова, Куликов, Лихачев, Никитенок, Сарсеков, Тумакова, Карстин, Киселёв, Кожевникова, Повелицина П.С., Шабаева, Носков, Повелицина А.С.
*(Темы: Мед. клиники, аренда студий/недвижимости, барбершопы, билеты, столики, фитнес).*

**Бизнес-логика:** У вас есть «Объекты» (врачи, мастера, залы, столики, места). Пользователи занимают эти объекты на определенное время. Главная проблема — не допустить пересечения времени (овербукинга).

**Необходимые таблицы:**
1. `services` / `rooms` / `masters` (Объекты бронирования).
2. `bookings` (Сами бронирования с привязкой ко времени).

**SQL-шаблон для внедрения:**
```sql
-- Таблица объектов (например, залы фотостудии или мастера)
CREATE TABLE `rooms` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(255) NOT NULL,
  `price_per_hour` DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

-- Таблица бронирований
CREATE TABLE `bookings` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `user_id` INT(11) NOT NULL,
  `room_id` INT(11) NOT NULL,
  `start_time` DATETIME NOT NULL COMMENT 'Начало брони',
  `end_time` DATETIME NOT NULL COMMENT 'Конец брони',
  `status` ENUM('pending', 'confirmed', 'completed', 'cancelled') DEFAULT 'pending',
  PRIMARY KEY (`id`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`room_id`) REFERENCES `rooms`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;
```

---

## КЛАСТЕР 3: Образование, Тестирование и Контент
**Студенты:** Вальтер, Кадыров, Каримова, Крестницкая, Лобов, Нугуманова, Федченко, Бастрыгина, Жихарев, Казарян, Миндулин.
*(Темы: Тестирование, ПДД, блоги, библиотеки, онлайн-курсы, опросы, афиши).*

**Бизнес-логика:** Иерархическая структура контента. Курс состоит из уроков. Тест состоит из вопросов, а вопросы — из ответов. Пользователи проходят тесты/курсы, и система сохраняет их результаты.

**Необходимые таблицы (на примере Тестирования):**
1. `tests` (Названия тестов).
2. `questions` (Вопросы, привязанные к тесту).
3. `answers` (Варианты ответов, привязанные к вопросу).
4. `user_results` (Результаты прохождения).

**SQL-шаблон для внедрения:**
```sql
-- Таблица вопросов
CREATE TABLE `questions` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `test_id` INT(11) NOT NULL,
  `question_text` TEXT NOT NULL,
  PRIMARY KEY (`id`),
  FOREIGN KEY (`test_id`) REFERENCES `tests`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Таблица вариантов ответов
CREATE TABLE `answers` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `question_id` INT(11) NOT NULL,
  `answer_text` VARCHAR(255) NOT NULL,
  `is_correct` BOOLEAN NOT NULL DEFAULT FALSE,
  PRIMARY KEY (`id`),
  FOREIGN KEY (`question_id`) REFERENCES `questions`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;
```

---

## КЛАСТЕР 4: CRM, Учет, Финансы и Управление
**Студенты:** Балагурова, Баратов, Баходуров, Дедов, Никулина, Осипов, Черемисина, Баймурзаев, Бернецян, Бессонов, Зерниченко, Зорькина, Ливинский, Линник, Мавлиханова, Сапожников, Тарасова, Шабанов, Турдиев, Носиков.
*(Темы: Таск-трекеры, бюджет, учет ТО, Helpdesk, бюро пропусков, склад, менеджеры паролей).*

**Бизнес-логика:** Фиксация транзакций, задач или заявок. Часто требуется история изменений (логирование) или категоризация.

**Необходимые таблицы (на примере Учета бюджета / Таск-трекера):**
1. `categories` (Категории расходов / Проекты).
2. `records` / `tasks` (Сами записи/задачи).
3. `logs` (История изменений статусов — *для продвинутых*).

**SQL-шаблон для внедрения:**
```sql
-- Категории (например, "Питание", "Транспорт" или "Проект А")
CREATE TABLE `categories` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `user_id` INT(11) NOT NULL COMMENT 'Категории могут быть личными',
  `name` VARCHAR(100) NOT NULL,
  PRIMARY KEY (`id`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Записи (Транзакции или Задачи)
CREATE TABLE `records` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `user_id` INT(11) NOT NULL,
  `category_id` INT(11) DEFAULT NULL,
  `amount` DECIMAL(10,2) DEFAULT NULL COMMENT 'Для бюджета',
  `title` VARCHAR(255) NOT NULL COMMENT 'Для задач',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`category_id`) REFERENCES `categories`(`id`) ON DELETE SET NULL
) ENGINE=InnoDB;
```

---

# ЗАДАНИЕ ДЛЯ УРОВНЯ "А" (Продвинутый уровень)

Студенты, претендующие на отличную оценку, должны реализовать в своих Моделях (MVC Models) запросы с использованием **JOIN**.

Поскольку данные теперь разнесены по разным таблицам, вы не можете просто сделать `SELECT * FROM orders`. Вам нужно получить имя пользователя, сделавшего заказ, и названия товаров.

**Инструкция:** В классе вашей Модели (например, `OrderModel.php` или `BookingModel.php`) напишите метод, который объединяет таблицы.

*Пример для Кластера 2 (Бронирования): Получить список броней вместе с именем клиента и названием зала:*
```php
public function getFullBookings() {
    $sql = "
        SELECT 
            bookings.id, 
            bookings.start_time, 
            bookings.status,
            users.username AS client_name,
            rooms.name AS room_name
        FROM bookings
        JOIN users ON bookings.user_id = users.id
        JOIN rooms ON bookings.room_id = rooms.id
        ORDER BY bookings.start_time DESC
    ";
    $stmt = $this->pdo->query($sql);
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

### ✅ Чек-лист проверки (Definition of Done):
1. В phpMyAdmin создано минимум 3 связанные таблицы.
2. В структуре таблиц (вкладка "Связи" / "Relation view") визуально видны связи `FOREIGN KEY`.
3. При удалении тестового пользователя из таблицы `users`, все его заказы/брони/задачи удаляются автоматически (срабатывает `ON DELETE CASCADE`).
4. В MVC проекте созданы соответствующие Модели для новых таблиц.
