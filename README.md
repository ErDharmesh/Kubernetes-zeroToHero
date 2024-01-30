# Kubernetes-zeroToHero
In this document, I will show you the from the basic, how to write a simple app, build it, containerize it, deploy in kubernetes and expose to external world using Kubernetes services.

# prerequisite
install minikube, docker, node,

# check minikube status
1. minikube status
    minikube
    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured

# build one application
1. we are going with angular application.
2. check the version or install it.

# HTML website simple one

1. vi index.html
           
        <!DOCTYPE html>
        <html>
        <head>
        <title>HTML Website</title>
        </head>
        <body>
        
        <h1>Kubernetes Demo</h1>
        <p>Here you will learn kubernetes.</p>
        
        </body>
        </html>

2. vi Dockerfile
FROM python
COPY index.html index.html
EXPOSE 6100
CMD ["python", "-m","http.server","6100"]

3. docker build .
4. docker images
5. docker run -it -p 6100:6100  48094f98be49
6. HTTP://localhost:6100
7. docker tag 48094f98be49 docker.io/dharmesh09/html-web:v1
8. docker push docker.io/dharmesh09/html-web:v1
9. now with docker your html application is working well.
10. with kubernetes let's try
11. For kubernetes required
    1. deployment.yaml
    2. service.yaml
12. vi deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: html-web
  labels:
    app: html-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: html-web
  template:
    metadata:
      labels:
        app: html-web
    spec:
      containers:
      - name: html-web
        image: docker.io/dharmesh09/html-web:v1
        ports:
        - containerPort: 6100

13. kubectl get pod -o wide
NAME                            READY   STATUS    RESTARTS       AGE     IP           NODE       NOMINATED NODE   READINESS GATES
html-web-799dcc499f-4ckt9       1/1     Running   0              5m20s   10.244.0.9   minikube   <none>           <none>

14. check with this ip pod can communicate or not.
15. minikube ssh 
16. curl -L 10.244.0.9:6100 [here you will see your HTML file]
docker@minikube:~$ curl -L 10.244.0.9:6100
        <!DOCTYPE html>
        <html>
        <head>
        <title>HTML Website</title>
        </head>
        <body>
        
        <h1>Kubernetes Demo</h1>
        <p>Here you will learn kubernetes.</p>
        
        </body>
        </html>

17. here when your pod is killed or recreated that time it's assigned with new ip address. so, here we need service.yaml becoz it's ip is not chnage that's advantage of it.
18. vi service.yaml
apiVersion: v1
kind: Service
metadata:
  name: html-web
spec:
  type: NodePort
  selector:
    app: html-web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 6100
      nodePort: 31080

19. here nodePort size between: 30k to 32k
20.kubectl apply -f service.yaml
21.  kubectl get svc -o wide
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
html-web       NodePort    10.109.119.202   <none>        80:31080/TCP   8s    app=html-web
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        23h   <none>

22. again you can check with now cluster ip your ssh will comunicate or not.
23. minikube ssh
24. docker@minikube:~$ curl -L 192.168.49.2:31080
        <!DOCTYPE html>
        <html>
        <head>
        <title>HTML Website</title>
        </head>
        <body>
        
        <h1>Kubernetes Demo</h1>
        <p>Here you will learn kubernetes.</p>
        
        </body>
        </html>

25. now your pod is 100 time recreated you will get response with this ip's only. that's advantage of k8s.
26. now this ip is only communicate only internal cluster communication.
27. i want to access it on my crome browser.
28. it happened by ingress only.
29. vi ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: html-web.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: html-web
            port:
              number: 80


30. kubectl apply -f ingress.yaml
31. kubectl get ingress -n default
NAME                    CLASS    HOSTS      ADDRESS        PORTS   AGE
ingress-wildcard-host   <none>   html-web   192.168.49.2   80      21h

done




●
● 
● 
● 
