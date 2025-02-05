# Evaluation

### Compatibility and Feasibility

|  | AWS | E2E |
| --- | --- | --- |
| VPC |  | - VPC CIDR is not configurable  - VPC creation costs credits  - Subnet creation is not configurable |
| Compute Node |  | - Smallest instance is 4vCPU, 8GB RAM  - By default login is via password, SSH access can be setup by creating SSH keys and adding via console |
| Kubernetes |  | - Is available in the Delhi/NCR region only (for now)  - Kubernetes gets exposed as a public endpoint  - Kubeconfig is downloaded from the console  - Smallest instance is 4vCPU, 8GB RAM  - LoadBalancers are created via MetalLB and hence need to be reserved first before they can be used in K8S for LB IPs.  - For ingress, installing the nginx ingress controller is required. |
| RDS |  | - Creates a private instance (to which a Public IP can be attached for internet access)  - Smallest instance is 4vCPU/16GB |
| Security |  | - Using API-key and Auth-token authenticated to the E2E cloud. |
| Reserved IP |  | - Need to created and reserved first before it can be attached to a Compute Node or LoadBalancer |
| IAM |  | - New users need to be assigned specific E2E resource permission before they can access the resource via console |

### CI/CD Architecture

**Continuous Integration (CI) with GitHub Actions**

GitHub Actions is used for CI in this architecture. It allows for automating workflows directly within GitHub repository. When changes are pushed to the repository, GitHub Actions triggers predefined workflows to build a docker image and deploy image in cloud registry. These actions running on ubuntu runner.

**Workflow Steps:**

1. **Trigger**: GitHub Actions triggers when changes are pushed to the repository.
2. **Checkout**: The repository is checked out to the runner environment.
3. **Build**: The application code is built using docker build.
4. **Push:** The docker image is push to cloud registry.

**Continuous Deployment (CD) with Argo CD**

Argo CD continuously monitors the Git repository for changes and automatically deploys the application to the Kubernetes cluster when changes are detected.

**Deployment Process:**

1. **Git Repository**: Argo CD watches the Git repository where the application manifests are stored.
2. **Change Detection**: Argo CD detects changes in the Git repository, such as new commits.
3. **Updates Values.yaml:** Updates a value.yaml in helm manifests i.e Image-tag
4. **Git Commit:** Then commit a values file and push to repository.
- Reference Images:
    
    ![Untitled](assets/Untitled-git1.png)
    
    ![Untitled](assets/Untitled.png)
    
    ![Untitled](assets/Untitled-argo1.png)
    
    ![Untitled](assets/Untitled-argo2.png)
    

### Cost Analysis

|  | AWS | E2E |
| --- | --- | --- |
| Reserved IP |  | - ₹ 0.26/Hourly ($0.003) <br> - ₹ 200/Monthly ($2.40) |
| Compute Nodes | Region - Mumbai  <br> T2.Micro - 1 vCPUs, 1.0GiB <br> $0.0124 hourly -  <br> $9.0520 monthly- <br> A1 Extra Large - 8.0 GiB, 4vCPU <br> -$0.1020 hourly -  <br> -$74.4600 monthly   | For 4vVPU, 8GB RAM <br> - ₹ 3.1/Hour ($0.037) <br> - ₹ 2263/Monthly ($27) |
| Kubernetes | Region - Mumbai <br> T2.Small - 1 vCPUs 2.0 GiB - <br> $0.0248 hourly - <br> $18.1040 monthly  <br> A1 Extra Large - 8.0 GiB, 4vCPU  <br> - $0.1020 hourly - <br> $74.4600 monthly | For 4vVPU, 8GB RAM <br> - ₹ 3.1/Hour ($0.037) <br> - ₹ 2263/Monthly ($27) |
| RDS | Region - Mumbai <br> T3.Micro - 2 vCPUs, 1 GiB <br> $0.0260 hourly -  <br> $18.9800 monthly <br> T3 Extra Large - 4vCPUs, 16GiB <br> - $0.4240 hourly <br> $309.5200 monthly  | For 4vCPU, 16GB RAM <br> - ₹ 9/Hour ($0.11) <br> - ₹ 6570/Monthly ($78.58) |
| VPC |  | - ₹ 4.8/Hour ($0.057) <br> - ₹ 3504/Monthly ($41.91) |

### Security Assessment
For the security E2E cloud have Auth token and API token which you can generate on console.
By passing this configure a terraform provider and also E2E CLI.

### Performance Benchmarking

**Yandex Performance Testing**

In both the cloud me perform a performance testing using Yandex-tank, below are the loads we are tested. Both the cloud have break on request count above 311.

- **E2E cloud**
    1. Test: const(100,5m)
    
    ![Untitled](assets/Untitled-tes1.png)
    
    2. Test: const(250,5m)
    
    ![Untitled](assets/Untitled-test2.png)
    
    3. Test: const(500,5m)
    
    ![Untitled](assets/Untitled-test3.png)
    
- **AWS cloud**
    1. Test: const(100,5m)
    
    ![Untitled](assets/Untitled-t1.png)
    
    2. Test:  const(250,5m)
    
    ![Untitled](assets/Untitled-t2.png)
    
    3. Test: const(500,5m)
    
    ![Untitled](assets/Untitled-t3.png)
    

### Implementation Roadmap

### **In E2E cloud.**

Setup the infra in E2E cloud we used terraform for VPC and Kubernetes cluster ( The code is in terraform/e2e folder), for database and container registry used console.

- E2E cloud resources images.
    
    ![Untitled](assets/Untitled-e2e-1.png)
    
    ![Untitled](assets/Untitled-e2e-2.png)
    
    ![Untitled](assets/Untitled-e2e-3.png)
    
    ![Untitled](assets/Untitled-e2e-4.png)
    
- App deployments in K8 cluster.
    
    ![Untitled](assets/Untitled-app1.png)
    
    Host: [**http://e2e.blacksilky.com/api/v1/student**](http://e2e.blacksilky.com/api/v1/student)
    
    ![Untitled](assets/Untitled-app2.png)
    
- Below are some points we observe while setup the infra in E2E cloud
    1. Resources creating using terraform take long time to up.
        
        ![Untitled](assets/Untitled-cons-1.png)
        
    2. There is no terraform resources for RDS and reserve a LIP. We have try on CLI for creating database getting an error of password setup policy.
        
        ![Untitled](assets/Untitled-cons-2.png)
        
    3. For the nginx-ingress implementation need to reserved IP and attached to LIP.
        
        ![Untitled](assets/Untitled-cons-3.png)
        
    4. Admin user will create a resources using terraform/CLI. 
    

### **In AWS**

Setup the infra in AWS using terraform(The code is in terraform/aws).

- AWS cloud resources images.
    
    ![Untitled](assets/Untitled-aws1.png)
    
    ![Untitled](assets/Untitled-aws2.png)
    
    ![Untitled](assets/Untitled-aws3.png)
    
    ![Untitled](assets/Untitled-aws4.png)
    
- App deployments in k8 cluster.
    
    ![Untitled](assets/Untitled-aws5.png)
    
    ![Untitled](assets/Untitled-aws6.png)
    
    Host: [**http://aws.blacksilky.com/api/v1/student**](http://e2e.blacksilky.com/api/v1/student)
    
    ![Untitled](assets/Untitled-aws7.png)
