# Kubernetes Cost Monitoring in New Relic using Kubecost

Operating cloud infrastructure at scale requires insight into real-time cost analysis to help continuously monitor, measure and reduce overall cloud spend. A key aspect is to then breakdown costs by Kubernetes cluster concepts (such as namespace, pod, deployment, etc) to get an accurate infrastructure spend visual. The following exercise is an effort to use open source projects (such as Kubecost, Opencost, etc) to shine light into the Kubernetes cluster spend blackbox inside New Relic.

## Kubecost vs Opencost

Kubecost, a popular solution for K8s cost monitoring, offers community(free) and paid versions of their product. Opencost is a vendor-neutral open source project for measuring and allocating infrastructure/container costs. The cost allocation engine used by Opencost was originally built by Kubecost and is a founding contributor. This OpenCost Specification link will help understand how infrastructure and container costs in Kubernetes environments are measured and allocated. I chose Kubecost for this project but you can essentially achieve the same results using Opencost.

Below is the Reference Architecture on how this solution was implemented. Multi-cloud, multi-cluster, long term storage implementations are possible using kubecost but out of scope for this exercise.

![image](https://github.com/munwarm/kubecost-newrelic/assets/32727915/1d156403-e0e4-47ec-865f-002a57d12c24)

## Setup & Configuration

### Prerequisties: 

- Existing kubernetes cluster. Managed cluster types (EKS, GKE, AKS) and Kubernetes distributions (like OpenShift, Rancher, Tanzu, etc) are supported. I chose a NodeJS application (called Knote) to run a 2-node kubernetes cluster on AWS EKS
- Existing New Relic Account

### Step 1: Install Kubecost

Running the following helm commands will install Kubecost, Prometheus, Grafana, and kube-state-metrics in the namespace supplied. Kubecost recommends installing the bundled Prometheus, that is optimized to kubecost resulting in 70-90% fewer metrics than a standalone deployment.

```
kubectl create namespace kubecost
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm install kubecost kubecost/cost-analyzer --namespace kubecost --set kubecostToken="bW1vaGFtbWVkQG5ld3JlbGljLmNvbQ==xm343yadf98"
```

### Step 2 (Optional): Enable port-forward

If you would like to view the Kubecost frontend then enable port forward on port 9090 and go to this link http://localhost:9090
```
kubectl port-forward --namespace kubecost deployment/kubecost-cost-analyzer 9090
```

### Step 3: Prometheus Remote Write integration

Get Prometheus data flowing into New Relic by updating the prometheus.yml config file to include
```
remote_write:
- url: https://metric-api.newrelic.com/prometheus/v1/write?prometheus_server=YOUR_DATA_SOURCE_NAME
  bearer_token: YOUR_LICENSE_KEY
```

### Step 4: Visualization in New Relic

Building the integration pipeline was easy but converting some of the complex prometheus queries into NRQL was quite a challenge.
A special thanks to Amine Benzaied for providing assistance on constructing these visuals inside Kubernetes Cost Monitoring dashboard.

A short video linked below describes how this dashboard breaks down total K8s cluster cost by Node, Compute, Memory, Storage & Network. Storage costs are further split by Persistent Volume and Container, going granular up to Instance and Device level metrics. 

Finally Local Storage cost, Network Egress cost and Volume Discounts template variables are used to dynamically alter these widgets and get customized views for hourly, daily, weekly or monthly time frames.

## Conclusion

Every customer using kubernetes can leverage this solution and get insight into their K8s cost - how much they spend, opportunities for optimization, etc. For New Relic, this solution helps unify cost and observability data for Finance, FinOps and Engineering teams to help understand and take control of cloud costs.

![image](https://github.com/munwarm/kubecost-newrelic/assets/32727915/65fd20ea-d231-4824-8df2-8cd4c63085d5)

