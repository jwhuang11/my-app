## General Info
In this project, there are 3 backend services in Go. 
* discount
* product
* user

We assume that this project has 3 environments & branches:
* dev -> development
* stg -> staging
* prd -> production

The dockerfile & kubernetes manifest file are inside each of the service folder.
```shell
/cmd/discount
/cmd/product
/cmd/user
```

## Technology:
1. GitHub Action
2. AWS CodePipeline & CodeBuild
3. AWS ECR (Elastic Container Registry)
4. Helm Chart
5. Docker
6. AWS EKS (Elastic Kubernetes Service)
7. Terraform (The terraform script is not provided in this repo, assuming that it is centralized in infrastructure repo)

## CI/CD Flow
```shell
GitHub Action -> AWS CodePipeline -> AWS CodeBuild -> AWS EKS Cluster
```
#### Github Action
1. Any push to one of the 3 branches, will trigger Github Action. 
2. Github Action run based on the yml file bellow
```shell
/.github/workflows/trigger-aws-codepipeline.yml
```
3. It will read the file changes and adust to the condition. 
4. If common files changed ( internal/**, go.mod, go.sum), then all service codepipeline will be triggered
5. If only service files changed (e.g. cmd/discount/**, discount/**, buildspec/buildspec_discount_*.yml), then only the related service codepipeline will be triggered.

#### Build Process 
1. When a CodePipeline Triggered, it will then continue to CodeBuild. CodeBuild run the build command from the /buildspec/buildspec_service.yml
2. The service will be build by reading the Dockerfile : docker build --tag $REPOSITORY:latest .
3. The docker image will be pushed to AWS ECR : docker push $REPOSITORY:$TAG

#### Deploy Process
1. The docker image value will be added to helm values file : sed -ie "s@<IMAGE>@$IMAGE_URI@g" manifest/values/$STAGE.yaml
2. Deployment to AWS EKS cluster by using helm : helm upgrade -i discount-app-dev -f manifest/values/$STAGE.yaml
  
#### Expose Service
1. The service will be exposed by using AWS Application Load Balancer defined from ingress configuration file.

