
# RabbitMQ Heartbeat Timeout Resolution Guide

Hello,

Based on the logs you’ve shared, I’m confident that the connection drops are caused—almost certainly—by a **heartbeat configuration issue** on the RabbitMQ side. This type of issue typically occurs when the server does not receive heartbeat frames from the client within the expected interval, most often due to mismatched or missing timeout settings between client and server.

Below is a step-by-step guide to help you locate the relevant configuration files, update them properly, and apply the changes.

---

## 1. Locating RabbitMQ Configuration Files

RabbitMQ usually loads configuration from one of the following paths inside the container:

- `/etc/rabbitmq/rabbitmq.conf`
- Any `.conf` files inside `/etc/rabbitmq/conf.d/`

To check what exists in these paths:

```bash
sudo docker exec -it <container_id> ls -la /etc/rabbitmq/
sudo docker exec -it <container_id> ls -la /etc/rabbitmq/conf.d
```

To view the currently active configuration:

```bash
sudo docker exec -it <container_id> rabbitmq-diagnostics environment
```

This will display all runtime settings including `heartbeat`, `tcp_listen_options`, and other network-related parameters.

---

## 2. Updating the Configuration

To extend the server-side heartbeat timeout, you can update the configuration as follows:

### a. If `/etc/rabbitmq/rabbitmq.conf` exists:

Edit the file:

```bash
sudo docker exec -it <container_id> vi /etc/rabbitmq/rabbitmq.conf
```

Add or update the following line:

```ini
heartbeat = 120
```

### b. If you are using the `conf.d` directory instead:

Create a new config file:

```bash
sudo docker exec -it <container_id> sh -c "echo 'heartbeat = 120' > /etc/rabbitmq/conf.d/heartbeat.conf"
```

You can also modify existing `.conf` files inside this directory if necessary.

---

## 3. Applying the Changes

RabbitMQ needs to be restarted for the configuration changes to take effect:

```bash
sudo docker restart <container_id>
```

Then monitor the container logs to ensure the changes resolve the connection issue:

```bash
sudo docker logs -f <container_id>
```

---

## Additional Suggestions

- **Make sure the client also uses a matching heartbeat value**, such as `heartbeat=120` in the connection string.
- If the issue persists, further checks may be needed for network latency, resource constraints (CPU/memory), or container-level throttling.

---

## Example `.conf` File

You can create a config file under `/etc/rabbitmq/conf.d/heartbeat.conf` with the following content:

```ini
# /etc/rabbitmq/conf.d/heartbeat.conf

# Extends the heartbeat timeout interval to reduce unintended connection drops.
heartbeat = 120

# Optional: Tune TCP options if needed
# tcp_listen_options.backlog = 128
# tcp_listen_options.nodelay = true
# tcp_listen_options.keepalive = true
```

This file will be picked up automatically if placed in the correct directory, and the new settings will be active after restarting the container.

Best regards,  
Veysi
