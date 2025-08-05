# **Complete NGINX Reverse Proxy Setup for Go Gin Server**

This guide will walk you through the process of installing and configuring **NGINX** on **macOS** to act as a **reverse proxy** for your **Go Gin server** running on port **9090**. The guide includes two configuration options:

1. **Without SSL** (HTTP only)
2. **With SSL** (HTTPS via Let’s Encrypt)

---

## **Prerequisites:**

* **NGINX** installed on **macOS** via **Homebrew**.
* **Go Gin server** running on port `9090`.
* A **domain name** (e.g., `betzones.com`) pointing to your server’s IP address.
* **sudo** privileges to install packages and modify system files.

---

## **Step 1: Install Homebrew (if not installed)**

**Homebrew** is a package manager for macOS that simplifies the installation of software like **NGINX**.

1. Open your terminal and run the following command to install **Homebrew**:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. After the installation finishes, verify it by running:

   ```bash
   brew --version
   ```

   This confirms that **Homebrew** is installed.

---

## **Step 2: Install NGINX with Homebrew**

Once **Homebrew** is installed, you can easily install **NGINX**:

1. Run the following command to install **NGINX**:

   ```bash
   brew install nginx
   ```

2. Wait for **NGINX** to be installed.

---

## **Step 3: Start NGINX**

Once **NGINX** is installed, you can start it using:

```bash
sudo nginx
```

This will start **NGINX** with the default configuration file located at `/usr/local/etc/nginx/nginx.conf`.

---

## **Step 4: Verify NGINX Installation**

To verify that **NGINX** is running:

1. Open your browser and visit:

   ```
   http://localhost:8080
   ```

2. You should see the **NGINX welcome page**, confirming that **NGINX** is running correctly.

---

## **Step 5: Manage NGINX with Homebrew Services (Optional)**

You can manage **NGINX** using **Homebrew services**:

1. To start **NGINX** as a background service:

   ```bash
   brew services start nginx
   ```

2. To stop **NGINX**:

   ```bash
   brew services stop nginx
   ```

3. To restart **NGINX**:

   ```bash
   brew services restart nginx
   ```

---

## **Step 6: Configuring NGINX as a Reverse Proxy for Go Gin Server**

Now, we will configure **NGINX** to forward requests from `betzones.com` to your **Go Gin server** running on port `9090`.

### **Edit the NGINX Configuration File**

1. Open the **NGINX configuration file** in a text editor:

   ```bash
   sudo nano /usr/local/etc/nginx/nginx.conf
   ```

2. Inside the `nginx.conf` file, you will find a section called `http { ... }`. Depending on whether you want **HTTP only** or **HTTPS**, you can use one of the following configurations.

---

## **Option 1: Without SSL (HTTP Only)**

If you only want **HTTP** without SSL, use this configuration to reverse proxy requests from `betzones.com` to your Go server.

