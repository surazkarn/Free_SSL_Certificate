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
4. Test the Nginx configuration:
   ```bash
   sudo nginx -t
   ```
5. Restart Nginx:
   ```bash
   sudo systemctl restart nginx
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

## **Troubleshooting**
1. **Nginx Fails to Restart**:
   - Check for syntax errors:
     ```bash
     sudo nginx -t
     ```
   - Ensure no other service is using ports `80` or `443`.

2. **Certbot Fails to Obtain a Certificate**:
   - Ensure your domain's DNS points to the EC2 instance's public IP.
   - Ensure ports `80` and `443` are open in the EC2 security group.

3. **WebSocket Not Working**:
   - Verify the WebSocket path in the Nginx configuration matches the client-side path.
   - Check server logs for errors.

---

## **Conclusion**
You have successfully set up SSL with Certbot on your Ubuntu EC2 instance. Your application is now accessible over HTTPS, and WebSocket connections are supported if configured. Certbot will automatically handle certificate renewals, ensuring your site remains secure.

For further assistance, refer to the [Certbot documentation](https://certbot.eff.org/docs/) or the [Nginx documentation](https://nginx.org/en/docs/).
