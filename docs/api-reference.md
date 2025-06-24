---
layout: default
title: API Reference
nav_order: 3
description: "Complete API documentation with examples"
---

# API Reference
{: .no_toc }

Complete reference for all API endpoints with examples and response formats.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Authentication

All API requests require authentication using an API key. Include your API key in the request headers:

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

## Base URL

All API endpoints are relative to the base URL:

```
https://api.example.com/v1
```

## Users API

### Get All Users

Retrieve a list of all users.

```http
GET /users
```

**Parameters:**

| Parameter | Type | Description |
|:----------|:-----|:------------|
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Number of results per page (default: 20, max: 100) |
| `sort` | string | Sort field (name, email, created_at) |
| `order` | string | Sort order (asc, desc) |

**Example Request:**

```bash
curl -X GET "https://api.example.com/v1/users?page=1&limit=10" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Example Response:**

```json
{
  "data": [
    {
      "id": "user_123",
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2023-06-20T10:30:00Z",
      "updated_at": "2023-06-21T14:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 150,
    "pages": 15
  }
}
```

### Get Single User

Retrieve a specific user by ID.

```http
GET /users/{id}
```

**Parameters:**

| Parameter | Type | Description |
|:----------|:-----|:------------|
| `id` | string | User ID |

**Example Request:**

```bash
curl -X GET "https://api.example.com/v1/users/user_123" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Example Response:**

```json
{
  "data": {
    "id": "user_123",
    "name": "John Doe",
    "email": "john@example.com",
    "profile": {
      "avatar": "https://example.com/avatars/john.jpg",
      "bio": "Software developer"
    },
    "created_at": "2023-06-20T10:30:00Z",
    "updated_at": "2023-06-21T14:15:00Z"
  }
}
```

### Create User

Create a new user.

```http
POST /users
```

**Request Body:**

```json
{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "password": "secure_password_123"
}
```

**Example Request:**

```bash
curl -X POST "https://api.example.com/v1/users" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jane Smith",
    "email": "jane@example.com",
    "password": "secure_password_123"
  }'
```

## Projects API

### Get All Projects

Retrieve a list of all projects.

```http
GET /projects
```

**Example Response:**

```json
{
  "data": [
    {
      "id": "proj_456",
      "name": "My Awesome Project",
      "description": "A great project description",
      "status": "active",
      "owner_id": "user_123",
      "created_at": "2023-06-20T10:30:00Z"
    }
  ]
}
```

## Error Responses

The API uses conventional HTTP response codes:

| Status Code | Description |
|:------------|:------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 400 | Bad Request - Invalid request parameters |
| 401 | Unauthorized - Invalid or missing API key |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource not found |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |

**Error Response Format:**

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request parameters are invalid",
    "details": {
      "field": "email",
      "issue": "Email address is already in use"
    }
  }
}
```

## Rate Limiting

API requests are limited to 1000 requests per hour per API key. Rate limit information is included in response headers:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1623456789
```