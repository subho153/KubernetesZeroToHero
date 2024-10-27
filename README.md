# Kubernetes Installation Process

Step-1 Pre-work :
	
	1.1  Confirm the SG (Firewall) and allow ports for all nodes.
	1.2 set hostname and note down IP address  (private IP only)				[ Both VM ]		
	hostnamectl set-hostname master.tg.com
	hostnamectl set-hostname wn1.tg.com	

vi /etc/hosts 

master  172.31.47.190
w1 		172.31.41.203
w2      172.31.47.242
		
	save and exit 	
-------------------------------------------------------
For exmaple :
	172.31.0.0/24  --->   tg.com 
	172.31.10.231  -->  master.tg.com 
	172.31.4.40  -->  wn1.tg.com
		
-------------------------------------------------------	
		
	1.3  Public and private generate for communication between master and wn1   (ONLY ON MASTER)
	     Goto master node and generate public and private key :
			
	     ssh-keygen  -t  rsa
	    now copy your public key and share with all worker node.			
 	   cat  /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCx6l0ojML6GTOoN4oIWN3B4vLoat9FSs72rw8JauS4hDvxb9Wro17XEG7mu7G5v3pbdmYd3eBipnJzxSz5K9zrGSTh40IiHypF8aRy/UJt2w+P7fR4VstAdm9hti5V+dLA6OtXxp4QiDR1gLvNqKHLBHW4iFqqfVQVrZjyn5On5fKi2AZ8Miee0HRd003Vk2Rls9oIDaagbA0IMDPzd9Q5+MXsFg0ukcmEftZ56No0pU3jD2BtFeT57rYiVLQVb5+eq6qopHZN/uCTDN6RikV6qcUmInrHcZHACSVolEL+8fZipbuVIRfdyo11Yq3xQWVsM/tOQ7tmHX6OdyLVy/zB root@master.tg.com

	1.4  Now login to wn1  machine and open following file with user root
 
		vi  /root/.ssh/authorized_keys
			
		Now paster here your master node public key 
	        save and exit 
			
	1.5   Got master node and try to login from master to wn1 without using password  
	      ssh  root@wn1.tg.com 
-----------------------------------------------------------------------------------------------

Step-2  Kernel argument add / update 		[ Both VM ]

vi   /etc/sysctl.d/k8s.conf		
		
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1


save and exit 

2.1  sysctl --system

-----------------------------------------------------------------------------


Step-3  Docker installation and start the service before k8s installation [ Both VM ]

   yum install docker -y 
   
   systemctl start docker && systemctl enable docker

------------------------------------------------------------------------------

Step-4  Now create repo for yum package download of K8s software (Google)   [ Both VM ]

		
vi /etc/yum.repos.d/k8s.repo
  
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl


Save and exit 

----------------------------------------------------------------------------------  
  
Step-5 Now pull k8s packages from yum / dnf 				[ Both VM ]



	yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes


---------------------------------------------------------------------------------

Step-6  Start kubelet service     			[ Both VM ]


	systemctl start kubelet ; systemctl enable kubelet ; systemctl status kubelet

----------------------------------------------------------------------------------

Step-7  IMPORTANT step 					[ Master node ]

	kubeadm init



7.1 
  
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


-----------------------------------------------------------------------------------
Follow step 7.2 for master node only

For my master node dont copy / paste it , kindly check your own join code. 

----------------------------------------------------------------------------------

7.2

kubeadm join 172.31.47.190:6443 --token maysie.pgd4ovk0juxa0ze6 \
        --discovery-token-ca-cert-hash sha256:e8e42b1cd646d7a8f2afef184618e1c599c7e34ae3477e8ded8348185441d78d


-----------------------------------------------------------------------------------

7.3 Setup Container Network Interface (CNI)    [ master node ]  

			* calico
			* flunnel
			* weavenet    (This one we need setup)
			
Weave Net, often simply called "Weave," is an open source CNI plugin that creates virtual networks for containers hosted within a Kubernetes cluster. Weave provides the virtual networking infrastructure necessary for containers to talk to each other.	

------------------------------------------------------------------------------------	
Weavenet cni location :

https://github.com/weaveworks/weave/blob/master/site/kubernetes/kube-addon.md


you can run this command on MASTER NODE ONLY: 		[ master node ]  


kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

------------------------------------------------------------------------------------

7.4 		(only master node)
	kubectl get no
	
	kubectl get po  --all-namespaces

7.5  Repeat step  step 7.2   (ONLY On  Worker Node(s) )

kubeadm join 172.31.10.231:6443 --token h88whi.yyuw1l80ie6vq561 --discovery-token-ca-cert-hash sha256:1a4f283b0c01c48a495b2aaaa7cad3f4272ea2c79eaf851b071f85bcac08ec6    	
	
Note:   if we forget token / mistmatch  token ?

	kubeadm token create --print-join-command				(only master node)		

--------------------------------------------------------------------------------		
