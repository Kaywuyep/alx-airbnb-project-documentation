# Backend Requirement Specifications

This document outlines the technical and functional requirements for key backend features of the Airbnb Clone project.

## 1. User Authentication

**1.1. Registration**

* **Description:** Allows new users (both guests and hosts) to create an account.
* **API Endpoint:** `POST /api/auth/register`
* **Input:**
    * `email` (string, required, unique, valid email format, max length: 50)
    * `password` (string, required, minimum length: 8, must contain at least one uppercase letter, one lowercase letter, and one digit)
    * `first_name` (string, required, max length: 100)
    * `last_name` (string, required, max length: 100)
    * `user_type` (string, required, enum: ['guest', 'host'])
    * `phone_number` (string, optional, valid phone number format, max length: 20)
* **Output (Success - HTTP 201 Created):**
    ```json
    {
        "message": "User registered successfully",
        "user_id": "<uuid_of_new_user>"
    }
    ```
* **Output (Failure - HTTP 400 Bad Request):**
    ```json
    {
        "error": "<error_message>",
        "details": {
            "<field_name>": "<validation_error>"
        }
    }
    ```
* **Output (Failure - HTTP 409 Conflict):**
    ```json
    {
        "error": "Email address already exists"
    }
    ```
* **Validation Rules:**
    * All required fields must be present.
    * `email` must be a valid email format and unique in the system.
    * `password` must meet the specified complexity requirements.
    * `user_type` must be either 'guest' or 'host'.
    * `phone_number` must be a valid phone number format if provided.
* **Performance Criteria:**
    * Registration should complete within 500 milliseconds under normal load.

**1.2. Login**

* **Description:** Allows registered users to authenticate and obtain an access token.
* **API Endpoint:** `POST /api/auth/login`
* **Input:**
    * `email` (string, required, valid email format, max length: 255)
    * `password` (string, required, minimum length: 8)
* **Output (Success - HTTP 200 OK):**
    ```json
    {
        "message": "Login successful",
        "access_token": "<jwt_access_token>"
    }
    ```
* **Output (Failure - HTTP 401 Unauthorized):**
    ```json
    {
        "error": "Invalid credentials"
    }
    ```
* **Validation Rules:**
    * Both `email` and `password` are required.
    * Provided credentials must match a registered user in the system.
* **Performance Criteria:**
    * Login should complete within 300 milliseconds under normal load.

**1.3. Logout**

* **Description:** Invalidates the current user's access token (implementation details may vary based on token management strategy).
* **API Endpoint:** `POST /api/auth/logout`
* **Input:**
    * `Authorization` header with the Bearer token (required).
* **Output (Success - HTTP 200 OK):**
    ```json
    {
        "message": "Logout successful"
    }
    ```
* **Output (Failure - HTTP 401 Unauthorized):**
    ```json
    {
        "error": "Invalid or expired token"
    }
    ```
* **Validation Rules:**
    * A valid access token must be provided in the `Authorization` header.
* **Performance Criteria:**
    * Logout should complete within 100 milliseconds under normal load.

## 2. Property Management

**2.1. Create Listing**

* **Description:** Allows hosts to add a new property listing to the platform.
* **API Endpoint:** `POST /api/properties`
* **Input (Multipart Form Data or JSON):**
    * `host_id` (uuid, required, foreign key referencing the `users` table with `user_type = 'host'`)
    * `title` (string, required, max length: 255)
    * `description` (string, required, max length: 1000)
    * `address` (string, required, max length: 255)
    * `city` (string, required, max length: 100)
    * `country` (string, required, max length: 100)
    * `latitude` (decimal, required)
    * `longitude` (decimal, required)
    * `property_type` (string, required, enum: ['apartment', 'house', 'condo', 'etc.'])
    * `bedrooms` (integer, required, minimum: 1)
    * `bathrooms` (decimal, required, minimum: 0.5)
    * `beds` (integer, required, minimum: 1)
    * `amenities` (array of strings, optional)
    * `price_per_night` (decimal, required, minimum: 0.01)
    * `images` (array of files, optional, max 5)
* **Output (Success - HTTP 201 Created):**
    ```json
    {
        "message": "Property listing created successfully",
        "property_id": "<uuid_of_new_property>"
    }
    ```
