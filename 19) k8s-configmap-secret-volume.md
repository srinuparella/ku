# Kubernetes ConfigMap, Secret, and Volume Notes

---

## 1. ConfigMap

### Purpose
- Store **non-sensitive configuration data** for pods.  
- Make pod-level configuration **dynamic without changing container images**.  
- Can be used in pods in two ways:  
  - `envFrom` → inject as environment variables  
  - `volumeMounts` → mount as files inside containers  

### Example Use Cases
- Database configuration (DB name, root password).  
- External URLs (GitHub, SonarQube) for communication with the pod.  

### Example ConfigMap (`config.yml`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-db
data:
  MYSQL_ROOT_PASSWORD: abcd
  MYSQL_DATABASE: config
  github_url: https://github.com/srinuparella
  sonarqube_url: https://www.sonarsource.com/products/sonarqube/
```

### Example Pod Using ConfigMap (`pod-db.yml`)
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: test-db
spec: 
  containers:
    - name: my-db-container
      image: mysql:latest
      ports:
        - containerPort: 3306
      envFrom:
        - configMapRef:
            name: config-db
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config   # ConfigMap files mounted here
  volumes:
    - name: config-vol
      configMap:
        name: config-db
```

✅ **Notes:**
- Non-sensitive data goes in ConfigMap.  
- Can be used with `envFrom` or `volumeMounts`.  

---

## 2. Secret

### Purpose
- Store **sensitive data** (passwords, usernames, API keys).  
- Data is **base64 encoded** in Kubernetes.  
- Can be consumed in pods in two ways:  
  - `envFrom` → environment variables  
  - `volumeMounts` → mounted as files  

### Example Secret (`secrets.yml`)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-db
data:
  MYSQL_USER: c3JpbnU=         # base64 for 'srinu'
  MYSQL_PASSWORD: c3JpbnVAMTIz # base64 for 'srinu@123'
```

✅ **Notes:**
- Base64 encoding is **not fully secure** — for production use AWS Secrets Manager or Vault.  
- Pods can access Secrets via **environment variables** or **volume mounts**.  

### Example Pod Using ConfigMap + Secret
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-db
spec:
  containers:
    - name: my-db-container
      image: mysql:latest
      ports:
        - containerPort: 3306
      envFrom:
        - configMapRef:
            name: config-db
        - secretRef:
            name: secret-db
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
        - name: secrets-vol
          mountPath: /etc/secrets
  volumes:
    - name: config-vol
      configMap:
        name: config-db
    - name: secrets-vol
      secret:
        secretName: secret-db
```

---

## 3. Volumes: emptyDir

### Purpose
- Creates a **temporary, empty directory** on the pod’s node.  
- Exists **only while the pod is running**; deleted when pod is removed.  
- Commonly used for **temporary storage, caching, or database files**.  

### Syntax
```yaml
volumes:
  - name: vol-db
    emptyDir: {}
```

### Explanation
- `{}` → empty object → use default settings.  
- Can customize with:  
```yaml
emptyDir:
  medium: Memory   # store in RAM instead of disk
  sizeLimit: 1Gi   # optional size limit
```

### Mounting emptyDir in Pod
```yaml
volumeMounts:
  - name: vol-db
    mountPath: /var/lib/mysql  # container path
```

---

## 4. Key Differences Between ConfigMap, Secret, and emptyDir

| Feature       | ConfigMap              | Secret               | emptyDir                          |
|---------------|------------------------|----------------------|-----------------------------------|
| **Data Type** | Non-sensitive          | Sensitive            | Temporary storage                 |
| **Encoding**  | Plain text             | Base64 encoded       | N/A                               |
| **Pod Usage** | envFrom / volumeMounts | envFrom / volumeMounts | volumeMounts                    |
| **Example**   | DB name, URLs          | DB username/password | Temporary DB files, cache         |
| **Persistence** | Survives pod restarts if re-applied | Survives pod restarts if re-applied | Deleted when pod dies |

---

## ✅ Tips for Interview
- **ConfigMap** → Non-sensitive configuration.  
- **Secret** → Sensitive configuration.  
- **emptyDir** → Temporary storage in pod lifecycle.  
- Always explain **two ways of consuming ConfigMap/Secret**: `envFrom` and `volumeMounts`.  
