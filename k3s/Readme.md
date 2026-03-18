# Install k3s and RANCHER

## Install k3s of Scripts
 - link scripts <br>
     ``https://get.k3s.io/``
 - copy and create directory on server <br>
     ``mkdir /etc/k3s``
 - create file for install scripts <br>
     ``touch install_k3s.sh``
 - launch install <br>
     ``bash install_k3s.sh``

## Disable Swap
  ``systemctl mask swap.target``

## Token K3S
  ``cat /var/lib/rancher/k3s/server/node-token``

## Generate Certificate local machine for RANCHER
 - create directory
 - mkdir /etc/certs
 - create certificate <br>
   ``openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=cp-1.test.local/O=cp-1.test.local" ``
 - create secret <br>
 ``kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key``

## Add Agent for K3S
``K3S_URL=https://control-plane-1.test.local:6443 K3S_TOKEN=K101a50a06d0369761903d9300f308201c0d17efee8e7e3e17d56b90e8b54927906::server:8b7365c4a0893a2b60cf3b3d1157b000 ./install_k3s.sh``

## Settings Taints
  - create file <br>
   ``touch /etc/rancher/k3s/config.yaml``
  - insert strings <br>
  ``node-taint:
   "node-role.kubernetes.io/control-plane=true:NoSchedule"
   "node-role.kubernetes.io/master=true:NoSchedule"``

## Install Offline
  - download images <br>
   ``curl -L -o k3s-airgap-images-amd64.tar.zst "https://github.com/k3s-io/k3s/releases/download/v1.33.3%2Bk3s1/k3s-airgap-images-amd64.tar.zst"``
  - create folder
   ``sudo mkdir -p /var/lib/rancher/k3s/agent/images/`` <br>
   ``sudo cp k3s-airgap-images-amd64.tar.zst /var/lib/rancher/k3s/agent/images/k3s-airgap-images-amd64.tar.zst`` <br>
  - create cache file <br>
   ``touch /var/lib/rancher/k3s/agent/images/.cache.json`` 
   - download binary k3s <br>
  ``sudo curl -Lo /usr/local/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.33.3%2Bk3s1/k3s``
    - link bin <br>
  ``sudo chmod +x /usr/local/bin/k3s``
   - install offline server <br>
   ``INSTALL_K3S_SKIP_DOWNLOAD=true ./install_k3s.sh``
  - install offline agent

## Delete k3s
 - delete server
 - cd /usr/local/bin/
 - bash k3s-uninstall.sh
 - delete agent
 - cd /usr/local/bin/
 - bash k3s-agent-uninstall.sh