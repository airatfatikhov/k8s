## Создание приложения
- [x] Необходимо создать проект (к примеру whoami)
- [x] Затем создаем namespace либо создаем его через манифест <br>
  ````apiVersion: v1
      kind: Namespace
      metadata:
        name: whoami
- [x] Создаем Pod и Deployment <br>
  ````apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: whoami
        labels:
          app: whoami
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: whoami
        template:
          metadata:
          labels:
            app: whoami
        spec:
          containers:
          - name: whoami-container
            image: traefik/whoami:v1.8.0 # Use a specific image version
            ports:
              - name: web
                containerPort: 80
            livenessProbe:
              httpGet:
                path: /health # Use the health endpoint
                port: web
              initialDelaySeconds: 5
              periodSeconds: 10
            readinessProbe:
              httpGet:
                path: /health # Use the health endpoint
                port: web
           initialDelaySeconds: 5
           periodSeconds: 10
- [x] Создаем сервис type NodePort <br>
  ````apiVersion: v1
      kind: Service
      metadata:
        name: whoami-service
        namespace: whoami
      spec:
        type: NodePort
        selector:
          app: whoami
        ports:
          - protocol: TCP
            port: 80
            targetPort: 80
            nodePort: 30007

- [x] Запускаем манифесты <br>
```` 
      kubectl apply -f deployment.yaml
      kubectl apply -f service.yaml
````      
- [x] Проверяем <br>
   ````
    curl http://worker-node-1.test.local:30007/
    