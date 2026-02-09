# Step 12: Production Deployment and Operations

**Level:** Advanced
**Time:** ~35 minutes

In [Step 11](11-Building-Full-Applications-with-AI.md), you learned to build complete applications with AI using phased PRDs and state management. This step covers the critical transition from development to production, where AI-generated code faces real-world demands and scrutiny.

## Learning Objectives

By the end of this step, you will be able to:

1. Deploy AI-assisted applications with appropriate monitoring
2. Implement observability for AI-generated code
3. Use AI for incident response
4. Establish SRE practices for AI systems

---

## The Development-to-Production Gap

Code that works in development may fail in production due to:

1. **Scale differences:** Local testing doesn't reveal performance issues
2. **Environment differences:** Missing configurations, different dependencies
3. **Data differences:** Real data has edge cases test data doesn't
4. **Security exposure:** Production faces real attacks

**For AI-generated code, these risks are amplified** because the developer may not fully understand the implementation details.

---

## The Trust Checkpoint

Before deploying AI-assisted code to production, verify:

```
┌─────────────────────────────────────────────────────────────┐
│                 Pre-Production Checklist                    │
│                                                             │
│  UNDERSTANDING                                              │
│  □ Can explain what every module does                      │
│  □ Can trace request flow through the system               │
│  □ Know where errors are handled                           │
│                                                             │
│  SECURITY                                                   │
│  □ Security scan passed (SAST/DAST)                        │
│  □ No hardcoded secrets                                    │
│  □ Input validation on all endpoints                       │
│  □ Authentication/authorization tested                     │
│                                                             │
│  TESTING                                                    │
│  □ Unit test coverage > 80%                                │
│  □ Integration tests for critical paths                    │
│  □ Load testing completed                                  │
│  □ Edge cases documented and tested                        │
│                                                             │
│  OBSERVABILITY                                              │
│  □ Logging configured appropriately                        │
│  □ Metrics exported                                        │
│  □ Alerting rules defined                                  │
│  □ Dashboards created                                      │
│                                                             │
│  ROLLBACK                                                   │
│  □ Rollback procedure documented                           │
│  □ Database migrations are reversible                      │
│  □ Feature flags for major changes                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Four Golden Signals

Google SRE's four golden signals [^1] apply especially to AI-generated code:

[^1]: Beyer, B., et al. "Site Reliability Engineering: How Google Runs Production Systems." O'Reilly, 2016. Chapter 6: Monitoring Distributed Systems. Available at https://sre.google/sre-book/monitoring-distributed-systems/

### 1. Latency

**What to measure:**
- Request response time (p50, p95, p99)
- Database query time
- External API call time

**Why it matters for AI code:**
AI may generate functionally correct but inefficient code (N+1 queries, unnecessary loops).

```python
# Monitoring example
import time
from prometheus_client import Histogram

REQUEST_LATENCY = Histogram(
    'request_latency_seconds',
    'Request latency in seconds',
    ['endpoint', 'method']
)

@REQUEST_LATENCY.labels(endpoint='/api/users', method='GET').time()
def get_users():
    return fetch_users()
```

### 2. Traffic

**What to measure:**
- Requests per second
- Active users
- API calls by endpoint

**Why it matters:**
Traffic patterns reveal usage and help identify when AI-generated features are actually being used.

### 3. Errors

**What to measure:**
- Error rate (4xx, 5xx)
- Error types and messages
- Stack traces

**Why it matters for AI code:**
AI may not handle all edge cases. Monitoring errors reveals gaps.

```python
# Error tracking example
import sentry_sdk

sentry_sdk.init(
    dsn="your-sentry-dsn",
    traces_sample_rate=1.0,
    profiles_sample_rate=1.0,
)

try:
    process_data(user_input)
except Exception as e:
    sentry_sdk.capture_exception(e)
    logger.error(f"Processing failed: {e}")
    raise
```

### 4. Saturation

**What to measure:**
- CPU utilization
- Memory usage
- Database connections
- Queue depths

**Why it matters for AI code:**
AI may generate code that leaks resources or scales poorly.

---

## Observability Stack

### Logging Strategy

```python
# Structured logging configuration
import structlog

structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ],
)

logger = structlog.get_logger()

# Usage
logger.info(
    "user_action",
    action="login",
    user_id=user.id,
    ip_address=request.remote_addr,
    success=True
)
```

**What to log:**
- User actions (with context)
- System events
- Errors and exceptions
- Performance metrics

**What NOT to log:**
- Passwords or tokens
- Full credit card numbers
- Personal data (depending on regulations)
- Entire request/response bodies

### Metrics Collection

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'application'
    static_configs:
      - targets: ['app:8000']

  - job_name: 'database'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

### Distributed Tracing

For applications with multiple services:

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("process_order") as span:
    span.set_attribute("order_id", order_id)
    result = process_order(order_id)
    span.set_attribute("result", result.status)
```

### Complete Monitoring Stack Setup

The following configuration provides a complete observability stack:

```yaml
# docker-compose.monitoring.yml
# Complete monitoring stack for AI-assisted applications
#
# Usage:
#   docker-compose -f docker-compose.monitoring.yml up -d
#
# Access:
#   - Grafana: http://localhost:3001 (admin/admin)
#   - Prometheus: http://localhost:9090
#   - AlertManager: http://localhost:9093

version: '3.8'

services:
  # ============================================================
  # Prometheus - Metrics Collection
  # ============================================================
  prometheus:
    image: prom/prometheus:v2.47.0
    container_name: prometheus
    volumes:
      - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./monitoring/prometheus/rules/:/etc/prometheus/rules/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped

  # ============================================================
  # Grafana - Visualization
  # ============================================================
  grafana:
    image: grafana/grafana:10.2.0
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning/:/etc/grafana/provisioning/
      - ./monitoring/grafana/dashboards/:/var/lib/grafana/dashboards/
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SERVER_ROOT_URL=http://localhost:3001
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel
    ports:
      - "3001:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus
      - loki
    restart: unless-stopped

  # ============================================================
  # Loki - Log Aggregation
  # ============================================================
  loki:
    image: grafana/loki:2.9.0
    container_name: loki
    volumes:
      - ./monitoring/loki/loki-config.yml:/etc/loki/local-config.yaml
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    ports:
      - "3100:3100"
    networks:
      - monitoring
    restart: unless-stopped

  # ============================================================
  # Promtail - Log Shipping
  # ============================================================
  promtail:
    image: grafana/promtail:2.9.0
    container_name: promtail
    volumes:
      - ./monitoring/promtail/promtail-config.yml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    command: -config.file=/etc/promtail/config.yml
    networks:
      - monitoring
    depends_on:
      - loki
    restart: unless-stopped

  # ============================================================
  # AlertManager - Alert Routing
  # ============================================================
  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    volumes:
      - ./monitoring/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "9093:9093"
    networks:
      - monitoring
    restart: unless-stopped

  # ============================================================
  # Node Exporter - Host Metrics
  # ============================================================
  node-exporter:
    image: prom/node-exporter:v1.6.1
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: unless-stopped

  # ============================================================
  # cAdvisor - Container Metrics
  # ============================================================
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.47.2
    container_name: cadvisor
    privileged: true
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring
    restart: unless-stopped

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
  alertmanager_data:
```

**Prometheus Configuration:**

