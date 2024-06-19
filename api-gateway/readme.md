Stay in the same NGINX machine.

From this git project, copy the content of `api-gateway/config/api_gateway.conf` to `/etc/nginx/conf.d/api_gateway.conf`

```bash
sudo su -
cd /etc/nginx/conf.d
vi api_gateway.conf
```

Let's see how to configure different features for our API Gateway

**1. Routing**

Define server group

```bash
...
upstream f1-api {
    #resolver 127.0.0.11; #DNS NGINX Plus feature
    zone api_upstreams 64k;
    server 10.1.1.5:8080;
    server 10.1.1.5:8081;

    #Demo Persistency
    sticky cookie srv_id expires=1h;
}
...
```

Define api paths

```bash
...
location /api/f1/drivers {
    proxy_pass http://f1-api/drivers;
}
...
```

**2. SSL Termination** 

Define port as well as SSL cert and key location

```bash
...
listen 8443 ssl;
ssl_certificate apigwdemo.com.crt;
ssl_certificate_key apigwdemo.com.key;
ssl_protocols TLSv1.3 TLSv1.2 TLSv1.1;
ssl_prefer_server_ciphers on;

location /api/f1/drivers {
    proxy_pass http://f1-api/drivers;
}
...
```

To test

```curl -k https://localhost:8443/api/f1/drivers?year=2019 | jq && echo```

**3. Get Header Response in Result** 

```bash
...
location = /get {
    #Add response header to demo load balancing
    add_header X-Upstream $upstream_addr;
    proxy_pass http://f1-api/drivers;
...
```

To test

```curl -k -v https://localhost:8443/get | jq && echo```

**4. API Key Authentication**
```bash
...
#Demo API Key Authentication
map $http_x_api_key $api_client_name {
    default "";
    "P5FcvLwkyN7eethF" "client_one";
    "UEt5QAEtiCYLKV5v" "client_two";
}
...
```

```bash
...
# API key validation
location = /_validate_apikey {
    internal;
    if ($http_x_api_key = "") {
        return 401; # Unauthorized
    }
    if ($api_client_name = "") {
        return 403; # Forbidden
    }
    return 204; # OK (no content)
}
...
```

```bash
...
location /post {
    # if ( $request_method !~ ^(POST)$ ) {
    #     return 405;
    # }

    #Demo API Key Authentication
    auth_request /_validate_apikey;
    proxy_pass http://f1-api/seasons;

}
...
```

To test

```curl -k -H "x-api-key:P5FcvLwkyN7eethF" https://localhost:8443/post | jq && echo```

**5. Key-Value Store**
```bash
...
#Demo Key-Value Store - NGINX Plus feature
keyval_zone zone=apikeyzone:32k state=/etc/nginx/apikey.keyval;
keyval $http_x_kv_api_key $kv_isallowed zone=apikeyzone;
...
```

```bash
...
location = /anything {
    #Demo Key-Value store API Key authentication
    if ( $kv_isallowed != 1 ) {
        return 403;
    }
    proxy_pass http://f1-api/circuits;
}
...
```

To test

```curl -k -H "x-kv-api-key:2j1PM5rwgt1" https://localhost:8443/anything | jq && echo```

**6. Rate Limit**
```bash
...
#Demo Rate Limit
limit_req_zone $remote_addr zone=perclient:1m rate=2r/s;
...
```

```bash
...
location = /get {

    #Rate-Limit applied as below
    limit_req zone=perclient nodelay;
    limit_req_status 429;

    health_check; #NGINX Plus feature
    proxy_pass http://f1-api/drivers;
}
...
```

To test

```for i in {1..10}; do curl -k https://localhost:8443/get | jq && echo; done```

**7. JWT Authentication**
```bash
...
location /drivers {

    #Demo JWT Authentication - NGINX Plus Feature
    auth_jwt on;
    auth_jwt_key_file api_secret.jwk;

    # JWT Authentication Logs
    access_log  /var/log/nginx/jwt.access.log jwt;

    if ( $jwt_claim_uid = 222 ) {
        add_header X-jwt-claim-uid "$jwt_claim_uid";
        add_header X-jwt-status "Redirected to Backend-API";
        proxy_pass http://f1-api;
    }

    if ( $jwt_claim_uid != 222 ) {
        return 403;
    }

}
...
```

To test

```TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJ1aWQiOjIyMn0.L7cAao32jKJGKEgdWyfKzDn6FC-3baJv6Rl1E6lGwY0" ; curl -k -H "Authorization: Bearer $TOKEN" https://localhost:8443/drivers | jq && echo```

