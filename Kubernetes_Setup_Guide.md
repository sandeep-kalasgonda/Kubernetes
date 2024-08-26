# Setting Up a Kubernetes Cluster with Multipass

## Prerequisites
- **Multipass**: Ensure you have Multipass installed on your macOS. Follow the [Multipass installation guide](https://multipass.run/docs/installing-on-macos) if needed.
- **Access to a terminal**: You will run commands in the terminal.
- **clone this repo first
- git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git


---

## Step 1: Destroy Existing Vagrant Setup

To remove any previous Vagrant setups, execute the following command in the terminal:

```bash
./delete-virtual-machine.sh
```

> **Screenshot**: Add a screenshot of the terminal showing the command execution and output.

---

## Step 2: Create Master Node and Worker Node

To deploy one master node and one worker node, use the following command:

```bash
./deploy-virtual-machines.sh
```

> **Screenshot**: Capture the terminal output of the deployment process.

---

## Step 3: Install Docker and Containerd on Both Nodes

### 3.1 Add Docker's Official GPG Key

Run the following commands on both nodes to install Docker and containerd:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 3.2 Add the Docker Repository

Execute this command to add the Docker repository to your Apt sources:

```bash
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 3.3 Install Docker

Install Docker with the following command:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin
```

### 3.4 Verify Docker Installation

Run the hello-world image to verify Docker is installed correctly:

```bash
sudo docker run hello-world
```

> **Screenshot**: Include a screenshot of the successful output from the hello-world image.

### 3.5 Check Containerd Status

Verify that containerd is running:

```bash
systemctl status containerd
```

> **Screenshot**: Capture the output of the status command confirming that containerd is running.

---

## Step 4: Configure the Systemd Cgroup Driver

Edit the containerd configuration:

```bash
sudo vi /etc/containerd/config.toml
```

Replace the contents with:

```toml
[plugins.'io.containerd.grpc.v1.cri'.containerd.runtimes.runc]
  [plugins.'io.containerd.grpc.v1.cri'.containerd.runtimes.runc.options]
    SystemdCgroup = true
```

Save and exit the file.

---

## Step 5: Install kubeadm, kubectl, and kubelet

### 5.1 Update Apt Package Index

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 5.2 Download Kubernetes Signing Key

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 5.3 Add Kubernetes Apt Repository

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 5.4 Install Kubelet, Kubeadm, and Kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## Step 6: Initialize Kubernetes Cluster

### 6.1 Get Node IP Address

Run the following command to get the IP address of the master node:

```bash
ip add
```

### 6.2 Initialize Kubeadm

Run the kubeadm init command with the following format, replacing `192.168.64.15` with your master node's IP address:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.64.15
```

> **Screenshot**: Take a screenshot of the output from the kubeadm init command.

---

### 6.3 Configure kubectl

Run the following commands to set up kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 6.4 Verify Cluster Setup

To check the cluster status, use:

```bash
kubectl cluster-info
kubectl get pod --all-namespaces
```

> **Screenshot**: Include screenshots showing the cluster information and pods.

---

## Step 7: Join Worker Node to Cluster

### 7.1 Get Join Command

Run the following command to get the join command:

```bash
kubeadm token create --print-join-command
```

### 7.2 Join Worker Node

SSH into the worker node and execute the join command generated in the previous step:

```bash
sudo kubeadm join 192.168.56.2:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<your-hash>
```

> **Screenshot**: Capture the output of the join command execution.

---

## Step 8: Deploy NGINX Pod

To create an NGINX pod in your cluster, run:

```bash
kubectl run nginx --image=nginx
kubectl get pod
```

### 8.1 Verify Pod Creation

Check the details of the NGINX pod:

```bash
kubectl describe pod nginx
```

> **Screenshot**: Include screenshots showing the pod status and details.

---

## Conclusion

You have successfully set up a Kubernetes cluster with a master node and a worker node using Multipass. Docker and containerd were installed and configured, followed by the installation of kubeadm, kubectl, and kubelet. Finally, an NGINX pod was deployed on the cluster.

---
