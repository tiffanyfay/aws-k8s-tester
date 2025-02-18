

WARNING: Pre-alpha. Do not use this in production. Only for testing.


# aws-k8s-tester

[![Go Report Card](https://goreportcard.com/badge/github.com/aws/aws-k8s-tester)](https://goreportcard.com/report/github.com/aws/aws-k8s-tester)
[![Godoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](https://godoc.org/github.com/aws/aws-k8s-tester)
[![Releases](https://img.shields.io/github/release/aws/aws-k8s-tester/all.svg?style=flat-square)](https://github.com/aws/aws-k8s-tester/releases)
[![LICENSE](https://img.shields.io/github/license/aws/aws-k8s-tester.svg?style=flat-square)](https://github.com/aws/aws-k8s-tester/blob/master/LICENSE)

## `aws-k8s-tester eks`

Make sure AWS credential is located in your machine:

```sh
cat ~/.aws/credentials

# confirm credential is valid
aws s3 ls
```

To install:

```bash
cd ${GOPATH}/src/github.com/aws/aws-k8s-tester
go install -v ./cmd/aws-k8s-tester
aws-k8s-tester eks create cluster -h
```

```bash
aws-k8s-tester eks create config --path ./aws-k8s-tester-eks.yaml

# change default configurations
vi ./aws-k8s-tester-eks.yaml
```

If run locally, make sure to change private key path to:

```diff
-worker-node-private-key-path: /Users/leegyuho/.ssh/kube_aws_rsa
+worker-node-private-key-path: /tmp/aws-k8s-tester-eks-worker-node.insecure.private.key
```

Otherwise,

> {"level":"warn","ts":"2019-08-14T14:14:19.938-0700","caller":"eks/eks.go:347","msg":"failed to create EKS, reverting","error":"open /Users/leegyuho/.ssh/kube_aws_rsa: permission denied"}

(`aws-k8s-tester` automatically detects OS to change the default value)

Now, let's create an EKS cluster:

```bash
aws-k8s-tester eks create cluster --path ./aws-k8s-tester-eks.yaml
```

This will create an EKS cluster with a worker node (takes about 20 minutes).

Once cluster is created, check cluster state using AWS CLI:

```bash
aws eks describe-cluster \
  --name a8-eks-190225-nqsht \
  --query cluster.status

"ACTIVE"
```

Cluster states are persisted on disk as well. EKS tester uses this file to track status.

```bash
cat ./aws-k8s-tester-eks.yaml

# or
less +FG ./aws-k8s-tester-eks.yaml
```

Tear down the cluster (takes about 10 minutes):

```bash
aws-k8s-tester eks delete cluster --path ./aws-k8s-tester-eks.yaml
```

### `aws-k8s-tester eks` e2e tests

To test locally:

```bash
# set "AWS_K8S_TESTER_EKS_TAG" to avoid S3 bucket conflicts
# or just disable log uploads with "AWS_K8S_TESTER_EKS_UPLOAD_TESTER_LOGS=false"
cd ${GOPATH}/src/github.com/aws/aws-k8s-tester

# use darwin to run local tests on Mac
AWS_K8S_TESTER_EKS_KUBECTL_DOWNLOAD_URL=https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/$(go env GOOS)/amd64/kubectl \
  AWS_K8S_TESTER_EKS_KUBECONFIG_PATH=/tmp/aws-k8s-tester/kubeconfig \
  AWS_K8S_TESTER_EKS_AWS_IAM_AUTHENTICATOR_DOWNLOAD_URL=https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/$(go env GOOS)/amd64/aws-iam-authenticator \
  AWS_K8S_TESTER_EKS_KUBERNETES_VERSION=1.13 \
  AWS_K8S_TESTER_EKS_WAIT_BEFORE_DOWN=1m \
  AWS_K8S_TESTER_EKS_DOWN=true \
  AWS_K8S_TESTER_EKS_ENABLE_WORKER_NODE_HA=true \
  AWS_K8S_TESTER_EKS_ENABLE_NODE_SSH=true \
  AWS_K8S_TESTER_EKS_ENABLE_WORKER_NODE_PRIVILEGED_PORT_ACCESS=true \
  AWS_K8S_TESTER_EKS_LOG_ACCESS=false \
  AWS_K8S_TESTER_EKS_UPLOAD_TESTER_LOGS=false \
  AWS_K8S_TESTER_EKS_UPLOAD_WORKER_NODE_LOGS=false \
  AWS_K8S_TESTER_EKS_WORKER_NODE_PRIVATE_KEY_PATH=~/.ssh/kube_aws_rsa \
  AWS_K8S_TESTER_EKS_WORKER_NODE_AMI=ami-00b95829322267382 \
  AWS_K8S_TESTER_EKS_WORKER_NODE_INSTANCE_TYPE=m3.xlarge \
  AWS_K8S_TESTER_EKS_WORKER_NODE_ASG_MIN=1 \
  AWS_K8S_TESTER_EKS_WORKER_NODE_ASG_MAX=1 \
  AWS_K8S_TESTER_EKS_WORKER_NODE_ASG_DESIRED_CAPACITY=1 \
  ./tests/ginkgo.sh
```
