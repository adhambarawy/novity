# Step 1: Connection Setup - Detailed Breakdown

## Overview

Step 1 is where Novity establishes a secure, read-only connection to the customer's historian system. This is the foundation for all subsequent data pulls.

## Prerequisites

Before Step 1 begins, the customer must have:
- **Historian System**: OSIsoft PI System, Wonderware, GE DigitalWorks, or equivalent SCADA/historian
- **Network Access**: Historian accessible from Novity's cloud platform (or via VPN/secure gateway)
- **IT Approval**: Customer IT has approved Novity as a data consumer
- **Credentials**: Service account or API token for Novity to use

## Three Connection Methods

### Method 1: OPC UA (Most Common)

**What is OPC UA?**
- OPC UA = OLE for Process Control Unified Architecture
- Industry standard protocol for industrial data exchange
- Secure, encrypted, firewall-friendly
- Supports authentication and authorization

**How It Works:**

1. **Customer enables OPC UA server** on their historian
   - OSIsoft PI has built-in OPC UA server
   - Customer configures which tags are exposed via OPC UA
   - Sets up authentication (username/password or certificate)

2. **Novity creates OPC UA client**
   - Novity's data connector acts as an OPC UA client
   - Connects to customer's OPC UA server
   - Authenticates using provided credentials

3. **Connection Details**
   ```
   OPC UA Server Address: opc.tcp://customer-historian.company.com:4840
   Namespace: http://customer.company.com/historian
   Authentication: Username/Password or X.509 Certificate
   Encryption: TLS 1.2+
   ```

4. **Tag Subscription**
   - Novity subscribes to specific tags (e.g., "C-1402_DISCH_PRESS")
   - Historian pushes data updates to Novity when values change
   - Or Novity polls at regular intervals (5-15 min)

5. **Data Flow**
   ```
   Customer Historian (PI)
        ↓
   [OPC UA Server]
        ↓
   [Encrypted TLS Connection]
        ↓
   Novity OPC UA Client
        ↓
   Novity Cloud Platform
   ```

**Advantages:**
- Industry standard, widely supported
- Secure (TLS encryption, authentication)
- Firewall-friendly (single port)
- Supports both push and pull models

**Disadvantages:**
- Requires OPC UA server enabled on historian
- May require IT configuration

---

### Method 2: REST API

**What is REST API?**
- HTTP-based API for data retrieval
- Modern, cloud-friendly approach
- JSON data format
- Stateless requests

**How It Works:**

1. **Customer enables REST API** on historian
   - OSIsoft PI has REST API (PI Web API)
   - Customer creates API endpoint
   - Generates API token or OAuth credentials

2. **Novity makes HTTP requests**
   - Novity's connector sends HTTPS GET requests
   - Requests include authentication token
   - Historian responds with JSON data

3. **Connection Details**
   ```
   API Endpoint: https://customer-historian.company.com/piwebapi/
   Authentication: Bearer Token or API Key
   Encryption: HTTPS/TLS 1.2+
   Rate Limiting: Typically 1000 requests/hour
   ```

4. **Example Request**
   ```
   GET /piwebapi/streams/s-12345/value
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   Accept: application/json
   ```

5. **Example Response**
   ```json
   {
     "Timestamp": "2026-05-04T17:58:00Z",
     "Value": 450.5,
     "UnitsAbbreviation": "psi",
     "Good": true
   }
   ```

6. **Data Flow**
   ```
   Customer Historian (PI)
        ↓
   [REST API Server]
        ↓
   [HTTPS Requests/Responses]
        ↓
   Novity REST Client
        ↓
   Novity Cloud Platform
   ```

**Advantages:**
- Modern, cloud-native approach
- Easy to implement and debug
- Works through firewalls/proxies
- Stateless (no persistent connection)

**Disadvantages:**
- Higher latency than OPC UA
- More API calls needed (higher bandwidth)
- Rate limiting may apply

---

### Method 3: Native PI Connectors

**What is a Native PI Connector?**
- Direct connection to OSIsoft PI database
- Uses PI SDK or PI AF SDK
- Proprietary but highly optimized
- Lowest latency

**How It Works:**

1. **Novity installs PI connector** (on-premises or cloud)
   - Novity's connector uses OSIsoft PI SDK
   - Connects directly to PI server
   - Authenticates via Windows domain or PI credentials

2. **Connection Details**
   ```
   PI Server: customer-pi.company.com
   PI Database: PIServer
   Authentication: Windows domain or PI user account
   Encryption: Optional (depends on PI configuration)
   ```

3. **Data Retrieval**
   - Connector queries PI database directly
   - Retrieves tag values and timestamps
   - Supports both real-time and historical data

4. **Data Flow**
   ```
   OSIsoft PI Database
        ↓
   [PI SDK Connection]
        ↓
   Novity PI Connector
        ↓
   Novity Cloud Platform
   ```

**Advantages:**
- Lowest latency
- Most efficient (direct database access)
- Highest throughput
- Best for large tag counts

