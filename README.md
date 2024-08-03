# k8s notes

# node = server

- nodes need container runtime
  - using Kubelet it interact with the container and node.
  - starts the pod with a container inside so its responsible to allocate CPU Ram and more.
- cluster made of multiple worker node the communication using services sort of load balancer. 
- kubproxy will rote to correct node avoiding the request 
so kubelet, kube proxy and container run time must be installed to every K8s system.

- master node will manage all nodes
4 processes run on every master nodes
  - api server using some client. its like master gateway so first step communicate with the master node
  - scheduler like schedule new pod to start on one new node and its intelligent so its will 
    - kubelet responsible for starting the pod
  - controller-manager detect cluster state, for a dead pod for example.
  - etcd key value store for the cluster all the changes saved to etcd, consider a brain of the cluster. state.
- in multiple cluster can have multiple masters.
- master are more important but need less multiple.


## Kubectl with mini Kube
- Kubectl communicate with MiniKube
- Kubectl submit the changes to the MiniKube like create pod, destroy Pod …etc.

- MiniKube