```yaml
# monitoring/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'ai-assisted-app'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Application metrics
  - job_name: 'application'
    static_configs:
      - targets: ['app:9090']
    metrics_path: /metrics
    scheme: http

  # Node exporter
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  # Container metrics
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Database (if using postgres_exporter)
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

**Alert Rules for AI-Generated Code:**

```yaml
# monitoring/prometheus/rules/ai-code-alerts.yml
groups:
  - name: ai-code-alerts
    rules:
      # High latency (AI code often has performance issues)
      - alert: HighLatency
        expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint)) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected on {{ $labels.endpoint }}"
          description: "p99 latency is {{ $value }}s, which may indicate AI-generated code with performance issues."

      # Error rate spike
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 5%"
          description: "Current error rate is {{ $value | humanizePercentage }}. Check recent deployments for AI-generated code issues."

      # Memory leak detection
      - alert: PotentialMemoryLeak
        expr: increase(process_resident_memory_bytes[1h]) > 100000000
        for: 30m
        labels:
          severity: warning
        annotations:
          summary: "Potential memory leak detected"
          description: "Memory usage increased by {{ $value | humanize1024 }} in the last hour. AI-generated code may have resource leaks."

      # Database connection exhaustion
      - alert: DatabaseConnectionsHigh
        expr: pg_stat_activity_count / pg_settings_max_connections > 0.8
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Database connections at {{ $value | humanizePercentage }}"
          description: "Connection pool nearly exhausted. Check for connection leaks in AI-generated code."

      # Deployment-correlated issues
      - alert: PostDeploymentIssues
        expr: (sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) > 0.01 and changes(deployment_timestamp[10m]) > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Issues detected after recent deployment"
          description: "Error rate elevated following deployment. Review AI-generated changes in recent commit."
