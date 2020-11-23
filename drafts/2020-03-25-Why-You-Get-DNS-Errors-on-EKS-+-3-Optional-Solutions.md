---
layout: post
title: Why You Get DNS Errors on EKS + 3 Optional Solutions 
# slug: cvbnjklbjm,.
lang: en
categories:
    - Kubernetes
    - EKS
    - DNS
# tags:
#     - hoge
#     - foo
permalink: /:slug 
image:
 path: /assets/designer/tinified_meta.png
 width: 1200
 height: 630
---

On this post I am covering 3 ways to mitigate DNS overload on your EKS Cluster. DNS functioning is a critical part on your cluster, if the DNS system won't handle the load, DNS requests will time out and the apps will start splashing DNS errors. This is why you must constantly monitor your DNS load scale and once it grows. The first 2 are really easy to implement, and really helpful for small to medium sized clusters, but more cosmetic, compared to the third one which is the toughest, final answer that will buy you a inner peace for long time.


##### Level 0 Solution (Just in Case You haven't Tried it Yet)
### Upscaling CoreDNS Replicas

Yeah, You have already tried to upscale your CoreDNS replicas, right? Just wanted to make sure. Because it might help


### The VPC Resolver Kicks In

Adding to that sine the VPC resolver can accept only a maximum hard limit of 1024 "packets" per second per "network" interface. If more than one CoreDNS pod is on the "same" worker node, then the chances of hitting this limit are higher for external domains queries, and in that situation increasing core dns pod may not be helpful.


##### It didn't work - Time for Level 1 Solution-
### Spreading DNS Pods 

https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity

##### The Most Massive (and my favorite :yum:) Solution-
### Caching Local DNS

Though it took more time to implement than the others, and also might cause downtime (You are going to replace your node-groups) this is the most efficient solution, which will dramatically reduce your DNS traffic scale ()

<!-- https://aws.amazon.com/blogs/containers/eks-dns-at-scale-and-spikeiness/ -->
https://www.vladionescu.me/posts/eks-dns.html
https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/

<!-- 


## The Problem with That Solution 

While I saw bare metal kubernetes cluster handling massive DNS traffic through a handful of coreDNS pods, with EKS things seem to change a bit - a new player enters the scene - 

### Why Can't I Just Upscale the CoreDNS Replicas

This might help, 



### Level 1 Solution - Pod anti-affinity


### What if it's Still not Enough? 
 -->

