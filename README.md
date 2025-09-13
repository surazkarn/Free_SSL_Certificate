# Steps to get Free SSL Certificate (HTTP🔓 to HTTPS 🔒)

This guide provides step-by-step instructions to set up SSL using Certbot on an Ubuntu EC2 instance. It covers installing Certbot, configuring Nginx as a reverse proxy, obtaining an SSL certificate, and enabling HTTPS for your domain. It also includes support for WebSocket connections.

---

## **Prerequisites**
1. **Ubuntu EC2 Instance(or any other distribution)**: Ensure you have an EC2 instance running Ubuntu or any(only some command will change if using other than Ubuntu).
2. **Domain Name**: A registered domain name (e.g., `example.com`) pointing to your EC2 instance's public IP.
3. **Open Ports**: Ensure ports `80` (HTTP) and `443` (HTTPS) are open in your EC2 security group.

---

## **Step 1: Connect to Your EC2 Instance**
1. Use SSH to connect to your EC2 instance:
   ```bash
   ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
   ```
2. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## **Step 2: Install Nginx and Certbot**
1. Install Nginx:
   ```bash
   sudo apt install nginx -y
   ```
2. Install Certbot and the Nginx plugin:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

---

## **Step 3: Configure Nginx as a Reverse Proxy**
1. Create a new Nginx configuration file for your domain:
   ```bash
   sudo nano /etc/nginx/sites-available/example.com
   ```

   **⚠️ Important**: Replace `example.com` with your actual domain name throughout this guide.
