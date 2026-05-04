# Novity AWS Pipeline Architecture

## Overview

Novity pulls data from customer PI historians and processes it through an AWS pipeline to deliver predictive maintenance insights. The entire platform runs on AWS us-east-1.

## End-to-End Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CUSTOMER ENVIRONMENT (On-Premises)                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PI Historian (OSIsoft)                                                    │
│  ├─ Pressures, temperatures, flows, speeds                                │
│  ├─ Operating modes, configurations                                       │
│  └─ Historical data (6-12 months)                                         │
│                                                                             │
│  Novity Data Connector                                                     │
│  ├─ Connection Method: OPC UA / REST API                                  │
│  ├─ Authentication: Service account credentials                           │
│  ├─ Polling Interval: 5-15 minutes                                        │
│  └─ Encryption: HTTPS/TLS 1.2+                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
                    [Encrypted HTTPS over Internet]
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AWS us-east-1 (Novity Cloud Platform)                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    DATA INGESTION LAYER                             │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  API Gateway                                                        │  │
│  │  ├─ Receives HTTPS requests from data connectors                  │  │
│  │  ├─ Validates authentication (API keys/tokens)                    │  │
│  │  ├─ Rate limiting & throttling                                    │  │
│  │  └─ Routes to Lambda functions                                    │  │
│  │                                                                     │  │
│  │  Lambda (Data Validation & Normalization)                         │  │
│  │  ├─ Validates data format and quality                             │  │
│  │  ├─ Normalizes units and timestamps                               │  │
│  │  ├─ Detects missing values and outliers                           │  │
│  │  ├─ Enriches with metadata (asset ID, equipment type)             │  │
│  │  └─ Publishes to message queue                                    │  │
│  │                                                                     │  │
│  │  SQS / Kinesis (Message Queue)                                    │  │
│  │  ├─ Buffers incoming data                                         │  │
│  │  ├─ Decouples ingestion from processing                           │  │
│  │  ├─ Ensures no data loss                                          │  │
│  │  └─ Enables parallel processing                                   │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    DATA STORAGE LAYER                               │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  TimeStream (Time-Series Database)                                │  │
│  │  ├─ Optimized for time-series data                                │  │
│  │  ├─ Stores raw sensor/process data                                │  │
│  │  ├─ Automatic data retention policies                             │  │
│  │  ├─ Fast queries for trending analysis                            │  │
│  │  └─ Compression for cost efficiency                               │  │
│  │                                                                     │  │
│  │  RDS (Relational Database)                                        │  │
│  │  ├─ Stores diagnostic results                                     │  │
│  │  ├─ Stores alert history                                          │  │
│  │  ├─ Stores equipment metadata                                     │  │
│  │  ├─ Stores user data and configurations                           │  │
│  │  └─ Multi-AZ for high availability                                │  │
│  │                                                                     │  │
│  │  S3 (Object Storage)                                              │  │
│  │  ├─ Stores historical data backups                                │  │
│  │  ├─ Stores generated reports (PDF, CSV)                           │  │
│  │  ├─ Stores model artifacts and configurations                     │  │
│  │  └─ Lifecycle policies for cost optimization                      │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    AI PROCESSING LAYER                              │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  Lambda (Data Preparation)                                        │  │
│  │  ├─ Reads from message queue                                      │  │
│  │  ├─ Prepares features for ML models                               │  │
│  │  ├─ Applies feature engineering                                   │  │
│  │  └─ Publishes to processing queue                                 │  │
│  │                                                                     │  │
│  │  SageMaker (ML Model Execution)                                   │  │
│  │  ├─ Hosts pre-trained TruPrognostics models                       │  │
│  │  ├─ Runs physics-based diagnostics                                │  │
│  │  ├─ Runs ML fault classifiers                                     │  │
│  │  ├─ Generates RUL forecasts                                       │  │
│  │  ├─ Produces maintenance recommendations                          │  │
│  │  └─ Returns results to Lambda                                     │  │
│  │                                                                     │  │
│  │  EC2 (Compute for Heavy Processing)                               │  │
│  │  ├─ Runs complex diagnostic algorithms                            │  │
│  │  ├─ Performs batch processing for historical data                 │  │
│  │  ├─ Auto-scaling based on load                                    │  │
│  │  └─ Spot instances for cost optimization                          │  │
│  │                                                                     │  │
│  │  Lambda (Results Processing)                                      │  │
│  │  ├─ Receives diagnostic results                                   │  │
│  │  ├─ Formats results for storage                                   │  │
│  │  ├─ Generates alerts if thresholds exceeded                       │  │
│  │  ├─ Stores results in RDS                                         │  │
│  │  └─ Triggers notifications (SNS)                                  │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    RESULTS & ALERTS LAYER                           │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  SNS (Simple Notification Service)                                │  │
│  │  ├─ Sends email alerts to engineers                               │  │
│  │  ├─ Sends SMS alerts for critical faults                          │  │
│  │  ├─ Integrates with Slack/Teams (webhooks)                        │  │
│  │  └─ Triggers Lambda for custom actions                            │  │
│  │                                                                     │  │
│  │  RDS (Results Storage)                                            │  │
│  │  ├─ Fault diagnostics (name, confidence, RUL)                     │  │
│  │  ├─ Alert events (timestamp, severity, status)                    │  │
│  │  ├─ Trending data (efficiency over time)                          │  │
│  │  └─ Maintenance recommendations                                   │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    WEB DASHBOARD LAYER                              │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  CloudFront (CDN)                                                  │  │
│  │  ├─ Caches static assets (JS, CSS, images)                        │  │
│  │  ├─ Reduces latency for global users                              │  │
│  │  ├─ DDoS protection (AWS Shield)                                  │  │
│  │  └─ SSL/TLS termination                                           │  │
│  │                                                                     │  │
│  │  ALB (Application Load Balancer)                                  │  │
│  │  ├─ Distributes traffic across web servers                        │  │
│  │  ├─ Health checks for auto-scaling                                │  │
│  │  ├─ SSL/TLS termination                                           │  │
│  │  └─ Path-based routing                                            │  │
│  │                                                                     │  │
│  │  ECS / EC2 (Web Servers)                                          │  │
│  │  ├─ Hosts Novity web application                                  │  │
│  │  ├─ Serves dashboard UI                                           │  │
│  │  ├─ Handles user authentication                                   │  │
│  │  ├─ Auto-scaling based on traffic                                 │  │
│  │  └─ Multi-AZ deployment for HA                                    │  │
│  │                                                                     │  │
│  │  ElastiCache (Redis)                                              │  │
│  │  ├─ Caches frequently accessed data                               │  │
│  │  ├─ Speeds up dashboard queries                                   │  │
│  │  ├─ Session storage                                               │  │
│  │  └─ Real-time data updates                                        │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    API LAYER                                        │  │
│  ├─────────────────────────────────────────────────────────────────────┤  │
│  │                                                                     │  │
│  │  API Gateway (REST API)                                           │  │
│  │  ├─ Endpoints for dashboard queries                               │  │
│  │  ├─ Endpoints for report generation                               │  │
│  │  ├─ Endpoints for alert management                                │  │
│  │  ├─ Authentication via JWT tokens                                 │  │
│  │  └─ Rate limiting per customer                                    │  │
│  │                                                                     │  │
│  │  Lambda (API Handlers)                                            │  │
│  │  ├─ Queries RDS for asset health                                  │  │
│  │  ├─ Queries TimeStream for trending data                          │  │
│  │  ├─ Generates reports on-demand                                   │  │
│  │  └─ Manages user preferences                                      │  │
│  │                                                                     │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
                    [HTTPS / WebSocket over Internet]
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ENGINEER'S WORKSTATION                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Web Browser                                                               │
│  ├─ Navigates to login.novity.us                                          │
│  ├─ Authenticates with credentials                                        │
│  ├─ Views fleet health dashboard                                          │
│  ├─ Clicks on asset to see detailed diagnostics                           │
│  ├─ Receives real-time alerts                                             │
│  └─ Generates and downloads reports                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## AWS Services Used

