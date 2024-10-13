To install your Go-based IPTV provider management system on a VPS, you'll follow these general steps:

### Steps for Setting Up on a VPS

1. **Choose a VPS Provider**
   - You can use services like DigitalOcean, AWS, Google Cloud, Linode, or others.
   - Select an OS (usually Linux-based, such as Ubuntu, Debian, or CentOS).

2. **Access Your VPS**
   Once your VPS is set up, you'll get access credentials (usually via SSH).

   - **SSH into Your VPS**:
     ```bash
     ssh root@your-vps-ip
     ```

3. **Update the System**
   Before installing anything, update the package lists and install updates:

   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

4. **Install Go**
   You'll need to install Go on the VPS. Here's how to do it:

   - Download and install the latest Go version:
     ```bash
     wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz
     sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
     ```

   - Set up the Go environment by adding Go to your system's path:
     ```bash
     echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
     source ~/.bashrc
     ```

   - Confirm the installation by checking the Go version:
     ```bash
     go version
     ```

5. **Install PostgreSQL (or another DB)**
   For customer and subscription management, you'll need a database like PostgreSQL:

   - Install PostgreSQL:
     ```bash
     sudo apt install postgresql postgresql-contrib -y
     ```

   - Set up the PostgreSQL database:
     ```bash
     sudo -i -u postgres
     psql
     CREATE DATABASE iptvdb;
     CREATE USER iptvuser WITH ENCRYPTED PASSWORD 'your_password';
     GRANT ALL PRIVILEGES ON DATABASE iptvdb TO iptvuser;
     \q
     exit
     ```

6. **Clone or Copy Your Go Project to the VPS**

   - You can use Git to clone your project if it's hosted on a service like GitHub, or you can SCP (secure copy) the files to the VPS:
   
   **Clone via Git:**
   ```bash
   git clone https://github.com/yourusername/yourrepository.git
   cd yourrepository
   ```

   **Or, use SCP to transfer files from your local machine:**
   ```bash
   scp -r /path/to/your/project root@your-vps-ip:/path/to/destination
   ```

7. **Configure Your Go App**

   - Update the database connection in your Go code (in your `main.go` or configuration file):
     ```go
     db, err = sql.Open("postgres", "user=iptvuser dbname=iptvdb password=your_password sslmode=disable")
     ```

   - Install any dependencies your Go app needs:
     ```bash
     go mod tidy
     ```

8. **Run the Application**

   You can run your Go application manually to check if everything works:

   ```bash
   go run main.go
   ```

   However, for production deployment, you should compile the Go app and set it up to run in the background.

9. **Compile and Run as a Background Service**
   
   - Compile the Go binary:
     ```bash
     go build -o iptvapp main.go
     ```

   - Run the binary in the background:
     ```bash
     ./iptvapp &
     ```

   Alternatively, you can use `systemd` to run the app as a service:

   - Create a service file for your app:
     ```bash
     sudo nano /etc/systemd/system/iptvapp.service
     ```

   - Add the following configuration to the file:
     ```ini
     [Unit]
     Description=IPTV Provider Management App
     After=network.target

     [Service]
     ExecStart=/path/to/your/binary/iptvapp
     WorkingDirectory=/path/to/your/binary
     Restart=always
     User=root
     Environment=PATH=/usr/local/go/bin:/usr/bin
     Environment=GOPATH=/root/go

     [Install]
     WantedBy=multi-user.target
     ```

   - Enable and start the service:
     ```bash
     sudo systemctl enable iptvapp
     sudo systemctl start iptvapp
     ```

   - Check the status of the service:
     ```bash
     sudo systemctl status iptvapp
     ```

10. **Install Nginx for Reverse Proxy**

    You might want to expose your app via HTTP or HTTPS. Using **Nginx** as a reverse proxy is a good way to handle requests.

    - Install Nginx:
      ```bash
      sudo apt install nginx -y
      ```

    - Configure Nginx for your app:
      ```bash
      sudo nano /etc/nginx/sites-available/iptvapp
      ```

    - Add the following configuration to proxy traffic to your Go app (which runs on port 8080):
      ```nginx
      server {
          listen 80;
          server_name your-domain.com;

          location / {
              proxy_pass http://localhost:8080;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
          }
      }
      ```

    - Enable the site and restart Nginx:
      ```bash
      sudo ln -s /etc/nginx/sites-available/iptvapp /etc/nginx/sites-enabled/
      sudo systemctl restart nginx
      ```

11. **Setup SSL (Optional)**

    You should secure your app with HTTPS by using a tool like **Let's Encrypt** to obtain free SSL certificates.

    - Install Certbot:
      ```bash
      sudo apt install certbot python3-certbot-nginx -y
      ```

    - Run Certbot for SSL setup:
      ```bash
      sudo certbot --nginx -d your-domain.com
      ```

12. **Set Up a Domain Name (Optional)**
   
    If you have a domain name, you can point it to your VPS by updating the DNS settings with your VPS’s public IP address.

### Summary of Commands

Here’s a quick overview of key commands you’ll use:

1. **SSH into VPS**:
   ```bash
   ssh root@your-vps-ip
   ```

2. **Update system**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install Go**:
   ```bash
   wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz
   sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
   echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
   source ~/.bashrc
   ```

4. **Install PostgreSQL**:
   ```bash
   sudo apt install postgresql postgresql-contrib -y
   ```

5. **Clone your app from Git**:
   ```bash
   git clone https://github.com/yourusername/yourrepository.git
   cd yourrepository
   ```

6. **Build your app**:
   ```bash
   go build -o iptvapp main.go
   ```

7. **Run your app**:
   ```bash
   ./iptvapp &
   ```

8. **Configure Nginx for Reverse Proxy**:
   ```bash
   sudo apt install nginx -y
   sudo nano /etc/nginx/sites-available/iptvapp
   sudo ln -s /etc/nginx/sites-available/iptvapp /etc/nginx/sites-enabled/
   sudo systemctl restart nginx
   ```

By following these steps, your Go-based IPTV management system will be up and running on your VPS, and accessible through a web interface.
