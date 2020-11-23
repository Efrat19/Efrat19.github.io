---
layout: post
title: HPA Custom Metrics
# slug: cvbnjklbjm,.
lang: en
categories:
    - Kubernetes
    - monitoring
    - HPA
# tags:
#     - hoge
#     - foo
permalink: /:slug 
image:
 path: /assets/designer/tinified_meta.png
 width: 1200
 height: 630
---


# How to add HPA custom metrics

On this post I will cover the easy way to use your custom prometheus metrics as HPA scale factors

## The [HPA]()

> The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics)

Originally released on ??????, The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user.

### Installing HPA

HPA requires the metric-server, so you will have to 

```yaml
image:
    repository: gcr.io/google_containers/metrics-server-arm
args:
- --kubelet-insecure-tls=true
- --kubelet-preferred-address-types=InternalIP
```

`helm install stable/metrics-server --name metric-server --namespace kube-system -f values.yaml`

`helm install stable/prometheus-adapter --name prometheus-adapter --namespace monitoring --set image.repository=directxman12/k8s-prometheus-adapter-arm --set prometheus.url=ttp://prometheus-k8s.monitoring.svc.cluster.local`



now here is a little demo containing a sample app + its HPA:

gist-----------------

install it with `kubectl apply -f gist-----------------`




https://github.com/stefanprodan/k8s-prom-hpa

https://docs.okd.io/3.9/dev_guide/pod_autoscaling.html

https://github.com/helm/charts/tree/master/stable/prometheus-adapter