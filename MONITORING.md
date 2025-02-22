# Monitoring and Alerting

This document outlines the monitoring strategy, metrics, alerting configurations, and dashboard setups for the Market Correlation Agent.

## Table of Contents
- [Monitoring Strategy](#monitoring-strategy)
- [Key Metrics](#key-metrics)
- [Alerting Configuration](#alerting-configuration)
- [Dashboards](#dashboards)
- [Log Monitoring](#log-monitoring)
- [Infrastructure Monitoring](#infrastructure-monitoring)
- [Incident Response](#incident-response)
- [Monitoring Tools](#monitoring-tools)

## Monitoring Strategy

### Monitoring Layers
```
┌─────────────────────────────────────────────┐
│ Business Metrics                            │
│ - Tweet engagement                          │
│ - Analysis accuracy                         │
│ - User growth                               │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ Application Metrics                         │
│ - API call success/failure                  │
│ - Tweet posting rate                        │
│ - Analysis generation time                  │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ System Metrics                              │
│ - CPU/Memory usage                          │
│ - Disk space                                │
│ - Network connectivity                      │
└─────────────────────────────────────────────┘
```

### Collection Method
1. **System Metrics**: Collected via node_exporter for Prometheus
2. **Application Metrics**: Custom instrumentation with Prometheus client
3. **Business Metrics**: Application logs and scheduled queries

### Monitoring Frequency
- **High-frequency metrics**: 10-second intervals (system resources)
- **Medium-frequency metrics**: 1-minute intervals (API success rates)
- **Low-frequency metrics**: 15-minute intervals (business metrics)

## Key Metrics

### System Health
| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|-------------------|
| `cpu_usage` | CPU utilization percentage | 75% | 90% |
| `memory_usage` | Memory utilization percentage | 80% | 95% |
| `disk_space` | Available disk space | <20% | <10% |
| `network_errors` | Network error count | >5/min | >10/min |

### Performance Indicators
| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|-------------------|
| `api_response_time` | API response latency | >500ms | >1000ms |
| `browser_load_time` | Page load time in browser | >5s | >10s |
| `analysis_generation_time` | Time to generate analysis | >3s | >8s |
| `total_cycle_time` | Complete analysis cycle time | >30s | >60s |

### Error Rates
| Metric | Description | Warning Threshold | Critical Threshold |
|--------|-------------|-------------------|-------------------|
| `api_error_rate` | Failed API calls percentage | >5% | >10% |
| `tweet_posting_errors` | Failed tweet attempts | >2/hour | >5/hour |
| `browser_crashes` | Browser crash count | >1/day | >3/day |
| `analysis_failures` | Failed analysis generation | >3/day | >10/day |

### Business Metrics
| Metric | Description | Goal | Warning Threshold |
|--------|-------------|------|-------------------|
| `tweet_engagement` | Avg likes/retweets per post | >10 | <5 |
| `posting_frequency` | Posts per day | 12-24 | <10 or >30 |
| `follower_growth` | New followers per week | >50 | <20 |
| `market_coverage` | % of significant market moves covered | >90% | <75% |

## Alerting Configuration

### Alert Severity Levels
1. **INFO**: Informational only, no action required
2. **WARNING**: Potential issue, may require attention
3. **ERROR**: Issue affecting functionality, requires prompt attention
4. **CRITICAL**: Major issue affecting service, requires immediate action

### Alert Configurations

#### System Alerts
```yaml
# Prometheus alerting rules
groups:
- name: system_alerts
  rules:
  - alert: HighCPUUsage
    expr: cpu_usage > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage"
      description: "CPU usage is {{ $value }}%"

  - alert: HighMemoryUsage
    expr: memory_usage > 95
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "High memory usage"
      description: "Memory usage is {{ $value }}%"

  - alert: DiskSpaceLow
    expr: disk_free_percent < 10
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: "Low disk space"
      description: "Only {{ $value }}% disk space remaining"
```

#### Application Alerts
```yaml
- name: application_alerts
  rules:
  - alert: APIFailureRate
    expr: rate(api_failures[15m]) / rate(api_requests[15m]) > 0.1
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "High API failure rate"
      description: "API failure rate is {{ $value | humanizePercentage }}"

  - alert: AnalysisGenerationFailures
    expr: rate(analysis_failures[1h]) > 10
    for: 15m
    labels:
      severity: critical
    annotations:
      summary: "Analysis generation failures"
      description: "{{ $value }} analysis failures in the last hour"

  - alert: TweetPostingFailures
    expr: rate(tweet_posting_failures[1h]) > 5
    for: 15m
    labels:
      severity: error
    annotations:
      summary: "Tweet posting failures"
      description: "{{ $value }} tweet posting failures in the last hour"
```

#### Business Alerts
```yaml
- name: business_alerts
  rules:
  - alert: LowTweetEngagement
    expr: avg_tweet_engagement < 5
    for: 24h
    labels:
      severity: warning
    annotations:
      summary: "Low tweet engagement"
      description: "Average engagement is {{ $value }} per tweet"

  - alert: AbnormalPostingFrequency
    expr: posts_per_day < 10 or posts_per_day > 30
    for: 6h
    labels:
      severity: warning
    annotations:
      summary: "Abnormal posting frequency"
      description: "Current posting rate is {{ $value }} posts per day"
```

### Notification Channels
1. **Slack**: For all alerts
```yaml
slack_configs:
  - api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXX'
    channel: '#bot-alerts'
    send_resolved: true
    title: '{{ template "slack.default.title" . }}'
    text: '{{ template "slack.default.text" . }}'
```

2. **Email**: For ERROR and CRITICAL alerts
```yaml
email_configs:
  - to: 'team@example.com'
    from: 'alerts@example.com'
    smarthost: 'smtp.example.com:587'
    auth_username: 'alerts@example.com'
    auth_password: 'password'
    send_resolved: true
```

3. **PagerDuty**: For CRITICAL alerts only
```yaml
pagerduty_configs:
  - service_key: '<pagerduty-service-key>'
    description: '{{ template "pagerduty.default.description" . }}'
    client: 'ETH/BTC Bot Monitoring'
    client_url: 'https://monitoring.example.com'
    details:
      firing: '{{ template "pagerduty.default.instances" .Alerts.Firing }}'
      resolved: '{{ template "pagerduty.default.instances" .Alerts.Resolved }}'
    send_resolved: true
```

## Dashboards

### System Dashboard
![System Dashboard](https://via.placeholder.com/800x400?text=System+Dashboard)

```json
{
  "title": "ETH/BTC Bot - System Dashboard",
  "rows": [
    {
      "title": "CPU & Memory",
      "panels": [
        {
          "title": "CPU Usage",
          "type": "graph",
          "targets": [
            {
              "expr": "cpu_usage",
              "legendFormat": "CPU %"
            }
          ]
        },
        {
          "title": "Memory Usage",
          "type": "graph",
          "targets": [
            {
              "expr": "memory_usage",
              "legendFormat": "Memory %"
            }
          ]
        }
      ]
    },
    {
      "title": "Disk & Network",
      "panels": [
        {
          "title": "Disk Space",
          "type": "gauge",
          "targets": [
            {
              "expr": "disk_free_percent",
              "legendFormat": "Free Space %"
            }
          ]
        },
        {
          "title": "Network Traffic",
          "type": "graph",
          "targets": [
            {
              "expr": "network_in_bytes",
              "legendFormat": "Network In"
            },
            {
              "expr": "network_out_bytes",
              "legendFormat": "Network Out"
            }
          ]
        }
      ]
    }
  ]
}
```

### Application Dashboard
![Application Dashboard](https://via.placeholder.com/800x400?text=Application+Dashboard)

```json
{
  "title": "ETH/BTC Bot - Application Dashboard",
  "rows": [
    {
      "title": "API Performance",
      "panels": [
        {
          "title": "API Response Time",
          "type": "graph",
          "targets": [
            {
              "expr": "api_response_time{endpoint='coingecko'}",
              "legendFormat": "CoinGecko"
            },
            {
              "expr": "api_response_time{endpoint='claude'}",
              "legendFormat": "Claude AI"
            }
          ]
        },
        {
          "title": "API Success Rate",
          "type": "graph",
          "targets": [
            {
              "expr": "rate(api_success[5m]) / rate(api_requests[5m]) * 100",
              "legendFormat": "Success Rate %"
            }
          ]
        }
      ]
    },
    {
      "title": "Bot Activities",
      "panels": [
        {
          "title": "Analysis Generation",
          "type": "graph",
          "targets": [
            {
              "expr": "analysis_generation_time",
              "legendFormat": "Analysis Time"
            }
          ]
        },
        {
          "title": "Tweet Posting",
          "type": "graph",
          "targets": [
            {
              "expr": "rate(tweet_postings[1h])",
              "legendFormat": "Tweets/Hour"
            }
          ]
        }
      ]
    }
  ]
}
```

### Business Dashboard
![Business Dashboard](https://via.placeholder.com/800x400?text=Business+Dashboard)

```json
{
  "title": "ETH/BTC Bot - Business Dashboard",
  "rows": [
    {
      "title": "Engagement Metrics",
      "panels": [
        {
          "title": "Tweet Engagement",
          "type": "graph",
          "targets": [
            {
              "expr": "avg_tweet_likes",
              "legendFormat": "Avg Likes"
            },
            {
              "expr": "avg_tweet_retweets",
              "legendFormat": "Avg Retweets"
            }
          ]
        },
        {
          "title": "Follower Growth",
          "type": "graph",
          "targets": [
            {
              "expr": "twitter_followers",
              "legendFormat": "Followers"
            }
          ]
        }
      ]
    },
    {
      "title": "Content Analysis",
      "panels": [
        {
          "title": "Post Distribution",
          "type": "piechart",
          "targets": [
            {
              "expr": "tweets_by_category",
              "legendFormat": "{{category}}"
            }
          ]
        },
        {
          "title": "Market Movement Coverage",
          "type": "gauge",
          "targets": [
            {
              "expr": "market_moves_covered / total_significant_market_moves * 100",
              "legendFormat": "Coverage %"
            }
          ]
        }
      ]
    }
  ]
}
```

## Log Monitoring

### Log Patterns to Monitor
1. **Error Patterns**
   ```
   ERROR|Exception|Failed|Timeout|ConnectionError|AuthenticationError
   ```

2. **Warning Patterns**
   ```
   WARNING|Retry|Degraded|Unstable|Slow|rate limit
   ```

3. **Security Patterns**
   ```
   Authentication failure|Invalid token|Unauthorized|Suspicious|Blocked
   ```

### Log Parsing Rules
```yaml
# Loki log parsing rules
- name: error_logs
  pattern: 'level=ERROR|level=CRITICAL|Exception|Failed|Timeout|ConnectionError'
  labels:
    log_type: error

- name: warning_logs
  pattern: 'level=WARNING|Retry|Degraded|Unstable|Slow|rate limit'
  labels:
    log_type: warning

- name: api_logs
  pattern: 'API call|API response|API request|API error'
  labels:
    log_type: api

- name: twitter_logs
  pattern: 'Twitter|Tweet|posted|login|session'
  labels:
    log_type: twitter

- name: analysis_logs
  pattern: 'Analysis|analyze|sentiment|market|correlation'
  labels:
    log_type: analysis
```

### Log Volume Alerts
```yaml
# Alert on high log volume
- alert: HighErrorLogVolume
  expr: rate(error_logs[5m]) > 10
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "High error log volume"
    description: "{{ $value }} error logs per second"
```

## Infrastructure Monitoring

### Hardware Monitoring
```yaml
# Node exporter metrics
- job_name: 'node'
  static_configs:
    - targets: ['localhost:9100']
  metrics_path: /metrics
  scrape_interval: 15s
```

### Network Monitoring
```bash
# Network monitoring with ping
ping -c 1 api.coingecko.com &>/dev/null || echo "CoinGecko API unreachable" | logger -t network_monitor
ping -c 1 api.anthropic.com &>/dev/null || echo "Claude API unreachable" | logger -t network_monitor
ping -c 1 twitter.com &>/dev/null || echo "Twitter unreachable" | logger -t network_monitor
```

### Process Monitoring
```yaml
# Process monitoring
- name: process_monitoring
  rules:
  - alert: ProcessNotRunning
    expr: process_running{name="eth_btc_bot"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "ETH/BTC Bot process not running"
      description: "The main bot process is not running"
```

## Incident Response

### Alert Escalation Matrix
| Alert Severity | Initial Response | Escalation (15 min) | Escalation (1 hour) |
|----------------|------------------|---------------------|---------------------|
| INFO | Slack channel | None | None |
| WARNING | Slack channel | Email to team | None |
| ERROR | Slack + Email | On-call engineer | Team lead |
| CRITICAL | Slack + Email + Call | On-call team | Service owner |

### Incident Categories
1. **Availability Issues**: Service not running or not posting
2. **Performance Issues**: Slow response times or high resource usage
3. **Data Issues**: Incorrect data or analysis
4. **Security Issues**: Unauthorized access or suspicious activity

### Runbooks
```markdown
# Runbook: API Failure Recovery

## Symptoms
- High API failure rate alerts
- Analysis generation failures
- Missing market data

## Diagnosis
1. Check API status
   ```
   curl -I https://api.coingecko.com/api/v3/ping
   curl -I https://api.anthropic.com/v1/status
   ```

2. Check API error logs
   ```
   grep "API error" /var/log/eth-btc-bot/bot.log | tail -20
   ```

3. Verify network connectivity
   ```
   traceroute api.coingecko.com
   ```

## Resolution
1. If rate limited:
   ```
   # Modify config to reduce request frequency
   sed -i 's/CORRELATION_INTERVAL=5/CORRELATION_INTERVAL=10/' /opt/eth-btc-bot/.env
   ```

2. If API is down:
   ```
   # Switch to alternate data source
   cp /opt/eth-btc-bot/config/backup_sources.py /opt/eth-btc-bot/config/active_sources.py
   ```

3. Restart service
   ```
   systemctl restart eth-btc-bot
   ```

4. Monitor for recovery
   ```
   tail -f /var/log/eth-btc-bot/bot.log | grep "API"
   ```
```

## Monitoring Tools

### Core Monitoring Stack
- **Prometheus**: Metrics collection and alerting
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation and searching
- **Alertmanager**: Alert routing and notifications

### Deployment Configuration
```yaml
# docker-compose.yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:v2.38.0
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - 9090:9090

  node-exporter:
    image: prom/node-exporter:v1.3.1
    restart: always
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - 9100:9100

  grafana:
    image: grafana/grafana:9.1.5
    volumes:
      - grafana_data:/var/lib/grafana
      - ./dashboards:/etc/grafana/dashboards
      - ./grafana.ini:/etc/grafana/grafana.ini
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secure_password
    ports:
      - 3000:3000

  loki:
    image: grafana/loki:2.6.1
    ports:
      - 3100:3100
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:2.6.1
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yaml:/etc/promtail/config.yaml
    command: -config.file=/etc/promtail/config.yaml

  alertmanager:
    image: prom/alertmanager:v0.24.0
    ports:
      - 9093:9093
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:
```

### Custom Instrumentation
```python
# prometheus_metrics.py
from prometheus_client import Counter, Gauge, Histogram, Summary
import time

# Counters
api_requests = Counter('api_requests_total', 'Total API Requests', ['endpoint'])
api_errors = Counter('api_errors_total', 'Total API Errors', ['endpoint', 'error_type'])
tweet_postings = Counter('tweet_postings_total', 'Total Tweet Postings')
tweet_errors = Counter('tweet_errors_total', 'Total Tweet Posting Errors', ['error_type'])
analysis_generations = Counter('analysis_generations_total', 'Total Analysis Generations')
analysis_errors = Counter('analysis_errors_total', 'Total Analysis Errors', ['error_type'])

# Gauges
cpu_usage = Gauge('cpu_usage_percent', 'CPU Usage Percentage')
memory_usage = Gauge('memory_usage_percent', 'Memory Usage Percentage')
disk_free = Gauge('disk_free_percent', 'Disk Free Percentage')
active_connections = Gauge('active_connections', 'Number of Active Connections')

# Histograms
response_time = Histogram(
    'api_response_time_seconds', 
    'API Response Time in Seconds',
    ['endpoint'],
    buckets=[0.1, 0.25, 0.5, 0.75, 1.0, 2.5, 5.0, 10.0]
)

# Summaries
analysis_time = Summary(
    'analysis_generation_time_seconds', 
    'Time Spent Generating Analysis'
)

# Example usage
def track_api_request(endpoint):
    api_requests.labels(endpoint=endpoint).inc()
    start = time.time()
    yield
    duration = time.time() - start
    response_time.labels(endpoint=endpoint).observe(duration)
```

### Integration with Agent Code
```python
# Example integration in bot code
def _get_crypto_data(self) -> Optional[Dict[str, Any]]:
    """Fetch BTC and ETH data from CoinGecko with retries and metrics"""
    # Track API request
    with track_api_request('coingecko'):
        try:
            response = self.session.get(
                self.config.get_coingecko_markets_url(),
                params=self.config.get_coingecko_params(),
                timeout=(30, 90)
            )
            response.raise_for_status()
            
            # Successful request
            return self._process_crypto_data(response.json())
            
        except requests.exceptions.Timeout:
            # Track timeout error
            api_errors.labels(endpoint='coingecko', error_type='timeout').inc()
            logger.log_error("CoinGecko API", "Request timeout")
            return None
            
        except Exception as e:
            # Track other errors
            api_errors.labels(endpoint='coingecko', error_type='other').inc()
            logger.log_error("CoinGecko API", str(e))
            return None
```
