# 🏋️ AI Fitness Tracker — Microservices Architecture

A modern, production-oriented **AI-powered fitness tracking application** built with a **Spring Boot Microservices** architecture. The system leverages **Keycloak** for secure authentication & authorization, **Apache Kafka** for event-driven communication, **Eureka** for service discovery, and a centralized **Config Server** for configuration management.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Microservices](#microservices)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Getting Started](#getting-started)
- [API Endpoints](#api-endpoints)
- [Security](#security)
- [Event-Driven Communication](#event-driven-communication)

---

## 🧠 Overview

This project demonstrates enterprise-level backend engineering using microservices design patterns. Each service is independently deployable, loosely coupled, and registers itself with a **Eureka Discovery Server**. All configurations are managed centrally via a **Spring Cloud Config Server**. Services communicate asynchronously through **Kafka**, and **Keycloak** handles all identity and access management.

---

## 🏗️ Architecture

```
                   ┌──────────────────────────────┐
                   │     Config Server (8888)      │
                   │  (Centralized Configuration)  │
                   └──────────────┬───────────────┘
                                  │ config on startup
        ┌─────────────────────────┼────────────────────────┐
        │                         │                        │
┌───────▼──────┐        ┌─────────▼────────┐    ┌─────────▼────────┐
│ Eureka Server│        │   API Gateway    │    │    Keycloak      │
│   (8761)     │◄───────│   (8080)         │    │  (IAM / Auth)    │
│  (Discovery) │        │ JWT Validation + │    └──────────────────┘
└───────▲──────┘        │ Dynamic Routing  │
        │               └────────┬─────────┘
        │ register                │
        │                        │
  ┌─────┴──────────────────────────────────────────────┐
  │                                                    │
┌─▼──────────────┐    ┌─────────────────┐    ┌────────▼────────┐
│  User Service  │    │Activity Service │    │   AI Service    │
│  (MySQL)       │    │  (MongoDB)      │    │  (MongoDB)      │
│                │    │ Kafka Producer  │    │ Kafka Consumer  │
└────────────────┘    └────────┬────────┘    └────────▲────────┘
                               │                      │
                               └──── Kafka Topic ─────┘
                                  (activity.logged)
```

---

## 🧩 Microservices

| Service | Port | Responsibility | Database |
|---|---|---|---|
| **Config Server** | 8888 | Centralized configuration for all services | — |
| **Eureka Server** | 8761 | Service discovery & registration | — |
| **API Gateway** | 8080 | Single entry point, JWT validation, dynamic routing | — |
| **User Service** | 8081 | User registration, profiles, Keycloak integration | MySQL |
| **Activity Service** | 8082 | Log and manage fitness activities, Kafka producer | MongoDB |
| **AI Service** | 8083 | Consumes activity events, generates AI recommendations | MongoDB |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17+ |
| Framework | Spring Boot 3.x + Spring Cloud |
| Auth / IAM | Keycloak (OAuth2 / OpenID Connect) |
| Service Discovery | Netflix Eureka |
| Config Management | Spring Cloud Config Server |
| API Gateway | Spring Cloud Gateway |
| Messaging | Apache Kafka |
| User Database | MySQL |
| Fitness Database | MongoDB |
| Build Tool | Maven |
| Containerization | Docker / Docker Compose |

---

## ✨ Features

- 🔐 **Secure Authentication** — Keycloak-based login, registration, and role-based access control (RBAC)
- 🗂️ **Centralized Config** — All service configurations managed via Spring Cloud Config Server
- 🔍 **Service Discovery** — All services auto-register with Eureka; no hardcoded URLs
- 🌐 **API Gateway** — Single entry point with JWT validation and dynamic routing via Eureka
- 📨 **Event-Driven Design** — Kafka decouples services; activity events trigger AI recommendations asynchronously
- 🤖 **AI Recommendations** — Personalized fitness suggestions per user and per activity
- 🏃 **Activity Tracking** — Log and validate fitness activities stored in MongoDB
- 👤 **User Profiles** — User data securely stored in MySQL with email and ID lookup support

---

## ▶️ Getting Started

### Prerequisites

- Java 17+
- Docker & Docker Compose
- Maven

### 1. Clone the Repository

```bash
git clone https://github.com/anup-08/AI-Fitness-Using-MicroServices.git
cd AI-Fitness-Using-MicroServices/Backend-SpringBoot
```

### 2. Start Infrastructure

```bash
docker-compose up -d
# Starts: Keycloak, Kafka, Zookeeper, MySQL, MongoDB
```

### 3. Configure Keycloak

1. Open Keycloak at `http://localhost:9090`
2. Create a new realm: `fitness-app`
3. Create a client: `fitness-client`
4. Set up roles: `ROLE_USER`, `ROLE_ADMIN`

### 4. Set Environment Variables

```bash
# MySQL (User Service)
MYSQL_URL=jdbc:mysql://localhost:3306/fitnessdb
MYSQL_USER=root
MYSQL_PASSWORD=your_password

# MongoDB (Activity & AI Service)
MONGO_URI=mongodb://localhost:27017/fitnessactivities

# Keycloak
KEYCLOAK_URL=http://localhost:9090
KEYCLOAK_REALM=fitness-app
KEYCLOAK_CLIENT_ID=fitness-client

# Kafka
KAFKA_BOOTSTRAP_SERVERS=localhost:9092

# Eureka
EUREKA_SERVER_URL=http://localhost:8761/eureka
```

### 5. Start Services (in order)

```bash
# 1. Config Server — must start first
cd config-server && mvn spring-boot:run

# 2. Eureka Server
cd eureka-server && mvn spring-boot:run

# 3. Remaining services (any order)
cd api-gateway       && mvn spring-boot:run
cd userService       && mvn spring-boot:run
cd Activity-Service  && mvn spring-boot:run
cd Ai-Service        && mvn spring-boot:run
```

> 💡 Config Server must be running before any other service starts, as they fetch their configuration on startup.

---

## 📡 API Endpoints

All requests go through the **API Gateway** at `http://localhost:8080`.

### 👤 User Service — `/auth`

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/auth/addUser` | Register a new user |
| `GET` | `/auth/getUserByEmail/{email}` | Fetch user details by email |
| `GET` | `/auth/getUserById/{userId}` | Fetch user details by ID |
| `GET` | `/auth/isValid/{userId}` | Check if a user exists and is valid |

### 🏃 Activity Service — `/activity`

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/activity/addActivity` | Log a new fitness activity |
| `GET` | `/activity/isActivityValid/{activityId}` | Check if an activity exists and is valid |

### 🤖 AI Service — `/ai`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/ai/userRec/{userId}` | Get all AI recommendations for a user |
| `GET` | `/ai/activityRec/{activityId}` | Get AI recommendation for a specific activity |

---

## 🔐 Security

Authentication is handled by **Keycloak** using the **OAuth2 Authorization Code Flow** with **JWT tokens**.

- All API requests require a valid `Bearer` JWT token in the `Authorization` header
- The **API Gateway** validates JWT tokens before forwarding to downstream services
- **Eureka** and **Config Server** are internal and not exposed publicly

```
Authorization: Bearer <your_jwt_token>
```

Obtain a token:
```bash
curl -X POST http://localhost:9090/realms/fitness-app/protocol/openid-connect/token \
  -d "client_id=fitness-client" \
  -d "username=youruser" \
  -d "password=yourpassword" \
  -d "grant_type=password"
```

---

## 📨 Event-Driven Communication

**Apache Kafka** powers async communication between the Activity and AI services.

| Topic | Producer | Consumer | Description |
|---|---|---|---|
| `activity.logged` | Activity Service | AI Service | Triggers recommendation generation when an activity is logged |

### Flow

```
User logs a fitness activity
          ↓
Activity Service saves to MongoDB
          ↓
Publishes event → Kafka: activity.logged
          ↓
AI Service consumes the event
          ↓
Generates & stores personalized recommendation in MongoDB
          ↓
Available via GET /ai/userRec/{userId}
         or GET /ai/activityRec/{activityId}
```

---

## 🔍 Service Discovery (Eureka)

All microservices register themselves with the **Eureka Server** on startup. The API Gateway uses Eureka for dynamic load-balanced routing — no hardcoded service URLs anywhere.

Access the Eureka dashboard at: `http://localhost:8761`

---

## ⚙️ Config Server

All service configurations (database URLs, Kafka settings, Keycloak properties) are centralized in the **Config Server**. Services fetch their config on startup via:

```
http://localhost:8888/{service-name}/{profile}
```

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---
