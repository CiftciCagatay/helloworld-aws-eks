# AWS EKS Infrastructure Code
One can tear up/down necessary kubernetes infrastructure for http web application by using these steps.

---

# AWS EKS Cluster Installation
```
cd cluster

terraform init

terraform apply -auto-approve

export KUBECONFIG=$(pwd)/kubeconfig_my-cluster
```

```
# Result
kubectl get node
NAME                                       STATUS   ROLES    AGE     VERSION
ip-10-0-3-180.eu-west-1.compute.internal   Ready    <none>   2m45s   v1.17.11-eks-cfdc40
````

# ArgoCD Installation
```
cd ..
kubectl apply -f argocd/namespace.yaml
kubectl apply -f argocd/install.yaml -n argocd
```

# Istio Installation
```
istioctl install
```

# Istio Prometheus Addon Installation
```
# https://istio.io/latest/docs/ops/integrations/prometheus/
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/prometheus.yaml
```

```
# Result
kubectl get po -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-ingressgateway-6f59657d45-vswl6   1/1     Running   0          117s
istiod-7744bc8dd7-m5mn5                 1/1     Running   0          2m8s
prometheus-788c945c9c-w2drt             2/2     Running   0          27s
```

# Kube State Metrics for HPA with Istio
```
kubectl apply -f istio-hpa/kube-metrics-adapter/
```

```
# Result
kubectl get po -n kube-system
...
kube-metrics-adapter-77774fb48b-6q6h6   1/1     Running   0          23s
...
```

# ArgoCD Create Application for WebApp
```
kubectl apply -f argocd/manifests/
````

# Istio Custom Metrics for HPA
```
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta1",
  "resources": [
    {
      "name": "pods/istio-requests-total",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    }
  ]
}

# HPA Results
kubectl get po -n webapp
NAME                      READY   STATUS    RESTARTS   AGE
webapp-54bbcb756f-n65sc   2/2     Running   0          13s
webapp-54bbcb756f-rcmjw   2/2     Running   0          13m
webapp-54bbcb756f-t6d6z   2/2     Running   0          13s
webapp-54bbcb756f-tb74k   2/2     Running   0          13s

kubectl get hpa -n webapp
NAME                     REFERENCE           TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
custom-metric-hpa-test   Deployment/webapp   4777m/10m   1         10        4          13m
```

# Tearing down cluster
```
kubectl delete -f argocd/manifests/webapp.yml
kubectl delete ns webapp
cd cluster
terraform destroy -auto-approve
```