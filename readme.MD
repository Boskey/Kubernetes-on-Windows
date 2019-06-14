# Instructions to setup Kubernetes Cluster with Windows Nodes

###### These instructions are for attaching a Windows Node to an existing Kubernetes cluster running Flannel Network on vSphere ( although can be extended to other Infrastructure platforms

Step1. Create a Kubernetes cluster with Linux node

Step2. Add a Windows node to the cluster

## Kubeadm to deploy Kubernetes with a Linux Node on vSphere

	1. Deploy a Ubuntu 16.0 + image
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
	
```mkdir -p $HOME/.kube```
	
```sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```

```sudo chown $(id -u):$(id -g) $HOME/.kube/config```

	8. Install Pod-network ( Have to choose Flannel for now as that is what the Windows supports currently)
	
```sudo sysctl net.bridge.bridge-nf-call-iptables=1```
		
```kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml```

```Wget https://raw.githubusercontent.com/Microsoft/SDN/1d5c055bb195fecba07ad094d2d7c18c188f9d2d/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml```

 ```kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system```
 

## Adding Windows Worker nodes to k8's cluster

1. Deploy Windows 2019 server ( i simply had a template /OVA that i worked with)

2. Run below powershell commands
```
Get-WindowsOptionalFeature -Online -FeatureName TelnetClient            
Enable-WindowsOptionalFeature -Online -FeatureName TelnetClient            
Get-WindowsOptionalFeature -Online -FeatureName TelnetClient
$PSDefaultParameterValues = @{"*WindowsOptionalFeature:Online"=$true}
```
3. Deploy Docker
```
Enable-WindowsOptionalFeature -FeatureName Containers
```
_Will need to reboot OS_
```
Restart-Computer -Force
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Start-Service docker
```
4. Create a “Kubernetes for Windows” directory to store Kubernetes binaries as well as any deployment scripts and config files.

```mkdir c:\k```

5.Copy the Kubernetes certificate file $HOME/.kube/config from the Linux controller to this new C:\k directory on your Windows node.

6. Downlaod Kubernetes Binaries with Windows support ( link to v.1.14)
https://dl.k8s.io/v1.14.1/kubernetes-node-windows-amd64.tar.gz

7.Save the below content in a file with .bat extension, right click run as admin
```
sc config wuauserv start= auto
sc config bits start= auto
sc config DcomLaunch start= auto
net stop wuauserv
net start wuauserv
net stop bits
net start bits
net start DcomLaunch
```
8. Get the start.ps1 script to initialize Windows node
```
cd c:\k
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```
9. Execute script
```
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode overlay  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```

At this point if you run `kubectl get nodes` you should see windows node attached to the cluster






