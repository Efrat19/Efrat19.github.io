---
layout: post
title: What is SAN Certificate and Why Should I Care 
lang: en
categories:
    - aws
    - SSL
    - cert
    - -manager
    - ingress
    - nginx
    - kubernetes
# tags:
#     - hoge
#     - foo
permalink: /:slug
image:
 path: /assets/designer/tinified_meta.png
 width: 1200
 height: 630
---



https://stackoverflow.com/questions/2115611/wildcard-ssl-on-sub-subdomain

https://docs.aws.amazon.com/cli/latest/reference/acm/request-certificate.html

https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html
https://aws.amazon.com/premiumsupport/knowledge-center/terminate-https-traffic-eks-acm/

aws acm request-certificate \
--domain-name "*.mad2.co.il" \
--validation-method DNS \
--idempotency-token 1234 \
--subject-alternative-names "*.adminportal.mad2.co.il" "*.mobile.mad2.co.il" "*.make.mad2.co.il" "*.price.mad2.co.il" "*.prod.mad2.co.il" 
{
    "CertificateArn": "arn:aws:acm:eu-west-1:123000000321:certificate/xxxxxxx"
}
