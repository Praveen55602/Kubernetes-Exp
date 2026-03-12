# Local Kubernetes HA(high availability) Cluster Simulation on Windows

This repository provides the configuration and documentation required to simulate a highly available (HA) Kubernetes control plane locally on a Windows machine.

This multi-node `kind` setup is an ideal sandbox for testing distributed systems. It allows you to safely experiment with control plane leader election, deploy multiple worker pods, test gRPC communication across nodes, and observe how worker pools react when a node drops offline.

## 🛠️ Prerequisites & Tooling

To run this simulation, you need the following tools installed on your Windows machine:

1. **Docker Desktop:** The container runtime engine.
   - _Requirement:_ Must be installed and actively running (WSL 2 backend is recommended).
2. **Windows Package Manager (`winget`):** The built-in Windows command-line tool.
3. **`kind` (Kubernetes in Docker):** A tool for running local Kubernetes clusters using Docker container "nodes".
   ```powershell
   winget install Kubernetes.kind
   ```
4. **`kubectl` The Kubernetes command-line tool used to communicate with the cluster's API server:**
   ```powershell
   winget install Kubernetes.cli
   ```
5. Configuration file - ha-cluster.yaml

Execution Steps: The Leader Election Experiment

1. kind create cluster --name ha-cluster --config ha-cluster.yaml
2. kubectl get nodes
3. kubectl describe lease kube-controller-manager -n kube-system(Look for the Holder Identity field. It will display the winning node's name followed by a unique session UUID)
4. Simulate a Node Crash - docker stop <leader-node-name>
5. Observe the Failover - Wait exactly 15 seconds (the default LeaseDurationSeconds lock time). In your first terminal, run the describe command again:
   kubectl describe lease kube-controller-manager -n kube-system
6. Cleanup - kind delete cluster --name ha-cluster
