### Upgrade CNI

kubectl describe daemonset aws-node --namespace kube-system | grep Image | cut -d "/" -f 2
amazon-k8s-cni:v1.5.7

curl -o aws-k8s-cni.yaml https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.6/config/v1.6/aws-k8s-cni.yaml
sed -i -e 's/us-west-2/eu-west-1/' aws-k8s-cni.yaml
kubectl apply -f aws-k8s-cni.yaml



