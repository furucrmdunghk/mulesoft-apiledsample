# API-Led Connectivity Sample - MuleSoft

Sample project demonstrating MuleSoft's **API-Led Connectivity** architecture with 3 API layers:
- **Experience APIs** (Experience Layer) - RAML validation, APIKit router
- **Process APIs** (Process Layer) - Business validation
- **System APIs** (System Layer) - Salesforce integration

## ✨ Key Features (Updated Feb 2026)

### 🎯 API Validation Strategy
- **RAML Validation** at Experience APIs (Web & Mobile)
  - Contract validation: required fields, data types, constraints
  - APIKit router pattern with main flow + implementation flow
  - Comprehensive error handlers for validation errors
  
- **Business Validation** at Process API
  - Cross-field calculations (RAML cannot validate)
  - Example: Minimum order value $100
  - Centralized business rules for all channels

### 🛡️ Error Handling Pattern
- **Experience APIs**: 
  - `APIKIT:BAD_REQUEST` - RAML validation failed
  - `HTTP:BAD_REQUEST` - Downstream API errors (400)
  - `APIKIT:NOT_FOUND`, `METHOD_NOT_ALLOWED`, `NOT_ACCEPTABLE`, `UNSUPPORTED_MEDIA_TYPE`
  - `ANY` - Fallback for unexpected errors
  
- **Process APIs**: 
  - Business validation errors with descriptive messages
  - Propagate errors to Experience APIs

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  EXPERIENCE APIs (Layer 1)                   │
│              [RAML Validation + APIKit Router]               │
│                                                               │
│  ┌──────────────────────┐    ┌──────────────────────┐      │
│  │   Web Experience     │    │  Mobile Experience   │      │
│  │      :8084           │    │       :8085          │      │
│  │ • Full response      │    │ • Minimal payload    │      │
│  │ • Rich UI data       │    │ • Optimized for 3G   │      │
│  └──────────────────────┘    └──────────────────────┘      │
└────────────────────────┬─────────────────────────────────────┘
                         │ REUSE
┌────────────────────────┴─────────────────────────────────────┐
│                     PROCESS API (Layer 2)                     │
│                 [Business Validation]                         │
│                                                               │
│  ┌───────────────────────────────────────────────────┐      │
│  │   Customer Orders Process API (:8082)             │      │
│  │   • Minimum order value validation ($100)         │      │
│  │   • Customer creation/lookup orchestration        │      │
│  │   • Order calculation & Salesforce integration    │      │
│  └───────────────────────────────────────────────────┘      │
└────────────────────────┬─────────────────────────────────────┘
                         │ REUSE
┌────────────────────────┴─────────────────────────────────────┐
│                      SYSTEM API (Layer 3)                     │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │   Customer System API (:8081)                         │  │
│  │   • POST /customers (Create Account + Contacts)       │  │
│  │   • GET /customers/{id} (Get Account + Contacts)      │  │
│  │   • POST /orders (Create Order in Salesforce)         │  │
│  └───────────────────────────────────────────────────────┘  │
│                           │                                   │
│                           ▼                                   │
│                   ┌──────────────┐                           │
│                   │  Salesforce  │                           │
│                   └──────────────┘                           │
└───────────────────────────────────────────────────────────────┘
```

## API Details

### 🔷 SYSTEM API (Port 8081)

#### Customer System API
Direct access to Salesforce for managing Accounts, Contacts and Orders.

**1. Create Customer**
```
POST http://localhost:8081/customers
```

**Request Body**:
```json
{
  "customerName": "ACME Corporation",
  "phone": "+1-555-0123",
  "website": "https://acme.com",
  "billingAddress": {
    "street": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "postalCode": "94102",
    "country": "USA"
  },
  "contacts": [
    {
      "firstName": "John",
      "lastName": "Doe",
      "email": "john.doe@acme.com",
      "phone": "+1-555-0124"
    }
  ]
}
```

**2. Get Customer**
```
GET http://localhost:8081/customers/{accountId}
```

**3. Create Order**
```
POST http://localhost:8081/orders
```

---

### 🔶 PROCESS API

#### Customer Orders Process API (Port 8082)
Orchestrates business logic for order processing.

**Endpoint**: `POST http://localhost:8082/customer-orders`

**Business Rules**:
- ✅ Minimum order value: $100
- ✅ Automatic customer creation if not exists
- ✅ Order total calculation: sum(quantity × unitPrice)
- ✅ Salesforce Account & Order creation

**Request Body**:
```json
{
  "customer": {
    "customerName": "John Doe",
    "phone": "+1-555-9999"
  },
  "orderItems": [
    {
      "productName": "iPhone 15",
      "quantity": 1,
      "unitPrice": 999
    }
  ]
}
```

