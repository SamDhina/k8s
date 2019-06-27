# k8s

 1.	Create a Highly available Kubernetes cluster manually using Google Compute Engines (GCE). Do not create a Kubernetes hosted solution using Google Kubernetes Engine (GKE). Use Kubeadm(preferred)/kubespray. 

o/p : Below the highly available kubernetes cluster :
samdhina_x11@cloudshell:~/kubernetes/cluster (smooth-loop-245005)$ kubectl get nodes
NAME                           STATUS                     ROLES    AGE   VERSION
kubernetes-master              Ready,SchedulingDisabled   <none>   32m   v1.15.0
kubernetes-minion-group-8t0k   Ready                      <none>   31m   v1.15.0
kubernetes-minion-group-w65t   Ready                      <none>   31m   v1.15.0
  
  
  3.	Create a development namespace.
  
  o/p :
  
  samdhina_x11@cloudshell:~/kubernetes/cluster (smooth-loop-245005)$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   37m
development       Active   12m
kube-node-lease   Active   37m
kube-public       Active   37m
kube-system       Active   37m

