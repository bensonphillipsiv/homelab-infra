BOOTSTRAP_DIR=./bootstrap
TALOS_DIR=./talos
CLUSTER_NAME=k8s

.PHONY: all genconfig build_talos bootstrap clean

all: genconfig build_talos bootstrap clean

genconfig:
	@echo "Generating configuration using talhelper and doppler..."
	cd $(TALOS_DIR) && \
	doppler run -p talhelper -c env talhelper genconfig

build_talos:
	@echo "Building Talos cluster..."
	cd $(TALOS_DIR)/clusterconfig && \
	talosctl apply-config -n 192.168.1.100 --insecure --file $(CLUSTER_NAME)-b0.yaml
	@sleep 120
	
	@echo "Bootstrapping Talos cluster..."
	talosctl bootstrap --nodes 192.168.1.100 --endpoints 192.168.1.100 --talosconfig=$(TALOS_DIR)/clusterconfig/talosconfig
	@sleep 240

	cd $(TALOS_DIR)/clusterconfig && \
	talosctl apply-config -n 192.168.1.101 --insecure --file $(CLUSTER_NAME)-b1.yaml
	@sleep 120

bootstrap:
	@echo "Bootstrapping cluster with ArgoCD and base manifests..."
	cd $(BOOTSTRAP_DIR) && \
	doppler run -p talhelper -c env -- bash -c "kustomize build --enable-helm . | envsubst" | kubectl apply -f -

clean:
	@echo "Cleaning up generated files..."
	rm -rf $(TALOS_DIR)/clusterconfig
	rm -rf $(BOOTSTRAP_DIR)/charts
