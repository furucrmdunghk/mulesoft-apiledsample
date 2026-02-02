# API-Led Connectivity Sample - MuleSoft

Dự án mẫu thể hiện kiến trúc **API-Led Connectivity** của MuleSoft với 3 tầng API:
- **Experience APIs** (Tầng trải nghiệm) - RAML validation, APIKit router
- **Process APIs** (Tầng xử lý nghiệp vụ) - Business validation
- **System APIs** (Tầng hệ thống) - Salesforce integration

## ✨ Tính năng chính (Updated Feb 2026)

### 🎯 API Validation Strategy
- **RAML Validation** ở Experience APIs (Web & Mobile)
  - Contract validation: required fields, data types, constraints
  - APIKit router pattern với main flow + implementation flow
  - Comprehensive error handlers cho validation errors
  
- **Business Validation** ở Process API
  - Cross-field calculations (RAML cannot validate)
  - Example: Minimum order value $100
  - Centralized business rules cho tất cả channels

### 🛡️ Error Handling Pattern
- **Experience APIs**: 
  - `APIKIT:BAD_REQUEST` - RAML validation failed
  - `HTTP:BAD_REQUEST` - Downstream API errors (400)
  - `APIKIT:NOT_FOUND`, `METHOD_NOT_ALLOWED`, `NOT_ACCEPTABLE`, `UNSUPPORTED_MEDIA_TYPE`
  - `ANY` - Fallback for unexpected errors
  
- **Process APIs**: 
  - Business validation errors với descriptive messages
  - Propagate errors lên Experience APIs

## Kiến trúc

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

## APIs Chi tiết

### 🔷 SYSTEM API (Port 8081)

#### Customer System API
Truy cập trực tiếp vào Salesforce để quản lý Accounts, Contacts và Orders.

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
Orchestrates business logic cho order processing.

**Endpoint**: `POST http://localhost:8082/customer-orders`

**Business Rules**:
- ✅ Minimum order value: $100
- ✅ Automatic customer creation nếu chưa tồn tại
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

**Đặc điểm**:
- ✅ RAML validation (required fields, data types, constraints)
- ✅ APIKit router với comprehensive error handling
- 🎨 Rich response format với detailed information
- 🖥️ Optimized cho web browsers (WiFi/LAN)

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

**Đặc điểm**:
- ✅ RAML validation (same rules as Web)
- ✅ APIKit router pattern
- 📱 Minimal payload (~70% smaller than Web)
- ⚡ Optimized cho mobile networks (3G/4G/5G)
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

## 🚀 Run Project
      "unitPrice": 1500.00
    }
  ]
}
```

#### 2. Mobile API (Port 8085)
- **Endpoint**: `POST http://localhost:8085/api/mobile/orders`
- **Mô tả**: API tối ưu cho Mobile App
- **Format**: Response ngắn gọn, tiết kiệm bandwidth

#### 3. Customer Service API (Port 8086)
- **Endpoint 1**: `GET http://localhost:8086/api/customer-service/customer/{customerId}`
  - Lấy thông tin customer đầy đủ cho Customer Service
  
- **Endpoint 2**: `POST http://localhost:8086/api/customer-service/process-order`
  - Xử lý đơn hàng kèm fulfillment (aggregated flow)
  - Gọi cả Customer Orders API và Order Fulfillment API

**Request Body**:
```json
{
  "customer": {
    "customerId": "0015g00000XYZ123"
  },
  "items": [
    {
      "productName": "Service Package",
      "quantity": 1,
      "unitPrice": 500.00
    }
  ]
}
```

---

## Ưu điểm của kiến trúc API-Led

### 1. ♻️ **Reusability (Tái sử dụng)**
- System APIs có thể được gọi bởi nhiều Process APIs
- Process APIs có thể được gọi bởi nhiều Experience APIs
- Giảm duplicate code và effort

### 2. 🔌 **Loose Coupling**
- Mỗi tầng độc lập với nhau
- Thay đổi implementation ở System API không ảnh hưởng Experience API
- Dễ dàng thay thế backend systems

### 3. 🎯 **Separation of Concerns**
- **System APIs**: Chỉ lo kết nối với hệ thống backend (Salesforce)
- **Process APIs**: Xử lý business logic, orchestration
- **Experience APIs**: Tối ưu cho từng channel (Web, Mobile, CS)

