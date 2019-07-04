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

5.	Install and configure Helm in Kubernetes

Below is the o/p that installation and configuration of helm in kubernetes .

samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get pods -n kube-system |grep tiller
tiller-deploy-7bf78cdbf7-xw26c              1/1     Running   0          18m
samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get deploy,svc tiller-deploy -n kube-system
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/tiller-deploy   1/1     1            1           18m

NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
service/tiller-deploy   ClusterIP   10.0.204.83   <none>        44134/TCP   18m
 
 -----------------------------------------------------------------------------------------------------------------------------------
 6.	Use Helm to deploy the application on Kubernetes Cluster from CI server.
 
 Below is the docker-registry application deployed using helm
 
 samdhina_x11@cloudshell:~ (smooth-loop-245005)$ helm install stable/docker-registry
NAME:   funny-robin
LAST DEPLOYED: Thu Jun 27 19:16:41 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                                DATA  AGE
funny-robin-docker-registry-config  1     1s

==> v1/Pod(related)
NAME                                          READY  STATUS             RESTARTS  AGE
funny-robin-docker-registry-866b8f6644-ljflt  0/1    ContainerCreating  0         1s

==> v1/Secret
NAME                                TYPE    DATA  AGE
funny-robin-docker-registry-secret  Opaque  1     1s

==> v1/Service
NAME                         TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
funny-robin-docker-registry  ClusterIP  10.0.95.79  <none>       5000/TCP  1s

==> v1beta1/Deployment
NAME                         READY  UP-TO-DATE  AVAILABLE  AGE
funny-robin-docker-registry  0/1    1           0          1s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=docker-registry,release=funny-robin" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:5000
  
  samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get pods 
NAME                                           READY   STATUS    RESTARTS   AGE
funny-robin-docker-registry-866b8f6644-ljflt   1/1     Running   0          99s
samdhina_x11@cloudshell:~ (smooth-loop-245005)$
----------------------------------------------------------------------------------------------------------------------------------
7.	Create a monitoring namespace in the cluster.

samdhina_x11@cloudshell:~/k8s/application/guestbook (smooth-loop-245005)$ kubectl create namespace monitoring
namespace/monitoring created
samdhina_x11@cloudshell:~/k8s/application/guestbook (smooth-loop-245005)$ kubectl get namespace
NAME              STATUS   AGE
default           Active   7h26m
development       Active   7h1m
kube-node-lease   Active   7h26m
kube-public       Active   7h26m
kube-system       Active   7h26m
monitoring        Active   4s

-----------------------------------------------------------------------------------------------------------------------------------
8.	Setup Prometheus (in monitoring namespace) for gathering host/container metrics along with health check status of the application. 

Prometheus pod and grafana pods have been created in monitoring namespace with alll the metrics included launched the prometheus dashboard outside the GCP console .



samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get pods -n monitoring
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          25s
alertmanager-main-1                    2/2     Running   0          25s
alertmanager-main-2                    2/2     Running   0          25s
grafana-565dc89fbb-cjz9b               1/1     Running   0          25s
kube-state-metrics-c8c5f4cc9-mmkfr     4/4     Running   0          25s
node-exporter-4vb5h                    2/2     Running   0          25s
node-exporter-ff9sp                    2/2     Running   0          25s
prometheus-adapter-7c76f9d558-p8xbw    1/1     Running   0          25s
prometheus-k8s-0                       3/3     Running   1          25s
prometheus-k8s-1                       3/3     Running   1          25s
prometheus-operator-7447bf4dcb-2ncld   1/1     Running   5          25s

Note : Prometheus and Grafana Dashboard Screen shots have been attached in this Link for your reference .



------------------------------------------------------------------------------------------------------------------------------------
9.	Create a dashboard using Grafana to help visualize the Node/Container/API Server etc. metrices from Prometheus server. Optionally create a custom dashboard on Grafana .

Grafana Pod Created and Forwarded Port to access the dashboard from outside the GCP and attached Screen shot of GRAFANA custom dashboard and Nodes , API server , container metrics collected from prometheus server to grafana dashboard .

samdhina_x11@cloudshell:~ (smooth-loop-245005)$ kubectl get pods -n monitoring
grafana-565dc89fbb-cjz9b               1/1     Running   0          25s

Note : Grafana pod deployment YAML script have been uploaded in this Link for your reference .
------------------------------------------------------------------------------------------------------------------------------------
 10.	Setup log analysis using Elasticsearch, Fluentd (or Filebeat), Kibana.
 
 Below is the log analysis using Elasticsearch in kubernetes cluster :
 
 Elasticsearch Pod o/p :
 
 samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$ kubectl get pods |grep esnode
esnode-0                                       1/1     Running   0          25m
esnode-1                                       1/1     Running   0          24m
samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$

ES SVC o/p :

samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$ kubectl get svc |grep elas
elasticsearch-cluster         ClusterIP      None          <none>        9300/TCP       29m
elasticsearch-loadbalancer    LoadBalancer   10.0.222.34   34.66.142.138     80:32316/TCP   27m
samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$
 
 Below is the sample Query the cluster for information about its nodes.
 
 samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$ curl 127.0.0.1:32316/_cat/indices?v
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   bank        GVgO92fHSm6Tglm4I3lx5w   5   1       1000            0    950.1kb          475kb
green  open   shakespeare Y86CuJF2QcO38NycTPzYBg   5   1     111396            0     46.6mb         23.4mb

Checking the available indices:
samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$ curl 127.0.0.1:32316/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.64.2.90           60          84  16    0.10    0.34     0.48 mdi       *      0snC6yZ
10.64.2.91           64          84  16    0.10    0.34     0.48 mdi       -      WlqPHi6
samdhina_x11@cloudshell:~/elastic (smooth-loop-245005)$

Note : SVC , deployment config YAML files for elastic search have been uploaded for your reference 
---------------------------------------------------------------------------------------------------------------------


