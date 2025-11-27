# Monitoring and Logging

## What is Monitoring?

Monitoring is the practice of collecting, analyzing, and using information to track the performance and health of your systems and applications.

## What is Logging?

Logging is the process of recording events, errors, and information about system and application behavior for debugging, auditing, and analysis.

## Why Monitor and Log?

### Benefits
- **Early Problem Detection**: Identify issues before users notice
- **Performance Optimization**: Find bottlenecks and optimize
- **Capacity Planning**: Understand resource usage trends
- **Security**: Detect anomalies and security threats
- **Compliance**: Meet regulatory requirements
- **Debugging**: Troubleshoot issues faster
- **Business Insights**: Understand user behavior

## The Three Pillars of Observability

### 1. Metrics
Numerical measurements over time (CPU, memory, request rate, etc.)

### 2. Logs
Discrete events with timestamps (application logs, error logs, etc.)

### 3. Traces
Request flow through distributed systems (distributed tracing)

## Metrics

### Types of Metrics

#### System Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network traffic

#### Application Metrics
- Request rate
- Response time
- Error rate
- Throughput

#### Business Metrics
- User signups
- Transactions
- Revenue
- Conversion rate

### Key Performance Indicators (KPIs)

#### The Four Golden Signals (Google SRE)
1. **Latency**: Time to service a request
2. **Traffic**: Demand on your system
3. **Errors**: Rate of failed requests
4. **Saturation**: How "full" your service is

#### RED Method (for services)
- **Rate**: Requests per second
- **Errors**: Failed requests per second
- **Duration**: Request latency

#### USE Method (for resources)
- **Utilization**: % time resource is busy
- **Saturation**: Amount of queued work
- **Errors**: Error count

## Monitoring Tools

### Prometheus

**Type**: Open-source monitoring and alerting system

**Features**:
- Time-series database
- Pull-based metrics collection
- PromQL query language
- Multi-dimensional data model
- Service discovery

**Architecture**:
```
┌──────────────┐
│ Prometheus   │
│   Server     │
└──────┬───────┘
       │ Pull metrics
   ┌───▼────┐  ┌─────────┐  ┌─────────┐
   │ Node   │  │  App    │  │ Custom  │
   │Exporter│  │Exporter │  │Exporter │
   └────────┘  └─────────┘  └─────────┘
```

**Installation (Docker)**:
```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

**prometheus.yml**:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
  
  - job_name: 'application'
    static_configs:
      - targets: ['app:8080']
```

**PromQL Examples**:
```promql
# CPU usage
rate(cpu_usage_seconds_total[5m])

# Memory usage percentage
(node_memory_MemTotal - node_memory_MemAvailable) / node_memory_MemTotal * 100

# Request rate
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m])
```

**Instrumenting Code (Python)**:
```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Define metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])
REQUEST_LATENCY = Histogram('http_request_duration_seconds', 'HTTP request latency')
ACTIVE_USERS = Gauge('active_users', 'Number of active users')

# Use metrics
@REQUEST_LATENCY.time()
def handle_request(method, endpoint):
    REQUEST_COUNT.labels(method=method, endpoint=endpoint).inc()
    # Process request
    time.sleep(0.1)

# Start metrics server
start_http_server(8000)
```

### Grafana

**Type**: Visualization and analytics platform

**Features**:
- Dashboards and visualizations
- Multiple data sources (Prometheus, InfluxDB, Elasticsearch, etc.)
- Alerting
- Templating
- Plugins

**Installation (Docker)**:
```bash
docker run -d \
  --name grafana \
  -p 3000:3000 \
  grafana/grafana
```

**Dashboard JSON Example**:
```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      }
    ]
  }
}
```

### Datadog

**Type**: Commercial SaaS monitoring platform

**Features**:
- Full-stack observability
- Infrastructure monitoring
- APM (Application Performance Monitoring)
- Log management
- Real user monitoring

**Agent Installation**:
```bash
# Ubuntu/Debian
DD_API_KEY=<your-api-key> bash -c "$(curl -L https://s.datadoghq.com/scripts/install_script.sh)"

# Configuration
# Edit /etc/datadog-agent/datadog.yaml
```

### New Relic

**Type**: Commercial APM platform

**Features**:
- Application monitoring
- Infrastructure monitoring
- Browser monitoring
- Synthetic monitoring
- Distributed tracing

### Nagios

**Type**: Open-source monitoring system

**Features**:
- Server, service, and network monitoring
- Alert system
- Plugin architecture
- Reporting

### Zabbix

**Type**: Enterprise-class monitoring solution

**Features**:
- Network monitoring
- Server monitoring
- Application monitoring
- Auto-discovery
- Distributed monitoring

## Logging

### Log Levels

Standard severity levels:
```
TRACE   - Very detailed information
DEBUG   - Debug information
INFO    - Informational messages
WARN    - Warning messages
ERROR   - Error messages
FATAL   - Critical errors
```

### Structured Logging

**Traditional (unstructured)**:
```
2023-11-21 10:30:45 User john logged in from 192.168.1.100
```

**Structured (JSON)**:
```json
{
  "timestamp": "2023-11-21T10:30:45Z",
  "level": "INFO",
  "message": "User logged in",
  "user": "john",
  "ip": "192.168.1.100",
  "session_id": "abc123"
}
```

### Logging Best Practices