### **NGINX Configuration Without SSL (HTTP Only)**

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # Reverse Proxy Configuration for betzones.com (HTTP only)
    server {
        listen 80;  # Listen on HTTP (port 80)
        server_name betzones.com;  # Your domain name

        location / {
            proxy_pass http://localhost:9090;  # Forward requests to Go server on port 9090
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    include servers/*;
}
```

---

## **Option 2: With SSL (HTTPS)**

To serve your site over **HTTPS**, we will use **Let’s Encrypt** and **Certbot** to automatically obtain an SSL certificate. Below is the configuration for **HTTP to HTTPS redirection** and **SSL/TLS**.

### **NGINX Configuration with SSL (HTTPS)**

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # Reverse Proxy Configuration for betzones.com (HTTP to HTTPS Redirect)
    server {
        listen 80;
        server_name betzones.com;  # Your domain name

        # Redirect HTTP to HTTPS
        return 301 https://$host$request_uri;  # Redirect HTTP to HTTPS
    }

    # Serve HTTPS with SSL/TLS
    server {
        listen 443 ssl;
        server_name betzones.com;

        # SSL configuration (replace with actual paths to your SSL certificates)
        ssl_certificate /path/to/fullchain.pem;  # Path to your SSL certificate
        ssl_certificate_key /path/to/privkey.pem;  # Path to your private key

        # Forward requests to your Go server running on port 9090
        location / {
            proxy_pass http://localhost:9090;  # Forward requests to your Go server
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        # Optional: Error page configuration
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    include servers/*;
}
```

### **Explanation**:

1. **HTTP to HTTPS Redirection (Port 80)**:

   * **`listen 80;`**: NGINX listens for **HTTP traffic** on port 80.
   * **`return 301 https://$host$request_uri;`**: Redirects all HTTP requests to HTTPS, ensuring secure connections.

2. **Serve HTTPS with SSL (Port 443)**:

   * **`listen 443 ssl;`**: NGINX listens for **HTTPS traffic** on port 443.
   * **`ssl_certificate /path/to/fullchain.pem;`** and **`ssl_certificate_key /path/to/privkey.pem;`**: These lines specify the **SSL certificate** and **private key**.
   * **`proxy_pass http://localhost:9090;`**: Forwards requests from `https://betzones.com` to your **Go server** on `localhost:9090`.
   * **`proxy_set_header` lines**: These headers ensure proper connection handling for WebSockets and other protocols.

---

## **Step 7: Set Up SSL (HTTPS) with Let’s Encrypt**

To set up **SSL/TLS** for your domain, we will use **Let’s Encrypt** and **Certbot**.

### **Install Certbot (Let’s Encrypt)**

1. Install **Certbot** via **Homebrew**:

   ```bash
   brew install certbot
   ```

### **Obtain SSL Certificate with Certbot**

1. Run the following command to automatically obtain and install the SSL certificate for `betzones.com`:

   ```bash
   sudo certbot --nginx -d betzones.com -d www.betzones.com
   ```

   **Certbot** will:

   * Automatically obtain the SSL certificate.
   * Configure NGINX to use the certificate.
   * Reload NGINX to apply the changes.

2. **Verify SSL Configuration**

After obtaining the SSL certificate, you can verify the SSL setup by visiting `https://betzones.com` in your browser. The **SSL padlock icon** will appear, indicating a secure connection.

---

## **Step 8: Test and Reload NGINX**

1. **Test the NGINX configuration** for syntax errors:

   ```bash
   sudo nginx -t
   ```

2. If the test is successful, **reload NGINX** to apply the changes:

   ```bash
   sudo nginx -s reload
   ```

---

## **Step 9: Update DNS Settings (If Necessary)**

Ensure that your **domain `betzones.com`** points to your server's **IP address**. If you're testing locally, you can modify your **hosts file**:

1. **Edit the hosts file**:

   ```bash
   sudo nano /etc/hosts
   ```

2. Add the following line to map `betzones.com` to `127.0.0.1` (your local machine):

   ```
   127.0.0.1   betzones.com
   ```

---

## **Step 10: Verify the Setup**

1. **Verify SSL**:

   * Open your browser and go to `https://betzones.com`.
   * You should see your **Go Gin server’s** response, and the **SSL padlock icon** should indicate that the connection is secure.

2. **Verify Redirection**:

   * Visit `http://betzones.com` to ensure that it redirects to `https://betzones.com`.

---

## **Step 11: Uninstall NGINX (Optional)**

If you ever need to uninstall **NGINX**, run the following command:

```bash
brew uninstall nginx
```

---

### **Conclusion**

By following this guide, you have successfully:

1. Installed **NGINX** on **macOS** using **Homebrew**.
2. Configured **NGINX** as a **reverse proxy** for your **Go Gin server** running on port `9090`.
3. Set up **SSL/TLS** using **Let’s Encrypt** and **Certbot** to serve your website securely over **HTTPS** (optional).
4. Optionally, set up **HTTP to HTTPS redirection** for secure connections.

Your **Go Gin server** is now accessible securely via **NGINX** at `https://betzones.com`.

---

### **Additional Notes**

* **SSL Certificate Renewal**: Certbot will automatically renew your SSL certificate, but you can manually renew it by running:

  ```bash
  sudo certbot renew
  ```
* **Firewall Settings**: Ensure **ports 80** and **443** are open in your firewall to allow HTTP and HTTPS traffic.
* **Auto-Renewal**: Certbot automatically handles SSL certificate renewals, but you can set up a cron job for periodic checks.

Let me know if you need further assistance!
