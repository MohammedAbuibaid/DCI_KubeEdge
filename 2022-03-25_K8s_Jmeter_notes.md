#Docker installation
```
sudo apt update
sudo ufw disable
sudo swapoff -a; sed -i '/swap/d' /etc/fstab
sudo cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
```

# K8s installation
```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
sudo apt update && apt -y install vim git curl wget kubelet kubeadm kubectl
```

Most probably, you will encounter the following error during the K8s installtion
```
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp 127.0.0.1:10248: connect: connection refused.
```
If so, then the problem was cgroup driver. Kubernetes cgroup driver was set to systems but docker was set to systemd.
So create /etc/docker/daemon.json
```
nano /etc/docker/daemon.json
```
and added below:
```
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
```
Then
```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```
Run kubeadm init or kubeadm join again. [Ref](https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy)

# CNI installation
Option 1: Calico
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```
Option 2: Flannel
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
# TeaStore installation
```
kubectl apply -f https://raw.githubusercontent.com/DescartesResearch/TeaStore/master/examples/kubernetes/teastore-clusterip.yaml
```
# Java Installtion
...
Jmeter Testing
```
java -jar $HOME/TeaStore/examples/jmeter/apache-jmeter-5.4.3/bin/ApacheJMeter.jar -t $HOME/TeaStore/examples/jmeter/teastore_browse_nogui.jmx -Jhostname 137.184.174.125 -Jport 30080 -JnumUser 20 -JrampUp 10 -l 03-24_20_10.log -n
```
OR 
```
java -jar ApacheJMeter.jar -t ~/apache-jmeter-5.4.3/teastore_browse_nogui.jmx -Jhostname 192.168.1.20  -Jport 31187  -JnumUser 10 -JrampUp 1 -l mylogfile.log -n
```
