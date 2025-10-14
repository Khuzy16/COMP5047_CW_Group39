# Security and Privacy protection

### Operational scenario: 

A society member posts a message. Only authenticated and authorized members of that society can fetch/read that message. Non-members and unauthenticated users are prevented from accessing message content or metadata other than public society metadata (if any). 

### Requirements (verifiable statements): 

- Authentication: All message-posting and message-fetching endpoints require a valid session token or OAuth2 bearer token. Tokens must be validated on every request.
- Authorization: The server must verify membership before returning message payloads. Membership checks must be enforced server-side for every fetch/broadcast request.
- Transport security: All client-server and inter-service communication MUST use TLS 1.3 or stronger. Weak ciphers and TLS <1.3 must be disabled.
- At-rest protection: Messages persisted to storage (primary DB and backups/archives) must be encrypted with AES-256.
- Least privilege and access control: Only the message sender and verified society members may read message content. Admin roles must be explicitly defined and limited.
- Auditing: Every successful or failed message read/write must produce an audit entry with (timestamp, actor id, actor role, message id, action, source IP) retained for ≥ 90 days. Access to audit logs is role-restricted.
- Privacy minimization: Do not expose recipient lists, IP addresses, or unnecessary metadata to unauthorized parties. Redact logs where appropriate.

### Metric and targets: 

- Unauthorized access rate: 0 successful unauthorized reads during controlled system testing. In production monitor and trigger alerts if unauthorized access attempts exceed 0.01% of total access attempts in any 30-day window.
- Encryption coverage: 100% of persisted messages and backups are encrypted.
- Audit coverage: 100% of message read/write operations produce an audit entry.

### Verification:

- Automated access-control tests. Scripted attempts by non-members and unauthenticated users to fetch message endpoint. Expect HTTP 401/403 for all.
- Penetration testing and automated scans targeting token replay, horizontal privilege escalation.
- TLS configuration check. Scan for protocols/ciphers. Verify TLS 1.3
- Storage inspection to confirm AES-256 encryption at rest and verify backups are encrypted.
- Audit-log spot checks and retention tests.

---

# Performance

### Operational scenario: 

A member posts a text message (≤ 5 KB). Server persists message and broadcasts it to all currently connected society members (real-time push). Measure time between server receive and last connected recipient receiving/acknowledging the message. (Perry, Luebbe, and Bates, n.d.)

### Requirements (verifiable statements): 

- Latency definition: Measure as t_last_ack - t_server_receive, where t_server_receive is server-side receive timestamp and t_last_ack is acknowledgement time from the last currently-connected recipient.
- Throughput definition: Sustained messages/second the subsystem can process for a single society while meeting latency targets.
- Targets:
1. Normal load (≤ 100 concurrent connected members): latency ≤ 2.5 s
2. Peak load (≤ 1,000 concurrent connected members): latency ≤ 5.0 s
Throughput: sustain 200 messages/sec for a single society while meeting the peak-load latency target.
3. Precision: latency ±0.1s, throughput ±1 message/s 

### Verification: 

- Use load-testing tools that emulate connected clients (websocket/long-polling channels) and measure t_last_ack under: idle, normal, and peak scenarios. (McGahey 2025)
- Run stress tests that maintain sustained 200 msg/s while gradually increasing connected clients to 1,000. Collect percentile latencies. Requirements considered meet if 95% ≤ target latency.
- Repeat tests across different network conditions and show results.

---

# Reliability

### Operational scenario: 

Messaging subsystem must continue to operate under transient faults (node crash, network partition). Messages must not be silently lost. They should be delivered at least once (or exactly-once if implemented) with safeguards against duplicates. (Perry, Luebbe, and Bates, n.d.)

### Requirements (verifiable statements):

- Durability: Submitted messages must be durably persisted to a replicated storage layer before acknowledging receipt to sender.
- Delivery guarantee: At-least-once delivery to connected members. The system must provide deduplication/sequence IDs at the application layer if duplicates are possible.
- Replication and redundancy: Message storage must be replicated across ≥ 3 nodes.
- MTTR / availability targets: (Davidovič, n.d.)
- Message delivery success rate: ≥ 99.99% over any 30-day window.
- MTTR: ≤ 5 minutes for automated transient failovers. ≤ 60 minutes for major outages requiring manual intervention.
- Monitoring and alerts: Missed/delayed delivery, storage errors, or replication lag above thresholds must generate alerts.

### Metric and precision: 

- Message delivery success rate measured as delivered_messages / expected_deliveries per 30-day window. Precision ±0.01%.
- Mean time to Recovery (MTTR) measured in minutes with ±1 min precision. (Davidovič, n.d.)

### Verification: 

- Fault-injection tests (kill node, simulate network partition) and assert that messages are either delivered or persistently stored and delivered upon recovery.
- Run continuous monitoring in staging for at least 20 days to compute delivery success rate.
- Validate failover procedures (automatic and manual) and measure MTTR.

---

# Scalability

### Operational scenario: 

Platform user base and activity increase over time. The messaging subsystem must scale horizontally to preserve performance and reliability targets. (Barr, n.d.)

### Requirements (verifiable statements):

- Horizontal scaling: Adding an application node should increase capacity proportionally (linear scaling target), for the main processing path (message persist + broadcast). Document baseline capacity per node in staging.
- Platform target: Support 10,000 concurrently connected users across societies while meeting the performance (latency/throughput) and reliability targets above.
- Auto-scaling: Auto-scaling policy must provision and route traffic to a new instance within ≤ 5 minutes from scale-trigger to routing.
- Provisioning: Manual capacity addition must be possible in ≤ 10 minutes for standard node templates, with automated health checks and graceful connection draining.
- Operational limits and graceful degradation: On overload, system must degrade non-critical features first and preserve message delivery for members.

### Metric: 

- Support concurrent connected users across the platform while meeting performance targets (Barr, n.d.)
- Time/effort required to add capacity

### Metrics and precision:

- Concurrent users (±10 users), provisioning/scale time (±1 minute).

### Verification:

- Scale-out tests in staging: ramp simulated connections to 10k users, measure latencies and delivery success.
- Add nodes and measure incremental capacity. Compute scaling efficiency (e.g., % increase in throughput per node). Expect near-linear scaling within defined tolerance.
- Test auto-scaling trigger and measure end-to-end time to new node receiving traffic (≤ 5 min).
- Run failure/recovery tests while scaling to ensure reliability targets are preserved.