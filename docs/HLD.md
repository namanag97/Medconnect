# High-Level Design (HLD): MedConnect MVP (India) - Improved Version

**Document:** High-Level Design (HLD)  
**Product:** MedConnect  
**Phase:** MVP (Minimum Viable Product)  
**Status:** Final & Reviewed  
**Date:** July 8, 2025  

## 1. Guiding Principles & Core Strategy

This document outlines the architecture for the MedConnect MVP. Our technical strategy is driven by a singular focus: to validate our core business idea in the Indian market as quickly and efficiently as possible.

Every decision is guided by three principles:

**Build Less, Learn Faster:** We will consciously build the simplest possible system that solves the core problem. Features that are not essential for validating the initial user journey are postponed.

**Use Proven, Simple Technology:** We will use common, reliable tools that are easy to set up, manage, and hire for. This reduces technical risk and development time.

**Mobile is Everything:** The system is designed from the ground up for the reality of the Indian market: the primary user experience is on a mobile phone with a potentially variable internet connection.

## 2. High-Level Architectural Diagram

We will use a **Simplified Monolithic Architecture** with enhanced reliability patterns for the MVP. This is a deliberate choice to maximize development speed while ensuring production readiness.

'''
+------------------------------------------------------------------+
|                    User's Mobile Browser                         |
|              (PWA-enabled React Web App)                         |
|          + Offline Capability + Push Notifications              |
+--------------------------------+---------------------------------+
                                 |
                                 | (Secure HTTPS + CDN)
                                 v
+-----------------------------------------------------------------------------+
|                         Cloud Platform (AWS Multi-AZ)                      |
|                                                                             |
|    +---------------------------+    +--------------------------------------+
|    |    Application Gateway     |    |           Health Check               |
|    | - Rate Limiting            |--->|        & Monitoring                  |
|    | - DDoS Protection          |    |      (CloudWatch + Alerts)          |
|    | - SSL Termination          |    +--------------------------------------+
|    +---------------------------+                                             |
|                 |                                                           |
|                 v                                                           |
|    +---------------------------+    +--------------------------------------+
|    |    Auto-Scaling Group     |    |         Circuit Breaker              |
|    | (Min: 2, Max: 10 instances)|<-->|       & Retry Logic                  |
|    +---------------------------+    +--------------------------------------+
|                 |                                                           |
|                 v                                                           |
|    +------------------------------------------------------------------+    |
|    |                    Main Backend Server                           |    |
|    |              (Node.js with Express.js)                          |    |
|    |                                                                  |    |
|    |  Core Business Logic:                                           |    |
|    |  - Authentication & Authorization (JWT + OTP)                   |    |
|    |  - User Profile & CV Management                                 |    |
|    |  - Job Posting & Search (with caching)                         |    |
|    |  - Application Processing (with queues)                        |    |
|    |  - Analytics & Logging                                          |    |
|    +----------------------------------+-------------------------------+    |
|                                       |                                     |
|                   (Async Queue)       | (Background Processing)             |
|                          ^            v                                     |
|                          |    +-------------------------------+             |
|                          |    |    Message Queue System       |             |
|                          |    |     (AWS SQS + DLQ)          |             |
|                          |    +-------------------------------+             |
|                          |            |                                     |
|                          |            v                                     |
|                          |    +-------------------------------+             |
|                          |    |   Background Workers          |             |
|                          |    | - Email/WhatsApp Notifications|             |
|                          |    | - CV Processing               |             |
|                          |    | - Data Analytics              |             |
|                          |    +-------------------------------+             |
|                          |                                                  |
|    +---------------------------------------------------------------------+  |
|    |                        Data Storage Layer                          |  |
|    |                                                                     |  |
|    |  +----------------+ +--------------------+ +--------------------+  |  |
|    |  |  PostgreSQL    | |   Amazon S3        | |   Redis Cluster    |  |  |
|    |  | (Primary + RO  | |   (Multi-region    | |   (Sessions, Cache,|  |  |
|    |  |  Replica)      | |    with CDN)       | |    Queues)         |  |  |
|    |  +----------------+ +--------------------+ +--------------------+  |  |
|    |                                                                     |  |
|    +---------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------+
         |                      |                     |
         | (SES + Templates)    | (Twilio WhatsApp)   | (Analytics)
         v                      v                     v
  +--------------+       +--------------+      +--------------+
  |   Employer   |       |   Candidate  |      |   Metrics    |
  +--------------+       +--------------+      +--------------+
'''

