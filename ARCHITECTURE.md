# ELEVÉTION Platform - System Architecture

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Internet / Users                          │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   CDN / Load Balancer   │
                    │  (DigitalOcean/Nginx)   │
                    └────────────┬────────────┘
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
        ┌───────▼────────┐  ┌───▼──────────┐  ┌─▼──────────────┐
        │ Frontend       │  │ Backend      │  │ Static Assets  │
        │ (Next.js)      │  │ (FastAPI)    │  │ (S3/CDN)       │
        │ Port: 3100     │  │ Port: 8500   │  │                │
        └────────────────┘  └────┬─────────┘  └────────────────┘
                                 │
                        ┌────────▼──────────┐
                        │   PostgreSQL 15   │
                        │   Port: 5432      │
                        │ (Connection Pool) │
                        └───────────────────┘
```

---

## 🧬 Frontend Architecture (Next.js 14)

### **Application Structure**

```
frontend/
├── src/
│   ├── app/                          # Next.js app directory (13+)
│   │   ├── (DashboardLayout)/       # Protected routes wrapper
│   │   │   ├── page.tsx             # Main clinical dashboard
│   │   │   ├── layout.tsx           # Dashboard layout wrapper
│   │   │   │
│   │   │   ├── health-hub/
│   │   │   │   └── page.tsx         # Multi-omic health center
│   │   │   │
│   │   │   ├── patients/
│   │   │   │   ├── page.tsx         # Patient list
│   │   │   │   ├── [id]/
│   │   │   │   │   └── page.tsx     # Patient detail
│   │   │   │   └── new/
│   │   │   │       └── page.tsx     # New patient form
│   │   │   │
│   │   │   ├── genetic-reports/
│   │   │   │   └── page.tsx         # Report viewer
│   │   │   │
│   │   │   ├── analytics/
│   │   │   │   └── page.tsx         # System analytics (admin)
│   │   │   │
│   │   │   ├── layout/              # Layout components
│   │   │   │   ├── Sidebar.tsx      # Navigation drawer
│   │   │   │   ├── Header.tsx       # Top app bar
│   │   │   │   ├── MobileBottomNav.tsx  # Mobile tab bar
│   │   │   │   └── header/
│   │   │   │       ├── Profile.tsx
│   │   │   │       └── Notification.tsx
│   │   │   │
│   │   │   └── components/
│   │   │       ├── container/
│   │   │       ├── charts/
│   │   │       ├── forms/
│   │   │       └── dialogs/
│   │   │
│   │   ├── (PublicLayout)/          # Public pages wrapper
│   │   │   └── page.tsx             # Landing page (future)
│   │   │
│   │   └── authentication/
│   │       ├── login/
│   │       │   └── page.tsx
│   │       └── register/
│   │           └── page.tsx
│   │
│   ├── components/                  # Reusable UI components
│   │   ├── ui/
│   │   │   ├── CoreSpinLoader.tsx
│   │   │   ├── OnboardingTips.tsx
│   │   │   └── ...
│   │   ├── charts/
│   │   │   ├── HealthScoreChart.tsx
│   │   │   ├── RiskDistributionChart.tsx
│   │   │   └── ...
│   │   ├── forms/
│   │   ├── dialogs/
│   │   └── ...
│   │
│   ├── utils/                       # Utility functions
│   │   ├── authStore.ts             # Auth + role manager
│   │   ├── patientStore.ts          # Patient data store
│   │   ├── healthScore.ts           # Score calculation
│   │   ├── api.ts                   # API client
│   │   └── constants.ts
│   │
│   ├── styles/
│   │   └── globals.css
│   │
│   └── types/                       # TypeScript types
│       ├── patient.ts
│       ├── report.ts
│       └── ...
│
├── public/
│   ├── manifest.json               # PWA manifest
│   ├── favicon.ico
│   ├── logo.png
│   └── ...
│
├── package.json
├── tsconfig.json
├── next.config.js
└── .env.local

