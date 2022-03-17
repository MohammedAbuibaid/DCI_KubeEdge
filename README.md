# DCI_KubeEdge

# Using the node_setup bash script, install docker, kubeadm, kubectl, kubelet, and keadm on all three nodes (central and edge clouds):
bash node_setup.sh

# On K8s master node:
# 1) run the following to initialize K8s control-plane. Make sure to save the output kubeadm join <MasterNodeIP:Port> --token <...> --discovery-token-ca-cert-hash <...>
export KUBECONFIG=/etc/kubernetes/admin.conf
echo 'export KUBECONFIG=$HOME/admin.conf' >> $HOME/.bashrc
kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=all


# 2) install flannel CNI:
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 3) initialize KubeEdge Cloud Core and generate the KubeEdge token
keadm init --advertise-address=<MasterNodeIP:Port> --kube-config=$KUBECONFIG
keadm gettoken --kube-config=$KUBECONFIG

# 4) monitoring the status of the edge nodes
watch 'kubectl get nodes'

# On the Edge Clouds
# 1) run the keadm join command using the generated token in the previous step 3
keadm join --cloudcore-ipport=<MasterNodeIP>:10000 --token=<...>


# Verification:
# from the K8s master node, run the following command to deploy 2 replicas of custum nginx server.
kubectl apply -f https://raw.githubusercontent.com/MohammedAbuibaid/intro-to-kubernetes/master/manifests/nginx_dep.yaml