## 3. Component Deep Dive: Enhanced Architecture

### 3.1 Frontend Layer (Progressive Web App)

**What it is:** A React-based Progressive Web App optimized for mobile devices with offline capabilities.

**Key Features:**
- **Offline Support:** Core functionality works without internet using service workers
- **Push Notifications:** Real-time updates for job matches and application status
- **Responsive Design:** Optimized for various screen sizes and network conditions
- **Performance:** Lazy loading, code splitting, and image optimization

**Why this choice?** PWAs provide native app-like experience without app store dependencies, crucial for rapid iteration in the Indian market where users may have storage constraints.

### 3.2 API Gateway & Load Balancing

**What it is:** AWS Application Load Balancer with integrated security features.

**Key Features:**
- **Rate Limiting:** Prevents API abuse (100 requests/minute per user)
- **DDoS Protection:** AWS Shield Standard integration
- **SSL Termination:** Centralized certificate management
- **Health Checks:** Automatic failover for unhealthy instances

**Why this choice?** Essential for production reliability and security, especially given the sensitive nature of medical professional data.

### 3.3 Enhanced Backend Architecture

**What it is:** A Node.js monolith with microservice-ready patterns.

**Key Improvements:**
- **Modular Structure:** Clear separation of concerns with domain-driven design
- **Input Validation:** Comprehensive validation using Joi schemas
- **Error Handling:** Centralized error handling with proper logging
- **API Documentation:** Automatic OpenAPI/Swagger documentation
- **Security:** Helmet.js, CORS, input sanitization, and rate limiting

**Code Structure:**
'''
src/
├── controllers/     # Route handlers
├── services/       # Business logic
├── models/         # Database models
├── middleware/     # Custom middleware
├── utils/          # Helper functions
├── config/         # Configuration management
└── tests/          # Comprehensive test suite
'''

### 3.4 Message Queue System

**What it is:** AWS SQS with Dead Letter Queues for reliable background processing.

**Key Features:**
- **Job Prioritization:** High-priority notifications vs. batch processing
- **Retry Logic:** Exponential backoff for failed jobs
- **Dead Letter Queue:** Failed messages for manual review
- **Monitoring:** CloudWatch metrics for queue depth and processing time

**Why this choice?** Ensures reliable delivery of critical notifications and allows the system to handle traffic spikes gracefully.

### 3.5 Enhanced Data Layer

#### PostgreSQL (Primary Database)
**Improvements:**
- **Read Replicas:** Separate read operations to improve performance
- **Connection Pooling:** Efficient database connection management
- **Automated Backups:** Point-in-time recovery with 7-day retention
- **Indexing Strategy:** Optimized indexes for search and filtering

#### Redis Cluster
**Enhanced Usage:**
- **Session Management:** Distributed session storage
- **Application Cache:** Frequently accessed data caching
- **Rate Limiting:** Distributed rate limiting counters
- **Real-time Features:** Pub/Sub for real-time notifications

#### Amazon S3 (File Storage)
**Improvements:**
- **CDN Integration:** CloudFront for faster global access
- **Lifecycle Policies:** Automatic archival of old files
- **Virus Scanning:** Automated malware detection for uploaded files
- **Versioning:** File version control for audit trails

## 4. Security Architecture

### 4.1 Authentication & Authorization
- **JWT Tokens:** Stateless authentication with refresh tokens
- **OTP Verification:** SMS/WhatsApp OTP for phone verification
- **Role-Based Access:** Separate permissions for candidates, employers, and admins
- **API Key Management:** Secure handling of third-party service keys

### 4.2 Data Protection
- **Encryption at Rest:** All databases and file storage encrypted
- **Encryption in Transit:** TLS 1.3 for all communications
- **PII Handling:** Proper anonymization and data retention policies
- **GDPR Compliance:** Right to deletion and data portability

### 4.3 Monitoring & Logging
- **Structured Logging:** JSON logs with correlation IDs
- **Real-time Monitoring:** CloudWatch dashboards and alerts
- **Error Tracking:** Sentry integration for error monitoring
- **Performance Monitoring:** APM tools for bottleneck identification

## 5. Key User Flow: Enhanced Candidate Application Process