**Disadvantages:**
- Requires PI SDK installation
- May need on-premises connector (security)
- More complex setup

---

## Step-by-Step Connection Setup Process

### Phase 1: Planning (Week 1)

**Customer IT & Novity meet to decide:**

1. **Which connection method?**
   - OPC UA (most common, recommended)
   - REST API (cloud-friendly)
   - Native PI connector (high performance)

2. **Network topology**
   - Is historian accessible from internet? (cloud connection)
   - Or behind firewall? (VPN/secure gateway needed)
   - Firewall rules to allow outbound connections?

3. **Authentication method**
   - Username/password
   - API token
   - X.509 certificate
   - Windows domain account

4. **Which tags to expose?**
   - List of critical assets
   - Identify all relevant tags per asset
   - Determine read-only access scope

### Phase 2: Configuration (Week 1-2)

**Customer IT configures historian:**

1. **Enable connection method**
   - Enable OPC UA server (if using OPC UA)
   - Enable REST API (if using REST API)
   - Install PI SDK (if using native connector)

2. **Create service account**
   - Username: `novity-connector` (example)
   - Password: Strong, randomly generated
   - Permissions: Read-only access to specified tags
   - No write/delete permissions

3. **Configure firewall rules**
   - Allow outbound HTTPS (port 443) from Novity IP ranges
   - Or allow inbound connection from Novity (if historian is cloud-accessible)
   - Whitelist Novity's IP addresses or domain

4. **Generate credentials**
   - API token (if REST API)
   - Certificate (if X.509)
   - Document credentials securely

### Phase 3: Novity Configuration (Week 2)

**Novity's team configures the connector:**

1. **Create connector instance**
   - Specify connection method (OPC UA/REST/PI SDK)
   - Enter historian address/endpoint
   - Enter credentials (encrypted storage)

2. **Test connection**
   - Novity attempts to connect to historian
   - Verifies authentication works
   - Confirms read access to sample tags

3. **Configure tag mapping**
   - Map customer tag names to Novity's data model
   - Example:
     ```
     Customer Tag: "C-1402_DISCH_PRESS"
     Novity Model: "Compressor_C1402.DischargePresure"
     ```
   - Define units, scaling factors, data types

4. **Set polling parameters**
   - Polling interval: 5-15 minutes (typical)
   - Batch size: How many tags per request
   - Retry logic: What if connection fails

### Phase 4: Validation (Week 2)

**Novity validates the connection:**

1. **Live data test**
   - Pull sample data from historian
   - Verify data quality and format
   - Check timestamps and values

2. **Historical data pull**
   - Retrieve 6-12 months of historical data
   - Verify data completeness
   - Identify any gaps or anomalies

3. **Performance test**
   - Measure latency (time from historian to Novity)
   - Verify throughput (tags per second)
   - Check for connection stability

4. **Security validation**
   - Confirm encryption is enabled
   - Verify credentials are not exposed
   - Audit access logs

### Phase 5: Go Live (Week 3)

**Enable continuous data pull:**

1. **Start real-time polling**
   - Connector begins pulling data at regular intervals
   - Data flows continuously to Novity platform

2. **Monitor connection health**
   - Novity monitors for connection drops
   - Automatic reconnection on failure
   - Alerts if connection is down for >1 hour

3. **Verify data in dashboard**
   - Customer sees live data in Novity platform
   - Confirms tags are updating correctly
   - Validates data values make sense

## Security Considerations

### Authentication
- **Never use shared credentials** (use service account)
- **Rotate credentials** every 90 days
- **Use strong passwords** (16+ characters, mixed case, numbers, symbols)
- **Store credentials encrypted** (never in plain text)

### Authorization
- **Read-only access** (no write/delete permissions)
- **Scope to specific tags** (not all historian data)
- **Audit access logs** (who accessed what, when)

### Encryption
- **TLS 1.2 or higher** for all connections
- **Certificate pinning** (optional, for high security)
- **VPN or secure gateway** if historian is behind firewall

### Network
- **Firewall rules** to allow only Novity's IP ranges
- **No direct internet exposure** of historian
- **Proxy/gateway** for additional security layer

## Troubleshooting Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection refused | Firewall blocking | Whitelist Novity IP in firewall |
| Authentication failed | Wrong credentials | Verify username/password/token |
| No data received | Tags not exposed | Verify tags are in OPC UA namespace |
| High latency | Network congestion | Increase polling interval or reduce tag count |
| Connection drops | Network instability | Enable automatic reconnection, check network |

## Summary

**Step 1 Connection Setup:**
1. Choose connection method (OPC UA, REST API, or PI SDK)
2. Customer IT enables historian connection
3. Create read-only service account with credentials
4. Configure firewall rules to allow Novity access
5. Novity configures connector with historian details
6. Test connection and validate data flow
7. Enable continuous real-time data polling
8. Monitor connection health and data quality

**Timeline:** 2-3 weeks from planning to go-live

**Result:** Secure, encrypted, read-only connection established. Novity can now continuously pull data from customer's historian.