```

### **Frontend Technology Stack**

```
React 18 & Next.js 14
├── Rendering Modes
│   ├── SSR (Server-Side Rendering)
│   ├── SSG (Static Site Generation)
│   ├── ISR (Incremental Static Regeneration)
│   └── CSR (Client-Side Rendering)
│
├── Routing
│   ├── File-based routing (app directory)
│   ├── Dynamic routes [id]
│   ├── Route groups (DashboardLayout)
│   └── Protected routes with middleware
│
├── State Management
│   └── Zustand
│       ├── Auth store (user, role, permissions)
│       ├── Patient store (current patient)
│       ├── UI store (sidebar open/close)
│       └── Data stores (caching)
│
├── Data Fetching
│   └── TanStack Query
│       ├── Query client setup
│       ├── Automatic caching
│       ├── Background refetch
│       └── Mutation handling
│
├── UI Components
│   └── Material-UI (MUI) 5
│       ├── Base components (Button, Card, etc.)
│       ├── Layout components (Grid, Stack, Box)
│       ├── Form components (TextField, etc.)
│       ├── Dialog/Modal components
│       └── Data display (Table, etc.)
│
├── Visualization
│   ├── Chart.js with react-chartjs-2
│   ├── Recharts for advanced charts
│   ├── Custom SVG charts (radar, circular)
│   └── Data visualization libraries
│
├── Styling
│   ├── Emotion (CSS-in-JS via MUI)
│   ├── Theme provider
│   ├── Global styles
│   └── Component-level styling
│
└── Authentication
    ├── JWT token storage
    ├── Header injection (Authorization)
    ├── Role-based rendering
    └── Protected route wrapper
```

### **Frontend Data Flow**

```
User Action
    ↓
React Component
    ↓
Zustand Store (if needed)
    ↓
TanStack Query / API Call
    ↓
Backend API (REST endpoint)
    ↓
Response Processing
    ↓
Store State Update
    ↓
Component Re-render
    ↓
UI Display Update
```

### **Component Hierarchy**

```
<RootLayout>
  └── <MainWrapper> (Aurora background)
      ├── <Sidebar>
      │   ├── <MenuItems>
      │   └── <SidebarItems>
      │
      ├── <PageWrapper>
      │   ├── <Header>
      │   │   ├── <Profile>
      │   │   ├── <NotificationCenter>
      │   │   └── <IconButtons>
      │   │
      │   ├── <Container>
      │   │   └── {children} (Page content)
      │   │
      │   └── <MobileBottomNav>
      │       ├── <NavTabs>
      │       └── <MoreSheet>
      │
      └── <OnboardingTips>
```

---

## 🔙 Backend Architecture (FastAPI)

### **Application Structure**

```
backend/
├── main.py                          # FastAPI entry point
│   ├── App initialization
│   ├── CORS configuration
│   ├── Middleware setup
│   └── Route registration
│
├── config.py                        # Configuration
│   ├── Database URL
│   ├── API keys (Groq, OpenAI)
│   ├── JWT secret
│   └── Environment variables
│
├── models/                          # SQLAlchemy models
│   ├── user.py                      # User model
│   ├── patient.py                   # Patient model
│   ├── genetic_report.py            # Genetic report model
│   ├── lab_result.py                # Lab results model
│   ├── health_score.py              # Health score model
│   ├── audit_log.py                 # Audit log model
│   └── ...
│
├── schemas/                         # Pydantic schemas (DTOs)
│   ├── user_schema.py               # User DTO
│   ├── patient_schema.py            # Patient DTO
│   ├── report_schema.py             # Report DTO
│   └── ...
│
├── routes/                          # API endpoints
│   ├── auth.py
│   │   ├── POST /api/auth/register
│   │   ├── POST /api/auth/login
│   │   ├── POST /api/auth/refresh
│   │   └── ...
│   │
│   ├── patients.py
│   │   ├── GET /api/patients
│   │   ├── POST /api/patients
│   │   ├── GET /api/patients/{id}
│   │   ├── PUT /api/patients/{id}
│   │   └── ...
│   │
│   ├── health_data.py
│   │   ├── GET /api/health/{patient_id}/score
│   │   ├── GET /api/health/{patient_id}/labs
│   │   ├── GET /api/health/{patient_id}/sources
│   │   └── ...
│   │
│   ├── genetic_analysis.py
│   │   ├── POST /api/analysis/snp
│   │   ├── POST /api/analysis/prs
│   │   ├── GET /api/analysis/{id}/results
│   │   └── ...
│   │
│   ├── reports.py
│   │   ├── POST /api/reports/generate
│   │   ├── GET /api/reports/{id}
│   │   ├── GET /api/reports/{id}/pdf
│   │   └── ...
│   │
│   ├── ai_chat.py
│   │   ├── POST /api/ai/chat
│   │   ├── POST /api/ai/insights
│   │   └── ...
│   │
│   ├── users.py
│   │   ├── GET /api/users (admin)
│   │   ├── POST /api/users (admin)
│   │   └── ...
│   │
│   └── audit.py
│       ├── GET /api/audit-log (admin)
│       └── ...
│
├── services/                        # Business logic
│   ├── auth_service.py              # Auth logic
│   ├── patient_service.py           # Patient logic
│   ├── health_score_service.py      # Health score calculation
│   │
│   ├── ai_service.py
│   │   ├── Groq API integration
│   │   ├── OpenAI integration
│   │   └── Response caching
│   │
│   ├── report_generator.py
│   │   ├── PDF generation
│   │   ├── Template rendering
│   │   └── Data formatting
│   │
│   ├── genetic_analysis_service.py
│   │   ├── SNP interpretation
│   │   ├── PRS calculation
│   │   ├── Pathway analysis
│   │   └── Drug-gene interactions
│   │
│   └── data_processor.py
│       ├── File upload handling
│       ├── Data validation
│       ├── Data transformation
│       └── Error handling
│
├── utils/                           # Utility functions
│   ├── jwt_utils.py                 # JWT token management
│   ├── password_utils.py            # bcrypt hashing
│   ├── validators.py                # Data validation
│   ├── email_sender.py              # Email sending
│   └── file_processor.py            # File handling
│
├── database/                        # Database setup
│   ├── engine.py                    # SQLAlchemy engine
│   ├── session.py                   # Session management
│   └── base.py                      # Base model for ORM
│
├── middleware/                      # Custom middleware
│   ├── auth_middleware.py           # Auth verification
│   ├── logging_middleware.py        # Request logging
│   └── error_handler.py             # Error handling
│
├── dependencies/                    # FastAPI dependencies
│   ├── auth_deps.py                 # Auth dependency
│   └── db_deps.py                   # DB session dependency
│
└── alembic/                         # Database migrations
    ├── versions/
    │   ├── 001_initial.py
    │   ├── 002_add_audit_log.py
    │   └── ...
    └── env.py
