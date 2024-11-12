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
kubectl delete pod test
```
