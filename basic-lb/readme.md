#### Go to NGINX VM using vscode option

![vscode access](media/vscode_access.png)

---
#### NGINX Plus with App Protect has been installed in this lab. By default, NGINX configurations are located here:
- /etc/nginx/nginx.conf
- /etc/nginx/conf.d/default.conf

#### Let's examine the the content of each file respectively

![nginx conf](media/nginx_conf.png)

![default conf](media/default_conf.png)

This folder keeps NGINX default files for healthcheck dashboard and error pages

![usr share nginx html](media/usr_share_nginx_html.png)

---
#### The lab contains three html web pages. Let's check them out.
```cd /opt/hol/```

![three apps](media/three_apps.png)

```cat App1/index.html```

![app content](media/app_content.png)

Do the same for App2 and App3. All three web config are almost similar; the difference is color code.
```bash
cat App2/index.html
cat App3/index.html
```
---
#### Next we will showcase the power and simplicity of NGINX 

Change to root user and go to /etc/nginx/conf.d folder

```bash
sudo su -
cd /etc/nginx/conf.d
```

*Note: for testing, you can use curl from vscode command line or firefox from the NGINX Access Method panel.*

**1. Serve web content**
   
From this git project, copy the content of `basic-lb/config/web.conf` and paste it to `/etc/nginx/conf.d/web.conf` then check the result

```bash
vi web.conf
curl http://10.1.1.4:9001
curl http://10.1.1.4:9002
curl http://10.1.1.4:9003
```
In Firefox
```bash
http://10.1.1.4:9001
http://10.1.1.4:9002
http://10.1.1.4:9003
```

**2. Configure load balancer**
   
From this git project, copy the content of `basic-lb/config/lb.conf` and paste it to `/etc/nginx/conf.d/lb.conf` then check the result

```bash
vi lb.conf
curl http://10.1.1.4:9000
```
In Firefox
```bash
http://10.1.1.4:9000
```

**3. Configure healthcheck dashboard**
   
From this git project, copy the content of `basic-lb/config/dashboard.conf` and paste it to `/etc/nginx/conf.d/dashboard.conf` then check the result 

```bash 
vi dashboard.conf
curl http://10.1.1.4:8081/dashboard.html
```
In Firefox
```bash
http://10.1.1.4:8081/dashboard.html
```