2. Add the following configuration (replace `example.com` with your domain and `3000` with your app's port):
   ```nginx
   server {
       listen 80;
       server_name example.com;

       location / {
           proxy_pass http://localhost:3000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }

       # WebSocket support (optional)
       location /ws/ {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "Upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. Enable the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
   ```
   
   **⚠️ Note**: Make sure to replace `example.com` with your actual domain name in the command above.

4. Remove the default Nginx site (optional but recommended):
   ```bash
   sudo rm /etc/nginx/sites-enabled/default
   ```
4. Remove the default Nginx site (optional but recommended):
   ```bash
   sudo rm /etc/nginx/sites-enabled/default
   ```
5. Test the Nginx configuration:
   ```bash
   sudo nginx -t
   ```
   **Expected output**: `nginx: configuration file /etc/nginx/nginx.conf test is successful`

6. If the test fails, check the troubleshooting section below before proceeding.

7. Restart Nginx:
   ```bash
   sudo systemctl restart nginx
   ```
8. Verify Nginx is running:
   ```bash
   sudo systemctl status nginx
   ```

---

## **Step 4: Obtain an SSL Certificate with Certbot**
1. Run Certbot to obtain an SSL certificate:
   ```bash
   sudo certbot --nginx -d example.com
   ```
2. Follow the prompts:
   - Provide an email address for urgent renewal and security notices.
   - Agree to the terms of service.
   - Choose whether to redirect HTTP traffic to HTTPS (recommended: `2`).

Certbot will automatically configure Nginx to use the SSL certificate.

---

## **Step 5: Verify the SSL Configuration**
1. Check the Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/example.com
   ```
   You should see SSL-related directives like:
   ```nginx
   listen 443 ssl;
   ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
   ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
   ```
2. Test the configuration:
   ```bash
   sudo nginx -t
   ```
3. Restart Nginx:
   ```bash
   sudo systemctl restart nginx
   ```

---

## **Step 6: Test HTTPS Access**
1. Open your browser and visit:
   ```
   https://example.com
   ```
2. Verify that the connection is secure (look for the padlock icon in the address bar).

---

## **Step 7: Automate Certificate Renewal**
Certbot automatically sets up a cron job to renew certificates. You can manually test the renewal process:
```bash
sudo certbot renew --dry-run
```

---

## **Step 8: (Optional) WebSocket Support**
If your application uses WebSocket, ensure the Nginx configuration includes the following in the relevant `location` block:
```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "Upgrade";
```

---

---

## **Common Issues and Prevention**

### **Before You Start**
If you encounter SSL certificate errors during installation, this typically means:
1. You have existing nginx configuration files with SSL directives
2. Previous SSL certificate attempts left broken configurations
3. Package installation conflicts due to nginx startup failures

**Quick Fix for Immediate SSL Errors**:
```bash
# Stop nginx if it's failing to start
sudo systemctl stop nginx

# Check for existing SSL configurations
sudo grep -r "ssl_certificate" /etc/nginx/sites-available/ /etc/nginx/sites-enabled/

# If SSL directives are found, back up and temporarily remove them
sudo cp /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-available/your-domain.com.backup
sudo sed -i 's/^[[:space:]]*ssl_/#ssl_/g' /etc/nginx/sites-available/your-domain.com
sudo sed -i 's/^[[:space:]]*listen.*443.*ssl/#listen 443 ssl/g' /etc/nginx/sites-available/your-domain.com

# Test configuration and restart
sudo nginx -t
sudo systemctl start nginx

# Now proceed with certbot to properly configure SSL
```

---

## **Troubleshooting**

### **1. SSL Certificate File Not Found Error**
**Problem**: Nginx fails to start with error:
```
nginx: [emerg] cannot load certificate "/etc/letsencrypt/live/domain.com/fullchain.pem": BIO_new_file() failed
```

**Solution**:
1. **Check if SSL directives exist in nginx config**:
   ```bash
   sudo grep -r "ssl_certificate" /etc/nginx/sites-available/
   ```

2. **If SSL directives are found, temporarily comment them out**:
   ```bash
   sudo nano /etc/nginx/sites-available/your-domain.com
   ```
   Comment out SSL-related lines by adding `#` at the beginning:
   ```nginx
   # listen 443 ssl;
   # ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
   # ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
   ```

3. **Test and restart nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

4. **Run certbot to obtain certificates**:
   ```bash
   sudo certbot --nginx -d your-domain.com
   ```

5. **Certbot will automatically uncomment and configure SSL directives**.

### **2. Nginx Configuration Conflicts**
**Problem**: Multiple nginx configurations or broken symlinks.

**Solution**:
1. **Remove broken symlinks**:
   ```bash
   sudo find /etc/nginx/sites-enabled/ -type l ! -exec test -e {} \; -delete
   ```

2. **Check for duplicate configurations**:
   ```bash
   sudo ls -la /etc/nginx/sites-enabled/
   ```

3. **Remove unwanted configurations**:
   ```bash
   sudo rm /etc/nginx/sites-enabled/example.com  # Replace with actual unwanted file
   ```

4. **Create proper symlink**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/
   ```

### **3. Package Installation Issues**
**Problem**: nginx-core package fails to configure due to SSL errors.

**Solution**:
1. **Stop nginx service first**:
   ```bash
   sudo systemctl stop nginx
   ```

2. **Fix nginx configuration** (follow steps in troubleshooting #1).

3. **Reconfigure packages**:
   ```bash
   sudo dpkg --configure -a
   ```

4. **Start nginx**:
   ```bash
   sudo systemctl start nginx
   ```

### **4. Nginx Fails to Restart**
**Problem**: General nginx startup failures.

**Solution**:
- Check for syntax errors:
  ```bash
  sudo nginx -t
  ```
- Check detailed error logs:
  ```bash
  sudo systemctl status nginx.service
  sudo journalctl -xeu nginx.service
  ```
- Ensure no other service is using ports `80` or `443`:
  ```bash
  sudo netstat -tulpn | grep :80
  sudo netstat -tulpn | grep :443
  ```

### **5. Certbot Fails to Obtain a Certificate**
**Problem**: Certificate generation fails.

**Solution**:
- Ensure your domain's DNS points to the EC2 instance's public IP.
- Ensure ports `80` and `443` are open in your EC2 security group.
- Check if nginx is running:
  ```bash
  sudo systemctl status nginx
  ```
- Verify domain accessibility:
  ```bash
  curl -I http://your-domain.com
  ```

### **6. WebSocket Not Working**
**Problem**: WebSocket connections fail over HTTPS.

**Solution**:
- Verify the WebSocket path in the Nginx configuration matches the client-side path.
- Check server logs for errors:
  ```bash
  sudo tail -f /var/log/nginx/error.log
  ```

---

---

## **Quick Reference Commands**

### **Emergency SSL Fix**
If nginx won't start due to SSL certificate errors:
```bash
# Stop nginx
sudo systemctl stop nginx

# Comment out SSL directives temporarily  
sudo sed -i 's/^[[:space:]]*ssl_/#ssl_/g' /etc/nginx/sites-available/your-domain.com
sudo sed -i 's/^[[:space:]]*listen.*443.*ssl/#listen 443 ssl/g' /etc/nginx/sites-available/your-domain.com

# Start nginx and run certbot
sudo nginx -t && sudo systemctl start nginx
sudo certbot --nginx -d your-domain.com
```

### **Useful Debug Commands**
```bash
# Check nginx status
sudo systemctl status nginx

# Test nginx configuration
sudo nginx -t

# Check SSL certificate expiry
sudo certbot certificates

# Check which process is using port 80/443
sudo netstat -tulpn | grep :80
sudo netstat -tulpn | grep :443

# View nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check domain DNS resolution
nslookup your-domain.com
```

---

## **Conclusion**
You have successfully set up SSL with Certbot on your Ubuntu EC2 instance. Your application is now accessible over HTTPS, and WebSocket connections are supported if configured. Certbot will automatically handle certificate renewals, ensuring your site remains secure.

For further assistance, refer to the [Certbot documentation](https://certbot.eff.org/docs/) or the [Nginx documentation](https://nginx.org/en/docs/).