```

**AlertManager Configuration:**

```yaml
# monitoring/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: '${SLACK_WEBHOOK_URL}'

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
      continue: true
    - match:
        severity: warning
      receiver: 'warning-alerts'

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }}: {{ .CommonLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'critical-alerts'
    slack_configs:
      - channel: '#critical-alerts'
        send_resolved: true
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SERVICE_KEY}'

  - name: 'warning-alerts'
    slack_configs:
      - channel: '#warnings'
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname']
```

---

## AI-Assisted Incident Response

### Incident Response Flow with AI SRE

The following diagram illustrates how AI SRE integrates into incident response:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INCIDENT RESPONSE FLOW WITH AI SRE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────┐                                                          │
│  │    ALERT      │                                                          │
│  │   TRIGGERED   │                                                          │
│  └───────┬───────┘                                                          │
│          │                                                                   │
│          ▼                                                                   │
│  ┌───────────────────────────────────────────────────────────────────┐     │
│  │                   AI SRE FIRST RESPONDER                          │     │
│  │                                                                    │     │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐               │     │
│  │  │   TRIAGE    │  │  DIAGNOSE   │  │   DECIDE    │               │     │
│  │  │             │  │             │  │             │               │     │
│  │  │ • Severity  │─►│ • Logs      │─►│ • Runbook   │               │     │
│  │  │ • Impact    │  │ • Metrics   │  │   exists?   │               │     │
│  │  │ • Scope     │  │ • Recent    │  │ • Confident?│               │     │
│  │  │             │  │   deploys   │  │ • Reversible│               │     │
│  │  └─────────────┘  └─────────────┘  └──────┬──────┘               │     │
│  │                                           │                       │     │
│  └───────────────────────────────────────────│───────────────────────┘     │
│                                              │                              │
│               ┌──────────────────────────────┼──────────────────────────┐  │
│               │                              │                          │  │
│               ▼                              ▼                          ▼  │
│     ┌─────────────────┐            ┌─────────────────┐      ┌─────────────┐
│     │  AUTO-REMEDIATE │            │    ESCALATE     │      │   MONITOR   │
│     │                 │            │                 │      │   (watch)   │
│     │ If: Runbook     │            │ If: High        │      │             │
│     │     exists      │            │     severity    │      │ If: Low     │
│     │     & confident │            │  OR uncertain   │      │    impact   │
│     │     & reversible│            │  OR no runbook  │      │  & stable   │
│     │                 │            │                 │      │             │
│     │ Actions:        │            │ Actions:        │      │ Actions:    │
│     │ • Restart svc   │            │ • Page on-call  │      │ • Log event │
│     │ • Scale up      │            │ • Prepare context│     │ • Trend     │
│     │ • Rollback      │            │ • Hand off      │      │   analysis  │
│     │ • Failover      │            │                 │      │             │
│     └────────┬────────┘            └────────┬────────┘      └─────────────┘
│              │                              │                               │
│              │                              ▼                               │
│              │                   ┌─────────────────────┐                   │
│              │                   │   HUMAN ENGINEER    │                   │
│              │                   │                     │                   │
│              │                   │ • Reviews AI triage │                   │
│              │                   │ • Makes decisions   │                   │
│              │                   │ • Executes complex  │                   │
│              │                   │   remediations      │                   │
│              │                   └──────────┬──────────┘                   │
│              │                              │                               │
│              └──────────────┬───────────────┘                              │
│                             ▼                                               │
│                   ┌─────────────────┐                                      │
│                   │    RESOLVED     │                                      │
│                   │                 │                                      │
│                   │ AI documents:   │                                      │
│                   │ • Timeline      │                                      │
│                   │ • Actions taken │                                      │
│                   │ • Root cause    │                                      │
│                   └─────────────────┘                                      │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  MTTR REDUCTION: AI handles initial triage in seconds, not minutes         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Using AI During Incidents

AI can accelerate incident response:

```
INCIDENT: Users reporting slow page loads

Please help diagnose this incident:

1. Check the metrics dashboards:
   - What does latency look like (p50, p95, p99)?
   - Any spike in error rates?
   - CPU/memory saturation?

2. Check recent deployments:
   - What changed in the last 24 hours?
   - Any new features launched?

3. Analyze logs:
   - Search for slow query warnings
   - Look for error patterns
   - Check for resource exhaustion

4. Propose investigation steps in priority order
```

### AI SRE Patterns

**Pattern 1: Automated Diagnosis**
```
Given these metrics and logs, identify the most likely root cause:

Metrics:
- p99 latency increased from 200ms to 2000ms at 14:30 UTC
- Database connection pool at 100% capacity
- CPU normal, memory normal

Logs:
- 14:25 UTC: Deploy started
- 14:30 UTC: First slow request warnings
- 14:35 UTC: "Connection pool exhausted" errors begin

Analyze the timeline and suggest root cause.
```

**Pattern 2: Runbook Execution**
```
We're seeing elevated error rates on the payment service.

Execute our payment-errors runbook:
1. Check payment provider status page
2. Review recent payment-related commits
3. Check for data migration issues
4. Verify API keys and credentials are valid

Report findings at each step.
```

**Pattern 3: Impact Assessment**
```
We need to restart the database to apply a critical patch.

Assess the impact:
1. What services depend on this database?
2. What's the current traffic to those services?
3. Is there a read replica we can fail over to?
4. What's the expected downtime?
5. Who should be notified?
```

---

## AI SRE Architecture

### AI SRE Architecture Diagram

The following diagram illustrates the AI SRE architecture for production operations:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AI SRE ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                    PRODUCTION ENVIRONMENT                             │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │  │
│  │  │   Web App   │  │   API Svc   │  │  Database   │  │    Queue    │ │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │  │
│  │         │                │                │                │        │  │
│  │         └────────────────┼────────────────┼────────────────┘        │  │
│  │                          │                │                          │  │
│  │                          ▼                ▼                          │  │
│  │              ┌─────────────────────────────────────┐                │  │
│  │              │        METRICS + LOGS + TRACES      │                │  │
│  │              │    (Prometheus, DataDog, OpenTelemetry)              │  │
│  │              └──────────────────┬──────────────────┘                │  │
│  └────────────────────────────────│────────────────────────────────────┘  │
│                                    │                                       │
│                                    ▼                                       │
│            ┌───────────────────────────────────────────────────────┐      │
│            │                   AI SRE AGENT                         │      │
│            │                                                        │      │
│            │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │      │
│            │  │   MONITOR    │  │    DETECT    │  │   RESPOND    │ │      │
│            │  │              │  │              │  │              │ │      │
│            │  │ • Metrics    │  │ • Anomaly    │  │ • Execute    │ │      │
│            │  │ • Logs       │  │   detection  │  │   runbooks   │ │      │
│            │  │ • Alerts     │  │ • Pattern    │  │ • Scale/     │ │      │
│            │  │ • Trends     │  │   matching   │  │   restart    │ │      │
│            │  └──────────────┘  └──────────────┘  └──────────────┘ │      │
│            │                                                        │      │
│            │              ┌──────────────────────┐                  │      │
│            │              │   DECISION ENGINE    │                  │      │
│            │              │                      │                  │      │
│            │              │  Confidence > 95%? ──┼──► Auto-action   │      │
│            │              │  Confidence < 95%? ──┼──► Escalate      │      │
│            │              │  Unknown situation?──┼──► HUMAN         │      │
│            │              └──────────────────────┘                  │      │
│            └─────────────────────────┬─────────────────────────────┘      │
│                                      │                                     │
│              ┌───────────────────────┼───────────────────────┐            │
│              ▼                       ▼                       ▼             │
│    ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│    │  AUTO ACTIONS   │     │   ESCALATION    │     │  DOCUMENTATION  │   │
│    │                 │     │                 │     │                 │   │
│    │ • Restart svc   │     │ • PagerDuty     │     │ • Timeline      │   │
│    │ • Scale up      │     │ • Slack alert   │     │ • Actions taken │   │
│    │ • Failover      │     │ • On-call       │     │ • Diagnostics   │   │
│    │ • Rollback      │     │   engineer      │     │ • Post-mortem   │   │
│    └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  BOUNDARY: AI acts autonomously within predefined limits; escalates beyond │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The AI SRE Concept

AI SRE agents can provide continuous monitoring and first-responder capabilities:

**Responsibilities:**
- Monitor metrics continuously (not just when alerts fire)
- Detect anomalies before they become incidents
- Execute runbooks for known issues
- Escalate to humans when judgment is required
- Document actions and maintain incident timelines

### AI SRE Operational Boundaries

Define clear boundaries for what AI SRE can and cannot do:

**AI Can Autonomously:**
- Restart services (with rate limits)
- Scale up resources within predefined limits
- Execute approved runbook steps
- Gather diagnostic information
- Update incident documentation

**AI Must Escalate:**
- Any action that could cause data loss
- Customer data access or modification
- Actions outside predefined runbooks
- High-severity incidents (P1, P2)
- Situations where diagnosis is uncertain
- Any action with irreversible consequences

### Runbook Structure for AI Execution

Structure runbooks so AI can execute them safely:

**Runbook Format:**
```
RUNBOOK: High Error Rate on Payment Service

TRIGGER: Error rate > 5% for 5 minutes

PRE-CONDITIONS:
- Verify this is not a known maintenance window
- Check deployment history for recent changes

DIAGNOSTIC STEPS:
1. Check payment provider status page
2. Review error logs for patterns
3. Check database connection status
4. Verify API credentials validity

REMEDIATION OPTIONS:
- If provider is down: Enable failover, notify customers
- If recent deployment: Rollback candidate
- If connection issue: Restart connection pool
- If unknown: Escalate to human

POST-ACTIONS:
- Update incident timeline
- Notify relevant Slack channel
- Create post-mortem ticket if P1/P2

ESCALATION TRIGGERS:
- Unable to diagnose within 10 minutes
- Multiple services affected
- Customer data potentially impacted
- All remediation options exhausted
```

### Escalation Criteria

AI SRE should always escalate when:

| Condition | Reason |
|-----------|--------|
| Data loss possible | Irreversible consequences |
| Customer data affected | Privacy and compliance |
| Diagnosis uncertain after 10min | Human judgment needed |
| Multiple services impacted | Complex coordination needed |
| P1 or P2 severity | High business impact |
| Outside normal remediation | Unknown territory |

### Human-AI Collaboration in Incidents

The handoff between AI and human should be smooth:

**AI-to-Human Handoff:**
1. AI provides complete incident summary
2. Actions already taken are documented
3. Current hypothesis and evidence listed
4. Recommended next steps proposed
5. Relevant dashboards and logs linked

**Human-to-AI Delegation:**
1. Human provides specific task
2. AI executes and reports back
3. Human maintains decision authority
4. AI documents all actions for timeline

### Complete AI SRE Agent System Prompt

Use this system prompt to configure an AI SRE agent for continuous monitoring:

```markdown
# AI SRE Agent System Prompt

## Identity and Role

You are an AI Site Reliability Engineering (SRE) agent responsible for monitoring
production systems and responding to incidents. You operate autonomously for
routine issues and escalate to humans for complex or high-risk situations.

## Core Responsibilities

### 1. Continuous Monitoring
- Check metrics dashboards every 5 minutes
- Monitor the four golden signals:
  * **Latency**: p50, p95, p99 response times
  * **Traffic**: Requests per second, active users
  * **Errors**: 4xx/5xx rates, error types
  * **Saturation**: CPU, memory, connections, queue depths

### 2. Anomaly Detection
- Compare current metrics to historical baselines
- Flag deviations > 2 standard deviations
- Correlate anomalies across services
- Track trends that may indicate future problems

### 3. Alert Response
When an alert fires, follow this protocol:

1. **Acknowledge** the alert immediately
2. **Gather context**:
   - Recent deployments (last 24 hours)
   - Configuration changes
   - Related alerts
   - Metric history
3. **Analyze logs** for error patterns
4. **Diagnose** using runbooks
5. **Remediate** if within your authority
6. **Escalate** if needed
7. **Document** all actions

## Runbooks

### Runbook: High Latency (p99 > 2s)

**Diagnostic Steps:**
1. Check database query times in slow query log
2. Check external API response times
3. Check for recent deployments
4. Check resource saturation (CPU, memory, connections)
5. Check for traffic spikes

**Remediation Options:**
- If DB queries slow: Check for missing indexes, long-running queries
- If external API slow: Check provider status, consider circuit breaker
- If resource exhaustion: Scale up pods (within limits)
- If recent deployment: Consider rollback

**Escalate if:**
- Unable to identify cause within 10 minutes
- Latency affects > 50% of users
- Problem persists after initial remediation

### Runbook: High Error Rate (> 5%)

**Diagnostic Steps:**
1. Identify error types from logs
2. Check if errors correlate with specific endpoints
3. Review recent deployments
4. Check database connectivity
5. Verify external dependencies

**Remediation Options:**
- If 5xx from specific endpoint: Check code, recent changes
- If database errors: Check connection pool, credentials
- If external service down: Activate fallback/cache
- If related to deployment: Rollback candidate

**Escalate if:**
- Errors involve data integrity
- Unable to diagnose
- Errors affecting payments or auth

### Runbook: Resource Exhaustion

**Diagnostic Steps:**
1. Identify exhausted resource
2. Check for memory leaks (increasing trend)
3. Check for connection leaks
4. Review pod resource limits
5. Check for traffic increase

**Remediation Options:**
- Scale horizontally (add pods)
- Increase resource limits (within budget)
- Restart pods if memory leak suspected
- Enable request shedding if overloaded

**Escalate if:**
- Scaling not effective
- Suspected code issue
- Budget implications

## Autonomous Actions (Within Authority)

You MAY autonomously:
- Scale pods: 2-10 replicas for stateless services
- Restart pods showing symptoms of memory leaks
- Clear caches if stale data suspected
- Enable/disable feature flags for known issues
- Execute approved runbook remediations
- Update incident documentation
- Notify on-call of non-critical issues

## Escalation Requirements (Must Notify Human)

You MUST escalate when:
- Any action could cause data loss
- Customer PII potentially affected
- Unable to diagnose within 10 minutes
- Multiple services affected simultaneously
- Severity is P1 or P2
- Remediation requires code changes
- Action is outside approved runbooks
- Financial systems affected
- Security incident suspected

## Communication Protocol

### Incident Updates
Provide updates to Slack #incidents channel every 5 minutes during active incidents:

```
INCIDENT UPDATE - [timestamp]
Status: Investigating / Mitigating / Resolved
Impact: [affected users/services]
Current action: [what you're doing]
Next step: [planned action]
```

### Escalation Format
When escalating to human on-call:

```
ESCALATION REQUIRED

Severity: P1/P2/P3
Service: [affected service]
Impact: [user impact]
Duration: [time since start]

Summary: [1-2 sentence description]

Actions Taken:
- [list of actions]

Why Escalating:
- [specific reason]

Recommended Next Steps:
1. [suggestion]
2. [suggestion]

Dashboards:
- [link to relevant dashboard]
- [link to logs]
```

### Handoff Protocol
When handing off to human:
1. Provide complete incident summary
2. List all actions taken with timestamps
3. Share current hypothesis
4. Link relevant dashboards and logs
5. Specify recommended next steps
6. Remain available to assist

## Context and Constraints

### System Architecture
- Frontend: Next.js on Vercel
- API: Node.js on Kubernetes (GKE)
- Database: PostgreSQL on Cloud SQL
- Cache: Redis on Memorystore
- Queue: Cloud Pub/Sub

### SLOs
- Availability: 99.9% (43 minutes downtime/month)
- Latency: p95 < 500ms, p99 < 2s
- Error rate: < 0.1% for critical paths

### On-Call Contacts
- Primary: Check PagerDuty schedule
- Secondary: [backup contact]
- Engineering Lead: [for P1 only]

## Audit Trail

Log all actions with:
- Timestamp
- Action taken
- Reason
- Outcome
- Related incident ID

## Remember

1. **Safety first**: When in doubt, escalate
2. **Document everything**: Future you (and humans) need the trail
3. **Be conservative**: Prefer reversible actions
4. **Stay focused**: One incident at a time
5. **Communicate proactively**: No one likes surprises
```

**Using the AI SRE Agent:**

This prompt can be used with:
- **Scheduled jobs**: Run every 5 minutes to check metrics
- **Alert webhooks**: Trigger on Prometheus/PagerDuty alerts
- **Chat integration**: Allow on-call to query the agent

**Example Integration:**

```python
# ai_sre_agent.py
import anthropic
from datetime import datetime

SYSTEM_PROMPT = """[The AI SRE Agent System Prompt above]"""

class AISREAgent:
    def __init__(self):
        self.client = anthropic.Anthropic()
        self.incident_context = []

    def process_alert(self, alert: dict) -> str:
        """Process an incoming alert."""
        prompt = f"""
        An alert has fired. Please respond according to your runbooks.

        Alert Details:
        - Name: {alert['name']}
        - Severity: {alert['severity']}
        - Description: {alert['description']}
        - Labels: {alert['labels']}
        - Started: {alert['started_at']}

        Current Metrics:
        {self.get_current_metrics()}

        Recent Logs:
        {self.get_recent_logs(alert['service'])}

        Recent Deployments:
        {self.get_recent_deployments()}

        Respond with your diagnosis and planned actions.
        """

        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            system=SYSTEM_PROMPT,
            messages=[
                {"role": "user", "content": prompt}
            ]
        )

        return response.content[0].text

    def get_current_metrics(self) -> str:
        # Query Prometheus for current metrics
        pass

    def get_recent_logs(self, service: str) -> str:
        # Query Loki for recent logs
        pass

    def get_recent_deployments(self) -> str:
        # Query deployment history
        pass
