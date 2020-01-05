# W14 20191210

- [W14 20191210](#w14-20191210)
  - [Kubernetes](#kubernetes)
  - [MiniKube](#minikube)
    - [minikube start](#minikube-start)
    - [About Pod](#about-pod)
    - [Debug](#debug)
    - [Expose Port &amp; Test](#expose-port-amp-test)
    - [Scale](#scale)
    - [Rolling Update Image](#rolling-update-image)
    - [Rollout Image](#rollout-image)
  - [安裝 Kubernetes 在 Ubuntu 上](#%e5%ae%89%e8%a3%9d-kubernetes-%e5%9c%a8-ubuntu-%e4%b8%8a)
    - [環境需求](#%e7%92%b0%e5%a2%83%e9%9c%80%e6%b1%82)

## Kubernetes

- 參見：[Learn Kubernetes Basics - Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- 參見：[[Kubernetes] Cluster Architecture | 小信豬的原始部落](https://godleon.github.io/blog/Kubernetes/k8s-CoreConcept-Cluster-Architecture/)
- 老師推薦影片：[尚硅谷Kubernetes（k8s基於最新2019年8月發佈的1.15.1）_嗶哩嗶哩 (゜-゜)つロ 乾杯~-bilibili](https://www.bilibili.com/video/av66617940?from=search&seid=7144140822222817688)
- 類似於 Docker Swarm，但功能叫強大，也因此較為複雜

## MiniKube

- [Interactive Tutorial - Creating a Cluster - Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)

### minikube start

```
$ minikube start

$ hostname
minikube

$ kubectl cluster-info // 查看 cluster 資訊
Kubernetes master is running at https://172.17.0.47:8443
KubeDNS is running at https://172.17.0.47:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

- 部署一個名為 `kubernetes-bootcamp` 的應用，使用 `docker.io/jocatalin/kubernetes-bootcamp:v1` 鏡像檔

```
$ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080 // --port 設置對外服務的 Port

$ kubectl get pods // 查看當前所有的 Pod
$ kubectl get pods -o wide
/* -o: output 通常用來顯示執行在哪個 NODE 上
   List all pods in ps output format with more information (such as node name). */
```

### About Pod

- Kubernetes 最小單位並非為 Docker，而是 Pod
- 一個 Pod 至少有一個 Docker Container
- 緊密耦合的容器可以放在同一個 Pod 中，反之則建議拆散

### Debug

```
$ kubectl get pods // 顯示所有的 pod
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-7dc9765bf6-g8qq9   1/1     Running   0          30m

$ kubectl describe pod <podName> // 查看 pod 相關資訊來偵錯

// 在老師的案例中，image 位置輸入錯誤，因此需要移除來重新安裝
$ kubectl delete deployment kubernetes-bootcmp // 移除相對應的 Pod
$ kubectl gets pod // 確定移除後再重新安裝
```

```
$ kubectl get deployment // 顯示所有 deployment
$ kubectl describe deployment <deploymentName> // 查看指定 deployment 的相關資訊
```

### Expose Port & Test

- 註：`services` 可縮寫成 `svc`

```
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8000

$ kubectl get services
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S) AGE
kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP 5m18s
kubernetes-bootcamp   NodePort    10.103.52.57   <none>        8080:31109/TCP 10s

$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-62cn8 | v=1
```

### Scale

```
$ kubectl scale deployment/kubernetes-bootcamp --replicas=3
deployment.extensions/kubernetes-bootcamp scaled

$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/3     3            3           19m

$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-7dc9765bf6-62cn8   1/1     Running   0          21m
kubernetes-bootcamp-7dc9765bf6-bp25r   1/1     Running   0          88s
kubernetes-bootcamp-7dc9765bf6-n5mdq   1/1     Running   0          88s

/* 會執行在不同的 Pods 上 */
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-62cn8 | v=1
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-n5mdq | v=1
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-bp25r | v=1
```

### Rolling Update Image

```
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment.extensions/kubernetes-bootcamp image updated

$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-7dc9765bf6-62cn8   1/1     Terminating   0          35m
kubernetes-bootcamp-7dc9765bf6-n5mdq   1/1     Terminating   0          15m
kubernetes-bootcamp-cfc74666-fx5jd     1/1     Running       0          10s
kubernetes-bootcamp-cfc74666-jzfsk     1/1     Running       0          13s

/* 最後面的版本升級為 v.2 */
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-cfc74666-fx5jd | v=2
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-cfc74666-jzfsk | v=2
```

### Rollout Image

```
$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp rolled back
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-nzpr4 | v=1
$ curl minikube:31109
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7dc9765bf6-w47jn | v=1
```

## 安裝 Kubernetes 在 Ubuntu 上

### 環境需求

1. Master * 1: 2 CPU Cores, at least 1GB RAM
2. Node * 2: at least one machine, two is better.

```
user@ubuntu:~$ sudo su

root@ubuntu:/home/user# gedit /etc/fstab // 在 UUID 那行前面加上 # 以註解

root@ubuntu:/home/user# gedit /etc/resolv.conf // 將 nameserver 更改為 8.8.8.8 (Google Public DNS)

root@ubuntu:/home/user# apt-get update && apt-get install docker.io

root@ubuntu:/home/user# apt-get update && apt-get install -y apt-transport-https

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

apt-get install -y kubelet kubeadm kubectl
```