* **Output (Failure - HTTP 400 Bad Request):** (Similar structure to registration failure)
* **Output (Failure - HTTP 401 Unauthorized):** (If authentication is required for this action)
* **Validation Rules:**
    * All required fields must be present.
    * `host_id` must correspond to an existing host user.
    * `title` and `description` must adhere to length constraints.
    * `latitude` and `longitude` must be valid decimal numbers.
    * `bedrooms`, `bathrooms`, and `beds` must be positive numbers.
    * `price_per_night` must be a positive decimal.
    * If images are provided, there should be a maximum of 5.
* **Performance Criteria:**
    * Listing creation (excluding image processing) should complete within 1 second under normal load.
    * Image uploads should complete within 2 seconds per image.

**2.2. Get Property Details**

* **Description:** Retrieves detailed information for a specific property.
* **API Endpoint:** `GET /api/properties/{property_id}`
* **Input:**
    * `property_id` (uuid, required, in the URL path)
* **Output (Success - HTTP 200 OK):**
    ```json
    {
        "id": "<uuid_of_property>",
        "host_id": "<uuid_of_host>",
        "title": "Cozy Apartment in Downtown",
        "description": "...",
        "address": "123 Main St",
        "city": "New York",
        "country": "USA",
        "latitude": 40.7128,
        "longitude": -74.0060,
        "property_type": "apartment",
        "bedrooms": 2,
        "bathrooms": 1.5,
        "beds": 2,
        "amenities": ["wifi", "kitchen", "tv"],
        "price_per_night": 120.00,
        "images": ["url1.jpg", "url2.jpg"],
        "created_at": "2025-05-09T14:00:00Z",
        "updated_at": "2025-05-09T14:00:00Z"
    }
    ```
* **Output (Failure - HTTP 404 Not Found):**
    ```json
    {
        "error": "Property not found"
    }
    ```
* **Validation Rules:**
    * `property_id` must be a valid UUID.
* **Performance Criteria:**
    * Retrieving property details should complete within 200 milliseconds under normal load.

## 3. Booking System

**3.1. Check Availability**

* **Description:** Checks if a specific property is available for a given date range.
* **API Endpoint:** `GET /api/properties/{property_id}/availability`
* **Input:**
    * `property_id` (uuid, required, in the URL path)
    * `start_date` (string, required, format: YYYY-MM-DD)
    * `end_date` (string, required, format: YYYY-MM-DD)
* **Output (Success - HTTP 200 OK):**
    ```json
    {
        "available": true
    }
    ```
    or
    ```json
    {
        "available": false,
        "conflicting_bookings": [
            {
                "booking_id": "<uuid_of_conflicting_booking>",
                "start_date": "2025-05-15",
                "end_date": "2025-05-18"
            }
        ]
    }
    ```
* **Output (Failure - HTTP 400 Bad Request):** (e.g., invalid date format)
* **Output (Failure - HTTP 404 Not Found):** (Property not found)
* **Validation Rules:**
    * `property_id` must be a valid UUID.
    * `start_date` and `end_date` must be valid dates in YYYY-MM-DD format.
    * `start_date` must be before or equal to `end_date`.
* **Performance Criteria:**
    * Checking availability should complete within 300 milliseconds under normal load.

**3.2. Create Booking**

* **Description:** Allows a guest to book a property for a specified date range.
* **API Endpoint:** `POST /api/bookings`
* **Input:**
    * `guest_id` (uuid, required, foreign key referencing the `users` table with `user_type = 'guest'`)
    * `property_id` (uuid, required, foreign key referencing the `properties` table)
    * `start_date` (string, required, format: YYYY-MM-DD)
    * `end_date` (string, required, format: YYYY-MM-DD)
    * `number_of_guests` (integer, required, minimum: 1)
    * `total_price` (decimal, required, minimum: 0.01)
* **Output (Success - HTTP 201 Created):**
    ```json
    {
        "message": "Booking created successfully",
        "booking_id": "<uuid_of_new_booking>"
    }
    ```
* **Output (Failure - HTTP 400 Bad Request):** (e.g., invalid date format, property not available, guest count exceeds capacity)
* **Output (Failure - HTTP 401 Unauthorized):** (If authentication is required)
* **Output (Failure - HTTP 404 Not Found):** (Guest or property not found)
* **Validation Rules:**
    * All required fields must be present.
    * `guest_id` and `property_id` must correspond to existing records.
    * `start_date` and `end_date` must be valid dates, with `start_date` before `end_date`.
    * The property must be available for the specified date range (check against existing bookings).
    * `number_of_guests` must be within the property's capacity (if applicable).
    * `total_price` must be consistent with the calculated price based on the property's rate and booking duration.
* **Performance Criteria:**
    * Creating a booking (excluding payment processing) should complete within 1 second under normal load.