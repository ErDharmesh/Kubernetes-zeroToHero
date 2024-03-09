# Steps:

- Clone below repo first

       git clone https://github.com/iam-veeramalla/Docker-Zero-to-Hero
       cd Docker-Zero-to-Hero/examples/python-web-app
   
- Now build your application using Docker build cmd:

       docker build -t dharmesh009/python-app-demo:v1 .

- Make one deploy file: python-deploy.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: sample-python-app
          labels:
            app: sample-python-app
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: sample-python-app
          template:
            metadata:
              labels:
                app: sample-python-app
            spec:
              containers:
              - name: python-demo-app
                image: dharmesh009/python-app-demo:v1
                ports:
                - containerPort: 8000

-  kubectl create -f  python-deploy.yaml
-  <img width="823" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/9d689aa6-d61e-4b92-84a8-3a94b06dd271">
-  <img width="602" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/3d05ddc1-dd17-43a6-ba8b-3322fba57746">
- Here your app is listening on 8000 port with: /demo so,
- <img width="783" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/a5d5f41d-1038-42b4-b104-2420e890daea">
- it's worked!!
- but till now my application is listen only internally.
- Here we have to solve 2 problems:
- 1. peoples within organizations (enzigma) should able to access this application.
  2. people who are not in organization should able to access this app [for this need public Ip Address assign to VM which is accessible from anywhere using internet].
 
# 1. How to access within organization

- Create service: python-service.yaml

      apiVersion: v1
      kind: Service
      metadata:
        name: python-app
      spec:
        type: NodePort
        selector:
           app: sample-python-app
        ports:
          - port: 80
            # By default and for convenience, the `targetPort` is set to
            # the same value as the `port` field.
            targetPort: 8000
            # Optional field
            # By default and for convenience, the Kubernetes control plane
            # will allocate a port from a range (default: 30000-32767)
            nodePort: 30007

- kubectl apply -f python-service.yaml
- <img width="471" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/78875809-b48d-4755-bbe1-812866a71c75">
- Here cluster ip (10.101.24.10:80) is mapped with node ip (192.168.60.33) with port 30007.
- <img width="611" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/7f6b48d5-b512-4ad6-a0bf-d1c434b6befa">
- see same output here.
- Now you can use here on your browser: http://192.168.60.33:30007/demo
- <img width="960" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/ca58fb3d-2dd3-432e-ae35-5e11bc8ef895">
- here we have seen within organization by nodePort
- now to expose to public over the internet you need to edit service and change type: NodePort to type: LoadBalancer and save it.
- now when you get svc again then you can find external ip.
- <img width="489" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/d2b88425-b467-456c-87e4-0c04f9d9134e">
- In my case I'm on my company VM so, i don't have public ip but insted of company/personal VM if you have cloud provider like EKS/AKS they provide EKL (elastics loadBalance with public ip).
- 
