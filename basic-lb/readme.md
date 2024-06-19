#### Go to NGINX VM using vscode option

![vscode access](media/vscode_access.png)
---
NGINX Plus with App Protect has been installed in this lab. By default, NGINX configurations are located here:
- /etc/nginx/nginx.conf
- /etc/nginx/conf.d/default.conf

#### Let's examine the the content of each file respectively

![nginx conf](media/nginx_conf.png)

![default conf](media/default_conf.png)

This folder keeps NGINX default files for healthcheck dashboard and error pages

![usr share nginx html](media/usr_share_nginx_html.png)

---
#### Next, we'll check out three web pages and configure NGINX to load balance them
```cd /opt/hol/```

![three apps](media/three_apps.png)

```cat App1/index.html```

![app content](media/app_content.png)

Do the same for App2 and App3
---
