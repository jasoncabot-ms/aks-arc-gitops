# Azure Arc for Kubernetes

This will register your cluster (wherever it is provisioned) to Azure Arc

```
export CLUSTER_NAME=azurearctest1
export RESOURCE_GROUP=azurearctest
export REPOSITORY=jasoncabot-ms/aks-arc-gitops

# Connect cluster to Azure Arc
az connectedk8s connect --name "$CLUSTER_NAME" --resource-group "$RESOURCE_GROUP"

# Set up cluster administration
az k8sconfiguration create \
    --resource-group "$RESOURCE_GROUP" \
    --cluster-name "$CLUSTER_NAME" \
    --cluster-type connectedClusters \
    --name baseline-config \
    --scope cluster \
    --repository-url git@github.com:${REPOSITORY}.git \
    --operator-namespace azure-arc \
    --operator-instance-name baseline-config \
    --operator-params="--git-readonly --git-path=baseline/manifests --sync-garbage-collection"


# Find the public key created by the flux agent and add to GitHub as a Deploy Key. This is required even for read-only access via SSH on GitHub. See also: https://docs.fluxcd.io/en/1.17.0/tutorials/get-started.html#giving-write-access 

# Create an app
export APP_NAME=podinfo-app
mkdir -p ./apps/manifests/$APP_NAME
kubectl apply -k github.com/stefanprodan/podinfo//kustomize -o yaml --dry-run > ./apps/manifests/$APP_NAME/deploy.yaml

# Create a namespace for it
cat <<EOF >> ./baseline/manifests/namespaces/$APP_NAME.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: $APP_NAME
EOF

# Connect this new namespace to Arc
export APP_NAME=podinfo-app
az k8sconfiguration create \
    --resource-group "$RESOURCE_GROUP" \
    --cluster-name "$CLUSTER_NAME" \
    --cluster-type connectedClusters \
    --name $APP_NAME \
    --scope namespace \
    --repository-url git@github.com:${REPOSITORY}.git \
    --operator-namespace $APP_NAME \
    --operator-instance-name $APP_NAME \
    --operator-params="--git-readonly --git-path=apps/manifests/$APP_NAME --sync-garbage-collection"


```