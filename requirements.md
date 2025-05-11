# Airbnb Clone - Backend Requirements Specification

This document outlines the detailed technical and functional requirements for three key backend features of the Airbnb Clone platform: User Authentication, Property Management, and Booking System.

## 1. User Authentication System

### 1.1 Functional Requirements

#### 1.1.1 User Registration
- The system shall allow users to register as either hosts or guests
- Users shall provide the following mandatory information during registration:
  - Email address (must be unique in the system)
  - Password (minimum 8 characters, must include uppercase, lowercase, numbers)
  - First name
  - Last name
  - Phone number
  - Date of birth (users must be at least 18 years old)
- The system shall validate email using a verification link
- Registration shall support both email/password and OAuth options (Google, Facebook)

#### 1.1.2 User Login
- Users shall be able to log in using email/password or OAuth providers
- The system shall implement JWT (JSON Web Token) for authentication
- JWTs shall expire after 24 hours
- The system shall allow for "Remember Me" functionality that extends token life to 7 days
- After 3 failed login attempts, the system shall enforce a 15-minute lockout period

#### 1.1.3 Password Management
- Users shall be able to reset their password via email verification
- Password reset links shall expire after 1 hour
- Users shall be able to change their password when logged in
- The system shall not store plain-text passwords, only securely hashed values

#### 1.1.4 User Profile Management
- Users shall be able to update their profile information
- Users shall be able to upload a profile picture
- Users shall be able to delete their account
- The system shall provide role-based access control for guests, hosts, and admins

### 1.2 API Endpoints

#### 1.2.1 Registration and Login
```
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/logout
GET /api/v1/auth/verify/{token}
POST /api/v1/auth/password/reset
POST /api/v1/auth/password/reset/{token}
POST /api/v1/auth/oauth/{provider}
```

#### 1.2.2 User Profile
```
GET /api/v1/users/me
PUT /api/v1/users/me
DELETE /api/v1/users/me
POST /api/v1/users/me/picture
DELETE /api/v1/users/me/picture
```

### 1.3 Input/Output Specifications

#### 1.3.1 Registration Request
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "+12345678901",
  "dateOfBirth": "1990-01-01",
  "userType": "guest" // or "host"
}
```

#### 1.3.2 Registration Response
```json
{
  "success": true,
  "message": "Registration successful. Verification email sent.",
  "userId": "12345abcde"
}
```

#### 1.3.3 Login Request
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "rememberMe": false
}
```