### Data Ingestion
- **API Gateway**: Receives data from customer connectors
- **Lambda**: Validates and normalizes incoming data
- **SQS/Kinesis**: Buffers data for processing

### Data Storage
- **TimeStream**: Time-series database for sensor/process data
- **RDS**: Relational database for diagnostics, alerts, metadata
- **S3**: Object storage for backups, reports, models
- **ElastiCache**: Redis for caching and sessions

### AI/ML Processing
- **SageMaker**: Hosts and executes TruPrognostics models
- **EC2**: Compute for heavy processing and batch jobs
- **Lambda**: Orchestrates data preparation and results processing

### Web & API
- **CloudFront**: CDN for static assets
- **ALB**: Load balancing for web servers
- **ECS/EC2**: Web application servers
- **API Gateway**: REST API endpoints

### Notifications & Monitoring
- **SNS**: Email/SMS alerts
- **CloudWatch**: Logging and monitoring
- **CloudTrail**: Audit logging

### Security
- **VPC**: Network isolation
- **IAM**: Identity and access management
- **Secrets Manager**: Credential storage
- **KMS**: Encryption key management

## Data Flow Example: Real-Time Fault Detection

**Timeline:**

```
T=0:00   Customer PI historian records pressure drop
         Novity connector polls historian (OPC UA)
         
T=0:05   Data arrives at AWS API Gateway
         Lambda validates and normalizes
         Published to SQS queue
         
T=0:10   Lambda reads from SQS
         Prepares features for ML models
         Calls SageMaker endpoint
         
T=0:12   SageMaker runs TruPrognostics models
         Physics model detects anomaly
         ML classifier identifies fault type
         RUL forecast generated
         Results returned to Lambda
         
T=0:13   Lambda processes results
         Stores in RDS database
         Generates alert (confidence > 80%)
         Publishes to SNS
         
T=0:14   SNS sends email alert to engineer
         Dashboard updates in real-time
         
T=0:15   Engineer receives alert
         Logs into dashboard
         Views fault details and RUL forecast
         Schedules maintenance
```

