# 🏗️ Architecture - AI Video Hanzl Platform

**Version**: 1.0.0  
**Last Updated**: 2026-09-29  

---

## System Overview

The AI Video Hanzl Platform is a distributed, scalable system designed to handle massive concurrent video generation requests while maintaining complete traceability and auditability.

### Core Principles
1. **Traceability**: Every action logged immutably
2. **Scalability**: Horizontal scaling via microservices
3. **Reliability**: Multi-engine fallback & redundancy
4. **Security**: Zero-trust architecture
5. **Performance**: Sub-second API response times

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Client Layer                                 │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐  │
│  │  Web Portal  │  Mobile App  │  REST API    │  CLI Tool    │  │
│  └──────────────┴──────────────┴──────────────┴──────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ (HTTPS/TLS)
┌────────────────────────────▼────────────────────────────────────┐
│              API Gateway & Load Balancer                         │
│  ┌─ Authentication Service                                      │
│  ├─ Rate Limiting & Quota Management                            │
│  ├─ Request Validation & Sanitization                           │
│  └─ Request Routing & Load Distribution                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│         Core Service Layer (Microservices)                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Video Service          Project Service   Analytics Svc  │   │
│  │ - Generate             - CRUD Ops        - Metrics       │   │
│  │ - Status               - Sharing         - Reporting     │   │
│  │ - Download             - Settings        - Dashboard     │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Auth Service           Queue Service      Audit Service  │   │
│  │ - JWT Auth             - Job Management   - Logging       │   │
│  │ - API Keys             - Priority Queue   - Compliance    │   │
│  │ - RBAC                 - Worker Pool      - Export        │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│         Processing Engine Orchestration                          │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────┐        │
│  │ Sora API │ Runway   │ HeyGen   │ Pika     │ Kling    │        │
│  │ Bridge   │ Bridge   │ Bridge   │ Bridge   │ Bridge   │        │
│  └──────────┴──────────┴──────────┴──────────┴──────────┘        │
│       │          │         │         │         │                │
│       └──────────┴─────────┴─────────┴─────────┘                │
│              Worker Pool                                         │
│       (Manages concurrent jobs, retries, fallback)              │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│            Storage & Persistence Layer                           │
│  ┌──────────────┬──────────────┬──────────────┐                 │
│  │  PostgreSQL  │  Redis Cache │  S3/GCS      │                 │
│  │  Metadata DB │  Sessions    │  Video Store │                 │
│  │  Audit Logs  │  Job Queue   │  Backups     │                 │
│  └──────────────┴──────────────┴──────────────┘                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│        Monitoring, Observability & Analytics                     │
│  ┌──────────────┬──────────────┬──────────────┐                 │
│  │  Prometheus  │  Grafana     │  ELK Stack   │                 │
│  │  Metrics     │  Dashboards  │  Logs        │                 │
│  └──────────────┴──────────────┴──────────────┘                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. API Gateway

**Responsibilities**:
- HTTP/HTTPS request handling
- Load balancing across instances
- Authentication verification
- Rate limiting enforcement
- Request logging

**Technology**: 
- Nginx (reverse proxy)
- Kong (API management)

**Configuration**:
```yaml
upstream video_service {
  server service1:5000;
  server service2:5000;
  server service3:5000;
}

server {
  listen 443 ssl;
  server_name api.ai-video-hanzl.com;
  
  location /api/v1/ {
    auth_request /verify_token;
    proxy_pass http://video_service;
    limit_req zone=api burst=100 nodelay;
  }
}
```

### 2. Core Services

#### Video Service
```
Endpoints:
- POST /videos/generate        → Queue new job
- GET /videos/{id}             → Fetch status
- GET /videos                  → List user videos
- DELETE /videos/{id}          → Remove video
- GET /videos/{id}/download    → Download file

Database Tables:
- videos (id, project_id, prompt, engine, status, output_url)
- video_parameters (video_id, duration, resolution, style)
- video_output (video_id, url, file_size, duration, format)
```

#### Project Service
```
Endpoints:
- POST /projects               → Create project
- GET /projects/{id}           → Get project details
- PUT /projects/{id}           → Update project
- DELETE /projects/{id}        → Delete project
- POST /projects/{id}/share    → Share project

Database Tables:
- projects (id, owner_id, name, description, settings)
- project_members (project_id, user_id, role, joined_at)
- project_videos (project_id, video_id, order, tags)
```

