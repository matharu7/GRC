
## 🚀 Step 1: Prepare the Ubuntu System
Elasticsearch requires specific system settings to run correctly. Open your terminal and run these commands:

### **1. Update System**
```bash
sudo apt update && sudo apt upgrade -y
```

### **2. Install Docker & Docker Compose**
```bash
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

### **3. Configure Virtual Memory (Critical for Elasticsearch)**
Elasticsearch will fail to start if this is not set:
```bash
# Apply immediately
sudo sysctl -w vm.max_map_count=262144

# Make it permanent after reboot
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### **4. Create Jobs Directory**
Since your file uses `/tmp/cortex-jobs`, ensure it has the right permissions:
```bash
sudo mkdir -p /tmp/cortex-jobs
sudo chmod 777 /tmp/cortex-jobs
```

---

## 🛠️ Step 2: Deploy with Docker Compose
1. Create a project folder: `mkdir cortex-lab && cd cortex-lab`
2. Create the file: `nano docker-compose.yml`
3. Paste your working configuration exactly as it is:

```yaml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.9
    container_name: elasticsearch
    environment:
      - http.host=0.0.0.0
      - discovery.type=single-node
      - cluster.name=cortex-cluster
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  cortex:
    image: thehiveproject/cortex:latest
    container_name: cortex
    depends_on:
      - elasticsearch
    ports:
      - "9001:9001"
    environment:
      - JOB_DIRECTORY=/tmp/cortex-jobs
      - DOCKER_OBJFILTER=label=org.thehiveproject.cortex=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/cortex-jobs:/tmp/cortex-jobs
```

4. **Start the containers:**
```bash
sudo docker-compose up -d
```

---

## 🌐 Step 3: Initial Web Configuration
1. Open your browser to `http://<YOUR_SERVER_IP>:9001`.
2. **Database Update:** Click the **"Update Database"** button.
3. **Create Admin:** Set up your primary administrator account (e.g., User: `admin`).
4. **Create Organization:**
   * Go to **Organization** > **Add Organization**.
   * Name it (e.g., `Secpind_SOC`).
   * Add a user to this organization with the **OrgAdmin** and **Analyze** roles.

---

## 🔍 Step 4: Demo - Adding Analyzers
To make Cortex work, you must enable analyzers. Here are the two most common for a demo:

### **Demo 1: VirusTotal (IP/Hash Analysis)**
1. Log in with your **OrgAdmin** user.
2. Go to **Organization** > **Analyzers**.
3. Search for **VirusTotal_GetReport**.
4. Click **Edit Configuration**:
   * **Key:** Paste your VirusTotal API key here.
   * **Enable:** Toggle the switch to **On**.
5. **Test:** Go to **New Analysis**, enter an IP (like `8.8.8.8`), select VirusTotal, and click **Run**.

### **Demo 2: CyberChef (Local Decoding)**
1. In the **Analyzers** list, search for **CyberChef**.
2. Click **Enable** (CyberChef doesn't require an API key as it runs locally).
3. **Test:** You can now use this to decode Base64 or Hex strings directly in your job dashboard.

---

## 📝 Important Maintenance Notes
* **Check Status:** Use `sudo docker ps` to ensure both containers are "Up".
* **View Logs:** If the web page doesn't load, check the logs with `sudo docker logs -f cortex`.
* **Data Persistence:** Your current file saves logs in `/tmp/cortex-jobs`. Note that many Linux systems clear the `/tmp` folder on reboot. For a production lab, change this to a path like `/home/ubuntu/cortex-jobs`.

**Is there anything specific you want to add to your SOC lab next?**
