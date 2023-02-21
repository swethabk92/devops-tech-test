# Senior DevOps Engineer - technical interview

## Prerequisites
-  These templates will not configure aws cli tool, will not create vpc's, subnets or key pairs. Make sure you have run `aws configure` when you are running the cli.
-  `config.json` will Add your vpc, subnets as "subnet1, subnet2" and keypair
-  `aws-auth-cm.yml` replace "rolearn" in line no. 8 to  `arn:aws:iam::<accountid>:role/<iam_role_workernode_that_we_created_in_the_roles_securityGroups.yaml_stack> `

## Templates, scripts and configs

- `template/securityGroups.yaml` CloudFunction template creates the 2 IAM roles, master and worker role. Master role is used for the creation of EKS cluster and the worker role is used for the creation of worker nodes. It also creates the security group allowing full access to the kubernetes worker nodes.

- `template/eks_cluster.yaml` CloudFunction template creates the EKS cluster. The name of the cluster is defined in the `config.json`.

- `template/eks_nodeGroup.yaml` CloudFunction template creates the node group. It has the desized size to 1, so it will create one node and gets attached to the EKS cluster. Node group name comes from `config.json` file.

- `kubernetes/aws-auth-cm.yml` This lets the IAM role that was created to access the cluster. After running `kubectl apply -f kubernetes/aws-auth-cm.yml` the node(s) join the cluster.

- `kubernetes/deployment.yaml` Definition file of kind deployment, deploys the docker image that containes hello.py.

- `Dockerfile` Use the official python-3.6 image, installs flask and runs hello.py script. The image is built already and pushed to docker hub. `docker pull swethabk92/seniordevopsengineer-flask:1.0.0`

## Basic Architechture of the Setup
![alt text](./Architecture.png?raw=true "Architecture")

- `Makefile`
    - `make iam-securitygroup` Creates the cloud formation stack for iam roles and security group.
    - `make createCluster` Creates the cloud formation stack for EKS cluster.
    - `make applyConfigMap` kubectl apply aws auth configMap for the nodes to join the cluster.
    - `make CreateNodeGroup` Creates the node group for the EKS cluster.
    - `make deploy` Deploys the image into the pod.

- `config.json` Configuration file

### Manual Deployment steps
-   Clone the repo
-   Update VPC, subnets and keypair using the config.json
-   Run the following make commands
    `make iam-securitygroup`
    `make createCluster`
-   Update aws-auth-cm.yml file with the worker node IAM role
-   Run the following make commands
    `make applyConfigMap`
    `make CreateNodeGroup`
    `make deploy`

### Jenkins Automated Deployment steps
- Create a pipepline job in jenkins instance which has connectivity to our AWS account - Jenkins hosted in AWS EC2 instance is preferred for compatibility
- Select the scm and give this github url and save.
- Run the job and it will run:  `Clone the code --> Sonar analysis on the code --> Build the Docker Image --> Push to Dockerhub --> Create EKS Cluster in AWS --> Deploy to Kubernetes`

## Branching strategy needs to be followed to get standardized development of any application
- Branching strategy procedure is documented and uploaded in the same directory `./Git_Branching_Strategy.docx`

### NOTE:
Running the scripts behind a proxy throws an error while pulling the image `swethabk92/seniordevopsengineer-flask:1.0.0` from docker hub.
Solution: Update the proxy in the node under the file `/etc/systemd/system/docker.service.d/00proxy.conf`
