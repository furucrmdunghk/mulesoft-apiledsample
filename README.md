# API-Led Connectivity Sample - MuleSoft

Dự án mẫu thể hiện kiến trúc **API-Led Connectivity** của MuleSoft với 3 tầng API:
- **Experience APIs** (Tầng trải nghiệm)
- **Process APIs** (Tầng xử lý nghiệp vụ)
- **System APIs** (Tầng hệ thống)

## Kiến trúc

```
┌─────────────────────────────────────────────────────────────────┐
│                     EXPERIENCE APIs (Layer 1)                    │
│                                                                   │
│  ┌──────────┐    ┌──────────┐    ┌───────────────────────┐     │
│  │   Web    │    │  Mobile  │    │  Customer Service     │     │
│  │  :8084   │    │  :8085   │    │      :8086            │     │
│  └──────────┘    └──────────┘    └───────────────────────┘     │
└────────────────────────┬──────────────────────────────────────┘
                         │ REUSE
┌────────────────────────┴──────────────────────────────────────┐
│                      PROCESS APIs (Layer 2)                    │
│                                                                 │
│  ┌───────────────────────┐    ┌─────────────────────────┐    │
│  │   Customer Orders     │    │  Order Fulfillment      │    │
│  │       :8082           │    │       :8083             │    │
│  └───────────────────────┘    └─────────────────────────┘    │
└────────────────────────┬──────────────────────────────────────┘
                         │ REUSE
┌────────────────────────┴──────────────────────────────────────┐
│                       SYSTEM APIs (Layer 3)                    │
│                                                                 │
│  ┌───────────────────────────────────────────────────────┐    │
│  │   Customer & Contacts API (:8081)                     │    │
│  │   - POST /customers (Create Account + Contacts)       │    │
│  │   - GET /customers/{id} (Get Account with Contacts)   │    │
│  └───────────────────────────────────────────────────────┘    │
│                           │                                     │
│                           ▼                                     │
│                   ┌──────────────┐                             │
│                   │  Salesforce  │                             │
│                   └──────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

## APIs Chi tiết

### 🔷 SYSTEM APIs (Port 8081)

#### 1. Create Customer and Contacts
- **Endpoint**: `POST http://localhost:8081/api/system/customer/customers`
- **Mô tả**: Tạo Account mới và Contacts trong Salesforce
- **Request Body**:
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

#### 2. Get Customer Details
- **Endpoint**: `GET http://localhost:8081/api/system/customer/customers/{customerId}`
- **Mô tả**: Lấy thông tin Account và Contacts từ Salesforce

---

### 🔶 PROCESS APIs

#### 1. Customer Orders (Port 8082)
- **Endpoint**: `POST http://localhost:8082/api/process/customer-orders/customer-orders`
- **Mô tả**: Orchestration layer - xử lý logic nghiệp vụ cho đơn hàng
- **Chức năng**:
  - Validate order data
  - Tạo hoặc lấy customer từ System API
  - Tính toán tổng tiền đơn hàng
  - Xử lý business rules

**Request Body**:
```json
{
  "customer": {
    "customerId": "0015g00000XYZ123",
    "customerName": "ACME Corporation"
  },
  "orderItems": [
    {
      "productName": "Product A",
      "quantity": 2,
      "unitPrice": 100.00
    },
    {
      "productName": "Product B",
      "quantity": 1,
      "unitPrice": 250.00
    }
  ]
}
```

#### 2. Order Fulfillment (Port 8083)
- **Endpoint**: `POST http://localhost:8083/api/process/order-fulfillment/fulfillment`
- **Mô tả**: Xử lý quy trình fulfill order
- **Chức năng**:
  - Lấy thông tin customer
  - Chuẩn bị shipping information
  - Generate tracking number
  - Cập nhật trạng thái đơn hàng

---

### 🔵 EXPERIENCE APIs

#### 1. Web API (Port 8084)
- **Endpoint**: `POST http://localhost:8084/api/web/orders`
- **Mô tả**: API tối ưu cho Customer Website
- **Format**: Response đầy đủ thông tin, dễ đọc cho Web UI

**Request Body**:
```json
{
  "customer": {
    "customerName": "New Customer",
    "phone": "+1-555-9999",
    "contacts": [
      {
        "firstName": "Jane",
        "lastName": "Smith",
        "email": "jane@example.com"
      }
    ]
  },
  "items": [
    {
      "productName": "Laptop",
      "quantity": 1,
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
