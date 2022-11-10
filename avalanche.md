Setup avalanche archival node

#### Hardware Requirements 

- CPU: Equivalent of 8 AWS vCPU
- RAM: 16 GB
- Storage > 4 TB
- OS: Ubuntu 20.04 or MacOS >= 12
- Network: sustained 5Mbps up/down bandwidth

#### Assumptions

User running the script is not root and has superuser privileges (can run ```sudo```)
If using a firewall make sure Port 9651 (p2p Port) is open.

#### Instructions:

Get the install-script:
```
wget -nd -m https://raw.githubusercontent.com/ava-labs/avalanche-docs/master/scripts/avalanchego-installer.sh
```
Make the install-script executable:
```
chmod 755 avalanchego-installer.sh
```
Run the install-script (replace replace-with-ip with your servers public IP):
```
./avalanchego-installer.sh --archival --ip replace-with-ip --rpc local
```
The node should be syncing now. To check one can use:
```
sudo journalctl -f -u avalanchego
```
The RPC is locally available at 127.0.0.1:9650.

#### Reverse Proxy for access from another machine

#### Assumptions

You have a domain pointing to the public IP address of your server.
If using a Firewall make sure Port 80 and 443 are open.

#### Instructions

If you want to access the RPC from another machine, use nginx as proxy and LetsEncrypt for ssl certificates

Install nginx:
```
sudo apt install nginx
```

Deactivate standard configuration file:
```
sudo unlink /etc/nginx/sites-enabled/default
```

Create and open a new configuration file:
```
sudo nano /etc/nginx/sites-enabled/proxy.conf
```

Paste the following content into the file (replace your.domain.com with your domain):
```
server {
    listen 80;
    server_name localhost your.domain.com;

    location / {
        proxy_pass http://127.0.0.1:9650;
        proxy_read_timeout 60;
        proxy_connect_timeout 60;
        proxy_buffering off;
        client_max_body_size 100M;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Reload the nginx config: 
```
sudo systemctl reload nginx
```

Install snapd and certbot 
```
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Create a ssl certificate (replace your.domain.com with your domain, replace your@email.com with your e-mail address)
```
sudo certbot --nginx --agree-tos --no-eff-email -m your@email.com -d your.domain.com
```


