# Addons

## Usage
```bash
export AWS_REGION=ap-southeast-1
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

# eks version
aws eks describe-addon-versions | jq -r ".addons[] | .addonVersions[] | .compatibilities[] | .clusterVersion" | sort | uniq
EKS_VERSION=1.31

# eks addons
addons=("vpc-cni" "coredns" "kube-proxy")
for item in "${addons[@]}"; do
    echo $item
    aws eks describe-addon-versions --kubernetes-version $EKS_VERSION --addon-name $item \
      --query 'addons[].addonVersions[].{Version: addonVersion, Defaultversion: compatibilities[0].defaultVersion}' --output table
done
```
