# Docker Compose for a Full-Stack Application with Flask, Next.js, and PostgreSQL

This repository demonstrates how to set up a Flask, Next.js, and PostgreSQL database server inside docker containers and connect them all together.

TL;DR

To get this project up and running, follow these steps

1. Make sure you have Docker installed in your system. For installation steps, follow the 
2. Clone the repository into your device
3. Open a terminal from the cloned project's directory (Where the docker-compose.yml file is present)
4. Run the below docker commands: 
```
docker compose build (build all the containers)
docker compose up -d (run all the containers)
docker compose down (bring down all containers)
docker volume rm volume_name (to remove the docker volume created by compose)
```

That's all! That should get the project up and running. To see the output, you can access http://127.0.0.1:4000 from the browser and you should find a web page with a list of users. This entire system with the client, server & database are running inside of docker and being accessible from your machine.

Here is a detailed explanation on what is going on.

1. **_services_** Section:

This defines the containers that will be running. Each service represents a different container with its configuration.

- **_Service: db_** (PostgreSQL Database)

```
db:
  container_name: database
  image: postgres:13
  environment:
    POSTGRES_USER: ${POSTGRES_USER}
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    POSTGRES_DB: ${POSTGRES_DB}
  ports:
    - 5432:5432
  volumes:
    - pgdata:/var/lib/postgresql/data
```

- container_name: database: The container will be named database.
- image: postgres:13: It uses the official PostgreSQL 13 image.
- environment:
    - POSTGRES_USER, POSTGRES_PASSWORD, and POSTGRES_DB: Sets up PostgreSQL with a user postgres,       password postgres, and a database named postgres.
- ports: - 5432:5432: Exposes the PostgreSQL service on port 5432 of the host machine, mapping it   to the container's 5432 port.
- volumes: - pgdata:/var/lib/postgresql/data: Persists PostgreSQL data in a Docker-managed volume (pgdata), so database data remains intact even if the container is restarted.

**_Service: flaskapp_**  (Flask Backend Application)

```
flaskapp:
  container_name: flaskapp
  image: flaskapp:1.0.0
  build:
    context: ./backend
    dockerfile: flask.dockerfile
  ports:
    - 4000:4000
  restart: always
  environment:
    - DATABASE_URL=${DATABASE_URL}
  depends_on:
    - db
```

- container_name: flaskapp: The container will be named flaskapp.
- image: flaskapp:1.0.0: Uses the image flaskapp:1.0.0. If not available, the image is built from the specified Dockerfile.
- build:
    - context: ./backend: Specifies the ./backend folder as the build context, i.e., where the Dockerfile and app source code are located.
    - dockerfile: flask.dockerfile: Uses a custom Dockerfile named flask.dockerfile for building the flaskapp service.
- ports: - 4000:4000: Exposes the Flask app on port 4000 of the host machine and the container.
- restart: always: Ensures the container will always restart if it stops or crashes.
- environment:
    - DATABASE_URL: Specifies the database connection string for the Flask app, which connects to the db service (PostgreSQL) using the internal Docker network.
    - The URL postgresql://postgres:postgres@database:5432/postgres indicates:
    postgres as the user and password.
    database is the container name of the db service, which Docker uses to resolve the internal network address.
    5432 is the port used by PostgreSQL.
    postgres is the name of the database.
- depends_on: - db: Ensures that the db service starts before the flaskapp service.

**_Service: nextapp_** (Next.js Frontend Application)

```
  nextapp:
    container_name: nextapp
    image: nextapp:1.0.0
    build: 
      context: ./frontend
      dockerfile: next.dockerfile
    ports:
      - 3000:3000
    environment:
      - NEXT_PUBLIC_API_URL=${FLASK_APP_URL}
    depends_on:
      - flaskapp
```

- container_name: nextapp: The container will be named nextapp.
- image: nextapp:1.0.0: Uses the image nextapp:1.0.0. If not available, the image is built from the specified Dockerfile.
- build:
    - context: ./frontend: Specifies the ./frontend folder as the build context, i.e., where the Dockerfile and app source code are located.
    - dockerfile: next.dockerfile: Uses a custom Dockerfile named next.dockerfile for building the nextapp service.
- ports: - 3000:3000: Exposes the Next.js app on port 3000 of the host machine and the container.
- environment:
- NEXT_PUBLIC_API_URL: Specifies the URL for the Flask API. Since the flaskapp is exposed on port 4000 on the host machine, it can be accessed via http://localhost:4000.
- depends_on: - flaskapp: Ensures that the flaskapp service starts before the nextapp service.

 2. **_volumes_** Section:

 ```
 volumes:
  pgdata: {}
```
- pgdata: {}: Defines a Docker-managed volume named pgdata that is used by the db service to store PostgreSQL data. This ensures that the data persists even if the PostgreSQL container is removed or restarted.

NOTE: Where to find this Docker-managed volume in a host machine?

Docker stores volumes in a specific directory on your system:

- On Linux and macOS: Docker stores volumes in ```/var/lib/docker/volumes/```.
Your volume pgdata would be located at ```/var/lib/docker/volumes/pgdata/_data```.
- you can also run ```docker inspect pgdata``` to see more about specific volume.


This setup creates a full-stack application where the frontend (nextapp) interacts with the backend (flaskapp), and the backend communicates with the PostgreSQL database (db). All containers are configured to use the default Docker network for inter-container communication

Below is how the application will look like:-
