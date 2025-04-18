# Makefile for creating a KIND cluster (1 control-plane, 2 workers) with Cilium (and Hubble) enabled,
# and for retrieving logs/metrics of outbound TCP connections.

KIND_CLUSTER_NAME = kind-cluster
CILIUM_VERSION = 1.13.4
NAMESPACE = kube-system

.PHONY: kind cluster cilium hubble observe logs clean

# Combined target to run cluster, cilium, and hubble
kind: cluster cilium
	@echo "KIND cluster with Cilium and Hubble installed and configured!"

cluster:
	kind create cluster --name $(KIND_CLUSTER_NAME) --config kind-config.yaml

# Install Cilium via Helm with Hubble enabled.
cilium:
	@echo "Adding Cilium Helm repository..."
	helm repo add cilium https://helm.cilium.io/
	helm repo update
	@echo "Installing Cilium with Hubble and metrics enabled..."
	helm install cilium cilium/cilium --version $(CILIUM_VERSION) --namespace $(NAMESPACE) \
	  --set hubble.enabled=true \
		--set hubble.relay.enabled=true \
	  --set hubble.metrics.enabled[0]=dns \
	  --set hubble.metrics.enabled[1]=drop \
	  --set hubble.metrics.enabled[2]=port-distribution \
	  --set hubble.metrics.enabled[3]=flow
	@echo "Waiting for Cilium pods to be ready..."
	kubectl -n $(NAMESPACE) wait --for=condition=ready pod -l k8s-app=cilium --timeout=90s

observe:
	@echo "Observing TCP connections..."
	kubectl -n $(NAMESPACE) port-forward svc/hubble-relay 8080:80
	hubble observe --protocol tcp

logs:
	@echo "Fetching logs from Cilium pods filtered for 'tcp'..."
	kubectl -n $(NAMESPACE) logs -l k8s-app=cilium | grep -i "tcp"

down:
	@echo "Deleting KIND cluster: $(KIND_CLUSTER_NAME)"
	kind delete cluster --name $(KIND_CLUSTER_NAME)

