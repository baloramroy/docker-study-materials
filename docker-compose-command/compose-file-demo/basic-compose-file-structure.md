# **Basic Docker Compose File Structure (`docker-compose.yml`)**

```yaml
version: "3.9"   # optional for modern Compose, but good for clarity

services:
  service_name:
    image: nginx:latest        # Image to use
    container_name: my_nginx   # (optional) Name of the container
    ports:
      - "8080:80"              # host:container
    environment:
      - ENV_VAR=value          # environment variables
    volumes:
      - mydata:/usr/share/nginx/html   # attach volume
      - nginx_log:/var/log/nginx       # attach bind mount volume
    networks:
      - mynetwork              # network name
    restart: unless-stopped    # auto-restart policy

volumes:
  mydata:                      # named volume

networks:
  mynetwork:                   # custom network
    driver: bridge
```

## **Section-Wise Explanation**

### **1. version**

- Defines Compose file format.
- Modern Docker Compose **does not require** this, but adding it improves clarity.

### **2. services**

- Where you define each container.

  **Common keys inside a service:**
  
  * `image:` → Which Docker image to use
  * `container_name:` → Name of container (optional)
  * `build:` → Build from Dockerfile
  * `ports:` → Port mapping
  * `environment:` → Environment variables
  * `volumes:` → Mount volumes
  * `networks:` → Attach networks
  * `restart:` → Restart policy (`no`, `always`, `on-failure`, `unless-stopped`)

### **3. volumes**

- Used to define **named volumes**.
- If the **named volume** is created first using `docker volume create` command then use

  ```yml
  volumes:
  mydata:                      # named volume
    external: true
  ```
- No need to define this part for **bind mount volumes**

### **4. networks**

- Used to define **custom networks** such as `bridge`, `overlay`, or `host`.
- But if the network is created first using `docker network creat` command then use **external: true**.

  ```yml
  networks:
    mynetwork:
      external: true
  ``` 

---
