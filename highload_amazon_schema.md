### Таблица `users` (Пользователи)

**Назначение:** Хранение данных зарегистрированных пользователей  
**СУБД:** PostgreSQL (шардированная)  

**Индексы:**
- PRIMARY KEY (`user_id`)
- UNIQUE (`email`)
- UNIQUE (`phone`)
- INDEX (`is_blocked`)

| Поле          | Тип       | Описание                      | Размер (байт) |
|---------------|-----------|-------------------------------|----------------|
| user_id       | UUID (PK) | Уникальный идентификатор      | 16             |
| name          | VARCHAR   | Полное имя                    | 50             |
| email         | VARCHAR   | Электронная почта             | 50             |
| phone         | VARCHAR   | Телефон                       | 15             |
| password_hash | VARCHAR   | Хеш пароля                    | 64             |
| is_blocked    | BOOLEAN   | Статус блокировки             | 1              |
| created_at    | TIMESTAMP | Дата создания                 | 8              |
| updated_at    | TIMESTAMP | Дата обновления               | 8              |
| metadata      | JSONB     | Доп. метаданные               | ~100           |

**Итого:** ~312 байт на запись

---

### Таблица `sellers` (Продавцы)

**Назначение:** Хранение информации о продавцах  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`seller_id`)
- INDEX (`contact_email`)
- INDEX (`is_verified`)

| Поле              | Тип       | Описание                 | Размер (байт) |
|-------------------|-----------|--------------------------|----------------|
| seller_id         | UUID (PK) | Уникальный идентификатор | 16             |
| company_name      | VARCHAR   | Название компании        | 50             |
| contact_email     | VARCHAR   | Контактная почта         | 50             |
| contact_phone     | VARCHAR   | Контактный телефон       | 15             |
| is_verified       | BOOLEAN   | Статус верификации       | 1              |
| seller_rating_sum | DECIMAL   | Сумма оценок             | 8              |
| created_at        | TIMESTAMP | Дата создания            | 8              |
| updated_at        | TIMESTAMP | Дата обновления          | 8              |

**Итого:** ~156 байт на запись

---

### Таблица `products_with_categories` (Товары)

**Назначение:** Каталог товаров с денормализованными и категорийными данными  
**СУБД:** PostgreSQL (OLTP) + ElasticSearch (поиск)  

**Индексы:**
- PRIMARY KEY (`product_id`)
- INDEX (`seller_id`)
- FULLTEXT (`name`)
- INDEX (`created_at`)
- GIN INDEX (`category`) – jsonb

| Поле               | Тип       | Описание                    | Размер (байт) |
|--------------------|-----------|-----------------------------|----------------|
| product_id         | UUID (PK) | Уникальный ID товара        | 16             |
| seller_id          | UUID      | Продавец                    | 16             |
| name               | VARCHAR   | Название товара             | 100            |
| description_short  | VARCHAR   | Краткое описание            | 100            |
| description_full   | TEXT      | Полное описание             | ~500           |
| price              | DECIMAL   | Цена                        | 8              |
| discount_price     | DECIMAL   | Цена со скидкой             | 8              |
| main_image_url     | VARCHAR   | Главное изображение         | 200            |
| gallery_image_urls | VARCHAR[] | Изображения                 | ~300           |
| is_age_restricted  | BOOLEAN   | Ограничение по возрасту     | 1              |
| sum_rating         | DECIMAL   | Суммарный рейтинг           | 8              |
| created_at         | TIMESTAMP | Дата создания               | 8              |
| updated_at         | TIMESTAMP | Дата обновления             | 8              |
| seller_name        | VARCHAR   | Название продавца           | 50             |
| seller_rating_sum  | DECIMAL   | Сумма рейтингов продавца    | 8              |
| seller_is_verified | BOOLEAN   | Верификация продавца        | 1              |
| category           | JSONB     | Категория (вложенно)        | ~100           |

**Итого:** ~1480 байт на запись

---

### Таблица `categories` (Категории)

**Назначение:** Иерархия категорий  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`category_id`)
- INDEX (`parent_id`)

| Поле          | Тип       | Описание                 | Размер |
|---------------|-----------|--------------------------|--------|
| category_id   | UUID (PK) | ID категории             | 16     |
| name          | VARCHAR   | Название                 | 50     |
| path          | VARCHAR   | Путь (иерархия)          | 100    |
| parent_id     | UUID      | Родительская категория   | 16     |
| product_count | INTEGER   | Кол-во товаров           | 4      |

