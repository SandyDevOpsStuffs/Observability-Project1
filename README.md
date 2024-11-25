# Observability-Project1

This is an open-source demo project by OpenTelemetry. This project deploys a multi microservice e-commerce web application on an AWS EKS cluster. Some of the services are cartservice, checkoutservice, currencyservice, emailservice, flagd, frontend, frontendproxy, prometheus, grafana, loadgenerator, otelcollector, kafka, paymentservice, productcatalogservice, recommendationservice, shippingservice, etc. Each service is written in different languages such as C++, .Net, Java, Python, Go, Ruby, etc.

You will get the architecture of this project in the official documenation of OpenTelemtry:
https://opentelemetry.io/docs/demo/kubernetes-deployment/

You will get the source code and all other files in the GitHub repo of OpenTelemetry:
https://github.com/open-telemetry/opentelemetry-demo.git

To get metrics, logs and traces each micro service is instrumenated. To collect, store, expose, analyze and visualize these telemetry data the observability and monitoring tools such as Promethues, Grafana, Jaeger, etc are used.


## Steps to Implement this Project

### Step 1: Install AWS CLI

#### Update the package manager and install prerequisites:

sudo apt update

sudo apt install -y unzip curl

#### Download the AWS CLI v2 installer:

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

#### Unzip the installer:

unzip awscliv2.zip

sudo ./aws/install

#### Verify the installation:

aws --version

#### Configure the AWS CLI by providing access key and secret key of your IAM user.

aws configure

#### Verify the configuration: Test if the AWS CLI is working by running a simple command:

aws s3 ls

#### Add the following three policies to your IAM user.

AmazonEKSClusterPolicy
AmazonEKSVPCResourceController
AWSCloudFormationFullAccess

### Step 2: Install and configure eksctl

#### Create an EKS cluster: 

eksctl create cluster --name=observability \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --without-nodegroup

#### Associate an OIDC provider with your cluster:

	eksctl utils associate-iam-oidc-provider \
  				--region=ap-south-1 \
  				--cluster=observability \
  				--approve

#### Add the recommended policies for the vpc-cni addon to the cluster config file, then update the addon:

eksctl update addon --name vpc-cni --cluster observability --region ap-south-1

#### Create a node group:

eksctl create nodegroup --cluster=observability \
                        --region=ap-south-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

#### Update ./kube/config file:

aws eks update-kubeconfig --name observability

#### Change or add the context so that we can use the cluster observability now onwards:

aws eks update-kubeconfig --name observability

eksctl get clusters

eksctl get nodegroup

eksctl get nodegroup --cluster observability

kubectl get nodes

kubectl get pods

kubectl get ns

### Step 3: Install Helm

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

ls -la

chmod +x get_helm.sh

helm version

### Step 4: Install OpenTelemetry using Helm (recommended)

#### Add OpenTelemetry Helm repository:

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

#### To install the chart or application with the release name my-otel-demo, run the following command:

helm install my-otel-demo open-telemetry/opentelemetry-demo

kubectl get pods

### Step 5: Expose and access the recently installed e-commerce application

#### Use the following command if you have created your K8s cluster on your local machine (laptop) and installed/deployed the application.

kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080

#### Use the following command if you have created your K8s cluster on an EC2 instance or Azure VM and installed/deployed the application.

kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080 --address 0.0.0.0

#### Now access the application by pasting the following URL in the browser:

<Public IP of my EC2 instance>:8080

Perform some activities such as adding some products to the cart, checkout for payment, etc. Even though we do not perform these activities we will get some metrics and traces since this application has a fake load generator. But if we want to see our own request metrics and own request traces then perform some activities such as add some items to cart, change currency, click on other multiple micro services, etc at least.

### Step 6: Access Jaeger

Let us access Jaeger UI to see the traces.

Jager is already installed using the helm chart that we used recently to install the application. We can see the pod for Jaeger when we ran the command kubectl get pods. The demo application has only Jaeger, Prometheus and Grafana. If we want Datadog then we need to go to official documentation of Datadog and learn how to install Datadog using a helm chart.