**Response (Success - 201)**:
```json
{
  "success": true,
  "customerId": "0015j00000AhORwAAN",
  "orderId": "ORD-123456",
  "orderData": {
    "customerId": "0015j00000AhORwAAN",
    "totalAmount": 999,
    "status": "NEW",
    "items": [...]
  },
  "message": "Customer order processed successfully",
  "correlationId": "abc-123-def"
}
```

**Response (Validation Failed - 400)**:
```json
{
  "success": false,
  "error": "Order validation failed: Minimum order value is 100",
  "orderTotal": 50,
  "correlationId": "abc-123-def"
}
```

---

### 🔵 EXPERIENCE APIs

#### 1. Web Experience API (Port 8084)

**Features**:
- ✅ RAML validation (required fields, data types, constraints)
- ✅ APIKit router with comprehensive error handling
- 🎨 Rich response format with detailed information
- 🖥️ Optimized for web browsers (WiFi/LAN)

**Endpoint**: `POST http://localhost:8084/api/orders`

**Request Body**:
```json
{
  "customer": {
    "customerName": "Jane Smith",
    "phone": "+1-555-8888"
  },
  "items": [
    {
      "productName": "Laptop",
      "quantity": 2,
      "unitPrice": 1500
    }
  ]
}
```

**Response (Success - 201)**:
```json
{
  "orderId": "ORD-123456",
  "status": "success",
  "message": "Your order has been placed successfully",
  "orderSummary": {
    "orderId": "ORD-123456",
    "customerId": "0015j00000AhORwAAN",
    "totalAmount": 3000,
    "orderDate": "2026-02-02T18:00:00",
    "estimatedDelivery": "2026-02-09T18:00:00"
  }
}
```

**Response (RAML Validation Failed - 400)**:
```json
{
  "error": {
    "code": 400,
    "type": "BAD_REQUEST",
    "message": "Validation failed",
    "details": "Invalid request format or missing required fields"
  }
}
```

**Response (Business Validation Failed - 400)**:
```json
{
  "error": {
    "code": 400,
    "type": "BAD_REQUEST",
    "message": "Business validation failed",
    "details": "Order validation failed: Minimum order value is 100"
  }
}
```

#### 2. Mobile Experience API (Port 8085)

**Features**:
- ✅ RAML validation (same rules as Web)
- ✅ APIKit router pattern
- 📱 Minimal payload (~70% smaller than Web)
- ⚡ Optimized for mobile networks (3G/4G/5G)
- 🔋 Battery & bandwidth efficient

**Endpoint**: `POST http://localhost:8085/api/orders`

**Request Body** (same as Web):
```json
{
  "customer": {
    "customerName": "Mobile User",
    "phone": "+1-555-7777"
  },
  "items": [
    {
      "productName": "Headphones",
      "quantity": 3,
      "unitPrice": 50
    }
  ]
}
```

**Response (Success - 201)**:
```json
{
  "id": "ORD-123456",
  "status": "OK",
  "total": 150,
  "eta": "2026-02-09"
}
```

**Response (Validation Failed - 400)**:
```json
{
  "error": {
    "code": 400,
    "type": "BAD_REQUEST",
    "message": "Validation failed"
  }
}
```

---

## 🧪 Testing Guide

### Test Scenarios

#### ✅ **Happy Path - Valid Order**
Request (Web or Mobile):
```json
{
  "customer": {
    "customerName": "Test User",
    "phone": "555-1234"
  },
  "items": [
    {
      "productName": "Product A",
      "quantity": 2,
      "unitPrice": 75
    }
  ]
}
```
Expected: **201 Created** (Total = $150 >= $100)

---

#### ❌ **RAML Validation Failed**

**Test 1: Missing required field**
```json
{
  "customer": {
    "phone": "555-1234"
  },
  "items": [...]
}
```
Expected: **400 Bad Request** - "Validation failed"

**Test 2: Empty body**
```json
{}
```
Expected: **400 Bad Request** - "Invalid request format or missing required fields"

---

#### ❌ **Business Validation Failed**

**Test: Order < $100**
```json
{
  "customer": {
    "customerName": "Test User",
    "phone": "555-1234"
  },
  "items": [
    {
      "productName": "Cheap Item",
      "quantity": 1,
      "unitPrice": 50
    }
  ]
}
```
Expected: **400 Bad Request** - "Order validation failed: Minimum order value is 100"

---

## Benefits of API-Led Architecture

### 1. ♻️ **Reusability**
- System APIs can be called by multiple Process APIs
- Process APIs can be called by multiple Experience APIs
- Reduces duplicate code and effort

### 2. 🔌 **Loose Coupling**
- Each layer is independent
- Changes in System API implementation don't affect Experience API
- Easy to replace backend systems

### 3. 🎯 **Separation of Concerns**
- **System APIs**: Only handles connection to backend systems (Salesforce)
- **Process APIs**: Handles business logic and orchestration
- **Experience APIs**: Optimized for each channel (Web, Mobile, etc.)

