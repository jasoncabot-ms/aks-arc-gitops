# Kind

Steps for setting up a test cluster using kind

# Set Up

* Install Go
* Run `GO111MODULE="on" go get sigs.k8s.io/kind@v0.9.0`
* From: https://kind.sigs.k8s.io/docs/user/quick-start/

# Create Cluster

```
export PORT=9080
export SECURE_PORT=9443

cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: $PORT
    protocol: TCP
  - containerPort: 443
    hostPort: $SECURE_PORT
    protocol: TCP
EOF
```

This is required to allow `Ingress` resources to be accessible on local ports on the host