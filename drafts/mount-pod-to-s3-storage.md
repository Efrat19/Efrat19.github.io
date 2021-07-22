- suitible for images / legacy, not for DB etc
- https://github.com/datashim-io/datashim
- deploy DLF resources kubectl apply -f https://raw.githubusercontent.com/datashim-io/datashim/master/release-tools/manifests/dlf.yaml
- create bucket
- create user with bucket access
```terraform
resource "aws_iam_user" "user" {
  name     = "data-cluster-storage"
  path     = "/"
  tags = local.common_tags
}
module "data-cluster-storage" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-group-with-policies"
  version = "~> 3.0"
  name = "data-cluster-storage"

  group_users = ["data-cluster-storage"]

  custom_group_policy_arns = [
    "arn:aws:iam::${local.account_num}:policy/clusterS3FullAccess",
  ]
  tags = local.common_tags
}
resource "aws_s3_bucket" "data-cluster-cluster-backend" {
    bucket = "data-cluster-cluster-backend"
    acl    = "private"
    tags = local.common_tags
}
resource "aws_iam_policy" "clusterS3FullAccess" {
    name        = "clusterS3FullAccess"
    path        = "/"
    description = "allow full access to cluster s3 backend"
    tags = local.common_tags
    policy      = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "s3:PutAccountPublicAccessBlock",
        "s3:GetAccountPublicAccessBlock",
        "s3:ListAllMyBuckets",
        "s3:ListJobs",
        "s3:CreateJob",
        "s3:HeadBucket",
        "s3:ListBucket"
      ],
      "Resource": "*"
    },
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::data-cluster-cluster-backend",
        "arn:aws:s3:::data-cluster-cluster-backend/*"
      ]
    }
  ]
}
POLICY
}

```
- create secret for the user creds:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cluster-storage
  namespace: dlf
data:
  accessKeyID: {}
  secretAccessKey: {}
```
- create the dataset:
```yaml
apiVersion: com.ie.ibm.hpsys/v1alpha1
kind: Dataset
metadata:
  name: s3-dataset
  namespace: default
spec:
  local:
    type: "COS"
    secret-name: "cluster-storage" #see s3-secrets.yaml for an example
    secret-namespace: "dlf" #optional if the secret is in the same ns as dataset
    endpoint: "https://s3.eu-west-1.amazonaws.com/cluster-s3-storage"
    bucket: "cluster-s3-storage"
    readonly: "false" #OPTIONAL, default is false  
    region: "eu-west-1" #OPTIONAL
```
- after created you will see the pvc:
```
~ % (⎈ |cluster:monitoring) k get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
s3-dataset       Bound    pvc-818acd89-bbd9-45ea-8483-f8ba9943f623   9314Gi     RWX            csi-s3           63m
```
- mount the pvc to a pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "test-mount"
  namespace: monitoring
spec:
  containers:
  - name: test-mount
    image: nginx:alpine
    volumeMounts:
    - mountPath: /tmp
      name: s3-storage
      readOnly: false
      subPath: test_mount
  volumes:
  - name: s3-storage
    persistentVolumeClaim:
        claimName: s3-dataset 
```
- ensure the pod is mounted:
```
k exec -ti test-mount ash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
/ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                 100.0G     11.4G     88.6G  11% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                    14.9G         0     14.9G   0% /sys/fs/cgroup
cluster-storage          1.0P         0      1.0P   0% /tmp
.....
```
