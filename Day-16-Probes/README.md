
 A Hands-On Guide: Kubernetes Probes – Liveness, Readiness, and Startup Probes


Kubernetes uses **Probes** to monitor the health of containers running inside Pods. Probes are essential for ensuring your containers are working properly and can handle traffic. They are a key component of Kubernetes’ **self-healing** capabilities, enabling:

* 🔁 Automatic restarts
* 📊 Load balancing
* ✅ Health checks


## 🔍 What Are Kubernetes Probes?

Kubernetes **Probes** are diagnostic tools used to determine the health of a container.

They help Kubernetes:

* 🔁 Restart failed containers.
* 🏁 Ensure containers are initialized before serving traffic.
* 🛡️ Avoid routing traffic to unready containers.

Types of probes:

* ❤️ **Liveness Probe** – Is the app still running?
* 🟢 **Readiness Probe** – Is it ready to serve traffic?
* 🚀 **Startup Probe** – Has the app fully started?

---

## 🔧 Types of Probes

### ❤️ Liveness Probe

* **Purpose**: Checks if the container is still alive.
* **When it fails**: Kubernetes restarts the container.
* **Use Case**: Detects hung or crashed applications.

#### 🔨 How It Works:

* HTTP GET request 🌐
* TCP socket check 🔌
* Command execution inside the container 🖥️

#### 📌 Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5
```

> ✅ This example sends a GET request to `/health`. If the response is 2xx–3xx, the container is considered healthy.

---

### 🟢 Readiness Probe

* **Purpose**: Checks if the app is ready to serve traffic.
* **When it fails**: Container stays in service but is removed from the load balancer.
* **Use Case**: Ensures only healthy instances receive traffic.

#### 📌 Example:

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

> 🛡️ Ensures the container won’t receive traffic until it's ready.

---

### 🚀 Startup Probe

* **Purpose**: Checks if the app has finished starting.
* **When it fails**: Kubernetes assumes the app never started and restarts it.
* **Use Case**: For apps that take longer to initialize.

#### 📌 Example:

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

> ⏱️ Useful for legacy or slow-starting applications.

---

## ❓ Why Do We Use Probes?

* ✅ Self-healing for stuck/crashed apps
* 🧭 Smarter traffic routing
* 🔄 Automatic lifecycle management
* 🚫 Prevent sending requests to unready apps

---

## ⚖️ Differences Between Probes

| Feature       | Liveness Probe ❤️ | Readiness Probe 🟢 | Startup Probe 🚀 |
| ------------- | ----------------- | ------------------ | ---------------- |
| Checks If     | App is alive      | App is ready       | App has started  |
| Effect on Pod | Pod restarted     | Removed from LB    | Pod restarted    |
| Used When     | App crashes/hangs | App initializing   | App takes time   |

---

## 🧠 Best Practices for Probes

* Use `startupProbe` for slow-starting apps.
* Combine `readinessProbe` and `livenessProbe` for robust services.
* Tune delays and timeouts to match your app behavior.
* Test probes locally before deploying to production.

---

## ✅ Conclusion

Kubernetes Probes are vital for building **resilient, reliable, and scalable** applications.

* ❤️ **Liveness**: Detects and recovers from failure.
* 🟢 **Readiness**: Ensures smooth traffic handling.
* 🚀 **Startup**: Supports complex app bootstrapping.




    

    
