# Aiogram3 Support Bot

Телеграм бот для общения сотрудников поддержки от имени компании с 
пользователями.

## Обновлено 25.07.2023
Этот же бот, немного обновленный, и с уже подключенной базой данных PostgreSQL
https://github.com/dvkonstantinov/aiogram-support-bot-postgres

## Описание проекта

Идея была заимсвована отсюда: https://habr.com/ru/post/539766/

Пользователи пишут свои вопросы боту компании, бот пересылает эти сообщения 
в чат поддержки, сотрудники поддержки отвечают на эти сообщения через reply.
Основной плюс - анонимизация сотрудников поддержки.

Бот работает в режиме webhook, но может работать и в режиме polling

Для обхода запрета на пересылку сообщения у пользователя, бот копирует 
содержимое и уже затем отправляет его в чат поддержки.

## Технологический стек
- [Python](https://www.python.org/)
- [Aiogram 3](https://docs.aiogram.dev/en/dev-3.x/)
- [Asyncio](https://docs.python.org/3/library/asyncio.html)
- [Aiohttp](https://github.com/aio-libs/aiohttp)
- [Docker](https://www.docker.com/)
- [Nginx](https://www.nginx.com/)
- [Ubuntu](https://ubuntu.com/)


## Типы контента, которые может пересылать бот
- Текстовые сообщения
- Фотографии
- Группы фотографий (пересылаются по одной)
- Видео
- Аудиозаписи
- Файлы

## Разворачивание образа на личном или vps сервере

### Настройка Nignx

Предполагается, что есть готовый настроенный vps сервер с установленным 
docker, docker-compose и nginx (не контейнерным).

1. Перейти в каталог sites-available
```sh
cd /etc/nginx/sites-available/
```
2. Создать файл с именем вашего домена
```sh
nano domain.example.com
```
3. Внутри написать
```sh
server {
    listen 80;

    server_name domain.example.com;

    location /telegram/ {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://127.0.0.1:7772;
    }
}
```
server_name - ваш домен с подключенным ssl сертификатом (например, Let's Encrypt)

Вместо /telegram/ можно написать любой путь, на который должны приниматься 
данные.
4. Создать ярлык в каталоге sites-enabled
```sh
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```
5. Проверить что нет ошибок в конфигурации nginx
```sh
sudo nginx -t
```
6. Перезапустить службу nginx
```sh
sudo systemctl restart nginx
```
7. Установить https соединение, выпустив ssl сертификат с помощью certbot для вашего домена
```sh
sudo certbot --nginx 
```

### Запуск бота
1. Создать бота через botfather (см. ниже), добавить бота в группу с сотрудниками поддержки, дать боту права администратора, узнать id группы (см. ниже)
2. Cкопировать этот гит на сервер любым удобным способом
3. Создать .env файл в корне со следующим содержанием:
```sh
TELEGRAM_TOKEN=<телеграм_токен_вашего_бота>
GROUP_ID=<id_группы_или_супергруппы_в_телеграме>
WEBHOOK_DOMAIN=domain.example.com
WEBHOOK_PATH=/telegram/
APP_HOST=0.0.0.0
APP_PORT=7772
GROUP_TYPE=<тип группы, об этом ниже>
```
4. Запустить сборку docker-образа и его запуск из файла docker-compose.
```sh
sudo docker-compose up -d --build
```
Ключ -d для того чтобы контейнер запустился в фоне.

### Где что брать
1. WEBHOOK_DOMAIN - домен с подключенным ssl сертификатом

2. WEBHOOK_PATH - URL путь после домена. 

В данном случае WEBHOOK_DOMAIN + WEBHOOK_PATH будет domain.example.com/telegram/

3. Token получаем при создании бота через отца ботов (https://t.me/BotFather)

4. Свой личный id или id группы узнать можно через этого бота. 
   https://t.me/myidbot . Узнать свой id - написать боту в личку, узнать id 
   группы - добавить бота в чат группы (например группы поддержки), затем 
   ввести команду ```/getgroupid``` .
   
5. GROUP_TYPE или тип чата группы. Его можно понять по ID, который выдает бот. На выбор группа или супергруппа. Примерно так выглядит id чата группы -687545526, а так - супергруппы -1001790687540. После добавления бота в группу и назначения ему прав администратора, группа может превратиться в 
   супергруппу и id поменяется. Соответственно в переменной GROUP_TYPE файла .env надо присвоить group или supergroup 

5. APP_HOST - IP на котором будет работать приложение (по умолчанию на хосте 
127.0.0.1, localhost или можно указать 0.0.0.0)

6. APP_PORT - порт, который приложение будет использовать. Порт должен быть 
уникальным и не дублировать порты других приложений, работающих на сервере 
или в Docker.
   

## Запуск в режиме polling (на локальном компьютере)
1. Скопировать гит на локальный компьютер
2. Создать файл .env (см. выше)
3. В файле .env удалить WEBHOOK_DOMAIN
4. Установить виртуальное окружение, активировать его, 
   установить зависимости из requirements.txt
```sh
python -m venv venv
pip install -r requirements.txt
```
5. Запустить main.py ```python main.py```
PS: Для запуска необходим python 3.9 или выше

## Команды бота
В чате поддержки доступна только одна команда - ```/info```. Команда 
вводится через reply на вопрос пользователя и выдает информацию о нем (Имя, 
фамилия, имя пользователя (логин, начинается с @)

## Автор
dvkonstantinov
telegram: https://t.me/Dvkonstantinov

