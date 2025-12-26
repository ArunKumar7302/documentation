# Docker Network Isolation Demo (Tamil)

 **Docker Networking** video on the **Simply Byte** YouTube channel.

In this demo, we clearly show:

* How containers communicate in the **default bridge network**
* How **network isolation** works using **custom Docker networks**
* Why containers in **different networks cannot talk to each other**

---

## 📌 Prerequisites

* Docker installed
* Basic Docker knowledge
---

## 🐳 Docker Images Used

The following Docker images are used in this demo:

### 1️⃣ MySQL Database Image

**Image:**
`mysql:8`

- Official MySQL image from Docker Hub
- Used to demonstrate database container
- Environment variables used:
    - `MYSQL_ROOT_PASSWORD`
    - `MYSQL_DATABASE`

---

### 2️⃣ Backend Application Image

**Image:**
`simplybyte/simplybyte-backend:1.0`

- Custom Spring Boot backend image
- Exposes port **8090**
- Connects to MySQL using `SPRING_DATASOURCE_URL`
---

## 🔹 Scenario 1: Default Bridge Network

First, we run both containers in the **default bridge network**.

### Run MySQL Container

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=simplybyte \
  -e MYSQL_DATABASE=simplybyte \
  mysql:8
```

### Run Backend Container

```bash
docker run -d -p 8090:8090 \
  --name simplybyte_backend \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://172.18.0.2:3306/simplybyte \
  simplybyte/simplybyte-backend:1.0
```

✅ Both containers are in the **default bridge network**, so they can communicate using IP address.

---

## 🔹 Scenario 2: Create Custom Docker Networks

Now we create **two separate Docker networks** to demonstrate isolation.

```bash
docker network create A
docker network create B
```

Check available networks:

```bash
docker network ls
```

---

## 🔹 Scenario 3: Containers in Different Networks (Isolation Demo)

### Run MySQL in Network A

```bash
docker run -d \
  --name mysql-db \
  --network A \
  -e MYSQL_ROOT_PASSWORD=simplybyte \
  -e MYSQL_DATABASE=simplybyte \
  mysql:8
```

### Run Backend in Network B

```bash
docker run -d -p 8090:8090 \
  --name simplybyte_backend \
  --network B \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://172.18.0.2:3306/simplybyte \
  simplybyte/simplybyte-backend:1.0
```

❌ Backend **cannot connect** to MySQL because:

* MySQL is in **Network A**
* Backend is in **Network B**
* Docker networks are **isolated by default**

👉 This proves **Docker network isolation**.

---

## 🔹 Scenario 4: Both Containers in Same Network

Now we connect **both containers to Network A**.

```bash
docker run -d \
  --name mysql-db \
  --network A \
  -e MYSQL_ROOT_PASSWORD=simplybyte \
  -e MYSQL_DATABASE=simplybyte \
  mysql:8
```

```bash
docker run -d -p 8090:8090 \
  --name simplybyte_backend \
  --network A \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://172.18.0.2:3306/simplybyte \
  simplybyte/simplybyte-backend:1.0
```

✅ Now the backend can successfully communicate with MySQL.

---

## 🔍 Useful Docker Commands Used

### Inspect Docker Network

```bash
docker network inspect A
docker network inspect B
```

### Inspect Container Details

```bash
docker inspect mysql-db
docker inspect simplybyte_backend
```

### View Container Logs

```bash
docker logs mysql-db
docker logs simplybyte_backend
```

### Follow Logs in Real-Time

```bash
docker logs -f simplybyte_backend
```

---

## 🎯 Key Takeaways

* Containers in **same network** can communicate
* Containers in **different networks are isolated**
* Docker bridge networks provide **security and separation**
* Network isolation is critical in **real-world microservices**

---

## 📺 YouTube Channel

👉 **Simply Byte** – DevOps & Cloud Explained in Tamil

If you found this helpful, please **Like, Share & Subscribe** ❤️

Happy Learning 🚀
