1. Nodeport is used to expose a kubernetes service on a static port on a kubernetes node. It makes the service accessible from outside the cluster. For instance, in below output a single node minikube cluster has `192.168.39.53` IP address on minikube node. 
Any ervice running on this node can be mapped to a static port ranging between 30000-32767. If we try to access IP:PORT from outside then we will be redirected to the service which is mapped to port.

```bash
$ kubectl get nodes  -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    control-plane   18m   v1.30.0   192.168.39.53   <none>        Buildroot 2023.02.9   5.10.207         docker://26.0.2
```

2. Create a test deployment.

```bash
$ cat test-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2  # Number of replicas
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80

$ kubectl apply -f test-deployment.yaml 
```

3. Create the **NodePort** type service to expose the deployment on a port.

```bash
$ kubectl expose deployment nginx-deployment --type=NodePort --name=nginx-service --port=80 --target-port=80

```

4. Now check the port on which that service is mapped. As per the output service is mapped to **30937** port.

```bash
$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        26m
nginx-service   NodePort    10.110.3.127   <none>        80:30937/TCP   21m
````

5. Now try to access the pod from browser or with curl

```bash
 curl 192.168.39.53:30937
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

