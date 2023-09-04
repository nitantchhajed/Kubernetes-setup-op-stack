# Kubernetes-setup-op-stack


##Installing Kubernetes
https://phoenixnap.com/kb/install-kubernetes-on-ubuntu

1. `sudo apt update`
2. 
3. `sudo apt install docker.io -y`
4. 
5. `sudo systemctl enable docker`
6. 
7. `sudo systemctl status docker` //check status if docker is running.
8. 
9. `sudo apt-get install -y apt-transport-https ca-certificates curl`
10. 
11. `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`
12. 
13. `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`
14. 
15. `sudo apt-get update`
16. 
17. `sudo apt install kubeadm kubelet kubectl`
18. 
19. `sudo apt-mark hold kubeadm kubelet kubectl`
20. 
21. `kubeadm version` //check version
22. 
23. `sudo swapoff -a`
24. 
25. `sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`
26. 
27. `sudo nano /etc/modules-load.d/containerd.conf`
28. 
      add the lines into the containered.conf file-
      ```overlay
      br_netfilter
      ```
29. `sudo modprobe overlay`
    `sudo modprobe br_netfilter`

30. `sudo nano /etc/sysctl.d/kubernetes.conf`

    Add the following lines:
    ```net.bridge.bridge-nf-call-ip6tables = 1
     net.bridge.bridge-nf-call-iptables = 1
     net.ipv4.ip_forward = 1```

32. `sudo sysctl --system`


    **Assign Unique Hostname for Each Server Node**
- Decide which server to set as the master node. Then enter the command:

`sudo hostnamectl set-hostname master-node`

**Set the kubectl configuration**

`sudo mkdir /etc/systemd/system/kubelet.service.d`

`sudo nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf`

**Then paste this config file -**
```
  # Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

Then save the file and exit.

**Enable the runtme container**

`sudo mkdir /etc/containerd`

`sudo nano /etc/containerd/config.toml`

then update the file if already exists -

```
  #   Copyright 2018-2022 Docker Inc.

#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at

#       http://www.apache.org/licenses/LICENSE-2.0

#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

enabled_plugins = ["cri"]

#root = "/var/lib/containerd"
#state = "/run/containerd"
#subreaper = true
#oom_score = 0

#[grpc]
#  address = "/run/containerd/containerd.sock"
#  uid = 0
#  gid = 0

#[debug]
#  address = "/run/containerd/debug.sock"
#  uid = 0
#  gid = 0
#  level = "info"
```

save the file and exit

**Open the Docker daemon configuration file:**

`sudo nano /etc/docker/daemon.json`

 Append the following configuration block:

 ```
      {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
      "max-size": "100m"
   },

       "storage-driver": "overlay2"
       }
```

save the file and exit.


**Reload the kubelet:**

``` systemctl daemon-reload
      systemctl restart kubelet
```
 Initialize the cluster by typing:

 `sudo kubeadm init --control-plane-endpoint=[master-hostname/ip] --upload-certs`
