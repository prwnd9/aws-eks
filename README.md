aws-eks
=======

## Usage
```bash
export AWS_REGION=ap-southeast-1
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

atmos vendor pull

atmos workflow init -f main
atmos workflow plan -f main
atmos workflow apply -f main
atmos workflow destroy -f main

# get kubeconfig
CLUSTER_NAME=test-eks-v0
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
kubectl get pods -A

# karpenter
# https://github.com/aws/karpenter-provider-aws
# https://github.com/aws/karpenter-provider-aws/blob/main/charts/karpenter/values.yaml
# https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/
export KARPENTER_NAMESPACE="kube-system"
export KARPENTER_VERSION="1.0.7"
export K8S_VERSION="1.31"

# karpenter Pod fails to Start -The specified queue does not exist.
# https://github.com/kubernetes-sigs/karpenter/issues/1799
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version "${KARPENTER_VERSION}" \
  --namespace "${KARPENTER_NAMESPACE}" \
  --create-namespace \
  --set "settings.clusterName=${CLUSTER_NAME}" \
  --set "settings.interruptionQueue=${CLUSTER_NAME}" \
  --set "replicas=1" \
  --set logLevel=debug \
  --wait
```

## Issues

> [!ERROR] karpenter Pod fails to Start -The specified queue does not exist.
> https://github.com/kubernetes-sigs/karpenter/issues/1799
