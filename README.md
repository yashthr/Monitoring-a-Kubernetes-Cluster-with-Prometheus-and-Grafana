# Monitoring-a-Kubernetes-Cluster-with-Prometheus-and-Grafana

Using Helm 3.2.3 Package Manager for Installing Prometheus and Grafana.

Helm is used to install k8s application using commandline

References:

1) https://helm.sh/
2) https://github.com/helm/helm/releases
3) https://github.com/helm/charts
4) https://github.com/helm/charts/tree/master/stable/mysql
5) https://www.udemy.com/course/kubernetes-microservices/learn/lecture/11157662#questions/9078046
6) Udemy Course Reference: https://www.udemy.com/course/kubernetes-microservices/

Steps:

i) Helm Installation

1) Installing a Helm Pod on Bootstrap Instance using link https://github.com/helm/helm/releases https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz

2) sudo mv linux-amd64/helm /usr/local/bin

3) Check helm version using the command: helm version

Optional Installing mysql for learning: #Note there is no relation of mysql to prometheus or graphana
I) Add a stable repo :  helm repo add stable https://kubernetes-charts.storage.googleapis.com/

II) Check the mysql repo : helm search repo mysql

III) Install mysql repo : helm install my-special-installation stable/mysql --set mysqlPassword=password
   #my-special-installation is the pod created for helm with this name.
   
4) To list all helm installation : helm ls

5) To delete helm application : helm delete "application-name" #Example for above helm delete my-special-installation

ii) Prometheus Installation:

We are using prometheus-operator project on github (https://github.com/helm/charts/tree/master/stable/prometheus-operator) which is easy way to install prometheus grapha in one shot

1) Install prometheus named "monitoring" : helm install monitoring stable/prometheus-operator

2) Check pods installated or not : kubectl get pods 

3) To get a sneak peak into the prometheus service we would change the service from "ClusterIP" to "LoadBalancer" on AWS:

   So find "service/monitoring-prometheus-oper-prometheus" and edit ClusterIP to "LoadBalancer" using kubectl edit service/monitoring-prometheus-oper-prometheus and edit type:LoadBalancer.
   You can access the service using loadbalancer port 9090 or check via kubectl get services.
   
   You can now check the metrics available in prometheus. We would use "node_load15" which measure the node load avearaged out in 15 minutes, and Execute to check the metrics send by 4 nodes. You can click "Graph" for basic visual reading.
   
iii) Working with Grapha
  
You can edit the LoadBalancer service to ClusterIP to save costs in AWS for Prometheus, the way is similar to above. And remove the NodePort in ports
  
  1) Editing the service = service/monitoring-graphana to "type: LoadBalancer" using "kubectl edit service/monitoring-graphana" for external access.
  ]
  2) Access the Grapha Console using accessing the Loadbalancer DNS name and its Listener port i.e 80
  
  3) Credentials : username : admin
                   password : prom-operator #You can find it on https://github.com/helm/charts/tree/master/stable/prometheus-operator and find "grafana.adminPassword"
                   
 4) Click on top left corner on page and you will see list of Dashboards.
    Lets check the 'Kubernetes / Compute Resources / Pod' Dashboard, try changing datasource and namespaces,etc
    
    You can have a complete picture of the system usage by viewing 'Kubernetes / Compute Resources / Cluster' Dashboard. You can also   visit 'Node' Dashboard to have node metrics.
    
    The 'USE Method' Dashboard is used for Utilization, Saturation and Errors on the Clusters/Nodes 
    
    
    ------------------------------------------------------------------------------------------------------------------------------------
    Steps needed to get a full set of data from Prometheus:
    
    1: Edit your cluster definition with "kops edit cluster --name ${NAME}"

    2: Find the key called "kubelet:", it will have one entry below it called anonymousAuth: false. You're going to add two further entries so that this section looks like the following:

    kubelet:
      anonymousAuth: false    
      authenticationTokenWebhook: true    
      authorizationMode: Webhook


   3: Save your changes. Then "kops update cluster --yes".

   4: You will now need to do a "rolling update", which is where kops will kill each node in turn, and bring up a new node with the new configuration.

kops rolling-update cluster --yes

Sometimes this process hangs - not a problem. Just wait five minutes, and if no progress, "control-C", re-run the command again. Keep doing this until the rolling update reports there is nothing to do.