#### Queue Service
```
Components:
- Job Queue (Redis)
  → Priority queue management
  → Exponential backoff for retries
  → Dead letter queue for permanent failures

- Worker Pool
  → Configurable worker count per engine
  → Automatic scaling based on queue depth
  → Health checks & graceful shutdown

- State Machine
  queued → processing → completed/failed
    ↓        ↓             ↓
   LOG      LOG           LOG
```

#### Audit Service
```
Responsibilities:
- Immutable audit log storage
- Event tracking
- Compliance reporting
- Query interface

Log Entry Structure:
{
  log_id: UUID,
  timestamp: ISO-8601,
  user_id: UUID,
  action: String,
  resource: String,
  resource_id: UUID,
  status: String,
  details: Object,
  ip_address: String,
  user_agent: String,
  tags: [String],
  signature: String (HMAC for integrity)
}
```

### 3. Engine Integration Layer

Each engine has a dedicated adapter following the adapter pattern:

```python
class EngineAdapter(ABC):
    @abstractmethod
    def generate(self, prompt: str, params: Dict) -> str:
        """Returns video URL"""
        pass
    
    @abstractmethod
    def get_status(self, job_id: str) -> Dict:
        """Returns job status"""
        pass
    
    @abstractmethod
    def cancel(self, job_id: str) -> bool:
        """Cancels job"""
        pass

class SoraAdapter(EngineAdapter):
    def generate(self, prompt: str, params: Dict):
        response = self.client.videos.generate(
            model="gpt-4-vision",
            prompt=prompt,
            duration=params.get("duration", 30),
            resolution=params.get("resolution", "1080p")
        )
        return response.video_url
```

### 4. Database Schema

#### Core Tables

**videos**
```sql
CREATE TABLE videos (
  id UUID PRIMARY KEY,
  project_id UUID NOT NULL,
  engine VARCHAR(50) NOT NULL,
  prompt TEXT NOT NULL,
  status VARCHAR(50) DEFAULT 'queued',
  output_url VARCHAR(500),
  error_message TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  FOREIGN KEY (project_id) REFERENCES projects(id)
);

CREATE INDEX idx_videos_project_id ON videos(project_id);
CREATE INDEX idx_videos_status ON videos(status);
CREATE INDEX idx_videos_created_at ON videos(created_at DESC);
```

**projects**
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  owner_id UUID NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  settings JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (owner_id) REFERENCES users(id)
);

CREATE INDEX idx_projects_owner_id ON projects(owner_id);
```

**audit_logs**
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT NOW(),
  user_id UUID,
  action VARCHAR(255) NOT NULL,
  resource VARCHAR(50),
  resource_id UUID,
  status VARCHAR(50),
  details JSONB,
  ip_address INET,
  user_agent VARCHAR(500),
  signature VARCHAR(256),
  CONSTRAINT immutable CHECK (true)
);

CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp DESC);
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
```

### 5. Caching Strategy

**Multi-layer Caching**:

```
Layer 1: CDN Cache
  ├─ Video files (30 days)
  ├─ Metadata (5 minutes)
  └─ Static assets (1 year)

Layer 2: Application Cache (Redis)
  ├─ User sessions (24 hours)
  ├─ Project metadata (1 hour)
  ├─ API responses (5 minutes)
  └─ Job queue (in-memory)

Layer 3: Database Cache (Query Results)
  ├─ Prepared statements
  └─ Connection pooling
```

### 6. Message Queue

**Job Processing Pipeline**:

```
Producer:
  POST /videos/generate
       ↓
  Create Job Record
       ↓
  Push to Redis Queue
       ↓
  Return job_id

Consumer (Worker):
  Pull from Queue
       ↓
  Acquire Lock (Redis)
       ↓
  Call Engine API
       ↓
  Update Status
       ↓
  Store Result
       ↓
  Release Lock
       ↓
  Log Completion
```

**Queue Configuration**:
```yaml
queues:
  high_priority:
    max_workers: 10
    timeout: 600  # 10 minutes
  normal_priority:
    max_workers: 5
    timeout: 1800  # 30 minutes
  low_priority:
    max_workers: 2
    timeout: 3600  # 60 minutes
```

### 7. Security Architecture

