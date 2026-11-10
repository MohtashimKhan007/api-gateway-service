# GateKeeper: API Gateway Service

### Multi-Tenant Organization & Access Management System

GateKeeper is a Spring Boot API gateway with a built-in management console. A **control plane** lets platform admins provision organizations (tenants), while each tenant admin configures **gateway routes**, traffic policies and versions, all isolated per tenant and secured with stateless **JWT authentication**.

> **One control plane. Every tenant in its lane.**

![Java](https://img.shields.io/badge/Java-ED8B00?style=flat&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-6DB33F?style=flat&logo=springboot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring%20Security-6DB33F?style=flat&logo=springsecurity&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=flat&logo=jsonwebtokens&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=flat&logo=apachemaven&logoColor=white)

---

## Table of Contents

- [Overview](#overview)
- [Screenshots](#screenshots)
- [Key Features](#key-features)
- [Roles and Access](#roles-and-access)
- [Tenant Provisioning Workflow](#tenant-provisioning-workflow)
- [Route Management and Versioning](#route-management-and-versioning)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Authentication Flow](#authentication-flow)
- [Error Handling](#error-handling)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## Overview

GateKeeper is organized around three ideas:

| Step | Plane | What it does |
|------|-------|--------------|
| 01 | **Manage** (Control plane) | Provision organizations and assign tenant administrators |
| 02 | **Route** (Data plane) | Configure gateway routes, policies and versions per tenant |
| 03 | **Observe** (Analytics) | Keep every tenant boundary explicit and visible |

Every request passes through Spring Security filters that validate the JWT and enforce access based on the user's **role** and **tenant context**.

## Screenshots

### Login
<img width="1864" height="906" alt="image" src="https://github.com/user-attachments/assets/42768d9a-ec13-40e2-a091-7c57b5510c75" />


### Platform Overview (Control Plane)
<img width="1902" height="872" alt="image" src="https://github.com/user-attachments/assets/c012bfce-ee51-40c1-8c46-1e6f8b092253" />


### Organizations (Tenant Directory)
<img width="1634" height="670" alt="image" src="https://github.com/user-attachments/assets/f5d24f04-ae91-4e83-b65c-ecc2e798d33b" />


### Routes (Gateway Configuration)
<img width="1651" height="693" alt="image" src="https://github.com/user-attachments/assets/e04cd03a-47b9-412e-871a-4302dde66cfd" />


### Route Version History
<img width="931" height="513" alt="image" src="https://github.com/user-attachments/assets/b99d36b2-c0f6-46d3-8063-c890b360d692" />


## Key Features

- **Multi-tenant architecture** with isolated operations and data visibility per tenant
- **Hierarchical role-based access control** (Platform Admin and Tenant Admin)
- **Stateless JWT authentication** using custom Spring Security filters
- **Organization management**: create, edit, search, delete, and control status (Active, Pending, Suspended)
- **Tenant administrator assignment** for each organization
- **Gateway route management**: public path, upstream mapping, HTTP methods, activate/deactivate
- **Traffic policies** per route, such as rate limiting (for example 99 requests/min) and idempotency
- **Route versioning** with full configuration history (Created, Activated, Deactivated) and JSON snapshots
- **Management dashboard** with live organization counts and API connection status
- **DTO pattern** for secure request payload validation
- **JPA/Hibernate entities** for database persistence
- **Global exception handling** with `@ControllerAdvice` returning consistent JSON error responses

## Roles and Access

| Role | Scope | Capabilities |
|------|-------|--------------|
| `PLATFORM_ADMIN` | Whole platform | Provision organizations, change status, manage tenant administrators |
| `TENANT_ADMIN` | Own organization only | Manage routes, policies and versions for their organization |

Each tenant receives its own **JWT-scoped workspace**, so a tenant admin can never cross into another tenant's data.

## Tenant Provisioning Workflow

1. A Platform Admin creates an **organization** (name and slug, for example `/foodfast`).
2. Exactly **one tenant administrator** is assigned to the organization.
3. The organization is **activated**.
4. The tenant admin signs in and configures routes in their own workspace.

## Route Management and Versioning

Each route defines:

- **Name and upstream service** (for example `FoodFast Orders`)
- **Public path** (for example `/foodfast/orders/**`)
- **Allowed HTTP methods** (for example `POST`, `GET`)
- **Policies** (for example rate limit per minute, idempotent requests)
- **Status** (`ACTIVE` / `INACTIVE`)

Every change creates a new version, so the full configuration history stays available:

```
Version 1 - CREATED
Version 2 - ACTIVATED
Version 3 - DEACTIVATED
```

Each version has a **JSON snapshot** of the route configuration at that moment.

## Tech Stack

| Category | Technologies |
|----------|--------------|
| Language | Java |
| Framework | Spring Boot, Spring MVC |
| Security | Spring Security, JWT |
| Persistence | JPA / Hibernate |
| API Style | REST APIs, DTO Pattern |
| Error Handling | Global Exception Handling (`@ControllerAdvice`) |
| Build Tool | Maven |

## Architecture

```
Client / Management Console
  |
  v
Security Filter Chain  (JWT validation, role + tenant checks)
  |
  v
REST Controllers       (organizations, admins, routes; DTO validation)
  |
  v
Service Layer          (business logic, versioning)
  |
  v
Repository Layer       (Spring Data JPA)
  |
  v
Database
```

## Getting Started

### Prerequisites

- Java JDK 17 or higher
- Maven 3.6+
- A relational database (for example MySQL)
- Postman (optional, for testing APIs)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/MohtashimKhan007/api-gateway-service.git

# 2. Move into the project folder
cd api-gateway-service

# 3. Build the project
mvn clean install

# 4. Run the application
mvn spring-boot:run
```

The application starts on `http://localhost:8080` by default.

## Configuration

Update `src/main/resources/application.properties` with your own values:

```properties
# Database
spring.datasource.url=jdbc:mysql://localhost:3306/your_database
spring.datasource.username=your_username
spring.datasource.password=your_password

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=update

# JWT
jwt.secret=your_secret_key
jwt.expiration=86400000
```

> Never commit real credentials or secret keys. Use environment variables or a local, untracked properties file.

## Authentication Flow

1. A management user signs in with a username and password.
2. The server generates a signed **JWT** containing the user's identity, role and tenant.
3. The client sends the token in the `Authorization: Bearer <token>` header.
4. A Spring Security filter validates the token and sets the security context.
5. Access to endpoints and data is restricted by **role** and **tenant context**.

Authentication is fully **stateless**; no server-side session is stored.

## Error Handling

All errors are handled centrally with `@ControllerAdvice`, so every module returns a consistent JSON structure:

```json
{
  "timestamp": "2026-11-10T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "path": "/api/organizations"
}
```

## Future Improvements

- Refresh token support
- Request analytics and metrics dashboard
- Swagger / OpenAPI documentation
- Docker support for easier deployment
- Unit and integration tests

## Author

**Mohtashim Khan**
Java Backend Developer

- GitHub: [MohtashimKhan007](https://github.com/MohtashimKhan007)
- LinkedIn: [mohtashimkhansolutions](https://linkedin.com/in/mohtashimkhansolutions)
- Email: mohtashim.khan.tech@gmail.com
