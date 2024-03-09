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
- In my case I'm on my company VM so, i don't have public ip but insted of company/personal VM if you have cloud provider like EKS/AKS they provide EKL (elastics loadBalance with public IP).

# What to do?

- If you have bare metal or VM on which your k8s running than you need baremetal loadBalancer so, in my time : https://metallb.universe.tf/
- MetalLB is used for bareMetal or VM k8s.
- So, let's begin with it.

**Requirements** 
- running Kubernetes 1.13.0 or later [we have 1.29 +]
- A cluster network configuration that can coexist with MetalLB. [we have calico]
- Some IPv4 addresses for MetalLB to hand out.
  use cmd: ip a s ==> ens34 IP:  inet 192.168.60.33/22 brd 192.168.63.255 scope global ens34,
           sipcalc 192.168.60.33/22
- <img width="440" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/8378e662-0b90-4bf9-b01e-0fd5b36bb310">
- Here you need to assign some Ip which are not used by your master and worker node Ip's to MetalLB.
- <img width="827" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/62d74172-edcd-4eb5-8951-f5f5cceeddfd">
- I have only one master node with 33 ip.
- traffic port should allow : 7946 (TCP & UDP) if firewall is enabled in that case only.
- Now go installation page. : https://metallb.universe.tf/installation/
- Now check If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.
- To check

       kubectl get cm  -n kube-system
       kubectl edit cm kube-proxy  -n kube-system

- <img width="318" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/78255c39-0207-4698-b2ab-43b461d0175c">
- here mode= "" value is null so, not worry about it just leave it.

**Installation**

- Install by Manifest

      kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.3/config/manifests/metallb-native.yaml

**Configuration**

- use link: https://metallb.universe.tf/configuration/
- here we use Layer2 configuration.
- nano /tmp/metallb.yaml

       apiVersion: metallb.io/v1beta1
       kind: IPAddressPool
       metadata:
         name: first-pool
         namespace: metallb-system
       spec:
         addresses:
         - 192.168.63.240-192.168.63.250

- kubectl create -f  /tmp/metallb.yaml
- Now you able to see your External Ip
- <img width="654" alt="image" src="https://github.com/ErDharmesh/Kubernetes-zeroToHero/assets/108712914/e439a650-d12e-4544-932e-59e8ba5ff5a0">
