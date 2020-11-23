

trying operator framework
https://sdk.operatorframework.io/docs/golang/quickstart/
```console
brew install operator-sdk
operator-sdk new kube-load --repo=github.com/Efrat19/kube-load    
operator-sdk add api --api-version=load.efrat19.io/v1alpha1 --kind=LoadManager
edit pkg/apis/load/v1alpha1/loadmanager_types.go
operator-sdk generate k8s
operator-sdk add controller --api-version=load.efrat19.io/v1alpha1 --kind=LoadManager
FROM alpine:3.10
operator-sdk build efrat19/kube-load
sed -i "" 's|REPLACE_IMAGE|efrat19/kube-load:latest|g' deploy/operator.yaml
k apply -f deploy/ -n kube-load
```

trying kubebuilder https://book.kubebuilder.io/quick-start.html

install:
```bash
os=$(go env GOOS)
arch=$(go env GOARCH)

# download kubebuilder and extract it to tmp
curl -Lk https://go.kubebuilder.io/dl/2.3.1/${os}/${arch} | tar -xz -C /tmp/

# move to a long-term location and put it on your path
# (you'll need to set the KUBEBUILDER_ASSETS env var if you put it somewhere else)
sudo mv /tmp/kubebuilder_2.3.1_${os}_${arch} /usr/local/kubebuilder
export PATH=$PATH:/usr/local/kubebuilder/bin
```

new project:
```console
~ $ mkdir $GOPATH/src/github.com/Efrat19/kubeload
~ $ cd $GOPATH/src/github.com/Efrat19/kubeload
~ $ kubebuilder init --domain kubeload.efrat19.io

kubebuilder create api --group kubeload  --version v1 --kind LoadManager
vim api/v1/loadmanager_types.go
make
make install

```
