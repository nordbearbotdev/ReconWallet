# ReconWallet
Это краткая документация по боту.


# Содержание
- [Логика работы бота](https://github.com/nordbearbotdev/ReconWallet/edit/master/README.md#логика-работы-бота)
- [Как запустить]()
-   [Настройка окружения]()
-   [Описание]()
-   [Установка зависимостей]()
-   [Запуск]()

## Логика работы бота
```
       Меню
   ┌─────┼────────────────┬────────────────┐
   │     │                │                │
   │     │                │                │
   │     │                │                │
КОШЕЛЕК  │             ПОЛУЧЕНИЕ           │
 баланс  │          показ BTC-Адреса       │
кошелька │            и QR-кода            │
         │                                 │  
       ОТПРАВКА                          КУПИТЬ
    Ввод BTC-адреса                 Ввод BTC или RUB
      и отправка                 Информации о пльзователе
       средств                     Отправка средств 
```

## Как запустить
### Настройка окружения
Создайте `.env` файл в папке с ботом и откройте его в текстовом редакторе.
Заполните его следующим образом:
```
export BOT_TOKEN=172195613:BBFxbrBuVxPFj6ckKIqPraLv81c19Rad34Q
export WEBHOOK_URL=webhook.example.com
export PRIVATE_KEY=0e37e5feb349ce0c8е03963ddd4163b19k171b0be9ad1c7a7fe266edaedcf3
export ADMIN=205279061
export CARD_NUMBER="1231 213 1233 1323"
```
#### Описание
1) Токен бота, который вы болучается от BotFather
2) URL вебхука на который будут приходить сообщения [см. документацию BOT API](https://core.telegram.org/bots/api#setwebhook)
3) Приватный ключ от главного Bitcoin-кошелька, [см. документация Bitcoin](https://en.bitcoin.it/Private_key)
4) Telegram-Id администратора, можно получить в [@myidbot](https://t.me/myidbot)
5) Номер Сбербанк-карты для перевода

### Установка зависимостей
Для работы бота необходимо установить следующие зависимости:
* Python 3.x
* Redis-server
* Произвести установки библиотек с помощью PIP: `pip3 install -r requirements.txt`

