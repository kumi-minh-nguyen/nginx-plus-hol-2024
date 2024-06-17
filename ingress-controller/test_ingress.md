### Test cafe-vs and cafe-mtls-vs
```bash
curl -k -I https://cafe.example.com/tea
curl -k -I https://cafe.example.com/coffee
curl -k -I https://cafe.example.com/
```

### Health check and bluegreen testing
```bash
curl -k -I http://dashboard.example.com:9000/dashboard.html
```

### Monitor ingress logs
```bash
kubectl get pods -n nginx-ingress
export NIC=$(kubectl get pods -n nginx-ingress -o jsonpath='{.items[0].metadata.name}')
echo $NIC
kubectl logs -n nginx-ingress $NIC --follow --tail=20
```