```

### **Backend Technology Stack**

```
FastAPI (Python 3.11+)
├── Web Framework
│   ├── Async request handling
│   ├── Automatic OpenAPI docs
│   ├── Type validation
│   └── Dependency injection
│
├── ORM & Database
│   ├── SQLAlchemy 2.0
│   ├── Connection pooling
│   ├── Transaction management
│   └── Query optimization
│
├── Authentication & Security
│   ├── PyJWT for tokens
│   ├── bcrypt for hashing
│   ├── CORS handling
│   └── HTTPS enforcement
│
├── Data Validation
│   ├── Pydantic models
│   ├── Request validation
│   ├── Response schema enforcement
│   └── Error reporting
│
├── AI Integration
│   ├── Groq API client
│   ├── OpenAI API client
│   ├── Prompt engineering
│   ├── Response parsing
│   └── Token management
│
├── File Processing
│   ├── VCF/BAM parsing
│   ├── PDF extraction
│   ├── CSV import
│   └── Data validation
│
└── Logging & Monitoring
    ├── Structured logging
    ├── Error tracking
    ├── Performance monitoring
    └── Audit logging
```

### **Backend Request/Response Cycle**

```
Incoming Request
    ↓
Middleware (CORS, logging)
    ↓
Route Handler
    ↓
Dependencies: Get DB session, Auth user
    ↓
Service Layer: Business logic
    ↓
Database query/update (if needed)
    ↓
AI processing (if needed)
    ↓
Response schema validation
    ↓
Return JSON response
    ↓
Middleware: Response logging
    ↓
Send to client
```

---

## 💾 Database Architecture (PostgreSQL 15)

### **Database Schema**

```
PostgreSQL 15
├── Tables
│   ├── users
│   │   ├── id (UUID, PK)
│   │   ├── email (VARCHAR, UNIQUE, INDEXED)
│   │   ├── password_hash (VARCHAR)
│   │   ├── role (ENUM: admin, doctor, tech, visitor)
│   │   ├── first_name, last_name (VARCHAR)
│   │   ├── created_at, updated_at (TIMESTAMP)
│   │   └── is_active (BOOLEAN)
│   │
│   ├── patients
│   │   ├── id (UUID, PK)
│   │   ├── first_name, last_name (VARCHAR)
│   │   ├── date_of_birth (DATE)
│   │   ├── gender (ENUM)
│   │   ├── email, phone (VARCHAR, nullable)
│   │   ├── created_at, updated_at (TIMESTAMP)
│   │   ├── created_by (UUID, FK → users)
│   │   └── is_active (BOOLEAN)
│   │
│   ├── genetic_reports (with JSONB data)
│   ├── lab_results (with arrays and ranges)
│   ├── health_scores (denormalized for performance)
│   ├── audit_logs (immutable, indexed heavily)
│   └── ... (20+ total tables)
│
├── Indexes (optimized for queries)
│   ├── Patient lookups
│   ├── Report searches
│   ├── Audit trail queries
│   └── Time-series data access
│
├── View (for complex queries)
│   ├── Patient health overview
│   ├── Report summary
│   └── Analytics views
│
├── Functions (stored procedures)
│   ├── Health score calculation
│   ├── Data aggregation
│   └── Time-series analysis
│
└── Constraints
    ├── Foreign key relationships
    ├── NOT NULL constraints
    ├── UNIQUE constraints
    └── CHECK constraints
