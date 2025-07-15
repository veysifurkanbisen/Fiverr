
# Reliable RabbitMQ Deployment with Docker Compose

This document outlines a recommended setup for running RabbitMQ on an AWS EC2 Ubuntu server using Docker Compose, with a focus on reliability and stability under moderate message throughput.

---

## Minimum System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| vCPU     | 2       | 4           |
| RAM      | 4 GB    | 8 GB        |
| Disk     | 20 GB SSD | 50 GB SSD (gp3 or io2) |
| OS       | Ubuntu 20.04+ | Ubuntu 22.04 LTS |

> Note: It is strongly advised to use SSD-based storage (preferably `gp3` or `io2`) to ensure consistent disk I/O performance, especially for persistent messaging workloads.

---

## Docker Compose Configuration

Below is a Docker Compose setup designed for reliability and ease of monitoring:

```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: rabbitmq
    restart: unless-stopped
    hostname: rabbitmq
    ports:
      - "5672:5672"    # AMQP
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: strongpassword123
      RABBITMQ_VM_MEMORY_HIGH_WATERMARK: 0.7
      RABBITMQ_DISK_FREE_LIMIT: "2GB"
      RABBITMQ_HEARTBEAT: 120
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
      - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf:ro
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "check_running"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  rabbitmq_data:
```

---

## Configuration File: `rabbitmq.conf`

Create a file named `rabbitmq.conf` in the same directory as your Docker Compose file, with the following content:

```ini
# rabbitmq.conf

heartbeat = 120
vm_memory_high_watermark = 0.7
disk_free_limit.absolute = 2GB
loopback_users.guest = false
```

This configuration increases heartbeat timeout, sets safe memory and disk limits, and disables default guest access from outside the container.

---

## Post-Deployment Checklist

1. Start the services with `docker-compose up -d`.
2. Access the management UI at `http://<your-ec2-ip>:15672` using the credentials `admin / strongpassword123`.
3. Monitor resource usage with `docker stats`.
4. Verify configuration with `docker exec -it rabbitmq rabbitmq-diagnostics environment`.
5. Ensure the application properly uses acknowledgments, prefetch settings, and durable/persistent message flags to avoid queue buildup.

This setup is suitable for moderate traffic scenarios (e.g., a few hundred messages per minute) and designed to stay responsive under load, assuming client applications follow RabbitMQ best practices.
