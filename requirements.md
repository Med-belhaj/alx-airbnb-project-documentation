# Requirement Specifications

This document details the technical and functional requirements for three key backend features of the Airbnb Clone project: User Authentication, Property Management, and Booking System. Each section includes API endpoints, input/output specifications, validation rules, and performance criteria.

---
## 1. User Authentication

### 1.1 Overview
Enables secure user registration, login, and session management for guests, hosts, and admins.

### 1.2 API Endpoints
- **POST** `/api/auth/register/`
  - **Description**: Create a new user account (guest or host).
  - **Input (JSON)**:
    ```json
    {
      "username": "string",
      "email": "string",       # valid email format
      "password": "string",    # min 8 chars, at least one uppercase, one digit
      "role": "guest|host"
    }
    ```
  - **Output (201 Created)**:
    ```json
    { "id": 123, "username": "jdoe", "email": "jdoe@example.com", "role": "guest" }
    ```
  - **Errors**: 400 Bad Request on validation failure; 409 Conflict if email already exists.

- **POST** `/api/auth/login/`
  - **Description**: Authenticate user and return JWT.
  - **Input (JSON)**:
    ```json
    { "email": "string", "password": "string" }
    ```
  - **Output (200 OK)**:
    ```json
    { "access_token": "<jwt>", "refresh_token": "<jwt>", "expires_in": 3600 }
    ```
  - **Errors**: 401 Unauthorized on invalid credentials.

- **POST** `/api/auth/refresh/`
  - **Description**: Refresh access token.
  - **Input (JSON)**:
    ```json
    { "refresh_token": "<jwt>" }
    ```
  - **Output (200 OK)**:
    ```json
    { "access_token": "<jwt>", "expires_in": 3600 }
    ```

### 1.3 Validation Rules
- **Username**: 3–30 characters, alphanumeric and underscores only.
- **Email**: RFC 5322 compliant.
- **Password**: Minimum 8 characters, at least one uppercase letter, one lowercase letter, one digit.
- **Role**: Must be either `guest` or `host`.

### 1.4 Performance Criteria
- **Authentication latency**: < 200 ms per login request under normal load (100 concurrent users).
- **Scalability**: Support 5,000 active sessions per minute with horizontal scaling of auth service.

---
## 2. Property Management

### 2.1 Overview
Allows hosts to create, update, delete, and retrieve property listings.

### 2.2 API Endpoints
- **GET** `/api/properties/`
  - **Description**: List all available properties (paginated).
  - **Query Parameters**:
    - `page`: integer (default 1)
    - `page_size`: integer (max 50)
    - `location`: string (optional)
    - `price_min` / `price_max`: integers (optional)
  - **Output (200 OK)**:
    ```json
    {
      "results": [ { <property_object> }, ... ],
      "page": 1,
      "page_size": 20,
      "total": 125
    }
    ```

- **POST** `/api/properties/`
  - **Description**: Create a new property listing.
  - **Input (JSON)**:
    ```json
    {
      "title": "string",
      "description": "string",
      "location": "string",
      "price_per_night": number,    # >= 0
      "amenities": ["string", ...],
      "availability": [{ "start_date": "YYYY-MM-DD", "end_date": "YYYY-MM-DD" }, ...]
    }
    ```
  - **Output (201 Created)**:
    ```json
    { "id": 456, "title": "Cozy Cottage", "host_id": 123, ... }
    ```
  - **Errors**: 400 Bad Request on validation failure; 401 Unauthorized if not host.

- **PUT** `/api/properties/{id}/`
  - **Description**: Update an existing property.
  - **Input**: Same as creation payload.
  - **Output (200 OK)**: Updated property object.

- **DELETE** `/api/properties/{id}/`
  - **Description**: Delete a property listing.
  - **Output (204 No Content)**.
  - **Errors**: 403 Forbidden if host does not own the property.

### 2.3 Validation Rules
- **Title**: 5–100 characters.
- **Description**: 10–2000 characters.
- **Location**: Non-empty string.
- **Price per night**: Decimal >= 0.
- **Availability**: Valid date ranges, no overlaps.

### 2.4 Performance Criteria
- **List retrieval**: < 300 ms for paginated GET under 200 concurrent requests.
- **Write operations**: < 500 ms under normal load.

---
## 3. Booking System

### 3.1 Overview
Handles guest bookings, date availability checks, and cancellation policies.

### 3.2 API Endpoints
- **POST** `/api/bookings/`
  - **Description**: Create a new booking for a property.
  - **Input (JSON)**:
    ```json
    {
      "property_id": 456,
      "guest_id": 789,
      "start_date": "YYYY-MM-DD",
      "end_date": "YYYY-MM-DD",
      "payment_method": "stripe|paypal"
    }
    ```
  - **Output (201 Created)**:
    ```json
    { "id": 1011, "status": "pending", "total_price": 350.00, ... }
    ```
  - **Errors**: 400 Bad Request for invalid dates; 409 Conflict if dates overlap existing booking.

- **GET** `/api/bookings/{id}/`
  - **Description**: Retrieve booking details.
  - **Output (200 OK)**: Booking object.

- **PATCH** `/api/bookings/{id}/cancel/`
  - **Description**: Cancel a booking according to policy.
  - **Output (200 OK)**:`{ "status": "canceled" }`.

### 3.3 Validation Rules
- **Dates**: `start_date` < `end_date`; booking window <= 365 days.
- **Availability**: Dates must not overlap with existing bookings.
- **Payment method**: Must be one of supported gateways.

### 3.4 Performance Criteria
- **Booking creation**: < 400 ms under 100 concurrent bookings.
- **Availability check**: < 200 ms for date conflict queries.

---
