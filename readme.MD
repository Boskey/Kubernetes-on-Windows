# Instructions to setup Kubernetes Cluster with Windows Nodes

###### These instructions are for attaching a Windows Node to an existing Kubernetes cluster running Flannel Network on vSphere ( although can be extended to other Infrastructure platforms)

## Kubeadm to deploy Kubernetes on vSphere

	1. Deployed a Ubuntu 16.0 + image
	2. Make sure the VM has 2 vCPU's and 2 GB Memory
	3. run swapoff -a #This will immediately disable swap.
	4. Install Docker
		
	5. Install Kubeadm
	
  ```apt-get update && apt-get install -y apt-transport-https curl```
  
   ```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - ```
   
   
   ```
   cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF
   ```
        
```apt-get update```

```apt-get install -y kubelet kubeadm kubectl```

```apt-mark hold kubelet kubeadm kubectl```

	6. Init Kubeadm
	
```kubeadm init --pod-network-cidr=10.244.0.0/16```

		
	7. Configure kubectl
	
	``` mkdir -p $HOME/.kube```
	
```sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```

```sudo chown $(id -u):$(id -g) $HOME/.kube/config```

	8. Install Pod-network ( I chose Flannel)
		```sudo sysctl net.bridge.bridge-nf-call-iptables=1```
		
		```kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml```
		
		```Wget https://raw.githubusercontent.com/Microsoft/SDN/1d5c055bb195fecba07ad094d2d7c18c188f9d2d/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
    ```
    
    
		```kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system```