### 4. 📱 **Channel-Specific Optimization**
- Web API: Response đầy đủ, human-readable
- Mobile API: Response compact, tiết kiệm bandwidth
- Customer Service API: Aggregated data, rich information

### 5. 🔒 **Security Layers**
- Có thể apply security policies riêng cho từng tầng
- Rate limiting, throttling theo từng API

### 6. 📊 **Monitoring & Analytics**
- Track usage theo từng tầng
- Identify bottlenecks dễ dàng hơn
- Business metrics ở Process layer

---

## Cấu hình

### 1. Setup Salesforce Connected App
**⚠️ BẮT BUỘC**: Bạn cần tạo Connected App trong Salesforce để lấy OAuth 2.0 credentials.

👉 **Chi tiết**: Xem file [SALESFORCE_CONNECTED_APP_SETUP.md](SALESFORCE_CONNECTED_APP_SETUP.md)

Tóm tắt steps:
1. Salesforce Setup → App Manager → New Connected App
2. Enable OAuth Settings + Client Credentials Flow
3. Chọn "Run As User" có quyền API
4. Copy Consumer Key và Consumer Secret
5. Update vào config files

### 2. Update Configuration Files

Cập nhật file [src/main/resources/config.yaml](src/main/resources/config.yaml):

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
Cho development/sandbox, update `config-dev.yaml` với sandbox credentials.

---

## Kiến trúc kỹ thuật

### Authentication với Salesforce
- **OAuth 2.0 Client Credentials Flow** (không cần username/password)
- Connected App với Consumer Key/Secret
- Access token tự động refresh bởi Mule OAuth Module
- Secure, không lưu password trong code

### REST API Endpoints
System APIs gọi trực tiếp **Salesforce REST API**:
- `POST /services/data/v60.0/sobjects/Account` - Tạo Account
- `POST /services/data/v60.0/sobjects/Contact` - Tạo Contact
- `GET /services/data/v60.0/sobjects/Account/{id}` - Get Account
- `GET /services/data/v60.0/query?q=SELECT...` - SOQL Query

---

## Testing Flow

### Bước 1: Test System API
```bash
# Create Customer
curl -X POST http://localhost:8081/api/system/customer/customers \
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

# Response sẽ trả về accountId, lưu lại để test tiếp
```

### Bước 2: Test Process API
```bash
# Create Order (với existing customer)
curl -X POST http://localhost:8082/api/process/customer-orders/customer-orders \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "customerId": "<accountId-từ-bước-1>"
    },
    "orderItems": [{
      "productName": "Product A",
      "quantity": 2,
      "unitPrice": 100.00
    }]
  }'
```

### Bước 3: Test Experience API
```bash
# Web Order
curl -X POST http://localhost:8084/api/web/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "customerId": "<accountId-từ-bước-1>"
    },
    "items": [{
      "productName": "Laptop",
      "quantity": 1,
      "unitPrice": 1500.00
    }]
  }'

# Mobile Order (same request, different response format)
curl -X POST http://localhost:8085/api/mobile/orders \
  -H "Content-Type: application/json" \
  -d '<same-body-as-web>'
```

---

## Error Handling

Tất cả APIs đều có global error handler với format:
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

Mỗi request được track bởi `x-correlation-id` header:
- Tự động generate nếu client không gửi
- Được truyền qua tất cả các tầng API
- Giúp trace request end-to-end

---

## Dependencies

- **Mule Runtime**: 4.10.1
- **HTTP Connector**: 1.11.0 (Free)
- **OAuth Module**: 1.1.18 (Free)
- **APIKit Module**: 1.10.0 (Free)
- **No premium connectors required!** ✅

---

## Ưu điểm của approach này

### ✅ **Sử dụng Salesforce REST API**
- Không cần Salesforce Connector (premium)
- Chỉ dùng HTTP Connector + OAuth Module (free)
- No license issues
- Flexible và modern

### ✅ **OAuth 2.0 Client Credentials**
- Best practice authentication
- Secure - không lưu password
- Auto token refresh
- Production-ready

---

## Ghi chú

⚠️ **Chưa implement**: Flow tạo Order__c (custom object) trong Salesforce do custom object chưa được tạo trong Salesforce org.

Khi custom object Order__c đã sẵn sàng, có thể thêm System API tương tự như Customer API.

---

## Liên hệ

Dự án mẫu cho mục đích demo kiến trúc API-Led Connectivity của MuleSoft.
