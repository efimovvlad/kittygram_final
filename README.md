# Kittygram

## Описание

Kittygram - сервис для публикации данных о котиках.

### Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```
    git clone git@github.com:efimovvlad/kittygram_final.git
    ```
    ```
    cd kittygram_final
    ```
2. Создайте файл .env и заполните его своими данными. Пример данных указан в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram через терминал

    ```
    mkdir kittygram
    ```

3. Установка docker compose на сервер:

    ```
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:
    - ath_to_SSH — путь к файлу с SSH-ключом;
    - SSH_name — имя файла с SSH-ключом (без расширения);
    - username — ваше имя пользователя на сервере;
    - server_ip — IP вашего сервера.

    ```
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    scp -i path_to_SSH/SSH_name .env username@server_ip:/home/username/kittygram/.env
    ```

5. Запустите docker compose в режиме демона:

    ```
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в разделе server:

    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте работоспособность конфига Nginx:

    ```
    sudo nginx -t
    ```

10. Перезагружаем Nginx
    ```
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow находится в директории

    ```
    /.github/workflows/main.yml
    ```

2. Для адаптации его на своем замените username на ваш логин на DockerHub.

3. Добавьте секреты в GitHub Actions:

    ```
    DOCKER_USERNAME    # имя пользователя в DockerHub
    DOCKER_PASSWORD    # пароль пользователя в DockerHub
    HOST               # ip_address сервера
    USER               # имя пользователя
    SSH_KEY            # приватный ssh-ключ (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE     # кодовая фраза (пароль) для ssh-ключа
    TELEGRAM_TO        # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN     # токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```

### Автор
efimovvlad