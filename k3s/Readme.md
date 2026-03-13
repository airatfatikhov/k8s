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

## Generate Certificate local machine
- create directory
- mkdir /etc/certs
- create certificate <br>
  ``openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=cp-1.test.local/O=cp-1.test.local" ``
- create secret <br>
``kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key``

## Search Token RKE2
``cat /var/lib/rancher/rke2/server/node-token`` 


## Settings Config RKE2