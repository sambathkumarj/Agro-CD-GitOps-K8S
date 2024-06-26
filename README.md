# Agro-CD-GitOps-K8S

![Gitops](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/aed034cc-d55d-47c4-a9a4-39ce59726338)


# Argo CD - declarative  GitOps of CD for Kubernetes 

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

Application definitions, configurations, and environments should be declarative and version-controlled. Application deployment and lifecycle management should be automated, auditable, and easy to understand.

# Core Concepts

Let's assume you're familiar with core Git, Docker, Kubernetes, Continuous Delivery, and GitOps concepts. Below are some of the concepts that are specific to Argo CD.
      
• Application A group of Kubernetes resources as defined by a manifest. This is a Custom Resource Definition (CRD).
      
• Application source type Which Tool is used to build the application.
      
• Target state The desired state of an application, as represented by files in a Git repository. 
      
• Live state The live state of that application. What pods etc are deployed.


# Requirements

• Installed kubectlcommand-line tool.
      
• Have a kubeconfig file (default location is ~/.kube/config).
      
• CoreDNS. Can be enabled for microk8s by microk8s enable dns && microk8s stop && microk8s start

# 1. Install Argo CD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This will create a new namespace, argocd, where Argo CD services and application resources will live.

# 2. Download Argo CD CLI

Download the latest Argo CD version from https://github.com/argoproj/argo-cd/releases/latest. More detailed installation instructions can be found via the CLI installation documentation.
Also available in Mac, Linux and WSL Homebrew:
```
brew install argocd 
```
or 

```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

# 3. Access The Argo CD API Server

By default, the Argo CD API server is not exposed with an external IP. To access the API server, choose one of the following techniques to expose the Argo CD API server:
Service Type Load Balancer :
Change the argocd-server service type to LoadBalancer:
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

# Port Forwarding
```
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
The API server can then be accessed using https://localhost:8080

![Argo-ui](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/4d6f0605-9158-4501-9ad9-cc400cae6661)


# 4. Login Using The CLI

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using the argocd CLI:
```
argocd admin initial-password -n argocd
argocd login <ARGOCD_SERVER>
```

# Note
The CLI environment must be able to communicate with the Argo CD API server. If it isn't directly accessible as described above in step 3, you can tell the CLI to access it using port forwarding through one of these mechanisms: 1) add --port-forward-namespace argocd flag to every CLI command; or 2) set ARGOCD_OPTS environment variable: export ARGOCD_OPTS='--port-forward-namespace argocd'.
Change the password using the command:
```
argocd account update-password
```

# 5. Register A Cluster To Deploy Apps To (Optional)

This step registers a cluster's credentials to Argo CD, and is only necessary when deploying to an external cluster. When deploying internally (to the same cluster that Argo CD is running in), https://kubernetes.default.svc should be used as the application's K8s API server address.
First list all clusters contexts in your current kubeconfig:
```
kubectl config get-contexts -o name
edgex@edgex-VirtualBox:~$ kubectl config get-contexts -o name
microk8s
```

Choose a context name from the list and supply it to argocd cluster add CONTEXTNAME. For example, for docker-desktop context, run:
```
argocd cluster add <docker-desktop>
argocd cluster add microk8s
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `microk8s` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0004] ServiceAccount "argocd-manager" already exists in the namespace "kube-system" 
INFO[0004] ClusterRole "argocd-manager-role" updated    
INFO[0004] ClusterRoleBinding "argocd-manager-role-binding" updated 
```

The above command installs a ServiceAccount (argocd-manager), into the kube-system namespace of that kubectl context, and binds the service account to an admin-level ClusterRole. Argo CD uses this service account token to perform its management tasks (i.e. deployment/monitoring)

# 6. Create An Application From A Git Repository
An example repository containing a guestbook application is available at https://github.com/argoproj/argocd-example-apps.git to demonstrate how Argo CD works.
Creating Apps Via CLI
First we need to set the current namespace to argocd by running the following command:
```
kubectl config set-context --current –namespace=argocd
Context "microk8s" modified.
```

Create the example guestbook application with the following command:

![Screenshot from 2024-04-24 18-01-59](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/9832bd27-6164-4058-972c-6e35b09f6d85)


```
argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
application 'guestbook' created
```
![Argo-app](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/8e496b86-9185-443f-98ae-867b7f2e4412)

```
argocd app get guestbook
Name:               argocd/guestbook
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://10.152.183.155/applications/guestbook
Repo:               https://github.com/argoproj/argocd-example-apps.git
Target:             
Path:               guestbook
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (d7927a2)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH   HOOK  MESSAGE
       Service     default    guestbook-ui  Synced  Healthy        service/guestbook-ui created
apps   Deployment  default    guestbook-ui  Synced  Healthy        deployment.apps/guestbook-ui created
```
![Argo-health-status](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/7a19ee48-8b6c-476c-b6be-882d379f0ab1)

![guest-ui](https://github.com/sambathkumarj/Agro-CD-GitOps-K8S/assets/42794636/8752c379-2dce-4641-859f-692d80c9c7f6)



