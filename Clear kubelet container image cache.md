To **remove / clear cache in Kubernetes**, you must be clear what kind of "cache" you want to delete — because Kubernetes itself does not have a single cache.
Different layers have different caches.

So tell me which one you want.

Below I listed **all cache types** and **how to clear each**.

---

# ✅ 1. **Clear kubelet container image cache (containerd / k3s cluster)**

If you are using **k3s**, your container runtime is **containerd**, so run:

### **List images**

```bash
sudo k3s crictl images
```

### **Remove unused images**

```bash
sudo k3s crictl rmi --all
```

### **Force garbage collection**

```bash
sudo k3s crictl rmp --all
```

This removes old cached Docker/containerd layers.

---

# ✅ 2. **Clear Kubernetes DNS / CoreDNS cache**

CoreDNS caches DNS results.

Reload CoreDNS:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

Or delete the pod:

```bash
kubectl delete pod -n kube-system -l k8s-app=kube-dns
```

---

# ✅ 3. **Clear Ingress NGINX cache**

Ingress NGINX keeps:

* SSL session cache
* LRU proxy cache
* Upstream cache

Restart Ingress:

```bash
kubectl rollout restart deployment ingress-nginx-controller -n ingress-nginx
```

OR delete pod:

```bash
kubectl delete pod -n ingress-nginx -l app.kubernetes.io/component=controller
```

---

# ✅ 4. **Clear Redis cache (if you use Redis)**

If your app uses Redis for caching:

```bash
redis-cli FLUSHALL
```

Or for one DB:

```bash
redis-cli FLUSHDB
```

---

# ✅ 5. **Clear local cluster file-system (cache directories)**

Kubernetes uses `/var/lib` folders for containerd + kubelet cache.

### Delete containerd cache:

```bash
sudo rm -rf /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/*
```

### Delete k3s image cache:

```bash
sudo rm -rf /var/lib/rancher/k3s/agent/images/*
```

---

# ✅ 6. **Clear Pod logs (big memory usage)**

Logs stored in:

```
/var/lib/docker/containers
or
/var/lib/rancher/k3s/agent/containerd/io.containerd
```

Clear logs:

```bash
sudo find /var/lib -name "*.log" -type f -delete
```

---

# ⚠️ **If you want to free RAM immediately:**

Just restart kubelet:

```bash
sudo systemctl restart k3s
```

Or restart all pods:

```bash
kubectl delete pod --all -A
```

---

# 🧠 **Which “cache” did you mean?**

Tell me:

➡ Redis cache?
➡ Ingress cache?
➡ Docker/containerd image cache?
➡ Pod logs?
➡ DNS cache?
➡ System memory cache?

If you want to **free RAM**, I can give you the safest method.