```

---

## Kubernetes Deployment Patterns

### Production-Ready Configuration

For AI-generated applications deployed to Kubernetes, ensure these patterns are followed:

**Resource Management:**
- Define resource requests and limits
- Use horizontal pod autoscaling
- Configure pod disruption budgets

**Health Checks:**
- Liveness probes detect when to restart
- Readiness probes control traffic routing
- Startup probes for slow-starting containers

**Configuration Management:**
- Secrets for sensitive data (never in code)
- ConfigMaps for non-sensitive configuration
- Environment-specific overlays

### Key Kubernetes Considerations

**1. Health Check Design**

Health checks should be meaningful:
- Liveness: Can the process respond at all?
- Readiness: Can it handle requests correctly?
- Don't make health checks dependent on external services

**2. Graceful Shutdown**

Handle termination signals properly:
- Stop accepting new requests
- Finish processing in-flight requests
- Close database connections cleanly
- Exit with appropriate code

**3. Resource Right-Sizing**

Start conservative, then optimize:
- Begin with generous limits
- Monitor actual usage
- Adjust based on real data
- Leave headroom for spikes

**4. Auto-Scaling Configuration**

Configure HPA (Horizontal Pod Autoscaler) appropriately:
- Scale on CPU, memory, or custom metrics
- Set appropriate min/max replicas
- Configure scale-down delay to prevent flapping
- Test scaling behavior under load

### Common Kubernetes Issues with AI Code

| Issue | Symptom | Prevention |
|-------|---------|------------|
| No health checks | Unhealthy pods receive traffic | Always define probes |
| Missing resource limits | Pods consume unbounded resources | Always set limits |
| Hardcoded URLs | Fails across environments | Use ConfigMaps/env vars |
| No graceful shutdown | Requests dropped during deploy | Handle SIGTERM |
| Single replica | No high availability | Set minReplicas > 1 |

### Complete Kubernetes Deployment Configuration

The following configuration provides a production-ready deployment for AI-assisted applications:

```yaml
# kubernetes/deployment.yaml
# Complete Kubernetes deployment for AI-assisted applications
# Includes deployment, service, HPA, PDB, and supporting resources
#
# Usage:
#   kubectl apply -f kubernetes/
#
# Or with Kustomize:
#   kubectl apply -k kubernetes/overlays/production

---
# ConfigMap for application configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: ai-app-config
  labels:
    app: ai-assisted-app
    environment: production
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  METRICS_PORT: "9090"
  ENABLE_TRACING: "true"
  # AI-specific settings
  AI_CODE_MONITORING: "enabled"
  FEATURE_FLAGS_ENABLED: "true"

