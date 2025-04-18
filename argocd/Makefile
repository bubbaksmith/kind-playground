.PHONY: *

K8S_CLUSTER_NAME := argocd-playground
KUBE_CONTEXT := kind-$(K8S_CLUSTER_NAME)

kind: $(KIND_CONFIG) kind-up deploy-argocd apply-argocd-app port-forward

kind-up:
	kind get clusters | grep ${K8S_CLUSTER_NAME} || kind create cluster --name ${K8S_CLUSTER_NAME} --config ./kind/config.yaml

deploy-argocd:
	@echo "Deploying ArgoCD to the Kind cluster..."
	@kubectl --context $(KUBE_CONTEXT) create namespace argocd 2>/dev/null || true
	@kubectl --context $(KUBE_CONTEXT) apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
	@echo "Waiting for Argo CD server deployment to be available..."
	@kubectl --context $(KUBE_CONTEXT) wait --for=condition=available --timeout=600s deployment/argocd-server -n argocd

# deploy-namespace-generator:
# 	@echo "Deploying Namespace Generator to the Kind cluster..."
# 	@kubectl --context $(KUBE_CONTEXT) apply -k https://github.com/konflux-ci/namespace-generator/manifests?ref=main
# 	@kubectl --context $(KUBE_CONTEXT) wait --for=condition=available --timeout=600s deployment/namespace-generator -n argocd

apply-argocd-app:
	@echo "Applying ArgoCD application..."
	@kubectl --context $(KUBE_CONTEXT) -n argocd apply -f https://raw.githubusercontent.com/konflux-ci/namespace-generator/main/example/appset.yaml

port-forward:
	@echo "Login credentials:"
	@echo "Username: admin"
	@echo "Password: $(shell kubectl --context $(KUBE_CONTEXT) -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)"
	@echo "Port forwarding ArgoCD server. Access the UI at http://localhost:8080"
	@if [ -f .argocd-pf.pid ]; then \
	  PID=$$(cat .argocd-pf.pid); \
	  if kill -0 $$PID 2>/dev/null; then \
		echo "Port-forward already running with PID $$PID. Skipping..."; \
		exit 0; \
	  else \
		echo "Stale PID file found. Removing..."; \
		rm -f .argocd-pf.pid; \
	  fi; \
	fi
	@echo "Starting port-forward for ArgoCD server on http://localhost:8080..."
	@kubectl --context $(KUBE_CONTEXT) port-forward svc/argocd-server -n argocd 8080:443 > /dev/null 2>&1 & echo $$! > .argocd-pf.pid

clean:
	@if [ -f .argocd-pf.pid ]; then \
	  echo "Killing port-forward process with PID $$(cat .argocd-pf.pid)..."; \
	  kill $$(cat .argocd-pf.pid) && rm -f .argocd-pf.pid; \
	else \
	  echo "No port-forward process found."; \
	fi

kind-down:
	kind delete clusters ${K8S_CLUSTER_NAME}
