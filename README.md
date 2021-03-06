# HTTP and Reverse Proxy
Technical guide to protecting Kibana using nginx with HTTPS as a reverse proxy.

## Summary
One of the most well known set of policies that encrypts and authenticates data sources is called PKI (Public Key Infrastructure). 
This service is widely used to protect confidentiality and integrity of web services. In addition to it, HTTPS is also implemented to ensure a secure experience and protect web services. In this lab, we learn how to protect our unauthenticated web services, how to use a reverse proxy to protect this exposed web service, and how to issue a certificate to a webserver--enabling it on HTTPS.

## Methodology
We assume lab 3 is working appropriately, and that you are using Debian 9 or other Linux distro.
### * *Install and setup Nginx and a domain* *

**1. First, setup a domain:**
- If you already have a domain, you may add a new Zone Record with a subdomain. In this case, we are using `it366.gabeit.net` (see Appendix 1).

**2. Setup nginx:**
- `sudo apt-get update`
- `sudo apt install nginx`
- `sudo systemctl enable nginx` (Enable at boot)

**3. Setup a password for your Kibana instance hosted in your Nginx server:**
- ``echo "ENTER_YOUR_ADMIN_USERNAME_HERE:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users``
- You will input your secure password twice, and select the next options.

**4. Create an Nginx server block file:**
- `sudo nano /etc/nginx/sites-available/YOURSUBDOMAIN.DOMAIN.COM`
- Add the following block of code to your file (notice the part you have to edit and add your domain)
```
server {
    listen 80;

    server_name YOURSUBDOMAIN.DOMAIN.COM;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**5. Create a sybmolic link to the sites-enabled directory in your Nginx configuration.**
- `sudo ln -s /etc/nginx/sites-available/YOURSUBDOMAIN.DOMAIN.COM /etc/nginx/sites-enabledYOURSUBDOMAIN.DOMAIN.COM`
- Restart Nginx
   - `sudo systemctl restart nginx`

**6. Allow connections to Nginx from your ufw firewall and disable the HTTP allowance:**
- `sudo ufw allow 'Nginx Full'`
- `sudo ufw delete allow 'Nginx HTTP'`


### * *Use `Let's encrypt` to obtain SSL Certificate and install Certbot* *

**1. Add the backports repositories to your sources list to be updated when your server is updated and install Certbot:**
- `sudo nano /etc/apt/sources.list`
- Add the following two linest to the bottom of the file:
   - `deb http://deb.debian.org/debian stretch-backports main contrib non-free`
   - `deb-src http://deb.debian.org/debian stretch-backports main contrib non-free`
- Run:
   - `sudo apt update`
- Install Certbot Nginx package
   - `sudo apt install python-certbot-nginx -t stretch-backports`

**2. Test and reload/restart the Nginx configuration**
- `sudo nginx -t`
   - You should see a message saying: `nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`
- `sudo systemctl reload nginx`

**3. Obtain SSL Certificate**
- Run:
   - `sudo certbot --nginx -d YOURSUBDOMAIN.DOMAIN.COM`
- Enter your email when prompted.
   - Select `A` to agree with terms.
   - Select option `2` when prompted to ensure you install the most complete and secure SSL certificate.
- You can run the link provided in the terminal of your web service. It will test your SSL certificate (see Appendix 2). 

**4. Setup Certbot Auto-Renewal**
Test the renewal process. If you see no errors, all is well!
- `sudo certbot renew --dry-run`

**5. Google Cloud Platform Final Config**
- In your Firewall rules, make sure to remove the `tcp:5601` rule.

If you want to better understand how all of these programs work together, see Appendix 5. The diagram shows exactly how each part interacts with the client-side of things.

Lastly, now, when you type your website address, you should be redirected to use HTTPS with a trusted certificate, and be prompted to enter a username and password for the Kibana admin config (see Appendix 3).

## Troubleshooting
1. Indentation is really important. Check all your config files and make sure they are all indented as instructed in lab 3.
2. If your HTTPS configuration shows as "unsecure" when you browse to your site, consider closing all your tabs and reopen your site. You might need to use an incognito tab.
3. Consider restarting nginx and kibana if your configuration is not working after all the previous steps.
4. Sometimes if you try to configure your site using **www** it does not work. That's okay. If it works with your subdomain, you are all set!
5. As in the previous lab, your Kibana configuration must not have `0.0.0.0` as the host anymore. You must change it to `localhost`.

## Sources
- How to install ELK Stack + Filebeats. This tutorial also helps you install Nginx.
https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-ubuntu-16-04

- How to Secure Nginx with Let's Encrypt
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-debian-9

## Appendix
**1.** <br><img src="https://github.com/gabecoelho/https-ssl-certificate/blob/master/images/Domains%20dashboard.png"/><br>
**2.** <br><img src="https://github.com/gabecoelho/https-ssl-certificate/blob/master/images/SSL%20Certificate.png"/><br>
**3.** <br><img src="https://github.com/gabecoelho/https-ssl-certificate/blob/master/images/login.png"/><br>
**4.** <br><img src="https://github.com/gabecoelho/https-ssl-certificate/blob/master/images/HTTPS%20displaying.png"/><br>
**5.** <br><img src="https://github.com/gabecoelho/https-ssl-certificate/blob/master/images/topology.JPG"/>
