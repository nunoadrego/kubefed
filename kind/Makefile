CLUSTER_1=alpha
CLUSTER_2=beta
CLUSTER_1_CTX=kind-alpha
CLUSTER_2_CTX=kind-beta
K8S_VERSION=1.23.6

setup: clusters kubefed

kubefed: kubefed-install kubefed-join kubefed-kind-fix kubefed-check

kubefed-install:
	helm repo add kubefed-charts https://raw.githubusercontent.com/kubernetes-sigs/kubefed/master/charts
	helm --namespace kube-federation-system upgrade -i kubefed kubefed-charts/kubefed --create-namespace --kube-context ${CLUSTER_1_CTX}
	helm --namespace kube-federation-system upgrade -i kubefed kubefed-charts/kubefed --create-namespace --kube-context ${CLUSTER_2_CTX}

kubefed-join:
	kubefedctl join ${CLUSTER_1} --cluster-context ${CLUSTER_1_CTX} \
    	--host-cluster-context ${CLUSTER_1_CTX} --v=2
	kubefedctl join ${CLUSTER_2} --cluster-context ${CLUSTER_2_CTX} \
    	--host-cluster-context ${CLUSTER_1_CTX} --v=2

# Source: https://github.com/kubernetes-sigs/kubefed/blob/master/scripts/fix-joined-kind-clusters.sh
kubefed-kind-fix:
	kubectl config use-context ${CLUSTER_1_CTX}
	bash fix-joined-kind-clusters.sh

kubefed-check:
	kubectl -n kube-federation-system get kubefedclusters --context ${CLUSTER_1_CTX}

clusters:
	kind create cluster --name ${CLUSTER_1} --image kindest/node:v${K8S_VERSION}
	kind create cluster --name ${CLUSTER_2} --image kindest/node:v${K8S_VERSION}

kubefed-deploy-app:
	kubefedctl federate namespace default
	kubectl apply -f deployment.yaml --context ${CLUSTER_1_CTX}
	kubefedctl federate deploy test

clean:
	kind delete cluster --name ${CLUSTER_1}
	kind delete cluster --name ${CLUSTER_2}
