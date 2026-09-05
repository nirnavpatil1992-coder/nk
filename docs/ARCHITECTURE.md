# NK Architecture Overview

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (Browser)                          │
│              React/Next.js Web Application                   │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP/WebSocket
┌──────────────────────▼──────────────────────────────────────┐
│                    API Gateway                               │
│              (Express.js / FastAPI)                          │
└─────┬──────────────────┬──────────────────┬─────────────────┘
      │                  │                  │
      ▼                  ▼                  ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Auth       │ │   Design     │ │  Shopify     │
│   Service    │ │   Service    │ │  Service     │
└──────────────┘ └──────────────┘ └──────────────┘
      │                  │                  │
      └──────────┬───────┴────────┬─────────┘
                 │                │
┌────────────────▼─────────────────▼────────────────┐
│              Database                             │
│          (PostgreSQL / MongoDB)                   │
└─────────────────────────────────────────────────┘
      │
┌─────▼──────────────────────────────────────────┐
│   Caching Layer (Redis)                        │
│   - Session Management                         │
│   - Design Caching                             │
└────────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│        External Services                      │
├──────────────────────────────────────────────┤
│ • OpenAI API (Design Generation)              │
│ • Shopify Admin API (Store Integration)       │
│ • CDN (Asset Delivery)                        │
│ • Email Service (Notifications)               │
└──────────────────────────────────────────────┘
```

## Component Breakdown

### 1. Frontend (Client)
- **Tech Stack:** React/Next.js, TypeScript, Tailwind CSS
- **Responsibilities:**
  - User interface for design creation
  - Real-time design preview
  - Component drag-and-drop
  - User authentication
  - Project management
- **Key Components:**
  - Design Editor
  - Dashboard
  - Template Gallery
  - Settings Panel

### 2. API Gateway
- **Tech Stack:** Express.js/FastAPI
- **Responsibilities:**
  - Request routing
  - Authentication & authorization
  - Rate limiting
  - Request/response validation
  - API versioning
- **Endpoints:**
  - `/auth/*` - Authentication
  - `/designs/*` - Design CRUD operations
  - `/ai/*` - AI design generation
  - `/shopify/*` - Shopify integration
  - `/users/*` - User management

### 3. Services

#### Auth Service
- User registration and login
- JWT token management
- Shopify OAuth flow
- Session management
- Permission/role management

#### Design Service
- Design CRUD operations
- Design versioning
- Collaboration features
- Template management
- Export functionality

#### AI Service
- Prompt engineering
- OpenAI/AI Model integration
- Design generation
- Caching generated designs
- Quality assurance

#### Shopify Service
- Store connection
- Theme management
- Product sync
- Order integration
- Store data caching

### 4. Database Schema

```
Users
├── id (PK)
├── email
├── password_hash
├── shopify_store_id
├── plan (free/pro/enterprise)
└── created_at

Designs
├── id (PK)
├── user_id (FK)
├── title
├── description
├── content (JSON)
├── thumbnail_url
├── status (draft/published)
└── created_at

AIPrompts
├── id (PK)
├── design_id (FK)
├── prompt_text
├── model_used
├── tokens_used
└── created_at

ShopifyStores
├── id (PK)
├── user_id (FK)
├── store_url
├── access_token
└── connected_at

Themes
├── id (PK)
├── store_id (FK)
├── design_id (FK)
├── theme_name
├── version
└── deployed_at
```

### 5. Security Considerations

- JWT for API authentication
- HTTPS/TLS for all communications
- Rate limiting per user/IP
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- CORS configuration
- Environment variable protection
- Audit logging
- Data encryption at rest

### 6. Scalability Strategy

- **Horizontal Scaling:** Load balancing with multiple API instances
- **Caching:** Redis for session and design caching
- **Database:** Read replicas for heavy queries
- **CDN:** CloudFront/Cloudflare for static assets
- **Job Queue:** Background jobs for design generation (Bull/Celery)
- **Monitoring:** APM tools for performance tracking

### 7. Deployment Architecture

```
┌─────────────────┐
│  Git Repository │
└────────┬────────┘
         │
┌────────▼────────────────┐
│   GitHub Actions CI/CD  │
└────────┬────────────────┘
         │
    ┌────┴─────────┬──────────┐
    │              │          │
┌───▼──┐      ┌───▼──┐    ┌──▼──┐
│Build │      │ Test │    │Scan │
└───┬──┘      └───┬──┘    └──┬──┘
    │             │          │
    └─────────┬───┴──────────┘
              │
        ┌─────▼─────┐
        │  Docker   │
        │   Build   │
        └─────┬─────┘
              │
        ┌─────▼──────────────┐
        │ Push to Registry   │
        │ (Docker Hub/ECR)   │
        └─────┬──────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
┌───▼────────┐    ┌────▼───────┐
│  Staging   │    │ Production │
│  (AWS/GCP) │    │ (AWS/GCP)  │
└────────────┘    └────────────┘
```

## Data Flow Diagrams

### Design Generation Flow
```
User Request
    ↓
Validate Input
    ↓
Check Cache
    ├─→ [Cache Hit] → Return cached design
    │
└─→ [Cache Miss]
    ↓
Prepare Prompt
    ↓
Call AI API (OpenAI)
    ↓
Process Response
    ↓
Validate Design JSON
    ↓
Cache Result
    ↓
Return to Frontend
```

### Shopify Integration Flow
```
User Connects Store
    ↓
OAuth Authorization
    ↓
Save Access Token (Encrypted)
    ↓
Sync Store Data
    ├─→ Products
    ├─→ Collections
    ├─→ Store Settings
    └─→ Theme Info
    ↓
Store in Database
    ↓
Ready for Design
```

## API Versioning

- Current Version: v1
- Backwards compatibility maintained for previous versions
- Deprecation policy: 6 months notice before removal

## Monitoring & Observability

- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)
- **Metrics:** Prometheus + Grafana
- **Tracing:** Jaeger for distributed tracing
- **Alerts:** PagerDuty integration

## Tech Debt & Future Improvements

- Microservices decomposition
- GraphQL API addition
- Real-time collaboration (WebSocket)
- Mobile app development
- Advanced caching strategies
- Machine learning model optimization
