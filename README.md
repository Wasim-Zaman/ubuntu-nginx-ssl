# Enabling SSL Certification on Ubuntu VPS for Node.js Applications

This guide provides step-by-step instructions to enable SSL certification for your Node.js applications running on an Ubuntu VPS. We will use Let's Encrypt for the SSL certificate and Nginx as the reverse proxy.

## Prerequisites
- Ubuntu 22.04 VPS
- Node.js application
- Nginx installed
- Domain name pointed to your VPS

## Step 1: Install Certbot
Certbot is a tool to obtain SSL certificates from Let's Encrypt. Install it by running:

```ssh
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

## Step 2: Configure Nginx
Ensure your Nginx configuration is set up to serve your Node.js application. Here is an example configuration:

```ssh
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;

    location / {
        root /root/project_frontend/dist;
        index index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
    }
}

server {
    listen 80;
    server_name backend.your_domain.com;

    location / {
        proxy_pass http://localhost:1323;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save this configuration file in /etc/nginx/sites-available/default and create a symlink in /etc/nginx/sites-enabled/:

```ssh
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

Test the Nginx configuration:
```ssh
sudo nginx -t
```

Reload Nginx to apply changes:
```ssh
sudo systemctl reload nginx
```

## Step 3: Obtain SSL Certificate
Run Certbot to obtain and install the SSL certificate:
```ssh
sudo certbot --nginx -d leadgenadvertisements.com -d www.leadgenadvertisements.com -d backend.leadgenadvertisements.com
```

Follow the prompts to complete the domain verification process. Certbot will automatically update your Nginx configuration to use SSL.

## Step 4: Verify SSL Configuration
Certbot should update your Nginx configuration to include SSL settings. The updated configuration should look like this:
```ssh
server {
    listen 80;
    server_name your_domain www.your_domain;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your_domain www.your_domain;

    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        root /root/car_project_frontend/dist;
        index index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
    }
}

server {
    listen 80;
    server_name backend.your_domain;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name backend.your_domain;

    ssl_certificate /etc/letsencrypt/live/backend.your_domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/backend.your_domain/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:1323;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Step 5: Test SSL Configuration
Visit your website using https:// to ensure the SSL certificate is working correctly. You should see a secure connection indicated by the padlock icon in the browser's address bar.

## Step 6: Auto-Renewal Configuration
Let's Encrypt certificates are valid for 90 days. Certbot automatically sets up a cron job for renewal. To verify, you can check the cron job:
```ssh
sudo systemctl status certbot.timer
```

You can also test the renewal process manually:
```ssh
sudo certbot renew --dry-run
```

# Conclusion
By following these steps, you have successfully enabled SSL certification for your Node.js application running on an Ubuntu VPS using Nginx and Let's Encrypt. Your application is now secure and ready to handle HTTPS traffic.
