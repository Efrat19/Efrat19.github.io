<!-- helm repo add elastic https://helm.elastic.co
helm install --name elasticsearch elastic/elasticsearch --set persistence.enabled=false --namespace es --set replicas=1 --set minimumMasterNodes=1

helm install --name elasticsearch stable/elastic-stack  -->

https://logz.io/blog/deploying-the-elk-stack-on-kubernetes-with-helm/

Helm install --name elasticsearch elastic/elasticsearch --set persistence.enabled=false --set replicas=1 --set minimumMasterNodes=1 -f https://raw.githubusercontent.com/elastic/Helm-charts/master/elasticsearch/examples/minikube/values.yaml

Helm install --name kibana elastic/kibana 