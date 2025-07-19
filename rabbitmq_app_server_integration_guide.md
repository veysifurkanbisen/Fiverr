
# App Server Integration Guide: Connecting to RabbitMQ EC2 Server

This guide explains how to configure your application servers (EC2 instances) to successfully connect to a RabbitMQ server hosted on a separate EC2 instance.

---

## 1. Security Group Configuration (Networking)

Ensure the **RabbitMQ EC2 instance** has an associated security group that:

- Allows **inbound TCP traffic on port 5672** (AMQP) **from the private IPs or security group** of the application servers.
- (Optional) Allows port **15672** if you want access to the RabbitMQ management UI from app servers or developers.

### Example Inbound Rule for RabbitMQ EC2

| Type        | Protocol | Port Range | Source              |
|-------------|----------|------------|---------------------|
| Custom TCP  | TCP      | 5672       | App server SG or IP |
| (Optional)  | TCP      | 15672      | App server SG or IP |

---

## 2. RabbitMQ Connection Configuration

Your application must use a proper AMQP URI to connect to the RabbitMQ server.

### AMQP URI Format:

```
amqp://<username>:<password>@<rabbitmq-server-private-ip>:5672
```

- Replace `<username>` and `<password>` with the actual credentials (e.g., `admin`, `strongpassword123`).
- Replace `<rabbitmq-server-private-ip>` with the private IP of the EC2 server running RabbitMQ.

Example:

```
amqp://admin:strongpassword123@10.0.2.15:5672
```

---

## 3. Connectivity Test (Before Running Code)

From each app server, verify that the RabbitMQ server is reachable on port 5672:

```bash
telnet <rabbitmq-server-private-ip> 5672
```

or

```bash
nc -zv <rabbitmq-server-private-ip> 5672
```

If the connection is successful, you're ready to test your producer/consumer application.

---

## 4. Minimal Python Example for Testing (Pika Library)

Install Pika:

```bash
pip install pika
```

Create a test script:

```python
import pika

params = pika.URLParameters('amqp://admin:strongpassword123@10.0.2.15:5672')
connection = pika.BlockingConnection(params)
channel = connection.channel()
channel.queue_declare(queue='test-queue')

channel.basic_publish(exchange='', routing_key='test-queue', body='Hello from app server!')
print("Message sent successfully.")
connection.close()
```

---

## 5. Common Troubleshooting Tips

- Ensure correct username/password is used.
- Make sure the queue exists (or use `queue_declare` as above to auto-create).
- Double-check security group rules and private IP addresses.
- Verify RabbitMQ service is running using `docker ps` and `docker logs`.

