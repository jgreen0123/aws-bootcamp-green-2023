# Week 1 — App Containerization


### VSCode Docker Extenstion <br>
Verfied the Docker extension was installed in my VSCode: <br>


### Containerize the Backend <br>
Created the docker file for the back-end in the backend-flask directory and added comments for each step:
```FROM python:3.10-slim-buster

# Inside Container
# Make a new folder inside the container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# Contains the libraries we want to install to run the app
COPY requirements.txt requirements.txt

# Inside Container
# Installs the python libraries used for the app
RUN pip3 install -r requirements.txt

# Outside Container -> Inside Container
# Copies everything in the backend-flask directory to inside the container
COPY . .

# Set Enviroment variables inside the container
ENV FLASK_ENV=development

EXPOSE ${PORT}

# Command to run flask
# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

Built and ran the container using:
```
docker build -t backend-flask ./backend-flask
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

Retreive the container id and assign it as an environment cariable: <br>
```CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)```

Tested accessing the container using the docker exec command and by right clicking on the container itself within the Docker extension: <br>
```docker exec CONTAINER_ID -it sh```

### Containerize the Frontend <br>
Ran NPM install from the frontend-react-js directory:
```npm i```

Created the docker file for the front-end in the frontend-flask directory:
``` FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
Built and ran the front end container using:
``` 
docker build -t frontend-react-js ./frontend-react-js 
docker run -p 3000:3000 -d frontend-react-js
```

### Created a Docker Compose file to build and run multiple containers
```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

### Added Install for Postgres into the Gitpod build:
```
- name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### Added DynamoDB and Postgres services to the docker compose file:
DynamoDB Services: <br>
```
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```
PostGres Services: <br>
```
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

