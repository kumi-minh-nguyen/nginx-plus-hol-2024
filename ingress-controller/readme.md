#### Start the environment
Use vscode access method on Minikube machine

```minikube start```

Open minikube tunnel in background to expose load balancers 

```bash
screen -S minikube-tunnel
minikube tunnel
```

Press "Ctrl a" then "d" to detach

---

#### Introduction

The NGINX Ingress Controller is already running in this workshop. You will be checking and verifying the Ingress Controller is running.

#### Check your Ingress Controller

1. First, verify the NGINX Ingress controller is up and running correctly in the Kubernetes cluster:

   ```bash
   kubectl get pods -n nginx-ingress
   ```

   ```bash
   ###Sample output###
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-ingress-fd4b9f484-t5pb6   1/1     Running   1          12h
   ```

   **Note**: You must use the `kubectl` "`-n`", namespace switch, followed by namespace name, to see pods that are not in the default namespace.

1. Instead of remembering the unique pod name, `nginx-ingress-xxxxxx-yyyyy`, we can store the Ingress Controller pod name into the `$NIC` variable to be used throughout the lab.

   **Note:** This variable is stored for the duration of the terminal session, and so if you close the terminal it will be lost. At any time you can refer back to this step to save the`$NIC` variable again.

   ```bash
   export NIC=$(kubectl get pods -n nginx-ingress -o jsonpath='{.items[0].metadata.name}')
   ```

   Verify the variable is set correctly.
   ```bash
   echo $NIC
   ```
   **Note:** If this command doesn't show the name of the pod then run the previous command again.

---

#### Deploy the NGINX Dashboard Service

We will deploy a `Service` and a `VirtualServer` resource to provide access to the NGINX Plus Dashboard for live monitoring.  NGINX Ingress [`VirtualServer`](https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/) is a [Custom Resource Definition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) used by NGINX to configure NGINX Server and Location blocks for NGINX configurations.


1. In the `hol` folder, apply the `dashboard-vs.yaml` file to deploy a `Service` and a `VirtualServer` resource to provide access to the NGINX Plus Dashboard for live monitoring:

    ```bash
    kubectl apply -f /home/ubuntu/hol/dashboard-vs.yaml
    ```
    ```bash
    ###Sample output###
    service/dashboard-svc created
    virtualserver.k8s.nginx.org/dashboard-vs created
    ```

2. Test the dashboard in vscode terminal
   
```bash
http://dashboard.example.com:9000/dashboard.html
```

3. Test the dashboard in Minikube VM browser
* Use XRDP access method from Minikube machine
* Log in with credentials ubuntu/HelloUDF
* Open Firefox: `http://dashboard.example.com:9000/dashboard.html`

---

#### Deploy the Cafe Demo app

The Cafe application that you will deploy has Coffee and Tea pods and services, with NGINX Ingress routing the traffic for `/coffee` and `/tea` routes, using the `cafe.example.com` Hostname, and with TLS enabled. 

1. Deploy the Cafe application by applying the three manifests:

    ```bash
    kubectl apply -f /home/ubuntu/hol/cafe-secret.yaml
    kubectl apply -f /home/ubuntu/hol/cafe.yaml
    kubectl apply -f /home/ubuntu/hol/cafe-vs.yaml
    ```
    ```bash
    ###Sample output###
    secret/cafe-secret created
    deployment.apps/coffee created
    service/coffee-svc created
    deployment.apps/tea created
    service/tea-svc created
    virtualserver.k8s.nginx.org/cafe-vs created
    ```

2. Check that all pods are running, you should see **three** Coffee and **three** Tea pods:

    ```bash
    kubectl get pods
    ```
    ```bash
    ###Sample output###
    NAME                      READY   STATUS    RESTARTS   AGE
    coffee-56b7b9b46f-9ks7w   1/1     Running   0             28s
    coffee-56b7b9b46f-mp9gs   1/1     Running   0             28s
    coffee-56b7b9b46f-v7xxp   1/1     Running   0             28s
    tea-568647dfc7-54r7k      1/1     Running   0             27s
    tea-568647dfc7-9h75w      1/1     Running   0             27s
    tea-568647dfc7-zqtzq      1/1     Running   0          27s
    ```

