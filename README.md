# Running a Fullstack App Using Docker Compose

This guide provides step-by-step instructions on how to run a full-stack React and Node.js application using Docker Compose. The app consists of a React frontend, a Node.js backend, and a MySQL database.

Docker Compose simplifies the process of setting up multi-container applications by defining services, networks, and volumes in a single YAML file.

![Architecture](https://github.com/poridhiEng/poridhi-labs/blob/main/Poridhi%20Labs/Full-stack-app/Deploy%20App%20Using%20Docker%20Compose/images/image.png?raw=true)

## Introduction

We will run three services:

- **Frontend**: React app served via Node
- **Backend**: Node.js REST API
- **Database**: MySQL

### Folder Structure

```
/my-fullstack-app
├── /frontend
├── /backend
└── docker-compose.yml
```

## Step 1: Setup React Frontend

- Create app:
  ```bash
  npm create-react-app frontend .
  ```

- Define `frontend/.env`:
  ```
  REACT_APP_API_BASE_URL=http://localhost:5000
  ```

- Dockerfile for frontend:
  ```dockerfile
  FROM node:16-alpine
  WORKDIR /app
  COPY package*.json ./
  RUN npm install
  COPY . .
  COPY .env .env
  EXPOSE 3000
  CMD ["npm", "start"]
  ```

## Step 2: Setup Node.js Backend

- Init Node project:
  ```bash
  mkdir backend && cd backend
  npm init -y
  ```

- Install packages:
  ```bash
  npm install express sequelize mysql2 dotenv body-parser cors nodemon
  ```

- `backend/.env`:
  ```
  DB_USERNAME=myuser
  DB_PASSWORD=mypassword
  DB_NAME=my_db
  DB_HOST=db
  PORT=5000
  ```

- Dockerfile for backend:
  ```dockerfile
  FROM node:14
  WORKDIR /app
  COPY package*.json ./
  RUN npm install
  COPY . .
  COPY .env .env
  EXPOSE 5000
  CMD ["npm", "start"]
  ```

## Step 3: docker-compose.yml

```yaml
version: '3.8'

services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: my_db
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypassword
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 3s
      retries: 2

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      DB_USERNAME: myuser
      DB_PASSWORD: mypassword
      DB_NAME: my_db
      DB_HOST: db
      PORT: 5000
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./backend:/app
      - /app/node_modules
    command: npm start

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      backend:
        condition: service_started
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      REACT_APP_BACKEND_URL: http://localhost:5000
    command: npm start

volumes:
  db_data:
```

## Step 4: Run the App

```bash
cd my-fullstack-app
docker-compose up --build
```

Access:
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000/users

## Conclusion

Docker Compose makes it easy to run full-stack apps with multiple services. This guide walks you through the process using React, Node.js, and MySQL.

