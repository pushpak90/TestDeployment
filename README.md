RentVideo
Simple RESTful Video Rental Service — Spring Boot + MySQL

Project Overview
RentVideo is a simplified video rental REST API built with Spring Boot. It demonstrates typical backend functionality including authentication and authorization (JWT), role-based access control (CUSTOMER, ADMIN), secure user registration and login with BCrypt password hashing, video management, and a controlled rental lifecycle (rent / return).
Data is persisted in MySQL using Spring Data JPA and boilerplate code is minimized using Lombok.

This project is configured with Gradle (gradlew) and targets Java 21.

Deployed Application: https://testdeployment-cgoa.onrender.com
Hosting: Render (backend) + Aiven (MySQL database)

Key Features
JWT-based stateless authentication and role-based authorization

User registration and login with secure password hashing

Two access roles: CUSTOMER (default) and ADMIN

Public endpoints for registration and login

Private APIs for managing videos and rentals

ADMIN-only endpoints for creating, updating, or deleting videos

Limit of 2 active rentals per user at a time

Tech Stack / Dependencies
Java 21

Spring Boot (Web, Security, Data JPA, Validation)

MySQL

Lombok

JWT (via Spring Security)

Gradle (wrapper included)

Example dependencies:

text
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-validation'
implementation 'mysql:mysql-connector-java'
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
Project Structure
text
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
Database Schema
User
id (PK)

email (unique)

password (BCrypt hashed)

first_name

last_name

role (enum: CUSTOMER, ADMIN)

Video
id (PK)

title

director

genre

available (boolean)

Rental
id (PK)

user_id (FK -> User)

video_id (FK -> Video)

rented_at (timestamp)

returned_at (nullable timestamp)

status (enum: ACTIVE, RETURNED)

Business Rules:

On rent:

Rental created with status=ACTIVE

Video.available=false

On return:

status=RETURNED

Video.available=true

Each user can have maximum 2 ACTIVE rentals at once.

Configuration
text
spring.datasource.url=jdbc:mysql://localhost:3306/rentvideo?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_mysql_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# JWT configuration
app.jwt.secret=ReplaceThisWithASecretKey
app.jwt.expiration-ms=86400000

server.port=8080
Note:
In deployment, the app connects to a MySQL database hosted on Aiven with credentials managed through Render environment variables.

Getting Started Locally
Prerequisites
Java 21

MySQL installed and running (with a rentvideo database)

Gradle wrapper available (./gradlew or gradlew.bat)

Steps
Create the database:

sql
CREATE DATABASE rentvideo;
Update your MySQL credentials and JWT secret in src/main/resources/application.properties.

Run the application:

macOS/Linux: ./gradlew bootRun

Windows: gradlew.bat bootRun

Visit http://localhost:8080/ to access the API.

API Endpoints
Public
POST /api/auth/register — register new user
Example body:

json
{ "email": "a@b.com", "password": "pass123", "firstName": "John", "lastName": "Doe", "role": "CUSTOMER" }
POST /api/auth/login — login and receive JWT

Video APIs
GET /api/videos — list all videos

GET /api/videos/{id} — get video details

Admin Only

POST /api/videos — create video

PUT /api/videos/{id} — update video

DELETE /api/videos/{id} — delete video

Rentals (authenticated users)
POST /api/rentals/rent — rent a video
Example: { "videoId": 10 }

POST /api/rentals/return — return a video
Example: { "videoId": 10 }

GET /api/rentals — view user’s current and past rentals

Security Design
Authentication: JWT tokens issued at login; included in Authorization: Bearer <token> header.

Authorization: Role-based restrictions using @PreAuthorize and Spring Security.

Session Management: Stateless (SessionCreationPolicy.STATELESS).

Password Security: Uses BCryptPasswordEncoder for all stored passwords.

Annotations: @EnableWebSecurity, @EnableMethodSecurity.

Roles and Permissions

CUSTOMER: can view and rent/return videos.

ADMIN: can do all customer actions plus manage video inventory.

Deployment Details
Hosting Platform: Render (Spring Boot app)

Database Provider: Aiven (managed MySQL service)

Public API URL: https://testdeployment-cgoa.onrender.com

Environment Configuration:

All secrets (JWT key, DB credentials) stored in secure environment variables on Render.

Continuous deployment set up to auto-deploy on push to the main branch.
