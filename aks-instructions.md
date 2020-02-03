### Prerequisites
This install instruction is require a running AKS service that are set up and configured to use Azure [Application Gateway Ingress Controller](https://github.com/Azure/application-gateway-kubernetes-ingress). 

### Step 1: Wazuh namespace and StorageClass

The Wazuh namespace is used to handle all the Kubernetes elements (services, deployments, pods) necessary for Wazuh. In addition, you must create a StorageClass to use AWS EBS storage in our StateFulSet applications.
```BASH
$ kubectl apply -f base/wazuh-ns.yaml
$ kubectl apply -f base/aks-storage-fakeaws-class.yaml
```
If you have set up the AGIC to watch a specific namespace you will need to upgrade the helm chart and add the 'wazuh' namespace if you want to allow ingress through AGIC.

### Step 2: Deploy Elasticsearch

Elasticsearch deployment for cluster. There is a few changes from the AWS configuration. Since this install uses AGIC instead of creating a public load balancer I've added an annotation to elasticsearch-api-svc.yaml to create an ILB in Azure with 'service.beta.kubernetes.io/azure-load-balancer-internal: "true"'

```BASH
$ kubectl apply -f elastic_stack/elasticsearch/cluster/aks/elasticsearch-api-svc.yaml
$ kubectl apply -f elastic_stack/elasticsearch/cluster/aks/elasticsearch-data-sts.yaml
$ kubectl apply -f elastic_stack/elasticsearch/cluster/aks/elasticsearch-master-sts.yaml
```

### Step 3: Deploy Kibana and Nginx

Kibana and Nginx deployment.

In case you need to provide a domain name, update the `domainName` annotation value in the [nginx-svc.yaml](elastic_stack/kibana/nginx-svc.yaml) file before deploying that service. You should also set a valid AWS ACM certificate ARN in the [nginx-svc.yaml](elastic_stack/kibana/nginx-svc.yaml) for the `service.beta.kubernetes.io/aws-load-balancer-ssl-cert` annotation. That certificate should match with the `domainName`.

```BASH
$ kubectl apply -f elastic_stack/kibana/aks/kibana-svc.yaml
kubectl apply -f elastic_stack/kibana/aks/nginx-svc.yaml

$ kubectl apply -f elastic_stack/kibana/aks/kibana-deploy.yaml
$ kubectl apply -f elastic_stack/kibana/aks/nginx-deploy.yaml
```
kubectl delete -f elastic_stack/kibana/aks/kibana-svc.yaml
kubectl delete -f elastic_stack/kibana/aks/nginx-svc.yaml


### Step 5: Deploy Wazuh

Wazuh cluster deployment.

In case you need to provide a domain name, update the `domainName` annotation value in both the [wazuh-master-svc.yaml](wazuh_managers/wazuh-master-svc.yaml) and the [wazuh-workers-svc.yaml](wazuh_managers/wazuh-workers-svc.yaml) files before deploying those services. You should also set a valid AWS ACM certificate ARN in the [wazuh-master-svc.yaml](wazuh_managers/wazuh-master-svc.yaml) for the `service.beta.kubernetes.io/aws-load-balancer-ssl-cert` annotation. That certificate should match with the `domainName`.

```BASH
$ kubectl apply -f wazuh_managers/aks/wazuh-master-svc.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-cluster-svc.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-workers-svc.yaml

$ kubectl apply -f wazuh_managers/aks/wazuh-master-conf.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-worker-0-conf.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-worker-1-conf.yaml

$ kubectl apply -f wazuh_managers/aks/wazuh-master-sts.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-worker-0-sts.yaml
$ kubectl apply -f wazuh_managers/aks/wazuh-worker-1-sts.yaml
```