**Итого:** ~186 байт на запись

---

### Таблица `characteristics_product` (Характеристики товаров)

**Назначение:** Характеристики и параметры товаров  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`characteristic_id`)
- INDEX (`product_id`)

| Поле              | Тип     | Описание               | Размер |
|-------------------|--------|------------------------|--------|
| characteristic_id | UUID   | ID характеристики      | 16     |
| product_id        | UUID   | ID товара              | 16     |
| name              | VARCHAR | Название               | 50     |
| value             | VARCHAR | Значение               | 50     |
| ed_izmereniya     | VARCHAR | Ед. измерения          | 20     |

**Итого:** ~152 байт на запись

---

### Таблица `orders` (Заказы)

**Назначение:** История заказов  
**СУБД:** PostgreSQL (шардированная)  

**Индексы:**
- PRIMARY KEY (`order_id`)
- INDEX (`user_id`)
- INDEX (`created_at`)

| Поле          | Тип       | Описание          | Размер |
|---------------|-----------|-------------------|--------|
| order_id      | UUID (PK) | ID заказа         | 16     |
| user_id       | UUID      | ID пользователя   | 16     |
| status        | VARCHAR   | Статус            | 20     |
| subtotal      | DECIMAL   | Сумма без налогов | 8      |
| tax           | DECIMAL   | Налог             | 8      |
| shipping_cost | DECIMAL   | Доставка          | 8      |
| total         | DECIMAL   | Общая сумма       | 8      |
| created_at    | TIMESTAMP | Дата создания     | 8      |
| updated_at    | TIMESTAMP | Обновление        | 8      |

**Итого:** ~100 байт на запись

---

### Таблица `order_items` (Позиции в заказе)

**Назначение:** Товары, входящие в заказ  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`order_item_id`)
- INDEX (`order_id`)
- INDEX (`product_id`)

| Поле             | Тип       | Описание           | Размер |
|------------------|-----------|--------------------|--------|
| order_item_id    | UUID (PK) | ID позиции         | 16     |
| order_id         | UUID      | ID заказа          | 16     |
| product_id       | UUID      | ID товара          | 16     |
| product_name     | VARCHAR   | Название товара    | 100    |
| product_image    | VARCHAR   | Изображение        | 200    |
| product_main_image| VARCHAR  | Главное изображение| 200    |
| unit_price       | DECIMAL   | Цена за шт.        | 8      |
| quantity         | INTEGER   | Кол-во             | 4      |
| total_price      | DECIMAL   | Итого              | 8      |

**Итого:** ~568 байт на запись

---

### Таблица `cart_items` (Корзина)

**Назначение:** Содержимое корзины  
**СУБД:** Redis (временная) + PostgreSQL (на заказ)  

**Индексы:**
- PRIMARY KEY (`cart_item_id`)
- INDEX (`user_id`)
- INDEX (`product_id`)

| Поле            | Тип       | Описание           | Размер |
|-----------------|-----------|--------------------|--------|
| cart_item_id    | UUID (PK) | ID позиции         | 16     |
| user_id         | UUID      | ID пользователя    | 16     |
| product_id      | UUID      | ID товара          | 16     |
| product_name    | VARCHAR   | Название товара    | 100    |
| product_main_image | VARCHAR| Главное изображение| 200    |
| unit_price      | DECIMAL   | Цена за шт.        | 8      |
| quantity        | INTEGER   | Кол-во             | 4      |
| added_at        | TIMESTAMP | Время добавления   | 8      |

**Итого:** ~368 байт на запись

---

### Таблица `reviews` (Отзывы)

**Назначение:** Отзывы пользователей  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`review_id`)
- INDEX (`product_id`)
- INDEX (`user_id`)
- INDEX (`seller_id`)

| Поле        | Тип       | Описание            | Размер |
|-------------|-----------|---------------------|--------|
| review_id   | UUID (PK) | ID отзыва           | 16     |
| user_id     | UUID      | Автор отзыва        | 16     |
| seller_id   | UUID      | Продавец            | 16     |
| product_id  | UUID      | Товар               | 16     |
| order_id    | UUID      | Связанный заказ     | 16     |
| rating      | INTEGER   | Оценка              | 4      |
| title       | VARCHAR   | Заголовок           | 50     |
| comment     | TEXT      | Текст отзыва        | ~500   |
| product_name| VARCHAR   | Название товара     | 100    |
| images      | VARCHAR[] | Изображения         | ~200   |
| is_verified | BOOLEAN   | Проверен            | 1      |
| created_at  | TIMESTAMP | Время создания      | 8      |

