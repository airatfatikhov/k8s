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

## Settings Config RKE2