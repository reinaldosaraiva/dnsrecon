# DNSRECON

**Modernized DNS Reconnaissance REST API** - Collect comprehensive DNS records (SOA, NS, MX, A, AAAA, TXT, CNAME) through a high-performance concurrent lookup engine.

> ⚠️ **Defensive Security Tool Only** - For legitimate DNS reconnaissance and security analysis

## ✨ Features

- **Concurrent DNS Lookups** - Parallel queries across multiple public DNS servers
- **Comprehensive Records** - SOA, NS, MX, A, AAAA, TXT, CNAME support  
- **Rate Limiting** - Built-in protection against abuse
- **LRU Caching** - 24-hour cache with automatic cleanup
- **Docker Ready** - Production-ready containerized deployment
- **Modern Go** - Updated to Go 1.22 with latest dependencies

## 🚀 Quick Start

### Option 1: Docker (Recommended)

```bash
# Pull and run (when published)
docker run -d -p 8080:8080 --restart=unless-stopped --name dnsrecon dnsrecon:latest

# Or build locally
git clone <repository>
cd dnsrecon
docker build -t dnsrecon .
docker run -d -p 8080:8080 --restart=unless-stopped --name dnsrecon dnsrecon
```

### Option 2: Local Development

```bash
# Prerequisites: Go 1.22+
git clone <repository>
cd dnsrecon

# Install dependencies
go mod download

# Run locally
go run .
```

## 📡 API Usage

### Health Check
```bash
curl http://localhost:8080/
# Response: ok
```

### DNS Lookup
```bash
curl http://localhost:8080/domain/google.com | jq
```

### Example Response
```json
{
  "name": "google.com",
  "timestamp": "2025-06-22T13:12:37.554627576Z",
  "status": "NOERROR",
  "errors": {},
  "data": {
    "soa": {
      "name": "google.com",
      "primary_nameserver": {
        "ns1.google.com": {
          "a": ["216.239.32.10"],
          "aaaa": ["2001:4860:4802:32::a"]
        }
      },
      "mbox": "dns-admin.google.com"
    },
    "ns": {
      "ns1.google.com": {"a": ["216.239.32.10"], "aaaa": ["2001:4860:4802:32::a"]},
      "ns2.google.com": {"a": ["216.239.34.10"], "aaaa": ["2001:4860:4802:34::a"]},
      "ns3.google.com": {"a": ["216.239.36.10"], "aaaa": ["2001:4860:4802:36::a"]},
      "ns4.google.com": {"a": ["216.239.38.10"], "aaaa": ["2001:4860:4802:38::a"]}
    },
    "mx": {
      "10": {
        "smtp.google.com": {
          "a": ["172.217.192.27", "64.233.190.27"],
          "aaaa": ["2800:3f0:4003:c01::1b", "2800:3f0:4003:c02::1a"]
        }
      }
    },
    "a": ["142.251.135.238"],
    "aaaa": ["2800:3f0:4001:841::200e"],
    "txt": null,
    "cname": null,
    "cname_paths": {}
  }
}
```

## 🐳 Production Deployment

### Docker Compose
```yaml
version: '3.8'
services:
  dnsrecon:
    image: dnsrecon:latest
    ports:
      - "8080:8080"
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    environment:
      - TZ=UTC
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Kubernetes
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dnsrecon
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dnsrecon
  template:
    metadata:
      labels:
        app: dnsrecon
    spec:
      containers:
      - name: dnsrecon
        image: dnsrecon:latest
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: dnsrecon-service
spec:
  selector:
    app: dnsrecon
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

## ⚙️ Configuration

### config.yaml
```yaml
maximum_dns_servers: 5  # Number of concurrent DNS servers to use
```

### resolvers.yaml
Auto-generated file containing public DNS servers with rate limits. Modify to add custom DNS servers:

```yaml
resolvers:
  - nameserver: "custom-dns"
    ips: ["1.1.1.1:53", "1.0.0.1:53"]
    ratelimit: 20
    enable: true
```

## 🔧 Development

### Build
```bash
# Local binary
go build -o dnsrecon .

# Docker image
docker build -t dnsrecon:dev .

# Cross-platform build
GOOS=linux GOARCH=amd64 go build -o dnsrecon-linux .
```

### Dependencies
- Go 1.22+
- github.com/gorilla/mux - HTTP routing
- github.com/miekg/dns - DNS protocol
- github.com/hashicorp/golang-lru/v2 - LRU caching
- golang.org/x/time/rate - Rate limiting
- gopkg.in/yaml.v2 - Configuration

## 🛡️ Security Features

- **Non-root container** - Runs as unprivileged user
- **Rate limiting** - Per-resolver request throttling  
- **Input validation** - Domain name sanitization
- **Error handling** - Graceful failure modes
- **Resource limits** - Memory and CPU bounded
- **Timeout protection** - Request timeout enforcement

## 📊 Monitoring

### Metrics Endpoints
- `GET /` - Health check (returns "ok")
- Check container logs for performance metrics

### Performance
- **Concurrent lookups** - 5 DNS servers by default
- **Cache hit ratio** - LRU cache with 24h TTL
- **Response time** - Typically <100ms for cached results
- **Memory usage** - ~64MB baseline, ~128MB under load

## 🔍 Troubleshooting

### Common Issues

**Container won't start:**
```bash
docker logs dnsrecon
# Check for permission or configuration errors
```

**DNS lookups failing:**
```bash
# Test connectivity
docker exec dnsrecon nslookup google.com 8.8.8.8

# Check rate limits in logs
docker logs dnsrecon | grep -i rate
```

**Performance issues:**
- Increase `maximum_dns_servers` in config.yaml
- Add more DNS servers to resolvers.yaml
- Monitor container resource usage

### Debug Mode
```bash
# Run with verbose logging
docker run -p 8080:8080 dnsrecon:latest --debug
```

## 📝 API Reference

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check |
| GET | `/domain/{domain}` | DNS lookup for domain |

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Queried domain name |
| `timestamp` | string | RFC3339 timestamp |
| `status` | string | DNS response code |
| `errors` | object | Error messages by record type |
| `data.soa` | object | Start of Authority record |
| `data.ns` | object | Name Server records |
| `data.mx` | object | Mail Exchange records |
| `data.a` | array | IPv4 addresses |
| `data.aaaa` | array | IPv6 addresses |
| `data.txt` | array | Text records |
| `data.cname` | array | Canonical name records |

### Error Codes

| Status | Description |
|--------|-------------|
| `NOERROR` | Successful lookup |
| `NXDOMAIN` | Domain does not exist |
| `TIMEOUT` | DNS query timeout |
| `ERROR` | General lookup error |

## 📄 License

This project is provided for educational and defensive security purposes only.