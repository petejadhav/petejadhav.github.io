Develop & Deploy Fastapi + React with Postgres using Docker-Compose
26-05-2021


[![unsplash.com](https://petejadhav.in/static/8784cb962b0593b5efb51100b12ec2ed/4b190/christopher-gower-m_HRfLhgABo-unsplash.jpg)](/static/8784cb962b0593b5efb51100b12ec2ed/4409e/christopher-gower-m_HRfLhgABo-unsplash.jpg)

I finally sat down to create a fully production ready API with a frontend. On the deploy side, [Docker](https://www.docker.com/) seems to be an awesome tool I should have learnt by now. With no dearth of time on my hands (no thanks to Covid), I completed the code over the weekend.

The Tech Stack
--------------

I decided to use something fancy this time. The API backend has been written in Python using [fastapi](https://fastapi.tiangolo.com/). The frontend is a very simple [single page application](https://en.wikipedia.org/wiki/Single-page_application) written in [React JS](https://reactjs.org/). The user data is stored on a [postgres](https://www.postgresql.org/) database. The deployment will be handled by [Docker-Compose](https://docs.docker.com/compose/). The code is on Github [here](https://github.com/petejadhav/ml_fastapi_react).

### API Server Backend

The API server will be created in Python using fastapi. The API should be able to handle the following functions -

*   Login & User Registration
*   [JSON-Web-Token](https://auth0.com/docs/tokens/json-web-tokens) based authentication. Code is [here](https://github.com/petejadhav/ml_fastapi_react/blob/main/backend/jwt_auth_handler.py)
*   Writing & modifying user data on the database. Code is [here](https://github.com/petejadhav/ml_fastapi_react/blob/main/backend/user_handler.py)
*   API [endpoints](https://github.com/petejadhav/ml_fastapi_react/blob/main/backend/main.py) for general use (like exposing a Machine Learning model)

The [main](https://github.com/petejadhav/ml_fastapi_react/blob/main/backend/main.py) file has the code that defines the endpoints. The user handler file will handle the database calls. The user password will be stored as a hash. This is handled by the [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) algorithm. The **/login** endpoint will expect a username & password and return a JWT token.

The code will be deployed on a docker continer using the **_[uvicorn-gunicorn-fastapi](https://hub.docker.com/r/tiangolo/uvicorn-gunicorn-fastapi/dockerfile)_** image.

### React JS Frontend

The single-page-application frontend will be developed in Javascript using React JS. The frontend will have the following components -

*   Routing using [React Router](https://reactrouter.com/web/guides/quick-start)
*   Login component
*   Register component
*   Example API endpoint handler component
*   State management using [Recoil](https://recoiljs.org/). (Recoil is kinda cool. Check it out.)

The login, register & endpoint handler component will call the API we built earlier. The login & register component will POST request the API which will return the JSON Web token to make authenticated calls to the other endpoints. The app [state](https://github.com/petejadhav/ml_fastapi_react/blob/main/frontend/src/state.js) is just an authentication flag & user data for now. The example handler will check the auth flag before calling our API.

The frontend code will also be deployed on a docker container. We’ll use the basic [Nginx](https://www.nginx.com/) image. The react code will sit behind an nginx server. The nginx server will also handle the API requests at **/api** and redirect them to the API server. The nginx conf file is [here](https://github.com/petejadhav/ml_fastapi_react/blob/main/frontend/nginx.conf)

### Postgres Database

The user data will be stored on a Postgres database. For now, we will just store the user’s email and password. The easiest way to get postgres on your machine is to use a Docker image.

```
FROM postgres 
ENV POSTGRES_PASSWORD postgres 
ENV POSTGRES_DB testdb 
COPY init.sql /docker-entrypoint-initdb.d/
```


This will get the original postgres image from [docker-hub](https://hub.docker.com/). Create 2 environment variables for database name and password. The last line will copy a SQL script that will be executed on startup. Our startup script will just create an empty User table with the required schema.

```
CREATE TABLE public.users (
    id SERIAL PRIMARY KEY,
    email varchar(255),
    password varchar(255)
);
```


Run the docker container with the following commands in the directory that contains dockerfile.

```
docker build -t pgdb .
docker run -d --name pgdb_container -p 5432:5432 pgdb
```


### Deploy using Docker Compose

[Docker-Compose](https://docs.docker.com/compose/) is an awesome tool that can handle multiple container deployments on the same host. We are creating 3 different containers for the 3 services (api, web, db) which will run on the same host machine. All we need is to create the docker-compose.yaml file.

```
version: "3.7"

services:
    db:
        build: postgres_db
        healthcheck:
          test: [ "CMD", "pg_isready", "-q", "-d", "${POSTGRES_DB}", "-U", "postgres" ]
          timeout: 45s
          interval: 10s
          retries: 10
    
    web:
        build: frontend
        ports:
          - 80:80
        depends_on:
          - api

    api:
        build: backend
        environment:
          - PORT=80
          - DB_HOST=db
        depends_on:
          db:
            condition: service_healthy
```


We define the **_healthcheck_** because we need the postgres databse to be up & running before the API server starts.

Running this will start all 3 services -

```
docker-compose up --build
```


That’s all folks. You now have a production-ready API server with a simple frontend and a user database. Don’t forget to change the JWT secret in the backend [config](https://github.com/petejadhav/ml_fastapi_react/blob/main/backend/config.py) file.