### 4. 📱 **Channel-Specific Optimization**
- Web API: Full response, human-readable
- Mobile API: Compact response, bandwidth efficient
- Each channel gets optimal data format

### 5. 🔒 **Security Layers**
- Can apply different security policies per layer
- Rate limiting and throttling per API

### 6. 📊 **Monitoring & Analytics**
- Track usage by layer
- Easier to identify bottlenecks
- Business metrics at Process layer

---

## Configuration

### 1. Setup Salesforce Connected App
**⚠️ REQUIRED**: You need to create a Connected App in Salesforce to get OAuth 2.0 credentials.

👉 **Details**: See file [SALESFORCE_CONNECTED_APP_SETUP.md](SALESFORCE_CONNECTED_APP_SETUP.md)

Summary steps:
1. Salesforce Setup → App Manager → New Connected App
2. Enable OAuth Settings + Client Credentials Flow
3. Select "Run As User" with API permissions
4. Copy Consumer Key and Consumer Secret
5. Update config files

### 2. Update Configuration Files

Update file [src/main/resources/config.yaml](src/main/resources/config.yaml):

```yaml
salesforce:
  # From Connected App
  clientId: "your-connected-app-consumer-key"
  clientSecret: "your-connected-app-consumer-secret"
  
  # Your Salesforce instance URL (get from browser when logged in)
  instanceUrl: "https://your-instance.salesforce.com"
  tokenUrl: "https://login.salesforce.com/services/oauth2/token"
  apiVersion: "v60.0"
```

### 3. Environment-specific Config
For development/sandbox, update `config-dev.yaml` with sandbox credentials.

---

## Technical Architecture

### Authentication with Salesforce
- **OAuth 2.0 Client Credentials Flow** (no username/password needed)
- Connected App with Consumer Key/Secret
- Access token automatically refreshed by Mule OAuth Module
- Secure, no password stored in code

### REST API Endpoints
System APIs directly call **Salesforce REST API**:
- `POST /services/data/v60.0/sobjects/Account` - Create Account
- `POST /services/data/v60.0/sobjects/Contact` - Create Contact
- `GET /services/data/v60.0/sobjects/Account/{id}` - Get Account
- `GET /services/data/v60.0/query?q=SELECT...` - SOQL Query

---

## Testing Flow

### Step 1: Test System API
```bash
# Create Customer
curl -X POST http://localhost:8081/customers \
  -H "Content-Type: application/json" \
  -d '{
    "customerName": "Test Company",
    "phone": "+1-555-1234",
    "contacts": [{
      "firstName": "John",
      "lastName": "Test",
      "email": "john@test.com"
    }]
  }'

# Response will return accountId, save it for next tests
```

### Step 2: Test Process API
```bash
# Create Order (with existing customer)
curl -X POST http://localhost:8082/customer-orders \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "customerId": "<accountId-from-step-1>"
    },
    "orderItems": [{
      "productName": "Product A",
      "quantity": 2,
      "unitPrice": 100.00
    }]
  }'
```

### Step 3: Test Experience API
```bash
# Web Order
curl -X POST http://localhost:8084/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "customerId": "<accountId-from-step-1>"
    },
    "items": [{
      "productName": "Laptop",
      "quantity": 1,
      "unitPrice": 1500.00
    }]
  }'

# Mobile Order (same request, different response format)
curl -X POST http://localhost:8085/api/orders \
  -H "Content-Type: application/json" \
  -d '<same-body-as-web>'
```

---

## Error Handling

All APIs have global error handler with format:
```json
{
  "error": {
    "type": "ERROR_TYPE",
    "message": "Error description",
    "timestamp": "2026-02-02T10:00:00Z",
    "correlationId": "uuid-here"
  }
}
```

## Correlation ID

Each request is tracked by `x-correlation-id` header:
- Auto-generated if client doesn't send
- Passed through all API layers
- Helps trace request end-to-end

---

## Dependencies

- **Mule Runtime**: 4.10.1
- **HTTP Connector**: 1.11.0 (Free)
- **OAuth Module**: 1.1.18 (Free)
- **APIKit Module**: 1.10.0 (Free)
- **No premium connectors required!** ✅

---

## Advantages of This Approach

### ✅ **Using Salesforce REST API**
- No need for Salesforce Connector (premium)
- Only uses HTTP Connector + OAuth Module (free)
- No license issues
- Flexible and modern

### ✅ **OAuth 2.0 Client Credentials**
- Best practice authentication
- Secure - no password stored
- Auto token refresh
- Production-ready

---

## Notes

⚠️ **Not yet implemented**: Flow to create Order__c (custom object) in Salesforce because custom object has not been created in Salesforce org.

When custom object Order__c is ready, can add System API similar to Customer API.

---

## Contact

Sample project for demonstrating MuleSoft's API-Led Connectivity architecture.