---
# Secret reference (actual secrets managed externally)
apiVersion: v1
kind: Secret
metadata:
  name: ai-app-secrets
  labels:
    app: ai-assisted-app
type: Opaque
stringData:
  # These should be managed via external secrets operator or sealed secrets
  DATABASE_URL: "postgresql://user:password@db:5432/app"
  REDIS_URL: "redis://redis:6379"
  API_KEY: "your-api-key-here"

---
# Main Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app
    version: v1
  annotations:
    # Track AI-generated code deployments
    ai-assisted-code/version: "2026.02.08"
    ai-assisted-code/review-required: "true"
spec:
  replicas: 3
  revisionHistoryLimit: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: ai-assisted-app
  template:
    metadata:
      labels:
        app: ai-assisted-app
        version: v1
      annotations:
        # Prometheus scraping
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
        # Restart on config changes
        checksum/config: "{{ include (print $.Template.BasePath \"/configmap.yaml\") . | sha256sum }}"
    spec:
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000

      # Service account with minimal permissions
      serviceAccountName: ai-assisted-app

      # Topology spread for high availability
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: ai-assisted-app

      # Init container for database migrations
      initContainers:
        - name: migrate
          image: your-registry/ai-assisted-app:latest
          command: ["npm", "run", "db:migrate"]
          envFrom:
            - configMapRef:
                name: ai-app-config
            - secretRef:
                name: ai-app-secrets
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"

      containers:
        - name: app
          image: your-registry/ai-assisted-app:latest
          imagePullPolicy: Always

          ports:
            - name: http
              containerPort: 3000
              protocol: TCP
            - name: metrics
              containerPort: 9090
              protocol: TCP

          # Environment configuration
          envFrom:
            - configMapRef:
                name: ai-app-config
            - secretRef:
                name: ai-app-secrets

          # Resource management (critical for AI-generated code)
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"

          # Health checks
          startupProbe:
            httpGet:
              path: /health/startup
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 30

          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 0
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 0
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

          # Security context
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

          # Volume mounts
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/.cache

          # Graceful shutdown
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 10"]

      # Volumes
      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir:
            sizeLimit: 500Mi

      # Termination grace period
      terminationGracePeriodSeconds: 60

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app
spec:
  type: ClusterIP
  selector:
    app: ai-assisted-app
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
    - name: metrics
      port: 9090
      targetPort: metrics
      protocol: TCP

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-assisted-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
    # Scale on CPU
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    # Scale on memory
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    # Scale on custom metrics (request rate)
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max

---
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: ai-assisted-app

---
# Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app

---
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ai-assisted-app
  labels:
    app: ai-assisted-app
spec:
  podSelector:
    matchLabels:
      app: ai-assisted-app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
        - podSelector:
            matchLabels:
              app: prometheus
      ports:
        - protocol: TCP
          port: 3000
        - protocol: TCP
          port: 9090
  egress:
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: TCP
          port: 5432  # PostgreSQL
        - protocol: TCP
          port: 6379  # Redis
        - protocol: TCP
          port: 443   # External HTTPS
```

**Health Check Endpoints Implementation:**

```typescript
// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';
import { RedisHealthIndicator } from './redis.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private redis: RedisHealthIndicator,
  ) {}

  @Get('startup')
  @HealthCheck()
  checkStartup() {
    // Basic check - app started successfully
    return { status: 'ok' };
  }

  @Get('live')
  @HealthCheck()
  checkLive() {
    // Liveness - can the process respond at all?
    // Don't include external dependencies
    return { status: 'ok', timestamp: new Date().toISOString() };
  }

  @Get('ready')
  @HealthCheck()
  checkReady() {
    // Readiness - can we serve traffic?
    // Include critical dependencies
    return this.health.check([
      () => this.db.pingCheck('database', { timeout: 1500 }),
      () => this.redis.check('redis'),
    ]);
  }
}
```

**Graceful Shutdown Implementation:**

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable graceful shutdown
  app.enableShutdownHooks();

  // Handle shutdown signals
  const signals = ['SIGTERM', 'SIGINT'];
  signals.forEach(signal => {
    process.on(signal, async () => {
      console.log(`Received ${signal}, starting graceful shutdown...`);

      // Stop accepting new connections
      // (Kubernetes removes from service endpoints)

      // Wait for in-flight requests (preStop hook gives us time)
      await new Promise(resolve => setTimeout(resolve, 5000));

      // Close database connections
      await app.close();

      console.log('Graceful shutdown complete');
      process.exit(0);
    });
  });

  await app.listen(3000);
}
bootstrap();
```

---

## Deployment Strategies

### Feature Flags

Control rollout of AI-generated features:

```python
from unleash import UnleashClient

unleash_client = UnleashClient(
    url="https://unleash.example.com/api/",
    app_name="my-app"
)

def get_recommendations(user):
    if unleash_client.is_enabled("ai-recommendations"):
        return ai_recommendations(user)
    else:
        return legacy_recommendations(user)
```

### Canary Deployments

Gradually roll out to detect issues early:

```yaml
# kubernetes canary deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: {duration: 10m}
        - setWeight: 30
        - pause: {duration: 10m}
        - setWeight: 50
        - pause: {duration: 10m}
```

### Rollback Procedures

Document and practice rollbacks:

```markdown
## Rollback Procedure

### Immediate Rollback (< 5 min)
1. Run: `kubectl rollout undo deployment/my-app`
2. Verify: `kubectl rollout status deployment/my-app`
3. Confirm: Check error rate returning to baseline

### Database Rollback (if migrations involved)
1. Identify last good migration: `npx prisma migrate status`
2. Rollback: `npx prisma migrate reset --to <migration_id>`
3. Verify: Run health checks

### Feature Flag Rollback
1. Disable flag: `unleash.setEnabled("feature-name", false)`
2. No deployment needed
3. Immediate effect
```

---

## Monitoring AI-Specific Issues

### Code Quality Drift

Monitor for quality issues that emerge over time:

```
Weekly Quality Report:

1. Run security scan and compare to last week
2. Check test coverage trends
3. Measure technical debt metrics
4. Review error log patterns

Flag any degradation for investigation.
```

### Unexpected Behavior Patterns

```python
# Monitor for anomalies in AI-generated code behavior
from anomaly_detector import AnomalyDetector

detector = AnomalyDetector(baseline_days=30)

# Check if current behavior matches historical patterns
if detector.is_anomaly(metric="response_time", value=current_latency):
    alert("Response time anomaly detected")

if detector.is_anomaly(metric="error_rate", value=current_errors):
    alert("Error rate anomaly detected")
```

### Resource Usage Patterns

AI-generated code may have unexpected resource characteristics:

```
Monitor for these patterns:

1. Memory leaks: Gradual increase in memory usage
2. Connection leaks: Growing number of open connections
3. File handle leaks: Increasing open file count
4. CPU spikes: Periodic computation bursts

Alert if any metric exceeds 2 standard deviations from normal.
```

---

## Post-Incident Analysis

### Incident Review Template

