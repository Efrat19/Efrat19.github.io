why k8s storage class must have a valid DNS name

If you ever had the chance to incorrectly rename a storageClass and deploy it, you came across this error:

```console
The StorageClass "test_the_storage_class" is invalid: metadata.name: Invalid value: "test_the_storage_class": a DNS-1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')
```

When I heared about this rule it sounded wierd but I just now I had the time and passion to dig deeper, and here is what I found:

### Why Do We Even need storage class?

In all those charts aiming to explain k8s volume model, you will see 


and the StorageClass is just a part on the side and nobody fully understands what is its real purpose.

### How the storage class fits into the volume model


### Dynamic provisioning

>Dynamic volume provisioning allows storage volumes to be created on-demand. Without dynamic provisioning, cluster administrators have to manually make calls to their cloud or storage provider to create new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes.



### Why Does it need a DNS name:

