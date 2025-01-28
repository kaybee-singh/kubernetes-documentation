Inginx controller can be configured to give our services external access.

- To enable the nginx controller in minikube and check if it is enabled.
```bash
minikube addons enable ingress
kubectl get pods -n ingress-nginx
```
- Create a deployment
```bash
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```
- Expose the deployment and create a service
```bash
kubectl expose deployment web --type=NodePort --port=8080
```
- Create ingress
```bash
cat ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: hello-world.example
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
```
- Now apply the configuration
```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```
- Make an entry in /etc/hosts for your minikube node ip.
```bash
192.168.0.10 hello-world.example
```
- Now try to access the service by browsing the hello-world.example.
  - Create one more deployment and expose it
```bash
kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment web2 --port=8080 --type=NodePort
```
- Edit the ingress.yaml and add web2 service configuration at end.
```bash
- path: /v2
  pathType: Prefix
  backend:
    service:
      name: web2
      port:
        number: 8080
```
- Apply the configuration
```bash
kubectl apply -f ingress.yaml
```
