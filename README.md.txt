# База данных для онлайн-сервиса доставки еды

## Цель проекта
Разработать модель базы данных для онлайн-сервиса доставки еды.  
Система должна обрабатывать:
- создание, изменение и хранение заказов;
- управление статусами заказов;
- выбор блюд и работу с корзиной;
- оплату и доставку;
- управление пользователями (клиенты, курьеры, администраторы и др.).

---

##  1. Роли пользователей и их действия

### 1.1 Клиент
- Регистрация и авторизация.
- Просмотр меню и категорий блюд.
- Добавление блюд в корзину.
- Создание заказа.
- Оплата заказа онлайн или наличными.
- Просмотр истории заказов.
- Оценка качества доставки и блюд.

### 1.2 Курьер
- Просмотр назначенных заказов.
- Подтверждение получения заказа из ресторана.
- Изменение статуса заказа: «В пути», «Доставлен».
- Получение информации о клиенте и адресе доставки.

### 1.3 Администратор
- Управление пользователями (создание, блокировка, роли).
- Управление меню (добавление, удаление, редактирование блюд и категорий).
- Управление заказами (назначение курьеров, корректировка статуса).
- Просмотр аналитики: количество заказов, выручка, популярные блюда.

### 1.4 Менеджер ресторана
- Управление доступностью блюд (есть в наличии / нет в наличии).
- Контроль времени приготовления.
- Получение отчётов по продажам.

### 1.5 Оператор колл-центра
- Создание заказа от имени клиента (по телефону).
- Внесение изменений в заказ по просьбе клиента.
- Связь с курьером при возникновении проблем.

---

##  2. Объекты (таблицы)

1. **Пользователи** (`Users`)  
   - `user_id` (PK)  
   - `name`  
   - `phone`  
   - `email`  
   - `role` (client, courier, admin, manager, operator)  
   - `password_hash`  
   - `created_at`

2. **Категории блюд** (`Categories`)  
   - `category_id` (PK)  
   - `name`  
   - `description`

3. **Блюда** (`Dishes`)  
   - `dish_id` (PK)  
   - `category_id` (FK → Categories)  
   - `name`  
   - `description`  
   - `price`  
   - `is_available` (boolean)

4. **Заказы** (`Orders`)  
   - `order_id` (PK)  
   - `user_id` (FK → Users, клиент)  
   - `courier_id` (FK → Users, курьер)  
   - `status` (создан, оплачен, готовится, в пути, доставлен, отменён)  
   - `order_date`  
   - `delivery_address`  
   - `delivery_time`  
   - `payment_status`

5. **Состав заказа** (`Order_Items`)  
   - `order_item_id` (PK)  
   - `order_id` (FK → Orders)  
   - `dish_id` (FK → Dishes)  
   - `quantity`  
   - `price`

6. **Оплата** (`Payments`)  
   - `payment_id` (PK)  
   - `order_id` (FK → Orders)  
   - `amount`  
   - `payment_method` (карта, наличные, онлайн-сервис)  
   - `payment_date`

7. **Отзывы** (`Reviews`)  
   - `review_id` (PK)  
   - `order_id` (FK → Orders)  
   - `user_id` (FK → Users)  
   - `rating` (1–5)  
   - `comment`

8. **Промокоды** (`PromoCodes`)  
   - `promo_id` (PK)  
   - `code`  
   - `discount_percent`  
   - `expiration_date`  
   - `is_active`

9. **Журнал действий** (`Logs`)  
   - `log_id` (PK)  
   - `user_id` (FK → Users)  
   - `action`  
   - `timestamp`

---

## 3. Связи между объектами

- **Users → Orders**: клиент может иметь много заказов (1:M).  
- **Users (курьеры) → Orders**: курьер может доставлять много заказов (1:M).  
- **Orders → Order_Items**: заказ содержит несколько позиций (1:M).  
- **Order_Items → Dishes**: блюдо может входить в разные заказы (M:N через Order_Items).  
- **Dishes → Categories**: каждое блюдо относится к одной категории (M:1).  
- **Orders → Payments**: у заказа есть одна оплата (1:1).  
- **Orders → Reviews**: у заказа может быть один отзыв (1:1).  
- **PromoCodes → Orders**: заказ может использовать промокод (1:M).  
- **Users → Logs**: каждый пользователь может оставлять много записей в журнале действий (1:M).  

---

## 4. Схема объектной модели

```mermaid
erDiagram
    USERS ||--o{ ORDERS : "создаёт"
    USERS ||--o{ ORDERS : "назначен (курьер)"
    ORDERS ||--o{ ORDER_ITEMS : "содержит"
    DISHES ||--o{ ORDER_ITEMS : "входит в"
    DISHES }o--|| CATEGORIES : "относится к"
    ORDERS ||--|| PAYMENTS : "оплачивается"
    ORDERS ||--o| REVIEWS : "имеет"
    PROMOCODES ||--o{ ORDERS : "применяется"
    USERS ||--o{ LOGS : "совершает действие"

    USERS {
        int user_id PK
        string name
        string phone
        string email
        string role
        string password_hash
        datetime created_at
    }

    CATEGORIES {
        int category_id PK
        string name
        string description
    }

    DISHES {
        int dish_id PK
        int category_id FK
        string name
        string description
        decimal price
        boolean is_available
    }

    ORDERS {
        int order_id PK
        int user_id FK
        int courier_id FK
        string status
        datetime order_date
        string delivery_address
        datetime delivery_time
        string payment_status
    }

    ORDER_ITEMS {
        int order_item_id PK
        int order_id FK
        int dish_id FK
        int quantity
        decimal price
    }

    PAYMENTS {
        int payment_id PK
        int order_id FK
        decimal amount
        string payment_method
        datetime payment_date
    }

    REVIEWS {
        int review_id PK
        int order_id FK
        int user_id FK
        int rating
        string comment
    }

    PROMOCODES {
        int promo_id PK
        string code
        int discount_percent
        datetime expiration_date
        boolean is_active
    }

    LOGS {
        int log_id PK
        int user_id FK
        string action
        datetime timestamp
    }
