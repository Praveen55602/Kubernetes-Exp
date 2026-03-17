Test 5:
Rolling update to an exising deployment
basically in rolling updates we make changes to the exising deployment and kubernetes safely applies those to the pods without any downtime
this is done by first creating new pod with the changes if the new pod is created succesfully then the old one is safely terminated and new one replaces it, this way changes happen gradually without any down time.

if anything fails at any moment the entire rollout is halted.

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
7. now we'll change the image for the gateway-app to use a nginx:alpine image and see the rollout
   kubectl set image deployment/gateway-app web=nginx:alpine
   use below command to see the rollout happening.
   kubectl get pods -l app=gateway
8. you'll see that the rollout fails, it tries with the first pod and it fails so the entire rollout it halted until that pod is fixed.
   Q: Why does the new pod show "3 RESTARTS" in only 61 seconds? What is happening inside?
   A: You are entering a state known as CrashLoopBackOff.

   If you used a basic curl image, it doesn't have a long-running web server. The container boots up, realizes it has no specific curl command to run, and immediately exits.

   The kubelet (the node's agent) sees the container exit and says, "Oh no, it crashed! Let me restart it." It boots it up again. The container immediately exits again. The kubelet restarts it again.

   Eventually, Kubernetes realizes this is a lost cause and starts adding a delay between restarts (10 seconds, then 20, then 40) so it doesn't burn out the node's CPU. This is the "BackOff" phase.

9. to see the logs of a perticular pod we have 2 commands -
   a. kubectl logs <pod-name>(get it from previous command)
   b. kubectl describe pod <pod-name>
10. The Mismatch (Image vs. Arguments)
    The Concept: When we used the command kubectl set image, we told the Deployment manager to swap the container's core software (the image) from http-echo to nginx. However, a Deployment blueprint contains more than just the image—it also contains startup arguments, environment variables, and ports.

    If you swap the image but don't clear out the old startup arguments, the new software will be forced to boot up using the old software's instructions.

    Q: Why is the Nginx pod throwing an /docker-entrypoint.sh: exec: line 47: illegal option -t error and crashing?

    A: Because Nginx is trying to read the instructions meant for http-echo!

    Look closely at the describe pod output you pasted. Right under the nginx:alpine image line, you will see this:
    Args:
    -text=Public Gateway: Search for files here
    -listen=:8080

When Nginx boots up, its internal startup script (docker-entrypoint.sh) receives -text=.... Nginx doesn't know what -text means. It sees the -t and assumes you are trying to pass an illegal command-line flag. It panics and immediately throws an Exit Code: 2 (which means "misuse of shell built-ins").

Because it crashed, the kubelet restarted it, it crashed again, and you hit that CrashLoopBackOff state.

11. The Concept: Now you have a broken Deployment. You could open the YAML, delete the args, and re-apply it. But in a production environment at Amazon, if you push a bad update that crashes, you don't waste time fixing it while users are waiting—you instantly rewind time to the last known working state.

Kubernetes keeps a hidden history of your Deployment configurations specifically for this reason.
Because Kubernetes halted the rollout (leaving your two old http-echo pods running safely), rolling back is completely seamless. Tell the Deployment manager to throw away the broken Nginx update and revert to its previous revision by running:

kubectl rollout undo deployment/gateway-app

12. Imperative Commands vs. Declarative Blueprints

The Concept: Up until now, we have been using "Imperative" commands (like kubectl scale and kubectl set image). Imperative commands are like giving the cluster step-by-step verbal instructions: "Do this one specific thing right now." While imperative commands are fantastic for quick local debugging, they are dangerous in production. If you need to change an image and change its arguments, doing it imperatively requires typing two separate commands. That leaves a window of time where the configuration is mismatched, causing the exact crash you just experienced.

13. To completely avoid this, large companies use Declarative management. They write the final, perfect state into a YAML file and tell Kubernetes, "Make reality match this exact blueprint."

14. we'll use declarative blueprint and update our deployment file and the apply it. make changes to deployment file and then use command
    kubectl apply -f mutli-service-dep.yaml
    and check the pod state changing in real time using command
    kubectl get pods -l app=gateway -w

we've successfully rolled out an update in the deployment.