#### 1.3.4 Login Response
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresAt": "2023-05-12T15:30:45Z",
  "user": {
    "id": "12345abcde",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "userType": "guest",
    "profilePicture": "https://example.com/profiles/12345.jpg"
  }
}
```

### 1.4 Validation Rules

- Email: Must be a valid email format and unique in the system
- Password: Minimum 8 characters, must include uppercase, lowercase, numbers
- Phone: Must be a valid international phone number format
- Date of Birth: Must be a valid date and user must be at least 18 years old
- Profile Picture: Maximum size 5MB, formats accepted: JPG, PNG, GIF

### 1.5 Performance Criteria

- Registration and login requests shall be processed within 1 second under normal load
- The system shall support 1000 concurrent authentication requests
- Password hashing shall use bcrypt with a work factor of 12
- The system shall maintain 99.9% uptime for authentication services
- Database queries related to authentication shall execute in under 100ms

### 1.6 Security Requirements

- All API endpoints shall be HTTPS only
- Password reset tokens shall be single-use only
- Authentication failure logs shall be maintained for audit purposes
- User data shall be encrypted at rest
- Session tokens shall be invalidated upon password change

## 2. Property Management System

### 2.1 Functional Requirements

#### 2.1.1 Property Creation
- Hosts shall be able to create new property listings
- Each property listing shall include basic information, location details, amenities, pricing, and availability
- Hosts shall be able to upload multiple photos for each property
- Hosts shall be able to set house rules and policies
- Hosts shall be able to set pricing variables (e.g., weekend rates, seasonal pricing)

#### 2.1.2 Property Editing
- Hosts shall be able to edit all property details at any time
- Hosts shall be able to add, remove, or reorder property photos
- Editing a property that has pending bookings shall notify affected guests if the changes are substantial

#### 2.1.3 Property Deletion/Deactivation
- Hosts shall be able to temporarily deactivate their property listings
- Hosts shall be able to permanently delete property listings
- Properties with future bookings cannot be deleted until bookings are fulfilled or canceled

#### 2.1.4 Property Availability Management
- Hosts shall be able to manage property availability using a calendar interface
- Hosts shall be able to block dates when the property is unavailable
- The system shall automatically block dates when a booking is confirmed

### 2.2 API Endpoints

#### 2.2.1 Property Management
```
POST /api/v1/properties
GET /api/v1/properties
GET /api/v1/properties/{propertyId}
PUT /api/v1/properties/{propertyId}
DELETE /api/v1/properties/{propertyId}
PATCH /api/v1/properties/{propertyId}/status
```

#### 2.2.2 Property Media
```
POST /api/v1/properties/{propertyId}/media
GET /api/v1/properties/{propertyId}/media
DELETE /api/v1/properties/{propertyId}/media/{mediaId}
PUT /api/v1/properties/{propertyId}/media/reorder
```

#### 2.2.3 Property Availability
```
GET /api/v1/properties/{propertyId}/availability
POST /api/v1/properties/{propertyId}/availability/block
DELETE /api/v1/properties/{propertyId}/availability/block/{blockId}
```

### 2.3 Input/Output Specifications

#### 2.3.1 Create Property Request
```json
{
  "title": "Cozy Beachfront Villa",
  "description": "Beautiful villa with ocean views...",
  "propertyType": "villa",
  "roomType": "entire_place",
  "location": {
    "address": "123 Beach Road",
    "city": "Malibu",
    "state": "California",
    "zipCode": "90265",
    "country": "USA",
    "latitude": 34.025922,
    "longitude": -118.779757
  },
  "amenities": ["wifi", "kitchen", "pool", "parking"],
  "bathrooms": 2,
  "bedrooms": 3,
  "beds": 4,
  "maxGuests": 6,
  "pricing": {
    "basePrice": 150.00,
    "cleaningFee": 50.00,
    "securityDeposit": 200.00,
    "weekendPricing": 180.00,
    "currency": "USD"
  },
  "houseRules": {
    "petsAllowed": true,
    "smokingAllowed": false,
    "eventsAllowed": false,
    "quietHours": "10:00 PM - 8:00 AM"
  }
}
```

#### 2.3.2 Create Property Response
```json
{
  "success": true,
  "message": "Property created successfully",
  "propertyId": "prop12345",
  "createdAt": "2023-05-12T10:30:45Z"
}
```

#### 2.3.3 Get Property Response
```json
{
  "id": "prop12345",
  "title": "Cozy Beachfront Villa",
  "description": "Beautiful villa with ocean views...",
  "propertyType": "villa",
  "roomType": "entire_place",
  "location": {
    "address": "123 Beach Road",
    "city": "Malibu",
    "state": "California",
    "zipCode": "90265",
    "country": "USA",
    "latitude": 34.025922,
    "longitude": -118.779757
  },
  "amenities": ["wifi", "kitchen", "pool", "parking"],
  "bathrooms": 2,
  "bedrooms": 3,
  "beds": 4,
  "maxGuests": 6,
  "pricing": {
    "basePrice": 150.00,
    "cleaningFee": 50.00,
    "securityDeposit": 200.00,
    "weekendPricing": 180.00,
    "currency": "USD"
  },
  "houseRules": {
    "petsAllowed": true,
    "smokingAllowed": false,
    "eventsAllowed": false,
    "quietHours": "10:00 PM - 8:00 AM"
  },
  "host": {
    "id": "host12345",
    "name": "Jane Smith",
    "profilePicture": "https://example.com/profiles/host12345.jpg",
    "joinedDate": "2022-01-15",
    "responseRate": 98,
    "responseTime": "within an hour",
    "isSuperhost": true
  },
  "media": [
    {
      "id": "media1",
      "url": "https://example.com/properties/prop12345/1.jpg",
      "isPrimary": true,
      "caption": "Villa exterior"
    },
    {
      "id": "media2",
      "url": "https://example.com/properties/prop12345/2.jpg",
      "isPrimary": false,
      "caption": "Master bedroom"
    }
  ],
  "rating": {
    "average": 4.8,
    "cleanliness": 4.9,
    "accuracy": 4.7,
    "communication": 5.0,
    "location": 4.8,
    "checkIn": 4.6,
    "value": 4.7,
    "totalReviews": 23
  },
  "createdAt": "2023-05-12T10:30:45Z",
  "updatedAt": "2023-05-12T10:30:45Z",
  "status": "active"
}
```

### 2.4 Validation Rules

- Property Title: Required, 5-100 characters
- Property Description: Required, 100-5000 characters
- Property Type: Required, must be one of predefined types
- Room Type: Required, must be one of predefined types
- Location: All address fields required except for apartment/unit number
- Coordinates: Required and must be valid geographic coordinates
- Pricing: Base price required, must be greater than 0
- Currency: Required, must be a valid ISO currency code
- Photos: At least one photo required, maximum 50 photos

### 2.5 Performance Criteria

- Property creation shall complete within 3 seconds
- Property queries shall return results within 500ms
- Photo uploads shall process at 5MB/s minimum
- The system shall support up to 100 concurrent property creation/edit operations
- The system shall handle 10,000 property queries per minute

### 2.6 Security Requirements

- Only authenticated hosts can create, edit, or delete properties
- Hosts can only modify their own properties
- Property location details shall be visible only to users with confirmed bookings
- Property images shall be scanned for inappropriate content before publication

## 3. Booking System

### 3.1 Functional Requirements

#### 3.1.1 Availability Checking
- The system shall check property availability before allowing a booking
- The system shall prevent double bookings for the same dates
- The system shall account for check-in and check-out times when determining availability

#### 3.1.2 Booking Creation
- Guests shall be able to book available properties for specific dates
- Bookings shall require guest count and trip purpose information
- The system shall calculate the total price including all fees and taxes
- The system shall hold a tentative booking for 15 minutes during payment processing

#### 3.1.3 Booking Management
- Guests shall be able to view their upcoming, current, and past bookings
- Hosts shall be able to view and manage bookings for their properties
- The system shall support booking modification if within the host's modification policy
- The system shall support booking cancellation according to cancellation policies

#### 3.1.4 Booking Statuses
- The system shall track booking statuses: pending, confirmed, canceled, completed
- Status changes shall trigger appropriate notifications
- The system shall automatically update status based on check-in/check-out dates

### 3.2 API Endpoints

#### 3.2.1 Availability
```
GET /api/v1/properties/{propertyId}/availability?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD
```

#### 3.2.2 Booking Management
```
POST /api/v1/bookings
GET /api/v1/bookings
GET /api/v1/bookings/{bookingId}
PUT /api/v1/bookings/{bookingId}
DELETE /api/v1/bookings/{bookingId}
PATCH /api/v1/bookings/{bookingId}/status
```

#### 3.2.3 Guest-specific Endpoints
```
GET /api/v1/guests/me/bookings
GET /api/v1/guests/me/bookings/{status}
```

#### 3.2.4 Host-specific Endpoints
```
GET /api/v1/hosts/me/bookings
GET /api/v1/hosts/me/bookings/{status}
PATCH /api/v1/hosts/me/bookings/{bookingId}/approval
```

### 3.3 Input/Output Specifications

#### 3.3.1 Create Booking Request
```json
{
  "propertyId": "prop12345",
  "checkIn": "2023-06-15",
  "checkOut": "2023-06-20",
  "guestCount": 4,
  "adults": 2,
  "children": 2,
  "infants": 0,
  "specialRequests": "We'll be arriving late, around 9 PM.",
  "tripPurpose": "family_vacation"
}
```

#### 3.3.2 Create Booking Response
```json
{
  "success": true,
  "bookingId": "book67890",
  "status": "pending",
  "totalPrice": {
    "nightlyRate": 750.00, // 5 nights @ $150/night
    "cleaningFee": 50.00,
    "serviceFee": 120.00,
    "taxes": 92.00,
    "total": 1012.00,
    "currency": "USD"
  },
  "paymentDeadline": "2023-05-12T11:00:45Z",
  "paymentUrl": "https://example.com/payment/book67890"
}
```

#### 3.3.3 Get Booking Response
```json
{
  "id": "book67890",
  "property": {
    "id": "prop12345",
    "title": "Cozy Beachfront Villa",
    "address": "123 Beach Road, Malibu, CA",
    "primaryImage": "https://example.com/properties/prop12345/1.jpg"
  },
  "host": {
    "id": "host12345",
    "name": "Jane Smith",
    "profilePicture": "https://example.com/profiles/host12345.jpg",
    "phone": "+12345678901" // Only visible after booking confirmation
  },
  "guest": {
    "id": "guest98765",
    "name": "John Doe",
    "profilePicture": "https://example.com/profiles/guest98765.jpg"
  },
  "details": {
    "checkIn": "2023-06-15",
    "checkOut": "2023-06-20",
    "guestCount": 4,
    "adults": 2,
    "children": 2,
    "infants": 0,
    "nightsCount": 5,
    "checkInInstructions": "The lockbox code is 1234. Please call when you arrive.",
    "houseRules": "No parties or events. Quiet hours from 10 PM to 8 AM."
  },
  "pricing": {
    "nightlyRate": 750.00,
    "cleaningFee": 50.00,
    "serviceFee": 120.00,
    "taxes": 92.00,
    "total": 1012.00,
    "currency": "USD",
    "paymentStatus": "paid"
  },
  "status": "confirmed",
  "cancellationPolicy": {
    "type": "moderate",
    "description": "Full refund 5 days prior to check-in",
    "refundableUntil": "2023-06-10"
  },
  "createdAt": "2023-05-12T10:45:00Z",
  "updatedAt": "2023-05-12T11:15:30Z"
}
```

### 3.4 Validation Rules

- Check-in Date: Must be in the future and no more than 2 years in advance
- Check-out Date: Must be after check-in date
- Stay Duration: Cannot exceed maximum stay duration set by host
- Guest Count: Must not exceed property's maximum occupancy
- Property ID: Must be valid and active
- Guest must be verified before creating bookings

### 3.5 Performance Criteria

- Availability checks shall complete within 200ms
- Booking creation shall complete within 2 seconds
- The system shall support 5,000 concurrent availability checks
- The system shall support 500 concurrent booking creations
- The system shall maintain booking transaction atomicity under high load

### 3.6 Security Requirements

- Payment information shall be processed through PCI-compliant services only
- Guest contact information shall be shared with hosts only after booking confirmation
- Host contact information shall be shared with guests only after booking confirmation
- Booking details shall be accessible only to the involved guest, host, and administrators
- All booking transactions shall be logged for audit purposes