Multi Cluster deployment with ArgoCD:

we are going to deploy the application in multiple eks clusters using argocd with hub spoke model.


Prerequisites:
kubectl 
eksctl
aws cli

process:
1. Create 3 eks clusters

eksctl create cluster --name hub-cluster --region <region name>

eksctl create cluster --name spoke-cluster-1 --region <region name>

eksctl create cluster --name spoke-cluster-2 --region <region name>

2. Change context to hub-cluster where we want to install argocd.

kubectl config get-contexts - to get all the contexts

kubectl config use-context <hub cluster context>

kubectl config current-context

3. Install ArgoCD

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

check the pod status after argocd installation

kubectl get pods -n argocd

4. we need to change the api server into insecure mode

api server secure mode means https, for that we need to have self signed certificate.
for demo, http is enough

kubectl get cm -n argocd

kubectl edit configmap argocd-cmd-params-cm -n argocd

add in the last line

data:
 server.insecure: "true"
 
we will get this attributes information from http://github.com/argo-proj/argo-cd

5. we are not going to use ingress in this demo, so we will use NodePort 

kubectl get -svc -n argocd

kubect edit svc argocd-server -n argocd

change type from ClusterIP to NodePort

to know the argocd server service changed to NodePort, execute below command

kubectl get svc -n argocd

6. we have 2 ec2 instances for hub cluster, as we pointed to NodePort we can get access from any of the IP of two ec2 instances.

Get IP from ec2 and port from argocd server and enter into browser

enable all traffic in ec2 instances, as this is for demo.

Then we will get argocd console.

To login, we need secret

kubectl get secrets -n argocd

kubectl edit secret argocd-initial-admin-secret -n argocd

Get the secret and decode

echo <secret> | base64 --decode

enter username as admin and password we got


7. Add clusters to argocd

we cant add clusters using console, but we can delete the clusters.

we need argocd cli to add clusters into argocd.

install argocd cli

curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

argocd login <argocd ip:port>

argocd cluster add <context of spoke clusters> --server <argocdip:port>

8. Add application in argocd by following steps

a. application name
b. path
c. cluster url
d. namespace (default)

Need to add same app for cluster 2 as well with the same steps.

Now if you do the changes in infrastructure code in the github it will effect into the cluster and update that we can see.

If we do the changes manually into the cluster it will say out of sync, we can check into auto self heal means it will revert the manual changes.



 
 
 

















