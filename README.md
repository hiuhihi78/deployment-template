# 🚀 Fullstack Docker + Nginx Environment Template

This template helps you quickly set up a **Fullstack environment (Frontend + Backend + Nginx)** using Docker.  
Suitable for bootstrapping new projects, architecture demos, or basic deployments.

---

## 📁 Project Structure

```text
.
├── backend/
│   └── Dockerfile          # Backend image build 
│
├── frontend/
│   ├── Dockerfile          # Frontend build 
│   └── nginx.conf          # Serve static files
│
├── nginx/
│   ├── Dockerfile          # Nginx reverse proxy build
│   └── nginx.conf          # Route FE + BE
│
├── .env                    # Environment variables
├── docker-compose.yml      # System orchestration
```

---

## 🧩 Architecture Overview

```text
Client → Nginx (port 80)
           ↓
   ┌───────────────┐
   │               │
Frontend       Backend
:3000          :5000
```

- Nginx acts as a **reverse proxy**
- Frontend serves static files
- Backend handles APIs and realtime processing

---

## ⚙️ How to Run the Project

### 1. Clone the template

```bash
git clone <template-repo>
cd <template-repo>
```

---

### 2. Add source code

```bash
# Frontend
git clone <frontend-repo> frontend

# Backend
git clone <backend-repo> backend
```

---

### 3. Customize Dockerfiles

Each service requires a Dockerfile based on your framework and build process.

#### Frontend - React

```bash
npm install
npm run build
```

```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 3000
```

---

#### Frontend - Angular

```bash
npm install
ng build --prod
```

```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN ng build --prod

FROM nginx:alpine
COPY --from=build /app/dist/your-app /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 3000
```

---

#### Backend - .NET

```bash
dotnet restore
dotnet publish -c Release -o out
dotnet out/YourApp.dll
```

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /app
COPY *.sln ./
COPY src ./src
RUN dotnet restore
RUN dotnet publish ./src/YourApp.csproj -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 5000
ENTRYPOINT ["dotnet", "YourApp.dll"]
```

---

#### Backend - Java (Maven)

```bash
mvn clean package
java -jar target/app.jar
```

```dockerfile
FROM maven:3.9-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar .
EXPOSE 5000
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 4. Build & Run

```bash
docker-compose up -d --build
```

---

### 5. Access

- Frontend: http://localhost  
- Backend API: http://localhost/api  

---

# 🧪 Sample Build (Current Setup)

---

## 🔹 Backend Service (.NET 7)

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /app

COPY MeChat.sln ./
COPY src ./src
COPY test ./test

RUN dotnet restore MeChat.sln

RUN dotnet publish ./src/MeChat.API/MeChat.API.csproj \
    -c Release \
    -o /app/publish \
    --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app

COPY --from=build /app/publish .

ENV ASPNETCORE_URLS=http://+:5000
ENV ASPNETCORE_ENVIRONMENT=Production

EXPOSE 5000

ENTRYPOINT ["dotnet", "MeChat.API.dll"]
```

---

## 🔹 Frontend Service (React + Nginx)

```dockerfile
FROM node:18 AS build
WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 3000
```

---

## 🔹 Nginx Reverse Proxy

```nginx
events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://frontend:3000;
        }

        location /api/ {
            proxy_pass http://backend:5000;
        }

        location /realtime/ {
            proxy_pass http://backend:5000;
        }
    }
}
```

---

## 🔹 Environment Variables

```env
APP_NAME=test
ENV=dev
```

---

## 🔹 Docker Compose

```yaml
version: "3.9"

services:
  backend:
    build: ./backend
    container_name: ${APP_NAME}-be
    restart: always

  frontend:
    build: ./frontend
    container_name: ${APP_NAME}-fe
    restart: always

  nginx:
    image: nginx:alpine
    container_name: ${APP_NAME}-nginx-${ENV}
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend
      - frontend
    restart: always
```

---
