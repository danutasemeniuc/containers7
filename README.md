# Gestiunea unei aplicații multi-container cu Docker Compose

##  Scopul lucrării
Familiarizarea cu utilizarea Docker Compose pentru a organiza o aplicație web formată din 3 servicii containerizate:
- Web server: **Nginx**
- Interpret PHP: **PHP-FPM**
- Bază de date: **MariaDB**

## Sarcina
Realizați o aplicație PHP pe baza a **3 containere**: `nginx`, `php-fpm`, `mariadb`, folosind `docker-compose`.

### Clonarea/Inițializarea repository-ului
```bash
git clone git@github.com:danutasemeniuc/containers7.git
cd containers7
```

### Crearea structurii de directoare
```bash
mkdir -p mounts/site
mkdir nginx
```

###  Fișierul `.gitignore`
```bash
echo "mounts/site/*" > .gitignore
```

### Fișierul `nginx/default.conf`
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

###  Fișierul `docker-compose.yml`
```yaml
services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
    env_file:
      - app.env

  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
      - app.env

  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

###  Fișierul `mysql.env`
```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

###  Fișierul `app.env`
```env
APP_VERSION=1.0.0
```

### Scrie un fișier `index.php` de test
```php
<?php
phpinfo();
?>
```

###  Pornirea aplicației
```bash
docker-compose up -d
```

Verifică dacă serviciile rulează:
```bash
docker ps
```

##  Testarea în browser
Accesează:  
[http://localhost](http://localhost)

##  Întrebări și Răspunsuri

###  În ce ordine sunt pornite containerele?
1. `database`
2. `backend`
3. `frontend`

###  Unde sunt stocate datele bazei de date?
În volumul:
```yaml
volumes:
  db_data: {}
```

###  Cum se numesc containerele proiectului?
- `containers7_frontend_1`
- `containers7_backend_1`
- `containers7_database_1`

###  Cum adăugăm un fișier `app.env` cu `APP_VERSION`?
```bash
echo "APP_VERSION=1.0.0" > app.env
```

##  Concluzii
- Am creat o arhitectură modulară folosind Docker Compose.
- Fiecare serviciu rulează într-un container separat.
- Configurațiile sunt ușor de înțeles și extins.
- Am testat site-ul și conexiunea la baza de date.
- Ne-am familiarizat cu utilizarea fișierelor `.env`.

