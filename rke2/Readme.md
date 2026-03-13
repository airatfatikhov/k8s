# Install RKE2 and RANCHER

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