# Install k8s

## Settings hostname server
``nano /etc/hosts``

## Disable Swap
  ``systemctl mask swap.target`` <br>
  ``192.168.50.100 control-plane-1.test.local`` <br>
  ``192.168.50.104 worker-node-1.test.local``

## Load module core
``nano /etc/modules-load.d/k8s.conf`` <br><br>
``br_netfilter`` <br>
``overlay`` <br> <br>
``modprobe br_netfilter`` <br>
``modprobe overlay``

## Settings sysctl
``nano /etc/sysctl.conf`` <br> <br>
``net.ipv4.ip_forward=1`` <br>
``net.bridge.bridge-nf-call-iptables=1`` <br>
``net.ipv4.ip_nonlocal_bind=1``

## Disable Firewall
``systemctl disable ufw | systemctl disable firewalld``

## Install k8s(Ubuntu)
``sudo snap install k8s --classic --channel=1.35-classic/stable`` <br>

## Install core 
``apt install containerd`` <br>
``or`` <br>
``apt install crio``

## Settings cluster
 - control node init k8s <br> 
   ``sudo k8s bootstrap`` <br>

  - status init k8s <br>
    ``sudo k8s status --wait-ready`` <br>
    ``sudo k8s kubectl get all --all-namespaces``