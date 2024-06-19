### Access App1
```bash
curl http://localhost:9001
```

### Access App2
```bash
curl http://localhost:9002
```

### Access App3
```bash
curl http://localhost:9003
```

### Test load balancer
```bash
curl http://localhost:9000
```

### Check dashboard
```bash
curl http://10.1.1.4:8081
```

### Test App Protect with cross site scripting attack
```bash
curl localhost:9000
curl 'localhost:9000/?<script>'
```

### Monitor nginx error log
```bash
sudo tail -f /var/log/nginx/error.log
```
