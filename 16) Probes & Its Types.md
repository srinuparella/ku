# 🚀 Kubernetes Probes – Complete Guide

Kubernetes uses **probes** to check the health of containers.  
They help decide **when to restart, when to send traffic, and when a pod is fully started**.

---

## 🔹 Types of Probes

### 1. **Liveness Probe**
- Checks if the container is still alive.  
- If it fails → Kubernetes **restarts** the container.  
- Example: app is stuck in deadlock → restart.

### 2. **Readiness Probe**
- Checks if the container is ready to serve traffic.  
- If it fails → Pod is marked **NotReady** and removed from Service endpoints.  
- Example: app needs to load configs before handling requests.

### 3. **Startup Probe**
- Checks if the container has started successfully.  
- While this probe is running, **liveness and readiness are disabled**.  
- Example: slow-starting apps (databases, JVM apps).

---

## 🔹 Probe Mechanisms
Each probe type can use **one of three mechanisms**:

1. **httpGet** → sends HTTP GET request (success = HTTP 200–399).  
2. **tcpSocket** → tries to open a TCP connection (success = connection works).  
3. **exec** → runs a command inside container (success = exit code 0).  

---

## 🔹 Probe Parameters

| Parameter              | Meaning                                                                 | Example |
|------------------------|-------------------------------------------------------------------------|---------|
| **initialDelaySeconds** | Wait time before first probe runs.                                      | `5` |
| **periodSeconds**       | How often probe runs.                                                   | `10` |
| **timeoutSeconds**      | Max time to wait for response before marking fail.                      | `3` |
| **successThreshold**    | How many **successes in a row** are required to mark as healthy.        | `1` |
| **failureThreshold**    | How many **fails in a row** before action (restart / not-ready / kill). | `3` |

---

## 🔹 Example Pod with All Three Probes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probes-detailed
  labels:
    app: probes
spec:
  containers:
  - name: app-container
    image: busybox
    command:
      - sh
      - -c
      - |
        # simulate container lifecycle
        echo "container starting..."
        sleep 10
        echo "started" > /tmp/start   # used by startup probe
        echo "healthy" > /tmp/healthy # used by liveness probe
        echo "ready" > /tmp/ready     # used by readiness probe
        sleep 3600

    # 1. Startup Probe (exec)
    startupProbe:
      exec:
        command:
          - cat
          - /tmp/start
      initialDelaySeconds: 5     # wait 5s before first probe
      periodSeconds: 5           # check every 5s
      timeoutSeconds: 2          # fail if command >2s
      successThreshold: 1        # 1 success = started
      failureThreshold: 30       # allow 150s (30×5) before fail

    # 2. Liveness Probe (httpGet)
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 10    # wait 10s before check
      periodSeconds: 10          # check every 10s
      timeoutSeconds: 3          # fail if no HTTP response in 3s
      successThreshold: 1        # 1 success = alive
      failureThreshold: 3        # 3 fails = restart container

    # 3. Readiness Probe (tcpSocket)
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5     # wait 5s before checking
      periodSeconds: 5           # check every 5s
      timeoutSeconds: 2          # fail if no TCP response in 2s
      successThreshold: 1        # 1 success = Ready
      failureThreshold: 3        # 3 fails = remove from Service

    ports:
    - containerPort: 8080
    - containerPort: 80
```

---

## 🔹 Explanation of `sh -c |`

- **sh** → start shell inside container.  
- **-c** → run the following string as a command.  
- **|** → YAML multiline string (preserves newlines).  

So this block:

```yaml
command:
  - sh
  - -c
  - |
    echo "starting"
    sleep 10
    echo "ready" > /tmp/ready
    sleep 3600
```

is the same as running:

```bash
sh -c "
  echo 'starting'
  sleep 10
  echo ready > /tmp/ready
  sleep 3600
"
```

This lets us run multiple commands in sequence when the container starts.

---

## ✅ Summary
- Probes keep containers healthy in Kubernetes.  
- **Liveness = restart if dead**.  
- **Readiness = control traffic**.  
- **Startup = wait until boot done**.  

All probes can use **httpGet, tcpSocket, or exec**.  
Parameters like `initialDelaySeconds`, `periodSeconds`, etc. control when and how often probes run.  

Using `sh -c |` allows us to simulate app behavior with `echo` for probe testing.
