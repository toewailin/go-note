Deploying a Go (Golang) project on an Ubuntu server requires several steps, including setting up the server environment, building the Go application, and configuring the server for production use. Here’s a step-by-step guide:

### 1. **Prepare Your Ubuntu Server**

Make sure your Ubuntu server is up-to-date and has the necessary dependencies installed.

#### Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

#### Install Go (Golang)

If Go is not installed yet, you can install it as follows:

```bash
sudo apt install golang-go -y
```

Alternatively, you can download the latest Go release from [Go's official website](https://golang.org/dl/) and follow the instructions for installing the binary.

### 2. **Clone Your Project**

If you have your project in a Git repository, clone it to your server:

```bash
cd /home/username
git clone https://github.com/yourusername/your-golang-project.git
cd your-golang-project
```

### 3. **Install Dependencies**

If your project uses third-party dependencies, you can install them using Go Modules:

```bash
go mod tidy
```

This will download and install any missing dependencies listed in your `go.mod` file.

### 4. **Build Your Go Application**

Once the dependencies are installed, build your Go project.

```bash
go build -o yourapp .
```

This will create an executable file named `yourapp` in the current directory.

### 5. **Configure the Application for Production**

You need to set environment variables, configure ports, databases, or other services depending on your application’s needs.

#### Example:

Create a `.env` file or export environment variables directly on the server:

```bash
export APP_ENV=production
export DB_HOST=localhost
export DB_PORT=5432
export DB_USER=myuser
export DB_PASSWORD=mypassword
```

### 6. **Run the Application**

You can run the Go application directly:

```bash
./yourapp
```

If your application runs on a specific port, for example port `8080`, make sure to open the port in your firewall:

```bash
sudo ufw allow 8080
```

Alternatively, for long-running production services, you can run the application in the background.

```bash
nohup ./yourapp &
```

### 7. **Set Up the Application as a Service (Optional but Recommended)**

To make your application run as a service and automatically start on boot, you can create a `systemd` service.

#### Create a `systemd` Service File

Create a new service file in `/etc/systemd/system/yourapp.service`:

```bash
sudo nano /etc/systemd/system/yourapp.service
```

Add the following configuration:

```ini
[Unit]
Description=Go Application
After=network.target

[Service]
ExecStart=/home/username/your-golang-project/yourapp
WorkingDirectory=/home/username/your-golang-project
Restart=always
User=username
Group=username
Environment=APP_ENV=production

[Install]
WantedBy=multi-user.target
```

#### Reload systemd and Enable the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable yourapp.service
```

#### Start the Service

```bash
sudo systemctl start yourapp.service
```

You can check the status of the service:

```bash
sudo systemctl status yourapp.service
```

### 8. **Set Up Reverse Proxy (Optional)**

If you want to expose your Go application to the web via Nginx, you can set up a reverse proxy.

#### Install Nginx

```bash
sudo apt install nginx -y
```

#### Configure Nginx to Proxy Requests

Create a configuration file for your site:

```bash
sudo nano /etc/nginx/sites-available/yourapp
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;  # Change to your application's port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Create a symlink to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/yourapp /etc/nginx/sites-enabled/
```

#### Restart Nginx

```bash
sudo systemctl restart nginx
```

Now, Nginx will forward traffic to your Go application.

### 9. **Monitor and Manage Logs**

You can use tools like `journalctl` to check logs for your service:

```bash
sudo journalctl -u yourapp.service
```

Alternatively, you can set up log files or use a service like `logrotate` to manage logs.

---

### Recap

1. Update Ubuntu and install Go.
2. Clone your project and install dependencies.
3. Build the Go application.
4. Set environment variables for production.
5. Run the application or set it up as a service.
6. Optionally, set up Nginx as a reverse proxy.

Let me know if you need more detailed guidance on any of these steps!
