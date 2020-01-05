# W15 20191217

- [W15 20191217](#w15-20191217)
  - [安裝 Kubernetes 在 Ubuntu 上（續）](#%e5%ae%89%e8%a3%9d-kubernetes-%e5%9c%a8-ubuntu-%e4%b8%8a%e7%ba%8c)

## 安裝 Kubernetes 在 Ubuntu 上（續）

- Master 與 Slave 皆先關閉 swap

```
sudo swapoff -a
```

- 在 Master 上初始化 Kubeadm

```
kubeadm init --apiserver-advertise-address 172.16.45.131 --pod-network-cidr=10.244.0.0/16
```



```
kubectl get pods // 先取得 Pod Name
kubectl exec <httpd_pod_name> -it -- /bin/sh

// 上述的例子中 Pod 內只有一個 Container，但如果今天 Pod 內有多個 container，需要在後方加上 -c <container_name>
kubectl exec <httpd_pod_name> -it -c <container_name> -- /bin/sh
```

- 書中的內容較舊，因此部份檔案可能需要修改
- 但老師有放修正後的檔案在 `192.168.60.15/docker/k8s` 資料夾中