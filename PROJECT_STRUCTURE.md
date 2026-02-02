# MuleSoft API-Led Connectivity - Project Structure

## 📁 Cấu trúc dự án theo Best Practices

```
mulesoft-apiledsample/
├── src/main/mule/
│   ├── common/                          # 🔧 Shared components
│   │   ├── global-config.xml           # Global error handlers, config properties
│   │   └── salesforce-config.xml       # Salesforce OAuth & REST API configs
│   │
│   ├── system/                          # 🔌 System APIs (Layer 3 - Bottom)
│   │   └── salesforce-system-api.xml   # Direct Salesforce integration (CRUD)
│   │
│   ├── process/                         # ⚙️ Process APIs (Layer 2 - Middle)
│   │   ├── customer-orders-process-api.xml      # Customer order orchestration
│   │   └── order-fulfillment-process-api.xml    # Order fulfillment workflow
│   │
│   └── experience/                      # 🌐 Experience APIs (Layer 1 - Top)
│       ├── web-experience-api.xml              # Web browser optimized
│       ├── mobile-experience-api.xml           # Mobile app optimized
│       └── customer-service-experience-api.xml # CS representative optimized
│
├── src/main/resources/
│   ├── config.yaml                      # Main configuration
│   └── config-dev.yaml                  # Development environment config
│
├── pom.xml                              # Maven dependencies
├── README.md                            # Project documentation
├── SALESFORCE_CONNECTED_APP_SETUP.md   # Salesforce setup guide
└── MuleSoft-API-Led-Sample.postman_collection.json  # API tests

```

## 🎯 API-Led Connectivity Architecture

### Layer 1: Experience APIs (User-facing)
**Ports:** 8084-8086  
**Purpose:** Channel-specific optimization  
**Files:** `experience/` folder

- **Web Experience API** (`web-experience-api.xml`)
  - Port: 8084
  - Returns detailed, user-friendly responses
  - Optimized for web browsers

- **Mobile Experience API** (`mobile-experience-api.xml`)
  - Port: 8085
  - Returns compact JSON for low bandwidth
  - Optimized for mobile apps

- **Customer Service Experience API** (`customer-service-experience-api.xml`)
  - Port: 8086
  - Advanced workflows and detailed customer info
  - Optimized for CS representatives

### Layer 2: Process APIs (Business Logic)
**Ports:** 8082-8083  
**Purpose:** Orchestration and business rules  
**Files:** `process/` folder  
**⚠️ Internal only - Called by Experience APIs**

- **Customer Orders Process API** (`customer-orders-process-api.xml`)
  - Port: 8082
  - Orchestrates customer creation and order processing
  - Calculates order totals and validates business rules

- **Order Fulfillment Process API** (`order-fulfillment-process-api.xml`)
  - Port: 8083
  - Handles order fulfillment workflow
  - Generates tracking numbers and shipping info

### Layer 3: System APIs (Data Integration)
**Port:** 8081  
**Purpose:** Direct system integration  
**Files:** `system/` folder  
**⚠️ Internal only - Called by Process APIs**

- **Salesforce System API** (`salesforce-system-api.xml`)
  - Port: 8081
  - CRUD operations on Salesforce (Account, Contact)
  - OAuth 2.0 authentication via Connected App
  - REST API integration

### Shared Components
**Files:** `common/` folder  
**Purpose:** Reusable configurations

- **Global Config** (`global-config.xml`)
  - Configuration properties
  - Global error handler
  - Correlation ID tracking

- **Salesforce Config** (`salesforce-config.xml`)
  - OAuth 2.0 token management
  - Salesforce REST API connection configs
  - Reusable sub-flow for token acquisition

## 🔒 Security & Access Control

### Production Deployment
```
Internet/Users
     ↓
Experience APIs (8084-8086) ← PUBLIC (with API Gateway/Auth)
     ↓
Process APIs (8082-8083)    ← INTERNAL (VPC/Private Network)
     ↓
System APIs (8081)          ← INTERNAL (VPC/Private Network)
     ↓
Salesforce                  ← OAuth 2.0 via Connected App
```

### Development/Testing
All ports exposed on localhost for testing individual layers.

## 📊 Benefits of This Structure

### 1. **Separation of Concerns**
- Each file has a single, clear responsibility
- Easy to understand and maintain
- Follows SOLID principles

### 2. **Reusability**
- System APIs can serve multiple Process APIs
- Process APIs can serve multiple Experience APIs
- Shared configs in `common/` folder

### 3. **Independent Scaling**
- Each API layer can scale independently
- High-traffic Experience APIs can have more instances
- Process/System APIs scale based on backend load

### 4. **Team Productivity**
- Multiple developers can work on different layers simultaneously
- Clear boundaries reduce merge conflicts
- Easy to assign ownership (Web team, Mobile team, etc.)

### 5. **Testing & Debugging**
- Test each layer independently
- Mock downstream dependencies easily
- Trace requests via correlation IDs across layers

### 6. **Change Management**
- Update mobile response format without touching web
- Change Salesforce integration without affecting channels
- Add new experience channels without modifying process layer

## 🚀 How to Run

1. **Import project** into Anypoint Studio
2. **Configure Salesforce** credentials in `config.yaml`
3. **Run application** - All XML files will be loaded automatically
4. **Test** using Postman collection (only call Experience APIs)

## 📝 Naming Conventions

### Files
- `*-system-api.xml` - System layer APIs
- `*-process-api.xml` - Process layer APIs
- `*-experience-api.xml` - Experience layer APIs
- `*-config.xml` - Shared configurations

### Flows
- `system-api-*-flow` - System API flows
- `process-api-*-flow` - Process API flows
- `experience-api-*-flow` - Experience API flows

### HTTP Configs
- `System_*_HTTP_Listener_config` - System API listeners
- `System_*_HTTP_Request_config` - System API requests
- `Process_*_HTTP_Listener_config` - Process API listeners
- `Process_*_HTTP_Request_config` - Process API requests
- `Experience_*_HTTP_Listener_config` - Experience API listeners

## 🎓 Best Practices Applied

✅ Separation of concerns (each file = one API)  
✅ Shared configurations in common folder  
✅ Clear layer boundaries (System → Process → Experience)  
✅ Correlation ID tracking across all layers  
✅ Global error handling  
✅ Channel-specific optimizations  
✅ OAuth 2.0 token management  
✅ RESTful API design  
✅ Proper HTTP status codes  
✅ Comprehensive logging  

## 📚 Further Reading

- [MuleSoft API-Led Connectivity](https://www.mulesoft.com/resources/api/api-led-connectivity)
- [MuleSoft Best Practices](https://docs.mulesoft.com/general/getting-started/api-lifecycle-overview)
- [Anypoint Platform](https://www.mulesoft.com/platform/enterprise-integration)
