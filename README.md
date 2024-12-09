# SEMINAR - KUBERNETES
## Github repository:
- https://github.com/nqq203/Microservice
## Minikube: 
### 1. Pre-condition: 
- You have to install and setup minikube on your computer.
- https://k8s-docs.netlify.app/en/docs/tasks/tools/install-minikube/

### 2. Apply the following command to deploy your server in local:
- With minikbe it's just an environment for developer who wants to study about the kubernetes, so it's has own disadvantage that it provides just one node (single node) - Master Node or Control Plane.
- Before doing these following command, please ensure that you are in the root folder of this project.
```bash 
kubectl apply -f deployment-service.yml
```
- After apply the deployment-service.yml file, you had a cluster with a single node, deployments and services inside that.
- You can check the status of these by using the following command line:
```bash
kubectl get all ### to see all elements inside the cluster
kubectl get services ### to get all services
kubectl get deployments ### to get all deployments
kubectl get pods ### to get all pods
kubectl get nodes ### to get all nodes (just one node will be returned)
kubectl describe pods [POD_NAME] [flags] ### to describe the pod information, you can also do it with service and deployment.
### Example: 
kubectl describe svc frontend ## This shows you the information about the frontend service, inside this you can check the NodePort, Port, and TargetPort of this service
```

### 3. Access the website:
- There are two ways to access the website:
#### Using the minikube ip and NodePort to access:
- Get the minikube ip:
```bash
kubectl describe svc frontend # or
kubectl describe svc/frontend ### this shows you the NodePort, which accepts the client can access through inside the cluster with this port.
minikube ip ### this shows you the ip of minikube server
```
- After get the minikube ip, use minikube ip and node port to access server: <minikube-ip>:<node-port>
#### Using port-forwarding:
- Use the following command line to forward port and access via localhost or 127.0.0.1:
```bash
kubectl port-forward svc/frontend-external 8080:80 ### or
kubectl port-forward svc/frontend 8080:80
```
- Enter the the URL: localhost:8080

## Cloud Provider (AWS EKS):
### 1. Pre-conditions:
#### Step 1: Prepare for Security Group: 
- Prepare for below inbound rules inside security group:
![Security Group](https://github.com/nqq203/Microservice/blob/main/security-group.jpg)

#### Step 2: Create new User in IAM User and Prepare below policies:
![Policies To Attach](https://github.com/nqq203/Microservice/blob/main/Policies.png)
- One more policy we need to create with content as below:
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```
- Export Access Key for this user for excel file or anything else to save information.

#### Step 3: Launch new EC2 instance:
- Prepare a key pair with pem file or anything else to access instance.
- Launch an instance on AWS EC2 with minimum type is t2.large and about 25 GiB SSD (use Ubuntu Linux), select the security group you've already prepared before.

#### Step 4: Connect to EC2 instance:
- Use the key pair to connect via SSH, or you can also connect directly in that instance.

### 2. Applying these folllowing command line to deploy your server in EKS:
- Create new folder and prepare bash shell:
```bash
mkdir scripts
cd scripts
vi 1.sh
```

- Insert the following command line into the bash shell:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

- After that, approve the role for this file:
```bash
sudo chmod + 1.sh
```

- Check if it's approved and Run this bash file:
```bash
ls
./1.sh
```

- Configure the aws user:
```bash
aws config

### After that you will be required to enter some information, and you just need to enter two per three 3 parameters it requires
### Access Key ID: Enter the Access key ID
### Secret Access Key: Enter the Secret access key
### Region: You can pass this parameter
```

- Create the cluster: 
```bash
eksctl create cluster --name=<Your-EKS-Name> --region=<the-aws-region> --zones=<the-aws-zone-a>,<the-aws-zone-b> --without-nodegroup
### Example: with Sydney region, with the zone, you can choose 2 or 3 zones.
eksctl create cluster --name=EKS-1 \
                      --region=ap-southeast-2 \
                      --zones=ap-southeast-2,ap-southeast-2 \
                      --without-nodegroup
```

- Approve the oidc provider:
```bash
eksctl utils associate-iam-oidc-provider --region <the-aws-region> --cluster <Your-EKS-Name> --approve

### Example: 
eksctl utils associate-iam-oidc-provider \
    --region ap-southeast-2 \
    --cluster EKS-1 \
    --approve
```

- Create node group:
```bash
eksctl create nodegroup --cluster=<Your-EKS-Name> \
                       --region=<the-aws-region> \
                       --name=<node-name> \
                       --node-type=<node-type> \
                       --nodes=<node-amount> \
                       --nodes-min=<min-node> \
                       --nodes-max=<max-node> \
                       --node-volume-size=<volume-size> \
                       --ssh-access \
                       --ssh-public-key=<the-key-pair> \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access

### Example
eksctl create nodegroup --cluster=EKS-1 \
                       --region=ap-southeast-2 \
                       --name=node2 \
                       --node-type=t3.medium \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=devops \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

- Clone git project and apply the configuration file:
```bash
### Notes: You need to install git before
git clone https://github.com/nqq203/Microservice.git
cd Microservice
kubectl apply -f deployment-service.yml
```

- After apply, please wait for creating the deployments and service inside, you can do the following commands to check status: 
```bash
kubectl get all ### to see all elements inside the cluster
kubectl get services ### to get all services
kubectl get deployments ### to get all deployments
kubectl get pods ### to get all pods
kubectl get nodes ### to get all nodes (just one node will be returned)
kubectl describe pods [POD_NAME] [flags] ### to describe the pod information, you can also do it with service and deployment.
### Example: 
kubectl describe svc frontend-external ## This shows you the information about the frontend service, inside this you can check the NodePort, Port, and TargetPort of this service
```

- Finally, use the LoadBalancer Ingress it provides to access the website:
```bash
kubectl describe svc frontend-external ### Then it show you the information including the LoadBalancer Ingress
```

## Self-Evaluation: 
- Preparing source code: LNThai
- Setup local environment: NQQuy
- Demo local using minikube: NQQuy
- Setup EKS environment: NNNam + LNThai
- Demo with Cloud Provider with EKS: NNNam 


## References: 
(https://github.com/jaiswaladi246/Microservice.git)