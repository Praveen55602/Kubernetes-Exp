Test 2:
We are going to deploy a couple of dummy pods to simulate peers in a network, using a standard Go image.

This experiment will perfectly demonstrate the difference between the Controller Manager (which your new leader is running) and the Scheduler.

Because we only spun up control-plane nodes in our cluster, we are going to run into an intentional roadblock that will show us exactly how Kubernetes protects its management nodes.

We will create a simple Deployment that asks Kubernetes to spin up two Go-based containers.

1. first we create our test-deployment.yaml file
2. apply configuration to cluster by running:
   kubectl apply -f p2p-deployment.yaml
3. run: kubectl get pods
   we'll see the status for both pods will be pending

To find out exactly why the Scheduler is refusing to place them on a node, we can ask Kubernetes to describe the events of one of those pods. Run this (replacing <pod-name> with one of the names from your previous output):

Scroll to the very bottom of the output to the Events: section. You will see a Warning from the default-scheduler saying something like: 0/3 nodes are available: 3 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }.

4. The control plane nodes have a "Do Not Disturb" sign (a taint) to ensure they have enough processing power to manage the cluster without your worker pods getting in the way.

5. Since we are just simulating this locally and don't have dedicated worker nodes attached to this cluster yet, we can simply remove that "Do Not Disturb" sign from all our nodes so the scheduler can do its job.
   Run this command to strip the control-plane taint from every node in the cluster:
   kubectl taint nodes --all node-role.kubernetes.io/control-plane-
   (Note: that minus sign - at the very end of the command is important—it tells Kubernetes to remove the taint).
