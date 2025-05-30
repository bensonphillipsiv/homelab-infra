BOOTSTRAP_DIR=./bootstrap
TALOS_DIR=./talos
CLUSTER_NAME=k8s
NODE_NAME=b0

.PHONY: all build_talos bootstrap clean

all: build_talos bootstrap clean

build_talos:
	@echo "Generating configuration using talhelper and doppler..."
	cd $(TALOS_DIR) && \
	doppler run -p talhelper -c env talhelper genconfig

	@echo "Building Talos cluster..."
	cd $(TALOS_DIR)/clusterconfig && \
	talosctl apply-config -n 192.168.1.100 --insecure --file $(CLUSTER_NAME)-$(NODE_NAME).yaml
	
	@sleep 120
	@echo "Bootstrapping Talos cluster..."
	talosctl bootstrap --nodes 192.168.1.100 --endpoints 192.168.1.100 --talosconfig=$(TALOS_DIR)/clusterconfig/talosconfig
	@sleep 240

bootstrap:
	@echo "Bootstrapping cluster with ArgoCD and base manifests..."
	cd $(BOOTSTRAP_DIR) && \
	doppler run -p talhelper -c env -- bash -c "kustomize build --enable-helm . | envsubst" | kubectl apply -f -

clean:
	@echo "Cleaning up generated files..."
	rm -rf $(TALOS_DIR)/clusterconfig
	rm -rf $(BOOTSTRAP_DIR)/charts
