## Instruction on how to install prometheus grafana loki

Start your local kubernetes cluster

```sh
minikube start --driver=virtualbox
```
![start-k8-cluster](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/9ee14a39-d4b7-408c-8c41-fca4c88e43cb)

Create 2 namespaces for prometheus & loki

```sh
kubectl create ns <name-of-namespace>
```
check your helm repo

```sh
helm repo list
```
![helm-repo-list](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/eec91e59-618f-478b-959e-b7b2699e625f)

or you can also add
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana-loki https://grafana.github.io/helm-charts
```
update your helm repo
```sh
helm repo update
```
Search your helm repo 
```sh
helm search repo prometheus-community
```

Install the kube-prometheus-stack and loki

```sh
helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus
helm install loki grafana/loki-stack -n loki
```

As a part of you stack you get grafana as well as Alertmanager, wait till all the pods are in running state
```sh
kubectl get po -n prometheus
kubectl get po -n loki
```

now expose your services to view the gui

```sh
kubectl -n prometheus expose service prometheus-kube-prometheus-prometheus --type=NodePort --target-port=9090 --name prometheus-svc-ext

kubectl -n prometheus expose service prometheus-kube-prometheus-alertmanager --type=NodePort --target-port=9093 --name=alertmanager-svc-ext

kubectl -n prometheus expose service prometheus-grafana --type=NodePort --target-port=3000 --name=grafana-svc-ext

kubectl -n loki expose service loki --type=NodePort --target-port=3100 --name=loki-svc-ext

```
![prometheus-grafana-integration](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/61277853-fcf1-4e2b-a938-d4c701dfae64)

Grafana Dashboard after adding data sources 

![grafana-dashboard-id(9873)](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/372f5a15-ae48-4c96-b58f-81aa9b303853)


create a deployment to check if we are getting logs for the containers

```sh
kubectl create deployment nginx-deployment --image=nginx --replicas=4
```
After nginx deployment we can see loki logs 

![loki-logs](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/8d2e37cd-5784-4159-ab4b-b8b9b789052e)
![loki-logs-nginx](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/eca9fc5a-0c35-4a03-94df-2ccd00177be7)

Configure slack to receive notification

https://api.slack.com/ --> yor apps --> create new app --> From Scratch --> after creating --> incoming weebhooks--> Add new webhook to workspace

Upgrade the helm chart to receieve alerts
```sh
helm upgrade --reuse-values -f /home/supriyo/alertmanager-config.yaml prometheus prometheus-community/kube-prometheus-stack -n prometheus
```
![slack-notification](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/f3a12209-d6e0-4aea-8adf-e0ace41356aa)

Go to Komodor UI and install the helm mentioned or generated when adding cluster

![komodor](https://github.com/Supriyo-Roy/jenkins-demo/assets/101391805/65146f91-1a41-4af1-8535-ff106048f4b3)