```

### **Database Access Patterns**

```
Connection Pool (psycopg2)
├── Pool size: 50 connections
├── Max overflow: 10
├── Timeout: 30 seconds
└── Pre-ping enabled (dead connection detection)

Query Optimization
├── Prepared statements (parameterized queries)
├── Index selection
├── Query plan analysis
└── Caching layer (Redis optional)
```

---

## 🔐 Authentication & Authorization Flow

### **Authentication**

```
User Login
    ↓
POST /api/auth/login (email, password)
    ↓
Validate credentials
    ↓
Verify password with bcrypt
    ↓
Generate JWT tokens
├── Access token (24-hour expiry)
└── Refresh token (7-day expiry)
    ↓
Return tokens to client
    ↓
Client stores tokens
├── Access token: localStorage
└── Refresh token: httpOnly cookie
    ↓
Client includes Authorization header
    ↓
Backend validates token signature
    ↓
Extract user info from token claims
```

### **Authorization**

```
Request with JWT token
    ↓
Validate token signature
    ↓
Check token expiry
    ↓
Extract user_id and role
    ↓
Check role permissions (RBAC)
    ├── Admin: all endpoints
    ├── Doctor: clinical + reporting
    ├── Tech: data entry
    ├── Patient: own data + AI
    └── Visitor: limited access
    ↓
Check resource ownership
├── Patients: own data only
├── Doctors: assigned patients
└── Admins: all data
    ↓
Grant/deny access
    ↓
Audit the access attempt
```

---

## 🤖 AI Integration Architecture

### **Groq API Flow**

```
Patient Health Data
    ↓
Prepare context (curated patient information)
    ↓
Build prompt template
    ↓
Call Groq API
├── Model: LLaMA 3 70B
├── Max tokens: 2000
├── Temperature: 0.7
└── Stream responses: True
    ↓
Stream response to frontend
    ↓
Parse response chunks
    ↓
Cache response if applicable
    ↓
Store in chat history
```

### **Report Generation Flow**

```
Report generation request
    ├── Type: Clinical, Genetic, Recommendations, etc.
    ├── Patient ID
    └── Date range (optional)
    ↓
Aggregate patient data
├── Demographics
├── Genetic findings
├── Lab results
├── Health scores
└── Previous recommendations
    ↓
Call AI to interpret data
├── Groq API for generation
├── Custom templates
└── Data formatting
    ↓
Generate PDF
├── Header with branding
├── Formatted sections
├── Charts and visualizations
└── Professional layout
    ↓
Store report in database
    ↓
Return to user
```

---

## 📊 Data Flow Diagrams

### **Multi-Omic Data Integration**

```
External Data Sources
├── Genomics (VCF upload)
├── Labs (CSV import)
├── Microbiome (analysis file)
├── Epigenetics (methylation)
├── Mitochondria (expression)
├── PRS (variant data)
└── Wearables (API sync)
    ↓
Backend Processors
├── File validation
├── Format conversion
├── Data normalization
└── Quality checks
    ↓
Database Storage
├── Normalize/store data
├── Index for queries
└── Create aggregations
    ↓
Health Score Calculation
├── Weigh each source (15-25%)
├── Aggregate scores
└── Generate percentiles
    ↓
Frontend Display
├── Dashboard visualization
├── Source breakdown
└── Risk stratification
```

### **Clinical Decision Support**

```
Patient Medical Encounter
    ↓
Enter/upload patient data
    ↓
System processes
├── Validates data
├── Calculates scores
├── Identifies alerts
└── Prepares context
    ↓
AI Analysis
├── Interprets findings
├── Cross-references data
├── Generates insights
└── Produces recommendations
    ↓
Provider Reviews
├── Clinical dashboard
├── Severity highlighted
├── AI insights displayed
└── Action items listed
    ↓
Provider Actions
├── Generate reports
├── Share with patient
├── Document decisions
└── Log audit trail
    ↓
