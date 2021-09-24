## Compressio API &middot; [![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

```
1. Compressio API is a Open Source production grade Image Compression API.
2. It compresses JPG, PNG, GIF & SVG images in both Lossy and Lossless formats.
3. Compressio API Open Source Frontend (React.js) is coming soon.
4. It can run on Linux, Unix (MacOS) and Windows with Node.js
5. We are working hard to include more features. 
```

## How to use   

#### POST Request, No GET
```
It receives all parameters in a POST request at https://api.example.com/compress.     
You can choose maximum 5 images at a time.
Each image can't be larger than 10MB
Send image with 'inImgs' key through POST request.
```

#### Compression Type
```
By Default image compression type is lossy.   
You can choose lossless compression with 'isLossy' key and value 'false' through POST request.
```

#### Compression Quality
```
Note : Compression Quality (imgQuality) won't work if isLossy is set to false.  
You can choose between 1-100 with 'imgQuality' key and value through POST request. 

Default JPG, PNG, SVG Image Compression Quality = 85.
Default GIF Image Compression Quality = 50.   
You can choose between 1-100 with 'imgQuality' key and value through POST request.
For GIF, do not choose any value lower than 50, quality loss will be significant.
```

#### Strip Meta
```
By Default image metadata will be stripped.   
You can choose not to strip meta with 'stripMeta' key and value 'false' through POST request.
```

![alt text](https://github.com/twoabd/CompressioAPI/blob/main/docs/default.png?raw=true)  

![alt text](https://github.com/twoabd/CompressioAPI/blob/main/docs/lossy.png?raw=true)  
 
![alt text](https://github.com/twoabd/CompressioAPI/blob/main/docs/lossless.png?raw=true)  

## Installation Documentation (Ubuntu 20.04 LTS)   

#### Install Nginx (Currently v1.18.0) and NodeJs LTS (Currently v14.17.6) on Ubuntu 20.04
```
sudo apt update
sudo apt install nginx -y

curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
cat /etc/apt/sources.list.d/nodesource.list
sudo apt  install nodejs -y
node  -v
```

#### Install PNGQuant, JPEGOptim, Gifsicle, Scour

```
sudo apt install jpegoptim
sudo apt install pngquant -y
sudo apt install optipng
sudo apt install gifsicle
sudo apt install scour -y
```

#### Updating Nginx conf in etc/nginx/nginx.conf
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	multi_accept on;
}

http {

	# Basic Settings
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
        client_max_body_size 20M;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;


	# SSL Settings
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on;


	# Logging Settings
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;


	# Gzip Settings
	gzip on; 
	gzip_disable "msie6";
	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_buffers 16 8k;
	gzip_http_version 1.1;
        gzip_types application/json;


	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

#### Creating API Directory with Necessary Permission

```
sudo mkdir -p /var/www/api.example.com /var/www/api.example.com/input /var/www/api.example.com/output
sudo chown -R www-data:www-data /var/www/api.example.com
sudo chmod -R 755 /var/www/api.example.com
```

#### Creating Virtual Host
```
sudo nano /etc/nginx/sites-available/api.example.com
server {
    listen 80;
    server_name api.example.com;

    #  Web Root
    root /var/www/api.example.com;
   
    # API Folder
    location ^~ /compress {
	    proxy_pass http://localhost:3000;
	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection 'upgrade';
	    proxy_set_header Host $host;
	    proxy_cache_bypass $http_upgrade;
	    proxy_read_timeout 60s;
    }
    
    # Input Folder
    location ^~ "/input/([a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12})/.(jpg|jpeg|png|gif|svg)$" {
        try_files $uri $uri/ =404;
    }

	# Output Folder
    location ^~ "/output/([a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12})/.(jpg|jpeg|png|gif|svg)$" {
        try_files $uri $uri/ =404;
    }
}
sudo ln -s /etc/nginx/sites-available/api.example.com /etc/nginx/sites-enabled/
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

#### Installing SSL
```
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d api.example.com
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
sudo systemctl restart nginx
```

#### Copy Repo Files to /var/www/api.example.com & Install Dependencies
```
cd /var/www/api.example.com
npm install
npm install nodemon -g
```

#### Run Api Server
```
cd /var/www/api.example.com
nodemon app.js
```