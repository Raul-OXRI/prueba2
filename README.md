# Entorno Docker para Laravel 11 (PHP 8.2 + PostgreSQL externa)

## Servicios incluidos

- **app**: PHP 8.2 FPM con Composer y extensiones necesarias para Laravel 11 + PostgreSQL.
- **web**: Nginx para servir Laravel.

## Estructura esperada

El código de Laravel se guarda dentro del volumen Docker `app_code`. recomendable que usted tenga ssh y conectada con github para hacer mas factible la conexion dentro del entorno de debe de crear un .env para hacer la clonaxion más factible se debe de colcoar esto para poder clonar el proyecto:

```.env
SSH_PATH=C:\Users\joseb\.ssh
GIT_SSH_COMMAND=ssh -i /root/.ssh/id_ed25519 -o StrictHostKeyChecking=no
```

## Primer uso

1. Crear y levantar contenedores:

```bash
docker-compose up -d --build
```

2. Limpiar el volumen de código (trae `/var/www/html` por defecto):

```bash
docker-compose run --rm app bash -lc "rm -rf /var/www/html"
```

3. Clonar el proyecto Laravel dentro del contenedor (volumen `app_code`) usando SSH de tu PC:


```bash
docker-compose run --rm app bash -lc "rm -rf /var/www/* /var/www/.[!.]* /var/www/..?* 2>/dev/null || true; mkdir -p /tmp/ssh; cp /root/.ssh/id_ed25519 /tmp/ssh/id_ed25519; chmod 600 /tmp/ssh/id_ed25519; GIT_SSH_COMMAND='ssh -F /dev/null -i /tmp/ssh/id_ed25519 -o IdentitiesOnly=yes -o StrictHostKeyChecking=accept-new' git clone git@github.com:Raul-OXRI/reporte.git /var/www"
```

4. Instalar dependencias:

```bash
docker-compose exec app composer install --no-dev --optimize-autoloader
```

5. Crear archivo `.env` del proyecto:

```bash
docker-compose exec app bash -lc "cp -n /var/www/.env.example /var/www/.env"
```

6. Generar llave de aplicación:

```bash
docker-compose exec app php artisan key:generate
```

7. (Opcional) Si aparece error `500`, corregir permisos y limpiar caché:

```bash
docker-compose exec app bash -lc "chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache; chmod -R ug+rwx /var/www/storage /var/www/bootstrap/cache; php artisan optimize:clear"
```

# en casos que no se pueda conectar en base de produccion hacer un backup y subirla a un docker

```bash
version: "3.8"

services:
  postgres:
    image: postgres:18
    container_name: dev-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      # No defino POSTGRES_DB para quedarme con la BD "postgres" por defecto
      # desde ahí creas todas las demás
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql 

  pgadmin:
    image: dpage/pgadmin4
    container_name: dev-pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: dev@local.com
      PGADMIN_DEFAULT_PASSWORD: devpass
    ports:
      - "8080:80"
    depends_on:
      - postgres

volumes:
  postgres_data:
```
```bash
docker-compose up -d
```

```bash
servidor:
    -   postgre
    -   hsotname/addres : postgres
    -   puerto : 5432
    -   DB: postgres
    -   user: devuser
    -   password: devpass

```

crea una base nueva puede ser prueba o cualquier nombre:

en bash coloquese donde esta el backup y ejecute el siguiente comando :

```bash
docker cp BK2.sql dev-postgres:/BK2.sql
```
se copiara el backup dondreo del contenedor y ejecutaremos el siguiente comando:

```bash
docker exec -it dev-postgres pg_restore -U devuser -d prueba --clean --if-exists --no-owner --no-privileges --verbose /BK2.sql
```

ese ejecutara un arestaure porque el backup fue echo binario y espalno por eso no nos genero el bakup completo 

despues de todo esto se debera ejecutar lo siguiente :

```bash
docker network connect prueba2_laravel dev-postgres
```

esto servira para hacer conexion entre contenedore y le establecera una red network

de ahi se debe de ingresar al proyecto con 

```bash
docker exec -it laravel_app bash
```

luego ejecutaremos 

```bash
apt update
apt install nano 
nano .env 
``` 
busquemos el bloque de la conexion de base de datos y ponemos esto asi establecemos conexion 

```env
DB_CONNECTION=pgsql
DB_HOST=dev-postgres
DB_PORT=5432
DB_DATABASE=prueba
DB_USERNAME=devuser
DB_PASSWORD=devpass
```

## SSH para clonar repos privados

- El contenedor monta tu llave SSH desde Windows: `${USERPROFILE}\.ssh -> /root/.ssh`.
- Verifica que tu llave pública esté autorizada en GitHub/GitLab.
- En Windows, la llave montada puede tener permisos amplios (`0777`), por eso se copia a `/tmp/ssh` y se aplica `chmod 600` antes del clone.

## Variables de entorno

- Dentro del contenedor, ajusta `/var/www/.env` con:

```env
DB_CONNECTION=pgsql
DB_HOST=dev-postgres
DB_PORT=5432
DB_DATABASE=prueba
DB_USERNAME=devuser
DB_PASSWORD=devpass
```


Ejemplo para editar el `.env` dentro del contenedor:

```bash
docker-compose exec app bash -lc "cp -n /var/www/.env.example /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_CONNECTION=.*/DB_CONNECTION=pgsql/' /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_HOST=.*/DB_HOST=dev-postgres/' /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_PORT=.*/DB_PORT=5432/' /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_DATABASE=.*/DB_DATABASE=prueba/' /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_USERNAME=.*/DB_USERNAME=devuser/' /var/www/.env"
docker-compose exec app bash -lc "sed -i 's/^DB_PASSWORD=.*/DB_PASSWORD=devpass/' /var/www/.env"
```

## Comandos útiles

```bash
docker compose exec app php -v
docker compose exec app php -m
docker compose exec app composer install --no-dev --optimize-autoloader
docker compose logs -f web
docker compose logs -f app
```
