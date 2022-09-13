Kubernetes api server constantly dies after restart on Ubuntu 22:
    The connection to the server 192.168.1.135:6443 was refused - did you specify the right host or port?

The problem is in containerd default config, SystemdCgroup is false by default, need to set it to true:
    SystemdCgroup = true

After the fix kubelet is stable.