```markdown
## Incident Review: [Title]

### Summary
- Start time: [timestamp]
- End time: [timestamp]
- Duration: [minutes]
- Severity: [P1/P2/P3]
- Impact: [user-facing impact]

### Timeline
| Time | Event |
|------|-------|
| 14:00 | First alert fired |
| 14:05 | On-call acknowledged |
| 14:15 | Root cause identified |
| 14:30 | Fix deployed |
| 14:35 | Metrics recovered |

### Root Cause
[Detailed explanation of what went wrong]

### AI-Generated Code Involvement
- Was AI-generated code involved? [Yes/No]
- Which components? [List]
- Was the issue due to AI code specifically? [Yes/No]
- What could have caught this before production? [Analysis]

### Action Items
| Item | Owner | Due Date |
|------|-------|----------|
| Add test for edge case | @engineer | 2026-02-15 |
| Improve monitoring | @sre | 2026-02-12 |
| Update AI prompts to prevent | @team | 2026-02-20 |

### Lessons Learned
- What worked well?
- What could be improved?
- How do we prevent this class of issues?
```

---

## Complete Incident Case Study

### Incident: Payment Validation Failure

**Executive Summary:**
- **Severity:** P2
- **Duration:** 45 minutes (14:00 - 14:45 UTC)
- **Impact:** 15% of users experienced checkout failures
- **Root Cause:** AI-generated validation code rejected valid credit cards

### Timeline

| Time (UTC) | Event |
|------------|-------|
| 14:00 | Alert: Checkout error rate exceeds 5% |
| 14:05 | On-call acknowledged, began investigation |
| 14:10 | Identified errors in payment validation |
| 14:15 | Found recent deployment with AI-generated code |
| 14:20 | Reviewed AI code, found overly strict regex |
| 14:25 | Decision point: rollback vs hotfix |
| 14:30 | Deployed hotfix with corrected validation |
| 14:35 | Error rate decreasing |
| 14:45 | Error rate back to baseline, incident resolved |

### Root Cause Analysis

**What Happened:**
The credit card validation function was regenerated by AI during a refactoring task. The new regex rejected cards with spaces (which the UI allows) and certain American Express formats.

**AI Code Involvement:**
- Component affected: `src/payments/validation.ts`
- AI prompt used: "Refactor the validation module to reduce code duplication"
- Code was reviewed but reviewer missed edge cases
- Tests existed but didn't cover formatted card numbers

**The Problematic Code:**

The AI generated a "cleaner" validation that was more strict than intended:
- Original: Accepted cards with spaces, dashes, various formats
- AI version: Required exact 16-digit format without separators
- AI reasoning was sound (standardization) but context was missing

### Why Existing Safeguards Failed

| Safeguard | Why It Didn't Catch This |
|-----------|-------------------------|
| Unit tests | Only tested unformatted numbers |
| Code review | Reviewer focused on logic, not edge cases |
| Integration tests | Used ideal test data |
| Security scan | Not a security issue |
| Type checking | Types were correct |

### Action Items

| Action | Owner | Status |
|--------|-------|--------|
| Add formatted card number test cases | @engineer | Done |
| Add property-based tests for validation | @engineer | In Progress |
| Update AI prompts for validation changes | @tech-lead | Pending |
| Add "AI-assisted" label to PRs | @team | Done |
| Review all recent AI-generated validation | @security | Pending |
| Sample production data for test suite | @qa | Planned |

### Lessons Learned

**What Worked Well:**
- Fast detection via monitoring (5 min to alert)
- Quick diagnosis once investigating
- Hotfix deployed in 15 minutes
- Clear incident communication

**What Could Be Improved:**
- AI-generated validation code needs edge case checklist
- Test cases should include real-world data formats
- Property-based testing for format validation
- Prompts should specify backward compatibility requirements

### Changes to Prevent Recurrence

1. **Test Suite Enhancement:**
   - Add formatted input test cases
   - Include property-based tests for validation logic
   - Sample anonymized production data for test fixtures

2. **Review Process:**
   - AI-assisted PRs touching validation require extra review
   - Checklist for validation changes includes format examples

3. **Prompt Engineering:**
   - Validation prompts must specify existing format acceptance
   - Include "preserve backward compatibility" instruction
   - Require explicit test cases for edge cases

4. **Documentation:**
   - Added CLAUDE.md rule about validation changes
   - Documented accepted credit card formats

### Complete Incident Post-Mortem Template

Use this template for all production incidents, with special attention to AI-generated code involvement:

