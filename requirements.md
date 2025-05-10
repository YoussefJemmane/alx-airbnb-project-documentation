# Airbnb Clone Backend - Requirement Specifications

This document outlines the technical and functional requirements for core backend features of the Airbnb Clone project.

---

## 1. User Authentication

### API Endpoints

- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/profile`
- `PUT /api/v1/auth/profile`

### Input/Output Specifications

#### Register (`POST /api/v1/auth/register`)
- **Input:**  
  - `email` (string, required, valid email format)
  - `password` (string, required, min 8 chars)
  - `role` (string, required, enum: `guest`, `host`)
- **Output:**  
  - `201 Created`  
    ```json
    {
      "id": "uuid",
      "email": "user@example.com",
      "role": "guest",
      "token": "jwt-token"
    }
    ```
  - `400 Bad Request` for validation errors

#### Login (`POST /api/v1/auth/login`)
- **Input:**  
  - `email` (string, required)
  - `password` (string, required)
- **Output:**  
  - `200 OK`  
    ```json
    {
      "token": "jwt-token"
    }
    ```
  - `401 Unauthorized` for invalid credentials

#### Profile (`GET/PUT /api/v1/auth/profile`)
- **Input:**  
  - JWT token in Authorization header
  - For `PUT`: profile fields (e.g., `name`, `photo`, `phone`)
- **Output:**  
  - `200 OK` with user profile data

### Validation Rules

- Email must be unique and valid format.
- Password must be at least 8 characters, contain letters and numbers.
- Role must be either `guest` or `host`.

### Performance Criteria

- Registration and login responses within 500ms under normal load.
- JWT tokens expire after 24 hours.

---

## 2. Property Management

### API Endpoints

- `POST /api/v1/properties`
- `GET /api/v1/properties`
- `GET /api/v1/properties/{id}`
- `PUT /api/v1/properties/{id}`
- `DELETE /api/v1/properties/{id}`

### Input/Output Specifications

#### Create Property (`POST /api/v1/properties`)
- **Input:**  
  - `title` (string, required)
  - `description` (string, required)
  - `location` (string, required)
  - `price` (number, required, min 0)
  - `amenities` (array of strings, optional)
  - `availability` (date range, required)
  - `images` (array of file uploads, optional)
- **Output:**  
  - `201 Created` with property details

#### Get Properties (`GET /api/v1/properties`)
- **Input:**  
  - Query params: `location`, `price_min`, `price_max`, `guests`, `amenities`, `page`, `limit`
- **Output:**  
  - `200 OK` with paginated list of properties

#### Update/Delete Property
- **Input:**  
  - JWT token (host only)
  - Property fields to update
- **Output:**  
  - `200 OK` with updated property or `204 No Content` for delete

### Validation Rules

- Only authenticated hosts can create, update, or delete properties.
- Title, description, location, and price are required.
- Price must be a positive number.
- Images must be valid file types (jpg, png).

### Performance Criteria

- Search and filter queries return results within 1 second.
- Pagination supported for large datasets.

---

## 3. Booking System

### API Endpoints

- `POST /api/v1/bookings`
- `GET /api/v1/bookings`
- `GET /api/v1/bookings/{id}`
- `PUT /api/v1/bookings/{id}/cancel`

### Input/Output Specifications

#### Create Booking (`POST /api/v1/bookings`)
- **Input:**  
  - `property_id` (uuid, required)
  - `check_in` (date, required)
  - `check_out` (date, required)
  - `guests` (number, required)
- **Output:**  
  - `201 Created` with booking details

#### Get Bookings (`GET /api/v1/bookings`)
- **Input:**  
  - JWT token (guest or host)
  - Query params: `status`, `property_id`, `page`, `limit`
- **Output:**  
  - `200 OK` with list of bookings

#### Cancel Booking (`PUT /api/v1/bookings/{id}/cancel`)
- **Input:**  
  - JWT token (guest or host)
- **Output:**  
  - `200 OK` with updated booking status

### Validation Rules

- Check-in date must be before check-out date.
- No overlapping bookings for the same property.
- Only the booking owner or host can cancel a booking.
- Bookings cannot be made for unavailable dates.

### Performance Criteria

- Booking creation and availability checks complete within 1 second.
- Prevent race conditions on concurrent bookings.

---