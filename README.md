App for the Microservices 

![Chassis](chassis.png)
![Services](services.png)
![Monitoring](mon.png)
![Logs](logs.png)

Python framework for building microservices https://www.nameko.io 


```
Edit Dockerfiles

```


Edit services/config.yaml
```
AMQP_URI: amqp://${RABBIT_USER:guest}:${RABBIT_PASSWORD:guest}@${RABBIT_HOST:rabbitmq}:${RABBIT_PORT:5673}/
RABBIT_CTL_URI: http://guest:guest@rabbitmq:15673
WEB_SERVER_ADDRESS: '0.0.0.0:5000'
RPC_EXCHANGE: 'simplebank-rpc'
```
```
docker-compose build
```
```
Create dockerhub repos @https://hub.docker.com/u/davarski/

for i in `docker images|grep chapter12k8s|awk '{print $1}'`; do docker tag $i:latest davarski/$i:latest;done

for i in `docker images|grep chapter12k8s|awk '{print $1}'`; do docker push $i:latest;done
```
```
generating k8s from docker-compose.yaml
kompose convert 
INFO Kubernetes file "account-transactions-service.yaml" created 
INFO Kubernetes file "fees-service.yaml" created  
INFO Kubernetes file "gateway-service.yaml" created 
INFO Kubernetes file "market-service.yaml" created 
......

minikube ssh --- sysctl -w vm.max_map_count=262144 to fix ERROR: [1] bootstrap checks failed - elasticsearch

Edit *-deployment.yaml and setup image: - image: davarski/chapter12k8s_{service}:latest

Deploy ... kubectl create -f account-transactions-deployment.yaml,account-transactions-service.yaml,...

kubectl create -f ./tmp

for i in `ls -1 *service*`; do kubectl create -f  $i;done
for i in `ls -1 *deplo*`; do kubectl create -f  $i;done

$ kubectl get svc --namespace=default
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)               AGE
account-transactions   ClusterIP   10.108.41.27     <none>        5003/TCP              11m
alerts                 ClusterIP   10.111.183.2     <none>        5006/TCP              11m
elasticsearch          ClusterIP   10.106.238.14    <none>        9200/TCP              11m
fees                   ClusterIP   10.103.15.233    <none>        5004/TCP              11m
fluentd                ClusterIP   10.97.154.90     <none>        24224/TCP,24224/UDP   7m
gateway                ClusterIP   10.109.8.206     <none>        5001/TCP              11m
grafana                NodePort    10.97.13.168     <none>        3900:30623/TCP        11m
hello                  ClusterIP   10.109.91.30     <none>        80/TCP                1d
kibana                 NodePort    10.99.57.230     <none>        5601:30624/TCP        11m
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP               6h
market                 ClusterIP   10.102.152.34    <none>        5005/TCP              11m
orders                 ClusterIP   10.103.177.30    <none>        5002/TCP              11m
prometheus             ClusterIP   10.100.184.229   <none>        9090/TCP              11m
rabbitmq               ClusterIP   10.111.198.40    <none>        5673/TCP,15673/TCP    10m
redis                  ClusterIP   10.105.219.211   <none>        6380/TCP              10m
statsd                 ClusterIP   10.97.124.90     <none>        8125/UDP,8126/TCP     10m
statsd-exporter        ClusterIP   10.103.140.199   <none>        9102/TCP,9125/UDP     10m

$ kubectl get deployments --namespace=default
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
account-transactions   1         1         1            0           5m
alerts                 1         1         1            1           5m
elasticsearch          1         1         1            0           30s
fees                   1         1         1            0           5m
fluentd                1         1         1            0           5m
gateway                1         1         1            1           5m
grafana                1         1         1            0           5m
hello                  1         1         1            1           1d
kibana                 1         1         1            1           5m
market                 1         1         1            0           5m
orders                 1         1         1            0           5m
prometheus             1         1         1            0           5m
rabbitmq               1         1         1            0           5m
redis                  1         1         1            0           5m
statsd                 1         1         1            0           5m
statsd-exporter        1         1         1            0           5m

$ kubectl get pod --namespace=default
NAME                                   READY   STATUS    RESTARTS   AGE
account-transactions-895fcdd5f-chdlm   1/1     Running   1          1h
alerts-797cf6875f-njp47                1/1     Running   0          1h
elasticsearch-9965bfd94-bzv5m          1/1     Running   0          55m
fees-54f5845cdf-mj9bh                  1/1     Running   1          1h
fluentd-565cc9c796-gt2p2               1/1     Running   0          1h
gateway-5547b7bfc8-vlbn5               1/1     Running   1          1h
grafana-55c7956497-btb5s               1/1     Running   0          1h
hello-65ff564d-dvhq5                   1/1     Running   2          1d
kibana-55c56dff47-dfs8g                1/1     Running   0          1h
market-69b4c887c6-s7d8t                1/1     Running   1          1h
orders-b6644c97-jrxgf                  1/1     Running   1          1h
prometheus-6984678f96-cfvt6            1/1     Running   0          1h
rabbitmq-6ffd6bc47c-jfx25              1/1     Running   0          1h
redis-776cf5d95c-s8jv8                 1/1     Running   0          1h
statsd-64885496c4-6jrb2                1/1     Running   0          1h
statsd-exporter-6857d7d8c8-nbwld       1/1     Running   0          1h



$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl -X POST http://10.100.169.37:5001/shares/sell -H 'cache-control: no-cache' -H 'content-type: application/json'
{"ok": "sell order 2e6ff024-ccbf-45f3-adeb-9768f6aa9939 placed"}$ 

$ docker logs c901974b3a10
starting services: orders_service
Connected to amqp://guest:**@rabbitmq:5673//
```
```
$ cat gateway-service.yaml 
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.1.0 (36652f6)
  creationTimestamp: null
  labels:
    io.kompose.service: gateway
  name: gateway
spec:
  type: NodePort
  ports:
  - name: "5001"
    port: 5001
    targetPort: 5000
    nodePort: 30623
  selector:
    io.kompose.service: gateway
status:
  loadBalancer: {}

  480  kubectl delete svc/gateway
  481  kubectl create -f gateway-service.yaml 


$ curl -X POST http://`minikube ip`:30623/shares/sell -H 'cache-control: no-cache' -H 'content-type: application/json'
{"ok": "sell order bfbd8605-4362-40c9-8512-bfc1e44a5929 placed"}davar@home ~/LABS/microservices-in-action/chapter7-k8s $
```

