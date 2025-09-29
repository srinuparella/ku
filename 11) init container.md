<!-- Date:02-09-2025 -->

# Kubernetes Pod with Init Container + `emptyDir` — Step‑by‑Step Breakdown

This document explains, line by line, what the following Pod manifest does and how it works at runtime.

---

## 📄 Full Manifest (Reference)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
    - name: init-co
      image: alpine
      command:
        - sh
        - -c
        - echo '<h1> this my main application data </h1>' > /usr/share/nginx/html/index.html
      volumeMounts:
        - name: init-data
          mountPath: /usr/share/nginx/html

  containers:
    - name: main-app
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: init-data
          mountPath: /usr/share/nginx/html

  volumes:
    - name: init-data
      emptyDir: {}
```

---

## 1) Header

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
```
- **apiVersion: v1** — Uses the core Kubernetes API group (stable, built‑in types).
- **kind: Pod** — We are creating a *Pod* resource (the smallest deployable unit in K8s).
- **metadata.name** — The Pod will be named **`init-pod`**.

---

## 2) Init Container

```yaml
spec:
  initContainers:
    - name: init-co
      image: alpine
      command:
        - sh
        - -c
        - echo '<h1> this my main application data </h1>' > /usr/share/nginx/html/index.html
      volumeMounts:
        - name: init-data
          mountPath: /usr/share/nginx/html
```
- **initContainers** — Special containers that **must complete successfully** before normal containers start.
- **name: init-co** — Logical name for the init container.
- **image: alpine** — Minimal Linux image; small, fast to pull.
- **command (array form)** — Runs the shell with a command string:
  - `sh` — POSIX shell available in Alpine.
  - `-c` — *Execute the following string as a command*.
  - `echo '<h1> this my main application data </h1>' > /usr/share/nginx/html/index.html` — Writes the HTML into a file.

> 🔎 **Why `sh -c`?**
> It lets you use shell features such as redirection (`>`), pipes, `&&`, etc., inside one command string.

> 🔁 **What does `>` do?**
> `>` redirects the command output to a file (creates/overwrites). Use `>>` to append instead of overwrite.

- **volumeMounts** — Mounts the shared volume **`init-data`** at `/usr/share/nginx/html`.  
  Anything written here is persisted in the volume and becomes visible to other containers that mount the same volume.

---

## 3) Main (App) Container

```yaml
  containers:
    - name: main-app
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: init-data
          mountPath: /usr/share/nginx/html
```
- **containers** — Regular containers of the Pod (start after all init containers succeed).
- **image: nginx** — Nginx web server serving files from `/usr/share/nginx/html` by default.
- **ports.containerPort: 80** — Documents that this container listens on port 80.
- **volumeMounts** — Mounts the **same** `init-data` volume at the same path so nginx can serve the file created by the init container.

---

## 4) Volumes

```yaml
  volumes:
    - name: init-data
      emptyDir: {}
```
- **volumes** — Defines storage that containers can mount.
- **emptyDir: {}** — A temporary directory **created when the Pod starts** and **deleted when the Pod is removed**.
  - Lives for the **lifetime of the Pod**.
  - Shared across **all containers** in the Pod that mount it.
  - Great for **init -> app hand‑off** patterns like this one.

> 💡 If you need data to persist beyond the Pod’s life, use a **PersistentVolumeClaim** instead of `emptyDir`.

---

## 5) Execution Flow (What Happens at Runtime)

```
Pod created
 └─► Init container `init-co` starts
      └─► Runs: sh -c "echo '<h1> ... </h1>' > /usr/share/nginx/html/index.html"
           └─► Writes file into shared volume `init-data`
 └─► Init container completes successfully
 └─► Main container `main-app` (nginx) starts
      └─► Mounts the same volume at /usr/share/nginx/html
      └─► Serves /usr/share/nginx/html/index.html on port 80
```

---

## 6) How to Apply and Test

```bash
# Apply
kubectl apply -f init-pod.yaml

# Wait until the Pod is Ready (init must finish first)
kubectl get pod init-pod -w

# Quick test via port-forward (local machine -> Pod)
kubectl port-forward pod/init-pod 8080:80

# In another terminal or browser:
curl http://localhost:8080
# or open http://localhost:8080 in your browser
```

Expected response:
```html
<h1> this my main application data </h1>
```

---

## 7) Common Pitfalls & Tips

- **Quoting:** If your HTML contains quotes, escape or switch quote types appropriately.
- **Overwriting vs Appending:** Use `>` to overwrite (fresh file), `>>` to append lines from multiple init steps.
- **Path correctness:** Ensure you write to the **mounted path**, not the image’s original path unless it’s mounted.
- **Image availability:** Alpine and Nginx images are public; ensure the cluster can pull images (internet access or local registry mirror).

---

## 8) Variations (Optional)

- **Append content with multiple init containers:**
  ```yaml
  initContainers:
    - name: init-1
      image: alpine
      command: ["sh","-c","echo '<h1>Title</h1>' > /usr/share/nginx/html/index.html"]
    - name: init-2
      image: alpine
      command: ["sh","-c","echo '<p>Paragraph</p>' >> /usr/share/nginx/html/index.html"]
  ```

- **Expose externally (NodePort Service):**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: init-pod-svc
  spec:
    type: NodePort
    selector:
      # As this is a naked Pod, you'd need labels in the Pod metadata first:
      # metadata:
      #   labels: { app: init-pod }
      app: init-pod
    ports:
      - port: 80
        targetPort: 80
  ```

---

## 9) Cleanup

```bash
kubectl delete pod init-pod
```

---

### ✅ TL;DR
An **init container** writes an HTML file into a shared **`emptyDir`** volume.  
After it finishes, the **nginx** container starts and serves that file on **port 80**.
