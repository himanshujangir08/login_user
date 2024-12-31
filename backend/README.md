# User Registration System Documentation

## System Overview

A Node.js-based user registration system using Express.js, MongoDB, and JWT authentication.

## Architecture Components

## ENDPOINTS

`/users/user-register`
`/users/user-login`
`/users/user-profile`
`/users/user-logout`

### Models (user.model.js)

- **Schema Definition**:

  - `fullname`: Object containing `firstname` (required, min 3 chars) and `lastname` (min 3 chars)
  - `email`: String (required, unique, min 5 chars)
  - `password`: String (required, excluded from queries)
  - `socketId`: String (optional)

- **Methods**:
  - `generateAuthToken()`: Creates JWT token valid for 7 days
  - `comparePassword()`: Compares plain and hashed passwords
  - `hashPassword()`: Static method to hash passwords using bcrypt (10 rounds)

### Services (user.service.js)

- **createUser Function**
  - Parameters: `firstname`, `lastname`, `email`, `password`
  - Validates required fields
  - Creates user document in MongoDB
  - Returns created user object

### Controllers (user.controller.js)

- **registerUser Function**
  - Validates request data using express-validator
  - Extracts user details from request body
  - Hashes password
  - Creates user via service
  - Generates authentication token
  - Returns user data and token with 201 status

- **loginUser Function**
  - Validates request data using express-validator
  - Extracts user details from request body
  - Finds user by email
  - Compares password
  - Generates authentication token
  - Returns user data and token with 200 status

- **getUserProfile Function**
  - Retrieves user profile data from the request object
  - Returns user data with 200 status

- **logoutUser Function**
  - Clears the authentication token cookie
  - Adds the token to the blacklist
  - Returns a success message with 200 status

### Routes (user.routes.js)

- **POST /user-register**
  - Validation Rules:
    - First name: minimum 3 characters
    - Email: valid email format
    - Password: minimum 6 characters
  - Executes registerUser controller on validation success

- **POST /user-login**
  - Validation Rules:
    - Email: valid email format
    - Password: not empty
  - Executes loginUser controller on validation success

- **GET /user-profile**
  - Requires authentication
  - Executes getUserProfile controller on validation success

- **GET /user-logout**
  - Requires authentication
  - Executes logoutUser controller on validation success

## Request/Response Flow

1. Client sends POST request to `/user-register`
2. Request validation occurs
3. Password hashing
4. User creation in database
5. JWT token generation
6. Response with user data and token

1. Client sends POST request to `/user-login`
2. Request validation occurs
3. User lookup by email
4. Password comparison
5. JWT token generation
6. Response with user data and token

1. Client sends GET request to `/user-profile`
2. Authentication middleware validates token
3. User profile data is retrieved
4. Response with user data

1. Client sends GET request to `/user-logout`
2. Authentication middleware validates token
3. Token is added to blacklist
4. Response with success message

## Error Handling

- Input validation errors: 400 Bad Request
- Service-level validation for required fields
- Mongoose schema-level validation for field constraints

## Security Features

- Password hashing using bcrypt
- JWT-based authentication
- Password field excluded from queries
- Email uniqueness enforcement
- Input validation and sanitization

## API Examples

### Register User Request

```http
POST /user-register
Content-Type: application/json

{
  "fullname": {
    "firstname": "John",
    "lastname": "Doe"
  },
  "email": "john.doe@example.com",
  "password": "securepass123"
}
```

### Successful Response (201 Created)

```json
{
  "user": {
    "_id": "65a4f3d9c82b1a2e4c5d6e7f",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "socketId": null
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Validation Error Response (400 Bad Request)

```json
{
  "errors": [
    {
      "value": "Jo",
      "msg": "First name must be at least 3 characters long",
      "param": "fullname.firstname",
      "location": "body"
    },
    {
      "value": "invalid-email",
      "msg": "Invalid email",
      "param": "email",
      "location": "body"
    }
  ]
}
```

### Login User Request

```http
POST /user-login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "securepass123"
}
```

### Successful Response (200 OK)

```json
{
  "user": {
    "_id": "65a4f3d9c82b1a2e4c5d6e7f",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "socketId": null
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Validation Error Response (400 Bad Request)

```json
{
  "errors": [
    {
      "value": "invalid-email",
      "msg": "Invalid email",
      "param": "email",
      "location": "body"
    },
    {
      "value": "",
      "msg": "Password is required",
      "param": "password",
      "location": "body"
    }
  ]
}
```

### Get User Profile Request

```http
GET /user-profile
Authorization: Bearer <token>
```

### Successful Response (200 OK)

```json
{
  "user": {
    "_id": "65a4f3d9c82b1a2e4c5d6e7f",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "socketId": null
  }
}
```

### Logout User Request

```http
GET /user-logout
Authorization: Bearer <token>
```

### Successful Response (200 OK)

```json
{
  "message": "Logged out successfully"
}
```
