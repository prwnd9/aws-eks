# aws-eks

## Usage
```bash
export AWS_REGION=ap-southeast-1
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

atmos vendor pull
atmos workflow apply -f basic
atmos workflow destroy -f basic

# get kubeconfig
CLUSTER_NAME=test-eks-v0
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
kubectl get pods -A

# test
kubectl run test --image=alpine --command -- sleep infinity
kubectl exec -it test -- sh
apk update && apk add tcptraceroute
tcptraceroute s3.ap-southeast-1.amazonaws.com 443
kubectl delete pod test

# metrics-server
# https://artifacthub.io/packages/helm/metrics-server/metrics-server
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server --version 3.12.2 \
    --set resources.requests.cpu=null \
    --set resources.requests.memory=null

# autoscaler
AUTOSCALER_ROLE=arn:aws:iam::99999999999:role/eks-cluster-autoscaler
helm repo add cluster-autoscaler https://kubernetes.github.io/autoscaler
helm install cluster-autoscaler cluster-autoscaler/cluster-autoscaler --version 9.43.2 -f - <<EOF
awsRegion: $AWS_REGION
autoDiscovery:
  clusterName: $CLUSTER_NAME
rbac:
  serviceAccount:
    name: cluster-autoscaler
    annotations:
      eks.amazonaws.com/role-arn: "${AUTOSCALER_ROLE}"
EOF
kubectl --namespace=default get pods -l "app.kubernetes.io/name=aws-cluster-autoscaler,app.kubernetes.io/instance=cluster-autoscaler"
```