Patient Follow-up
├── Email/portal notification
├── Recommended protocols
├── Progress tracking
└── Outcome measurement
```

---

## 🚀 Deployment Architecture

### **Docker Compose Setup**

```yaml
services:
  frontend:
    # Next.js 14 container
    # - Node.js 18 base image
    # - Port 3100
    # - Environment: production
    
  backend:
    # FastAPI container
    # - Python 3.11 base image
    # - Port 8500
    # - Uvicorn ASGI server
    
  postgres:
    # PostgreSQL 15 container
    # - Port 5432
    # - Persistent volume
    # - Connection pooling

  nginx:
    # Reverse proxy
    # - Port 80, 443
    # - SSL/TLS termination
    # - Load balancing
```

### **Cloud Deployment (DigitalOcean)**

```
DigitalOcean App Platform
├── Frontend
│   ├── Next.js build optimization
│   ├── CDN distribution
│   └── Auto-scaling
│
├── Backend
│   ├── FastAPI instances
│   ├── Load balancer
│   └── Auto-scaling
│
└── Database
    ├── Managed PostgreSQL
    ├── Automated backups
    ├── Read replicas (optional)
    └── Point-in-time recovery
```

---

## 📈 Scalability Design

### **Horizontal Scaling**

```
Frontend (Next.js)
├── CDN caching layer
├── Multiple instances behind load balancer
├── Static asset optimization
└── ISR for data freshness

Backend (FastAPI)
├── Multiple application instances
├── Load balancer (nginx/HAProxy)
├── Connection pooling to database
└── Cache layer (Redis optional)

Database (PostgreSQL)
├── Connection pooling
├── Read replicas for read-heavy queries
├── Vertical scaling (more CPU/RAM)
└── Partitioning for large tables
```

### **Performance Optimization**

```
Frontend
├── Code splitting
├── Image optimization
├── Dynamic imports
├── CSS-in-JS optimization
└── Bundle analysis

Backend
├── Query optimization
├── Index tuning
├── Connection pooling
├── Response caching
└── Async processing

Database
├── Index creation
├── Query plan optimization
├── Statistics updates
└── Vacuum and analysis
```

---

## 🔄 CI/CD Pipeline (Potential)

```
Developer Push
    ↓
GitHub Actions
├── Lint & format check
├── Unit tests
├── Integration tests
├── Build Docker images
└── Push to registry
    ↓
Staging Deployment
├── Deploy to staging environment
├── Run smoke tests
├── Performance testing
└── Security scanning
    ↓
Production Deployment
├── Tag release
├── Deploy containers
├── Database migrations (if needed)
├── Health checks
└── Monitoring alerts
```

---

## 📊 Monitoring & Logging

### **Observability Stack**

```
Application Logging
├── Structured logging (JSON format)
├── Multiple levels (DEBUG, INFO, WARNING, ERROR)
├── Centralized log aggregation
└── Searchable and filterable

Metrics Collection
├── Prometheus metrics
├── Application performance monitoring
├── Resource utilization tracking
└── Business metrics

Error Tracking
├── Sentry integration (frontend)
├── Custom error middleware (backend)
├── Alert on critical errors
└── Error trend analysis

Distributed Tracing
└── Request ID tracking across services
```

---

## 🛡️ Security Architecture

```
Network Security
├── TLS/SSL encryption
├── CORS policy enforcement
├── Rate limiting
└── DDoS protection

Application Security
├── Input validation
├── SQL injection prevention
├── CSRF protection
├── XSS prevention
└── Secure headers

Authentication & Authorization
├── JWT tokens
├── Role-based access control
├── Multi-factor authentication (optional)
└── Secure session management

Data Security
├── Encryption at rest
├── Encryption in transit
├── Data anonymization
└── Audit logging
```

---

## 📋 API Standards

### **RESTful Design**

```
Resources
├── /api/patients
├── /api/patients/{id}
├── /api/reports
├── /api/reports/{id}
└── ...

Methods
├── GET - Retrieve resource
├── POST - Create resource
├── PUT - Update entire resource
├── PATCH - Partial update (optional)
└── DELETE - Delete resource

Status Codes
├── 200 OK - Success
├── 201 Created - Resource created
├── 204 No Content - Success, no response body
├── 400 Bad Request - Invalid input
├── 401 Unauthorized - Not authenticated
├── 403 Forbidden - No permission
├── 404 Not Found - Resource not found
└── 500 Server Error - Server issue
```

### **Response Format**

```json
{
  "success": true,
  "data": { /* resource data */ },
  "error": null,
  "metadata": {
    "page": 1,
    "page_size": 20,
    "total": 100
  }
}
```

---

**Document Version**: 1.0  
**Last Updated**: May 1, 2026  
**Classification**: Internal - Architecture Documentation
