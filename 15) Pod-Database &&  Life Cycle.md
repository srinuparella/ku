# 📅 03-09-2025

## 1. Pod Database Creation

### Steps:

1.  **Run Docker Container** (later define in Pod manifest file).

    -   **Docker Playground** → Login → Check `docker info` → Docker
        home path `/var/lib/docker`.\
    -   **Docker Hub** → Search `mysql` image → Pull image → Run
        container.

    ``` bash
    docker run -d -p 3306:3306    -e MYSQL_ROOT_PASSWORD=srinu    -e MYSQL_DATABASE=employee    -e MYSQL_USER=SrinuParella    -e MYSQL_PASSWORD=SrinuParella    --name sqlcontainer mysql:latest
    ```

    -   `docker ps` → shows DB running.\
    -   `docker exec -it sqlcontainer /bin/bash` → enter container.

2.  **Define Manifest File: `database.yml`**

    ``` yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: db-pod
    spec:
      containers:
        - name: db-container
          image: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "Srinu@123"
            - name: MYSQL_DATABASE
              value: "employee"
            - name: MYSQL_USER
              value: "srinu"
            - name: MYSQL_PASSWORD
              value: "Srinu@123"
    ```

    > **Note:** Each env variable uses `- name` & `value` because it's
    > an array.

    **When to Use:**\
    In real-time, env variables are mostly passed inside a
    **StatefulSet** (e.g., DB + App connection).

3.  **Login to Pod & MySQL**

    ``` bash
    kubectl exec -it db-pod -- bash
    mysql -u srinu -p
    # Enter password: Srinu@123
    ```

    ``` sql
    USE employee;

    CREATE TABLE employees (
        emp_id INT AUTO_INCREMENT PRIMARY KEY,
        first_name VARCHAR(50),
        last_name VARCHAR(50),
        department VARCHAR(50),
        salary DECIMAL(10,2)
    );

    INSERT INTO employees (first_name, last_name, department, salary)
    VALUES
    ('Ravi', 'Kumar', 'IT', 55000.00),
    ('Anita', 'Sharma', 'HR', 45000.00),
    ('Vikram', 'Patel', 'Finance', 60000.00),
    ('Priya', 'Singh', 'Marketing', 48000.00),
    ('Arjun', 'Reddy', 'IT', 70000.00);

    SELECT * FROM employees;
    ```

------------------------------------------------------------------------

## 2. Pod Life Cycle

A Pod goes through the following phases:

1.  **Pending** → Pod accepted but containers not yet running.\
2.  **Running** → At least one container is running.\
3.  **Succeeded** → All containers terminated successfully.\
4.  **Failed** → One or more containers failed (non‑zero exit).\
5.  **Unknown** → Pod state cannot be determined (e.g., Node not
    reachable).

### Example Manifest: Pending → Running → Succeeded

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-lifecycle
spec:
  containers:
    - name: lifecycleimage
      image: nginx
      command: ["sh", "-c", "exit 0"]
```

-   Apply manifest → Pod starts (Pending → Running).\

-   Command exits with `0` → Pod moves to **Succeeded**.\

-   Delete pod before re‑apply:

    ``` bash
    kubectl delete pod pod-lifecycle
    ```

### Failure Example

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-lifecycle
spec:
  containers:
    - name: lifecycleimage
      image: nginx
      command: ["sh", "-c", "exit 1"]
```

-   Command exits with `1` → Pod goes to **Failed** state.

------------------------------------------------------------------------

## 3. Container States in K8s

Just like Pod lifecycle, **containers** also have states:\
- **Waiting**\
- **Running**\
- **Terminated**

`exit 0` → Success (used in CI/CD pipelines like Jenkins).\
`exit 1` → Failure.