**Authentication Flow**:
```
Client Request
     ↓
API Gateway
     ↓
Extract Bearer Token
     ↓
Validate JWT Signature
     ↓
Check Token Expiry
     ↓
Load User from Cache/DB
     ↓
Verify Permissions (RBAC)
     ↓
Route to Service
```

**API Key Management**:
- Keys stored as bcrypt hashes
- Rotatable (optional expiry)
- Revocable at any time
- Rate limited per key

### 8. Deployment Architecture

**Kubernetes Configuration**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: video-service
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
      - name: video-service
        image: ai-video-hanzl/video-service:1.0.0
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
```

### 9. Monitoring & Observability

**Key Metrics**:
```
Application Metrics:
- videos_generated_total (counter)
- video_generation_duration_seconds (histogram)
- video_queue_depth (gauge)
- api_request_latency_seconds (histogram)
- api_request_errors_total (counter)
- generation_success_rate (gauge)

Infrastructure Metrics:
- container_memory_usage_bytes
- container_cpu_usage_seconds
- disk_io_operations
- network_bytes_transferred

Business Metrics:
- total_revenue
- videos_per_user
- average_generation_cost
- engine_usage_distribution
```

**Dashboards**:
- Real-time operations dashboard
- User analytics dashboard
- Financial dashboard (cost tracking)
- System health dashboard

---

## Data Flow

### Video Generation Flow

```
1. User submits request
   POST /videos/generate
   {
     "project_id": "...",
     "engine": "sora",
     "prompt": "..."
   }

2. API Gateway validates & routes
   ↓
3. Video Service creates database record
   INSERT INTO videos (...) VALUES (...)
   ↓
4. Audit Service logs action
   INSERT INTO audit_logs (...) VALUES (...)
   ↓
5. Job queued in Redis
   LPUSH queue:normal_priority job_123
   ↓
6. Worker picks up job
   LPOP queue:normal_priority
   ↓
7. Worker calls engine API
   SoraAdapter.generate(prompt, params)
   ↓
8. Engine processes video
   ~ 2-5 minutes processing time ~
   ↓
9. Engine returns video URL
   response.video_url = "https://..."
   ↓
10. Worker updates database
    UPDATE videos SET status='completed', output_url='...'
    ↓
11. Audit Service logs completion
    INSERT INTO audit_logs (...) VALUES (...)
    ↓
12. Webhook notification sent (if configured)
    POST callback_url with video data
    ↓
13. User retrieves via API
    GET /videos/{video_id}
    {
      "status": "completed",
      "output_url": "..."
    }
```

---

## Scalability Considerations

### Horizontal Scaling

- **Stateless Services**: All services designed to be horizontally scalable
- **Shared Storage**: PostgreSQL with read replicas
- **Distributed Cache**: Redis Cluster for queue & session storage
- **Load Balancing**: Round-robin with health checks
- **Auto-scaling**: Kubernetes HPA based on CPU/memory

### Vertical Scaling

- **Database**: Connection pooling, query optimization, indexing
- **Cache**: Memory allocation, eviction policies
- **Workers**: Concurrent job processing per worker

### Bottleneck Analysis

1. **API Gateway**: Nginx/Kong easily handles 10k+ RPS
2. **Database**: PostgreSQL with proper indexing ~1000 queries/sec
3. **Worker Pool**: Depends on engine availability & rate limits
4. **Storage**: S3/GCS virtually unlimited

---

## Disaster Recovery

### Backup Strategy
- **Database**: Daily snapshots, 30-day retention
- **Videos**: Replicated across availability zones
- **Audit Logs**: 1-year retention, immutable
- **Configuration**: Version controlled in Git

### Failover
- **Multi-region deployment**: Active-active in 2+ regions
- **Database replication**: PostgreSQL streaming replication
- **Service mesh**: Automatic service discovery & failover
- **RTO**: <5 minutes
- **RPO**: <1 minute

---

## Cost Optimization

### Caching Effectiveness
- CDN cache reduces API calls by 60%
- Application cache reduces database queries by 40%
- Overall cost reduction: ~30-40%

### Engine Selection
- Automatically select cheapest engine for given parameters
- Batch similar jobs to negotiate better rates
- Monitor engine pricing and adjust routing

### Resource Utilization
- Right-sized container resources
- Spot instances for batch processing
- Reserved capacity for baseline load

---

**Document Version**: 1.0.0  
**Last Updated**: 2026-09-29  
**Maintained By**: Jaroslav Hanzl MasterCaryl Andrelan
