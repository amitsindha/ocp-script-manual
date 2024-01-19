## NGINX Setup

Connect to main server

```sh
ssh ocpadmin@massuite.online
```

https://github.com/lukasoppermann/articles/blob/master/Setting%20up%20lets%20encrypt%20and%20nginx%20with%20docker%20on%20ubuntu%2016.04.md

```sh
sudo mkdir nginx-docker
sudo vi docker-compose.yml
```

add below content

```sh
version: '3'
services:
    nginx:
        # using alpine for newer openssl for http2 with ALPAN
        image: nginx:alpine

        container_name: nginx

        # these volumes are mounted to the nginx container
        volumes:
            # mount the folder ../letsencrypt from our server to the folder /etc/letsencrypt on the container
            - ../letsencrypt/:/etc/letsencrypt
            # mount ./nginx/var/www/html to the /var/www/html on the container
            - ./nginx/var/www/html:/var/www/html
            # mount ./logs/nginx/ to the /var/log/nginx on the container, rw = read-write
            - ./logs/nginx/:/var/log/nginx:rw
            # mount ./nginx/includes to the /etc/nginx/includes on the container, ro = read-only
            - ./nginx/includes:/etc/nginx/includes:ro
            # mount ./nginx/conf.d/default.conf to the /etc/nginx/conf.d/default.conf on the container
            - ./nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf
            # mount ./sites to the /sites on the container (will hold our node app)
            - ../sites:/sites
        ports:
            - "80:80"
            - "443:443"
        # makes container restart on server restart
        restart: always
        networks:
            - appnet
# used for interal communication within docker
networks:
    appnet:
        driver: "bridge"
```

Create NGINX services

```sh
sudo docker-compose up -d
```

It will give error, to resolve it create remove the directory 

```sh
sudo rm -rf /home/ocpadmin/nginx-docker/nginx/conf.d/default.conf
```

Create a new file

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/default.conf
```

Add below content 

```sh
# don't redirect proxy
proxy_redirect  off;
# turn off global logging
access_log off;
# logging format
log_format compression '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $bytes_sent '
                       '"$http_referer" "$http_user_agent" "$gzip_ratio"';
############################
#
# allow let`s Encrypt on port 80 for all domains
#
server {
  listen 80;
  listen [::]:80;
  server_name ~. ;

  location /.well-known/acme-challenge {
    root /var/www/html;
    default_type text/plain;
  }

  location / {
    return 301 https://$host$uri;
  }
}
```

Create NGINX services

```sh
sudo docker-compose up -d
```

Creating certificates (Make sure you change the email id and domain names). Below command execute with dry run. Once it sucessful then remove the dry run flag. Also stagin flag is used for testing. Once you tested everything then remove the staging flag and execute again for the Production. (There is a limi of generating letsencryp certificate with requests, so we are using the staging flag for non production testing) We are using the RM flag which will remove the contaner once certificate will be generated. 

To generate the actual certificate we need to remove the staging flag as well.

```sh
docker run -it --rm \
	--name letsencrypt \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  --volumes-from nginx \
  certbot/certbot certonly \
  --webroot \
  --webroot-path /var/www/html \
  --agree-tos \
  --staging \
  --dry-run \
  -m amit.sindha@gmail.com \
  -d massuite.online
  
```

Execute below command to genrate the certificate

```sh
docker run -it --rm \
	--name letsencrypt \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  --volumes-from nginx \
  certbot/certbot certonly \
  --webroot \
  --webroot-path /var/www/html \
  --agree-tos \
   -m amit.sindha@gmail.com \
  -d massuite.online
```

Renewing certificates (Not required at the moment)

```sh
docker run -it --rm \
  --volumes-from nginx \
  certbot/certbot renew
```

Testing certificate

make sure certificate is working you can add the following at the end of /home/ocpadmin/nginx-docker/nginx/conf.d/default.conf

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/default.conf
```

```sh
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name massuite.online; # replace with your domain
	
	# replace your domain in both paths
	ssl_certificate /etc/letsencrypt/live/massuite.online/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/massuite.online/privkey.pem;

	location / {
		root         /var/www/html;
	}
}
```

Reload NGINX Services

```sh
sudo docker exec -it nginx nginx -t
sudo docker exec -it nginx nginx -s reload
```

Test NGINX services

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/var/www/html/index.html
```

Add content as below 

```sh
<h1>Hello world!</h1>
```

Execute below command to genrate the certificate


Create a new file for OCP console

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-console.conf
```

Add below content 

