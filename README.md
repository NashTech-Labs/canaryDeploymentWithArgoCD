# canaryDeploymentWithArgoCD
A canary deployment is a progressive rollout of an application that splits traffic between an already-deployed version and a new version, rolling it out to a subset of users before rolling out fully. </br>

## Argo CD Setup
Pre-requistes : </br>
- Any Kubernetes cluster (EKS, AKS, GKE)  
- Ingress controller integration: NGINX, ALB 
- Service Mesh integration: Istio, Linkerd, SMI
- Metric provider integration: Prometheus, Wavefront, Kayenta, Web, Kubernetes Jobs

We are using "EKS with Nginx ingress"</br>
Custom Domain = ```<Your Domain or URL of laod-balancer>```


Connecting with my EKS Client (Bastion HOST): </br>
 ```ssh -i "PEM-FILE-NAME" user@linuxconfig.org```


To see the dashboard of Argoroll-out. Get the Public IP of EKS Client (Bastion HOST) and use port number 3100. Example= "Public IP:3100" </br>

Installation of Argo Rollouts controller </br>

- Create the namespace for installation of the Argo Rollouts controller </br>
```kubectl create namespace argo-rollouts```

- You will see the namespace has been created </br>
```kubectl get ns argo-rollouts```

- we will use the latest version to install the Argo Rollouts controller. </br>
```kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml```

- You will see the controller and other components have been deployed. Wait for the pods to be in the Running state. </br>
```kubectl get all -n argo-rollouts```

- Install Argo Rollouts Kubectl plugin with curl for easy interaction with Rollout controller and resources. </br>
```curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64``` </br>
```chmod +x ./kubectl-argo-rollouts-linux-amd64``` </br>
```sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts``` </br>
```kubectl argo rollouts version``` </br>

- Argo Rollouts comes with its own GUI as well that you can access with the below command </br>
```kubectl argo rollouts dashboard``` </br>
  ```Public IP:3100``` (Public IP of EKS Client (Bastion HOST) and 3100 is port number of Argo Dashboard) </br>

## How to deploy 

- To keep things simple, let’s create all these objects for now in the default namespace by running the below commands: </br>
  ###### ("argo-rollout/canary/" is the name of the directory  where all my YAML filre placed) </br>
  ```kubectl apply -f argo-rollout/canary/```

- You would be able to see all the objects been created in the default namespace by running the below command </br>
  ```kubectl get all```

- Now, you can access your sample app, by URL. </br>
  ```<Domain Name or URL of laod-balancer>```

- You can see the current status of this rollout by running the below command as well </br>
  ```kubectl argo rollouts get rollout rollouts-demo```

- Now, let's deploy the Yellow version of the app using canary strategy via command line </br>
  ```kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:yellow```

- See new version(Yellow) based set of pods of our sample app, coming up </br>
  ```kubectl get pods```

Currently, only 20% i.e 1 out of 5 pods with a yellow version will come online, and then it will be paused as we have mentioned in the steps above. 


- You can confirm the same now, by running the command below, which shows, the new version is in paused state </br>
  ```kubectl argo rollouts get rollout rollouts-demo```

- Now, let's promote the yellow version of our app, by executing the below command </br>
  ```kubectl argo rollouts promote rollouts-demo```

- Run the following command and you would see it's scaling the new i.e yellow version of our app completely.</br>
  ```kubectl argo rollouts get rollout rollouts-demo```

!! you have successfully completed the Canary Deployment using Argo Rollouts.!! </br>

- Deleting the entire setup by using the below command. </br>
  ```kubectl delete -f argo-rollout/canary/```