3. Check that the Cafe `VirtualServer` , **`cafe-vs`**, is running:

    ```bash
    kubectl get virtualserver cafe-vs
    ```
    ```bash
    ###Sample output###
    NAME      STATE   HOST               IP    PORTS   AGE
    cafe-vs   Valid   cafe.example.com                 4m6s
    ```

    **Note:** The `STATE` should be `Valid`.  If it is not, then there is an issue with your yaml manifest file `(cafe-vs.yaml)`.  You could also use `kubectl describe vs cafe-vs` to get more information about the `VirtualServer` we just created.

4. Compare VirtualServer and Ingress manifest

---

#### End to End Encryption  

Not just the traffic between customers and the Ingress Controller, but also between the Ingress Controller and all of the application pods, this is called `End to End Encryption`, and is the first part of a high security application design, often called Mutual TLS. So now you have to configure/enable TLS between the Ingress Controller and the Cafe coffee and tea pods, and verify that it works.

1. First, for proper testing, you need to remove the previous Cafe virtual server, as we will still be using the cafe.example.com hostname:

    ```bash
    kubectl delete vs cafe-vs
    ```

2. Check the Plus Dashboard, the Cafe HTTP Zone should now be gone.

3. Start a fresh Cafe Demo, deploy the TLS enabled pods and services and Virtual Server manifests:

    ```bash
    kubectl apply -f /home/ubuntu/hol/cafe-mtls.yaml
    kubectl apply -f /home/ubuntu/hol/cafe-mtls-vs.yaml
    ```
    
4. Check the Plus Dashboard, and your new End-to-End TLS Cafe Application - ensure all 6 "mtls" coffee and tea pods are now in Up/Green status.

5. Using Firefox, check the access to coffee and tea as before:

    https://cafe.example.com/coffee
    
    https://cafe.example.com/tea

    Do you see the pod `Server Name` now shows coffee-mtls-pod-name and tea-mtls-pod-name ?

    Do you see the pod `Server Address` now shows port 443, and not 80 ?

---

#### Blue/Green | A/B Testing

During the development cycle of modern applications for Kubernetes, developers will often want to test new versions of their software, using various test tools, and ideally, a final check with live customer traffic.  There are several names for this dev/test concept - `Blue/Green deployments, A/B testing, Canary testing,` etc.  

However, switching ALL customers to new versions that might still have a few bugs in the code is quite risky. Wouldn't it be nice if your Ingress Controller could split off just a small fraction of your live traffic, and route it to your new application pod for final testing?  

NGINX Plus Ingress Controller can do this, using a feature called `HTTP Split Clients.`  This feature allows you to define a percentage of traffic to be split between different k8s Services, representing different versions of your application.

You will use the currently running Cafe coffee-mtls and tea-mtls pods, and split the traffic at an 80:20 ratio between coffee-mtls and tea-mtls Services.  

**Assume that coffee is your existing application, and tea is your new test build of the application.**  

Having read the tea leaves you are highly confident in your new code. So you decide to route 20% of your live traffic to tea-mtls.

1. First, to see the split ratio more clearly, scale down the number of coffee and tea pods to just one each:

    ```bash
    kubectl scale deployment coffee-mtls --replicas=1
    kubectl scale deployment tea-mtls --replicas=1
    ```

2. Check the Plus Dashboard, there should only be one coffee and one tea upstream now.

    Inspect the `home/ubuntu/hol/cafe-bluegreen-vs.yaml` file, and note the `split and weight` directives on lines 49-56.

3. Next, remove the existing VirtualServer for mTLS from the previous exercise:

    ```bash
    kubectl delete -f /home/ubuntu/hol/cafe-mtls-vs.yaml
    ```

4. Now configure the Cafe VirtualServer to send `80%` traffic to coffee-mtls, and `20%` traffic to tea-mtls:

    ```bash
    kubectl apply -f /home/ubuntu/hol/cafe-bluegreen-vs.yaml
    ```

5. Open a Firefox tab for https://cafe.example.com/coffee, and check the Auto Refresh box at the bottom of the page.


    Check the statistics on the Plus Dashboard Cafe upstreams... Do you see approximately an 80/20 Requests ratio between coffee-mtls and tea-mtls?  You can configure the ratio in 1% increments, from 1-99%.  

    **Note:** NGINX will not load the Split configuration, if the ratio does not add up to 100%.
