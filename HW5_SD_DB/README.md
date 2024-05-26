### Базы Данных для раздела "Лотереи" игровой платформы Steam (Новый придуманный функционал)

>Общая архитектура построена на основе ***реляционных баз данных***. Тому могут быть следующие причины:

1. ***Высокая нормализация***: База данных разбита на несколько таблиц, связанных внешними ключами, что обеспечивает высокую степень нормализации. Это минимизирует дублирование данных и улучшает целостность данных.

2. Реляционные базы данных позволяют создавать индексы на столбцах, что значительно ***повышает*** ***производительность*** запросов. Это особенно полезно для больших объемов данных, где необходимо быстро искать записи.

3. ***Совместимость***: Реляционные базы данных совместимы с широким спектром операционных систем и аппаратных платформ.

4. Возможность принимать сложные запросы и получать основную информацию о пользователях, их друзьях, счетах, играх, взаимодействиях и взаимоотношениях между пользователями, текущих лобби и так далее по ***PRIMARY KEY*** или ***ID_{"Вставьте ID того, что вы хотели бы получить"}***. <br>

### Структура Базы Данных

* ***Users*** - Таблица пользователей. Содержит ***основную информацию о пользователях***: ключевое поле user_id, имя пользователя username, данные об аватарке, времени (регистрации или активности, например).
```sql
Table Users
{
  user_id integer [primary key]
  username varchar [not null]
  avatar_image blob [note: "Image of user's avatar"]
  date_time timestamp [default: 'now()', not null]
}
```
* ***Friendship*** - Таблица дружеских взаимосвязей. Содержит ***формат user_id2friend_id***, отражающий дружеское ребро. Так, мы можем получить список друзей, которых хотим добавить в лобби и им отослать уведомления о приглашении. Также по ***friend_id*** можем получить всю основную информацию о друзьях из таблицы ***Users***. (Users where user_id == friend_id or smth like that). Поле date_time - идентификатор времени начала дружбы.

```sql
Table Friendship
{
    user_id integer
    friend_id integer [note: "Friend ID to the User_ID, this table shows the friendship connections"]
    date_time timestamp [default: 'now()', not null]
}
```
* ***LobbyUsers*** - Таблица текущих лобби и пользователей внутри них. После получения списка друзей происходит создание полей типа ***lobby_id*** - ***user_id*** - ***date_time*** (аналогично, временная характеристика). Так мы сможем контроллировать сеанс данного лобби и пользователей внутри этого сеанса.

```sql
Table LobbyUsers
{
  lobby_id integer
  user_id integer
  date_time timestamp [default: 'now()', not null]
}
```

* ***LobbyGames*** - Таблица добавленных в лобби игр. Необходима для генерации выигрыша внутри лобби. Поле ***game_id*** может быть получено как из магазина (БД всех игр), так и на основе ***intersection*** для игр, которые у участников лобби во владении.

```sql
Table LobbyGames
{
  lobby_id integer
  game_id integer
  date_time timestamp [default: 'now()', not null]
} 
```

* ***LobbyTotalPrice*** - Таблица, которая по полю ***lobby_id*** (primary key) добавляет цену, которую надо заплатить для участия в генерации выигрыша (поле ***total_price***). Также есть поле ***transaction_flag***, которое говорит, можно ли начинать генерацию в данном ***lobby_id*** или нет (Успешно ли оплатили участники).

```sql
Table LobbyTotalPrice
{
  lobby_id integer [primary key]
  total_price float [note: "Price for group to pay for roulette spin"]
  created_at timestamp [default: 'now()', not null]
  transaction_flag bool
}
```

* ***GamesInMarket*** - Таблица игр в магазине. Содержит поля game_id, имени, цены, даты релиза, game_genre_id (id жанра), аватарки. Поле ***game_genre_id*** на то и id, чтобы ускорить поиск в случае запроса фильтрации. Поэтому вынесена дополнительная таблица вида ***key-value*** для жанров, чтобы работать не со строками, а с чиселками, что оптимизирует работу с базами данных.

```sql
Table GamesInMarket
{
  game_id integer [primary key]
  game_name varchar [not null]
  game_image blob [note: "Image of game's avatar"]
  game_price float [not null]
  game_genre_id integer [note: "Genre_id of game (example, shooter)"]
  crtated_at timestamp [default: 'now()', not null]
}

Table Genres
{
  genre_id integer [primary key]
  genre_name varchar [not null]
}
```

* ***GamesOwn*** - Таблица владения играми. Содержит основную связку ***user_id*** - ***game_id***. Необходима, чтобы добавлять пользователям выигрыш, генерировать игру на основе уже имеющихся и проверять список игр для юзера, чтобы избежать добавления дублей.


```sql
Table GamesOwn
{
  user_id integer
  game_id integer
}
```

* ***BankAccount*** - Таблица балансов. Необходима для проверки возможности транзакции в лобби для юзеров и списания денег в случае подтверждения транзакций. В принципе, понятная таблица.

```sql
Table BankAccount
{
  user_id integer [primary key]
  balance float [note: "Field of balance of user"]
}
```

Визуально База Данных выглядит именно так:

![lottery_database](Steam_Lottery_database_concept\tables_lottery_updating.png)

