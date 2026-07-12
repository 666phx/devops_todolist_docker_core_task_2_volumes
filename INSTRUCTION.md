# Instructions

## Docker Hub repositories

- MySQL image: https://hub.docker.com/r/666phx/mysql-local
- App image: https://hub.docker.com/r/666phx/todoapp

## 1. Run the MySQL container with a volume attached

Build the MySQL image (based on the official `mysql` image), or pull it from Docker Hub.

```bash
# Build locally
docker build -f Dockerfile.mysql -t mysql-local:1.0.0 .

# Or pull the pushed image
docker pull 666phx/mysql-local:1.0.0
```

Create a named volume so the database data persists across container restarts/removals:

```bash
docker volume create mysql_data
```

Run the container, attaching the volume to MySQL's data directory (`/var/lib/mysql`) and publishing port 3306:

```bash
docker run -d \
  --name mysql-local \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql-local:1.0.0
```

The image's `ENV` variables (`MYSQL_DATABASE=app_db`, `MYSQL_USER=app_user`, `MYSQL_PASSWORD=1234`) automatically create the `app_db` database and the `app_user` user with password `1234` on first start.

Because the data lives in the `mysql_data` volume, you can remove and recreate the container without losing data:

```bash
docker rm -f mysql-local
docker run -d --name mysql-local -v mysql_data:/var/lib/mysql -p 3306:3306 mysql-local:1.0.0
```

## 2. Run the App container connected to the MySQL container

Find the IP address of the running MySQL container:

```bash
docker inspect mysql-local --format '{{json .NetworkSettings.Networks}}'
```

Update `todolist/settings.py` -> `DATABASES['default']['HOST']` with that IP address (already set to `172.17.0.2` for this environment; update it if your container gets a different IP).

Build the app image:

```bash
docker build -t todoapp:2.0.0 .
```

(The build runs `python manage.py migrate` at build time, so the MySQL container must already be running and reachable at the configured `HOST` before building.)

Run the app container:

```bash
docker run -d \
  --name todoapp \
  -p 8080:8080 \
  todoapp:2.0.0
```

Or pull the pushed image instead of building locally:

```bash
docker pull 666phx/todoapp:2.0.0
docker run -d --name todoapp -p 8080:8080 666phx/todoapp:2.0.0
```

## 3. Access the application

Open your browser at:

- Landing page: http://localhost:8080/
- API: http://localhost:8080/api/

You should see the Django-Todolist landing page and API root, both served with data stored in the MySQL container.
