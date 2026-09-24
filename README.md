# E-Commerce Integration Suite — Mock Payment API

## Overview

The **Mock Payment API** is the System API responsible for simulating payment operations in the E-Commerce Integration Suite. It is a stateless mock implementation that simulates payment initiation, payment status checks, and payment reversals without connecting to any real payment gateway or database.

**Key responsibilities:**
- Simulating payment initiation and returning a transaction ID with a payment status
- Simulating payment status checks for pending transactions
- Simulating payment reversal on order cancellation

> **Note:** This is a mock API designed for development and testing purposes. It has no persistence layer and uses a round-robin mechanism to simulate real-world payment outcomes. It is not intended to be used in a production environment.

---

## Architecture

The Mock Payment API is one of two System APIs in the API-led connectivity model, sitting alongside the Database API.

```
    AI Agent (via MCP Server)
              ↓
    Experience API Layer
              ↓
      Process API Layer
              ↓
      System API Layer
    ┌──────────────────┐
    │                  │
    ▼                  ▼
Database API     Mock Payment API
                  (This Layer)
        │
        ▼
   Data Layer (MySQL Database on Aiven Cloud)
```

**Role of Mock Payment API:**
- Simulates payment processing for order placement
- Simulates payment status polling used by the Process API background scheduler
- Simulates payment reversal used during order cancellation
- Returns simulated outcomes without any real payment processing

For the complete system architecture, deployment topology, and AI integration details, refer to the [main project repository](https://github.com/Anurag180259/ecommerce_suite).

---

## Prerequisites

- **Mule Runtime**: 4.4.0 or later
- **Java**: JDK 11 or higher
- **MuleSoft Connector Packs**:
  - HTTP Connector
  - APIKit

---

## Setup & Installation

### 1. Clone the Repository

```bash
git clone https://github.com/Anurag180259/ecommerce_suite_mock_payment_api.git
cd ecommerce_suite_mock_payment_api
```

### 2. Configure Properties

Create a `configuration.properties` file in `src/main/resources/`:

1. Right-click on `src/main/resources/` folder
2. Select **New** → **File**
3. Name it `configuration.properties`
4. Copy the content from [`configuration.example.properties`](./src/main/resources/configuration.example.properties)
5. Update the values according to your environment

The following property must be configured:

```properties
# Mock Payment API Listener
http.port=PAYMENT_API_HTTP_PORT_NUMBER
```

### 3. Build the Project

In Anypoint Studio:

1. Right-click on the project in **Package Explorer**
2. Select **Run As** → **Mule Application**

The project will automatically build and deploy to the embedded Mule Runtime.

### 4. Verify Deployment

Once deployed, access the API Console:

```
http://localhost:<port>/console/
```

Replace `<port>` with your configured `http.port` value.

---

## API Endpoints

### Quick Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/payment/payments` | Initiate a payment |
| `GET` | `/payment/payments/{transactionId}/status` | Get payment status |
| `PATCH` | `/payment/payments/{transactionId}/reversePayment` | Reverse a payment |

> This API is a System API intended to be consumed only by the Process API.

---

### Initiate Payment

```
POST /payment/payments
Content-Type: application/json
```

**Request Body:**
```json
{
  "paymentToken": "mock_payment_token",
  "amount": 5299
}
```

**Response (200 OK):**
```json
{
  "transactionId": "PAY-3f1a2b4c",
  "paymentStatus": "success"
}
```

> `transactionId` is generated as `PAY-` followed by 8 hex characters from `uuid()[0 to 7]`.

**Simulated Payment Outcomes:**

The payment status is determined by a round-robin mechanism that cycles through the following outcomes in order:

| Call | `paymentStatus` |
|---|---|
| 1st | `success` |
| 2nd | `failure` |
| 3rd | `pending` |
| 4th | `success` |
| ... | continues cycling |

> When `paymentStatus` is `failure`, the Process API raises `APP:PAYMENT_FAILED` and the order is not created. When `paymentStatus` is `pending`, the order is created with a pending status and resolved later by the background scheduler.

---

### Get Payment Status

```
GET /payment/payments/{transactionId}/status
```

**Response (200 OK):**
```json
{
  "paymentStatus": "success"
}
```

**Simulated Status Outcomes:**

The payment status is determined by a round-robin mechanism that cycles through the following outcomes in order:

| Call | `paymentStatus` |
|---|---|
| 1st | `pending` |
| 2nd | `success` |
| 3rd | `pending` |
| ... | continues cycling |

> This endpoint is called by the Process API background scheduler every minute for orders that have a `pending` payment status. When `success` is returned, the order status is updated to `confirmed`.

---

### Reverse Payment

```
PATCH /payment/payments/{transactionId}/reversePayment
```

**Response (200 OK):**
```json
{
  "message": "Transaction reversed",
  "transactionId": "PAY-3f1a2b4c"
}
```

> This endpoint always returns `200 OK` regardless of whether the `transactionId` exists, since the Mock Payment API has no persistence layer. It is called by the Process API during order cancellation.

---

## Error Handling

| Status Code | Description |
|---|---|
| `200 OK` | Successful request |
| `400 Bad Request` | Invalid request format or missing required fields |
| `404 Not Found` | Resource not found |
| `405 Method Not Allowed` | HTTP method not supported |
| `406 Not Acceptable` | Content type not acceptable |
| `415 Unsupported Media Type` | Request body media type not supported |
| `501 Not Implemented` | Feature not yet implemented |

**Error Response Format:**
```json
{
  "message": "Descriptive error message"
}
```

---

## Logging

The Mock Payment API logs key events at `INFO` level:

- Incoming requests (endpoint, operation)
- Payment initiation and reversal
- Errors and exceptions

Logs are output to the Mule Runtime console and can be redirected to a file via Mule configuration.

---

## Deployment

1. Right-click on the project in **Package Explorer**
2. Select **Run As** → **Mule Application**
3. The embedded Mule Runtime will start and deploy the application
4. Access the API Console at `http://localhost:<http.port>/console/`

---

## Troubleshooting

### Payment Always Returns the Same Status
- **Note:** This is expected behavior — the Mock Payment API uses round-robin to cycle through `success`, `failure`, and `pending`. Restart the application to reset the round-robin counter.

### Payment Status Always Pending
- **Note:** This is expected behavior for the status check endpoint — it cycles between `pending` and `success`. The Process API background scheduler resolves pending payments automatically within one minute.

---

## Related Documentation

- **RAML Specification:** [`ecommercesuitemockpaymentapi.raml`](./src/main/resources/api)
- **API Console:** Available at `/console/` path after deployment
- **Main Project Repository:** [ecommerce_suite](https://github.com/Anurag180259/ecommerce_suite) — Contains overall architecture, deployment guide, and project scope

---

## Support

For issues, questions, or contributions, please refer to the main project repository.

---

**Last Updated:** September 2026
**Version:** 1.0
**Maintained by:** Anurag Ninave