## Scalability & Performance

### Throughput
- **Data Ingestion**: Millions of data points per day
- **Concurrent Customers**: Hundreds
- **Assets per Customer**: Up to 1,000+
- **Tags per Asset**: Up to 50+

### Latency
- **Data to Dashboard**: 5-15 minutes
- **Diagnostic Processing**: <1 minute
- **Dashboard Load**: <2 seconds
- **Alert Notification**: <5 minutes

### Availability
- **Multi-AZ Deployment**: High availability
- **Auto-Scaling**: Handles variable load
- **SLA**: 99.9% uptime
- **Disaster Recovery**: Automated backups to S3

## Security Architecture

### Data Protection
- **Encryption in Transit**: HTTPS/TLS 1.2+
- **Encryption at Rest**: AES-256 (RDS, S3, EBS)
- **Key Management**: AWS KMS

### Access Control
- **VPC**: Network isolation
- **Security Groups**: Firewall rules
- **IAM**: Role-based access control
- **API Authentication**: JWT tokens

### Compliance
- **SOC 2 Type II**: Certified
- **GDPR**: Compliant
- **Audit Logging**: CloudTrail
- **Data Residency**: us-east-1

## Cost Optimization

### Strategies
- **Spot Instances**: For batch processing
- **Reserved Instances**: For baseline compute
- **S3 Lifecycle**: Archive old data
- **Lambda**: Pay-per-execution
- **TimeStream**: Compression and retention policies

### Estimated Monthly Costs (per customer)
- **Data Ingestion**: $100-500
- **Storage**: $50-200
- **Processing**: $200-1,000
- **Web/API**: $100-300
- **Total**: $450-2,000 per customer

## Deployment & Operations

### CI/CD Pipeline
- **CodePipeline**: Orchestrates deployments
- **CodeBuild**: Builds and tests code
- **CodeDeploy**: Deploys to EC2/ECS
- **CloudFormation**: Infrastructure as code

### Monitoring & Alerting
- **CloudWatch**: Metrics and logs
- **X-Ray**: Distributed tracing
- **SNS**: Operational alerts
- **Dashboards**: Real-time visibility

## Summary

**Novity AWS Pipeline Architecture:**
1. **Ingestion**: Data flows from customer PI historians via HTTPS
2. **Validation**: Lambda validates and normalizes data
3. **Storage**: TimeStream stores time-series, RDS stores diagnostics
4. **Processing**: SageMaker runs TruPrognostics AI models
5. **Results**: Diagnostics stored, alerts generated
6. **Dashboard**: Web UI displays insights to engineers
7. **Notifications**: SNS sends alerts via email/SMS

**Key Characteristics:**
- Fully managed AWS services (serverless where possible)
- Multi-tenant architecture with data isolation
- Auto-scaling for variable load
- High availability across multiple AZs
- Enterprise-grade security and compliance
- Cost-optimized for SaaS delivery
