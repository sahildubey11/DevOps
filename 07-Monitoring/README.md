# Monitoring & Logging

## Why Monitoring and Logging?

- **Visibility**: Understand system behavior
- **Performance**: Identify bottlenecks
- **Reliability**: Detect issues early
- **Security**: Track security events
- **Compliance**: Meet regulatory requirements
- **Troubleshooting**: Debug production issues

## The Four Golden Signals

1. **Latency**: Time to service a request
2. **Traffic**: Demand on the system
3. **Errors**: Rate of failed requests
4. **Saturation**: Resource utilization

## Monitoring Tools

### Prometheus

**What is Prometheus?**
- Open-source monitoring system
- Time-series database
- Pull-based metrics collection
- Powerful query language (PromQL)
- Alerting capabilities

**Installation with Docker**
```bash
docker run -d -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

**prometheus.yml**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'

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

**PromQL Examples**
```promql
# CPU usage
rate(cpu_usage_seconds_total[5m])

# Memory usage
node_memory_Active_bytes / node_memory_MemTotal_bytes * 100

# HTTP request rate
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

### Grafana

**What is Grafana?**
- Visualization and analytics platform
- Multiple data source support
- Customizable dashboards
- Alerting

**Installation with Docker**
```bash
docker run -d -p 3000:3000 \
  --name=grafana \
  grafana/grafana
```

**Dashboard JSON (Example)**
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
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

### ELK Stack (Elasticsearch, Logstash, Kibana)

**Docker Compose Setup**
```yaml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.0.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.0.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

**Logstash Configuration**
```ruby
input {
  tcp {
    port => 5000
    codec => json
  }
  file {
    path => "/var/log/app/*.log"
    start_position => "beginning"
  }
}

filter {
  if [type] == "application" {
    json {
      source => "message"
    }
    date {
      match => ["timestamp", "ISO8601"]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
```

## Application Logging Best Practices

### Log Levels

```
TRACE - Very detailed debugging
DEBUG - Detailed debugging
INFO  - General information
WARN  - Warning messages
ERROR - Error messages
FATAL - Critical errors
```

### Structured Logging (JSON)

**Python Example**
```python
import logging
import json

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        return json.dumps(log_data)

logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info('Application started', extra={'user_id': 123, 'action': 'login'})
```

**Node.js Example**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

logger.info('User logged in', { userId: 123, ip: '192.168.1.1' });
logger.error('Database connection failed', { error: err.message });
```

## Metrics Collection

### Node Exporter (System Metrics)

```bash
docker run -d \
  --name=node-exporter \
  -p 9100:9100 \
  prom/node-exporter
```

### Application Metrics (Python)

```python
from prometheus_client import Counter, Histogram, start_http_server
import time

# Metrics
request_count = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
request_duration = Histogram('http_request_duration_seconds', 'HTTP request duration')

@request_duration.time()
def process_request(method, endpoint):
    # Process request
    time.sleep(0.1)
    request_count.labels(method=method, endpoint=endpoint, status=200).inc()

# Start metrics server
start_http_server(8000)

# Application code
process_request('GET', '/api/users')
```

### Application Metrics (Node.js)

```javascript
const promClient = require('prom-client');

const register = new promClient.Registry();

// Metrics
const httpRequestCounter = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status'],
  registers: [register]
});

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route'],
  registers: [register]
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestCounter.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
    httpRequestDuration.labels(req.method, req.route?.path || req.path).observe(duration);
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

## Alerting

### Prometheus Alerts

**alerts.yml**
```yaml
groups:
  - name: application_alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} for {{ $labels.instance }}"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"

      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance down"
          description: "{{ $labels.instance }} is down"

      - alert: HighMemoryUsage
        expr: (node_memory_Active_bytes / node_memory_MemTotal_bytes) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}%"
```

### AlertManager Configuration

**alertmanager.yml**
```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'password'

route:
  receiver: 'team-email'
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    - match:
        severity: warning
      receiver: 'slack'

receivers:
  - name: 'team-email'
    email_configs:
      - to: 'team@example.com'

  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'your-pagerduty-key'
```

## Distributed Tracing

### Jaeger

**Docker Compose**
```yaml
version: '3'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"
      - "14268:14268"
      - "9411:9411"
```

**Python with OpenTelemetry**
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Setup tracing
resource = Resource.create({"service.name": "my-service"})
tracer_provider = TracerProvider(resource=resource)
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)
tracer_provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
trace.set_tracer_provider(tracer_provider)

tracer = trace.get_tracer(__name__)

# Use tracing
with tracer.start_as_current_span("process_request"):
    # Your code here
    with tracer.start_as_current_span("database_query"):
        # Database query
        pass
```

## Centralized Logging Patterns

### Sidecar Pattern
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  - name: log-shipper
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
    - name: config
      mountPath: /fluent-bit/etc/

  volumes:
  - name: logs
    emptyDir: {}
  - name: config
    configMap:
      name: fluent-bit-config
```

## Interview Questions

1. What are the four golden signals of monitoring?
2. Explain the difference between metrics and logs
3. What is Prometheus and how does it work?
4. What is structured logging and why is it important?
5. How do you set up alerting in a monitoring system?
6. What is distributed tracing and when is it needed?
7. Explain the ELK stack
8. How do you monitor containerized applications?
9. What metrics would you track for a web application?
10. How do you handle log retention and rotation?

## Practical Exercises

1. Set up Prometheus and Grafana
2. Create custom metrics for an application
3. Build Grafana dashboards
4. Configure alerting rules
5. Set up ELK stack for log aggregation
6. Implement structured logging
7. Set up distributed tracing with Jaeger
8. Create monitoring for a microservices architecture

## Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Elastic Stack Documentation](https://www.elastic.co/guide/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Site Reliability Engineering (Book)](https://sre.google/sre-book/table-of-contents/)
