# Dockerized Attendance Portal Overview

## 1. Project Summary
This repository contains a full Dockerized attendance management portal with three main services:
- `mysql`: MySQL database for persistent storage
- `backend`: Spring Boot REST API server
- `employee-frontend`: Employee-facing React frontend
- `admin-frontend`: HR/Admin-facing React frontend

The project is orchestrated using `docker-compose.yml` from the `docker/sky-connect-build` folder.

## 2. Services and Purpose

### 2.1 MySQL Database (`mysql`)
- Image: `mysql:8.0`
- Container name: `skyconnect-mysql`
- Port: host `3307` → container `3306`
- Volume: `mysql_data` for persistent DB files
- Schema initialization:
  - `skyconnect_schema.sql`
  - `update_payslip_schema.sql`
- Healthcheck: `mysqladmin ping` using root credentials
- Profile: `local-db`

### 2.2 Backend API (`backend`)
- Build context: `./main-backend`
- Container name: `skyconnect-backend`
- Port: host `8086` → container `8086`
- Runtime: Java 17, Spring Boot
- Environment variables are injected from the host environment or `.env`
- Connects to MySQL via `SPRING_DATASOURCE_URL`

### 2.3 Employee Frontend (`employee-frontend`)
- Build context: `./employee-frontend`
- Container name: `skyconnect-employee`
- Port: host `5174` → container `80`
- Built with Vite + React
- Served via Nginx
- Uses backend API at `http://<host>:8086` by default

### 2.4 Admin Frontend (`admin-frontend`)
- Build context: `./admin-frontend`
- Container name: `skyconnect-admin`
- Port: host `5173` → container `80`
- Built with Vite + React
- Served via Nginx
- Uses backend API at `http://<host>:8086` by default

## 3. Key Features

### 3.1 Admin / HR Frontend Features
The admin portal includes the following functional pages:
- Login
- Dashboard
- Employees management
- Add Employee
- Attendance tracking
- Monthly attendance view
- Leave management
- HR notices
- Payslip generation or visibility
- Settings
- Authenticated admin layout and protected routes

### 3.2 Employee Frontend Features
The employee portal includes:
- Login and Register
- Dashboard
- Attendance view
- Holiday calendar and holiday list
- Leave application
- Payslip view
- Tasks
- Company details
- Employee details and profile information
- Protected routes for authenticated users

### 3.3 Backend Features
The Spring Boot backend provides:
- REST API endpoints for authentication and Google-based auth
- Leave request endpoints
- Payslip endpoints
- Reminder features
- Database persistence using Spring Data JPA and MySQL
- Security and API key validation
- CORS configuration for frontend origins
- AWS SES configuration placeholders for email functionality

## 4. Docker Architecture and Flow

### 4.1 Startup Process
1. `docker-compose --profile local-db up -d`
2. Docker creates the `sky-connect-build_default` network
3. MySQL starts and initializes the schema scripts
4. Backend builds and starts, then connects to MySQL
5. Both frontends build and serve the static React apps with Nginx

### 4.2 Build Flow
- Frontends: multi-stage Dockerfiles use Node 20 Alpine to build, then Nginx to serve `dist`
- Backend: multi-stage Dockerfile uses Maven to build the Spring Boot jar, then runs it on Java 17 JRE

## 5. Environment and Configuration

### 5.1 `.env` Values
The `.env` file contains defaults for:
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- `DB_USE_SSL`, `DB_ALLOW_PUBLIC_KEY_RETRIEVAL`, `DB_TIMEZONE`
- `APP_API_KEY`
- `APP_CORS_ALLOWED_ORIGINS`
- `APP_AUTH_ALLOWED_DOMAINS`
- `APP_GOOGLE_CLIENT_ID`
- AWS SES and keys
- `BACKEND_HOST_PORT`

### 5.2 Important backend variables
- `SPRING_DATASOURCE_URL`: MySQL connection string
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`
- `APP_API_KEY`: required by frontends to call backend APIs
- `APP_CORS_ALLOWED_ORIGINS`: allows `http://localhost:5173` and `http://localhost:5174`

## 6. Working Criteria and Validation

### 6.1 Service readiness criteria
- MySQL must be healthy and accept connections
- Backend must start on port `8086`
- Admin portal must serve at `http://localhost:5173`
- Employee portal must serve at `http://localhost:5174`

### 6.2 How to verify each service
- Check Docker containers: `docker ps`
- Check backend: `curl http://localhost:8086/actuator/health` or API endpoints
- Check admin UI: open `http://localhost:5173`
- Check employee UI: open `http://localhost:5174`

### 6.3 Expected functional verification
- Admin login should reach the admin portal and show dashboard pages
- Employee login or registration should reach the employee portal
- Attendance and leave pages should render and connect to backend APIs
- HR notices and payslip pages should be reachable from admin side
- The backend should be able to use the MySQL database initialized from SQL scripts

## 7. Ports and URLs
- Admin frontend: `http://localhost:5173`
- Employee frontend: `http://localhost:5174`
- Backend API: `http://localhost:8086`
- MySQL: `localhost:3307`

## 8. Useful Commands
- Start all services: `docker-compose --profile local-db up -d`
- Stop all services: `docker-compose --profile local-db down`
- View logs for a service: `docker logs -f skyconnect-backend`
- Rebuild just one service: `docker-compose build backend`

## 9. Notes and Caveats
- The backend uses `spring.jpa.hibernate.ddl-auto=none`, so database schema is not auto-created by Hibernate. The SQL scripts are the primary schema source.
- The frontends rely on `VITE_API_BASE` or the browser host for backend URL resolution.
- The MySQL service is configured to initialize data only on first start using mounted SQL scripts.
- The admin and employee apps share the same backend port, so the backend must be running before frontend actions requiring API calls.

---

This document is intended to explain the full Dockerized application, the services it runs, the key features available in each UI, and the validation/working criteria for the setup.
