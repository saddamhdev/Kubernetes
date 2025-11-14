Perfect — you want Kafka 4.1.1 (KRaft mode) to run as a **systemd service** so it keeps running even after you close the terminal.

I will give you the **correct systemd service file** that works with **your final Kafka configuration** (including your preferred KRaft config stored in memory).

This ensures:

✔ Kafka starts at boot
✔ Runs in background
✔ Restarts if crashed
✔ No need to keep terminal open
✔ Compatible with KRaft mode

---

# ✅ **STEP 1 — Create Kafka Systemd Service File**

Run:

```bash
sudo nano /etc/systemd/system/kafka.service
```

Paste the following **exact working config**:

```ini
[Unit]
Description=Apache Kafka 4.1.1 (KRaft Mode)
After=network.target

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5

ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

Save file:

* CTRL + O
* ENTER
* CTRL + X

---

# ✅ **STEP 2 — Reload systemd**

```bash
sudo systemctl daemon-reload
```

---

# ✅ **STEP 3 — Enable Kafka to start on boot**

```bash
sudo systemctl enable kafka
```

You should see:

```
Created symlink /etc/systemd/system/multi-user.target.wants/kafka.service
```

---

# ✅ **STEP 4 — Start Kafka**

```bash
sudo systemctl start kafka
```

Check status:

```bash
sudo systemctl status kafka
```

You should see:

```
Active: active (running)
```

---

# 🧪 **STEP 5 — Verify Kafka is working**

Test topic list:

```bash
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server 159.89.172.251:9092
```

Test producer:

```bash
/opt/kafka/bin/kafka-console-producer.sh --topic test1 --bootstrap-server 159.89.172.251:9092
```

Test consumer:

```bash
/opt/kafka/bin/kafka-console-consumer.sh --topic test1 --from-beginning --bootstrap-server 159.89.172.251:9092
```

---

# 👍 **Now you can close SSH terminal**

Kafka will **continue running** in the background.

---

# 🔧 Optional Management Commands

### Restart Kafka:

```bash
sudo systemctl restart kafka
```

### Stop Kafka:

```bash
sudo systemctl stop kafka
```

### View Logs:

```bash
journalctl -u kafka -f
```

---

# 🎉 Kafka is now a permanent background service!

If you want, I can also set up:

* **ZGC GC tuning**
* **JVM heap settings**
* **Dedicated Kafka user instead of root**
* **SASL/SCRAM authentication**
* **Prometheus + Grafana monitoring**

Just tell me:
**“secure kafka”**, **“optimize kafka”**, or **“monitor kafka”**.
