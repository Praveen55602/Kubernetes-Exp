Test 4:
we'll do the following

1. create a yaml file with multiple deployments, simulating 4 microservices we'lll have 1 service for each.
2. now the 2 services will have nodePort type so we can have external communication with them, while the other 2 will be of type clusterIp so that we can only call them from within the cluster.

3. create cluster with updated nodes
   kind create cluster --name multi-node-cluster --config multi-node-cluster.yaml
4. see all the nodes
   kubectl get nodes
5. apply the multi-service-deployment
   kubectl apply -f multi-service-dep.yaml
6. use this command to see where each pod is allocated
   kubectl get pods
