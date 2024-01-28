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

# for complex app








**commands**

npm install -g create-react-app
npx create-react-app my-react-app
cd my-react-app
npm start

● now open VScode
● add below Dockerfile

  FROM node:alpine
  WORKDIR /usr/src/app
  COPY ["package.json", "./"]
  RUN npm install
  COPY . .
  EXPOSE 3000
  CMD ["npm", "run", "start"]

● docker build -t react-app .
● docker run -it -p 3000:3000 react-app

● docker tag react-app docker.io/dharmesh09/my-react-app:v1
● docker push docker.io/dharmesh09/my-react-app:v1

# For kubernetes required
1. deployment.yaml
2. service.yaml

# steps
1. Go to this website and copy deployment.yaml
2. https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
3. now create one deployment.yaml file with below contains

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-react-app
  labels:
    app: my-react-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: my-react-app
        image: docker.io/dharmesh09/my-react-app:v1
        ports:
        - containerPort: 3000

4. kubectl apply -f deployment.yaml
5. kubectl get pod -n default
NAME                            READY   STATUS              RESTARTS   AGE
my-react-app-867cd5f854-mvpjv   0/1     ContainerCreating   0          25s

6. kubectl get svc  -n default  
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   172m

7. kubectl get pod -o wide    
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
my-react-app-867cd5f854-mvpjv   1/1     Running   0          3m14s   10.244.0.3   minikube   <none>           <none>

8. https://10.244.0.3:3000 it is not working...becoz it is internal cluster ip which will used internally by cluster.
9. here understand that whenver your pod is stopped or kill and new pod will created that time your pod ip will change so, for that reason we are using service.yaml
10.   create one service.yaml file

apiVersion: v1
kind: Service
metadata:
  name: my-react-app
spec:
  selector:
    app: demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000

11. kubectl apply -f service.yaml
12. here new service is created
13. kubectl get svc  -n default
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP   3h3m
my-react-app   ClusterIP   10.106.42.182   <none>        80/TCP    17s

14. minikube ssh [for login to cluster]
15.  minikube ssh
docker@minikube:~$ curl -L 10.106.42.182   ==> html content you will got so, you are on right path...
16. now nodePort:
17. change service.yaml and again apply it.here nodePort range is between: 30k to 32k

apiVersion: v1
kind: Service
metadata:
  name: my-react-app
spec:
  type: NodePort
  selector:
    app: demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30123

18.  kubectl apply -f service.yaml
service/my-react-app configured

19.  kubectl get svc  -n default
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        3h15m
my-react-app   NodePort    10.106.42.182   <none>        80:30123/TCP   11m

20. minikube ip
192.168.49.2

21. now access your pod with: 192.168.49.2:30123
22. minikube ssh
23. curl -L 192.168.49.2:30123

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="/manifest.json" />
    <!--
      Notice the use of  in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  <script defer src="/static/js/bundle.js"></script></head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  </body>
</html>

24. successfully it's worked.
25. for the ingress you have to add ingress on your local.
26. minikube addons enable ingress
* ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
* After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
  - Using image registry.k8s.io/ingress-nginx/controller:v1.9.4
* Verifying ingress addon...
* The 'ingress' addon is enabled

27. minikube tunnel
28. after this in your ingress you will be able to see ip address.
29. make ingress.yaml file and paste below code.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: demo-react
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: my-react-app
            port:
              number: 80

31. here should be same service name used.
32. kubectl apply -f ingress.yaml
33. add your host into etc/hosts file with ingress ipAddress.
34. here your ingress ip is same as to minikube ip.
35. kubectl get ingress
NAME                    CLASS    HOSTS        ADDRESS        PORTS   AGE
ingress-wildcard-host   <none>   demo-react   192.168.49.2   80      19h






●
● 
● 
● 
