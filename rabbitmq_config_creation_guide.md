
# Creating and Mounting `rabbitmq.conf` for Docker Compose

This guide explains how to correctly create and mount a custom `rabbitmq.conf` configuration file when deploying RabbitMQ with Docker Compose.

---

## Step 1: Create a working directory

Choose a location on your EC2 server to store your RabbitMQ setup:

```bash
mkdir ~/rabbitmq-setup
cd ~/rabbitmq-setup
```

---

## Step 2: Create the `rabbitmq.conf` file

Use a text editor to create the configuration file:

```bash
nano rabbitmq.conf
```

Paste the following content into the file:

```ini
# rabbitmq.conf

heartbeat = 120
vm_memory_high_watermark = 0.7
disk_free_limit.absolute = 2GB
loopback_users.guest = false
```

Save and close the file. This file must remain in the same folder as your `docker-compose.yml`.

---

## Step 3: Create the `docker-compose.yml` file

In the same directory:

```bash
nano docker-compose.yml
```

Make sure your Compose file includes the following volume mapping:

```yaml
volumes:
  - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf:ro
```

This ensures the config file is loaded inside the container at the expected path.

---

## Step 4: Start RabbitMQ

```bash
docker-compose up -d
```

---

## Step 5: Verify the Configuration

Check the logs:

```bash
docker logs -f rabbitmq
```

Verify the applied configuration:

```bash
docker exec -it rabbitmq rabbitmq-diagnostics environment
```

If you see your values (e.g., `heartbeat = 120`) in the output, the configuration is successfully applied.