```markdown
<!-- post-mortems/YYYY-MM-DD-incident-title.md -->

# Incident Post-Mortem: [Descriptive Title]

## Executive Summary

| Field | Value |
|-------|-------|
| **Incident ID** | INC-2026-0042 |
| **Date** | 2026-02-08 |
| **Severity** | P2 - Major |
| **Duration** | 45 minutes (14:00 - 14:45 UTC) |
| **Impact** | 15% of users experienced checkout failures |
| **Detection** | Automated alerting (Prometheus) |
| **Root Cause** | AI-generated validation code rejected valid inputs |

**One-line summary:** AI-generated credit card validation regex was more
restrictive than the previous implementation, rejecting cards with spaces.

---

## Impact Assessment

### User Impact
- **Users Affected:** ~2,500 (15% of active users during incident)
- **Transactions Failed:** 342 checkout attempts
- **Revenue Impact:** Estimated $8,500 in delayed/abandoned purchases
- **Support Tickets:** 47 tickets opened

### Service Impact
- **Services Affected:** Checkout Service, Payment Gateway
- **SLO Violation:** Error rate SLO breached (0.1% target, actual 5.2%)
- **Data Loss:** None
- **Security Impact:** None

### Customer Communication
- Status page updated at 14:10 UTC
- Email sent to affected customers at 15:30 UTC with apology and retry instructions

---

## Timeline

| Time (UTC) | Event | Actor |
|------------|-------|-------|
| 13:45 | Deployment of PR #1234 completed | CI/CD |
| 14:00 | Alert: Checkout error rate > 5% | Prometheus |
| 14:02 | Alert acknowledged | AI SRE Agent |
| 14:05 | Initial triage: payment validation errors identified | AI SRE Agent |
| 14:08 | Escalation to on-call engineer | AI SRE Agent |
| 14:10 | On-call begins investigation | @engineer |
| 14:15 | Root cause identified: regex in validation.ts | @engineer |
| 14:18 | Recent deployment identified as suspect | @engineer |
| 14:20 | Code review confirms AI-generated regex issue | @engineer |
| 14:22 | Decision: hotfix vs rollback | @engineer, @tech-lead |
| 14:25 | Hotfix PR created and approved | @engineer |
| 14:30 | Hotfix deployed | CI/CD |
| 14:35 | Error rate declining | Prometheus |
| 14:40 | Error rate back to baseline | Prometheus |
| 14:45 | Incident resolved, monitoring continues | @engineer |
| 15:00 | All-clear declared | @engineer |

---

## Root Cause Analysis

### What Happened

During a refactoring task, the AI coding assistant regenerated the credit card
validation function. The new implementation used a stricter regex pattern that
required exactly 16 digits with no separators, whereas the previous
implementation accepted various formats including:
- Cards with spaces: "4111 1111 1111 1111"
- Cards with dashes: "4111-1111-1111-1111"
- Amex 15-digit format: "3782 822463 10005"

### The Problematic Code

**Before (working):**
```typescript
const isValidCard = (number: string) => {
  // Remove all non-digits, then validate
  const cleaned = number.replace(/\D/g, '');
  return /^[0-9]{13,19}$/.test(cleaned) && luhnCheck(cleaned);
};
```

**After (AI-generated, broken):**
```typescript
const isValidCard = (number: string) => {
  // Validate card number format
  return /^[0-9]{16}$/.test(number) && luhnCheck(number);
};
```

### AI Code Analysis

| Question | Answer |
|----------|--------|
| Was AI-generated code involved? | Yes |
| Which component? | `src/payments/validation.ts` |
| What was the AI prompt? | "Refactor the validation module to reduce code duplication and improve clarity" |
| Did the prompt specify constraints? | No backward compatibility requirements |
| Was the code reviewed? | Yes, by 1 engineer |
| Did tests catch this? | No |
| Was there an MRP? | Yes, but it passed because tests passed |

### Why This Happened

1. **AI interpretation:** The AI interpreted "improve clarity" as simplifying the
   regex, not understanding that the complexity served a purpose (format tolerance)

2. **Missing context:** The AI didn't have access to:
   - Production data showing format variety
   - UI behavior allowing formatted input
   - Original requirement for format tolerance

3. **Test gap:** Existing tests only used unformatted card numbers:
   ```typescript
   // All test cases used this format
   expect(isValidCard('4111111111111111')).toBe(true);
   // No tests for formatted input
   ```

4. **Review limitation:** Reviewer focused on:
   - Does it compile? Yes
   - Do tests pass? Yes
   - Is it simpler? Yes
   Didn't consider: Does it handle the same inputs as before?

### Contributing Factors

- [ ] Inadequate testing - **YES** (no formatted input tests)
- [ ] Missing requirements - **YES** (format tolerance not documented)
- [ ] Code review gap - **YES** (reviewer didn't check edge cases)
- [ ] AI prompt issue - **YES** (no backward compatibility instruction)
- [ ] Monitoring gap - **NO** (alert fired quickly)
- [ ] Deployment issue - **NO** (deployment worked correctly)

---

## Detection and Response

### What Worked Well

1. **Fast detection:** Alert fired within 15 minutes of deployment
2. **AI SRE triage:** Initial investigation done automatically
3. **Clear escalation:** On-call engaged quickly with context
4. **Quick diagnosis:** Root cause found in 10 minutes
5. **Fast remediation:** Hotfix deployed in 25 minutes from detection

### What Could Be Improved

1. **Pre-production detection:** Should have caught before deployment
2. **AI prompt engineering:** Should specify backward compatibility
3. **Test coverage:** Need formatted input test cases
4. **Review checklist:** Need validation-specific checks

---

## Action Items

| ID | Action | Owner | Priority | Due Date | Status |
|----|--------|-------|----------|----------|--------|
| 1 | Add formatted card number test cases | @engineer | P0 | 2026-02-09 | Done |
| 2 | Add property-based tests for validation | @engineer | P1 | 2026-02-12 | In Progress |
| 3 | Update AI prompts with backward compatibility rule | @tech-lead | P1 | 2026-02-10 | Pending |
| 4 | Add "AI-assisted" label to PRs | @team | P2 | 2026-02-08 | Done |
| 5 | Create validation change checklist | @tech-lead | P1 | 2026-02-15 | Pending |
| 6 | Review all recent AI-generated validation code | @security | P1 | 2026-02-20 | Pending |
| 7 | Add production data samples to test fixtures | @qa | P2 | 2026-02-25 | Planned |
| 8 | Update CLAUDE.md with validation rules | @tech-lead | P1 | 2026-02-10 | Pending |

---

## Lessons Learned

### For AI-Assisted Development

1. **AI doesn't understand implicit requirements:** Format tolerance was
   "obvious" to humans but not specified anywhere AI could see

2. **Simplification isn't always improvement:** AI's "cleaner" code removed
   important functionality

3. **Tests must cover what matters, not what's easy:** All test cases were the
   easy path; edge cases weren't tested

4. **Review prompts, not just code:** The prompt lacked critical instructions

### For the Team

1. **Document implicit requirements:** Things "everyone knows" should be written
   down for AI and new team members

2. **Validation changes are high-risk:** Add to review checklist

3. **Test with production-like data:** Sanitized samples would have caught this

### Process Changes

**New CLAUDE.md Rule:**
```markdown
## Validation Changes

When modifying validation logic:
1. PRESERVE all previously accepted input formats unless explicitly told otherwise
2. ADD tests for formatted and edge case inputs
3. TEST with production data samples (see test-fixtures/validation/)
4. REQUIRE explicit approval for any regex changes
```

**New Review Checklist Item:**
```markdown
For validation/parsing changes:
- [ ] Tested with formatted input (spaces, dashes, etc.)
- [ ] Tested with edge case lengths
- [ ] Compared accepted inputs before vs after
- [ ] Checked production data patterns
```

---

## Appendix

### Related Incidents
- INC-2025-0089: Similar issue with phone number validation (2025-08-15)

### References
- PR #1234: Original change
- PR #1235: Hotfix
- Slack thread: #incident-2026-02-08
- PagerDuty incident: PD-12345

### Metrics During Incident

```
Checkout Error Rate:
13:45 - 0.1%
14:00 - 5.2% (alert threshold: 1%)
14:15 - 5.8% (peak)
14:30 - 3.1%
14:45 - 0.15% (resolved)
```

---

## Sign-off

| Role | Name | Date |
|------|------|------|
| Incident Commander | @tech-lead | 2026-02-09 |
| Author | @engineer | 2026-02-09 |
| Reviewer | @sre-lead | 2026-02-10 |

**Post-mortem meeting:** 2026-02-10 14:00 UTC
**Action items review:** 2026-02-17 (weekly sync)
```

**Post-Mortem Process Checklist:**

```markdown
## Post-Mortem Process

### Within 24 Hours
- [ ] Create post-mortem document from template
- [ ] Fill in timeline from incident channel/logs
- [ ] Identify root cause (5 Whys analysis)
- [ ] Document AI code involvement if applicable
- [ ] Create initial action items

### Within 48 Hours
- [ ] Hold post-mortem meeting with relevant stakeholders
- [ ] Finalize action items with owners and due dates
- [ ] Share summary with broader team

### Within 1 Week
- [ ] Complete P0 action items
- [ ] Begin P1 action items
- [ ] Update documentation/CLAUDE.md as needed

### Follow-up
- [ ] Review action item completion in weekly sync
- [ ] Close post-mortem when all P0/P1 items complete
- [ ] Add learnings to team knowledge base
```

---

## Cost Monitoring

### AI Usage Costs

AI-assisted development introduces new cost categories:

| Cost Type | Source | Typical Range |
|-----------|--------|---------------|
| AI API calls | Development prompts | $50-500/dev/month |
| AI-assisted reviews | Automated review tools | $20-100/month |
| AI SRE agents | Continuous monitoring | $100-500/month |
| Model inference | Production AI features | Variable |

### Tracking AI Development Costs

Monitor AI API usage to prevent surprises:

**Per-Developer Tracking:**
- API calls per day/week
- Tokens consumed per task type
- Cost by project/feature

**Team-Level Metrics:**
- Total AI spend vs. budget
- Cost per feature delivered
- ROI of AI assistance

### Infrastructure Cost Considerations

AI-generated code may have cost implications:

**Watch For:**
- Inefficient algorithms that consume more CPU
- Unnecessary API calls
- Over-provisioned resources
- Missed caching opportunities

**Optimization Strategies:**
- Profile AI-generated code for performance
- Review resource requests in Kubernetes manifests
- Audit external API call patterns
- Implement caching where appropriate

### Cost Alerting

Set up alerts for unexpected cost increases:

