# Senior DevOps Engineer - technical interview

## Templates, scripts and configs

- `template/roles_securityGroups.yaml` CF template creates the 2 IAM roles, master and worker role. Master role is used for the creation of EKS cluster and the worker role is used for the creation of worker nodes. It also creates the security group allowing full access to the kubernetes worker nodes.

- `template/cluster.yaml` CF template creates the EKS cluster. The name of the cluster is defined in the `config.json`.

- `template/nodeGroup.yaml` CF template creates the node group. It has the desized size to 1, so it will create one node and gets attached to the EKS cluster. Node group name comes from `config.json` file.

- `kubernetes/aws-auth-cm.yml` This lets the IAM role that was created access the cluster. After running `kubectl apply -f kubernetes/aws-auth-cm.yml` the node(s) join the cluster.

- `kubernetes/deployment.yaml` Definition file of kind deployment, deploys the docker image that containes hello.py.

- `Dockerfile` Use the official python3 image, installs flask and runs hello.py script. The image is built already and pushed to docker hub. `docker pull udhaya/flask:one`

- `Makefile`
    - `make iam-sg` Creates the cloud formation stack for iam roles and security group.
    - `make createCluster` Creates the cloud formation stack for EKS cluster.
    - `make applyConfigMap` kubectl apply aws auth configMap for the nodes to join the cluster.
    - `make CreateNodeGroup` Creates the node group for the EKS cluster.
    - `make deploy` Deploys the image into the pod.

- `config.json` Configuration file

### Prerequisites
As of 16th Aug 2022,
-  The template will not configure aws cli, will not create vpc, subnets and key pair. Make sure you have run `aws configure` where you running the cli.
-  `config.json` Add your vpc, subnets as "subnet1, subnet2" and keypair
-  `aws-auth-cm.yml` replace rolearn in line 8 to  `arn:aws:iam::<accountid>:role/<iam_role_workernode_that_we_created_in_the_roles_securityGroups.yaml_stack> `

### Deployment steps
-   Clone the repo
-   Update VPC, subnets and keypair in config.json
-   Run the following make commands
    `make iam-sg`
    `make createCluster`
-   Update aws-auth-cm.yml file with the worker node IAM role
-   Run the following make commands
    `make applyConfigMap`
    `make CreateNodeGroup`
    `make deploy`

### NOTE:
If you are running the scripts behind a proxy, you will face an issue with docker pulling the image `udhaya/flask:one` from docker hub. If so, please update the proxy in the node under the file `/etc/systemd/system/docker.service.d/00proxy.conf`
