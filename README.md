Test 3:
we'll do the following

1. create a multinode setup with 3 control plane and 3 worker nodes
2. create 2 pods and watch them get scheduled by scheduler on the worker nodes
3. now we'll scale up the replicas and have 10 of them and see the scheduler in action
4. then we'll delete one of the pod and see the kubernetes healing in action.

steps

1. delete the old cluster if exists
   kind delete cluster --name ha-cluster
2. create new cluster with updated nodes
   kind create cluster --name multi-node-cluster --config multi-node-cluster.yaml
3. see all the nodes
   kubectl get nodes
4. apply the test-deployment
   kubectl apply -f test-deployment.yaml
5. use this command to see where each pod is allocated
   kubectl get pods -o wide
6. scale the replica count
   kubectl scale deployment test-pod --replicas=10
7. use command - kubectl scale deployment test-pod --replicas=10 and see all the replicas created and assigned to worker nodes
8. in one terminale run this command to see live updates for the pods
   kubectl get pods -w
   In another one use this command to delete one of the pod and see the other to watch kubernetes heal in action
   kubectl delete pod <pod name> (delete one of the pods)
