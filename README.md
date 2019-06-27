# k8s

 1.	Create a Highly available Kubernetes cluster manually using Google Compute Engines (GCE). Do not create a Kubernetes hosted solution using Google Kubernetes Engine (GKE). Use Kubeadm(preferred)/kubespray. 

o/p : Below the highly available kubernetes cluster :
samdhina_x11@cloudshell:~/kubernetes/cluster (smooth-loop-245005)$ kubectl get nodes
NAME                           STATUS                     ROLES    AGE   VERSION
kubernetes-master              Ready,SchedulingDisabled   <none>   32m   v1.15.0
kubernetes-minion-group-8t0k   Ready                      <none>   31m   v1.15.0
kubernetes-minion-group-w65t   Ready                      <none>   31m   v1.15.0
  
------------------------------------------------------------------------------------------------------------------------------------  
  3.	Create a development namespace.
  
  o/p :
  
  samdhina_x11@cloudshell:~/kubernetes/cluster (smooth-loop-245005)$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   37m
development       Active   12m
kube-node-lease   Active   37m
kube-public       Active   37m
kube-system       Active   37m
----------------------------------------------------------------------------------------------------------------------------------
 4.	Deploy guest-book application  in the development namespace.
 
 Here I deployed backend as redis server with master and slave , And Frontend  is guestbook to acess this i have exposed the service using type : LoadBalancer .
 Pods :
 samdhina_x11@cloudshell:~/k8s/application/guestbook (smooth-loop-245005)$ kubectl get pods -n development
NAME                            READY   STATUS    RESTARTS   AGE
frontend-678d98b8f7-bxkkn       1/1     Running   0          85m
frontend-678d98b8f7-gtb5j       1/1     Running   0          85m
frontend-678d98b8f7-l69ws       1/1     Running   0          85m
redis-master-7dd774b664-hlw9g   1/1     Running   0          3h58m
redis-slave-546fc99d45-cf8vd    1/1     Running   0          94m
redis-slave-546fc99d45-nd8qf    1/1     Running   0          94m
SVC :
samdhina_x11@cloudshell:~/k8s/application/guestbook (smooth-loop-245005)$ kubectl get svc  -n development
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
frontend       LoadBalancer   10.0.216.162   172.17.0.3    80:32461/TCP   18m
redis-master   ClusterIP      10.0.84.248    <none>        6379/TCP       109m
redis-slave    ClusterIP      10.0.68.119    <none>        6379/TCP       94m
 
 Deployments
samdhina_x11@cloudshell:~/k8s/application/guestbook (smooth-loop-245005)$ kubectl get deployments  -n development
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
frontend       3/3     3            3           90m
redis-master   1/1     1            1           4h2m
redis-slave    2/2     2            2           98m

samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get --all-namespaces pods
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
development   frontend-678d98b8f7-bxkkn                   1/1     Running   0          57m
development   frontend-678d98b8f7-gtb5j                   1/1     Running   0          57m
development   frontend-678d98b8f7-l69ws                   1/1     Running   0          57m
development   redis-master-7dd774b664-hlw9g               1/1     Running   0          3h30m
development   redis-slave-546fc99d45-cf8vd                1/1     Running   0          66m
development   redis-slave-546fc99d45-nd8qf                1/1     Running   0          66m

--------------------------------------------------------------------------------------------------------------------------------------





