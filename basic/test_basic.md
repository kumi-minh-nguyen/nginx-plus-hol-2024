# Access App1
curl http://localhost:9001

# Access App2
curl http://localhost:9002

# Access App3
curl http://localhost:9003

# Test load balancer
curl http://localhost:9000

# Check dashboard
curl http://10.1.1.4:8081

# Test App Protect with cross site scripting attack
curl localhost:9000
curl 'localhost:9000/?<script>'

# Monitor nginx error log
sudo tail -f /var/log/nginx/error.log
