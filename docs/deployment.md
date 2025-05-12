# Continuous Deployment - CD

Deploying the EasyShop E-Commerce Application through Argo CD

> [!CAUTION]
>
> - You must be inside you Bastion Server
> - If you haven't configured AWS Credentials, do configure it
> ```bash
> aws configure
> ```
> - EKS Cluster should be configured
> ```bash
> aws eks update-kubeconfig --region us-east-2 --name easyshop-cluster
> ```
> - Already, done these steps. You are ready to go.

## Setting up Argo CD

0. Create a Namespace for Argo CD<br/>
```bash
kubectl create namespace argocd
```
1.  Install Argo CD using Manifest
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2.  Watch Pod Creation
```bash
watch kubectl get pods -n argocd
```
3.  This helps monitor when all Argo CD pods are up and running.<br/>

4.  Check Argo CD Services
```bash
kubectl get svc -n argocd
```

5.  Change Argo CD Server Service to NodePort
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

6.  Access Argo CD GUI<br/>
Check Argo CD Server Port (again, post NodePort change)<br/>
```bash
kubectl get svc -n argocd
```

7.  Port Forward to Access Argo CD in Browser<br/>
 Forward Argo CD service to access the GUI:
```bash
kubectl port-forward svc/argocd-server -n argocd <your-port>:443 --address=0.0.0.0 &
```

8.  Replace <your-port> with a local port of your choice (e.g., 8443 port already opened in server).<br/>
 Now, open https://<bastion-ip>:<your-port> in your browser.


9. Get the Argo CD Admin Password<br/>
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

10. Log in to the Argo CD GUI
* Username: admin
* Password: (Use the decoded password from the previous command)

11. Update Your Password
* On the left panel of Argo CD GUI, click on "User Info"
* Select Update Password and change it.

11. Check ArgoCD CLI
* Make sure you are at the Bastion Server
* Check if you have argocd cli installed
```bash
argocd --version
```

12. Login to ArgoCD through CLI
```bash
argocd login http://IP-ADDRESS:8443/ --username admin --password `password`
```

> [!NOTE]
>
> You can try with both New Password and the old one, one of them will surely work.

13. Get KubeConfig
```bash
kubectl config get-contexts
```
See where this `*` get their address it should be something like this
```
arn:aws:eks:us-east-2:ACCOUNT_ID:cluster/easyshop-cluster
```

14. Add the Kubernetes Cluster
```bash
argocd cluster add arn:aws:eks:us-east-2:ACCOUNT_ID:cluster/easyshop-cluster --name easyshop-cluster
```

15. Your Kubernetes Cluster will be added
* You can see it when you go to the ArgoCD Dashboard

## Deploying Your Application

 1. On the Argo CD homepage, click on the “New App” button.<br/>

 2. Fill in the following details:<br/>
  -  **Application Name:**
    `Enter your desired app name`
  -  **Project Name:**
Select `default` from the dropdown.
    * **Sync Policy:**
 Choose `Automatic`.
    * **Automatic Namespace Create**

3. In the `Source` section:
 - **Repo URL:**
Add the Git repository URL that contains your Kubernetes manifests.
  - **Path:** 
 `Kubernetes` (or the actual path inside the repo where your manifests reside)

4. In the “Destination” section:
 -  **Cluster URL:**
    add the cluster we just added with its name

5. Click on “Create”.