First let us access Jaeger UI by pasting the following URL in the browser. This URL is provided by the official documentation of OpenTelemetry since this demo application has already included Jaeger, Prometheus and Grafana.

If you are doing project on local machine (laptop):
http://localhost:8080/jaeger/ui/

If you are doing project on EC2 instance:
http://<Public IP of EC2 instance>:8080/jaeger/ui/

Now select the various services such as ‘checkoutservice’, 'cartservice' etc and try to see the traces of the last 5 minutes, 10 minutes, 30 minutes, 60 minutes, etc.

### Step 7: Access Grafana

If you are doing project on EC2 instance:

http://<Public IP of EC2 instance>:8080/grafana/

No password is required. Authentication is disabled.

If I go to the Dashboards tab / option, I have a dashboard called ‘Demo Dashboard’. I get RED metrics of frontend and Application metrics if I click on Demo Dashboard.

If we want to create our own dashboard then we can do so by clicking on ‘New’, ‘Add Visualization’, ‘Prometheus’ as Data Source, toggle from ‘Builder’ to ‘Code’, etc.

This prometheus instance does scrape metrics of nodes or K8s cluster. If you want you can check it by trying to type node or pod in PromQL query field and we do not get anything related to node or pod. Why? Because to scrape metrics from nodes, K8s cluster or databases, the Prometheus needs Exporters. If we run kubectl get pods command then we will not see pods related to ‘node exporter’ and ‘kube-state-metrics’. That is why the Prometheus can not get any information about underlying nodes or K8s cluster. If we need those metrics then we need to install node exporter and kube-state-metrics. We will install them shortly. Before that let us run a PromQL query http_server_duration_seconds_bucket. Change the time to last 5 minutes and look ate the metrics of the service ‘flagd’.

Now let us run the following query and see the count of recommendations for the service ‘recommendations’ for last 1 hour:

app_recommendations_counter_total

### Step 8 (Optional): Add the Prometheus Community Helm Chart Repository

The kube-state-metrics and node-exporter charts are hosted in the Prometheus community repository.

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

#### Install kube-state-metrics

kube-state-metrics is a service that generates metrics about the state of Kubernetes objects

Since the namespace kube-system is active and typically used for cluster monitoring tools, we'll use it for these deployments. No need to run the following command:

helm install kube-state-metrics prometheus-community/kube-state-metrics \
  --namespace kube-system \
  --create-namespace

Let us directly run the following command now:

helm install kube-state-metrics prometheus-community/kube-state-metrics \
  --namespace kube-system

### Step 9 (Optional): Install Node Exporter

#### Run the following command to install node-exporter:

helm install node-exporter prometheus-community/prometheus-node-exporter \
  --namespace kube-system

#### Verify Deployments: Check the pods in the kube-system namespace to ensure the deployments are successful:

kubectl get pods -n kube-system

kubectl get svc --namespace kube-system

### Step 10 (Optional): Access Metrics

These components will automatically expose their metrics within the cluster. If you want to verify them:

Port-forward to view metrics locally:

First let us change the service type from ClusterIP to LoadBalancer.

kubectl edit svc kube-state-metrics -n kube-system

kubectl edit svc node-exporter-prometheus-node-exporter -n kube-system

For kube-state-metrics:

kubectl port-forward -n kube-system svc/kube-state-metrics 8080:8080

Access metrics at: http://localhost:8080/metrics
OR
Access metrics at: http://<Public IP of EC2 instance>:8080/metrics
OR
Access metrics at: http://<Public IP of EC2 instance>:8080

#### Add an inbound rule 9100 to the security group of your EC2 instance:

For node-exporter

kubectl port-forward -n kube-system svc/node-exporter-prometheus-node-exporter 9100:9100

Access metrics at: http://localhost:9100/metrics
OR
Access metrics at: http://<Public IP of EC2 instance>:9100/metrics
OR
Access metrics at: http://<Public IP of EC2 instance>:9100

