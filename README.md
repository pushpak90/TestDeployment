# RentVideo

**Simple RESTful Video Rental Service** — Spring Boot + MySQL

---

## Project Overview

RentVideo is a simplified video rental REST API built with Spring Boot. It demonstrates typical backend functionality including authentication and authorization (JWT), role-based access control (`CUSTOMER`, `ADMIN`), secure user registration and login with BCrypt password hashing, video management, and a controlled rental lifecycle (rent / return).

Data is persisted in MySQL using Spring Data JPA and boilerplate code is minimized using Lombok.  
This project is configured with Gradle (`gradlew`) and targets **Java 21**.

**Deployed Application:** [https://testdeployment-cgoa.onrender.com](https://testdeployment-cgoa.onrender.com/)  
**Hosting:** Render (backend) + Aiven (MySQL database)

---

## Key Features

- JWT-based stateless authentication and role-based authorization  
- User registration and login with secure password hashing  
- Two access roles: `CUSTOMER` (default) and `ADMIN`  
- Public endpoints for registration and login  
- Private APIs for managing videos and rentals  
- `ADMIN`-only endpoints for creating, updating, or deleting videos  
- Limit of **2 active rentals per user** at a time  

---

## Tech Stack / Dependencies

- Java 21  
- Spring Boot (Web, Security, Data JPA, Validation)  
- MySQL  
- Lombok  
- JWT (via Spring Security)  
- Gradle (wrapper included)

### Example dependencies:
```
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-validation'
implementation 'mysql:mysql-connector-java'
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

---

## Project Structure

```
rentvideo/
├── src/main/java/com/crio/rentvideo
│   ├── Controller
│   ├── Dto
│   ├── Entity
│   ├── Repository
│   ├── Security
│   └── Service
├── src/main/resources
│   ├── application.properties
│   └── templates
└── build.gradle
```

---

## Database Schema

### User
- `id` (PK)  
- `email` (unique)  
- `password` (BCrypt hashed)  
- `first_name`  
- `last_name`  
- `role` (enum: `CUSTOMER`, `ADMIN`)  

### Video
- `id` (PK)  
- `title`  
- `director`  
- `genre`  
- `available` (boolean)  

### Rental
- `id` (PK)  
- `user_id` (FK -> User)  
- `video_id` (FK -> Video)  
- `rented_at` (timestamp)  
- `returned_at` (nullable timestamp)  
- `status` (enum: `ACTIVE`, `RETURNED`)  

#### Business Rules
- On rent:
  - `Rental` created with `status=ACTIVE`
  - `Video.available=false`
- On return:
  - `status=RETURNED`
  - `Video.available=true`
- Each user can have a **maximum of 2 ACTIVE rentals** at once.

---

## Configuration

```
spring.datasource.url=jdbc:mysql://localhost:3306/rentvideo?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_mysql_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# JWT configuration
app.jwt.secret=ReplaceThisWithASecretKey
app.jwt.expiration-ms=86400000

server.port=8080
```

**Note:**  
In deployment, the app connects to a **MySQL database hosted on Aiven**, with credentials securely managed through Render environment variables.

---

## Getting Started Locally

### Prerequisites
- Java 21  
- MySQL installed and running (with a `rentvideo` database)  
- Gradle wrapper available (`./gradlew` or `gradlew.bat`)  

### Steps
1. Create the database:
   ```
   CREATE DATABASE rentvideo;
   ```
2. Update your MySQL credentials and JWT secret in `src/main/resources/application.properties`.
3. Run the application:
   - macOS/Linux: `./gradlew bootRun`
   - Windows: `gradlew.bat bootRun`
4. Visit [http://localhost:8080/](http://localhost:8080/) to access the API.

---

## API Endpoints

### Public

**Register User**  
`POST /api/auth/register`  
Example body:
```
{
  "email": "a@b.com",
  "password": "pass123",
  "firstName": "John",
  "lastName": "Doe",
  "role": "CUSTOMER"
}
```

**Login**  
`POST /api/auth/login` — login and receive JWT token  

---

### Videos

**Public Access**  
- `GET /api/videos` — list all videos  
- `GET /api/videos/{id}` — get details of a video  

**Admin Only**  
- `POST /api/videos` — create video  
- `PUT /api/videos/{id}` — update video  
- `DELETE /api/videos/{id}` — delete video  

---

### Rentals (Authenticated Users)

- `POST /api/rentals/rent` — rent a video  
  Example:  
  ```
  { "videoId": 10 }
  ```

- `POST /api/rentals/return` — return a video  
  Example:  
  ```
  { "videoId": 10 }
  ```

- `GET /api/rentals` — list current user’s rentals  

---

## Security Design

- **Authentication:** JWT tokens generated at login; required in `Authorization: Bearer <token>` header for protected routes.  
- **Authorization:** Role-based access enforced with `@PreAuthorize` and Spring Security configuration.  
- **Session Management:** Stateless (`SessionCreationPolicy.STATELESS`).  
- **Password Security:** All passwords encrypted via `BCryptPasswordEncoder`.  
- **Key Annotations:** `@EnableWebSecurity`, `@EnableMethodSecurity`.  

### Roles and Permissions
- `CUSTOMER`: view, rent, and return videos.  
- `ADMIN`: full CRUD control over videos as well as all customer capabilities.

---

## Deployment Details

- **Hosting Platform:** Render (Spring Boot backend)  
- **Database Provider:** Aiven (managed MySQL service)  
- **Public API URL:** [https://testdeployment-cgoa.onrender.com](https://testdeployment-cgoa.onrender.com/)  
- **Environment Configuration:**
  - All secrets (JWT key, DB credentials) stored as secure environment variables in Render.
  - Continuous deployment enabled to auto-deploy on pushes to the main branch.

---

```

Would you like me to append a brief “Example Usage with curl” section at the end for quick testing commands?
