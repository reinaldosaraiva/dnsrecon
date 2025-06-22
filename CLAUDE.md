# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DNSRECON is a prototype REST API for DNS reconnaissance that collects comprehensive DNS records (SOA, NS, MX, A, AAAA, TXT, CNAME). It's built in Go and uses concurrent DNS lookups with multiple public DNS servers for performance and reliability.

**Warning: This is prototype software not intended for production use.**

## Development Commands

### Local Development
```bash
# Run locally (from project root)
go run *.go

# Install dependencies manually if needed
go get github.com/miekg/dns
go get github.com/gorilla/mux
go get golang.org/x/time/rate
go get github.com/golang/groupcache/lru
go get gopkg.in/yaml.v2
```

### Build and Install
```bash
# Build binary
go build -o dnsrecon .

# Install globally
go install dnsrecon
```

### Docker
```bash
# Build Docker image
docker build --rm -t "dnsrecon" .

# Run container
docker run -d -p 8080:8080 --restart=unless-stopped --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 --name dnsrecon dnsrecon
```

## Architecture

### Core Components

- **Main Server** (`dnsrecon.go`): HTTP server setup, DNS client pool initialization, and routing
- **DNS Client** (`dnsrecon/client.go`): Concurrent DNS lookup engine with rate limiting and caching
- **Handlers** (`handlers/`): HTTP request handlers with context-based middleware
- **Resolvers** (`resolvers/`): DNS server pool management with hardcoded public DNS servers
- **Config** (`config/`): YAML-based configuration system
- **Logging** (`logging/`): Structured logging utilities

### Key Design Patterns

1. **Concurrent DNS Lookups**: Uses goroutines for parallel SOA, A, AAAA, NS, MX, TXT, and CNAME queries
2. **DNS Client Pool**: Maintains a channel-based pool of DNS clients, each with different public DNS servers
3. **Rate Limiting**: Per-resolver rate limiting using golang.org/x/time/rate
4. **LRU Caching**: Implements caching with automatic 24-hour cache clearing
5. **Retry Logic**: Falls back to different DNS servers on query failures
6. **Configuration**: Auto-generates `config.yaml` on first run if missing

### HTTP API

- **GET /**: Health check endpoint
- **GET /domain/{domain}**: DNS reconnaissance for specified domain
- **Port**: 8080 (hardcoded)
- **Response**: JSON with complete DNS record set and metadata

### Data Flow

1. HTTP request received for domain lookup
2. DNS client retrieved from pool (with timeout)
3. Concurrent goroutines query different record types
4. SOA/A/AAAA validated first (domain existence check)
5. If valid, additional record types queried (NS, MX, TXT, CNAME)
6. Results aggregated into structured JSON response
7. Client returned to pool for reuse

### Configuration Files

- `config.yaml`: Auto-generated on first run, controls max DNS servers
- `resolvers.yaml`: Referenced in README but not found in codebase (TODO item)

### Dependencies

- `github.com/miekg/dns`: DNS protocol implementation
- `github.com/gorilla/mux`: HTTP routing
- `golang.org/x/time/rate`: Rate limiting
- `github.com/golang/groupcache/lru`: LRU cache
- `gopkg.in/yaml.v2`: YAML configuration

## Testing

No test files or testing framework detected in the codebase. Manual testing via curl:

```bash
curl http://127.0.0.1:8080/domain/google.com | json_pp
```