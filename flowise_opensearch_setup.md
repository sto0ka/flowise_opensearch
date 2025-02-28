# 🚀 Flowise + OpenSearch Setup Guide (Fixed & Working)

This guide provides a **step-by-step installation** of **Flowise** with **OpenSearch** on Ubuntu using **Docker**, incorporating all necessary fixes.

---

## **1️⃣ Install Required Dependencies**
Before starting, install Docker if it's not already installed.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose curl
```

Ensure Docker is running:
```bash
sudo systemctl enable --now docker
```

---

## **2️⃣ Create a Custom Docker Network**
To ensure Flowise and OpenSearch can communicate, create a dedicated **Docker network**:

```bash
docker network create flowise-net
```

---

## **3️⃣ Run OpenSearch (Fixing SSL/TLS Issues)**
Start OpenSearch **without security** (no HTTPS, no SSL errors):

```bash
docker run -d --name opensearch   --network flowise-net   -p 9200:9200 -p 9600:9600   -e "discovery.type=single-node"   -e "plugins.ml_commons.only_run_on_ml_node=false"   -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=StrongPass!123"   -e "DISABLE_SECURITY_PLUGIN=true"   -e "OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g"   opensearchproject/opensearch:latest
```
✅ **Fixes Applied:**
- `DISABLE_SECURITY_PLUGIN=true` → Disables SSL so OpenSearch works with **HTTP**.
- `-Xms2g -Xmx2g` → Ensures OpenSearch **does not crash** due to low memory.

### **Wait 2-3 minutes for OpenSearch to fully start** before moving to the next step.

Check if OpenSearch is running:
```bash
docker ps
```

Verify OpenSearch responds:
```bash
curl -u 'admin:StrongPass!123' http://localhost:9200
```
✅ **Expected Output (JSON Response)**
```json
{
  "name" : "opensearch-node",
  "cluster_name" : "opensearch",
  "version" : {
    "number" : "2.x.x",
    ...
  }
}
```

---

## **4️⃣ Run Flowise in the Same Network**
Start Flowise:
```bash
docker run -d --name flowise   --network flowise-net   -p 3000:3000   -v ~/.flowise:/root/.flowise   flowiseai/flowise:latest
```

Confirm Flowise is running:
```bash
docker ps
```

---

## **5️⃣ Configure Flowise to Connect to OpenSearch**
1. Open **Flowise UI** at **`http://localhost:3000`**
2. **Go to `Settings > Database`**
3. **Enter OpenSearch Details:**
   - **Host**: `http://opensearch:9200` *(HTTP, not HTTPS)*
   - **Username**: `admin`
   - **Password**: `StrongPass!123`
   - **Index Name**: `flowise_vectors`
   - **Dimension**: `1536`
4. **Save & Restart Flowise.**

---

## **6️⃣ Create the OpenSearch Index for Vector Storage**
Run this command to create the **vector index**:

```bash
curl -X PUT "http://localhost:9200/flowise_vectors"   -H "Content-Type: application/json"   -u 'admin:StrongPass!123'   -d '{
    "settings": { "index": { "knn": true } },
    "mappings": {
      "properties": {
        "vector": { "type": "knn_vector", "dimension": 1536 }
      }
    }
  }'
```

Confirm it exists:
```bash
curl -u 'admin:StrongPass!123' http://localhost:9200/_cat/indices?v
```
✅ **If `flowise_vectors` appears, the index is ready!**

---

## **7️⃣ Test Vector Insert in Flowise**
Try inserting a test vector **manually**:

```bash
curl -X POST "http://localhost:9200/flowise_vectors/_doc/1"   -H "Content-Type: application/json"   -u 'admin:StrongPass!123'   -d '{
    "vector": ['$(seq -s, 0 0.001 1.535)'],
    "text": "This is a test vector"
  }'
```

Retrieve the vector:
```bash
curl -X GET "http://localhost:9200/flowise_vectors/_doc/1"   -H "Content-Type: application/json"   -u 'admin:StrongPass!123'
```
✅ **If the data appears, OpenSearch is working with vectors!**

---

## **8️⃣ Troubleshooting**
### **Issue: Flowise Still Can’t Connect to OpenSearch?**
1. **Check OpenSearch logs for errors:**
   ```bash
   docker logs opensearch | tail -50
   ```
2. **Restart OpenSearch & Flowise:**
   ```bash
   docker restart opensearch flowise
   ```
3. **Test OpenSearch from inside Flowise:**
   ```bash
   docker exec -it flowise sh
   curl -u 'admin:StrongPass!123' http://opensearch:9200
   exit
   ```
   ✅ **If this works, Flowise can reach OpenSearch.**

---

## **✅ Final Recap: What We Fixed**
1. **Fixed OpenSearch Connection Issues** 🚀  
   - Disabled SSL (`DISABLE_SECURITY_PLUGIN=true`)
   - Used `http://opensearch:9200` instead of HTTPS.
   - Ensured enough memory (`-Xms2g -Xmx2g`).

2. **Fixed Docker Network Issues** 🌐  
   - Created `flowise-net` so Flowise & OpenSearch can communicate.

3. **Fixed Indexing & Vector Storage Issues** 📊  
   - Created the `flowise_vectors` index **before** running Flowise.
   - Verified vector insertion manually.

4. **Tested & Verified Connectivity** 🛠  
   - Used `curl` from both **host** and **inside Flowise container**.

---

### **🚀 Success! Now You Can Start Using Flowise & OpenSearch**
Flowise is now connected to OpenSearch and ready for **vector-based AI workflows**.

👉 **Open Flowise at `http://localhost:3000` and start building!** 🔥
