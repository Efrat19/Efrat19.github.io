---
layout: post
title: SSL LoadBalancer on EKS 
lang: en
categories:
    - Kubernetes
    - Ingress
    - AWS Cert Manager
    - SSL
    - EKS
    - AWS
    - Helm
# tags:
#     - hoge
#     - foo
permalink: /:slug
image:
 path: /assets/designer/tinified_meta.png
 width: 1200
 height: 630
---

In this post I will walk you through a painless way to get a running kubernetes LoadBalancer ingress with SSL support. Assuming you have an EKS cluster running and you wish to add an SSL ingress.

### Step 1: Creating a Certificate-Request:

Here is my certificate request fot every subdomain of `mydomain.com` and every subdomain of `subdomain.mydomain.com`:

```console
~ $ aws acm request-certificate \
--domain-name "*.mydomain.com" \
--validation-method DNS \
--idempotency-token 1234 \
--subject-alternative-names "*.subdomain.mydomain.com" 
{
    "CertificateArn": "arn:aws:acm:eu-west-1:123123123123:certificate/876r5ytfuyigho987ytfgiub"
}
```
I got the certificate ARN, which will be put to use on step 3. The certificate is requested but can not be used yet, Until we validate it:

### Step 2: Passing The DNS Challenge

So far I requested a cert for my domains (`mydomain.com` and `subdomain.mydomain.com`) but to make Amazon issue that cert, I must pass a certain challenge to prove that I am the owner of those domains. To do that I usually go for DNS validation because it is very simple explain & complete: You get from the issuer (amazon in this case) a CNAME record pair for each domain you requested a cert for. Something like:

| name:  | _erfvd89jfdcx.mydomain.com            | 
|--------|----------------------------------------|
| value: | _4erjvd9jmespdap0.acm-validations.aws. |  

amazon cert manager will start running DNS lookups for `_erfvd89jfdcx.mydomain.com`. Once it will lead it to `_4erjvd9jmespdap0.acm-validations.aws.`, it will be the proof that I am the owner of `mydomain.com` (because I was able to register the CNAME under its subdomain `_erfvd89jfdcx`)

lets head over to [cert manager console](https://console.aws.amazon.com/acm/home) and find the cert:

<img src="{{"/assets/img/acm/Untitled5.png" | relative_url }}">

as you can see, the status is still `Pending Validation`. open up the domain list and copy the name-value pair (covered in pink):

<img src="{{"/assets/img/acm/Untitled4.png" | relative_url }}">

now here is a little script `register-cname.sh` for registering the CNAME via Route53:

```bash
#!/bin/bash

# name item in the pair:
NAME=
# value item in the pair:
VALUE=
# go to route53 and find the hosted zone id of your domain:
ZONE_ID=

aws route53 change-resource-record-sets --hosted-zone-id $ZONE_ID  
--change-batch '{ "Comment": "DNS Validation For ACM", 
"Changes": [ { "Action": "CREATE", "ResourceRecordSet": { "Name": 
"'"$NAME"'", "Type": "CNAME", "TTL": 
120, "ResourceRecords": [ { "Value": "'"$VALUE"'" } ] } } ] }'
```

```console
~ $ chmod +x register-cname.sh && ./register-cname.sh  
{
    "ChangeInfo": {
        "Status": "PENDING", 
        "Comment": "DNS Validation For ACM", 
        "SubmittedAt": "2020-05-23T22:49:03.349Z", 
        "Id": "/change/C01902743XXXXXXLRPP6"
    }
}
```
The status is pending. After a few minutes it will be created, and by going back to cert manager and opening the domain row, you will see a message:
> **SUCCESS: The DNS record was written to your Route 53 hosted zone. It can take 30 minutes or longer for the changes to propagate and for AWS to validate the domain and issue the certificate.**

<img src="{{"/assets/img/acm/image1.png" | relative_url }}">

Repeat this step for each domain you would like to validate. (in my example - `mydomain.com` and `subdomain.mydomain.com`)

Once you are done, wait until you see the certificate status changing from `Pending Validation` to `Issued`:

<img src="{{"/assets/img/acm/Untitled3.png" | relative_url }}">

### Step 3: Creating Ingress Controller:

The easiest way to get an ingress LoadBalancer is using the [nginx ingress helm chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress)

lets create a `values.yaml` file to customize it:

```yaml
controller:
  replicaCount: 1
  ingressClass: nginx-ssl
  service:
    targetPorts:
      http: http
      https: http
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <paste here the cert ARN from step 1>
      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
      service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "https"
      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: '3600'
# you should place your ingress on stable nodes, e.g not on spot instances:
#   nodeSelector:
#     spot: "false" 
  config:
    use-forwarded-headers: "true"
```

and install the chart:

```console
~ $ helm install --name nginx-ssl \
--namespace ingress stable/nginx-ingress -f values.yaml
```

### Step 4: Creating a Test Deployment:

For the test I will be using the official rabbitMQ helm chart
first lets create the `values.yaml` file:

```yaml
rabbitmq:
  password: test
persistence:
  enabled: false
ingress: 
  enabled: true
  hostName: rabbit.mydomain.com
  annotations:
    # The class we set on the ingress deployment:
    kubernetes.io/ingress.class: "nginx-ssl"
    # make sure that access to the admin dashboard will always be ssl secured:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

Now we can add the repo:
`helm install rabbitmq-ssl-test stable/rabbitmq -f values.yaml`


once applied, you should be able to securely access rabbitmq admin dashboard via `https://rabbit.mydomain.com` :star-struck: (user is `user` and password is `test`)


