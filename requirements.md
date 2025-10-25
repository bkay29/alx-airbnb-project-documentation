
# Airbnb Clone Backend - Technical & Functional Requirements

This document outlines the **technical and functional requirements** for key backend features of the Airbnb Clone project.
Each section includes API endpoint specifications, input/output structures, validation rules, and performance criteria.

---

## 1. User Authentication

### Functional Requirements
- Allow users to **register** as guests or hosts.
- Enable **login** via email/password and OAuth (Google/Facebook).
- Use **JWT** for secure session management.
- Provide **password recovery** via email verification link.
- Allow users to **update their profile** (photo, contact info, preferences).

###  API Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/auth/register` | Register a new user (guest or host) |
| `POST` | `/api/v1/auth/login` | Authenticate user and issue JWT |
| `POST` | `/api/v1/auth/forgot-password` | Send password reset link |
| `PUT` | `/api/v1/users/{id}/profile` | Update user profile info |
| `GET` | `/api/v1/users/{id}` | Retrieve user profile |

### Input Specifications

#### Register
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "StrongPass123!",
  "role": "host"
}
```

#### Login
```json
{
  "email": "jane@example.com",
  "password": "StrongPass123!"
}
```

### Output Specifications
```json
{
  "user": {
    "id": 1,
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "host"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Validation Rules
- Email must be valid and unique.  
- Password must contain at least 8 characters, 1 uppercase, 1 digit, and 1 symbol.  
- Role must be either `guest` or `host`.
- JWT expiration: 1 hour; refresh token valid for 7 days.

### Performance Criteria
- Authentication response time ≤ 300ms.
- Login rate limit: 5 requests/min per IP (to prevent brute force).  
- Passwords encrypted with bcrypt (≥12 salt rounds).

---

## 2. Property Management

### Functional Requirements
- Hosts can **create, edit, view, and delete** property listings.  
- Listings include **title, description, location, price, amenities, and images**.  
- Support **upload and storage** of images (e.g., AWS S3 or local storage).  
- Only property owners (hosts) can modify or delete their listings.

### API Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/properties` | Create a new property listing |
| `GET` | `/api/v1/properties` | Retrieve all properties (with filters) |
| `GET` | `/api/v1/properties/{id}` | Retrieve property by ID |
| `PUT` | `/api/v1/properties/{id}` | Update property details |
| `DELETE` | `/api/v1/properties/{id}` | Delete a property |

### Input Specifications
```json
{
  "title": "Cozy Beachfront Villa",
  "description": "A modern villa with an ocean view.",
  "price_per_night": 150.0,
  "location": "Diani, Kenya",
  "max_guests": 4,
  "amenities": ["Wi-Fi", "Pool", "Air Conditioning"]
}
```

### Output Specifications
```json
{
  "id": 101,
  "host_id": 12,
  "title": "Cozy Beachfront Villa",
  "location": "Diani, Kenya",
  "price_per_night": 150.0,
  "created_at": "2025-10-25T09:00:00Z"
}
```

### Validation Rules
- Title and description: required, max length 255 characters.
- Price: must be positive number.
- Location: must include city and country.
- Amenities: must be an array of valid predefined strings.
- Only the host who created the property can modify or delete it.

### Performance Criteria
- Query optimization with pagination (`limit`, `offset`).  
- Cached search results via Redis.  
- Average property fetch time ≤ 500ms.  
- Indexing on location and price fields.

---

## 3. Booking System

### Functional Requirements
- Guests can book available properties for specific dates.  
- Prevent **double bookings** using date range validation.  
- Maintain **booking statuses**: pending, confirmed, cancelled, completed.  
- Allow **cancellations** (guest/host) with reason tracking.  
- Trigger **email notifications** for booking events.

### API Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/bookings` | Create a new booking |
| `GET` | `/api/v1/bookings` | Retrieve all bookings for user |
| `GET` | `/api/v1/bookings/{id}` | Retrieve booking details |
| `PATCH` | `/api/v1/bookings/{id}/cancel` | Cancel an existing booking |
| `GET` | `/api/v1/properties/{id}/availability` | Check available dates |

### Input Specifications

#### Create Booking
```json
{
  "property_id": 101,
  "guest_id": 56,
  "check_in": "2025-11-05",
  "check_out": "2025-11-10"
}
```

### Output Specifications
```json
{
  "booking_id": 301,
  "property_id": 101,
  "guest_id": 56,
  "status": "pending",
  "check_in": "2025-11-05",
  "check_out": "2025-11-10",
  "total_price": 750.0
}
```

### Validation Rules
- Check-in and check-out must be valid ISO dates.  
- Check-out date must be after check-in.  
- Property must be available for selected dates.  
- Prevent overlapping bookings for same property.  
- Only guests can create bookings; only hosts or admins can approve/cancel them.

### Performance Criteria
- Booking creation latency ≤ 600ms.  
- Transactions must ensure **atomicity** (no double booking).  
- Use background jobs to send confirmation emails asynchronously.  
- Database indexing on property_id, guest_id, and date ranges.

---

## Notes
These requirements provide a foundation for implementing a production-ready backend for the Airbnb Clone project.  
Additional modules (Payments, Reviews, Notifications, Admin Dashboard) will extend these core features in future iterations.