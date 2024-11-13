# aws-eks

## Usage
```bash
AWS_ACCOUNT_ID=
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
# https://artifacthub.io/packages/helm/cluster-autoscaler/cluster-autoscaler
AUTOSCALER_ROLE=arn:aws:iam::${AWS_ACCOUNT_ID}:role/eks-cluster-autoscaler # eks- prefix
helm repo add cluster-autoscaler https://kubernetes.github.io/autoscaler
helm upgrade --install cluster-autoscaler cluster-autoscaler/cluster-autoscaler --version 9.43.2 -f - <<EOF
awsRegion: $AWS_REGION
autoDiscovery:
  clusterName: $CLUSTER_NAME
rbac:
  serviceAccount:
    name: cluster-autoscaler
    annotations:
      eks.amazonaws.com/role-arn: "${AUTOSCALER_ROLE}"
EOF

# aws load balancer controller
# https://artifacthub.io/packages/helm/aws/aws-load-balancer-controller
ALBC_ROLE=arn:aws:iam::${AWS_ACCOUNT_ID}:role/aws-load-balancer-controller
helm repo add aws https://aws.github.io/eks-charts
helm upgrade --install aws-load-balancer-controller aws/aws-load-balancer-controller --namespace kube-system --version 1.10.0 -f - <<EOF
replicaCount: 1
clusterName: $CLUSTER_NAME
serviceAccount:
  name: aws-load-balancer-controller
  annotations:
    eks.amazonaws.com/role-arn: "${ALBC_ROLE}"
EOF

# https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/pod_readiness_gate/
kubectl label namespace default elbv2.k8s.aws/pod-readiness-gate-inject=enabled
kubectl apply -f manifests/whoami.yaml
kubectl delete -f manifests/whoami.yaml
# test
ALB_IP=
echo "$ALB_IP whoami.test.com" >> /etc/hosts
curl whoami.test.com

# argocd
# https://artifacthub.io/packages/helm/argo/argo-cd
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade --install argo-cd argo/argo-cd --version 7.7.2
```
