# Infrastructure Architecture

## Scaling to 10K User Base

### Infrastructure Architecture for 10K Users

#### Growth Phases

**Phase 1: 0-1,000 Users (Week 1-4)**

![Phase 1 Architecture](../../diagrams/architecture-phase1.png)

```
Single Server Architecture
┌─────────────────────────────────┐
│  Load Balancer (Basic)          │
└────────────┬────────────────────┘
             │
┌────────────▼────────────────────┐
│  App Server (t3.medium)         │
│  - API + WebSocket              │
│  - 2 vCPU, 4GB RAM              │
└────────────┬────────────────────┘
             │
    ┌────────┼────────┐
    │        │        │
    ▼        ▼        ▼
┌───────┐ ┌──────┐ ┌────────┐
│MongoDB│ │Redis │ │   S3   │
│(M10)  │ │(1GB) │ │(Images)│
└───────┘ └──────┘ └────────┘

Cost: $300-500/month
```

**Phase 2: 1,000-5,000 Users (Week 5-8)**

![Phase 2 Architecture](../../diagrams/architecture-phase2.png)

```
Horizontal Scaling Architecture
┌─────────────────────────────────┐
│  Load Balancer (ALB)            │
└────────┬─────────┬──────────────┘
         │         │
    ┌────▼────┐ ┌──▼───────┐
    │  App 1  │ │  App 2   │
    │(t3.med) │ │(t3.med)  │
    └────┬────┘ └──┬───────┘
         │         │
         └────┬────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌───────┐ ┌───────┐ ┌────────┐
│MongoDB│ │Redis  │ │   S3   │
│Primary│ │Cluster│ │+ CDN   │
│+ Read │ │(2GB)  │ │        │
│Replica│ │       │ │        │
172: │       │ │        │
└───────┘ └───────┘ └────────┘

Cost: $800-1,200/month
```

**Phase 3: 5,000-10,000 Users (Week 9-12)**

![Phase 3 Architecture](../../diagrams/architecture-phase3.png)

```
Microservices Architecture
┌─────────────────────────────────┐
│  Load Balancer (ALB)            │
└────┬─────┬─────┬─────┬──────────┘
     │     │     │     │
     ▼     ▼     ▼     ▼
┌────────┐ ┌────────┐ ┌────────┐
│ App 1  │ │ App 2  │ │ App 3  │
│(t3.med)│ │(t3.med)│ │(t3.med)│
└────────┘ └────────┘ └────────┘
     │          │          │
     └──────────┼──────────┘
                │
    ┌───────────┼───────────┐
    │           │           │
    ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│Auth Svc │ │Match Svc│ │Chat Svc │
└─────────┘ └─────────┘ └─────────┘
    │           │           │
    └───────────┼───────────┘
                │
    ┌───────────┼───────────┐
    │           │           │
    ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ MongoDB │ │  Redis  │ │   S3    │
│Primary +│ │ Cluster │ │ + CDN   │
│2 Replicas│ │ (4GB)   │ │         │
│(Sharded)│ │         │ │         │
└─────────┘ └─────────┘ └─────────┘

Cost: $1,500-3,000/month
```

### Detailed Infrastructure Specifications

#### For 10,000 Active Users

**Application Servers**:
- 3x EC2 t3.medium instances (2 vCPU, 4GB RAM each)
- Auto-scaling group (min: 2, max: 5)
- Application Load Balancer
- **Cost**: $150/server × 3 = $450/month

**Database (MongoDB)**:
- MongoDB Atlas M30 (8GB RAM) or self-hosted cluster
- 1 Primary + 2 Read Replicas (Replica Set)
- 500GB SSD storage
- Automated backups (7-day retention)
- **Cost**: Atlas M30 $280/month or self-hosted $400/month

**Caching Layer**:
- ElastiCache Redis cluster
- 2 nodes, 4GB RAM each
- Multi-AZ for high availability
- **Cost**: $150/month

**Storage & CDN**:
- S3 bucket: 500GB storage, 2TB transfer
- CloudFront CDN: 5TB/month data transfer
- Estimated 100,000 images (average 500KB each)
- **Cost**: S3 $25 + CDN $400 = $425/month

**Message Queue**:
- AWS SQS for background jobs
- 10 million requests/month
- **Cost**: $50/month

**Monitoring & Logging**:
- CloudWatch logs and metrics
- Error tracking (Sentry)
- **Cost**: $100/month

**Third-Party Services**:
- Twilio SMS: 10,000 verifications × $0.02 = $200
- Firebase Push: Free tier (unlimited)
- Image processing: $100/month
- **Cost**: $300/month

**Total Infrastructure Cost for 10K Users**: **$1,755/month**

With buffer and contingency: **$2,000-2,500/month**

### Performance Targets for 10K Users

| Metric | Target | Strategy |
|--------|--------|----------|
| API Response Time | <200ms | Caching, MongoDB indexing |
| Profile Load Time | <1s | CDN, image optimization |
| Match Detection | <500ms | Redis queue, async processing |
| Message Delivery | <300ms | WebSocket, Redis pub/sub |
| Discovery Feed | <1s | Pre-computed matches, caching |
| Image Load Time | <500ms | CDN, progressive loading |
| Concurrent Users | 1,000+ | Horizontal scaling, load balancer |
| Database Queries/sec | 500+ | Read replicas, connection pooling |
| Uptime | 99.5%+ | Multi-AZ deployment, monitoring |

### Scalability Bottlenecks & Solutions

| Bottleneck | Impact at Scale | Solution |
|------------|----------------|----------|
| **Database Writes** | Matches, messages pile up | Write queue, batch inserts, MongoDB sharding |
| **Image Storage** | S3 GET requests expensive | CloudFront CDN caching |
| **Discovery Algorithm** | Slow profile queries | Pre-compute matches, Redis cache, geospatial indexes |
| **WebSocket Connections** | Memory exhaustion | Separate chat server, Socket.io Redis adapter |
| **Location Queries** | Slow geospatial searches | MongoDB 2dsphere indexes |
| **User Sessions** | Memory bloat | Redis session store |

### Monitoring & Alerts (for 10K users)

**Key Metrics to Track**:
- Server CPU/Memory usage (alert at 70%)
- MongoDB connections (alert at 80% pool)
- API response times (alert if p95 > 500ms)
- Error rates (alert if > 1%)
- Active WebSocket connections
- Cache hit rates (target: 80%+)
- Queue length (SQS message backlog)

**Tools**:
- CloudWatch for infrastructure
- MongoDB Atlas monitoring (if using Atlas)
- Sentry for error tracking
- Custom dashboard for business metrics
- PagerDuty for on-call alerts

### Cost Breakdown for 10K Users

| Category | Monthly Cost |
|----------|--------------|
| Application Servers (3x) | $450 |
| MongoDB (Atlas M30 or self-hosted) | $280-400 |
| Redis Cache | $150 |
| S3 Storage + CDN | $425 |
| Load Balancer | $50 |
| Message Queue | $50 |
| Monitoring | $100 |
| SMS Verification (Twilio) | $200 |
| Push Notifications | $0 (free) |
| Image Processing | $100 |
| Backup Storage | $75 |
| **Total Infrastructure** | **$1,880-2,000** |
| **With 20% buffer** | **$2,256-2,400** |

**Team Costs** (not infrastructure):
- 2 Backend Developers: $10,000
- 2 Mobile Developers: $10,000
- 1 DevOps Engineer: $6,000
- 1 QA Tester: $3,000
- 1 Content Moderator: $2,000
- **Total Team**: **$31,000/month**

**Grand Total Operating Cost**: **$33,256-33,400/month** for 10K users
