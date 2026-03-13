# Install Rancher

## Helm Rancher
``helm repo add rancher-latest https://releases.rancher.com/server-charts/latest`` <br>
``kubectl create namespace cattle-system``

## Generate Certificate local machine
- create directory
- mkdir /etc/certs
- create certificate <br>
  ``openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=control-plane-1.test.local/O=control-plane-1.test.local" ``
- create secret <br>
``kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key``

# Export Kubeconfig
``export KUBECONFIG=/etc/rancher/k3s/k3s.yaml``

## Install Rancher Helm
``helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=control-panel-1.test.local \
  --set bootstrapPassword=admin \
  --set ingress.tls.source=my-tls-secret``

## Get Dashboard
''https://control-plane-1.test.local''