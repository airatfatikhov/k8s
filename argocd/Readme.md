# Install ArgoCd

## Download manifest
- Download manifest https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
- Save manifest in file
- Create file and folder
``mkdir /etc/argocd/`` <br>
``touch /etc/argocd/install.sh`` <br>
``nano /etc/argocd/install.sh``
- create namespace <br>
``kubectl create namespace argocd``
- launch apply manifest
``kubectl apply -n argocd -f install.yaml``

## Get install pods
``kubectl get pods -n argocd``

## Access to ArgoCd
``kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`` <br>
``kubectl get svc -n argocd argocd-server`` <br>
``kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d``