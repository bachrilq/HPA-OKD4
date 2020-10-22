# HPA-OKD4

```bash
#Create New Project
oc new-project hpa

#Create new deployment
mkdir HPA
cd HPA
vim hpa-deploy.yaml
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: twalter/openshift-nginx
        resources:
          requests:
            cpu: 400m
            memory: 100Mi
        ports:
        - containerPort: 8081
...

vim svc-hpa.yaml
...
apiVersion: v1
kind: Service
metadata:
  name: hpa-cluster
  namespace: hpa
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8081
...

#deploy 
oc create -f hpa-deploy.yaml
oc create -f svc.yaml

#expose service
oc expose service hpa-cluster --port=8081

#create HPA
oc autoscale deployment/nginx --min 1 --max 5 --cpu-percent=5

#test benchmark
ab -kc 20 -t 100000 http://hpa-cluster-hpa.apps.openshift.instructor.io/
```