```sh
############################
#
# allow let`s Encrypt on port 80 for all domains
#
server {
  listen 80;
  listen [::]:80;
  server_name console-openshift-console.apps.ocp4.massuite.online ;

  location /.well-known/acme-challenge {
    root /var/www/html;
    default_type text/plain;
  }

  location / {
    return 301 https://$host$uri;
  }
}
```

Execute below command to generate OCP console certificate

```sh
docker run -it --rm \
	--name letsencrypt \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  --volumes-from nginx \
  certbot/certbot certonly \
  --webroot \
  --webroot-path /var/www/html \
  --agree-tos \
   -m amit.sindha@gmail.com \
  -d console-openshift-console.apps.ocp4.massuite.online
```

Edit OCP conf file

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-console.conf
```

```sh
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name console-openshift-console.apps.ocp4.massuite.online; # replace with your domain
	
	# replace your domain in both paths
	ssl_certificate /etc/letsencrypt/live/console-openshift-console.apps.ocp4.massuite.online/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/console-openshift-console.apps.ocp4.massuite.online/privkey.pem;

	location / {
		root         /var/www/html;
	}
}
```

Reload NGINX Services

```sh
sudo docker exec -it nginx nginx -t
sudo docker exec -it nginx nginx -s reload
```

Create a new file for OCP API

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-api.conf
```

Add below content 

```sh
############################
#
# allow let`s Encrypt on port 80 for all domains
#
server {
  listen 80;
  listen [::]:80;
  server_name api.ocp4.massuite.online ;

  location /.well-known/acme-challenge {
    root /var/www/html;
    default_type text/plain;
  }

  location / {
    return 301 https://$host$uri;
  }
}
```

Execute below command to generate OCP console certificate

```sh
docker run -it --rm \
	--name letsencrypt \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  --volumes-from nginx \
  certbot/certbot certonly \
  --webroot \
  --webroot-path /var/www/html \
  --agree-tos \
   -m amit.sindha@gmail.com \
  -d api.ocp4.massuite.online
```

Edit OCP conf file

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-api.conf
```

```sh
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name api.ocp4.massuite.online; # replace with your domain
	
	# replace your domain in both paths
	ssl_certificate /etc/letsencrypt/live/api.ocp4.massuite.online/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/api.ocp4.massuite.online/privkey.pem;

	location / {
		root         /var/www/html;
	}
}
```

Reload NGINX Services

```sh
sudo docker exec -it nginx nginx -t
sudo docker exec -it nginx nginx -s reload
```

Create a new file for OCP API

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-oauth.conf
```

Add below content 

```sh
############################
#
# allow let`s Encrypt on port 80 for all domains
#
server {
  listen 80;
  listen [::]:80;
  server_name oauth-openshift.apps.ocp4.massuite.online ;

  location /.well-known/acme-challenge {
    root /var/www/html;
    default_type text/plain;
  }

  location / {
    return 301 https://$host$uri;
  }
}
```

Execute below command to generate OCP console certificate

```sh
docker run -it --rm \
	--name letsencrypt \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  --volumes-from nginx \
  certbot/certbot certonly \
  --webroot \
  --webroot-path /var/www/html \
  --agree-tos \
   -m amit.sindha@gmail.com \
  -d oauth-openshift.apps.ocp4.massuite.online
```

Edit OCP conf file

```sh
sudo vi /home/ocpadmin/nginx-docker/nginx/conf.d/ocp-oauth.conf
```

```sh
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name oauth-openshift.apps.ocp4.massuite.online; # replace with your domain
	
	# replace your domain in both paths
	ssl_certificate /etc/letsencrypt/live/oauth-openshift.apps.ocp4.massuite.online/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/oauth-openshift.apps.ocp4.massuite.online/privkey.pem;

	location / {
		root         /var/www/html;
	}
}
```

Reload NGINX Services

```sh
sudo docker exec -it nginx nginx -t
sudo docker exec -it nginx nginx -s reload
```

Verify listning ports 

https://linuxize.com/post/check-listening-ports-linux/

Temporary start listning 6443 port for OC client

```sh
sudo vi /etc/systemd/system/listen-6443.service
```
Update the file with below content

```sh
[Unit]
Description=Permanent listen at :::6443
After=network-online.target

[Service]
User=root
ExecStart=/usr/bin/nc -l6 6443
ExecStop=/usr/bin/killall -s KILL nc
Restart=always
RestartSec=1

[Install]
WantedBy=multi-user.target
```

Enable and start the listen-6443.service

```sh
sudo systemctl daemon-reload
sudo systemctl enable listen-6443.service 
sudo systemctl start listen-6443.service 
sudo systemctl status listen-6443.service
```

Disable and stop the listen-6443.service

```sh
sudo systemctl stop listen-6443.service 
sudo systemctl disable listen-6443.service
```
