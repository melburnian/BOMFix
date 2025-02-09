# BOMFix - Local HTTPS Proxy with Nginx on macOS

This guide explains how to set up a local HTTPS proxy using Nginx on macOS to directly access content hosted on [http://www.bom.gov.au/](http://www.bom.gov.au/), allowing users to directly access BOM content without the following message:

![BomRedirect](https://github.com/user-attachments/assets/5c3fe83c-9d46-4b70-9820-93eeb37f3763)

The proxy will be added to a device, not a network.  

**Note** This is a basic workaround intended for use when seeking to access BOM content. Other website will not work while the solution is active. To resume normal browsing, use the command  ```brew services stop nginx ``` and use ```brew services restart nginx``` to reactivate the workaround.

## Prerequisites

- [Homebrew](https://brew.sh/) installed.
  
  ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
- Basic familiarity with the command line (Terminal).

## Steps

### 1. Install Nginx

Open Terminal and run:

```bash
brew update
```
```bash
brew install nginx
```

### 2. Create a Certificates Directory

```bash
mkdir ~/certs
```
```bash
cd ~/certs
```
### 3. Generate a Self-Signed Certificate

```bash
openssl genpkey -algorithm RSA -out localhost.key
```
```bash
openssl req -x509 -new -nodes -key localhost.key -sha256 -days 365 -out localhost.crt -subj "/CN=localhost"
```

### 4. Configure Nginx

1. Locate nginx.conf (usually at /opt/homebrew/etc/nginx/nginx.conf)
   
2. Open it with a text editor:

```bash
open /opt/homebrew/etc/nginx/nginx.conf
```

3. Replace the contents with:
  
```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /Users/<your_username>/certs/localhost.crt; # Replace with your path
        ssl_certificate_key /Users/<your_username>/certs/localhost.key; # Replace with your path

        location / {
            proxy_pass http://www.bom.gov.au/; # Replace with your target HTTP website
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```
### 5. Start Nginx

```bash
brew services start nginx
```

If Nginx is already running, restart it:

```bash
brew services restart nginx
```

### 6. Configure Your Browser (Chrome Example)

Visit [https://localhost](https://localhost).
Accept the self-signed certificate warning by clicking “Advanced” > “Proceed to localhost (unsafe)”.

### 7. Test

After accepting the certificate warning, you should see the proxied HTTP website when visiting [https://localhost](https://localhost).

### 8. (Optional) System-Wide Proxy (macOS - Advanced)

1. Go to System Preferences > Network > Wi-Fi (or Ethernet) > Advanced > Proxies.
2. Check “Web Proxy (HTTPS)”.
3. Set:
4. Server: localhost
5. Port: 443
6. Click OK and Apply.
7. Stop Nginx (When Finished)

```bash
brew services stop nginx
```

![BOMFix](https://github.com/user-attachments/assets/6179e126-66b1-416e-884f-cd209be6fdc2)

## Troubleshooting

- Port Conflicts: Use lsof -i :443 to check for processes using port 443.
- Configuration Errors: Check the Nginx error log at /opt/homebrew/var/log/nginx/error.log.
- Certificate Issues: Verify the paths to your certificate and key files in nginx.conf