1. **Use Appropriate Log Levels**: Don't log everything as ERROR
2. **Include Context**: User ID, request ID, transaction ID
3. **Structured Logging**: Use JSON for easier parsing
4. **Don't Log Sensitive Data**: Passwords, credit cards, PII
5. **Correlation IDs**: Track requests across services
6. **Log to stdout/stderr**: Let log aggregators handle files
7. **Sampling**: In high-traffic systems, sample logs
8. **Meaningful Messages**: Clear, actionable information

### Logging in Code

**Python**:
```python
import logging
import json

# Configure structured logging
logging.basicConfig(
    level=logging.INFO,
    format='%(message)s'
)

logger = logging.getLogger(__name__)

def log_structured(level, message, **kwargs):
    log_entry = {
        'timestamp': datetime.utcnow().isoformat(),
        'level': level,
        'message': message,
        **kwargs
    }
    logger.log(getattr(logging, level), json.dumps(log_entry))

# Usage
log_structured('INFO', 'User logged in', user_id=123, ip='192.168.1.100')
```

**Node.js (Winston)**:
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

logger.info('User logged in', {
  userId: 123,
  ip: '192.168.1.100'
});
```

## Log Aggregation

### ELK Stack (Elasticsearch, Logstash, Kibana)

**Architecture**:
```
Applications → Logstash → Elasticsearch → Kibana
     │              │
     └──────────────┴──> Beats (Filebeat, Metricbeat)
```

**Logstash Configuration**:
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  json {
    source => "message"
  }
  
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  
  date {
    match => [ "timestamp", "ISO8601" ]
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

**docker-compose.yml for ELK**:
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
  
  logstash:
    image: docker.elastic.co/logstash/logstash:8.10.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch
  
  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

### Fluentd

**Type**: Open-source data collector

**Configuration (fluent.conf)**:
```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
  @type record_transformer
  <record>
    hostname ${hostname}
  </record>
</filter>

<match **>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
</match>
```

### Loki (Grafana Loki)

**Type**: Log aggregation system designed for Grafana

**Features**:
- Efficient log storage
- Labels for indexing
- LogQL query language
- Grafana integration

**Promtail Configuration**:
```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

## Alerting

### Alert Manager (Prometheus)

**Configuration**:
```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'email'
    email_configs:
      - to: 'team@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
```

**Prometheus Alert Rules**:
```yaml
groups:
  - name: example
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests/second"
      
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal - node_memory_MemAvailable) / node_memory_MemTotal > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"
      
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
```

### PagerDuty Integration

**Alert Manager Configuration**:
```yaml
receivers:
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_SERVICE_KEY'
        description: '{{ .GroupLabels.alertname }}'
```

## Distributed Tracing

### Jaeger

**Type**: Open-source distributed tracing

**Architecture**:
```
Application → Jaeger Client → Jaeger Agent → Jaeger Collector → Storage (Cassandra/ES)
                                                                         ↓
                                                                   Jaeger Query
                                                                         ↓
                                                                   Jaeger UI
```

**Installation (All-in-one)**:
```bash
docker run -d --name jaeger \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  jaegertracing/all-in-one:latest
```

**Instrumenting Code (Python)**:
```python
from jaeger_client import Config
from flask import Flask
from flask_opentracing import FlaskTracing

app = Flask(__name__)

config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
    },
    service_name='my-service',
)
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

@app.route('/api/users')
def get_users():
    with jaeger_tracer.start_span('database-query') as span:
        # Database query
        users = query_users()
        span.set_tag('user_count', len(users))
    return users
```

### OpenTelemetry

**Type**: Observability framework (successor to OpenTracing and OpenCensus)

**Features**:
- Vendor-neutral
- Metrics, traces, and logs
- Auto-instrumentation
- Multiple language SDKs

## Monitoring Dashboards

### Sample Grafana Dashboard Structure

1. **Overview Panel**
   - Service health status
   - Request rate
   - Error rate
   - Latency

2. **Resource Metrics**
   - CPU usage
   - Memory usage
   - Disk I/O
   - Network traffic

3. **Application Metrics**
   - Active users
   - Queue lengths
   - Cache hit rate
   - Database queries

4. **Business Metrics**
   - Transactions per minute
   - Revenue
   - User signups

## Best Practices

### 1. Define SLIs, SLOs, and SLAs

- **SLI** (Service Level Indicator): Metric (e.g., 95th percentile latency)
- **SLO** (Service Level Objective): Target (e.g., 95th percentile < 200ms)
- **SLA** (Service Level Agreement): Contract with consequences

### 2. Set Meaningful Alerts

- Alert on symptoms, not causes
- Avoid alert fatigue
- Make alerts actionable
- Use alert grouping

### 3. Monitor the Right Things

Focus on:
- User-facing metrics
- Business KPIs
- Resource saturation
- Error rates

### 4. Use Dashboards Effectively

- Start with high-level overview
- Drill down to details
- Use consistent color schemes
- Include documentation

### 5. Retention Policies

- High-resolution data: 7-30 days
- Aggregated data: 90 days - 1 year
- Logs: Based on compliance requirements

### 6. Security

- Encrypt data in transit and at rest
- Control access to sensitive logs
- Audit log access
- Redact sensitive information

## Resources

- Prometheus Documentation: https://prometheus.io/docs/
- Grafana Documentation: https://grafana.com/docs/
- ELK Stack: https://www.elastic.co/what-is/elk-stack
- Jaeger Documentation: https://www.jaegertracing.io/docs/
- Google SRE Book: https://sre.google/books/
- OpenTelemetry: https://opentelemetry.io/

## Next Steps

Continue to [Cloud Platforms](../09-Cloud-Platforms/) to learn about AWS, Azure, and Google Cloud Platform.
