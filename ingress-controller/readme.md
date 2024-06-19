## Introduction

The NGINX Ingress Controller is already running in this workshop. You will be checking and verifying the Ingress Controller is running.

## Learning Objectives 
- Intro to NGINX Ingress Controller
- Intro to Kubernetes environment, interacting with `kubectl` command
- Access the NGINX Plus Dashboard

## Check your Ingress Controller

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

## Inspect the details of your Ingress Controller:

1. Inspect the details of the NGINX Ingress Controller pod using the `kubectl describe` command

   ```bash
   kubectl describe pod $NIC -n nginx-ingress
   ```

   **Note:** The IP address and TCP ports that are open on the Ingress. We have the following Ports:

   * Port `80 and 443` for web/ssl traffic,
   * Port `8081` for Readiness Probe, 
   * Port `9000` for the Dashboard, and 

2. Check the NIC Plus Dashboard
   
   * Use XRDP access method from Minikube machine
   * Log in with credentials ubuntu/HelloUDF
   * Open Firefox: `http://dashboard.example.com:9000/dashboard.html`
     
