

# **Basic Docker Compose File Structure for Prometheus**

```yaml
version: "3.9"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - prometheus_data:/prometheus              # Named volume
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro   # Bind mount (config file)
    networks:
      - monitoring_net
    restart: unless-stopped

volumes:
  prometheus_data:                                # Named volume definition

networks:
  monitoring_net:
    external: true                                # Pre-created bridge network
```

### **What you must have before running this**

- Create the external docker network

```bash
docker network create monitoring_net
```