### 5.1 Application Submission
1. **Client-side Validation:** Immediate feedback on form completion
2. **Optimistic UI:** Instant visual feedback while processing
3. **Server Processing:** 
   - Validate application data
   - Check for duplicate applications
   - Store in database with transaction safety
   - Queue background tasks
4. **Immediate Response:** Success confirmation within 200ms

### 5.2 Background Processing
1. **Queue Processing:** SQS picks up notification job
2. **CV Processing:** Extract text, validate format, generate thumbnail
3. **Employer Notification:** 
   - Retrieve employer preferences
   - Generate personalized email template
   - Send via SES with tracking
4. **Candidate Confirmation:** WhatsApp confirmation with application ID
5. **Analytics:** Track application metrics and user journey

### 5.3 Error Handling
- **Graceful Degradation:** System continues working if notifications fail
- **Retry Logic:** Failed jobs retry with exponential backoff
- **Dead Letter Queue:** Manual review of persistent failures
- **User Notification:** Inform users if critical operations fail

## 6. Technology Stack Summary

### Core Technologies
- **Frontend:** React 18 + TypeScript + PWA
- **Backend:** Node.js 18 + Express.js + TypeScript
- **Database:** PostgreSQL 15 with read replicas
- **Cache:** Redis 7 cluster
- **File Storage:** Amazon S3 with CloudFront CDN
- **Message Queue:** AWS SQS with DLQ

### Development & Operations
- **Container:** Docker with multi-stage builds
- **Deployment:** AWS ECS with Auto Scaling
- **CI/CD:** GitHub Actions with automated testing
- **Monitoring:** CloudWatch + Sentry + DataDog
- **Security:** AWS WAF + Shield + Secrets Manager

### External Services
- **Email:** Amazon SES with bounce/complaint handling
- **SMS/WhatsApp:** Twilio with fallback providers
- **Search:** PostgreSQL full-text search (upgrade to Elasticsearch later)
- **Analytics:** Custom analytics + Google Analytics

## 7. Performance & Scalability

### 7.1 Performance Targets
- **API Response Time:** < 200ms for 95% of requests
- **Page Load Time:** < 3 seconds on 3G networks
- **Database Query Time:** < 100ms for 99% of queries
- **File Upload Time:** < 5 seconds for 5MB files

### 7.2 Scalability Approach
- **Horizontal Scaling:** Auto-scaling groups (2-10 instances)
- **Database Scaling:** Read replicas for read-heavy operations
- **Cache Strategy:** Multi-level caching (Redis + CDN)
- **Queue Processing:** Multiple worker instances for background jobs

## 8. Migration Path & Future Evolution

### 8.1 Phase 1: MVP (Current)
- Monolithic architecture with enhanced reliability
- Basic search functionality
- Simple notification system
- Manual content moderation

### 8.2 Phase 2: Scale (3-6 months)
- Elasticsearch for advanced search
- Machine learning for job matching
- Automated content moderation
- Advanced analytics dashboard

### 8.3 Phase 3: Expansion (6-12 months)
- Microservices extraction (Auth, Jobs, Notifications)
- Multi-region deployment
- Advanced employer tools
- Mobile app development

## 9. Risk Mitigation

### 9.1 Technical Risks
- **Database Failures:** Automated failover with read replicas
- **Service Outages:** Multi-AZ deployment with health checks
- **Security Breaches:** Regular security audits and penetration testing
- **Performance Degradation:** Real-time monitoring with auto-scaling

### 9.2 Business Risks
- **Compliance:** Built-in GDPR and Indian data protection compliance
- **Vendor Lock-in:** Abstraction layers for easy service switching
- **Cost Overruns:** Automated cost monitoring and budget alerts
- **Data Loss:** Automated backups with tested recovery procedures

## 10. Success Metrics & Monitoring

### 10.1 Technical Metrics
- **Uptime:** 99.9% availability target
- **Performance:** Response time percentiles
- **Error Rate:** < 0.1% error rate target
- **Security:** Zero successful security breaches

### 10.2 Business Metrics
- **User Engagement:** Daily/Monthly active users
- **Conversion:** Application-to-hire ratio
- **Growth:** New user registration rate
- **Satisfaction:** User feedback scores

This enhanced HLD maintains the MVP principles while ensuring production readiness, security, and scalability for the Indian healthcare job market.