# infra_sprint1
Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Проект состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## Описание проекта
Проект написан в рамках учебного курса по Python от Яндекс.Практикум.
Пользователи могут зарегистрироваться и авторизоваться, добавить нового котика на сайт или изменить существующего, загружать фотографии своих котов с кратким описанием, а также просмотреть записи других пользователей. Основная задача  - деплой проекта на удаленный сервер.

## Стек технологий использованный в проекте:
- Python 3.x
- backend: Django
- frontend: React
- Nginx
- Gunicorn
- Аутентификация по JWT-токену


# Деплой проекта на удаленный сервер

## Подключение сервера к аккаунту на GitHub
- На сервере должен быть установлен Git. Для проверки выполнить команды:
```
    sudo apt update
    git --version
```
  если Git не установлен - установить командой
```
    sudo apt install git
```

- Находясь на сервере сгенерировать пару SSH-ключей командой
```
    ssh-keygen
```
- Сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой
```
    cat .ssh/id_rsa.pub 
```
  и скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub.

## Запуск backend проекта на сервере
1. Установить пакетный менеджер и утилиту для создания виртуального окружения
```
    sudo apt install python3-pip python3-venv -y 
```
2. Клонировать проект с GitHub
```
    git clone git@github.com:matrosovmn/infra_sprint1.git
```
3. Перейти в директорию с клонированным проектом
```
    cd путь_расположения_проекта/infra_sprint1
```
4. Создать виртуальное окружение
```
    python -m venv venv
```
5. Запустить виртуальное окружение
```
    source venv/bin/activate
```
6. Установить зависимости для Python
```
    pip install -r requirements.txt
```
7. Перейти в папку backend и выполнить миграции
```
    cd infra_sprint1/backend/
```
```
    python manage.py migrate
```
8. В файле infra_sprint1/backend/kittygram_backend/settings.py проекта:
   - установить для константы `DEBUG` значение `False`
   - в переменную ALLOWED_HOSTS добавить локальные адреса, а также доменное имя или внешний IP (если есть)
```
ALLOWED_HOSTS = ['127.0.0.1', '0.0.0.0', 'localhost', 'xxx.xxx.xxx.xxx']
```
   - поменять значение переменной STATIC_URL во избежание конфликта имён и добавить STATIC_ROOT
```
STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```

9. Создать суперюзера
```
    python manage.py createsuperuser
```
10. В корневой директории проекта создать файл .env
```
    touch .env
```
11. В файле .env прописать ваш SECRET_KEY в виде: `SECRET_KEY=<ваш_ключ>`
```
    nano .env
```

## Для backend-приложения установить WSGI-сервер gunicorn
1. При активированном виртуальном окружении проекта установить пакет gunicorn
```
    pip install gunicorn
```
2. Создать файл конфигурации для автозапуска WSGi-сервера. В директории _/etc/systemd/system/_ создайте файл _gunicorn.service_
```
    sudo nano /etc/systemd/system/gunicorn.service
```
   со следующим кодом:

            [Unit]    
            Description=gunicorn daemon
            After=network.target

            [Service]    
            User=<имя-пользователя-в-системе>  # (!) заменить на собственное
            WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
            ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
            [Install]
            WantedBy=multi-user.target

   Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду
```
    which gunicorn
```
3. Запустить созданную службу и внести её в автозапуск
```
    sudo systemctl start gunicorn
```
```
    sudo systemctl enable gunicorn
```

## Запуск frontend проекта на сервере
1. Установить на сервер `Node.js`   командами
```
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```
2. Установить зависимости для frontend приложения. Из директории `infra_sprint1/frontend/` выполнить команду:
```
    npm i
```

## Установить веб-сервер Nginx, запустить и настроить его

1. На сервере выполнить команду:
```
    sudo apt install nginx -y
```
2. Запустить веб-сервер и внести его в автозапуск
```
    sudo systemctl start nginx
```
```
    sudo systemctl enable nginx
```

3. Для установки ограничений на открытые порты выполнить по очереди команды:

```
    sudo ufw allow 'Nginx Full'
```
```
    sudo ufw allow OpenSSH
```
4. Включить файервол
```
    sudo ufw enable   
```
	### собрать и разместить статику frontend-приложения.
5. Перейти в директорию _infra_sprint1/frontend/_  и скомпилировать frontend-приложение, выполнив команду
```
    npm run build   
```
   Результат сохранится в директории ..._/frontend/build/_.
   Далее (для корректной работы статики) в системную директорию веб-сервера _/var/www/_ скопировать содержимое папки _/frontend/build/_
```
    sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/kittygram/   
```
6. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа
```
    cd /var/www/kittygram/   
```
```
    mkdir media   
```
```
    udo chown -R <имя_пользователя> /var/www/infra_sprint1/media/   
```
7. Открыть файл конфигурации веб-сервера
```
    sudo nano /etc/nginx/sites-enabled/default   
```
   и изменить его настройки на следующие:
```
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }

    location / {
    root    /var/www/infra_sprint1;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```

8. Проверить корректность конфигурации
```
    sudo nginx -t   
```
8. Перезагрузить конфигурацию Nginx
```
    sudo systemctl reload nginx   
```
	### собрать и настроить статику для backend-приложения
1. Собрать статику для админки Django 
```
    python manage.py collectstatic   
```
2. Скопировать его в системную директорию веб-сервера

```
    sudo cp -r infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/   
```

## Получить SSL-сертификат для вашего доменного имени при помощи certbot
 - Зайдите на сервер и последовательно выполните команды:
```
    sudo apt install snapd   
```
```
    sudo snap install core; sudo snap refresh core   
```
```
    sudo snap install --classic certbot   
```
```
    sudo ln -s /snap/bin/certbot /usr/bin/certbot   
```
```
    sudo certbot --nginx   
```
 - Сертификат автоматически сохранится на вашем сервере в системной директории _/etc/ssl/_  Также будет автоматически изменена конфигурация Nginx: в файл _/etc/nginx/sites-enabled/default_ добавятся новые настройки и будут прописаны пути к сертификату.
 - Перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`

## Автор
Михаил Матросов - [GitHub](https://github.com/matrosovmn)