```
Rancher : RKC deploy

$ sudo mkdir -p /vagrant/rancher_db
$ docker run -d -v /vagrant/rancher_db/mysql/:/var/lib/mysql --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.0.5

Login to rancher UI and add custom cluster

$ sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.5 --server https://192.168.0.102 --token dm4hrq7kn8xm5p9pkhl8xf4rxfr6mr89rxvtmz8fvcjllmz8j27cfc --ca-checksum 33bf59216e3e370cb3cbede1c8117c9694e32a0bf85b34267583e31d72a0300d --address 192.168.0.102 --etcd --controlplane --worker

download rancher (UI) and cp /usr/local/bin

Create new Key --- : Global/Users -> API&Keys .. create nen key "deploy" and get token

$ rancher login https://192.168.0.103 --token token-8lxb5:c7f5pjk462bs9v97rfcw5pfhtk9tgw94tt2hv8m45fn7qlwx58gwj9

$ ls
account-transactions-deployment.yaml  gateway-deployment.yaml  orders-deployment.yaml    redis-deployment.yaml
account-transactions-service.yaml     gateway-service.yaml     orders-service.yaml       redis-service.yaml
fees-deployment.yaml                  market-deployment.yaml   rabbitmq-deployment.yaml  statsd-agent-deployment.yaml
fees-service.yaml                     market-service.yaml      rabbitmq-service.yaml     statsd-agent-service.yaml


$ rancher kubectl create -f .

Add ingress .... service gateway
```