**Alert Thresholds:**
- Daily AI API cost exceeds 2x average
- Infrastructure cost spike > 20%
- Individual developer cost > monthly budget
- Production AI feature cost growing faster than usage

### Cost-Aware Prompting

Include cost considerations in prompts:

```
Implement this feature with the following constraints:
- Minimize external API calls (cache when possible)
- Use efficient algorithms (O(n) preferred over O(n^2))
- Avoid over-fetching data
- Consider pagination for large datasets
```

---

## CI/CD for AI-Assisted Projects

### Deployment Pipeline with AI Quality Gates

The following diagram shows a deployment pipeline with AI-specific quality gates:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              DEPLOYMENT PIPELINE WITH AI QUALITY GATES                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │    CODE     │     │    BUILD    │     │    TEST     │                   │
│  │   COMMIT    │────►│             │────►│             │                   │
│  └─────────────┘     └─────────────┘     └──────┬──────┘                   │
│                                                 │                           │
│                                                 ▼                           │
│                    ┌────────────────────────────────────────────┐          │
│                    │           AI QUALITY GATES                  │          │
│                    │                                             │          │
│                    │  ┌─────────────────────────────────────┐  │          │
│                    │  │ GATE 1: Unit Tests                   │  │          │
│                    │  │ Pass@1 > 95% for AI-generated code   │  │          │
│                    │  └─────────────────┬───────────────────┘  │          │
│                    │                    │                       │          │
│                    │  ┌─────────────────▼───────────────────┐  │          │
│                    │  │ GATE 2: Security Scan (SAST)         │  │          │
│                    │  │ No critical or high severity issues  │  │          │
│                    │  └─────────────────┬───────────────────┘  │          │
│                    │                    │                       │          │
│                    │  ┌─────────────────▼───────────────────┐  │          │
│                    │  │ GATE 3: Code Coverage                │  │          │
│                    │  │ Coverage > 80% on changed files      │  │          │
│                    │  └─────────────────┬───────────────────┘  │          │
│                    │                    │                       │          │
│                    │  ┌─────────────────▼───────────────────┐  │          │
│                    │  │ GATE 4: MRP Validation               │  │          │
│                    │  │ All 5 criteria met                   │  │          │
│                    │  └─────────────────┬───────────────────┘  │          │
│                    │                    │                       │          │
│                    └────────────────────│────────────────────────┘          │
│                                         │                                   │
│             ┌───────────────────────────┴───────────────────────┐          │
│             ▼                                                   ▼           │
│       [ALL PASS]                                           [ANY FAIL]       │
│             │                                                   │           │
│             ▼                                                   ▼           │
│  ┌─────────────────┐                                 ┌─────────────────┐   │
│  │    STAGING      │                                 │    BLOCKED      │   │
│  │   DEPLOYMENT    │                                 │                 │   │
│  │                 │                                 │  Return to dev  │   │
│  │  • E2E tests    │                                 │  with feedback  │   │
│  │  • Smoke tests  │                                 │                 │   │
│  │  • Performance  │                                 └─────────────────┘   │
│  └────────┬────────┘                                                       │
│           │                                                                 │
│           ▼                                                                 │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐      │
│  │   HUMAN GATE    │────►│   PRODUCTION    │────►│    VERIFY       │      │
│  │  (Approval)     │     │   DEPLOYMENT    │     │  + MONITOR      │      │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘      │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  KEY: AI quality gates catch issues before human review stage               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Pipeline Configuration

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: npm test

      - name: Run security scan
        run: npm run security-scan

      - name: Check test coverage
        run: |
          npm run coverage
          if [ $(cat coverage/coverage-summary.json | jq '.total.lines.pct') -lt 80 ]; then
            echo "Coverage below 80%"
            exit 1
          fi

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: ./deploy.sh staging

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Run smoke tests
        run: npm run test:smoke

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to production
        run: ./deploy.sh production

      - name: Verify deployment
        run: ./verify-deployment.sh
```

---

## Key Takeaways

1. **AI-generated code needs MORE monitoring**, not less
2. **The four golden signals** (latency, traffic, errors, saturation) are essential
3. **AI SRE agents** can provide continuous monitoring and first-response capabilities
4. **Define clear escalation criteria** for when AI must hand off to humans
5. **Use AI for incident response** — it can accelerate diagnosis significantly
6. **Kubernetes deployments** need proper health checks, resource limits, and graceful shutdown
7. **Feature flags** enable safe rollout of AI-generated features
8. **Always have rollback procedures** documented and tested
9. **Post-incident reviews** should specifically analyze AI code involvement
10. **Track AI-related costs** — both development API usage and production efficiency
11. **Monitor for AI-specific issues** like resource leaks and unexpected behavior
12. **CI/CD pipelines** must include security and quality gates for AI-generated code

---

## Practical Exercise: Production Readiness Review

Take an AI-assisted application (real or from a previous exercise) and:

### Part 1: Observability Setup
1. Add structured logging
2. Configure error tracking (e.g., Sentry)
3. Export metrics for the four golden signals

### Part 2: Deployment Strategy
1. Document rollback procedure
2. Plan canary deployment strategy
3. Identify feature flag candidates

### Part 3: Monitoring
1. Create dashboard for key metrics
2. Define alerting rules
3. Set up anomaly detection

### Part 4: Incident Simulation
1. Simulate an incident (e.g., slow database)
2. Use AI to help diagnose
3. Document the incident response
4. Identify improvements

---

## Next Step

[Step 13: Team Workflows and Governance](13-team-workflows-and-governance.md) — Learn to scale AI-assisted development across teams.

---

## Sources

### Primary References

1. **Google SRE Resources**
   - Beyer, B., et al. "Site Reliability Engineering: How Google Runs Production Systems." O'Reilly, 2016.
   - Chapter 6: "Monitoring Distributed Systems" - Source for the Four Golden Signals
   - Available free online: https://sre.google/sre-book/monitoring-distributed-systems/

2. **Academic Papers**
   - Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)
     - AI SRE operational boundaries and escalation patterns

3. **Industry Resources**
   - Resolve.ai. "Kubernetes Troubleshooting with AI" (2025) - https://resolve.ai/
   - Resolve.ai. "Benefits of Agentic AI in On-call Engineering" (2025)

4. **Stanford CS146S: The Modern Software Developer**
   - Week 9: Agents Post-Deployment
   - Operational patterns for AI-assisted systems

5. **Documentation**
   - Kubernetes Documentation: Best Practices for Production Deployments - https://kubernetes.io/docs/setup/best-practices/
   - OpenTelemetry Documentation: https://opentelemetry.io/docs/

---

## Further Reading

1. **Google SRE Book (Free Online)** - The definitive guide to site reliability engineering, including the Four Golden Signals chapter essential for monitoring AI-generated applications.
   https://sre.google/sre-book/table-of-contents/

2. **Resolve.ai Documentation** - Practical guidance on using AI for Kubernetes troubleshooting and on-call engineering.
   https://resolve.ai/

3. **SASE Paper: Structured Agentic Software Engineering** (arXiv:2509.06216v2) - Includes patterns for AI SRE agents and operational boundaries.
   https://arxiv.org/abs/2509.06216

4. **OpenTelemetry Documentation** - Standard for observability that works well with AI-generated applications.
   https://opentelemetry.io/docs/

5. **Prometheus Alerting Best Practices** - Guidelines for setting up effective alerts for production systems.
   https://prometheus.io/docs/practices/alerting/
