# Creating a cluster
You will get a cluster that has docker installed. Install git, vim, and autocompletion:
`sudo yum install vim bash-completion git`

Name each node for easier reference in the host file `hostname -I`. Then add each host to  `sudo vi /etc/hosts`

Make sure ports are open for master (search docs).

**Control-plane node(s)** 
|Protocol |	Direction |	Port Range |	Purpose	Used By |
|---------|---------|---------|---------|---------|
|TCP    |	Inbound	6443*   |	Kubernetes API server   |	All |
|TCP    |	Inbound	2379-2380   |	etcd server client API  |	kube-apiserver, etcd    |
|TCP    |	Inbound	10250   |	kubelet     |   API	Self, Control plane |
|TCP    |	Inbound	10251   |	kube-scheduler  |	Self    |
|TCP    |	Inbound	10252   |	kube-controller-manager |	Self    |

**Worker node(s)**
Protocol	Direction	Port Range	Purpose	Used By
TCP	Inbound	10250	kubelet API	Self, Control plane
TCP	Inbound	30000-32767	NodePort Services†	All

Note: All kubeadm commands should be run as root... all kubectl command should be run as a user.

As root, on the control plan run:
`kubeadmin init`

Copy the join command to a text file. If you forget and you need to get this token later, you can run `kubeadm token list` and then search for joining a node in the docs... there is a skip validation command near the end. It looks like this:
`kubeadm join --discovery-token r4sbkj.cmqmyw1aprmy0t5c --discovery-token-unsafe-skip-ca-verification 172.31.73.44:6443`

The last ip is your control node.

Install a CNI... you can find an example in the docs by search for kubeadm init and looking at the HA cluster doc. There is a note in there about installing weave. Run that on the control node as a user. It looks like this:
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

On control plane, setup kubeconfig:
`mkdir -p $HOME/.kube`
`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
`sudo chown $(id -u):$(id -g) $HOME/.kube/config`
