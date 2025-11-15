You are **90% done**, but now we hit a COMMON, CRITICAL issue:

# ❌ **Spring Boot cannot connect to your VPS Kafka**

Even though:

✔ Kafka is running
✔ You can produce/consume from VPS terminal
✔ Topic exists
✔ Spring Boot starts normally

…but Spring Boot logs say:

```
Disconnecting from node -1 due to socket connection setup timeout.
Bootstrap broker 159.89.172.251:9092 disconnected
```

This means:

# 🟥 **Kafka is NOT reachable externally**

Spring Boot cannot open a TCP connection to **159.89.172.251:9092**.

This happens only for 2 reasons.

---

# 🟥 1. **Firewall/DO Networking is blocking external Kafka traffic**

DigitalOcean Firewall **blocks 9092 by default**.

### ✔ FIX: Open the ports

If you use UFW:

```bash
sudo ufw allow 9092
sudo ufw allow 9093
sudo ufw reload
```

If using DigitalOcean Cloud Firewall:

Go to
`DigitalOcean → Networking → Firewalls → Your Droplet → Inbound Rules`

Add inbound rules:

| Port | Protocol | Source    |
| ---- | -------- | --------- |
| 9092 | TCP      | 0.0.0.0/0 |
| 9093 | TCP      | 0.0.0.0/0 |

⚠ MUST ALLOW from **your Spring Boot server IP**, at least.

---

# 🟥 2. **Kafka binding is correct, BUT advertised.listeners was not applied properly**

Spring Boot sees Kafka saying:

✔ "I am available at localhost:9092"
❌ Not at your VPS public IP

This happens if:

* Wrong systemd environment
* Old Kafka instance still running
* Wrong config file used
* Duplicate server.properties
* Missing reboot after reinstall

---

# 🟩 CHECKLIST (Run these commands NOW)

### ✔ 1. Check what Kafka is actually advertising

On VPS:

```bash
/opt/kafka/bin/kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --entity-type brokers \
  --entity-name 1 \
  --describe
```

Output MUST include:

```
advertised.listeners=PLAINTEXT://159.89.172.251:9092
```

If it shows:

```
advertised.listeners=PLAINTEXT://localhost:9092
```

→ Spring Boot **will fail ALWAYS**

---

### ✔ 2. Check your systemd service is reading the right config

Show service file:

```bash
cat /etc/systemd/system/kafka.service
```

Must contain:

```
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
```

---

### ✔ 3. Check correct config file:

```bash
cat /opt/kafka/config/server.properties | grep advertised
```

Expected:

```
advertised.listeners=PLAINTEXT://159.89.172.251:9092
```

---

### ✔ 4. Check if port 9092 is listening:

```bash
sudo lsof -i :9092
```

Expected:

```
java    xxx   ...   TCP *:9092 (LISTEN)
```

---

### ✔ 5. Check external connectivity from your home PC (VERY IMPORTANT)

Run:

```
telnet 159.89.172.251 9092
```

or:

```
nc -vz 159.89.172.251 9092
```

If it shows:

```
Connection refused / timeout
```

→ The port is BLOCKED.

---

# 🟢 100% SOLUTION (Do these steps in order)

---

## ✔ Step 1 — Fix Firewall (DO or UFW)

#### UFW:

```bash
sudo ufw allow 9092/tcp
sudo ufw allow 9093/tcp
sudo ufw reload
```

#### DigitalOcean Cloud Firewall (if enabled):

Add inbound:

```
TCP 9092  → 0.0.0.0/0
TCP 9093  → 0.0.0.0/0
```

---

## ✔ Step 2 — Fix advertised.listeners

Edit:

```
sudo nano /opt/kafka/config/server.properties
```

Ensure:

```
advertised.listeners=PLAINTEXT://159.89.172.251:9092
```

NO localhost
NO wrong IP
NO missing listener

---

## ✔ Step 3 — RELOAD Kafka (required)

Stop:

```bash
sudo systemctl stop kafka
```

Delete metadata:

```bash
sudo rm -rf /tmp/kraft-combined-logs/*
```

Format again:

```bash
uuid=$(sudo /opt/kafka/bin/kafka-storage.sh random-uuid)
sudo /opt/kafka/bin/kafka-storage.sh format -t $uuid -c /opt/kafka/config/server.properties
```

Restart:

```bash
sudo systemctl start kafka
sudo systemctl status kafka
```

---

## ✔ Step 4 — TEST externally

From your Spring Boot host:

```
nc -vz 159.89.172.251 9092
```

If it connects → problem is FIXED.

---

# 🎯 When you share **three outputs**, I will tell you EXACTLY what is wrong:

Send me:

```
cat /opt/kafka/config/server.properties | grep advertised
sudo lsof -i :9092
nc -vz 159.89.172.251 9092
```

I will diagnose 100% accurately.
