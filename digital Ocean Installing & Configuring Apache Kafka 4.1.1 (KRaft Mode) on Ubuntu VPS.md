Great — I'll prepare a **clean, professional, GitHub-ready documentation** that explains:

✔ Kafka 4.1.1 Installation on Ubuntu VPS
✔ KRaft configuration (single node)
✔ Error debugging steps
✔ Topic creation
✔ Producer & consumer testing
✔ Restart, reset, reinstallation
✔ Java (JDK 21) fix
✔ System overview

It will follow perfect GitHub style with headings, code blocks, and explanations.

---

# 📘 **GitHub Documentation: Installing & Configuring Apache Kafka 4.1.1 (KRaft Mode) on Ubuntu VPS**

You can copy-paste this directly into a GitHub README.md of your repo.

---

# 🚀 **Apache Kafka 4.1.1 (KRaft Mode) Installation Guide – Ubuntu 22.04 (DigitalOcean VPS)**

This guide documents the full installation process of **Kafka 4.1.1** in **KRaft Mode** (no ZooKeeper) — including setup, configuration, debugging, and message testing.

---

## 📋 **Prerequisites**

* Ubuntu 22.04 or newer
* sudo access
* Port access: **9092** (broker), **9093** (controller)
* Public IP: `159.89.172.251`
* Java 17 or 21 installed (`openjdk-21` recommended)

Check Java:

```bash
java -version
```

Correct Java Home used:

```bash
/usr/lib/jvm/java-21-openjdk-amd64
```

---

# 1️⃣ **Download & Install Kafka 4.1.1**

```bash
cd /opt
sudo wget https://downloads.apache.org/kafka/4.1.1/kafka_2.13-4.1.1.tgz
sudo tar -xvf kafka_2.13-4.1.1.tgz
sudo mv kafka_2.13-4.1.1 kafka
sudo chmod -R 755 /opt/kafka
```

Create log directory:

```bash
sudo mkdir -p /opt/kafka/logs
```

---

# 2️⃣ **Configure Kafka for KRaft Mode**

Edit config:

```bash
sudo nano /opt/kafka/config/server.properties
```

Replace all content with:

```properties
controller.quorum.bootstrap.servers=localhost:9093
listeners=PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
inter.broker.listener.name=PLAINTEXT
advertised.listeners=PLAINTEXT://159.89.172.251:9092
controller.listener.names=CONTROLLER
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kraft-combined-logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
share.coordinator.state.topic.replication.factor=1
share.coordinator.state.topic.min.isr=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
controller.quorum.voters=1@localhost:9093

```

---

# 3️⃣ **Format the KRaft Storage Directory**

Generate UUID:

```bash
sudo /opt/kafka/bin/kafka-storage.sh random-uuid
```

Format storage:

```bash
sudo /opt/kafka/bin/kafka-storage.sh format \
  -t <UUID> \
  -c /opt/kafka/config/server.properties
```

Example:

```bash
sudo /opt/kafka/bin/kafka-storage.sh format \
  -t tvFT85LHSySpjeBXFfOtjg \
  -c /opt/kafka/config/server.properties
```

---

# 4️⃣ **Start Kafka Server**

Start in foreground:

```bash
sudo /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
```

Kafka is ready once you see:

```
KRaft controller started
Broker started on PLAINTEXT://0.0.0.0:9092
```

---

# 5️⃣ **Test Kafka: Topic, Producer, Consumer**

### Create topic:

```bash
/opt/kafka/bin/kafka-topics.sh \
  --create \
  --topic test1 \
  --bootstrap-server 159.89.172.251:9092
```

### Test producer:

```bash
/opt/kafka/bin/kafka-console-producer.sh \
  --topic test1 \
  --bootstrap-server 159.89.172.251:9092
```

Send messages:

```
hello kafka
test message
```

### Test consumer:

```bash
/opt/kafka/bin/kafka-console-consumer.sh \
  --topic test1 \
  --from-beginning \
  --bootstrap-server 159.89.172.251:9092
```

You should see:

```
hello kafka
test message
```

---

# 🐞 6️⃣ **Debugging Common Errors**

## ❌ **Error: UNKNOWN_TOPIC_OR_PARTITION**

Occurs when topic auto-creation triggered late.

Fix: manually create topic before producing.

---

## ❌ **Error: controller.quorum.voters is not set**

Cause: Missing KRaft config.

Fix: Ensure these lines exist:

```properties
controller.listener.names=CONTROLLER
controller.quorum.voters=1@localhost:9093
```

---

## ❌ **GraalVM error**

```
/opt/graalvm/graalvm/bin/java: No such file or directory
```

Fix JAVA_HOME:

```bash
sudo nano /etc/environment
```

Add:

```
JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
```

Reload:

```bash
source /etc/environment
source ~/.bashrc
```

---

## ❌ **Consumer gets no messages**

Causes:

* Wrong `advertised.listeners`
* Old metadata in `/opt/kafka/logs`
* KRaft not formatted properly

Fix:

```bash
sudo rm -rf /opt/kafka/logs/*
sudo /opt/kafka/bin/kafka-storage.sh random-uuid
sudo /opt/kafka/bin/kafka-storage.sh format -t <UUID> -c /opt/kafka/config/server.properties
```

---

# 🔁 7️⃣ **How to Reset / Reinstall Kafka**

Stop Kafka:

```bash
pkill -f kafka
```

Remove installation:

```bash
sudo rm -rf /opt/kafka
sudo rm -f /etc/systemd/system/kafka.service
sudo systemctl daemon-reload
```

Reinstall using steps above.

---

# 🛠 8️⃣ **Optional: Run Kafka as a Systemd Service**

Create service file:

```bash
sudo nano /etc/systemd/system/kafka.service
```

Add:

```ini
[Unit]
Description=Apache Kafka 4.1.1 (KRaft Mode)
After=network.target

[Service]
Type=simple
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable kafka
sudo systemctl start kafka
```

Check:

```bash
sudo systemctl status kafka
```

---

# 📦 **Summary**

This document covered:

* Installing Kafka 4.1.1
* Setting up KRaft mode
* Fixing JAVA_HOME issues
* Correct listener configuration
* Metadata formatting
* Producer/consumer testing
* Debugging all errors
* Clean reinstallation
* Optional systemd service

---

If you want, I can also generate:

✔ **GitHub Wiki version (multi pages)**
✔ **Architecture diagram (PNG)**
✔ **ASCII Kafka cluster diagram**
✔ **Spring Boot integration code section**

Just tell me:
**“Add wiki version”** or **“Add Spring Boot section”**.
