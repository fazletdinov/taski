### Проект Taski: Описание

Приложение для планирования своих задач.
Приложение создано с помощью фрэимворка Django Rest Framework
и REACT, Nginx, gunicorn.

#### При необходимости, можете развернуть приложение у себя локально, 
#### или на удаленном сервере, хотя для этого вам понадобиться арендовать сервер

Первое что вам для этого нужно, это 
клонировать репозиторий и перейти в него в командной строке:

```
git clone https://github.com/fazletdinov/taski
```

```
cd taski
```

Cоздать и активировать виртуальное окружение:

```
python3 -m venv env
```

```
source env/bin/activate
```

Установить зависимости из файла requirements.txt
и nginx, выполните команды по очереди
```
python3 -m pip install --upgrade pip
pip install -r requirements.txt
sudo apt-get install nginx
```

Выполнить миграции:

```
python3 manage.py migrate
```
Необхоидмо собрать всю статику в одну директорию.
Расположение директории указываются в файле settings
```
STATIC_URL = 'static_backend/' 
STATIC_ROOT = BASE_DIR / 'static_backend' 
```
Далее запустите команду.

```
python manage.py collectstatic
```
Вся статика вашего бекенда соберется в файл, указанный в STATIC_ROOT
После чего создайте в директории /var/www/ директорию taski и 
поместите туда всю собранную статику с бэкенда. Делается это для того,
чтобы ваш веб-сервер Nginx мог отдавать статику.
Так же в директории taski необходимо создать директорию media
/var/www/taski/media/ и в файле settings в директории backend
необходимо поставить следующие константы

```
MEDIA_URL = '/media/' 
MEDIA_ROOT = '/var/www/taski/media/'  
```
Для frontend необходимо так же установить зависимости.
Перейдите в директорию frontend и запустите команды
по очереди
```
npm i
npm run build
```
Далее вся статика соберется в директории build, которую необходимо будет перенести
в /var/www/taski/

Далее необходимо запустить gunicorn юнитом.
Необходимо в директории /etc/systemd/system/ создать файл
gunicorn.service, где необходимо прописать следующее

```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User= Пропишите своего пользователя
WorkingDirectory=/home/Ваш юзер/taski/backend/
ExecStart=/home/Ваш юзер/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 backend.wsgi:application

[Install]
WantedBy=multi-user.target
```
Далее необходимо настроить nginx.
Внесите следующие настройки в /etc/nginx/sites-enabled/default

```
server {
    server_name Введите свое ip;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /media/ {
        alias /var/www/taski/media/;
    }

    location / {
        root   /var/www/taski;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
```
Далее запустите nginx и gunicorn следующими командами
по очереди

```
sudo systemctl enable gunicorn
sudo systemctl enable nginx
sudo systemctl start gunicorn
sudo systemctl start nginx
```
Базовые настройки готовы