**Итого:** ~943 байт на запись

---

### Таблица `addresses` (Адреса)

**Назначение:** Адреса доставки  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`address_id`)
- INDEX (`user_id`)
- INDEX (`is_default`)

| Поле         | Тип       | Описание            | Размер |
|--------------|-----------|---------------------|--------|
| address_id   | UUID (PK) | ID адреса           | 16     |
| user_id      | UUID      | Пользователь        | 16     |
| type         | VARCHAR   | Тип адреса          | 10     |
| recipient    | VARCHAR   | Получатель          | 50     |
| line1        | VARCHAR   | Адрес, строка 1     | 100    |
| line2        | VARCHAR   | Адрес, строка 2     | 100    |
| city         | VARCHAR   | Город               | 50     |
| state        | VARCHAR   | Регион              | 50     |
| postal_code  | VARCHAR   | Индекс              | 20     |
| country      | VARCHAR   | Страна              | 50     |
| phone        | VARCHAR   | Телефон             | 15     |
| is_default   | BOOLEAN   | По умолчанию        | 1      |

**Итого:** ~478 байт на запись

---

### Таблица `payment_methods` (Оплата)

**Назначение:** Способы оплаты пользователя  
**СУБД:** MongoDB  

| Поле             | Тип     | Описание           | Примечание              |
|------------------|---------|--------------------|--------------------------|
| payment_method_id| UUID    | Уникальный ID      |                          |
| user_id          | UUID    | Пользователь       |                          |
| type             | VARCHAR | Тип (карта, PayPal)|                          |
| details          | JSONB   | Данные оплаты      | Гибкие структуры         |
| is_default       | BOOLEAN | По умолчанию       |                          |
| added_at         | TIMESTAMP | Дата добавления  |                          |

---

### Таблица `shipments` (Доставка)

**Назначение:** Отслеживание доставки  
**СУБД:** PostgreSQL  

**Индексы:**
- PRIMARY KEY (`shipment_id`)
- INDEX (`order_id`)
- INDEX (`tracking_number`)

| Поле               | Тип       | Описание            | Размер |
|--------------------|-----------|---------------------|--------|
| shipment_id        | UUID (PK) | ID отправки         | 16     |
| order_id           | UUID      | Заказ               | 16     |
| carrier            | VARCHAR   | Перевозчик          | 50     |
| tracking_number    | VARCHAR   | Трек-номер          | 50     |
| status             | VARCHAR   | Статус              | 20     |
| shipped_at         | TIMESTAMP | Отправлено          | 8      |
| estimated_delivery | TIMESTAMP | Прогноз доставки    | 8      |
| delivered_at       | TIMESTAMP | Дата доставки       | 8      |

**Итого:** ~176 байт на запись

---

### Таблица `search_index` (Поисковый индекс)

**Назначение:** Полнотекстовый поиск  
**СУБД:** ElasticSearch  

| Поле           | Тип       | Описание          |
|----------------|-----------|-------------------|
| product_id     | UUID (PK) | ID товара         |
| search_vector  | TEXT      | Полнотекстовый индекс |
| suggest_terms  | VARCHAR[] | Подсказки         |
| facets         | JSONB     | Фасетные фильтры  |

---

### Таблица `sessions` (Сессии пользователей)

**Назначение:** Хранение авторизационных сессий  
**СУБД:** Redis (актуальные) + PostgreSQL (архив)  

**Индексы:**
- PRIMARY KEY (`session_id`)
- INDEX (`user_id`)
- INDEX (`expires_at`)

| Поле        | Тип       | Описание           | Размер |
|-------------|-----------|--------------------|--------|
| session_id  | UUID (PK) | ID сессии          | 16     |
| user_id     | UUID      | Пользователь       | 16     |
| ip_address  | VARCHAR   | IP-адрес           | 40     |
| device_info | JSONB     | Инфо об устройстве | ~100   |
| created_at  | TIMESTAMP | Начало сессии      | 8      |
| expires_at  | TIMESTAMP | Время истечения    | 8      |

**Итого:** ~188 байт на запись