### Запуск
1) Обновите окружение, добавив переменные из файла `.env` с помощью команды `source .env`
2) Запустите redis-server: `redis-server`
3) Для тестового запуска бота, можете воспользоваться стандартным интепритатором Python:  `python3 app.py`
4) Для продакшена советую воспользоваться [Gunicorn](http://gunicorn.org): `gunicorn app:app`

## Структура бота
### Bot-Core
Ядро бота состоит из двух модулей в корне проекта: `app.py` и `bot.py`

Первый модуль представляет из себя стартовый модуль, который запускает и инициализирует бота, он отвечает за создание `Flask` веб-приложения, создание объекта бота, инициализацию бота и вебхуков, получение и обработку данных с вебхуков.

`Bot.py` отвечает за класс `Bot`, который как представляет из центральную часть прилоежения, которая отвечает за работу бота. В нем описаны методы работы с: базой данных, Telegram API, обработку сообщений пользователя, загрузку модулей бота, работу с пользовательской сессией, генерирование пользовательских клавиатур и сообщений из шаблонов.
    
### Modules
Модули представляют из себя некоторые скрипты, сценарии, которым бот передает сообщения от пользователя, а они в свою очередь решают что делать в той или иной ситуации. 
Модули состоят из некоторых функций обработчиков **handler**'ов. Самое первое сообщение пользователя обрабатывается стандартным хендлером, который указан в конфиге в дальнейшей работе бота хендлеры могут не только получать данные, но и указывать какой хендлер будет обрабатывать следующее сообщение. Таким образом строится неявный граф обработки сообщений пользователя.

Внутри модулей хендлеры могут оперировать любой информацией: 
* Получать и записывать данные в базу данных
* Отправлять сообщение через Telegram API
* Запрашивать или отправлять средства по средствам Bitcoin API
* Отрисовывать пользовательские клавиатуры
* И многое другое..

### Config
Конфиги представляют из себя статические JSON-файлы, которые хранят необходимую боту информацию. На данный момент в боте существует три конфиг-файлов: `init.json`, `keyboards.json`, `messages.json`.

Первый файл отвечает за основные настройки бота:
* **default-handler** - стандартный обработчик сообщения, когда пользователь пишет первый раз или не указан обработчик который будет обрабатывать следующее собщение
* **menu-button** - сообщение при получении которого, бот всегда будет возвращаться в главное меню
* **comission** - Bitcoin комиссия для совершения транзакции, указвается в **сатоши!**

**keyboards.json** - отвечает за хранение шаблонов клавиатур, о работе с которыми вы можете прочитать далее
**messages.json** - отвечает за хранение шаблонов сообщений, о работе с которыми вы можете прочитать далее

### Bitcoin API
Для работы с **Bitcoin** используется библиотека [pybitcointools](https://github.com/vbuterin/pybitcointools), которая была склонирована в папку с проектом и дописана для совместимости с третьим питоном.

За работу кошельков отвечают два модуля: `wallet.py` и `main_wallet.py`. Первый - отвечает за работу с кошельками пользователей, второй - за работу с главным кошельком.


---


# Bot
Здесь описаны все публичные методы и переменные класса **Bot**.

## Методы

### Работа с базой данных
Методы отвечающие за хранение, получение и удаление информации о пользователе.


#### user_set(user_id, field, value)
Метод отвечает за запись некоторой информации для определенного пользователя, по определенному полю. Значение **value** автоматически сериализуется в **json** и записывается в базу, в нашем случае **redis**.

В качестве значения **u\_id**, передается **telegram-id** пользователя, который можно получить из объекта сообщения: **message.u\_id**. По договоренности **нулевой id** используется для хранения системных данных.



#### user_get(user_id, field, default=None)
Метод отвечает за получение некоторой информации для определенного пользователя, по определенному полю. Значение **default** отвечает за значение, которое будет возвращаться в случае если значения по данному полю не существует.

В качестве значения **u\_id**, передается **telegram-id** пользователя, который можно получить из объекта сообщения: **message.u\_id**. По договоренности **нулевой id** используется для хранения системных данных.



#### user_delete(user_id, field)
Метод отвечает за удаление некоторой информации для определенного пользователя, по определенному полю. 

### Работа с API Telegram
Для работы с Telegram API используется библиотека pytelegrambotapi. Для того, что бы взаимодействовать с telegram api обратитесь к объекту *телеграм бота*: **bot.telegram** и вызовите необходимый метод. Например, отправка сообщений: **bot.telegram.send_message(u\_id, message)**.
Подробную документацию о библиотеке читайте [здесь](https://github.com/eternnoir/pyTelegramBotAPI).

### Работа с сообщениями и калавиатурами
#### Сообщения
Сообщения в telegram предстваляют из себя обычный текст, который мы передаем параметром в функцию.
В данном боте встроена система шаблонизации сообщений, шаблоны хранятся в файле `/config/messages.json`. Сообщения используют шаблонизатор **Jinja2**, документацию читать можно [здесь](http://jinja.pocoo.org).
<br>
Так же шаблоны сообщений могут содержать в себе разметку [Markdown](https://core.telegram.org/bots/api#markdown-style), но для этого в метод **send_message** должен быть передан параметр `parse_mode="Markdown"`. 

#### Клавиатуры
Телеграм имеет два вида клавиатур: инлайн и обычные.

##### Обычные клавиатуры
Данный тип клавиатур появляется вместо клавиатуры набора текста и обычно представляют готовые варианты для набора. При нажатии на любую кнопку данной клавиатуры отправляется сообщение. 

Шаблоны клавиатур распологаются в файле `config/keyboards.json' и представляют из себя следующую структуру.
```
[
	[["Заголовок-Кнопки1", "возвращаемое значение"], ["Заголовок-Кнопки1", "возвращаемое значение"]], //строка клавиатуры
	[["Заголовок-Кнопки1", "возвращаемое значение"], ["Заголовок-Кнопки1", "возвращаемое значение"]]
]
```


Для отрисовки клавиатуры используется метод `bot.get_keyboard("имя-клавиатуры")`, после чего полученый объект передается в метод `bot.telegram.send_message` парметром `reply_markup=...`.


Хендлер который получит сообщение с вашей кнопкой должен вызвать метод `bot.get_key("название-клавиатуры", "текст-полученного-сообщения")`, после чего будет возвращена строка с значением кнопки, или `None` если такой кнопки нет.

##### Инлайн клавиатуры
Инлайн клавиатуры отличаются тем, что данная клавиатура прикреплена к сообщению и при нажжатии на нее не происходит отправка сообщения, а вызывается **callback функция**.


Формат хранения и место хранения не отличается от обычных клавиатур, однако вместо "значения" кнопки указывается имя вызываемой **callback функции** в ответ на нажатие кнопки, о том как создать подобную функцию читайте далее.

#### Хендлеры
Хендлеры представляют из себя некоторые функции, которые обрабатывают различные типы сообщений. Существует два вида хендлеров: callback и стандартные.

##### Callback-Handler
Callback хендлеры - функции обрабатывающие запросы от инлайн клавиатур. Для создания callback-handler, просто создайте функцию с двумя входными параметрами: **bot** и **query**.


Первый парметр - объект бота, с помощью которого вы можете обращаться к базе данных, отправлять сообщения и многое другое. 


Второй параметр - объект типа [inlineQuery](https://core.telegram.org/bots/api#inlinequery) который несет всю необходимую информацию о запросе.

После создания функции, просто добавьте ее в список callback хендлеров бота уазав ее имя: `bot.callback_handlers["имя-коллбека"] = функция`


Работа с этим типом хендлеров описана в главе "Клавиатуры".

##### Стандартные хендлеры
Данный тип хендлеров отличается от предыдущих тем, что отвечает за обработку обычных сообщений пользователя, в связи с чем вместо объекта **query** используется объект типа [message](https://core.telegram.org/bots/api#message)


Добавление функции в список выглядит следующим образом: `bot.handlers["имя-хенлера"] = функция`


Для установки хендлера, который будет обрабатывать следующее сообщение пользователя воспользуйтесь методом: `bot.set_next-handler(u\_id, "название-хендлера")`


Для вынужденного вызова хендлера из текущего хендлера воспользуйтесь методом: `bot.call_handler("название-хендлера", message, forward_flag=True)`

Параметр **message** - объект сообщения, который пришел в текущий хендлер, параметр **forward_flag** - указывает, ставить ли флаг в сообшении о том, что оно было переадресовано из другого хендлера, получить значение этого флага можно следующим образом: `message.forward`.


## Написание своих модулей
Для того, что бы создать свой модуль просто создайте файл с расширением *.py* в папке `/modules`. В файле обязательно должен присутсвовать метод `init(bot)`. Данный метов вызывается при инициализации модуля и в него передается объект бота. Данный метод необходим для установки начальных настроект модуля, например в этом  методе инициализируются все хендлеры (добавляются в бота), о том как это сделать читайте в главе "Хендлеры".



# Bitcoin
Для работы с Bitcoin используется библиотека **pybitcointools**, вся работа с ней реализована в модулях **main_wallet** и **wallet**.


**Wallet** - класс кошелька пользователя, при создании объекта в него передается *user id*, по которому в базе ищется приватный ключ пользователя, если его нет (пользователь впервые использует бота) - он создается. 


Класс содержит следующие методы:
* `get_curency` - Запрос текущего курса BTC/RUB
* `get_balance` -  Получение текущего баланса кошелька пользователя
* `send_money` - Отправка средств на кошелек другого пользователя

**MainWallet** - класс главного кошелька, на котором хранится запас BTC для совершения операций покупки. Класс наследует все методы **Walllet** и имеет некоторые особенности.


При создании передается приватный ключ с имеющимися на счету BTC.
Пользователь может передать в кошелек свою транзакцию для перевода денег на его кошелек (запрос на покупку), транзакция представляет из себя следующий объект:
```
{
	"id": "uGFDJK",
	"username": "@alxmamaev",
	"time": 2918481740974890827190,
	"address": "aosg9uUFBuoiqwijmdihwiNPIDnfj=FWJoj",
	"btc_value": 10,
	"rub_value": 2500000,
}
```
Администратор в свою очередь может подтвердить прохождение транзакции или отменить ее.

 кошелек имеет следующие дополнительные методы:
* `add_tx` - добавить транзакцию в список транзакций
* `get_tx` - получить транзакцию по ее *id*
* `confirm_tx` - подтвердить транзакцию по ее id
* `not_confirm_tx` - отменить транзакцию по ее Id
