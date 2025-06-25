# Docker apps with nginx reverse proxy

## Overview
A containerized Nginx reverse proxy is setup to listen on localhost:8080 that forwards requests to the appropriate endpoints on 2 different apps named service1 and service2 running on separate
docker containers.

## Routing
Requests that match `service1/` are forwarded to port 8001 of service1 container.

Requests that match `service2/` are forwarded to port 8002 of service2 container.

##### nginx.conf
```nginx
    location /service1/ {
        proxy_pass http://service1:8001/;
        rewrite ^/service1(/.*)$ $1 break;
    }

    location /service2/ {
        proxy_pass http://service2:8002/;
        rewrite ^/service2(/.*)$ $1 break;
    }

    location = /favicon.ico {
           access_log off;
           log_not_found off;
           return 204;
    }
```
The rewrite directive uses regex to match everything after service1 or service2 (/ping and /hello endpoints)
and adds it to a capture group which is later referenced with the $1 variable to be passed on to the request.

Logging the 404 error for the missing favicon is disabled to make the logs cleaner.

## Health checks

A health check is configured in the docker-compose file that uses curl to try and access the /ping endpoint of each service every 30s.

##### docker-compose.yml
```yaml
healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8001/ping"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s

healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8002/ping"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s
```

## Logging

A custom log_format is specified which includes a timestamp, request path, status code and request time.

##### nginx.conf
```nginx
log_format custom_log '$time_local | $remote_addr | "$request" '
                      '| upstream: $upstream_addr | status: $status '
                      '| request_time: $request_time';

access_log /dev/stdout custom_log;
```
## Setup and run

```bash
git clone https://github.com/liquidsworrds/dpdzero-nginx-reverse-proxy
```

```bash
cd dpdzero-nginx-reverse-proxy/
```

```bash
docker-compose up --build
```

**Serivce 1**
- **ping**: http://localhost:8080/service1/ping
- **hello**: http://localhost:8080/service1/hello

**Serivce 2**
- **ping**: http://localhost:8080/service2/ping
- **hello**: http://localhost:8080/service2/hello

## Stop and remove
```bash
docker-compose down
```
