# Kubernetes StatefulSet

[TOC]


---

## 認識 StatefulSet

![](https://i.imgur.com/2lY7OHu.png)

- StatefulSet 中所有 Pod 的名字都會以遞增整數作為結尾(以0為起點), 因此 Pod DNS 變得容易訪問, ReplicaSet 則隨機生成 Pod 名字 。

![](https://i.imgur.com/UeO9NoL.png)

- 當 Pod 所在的 worker node 損毀, StatefulSet 會在其他 worker node 上創建該 Pod, 且保持 Pod 的名字和 hostname, ReplicaSet 則會重新分配 Pod 的名字和 hostname 。

![](https://i.imgur.com/UMfVubC.png)

- 當減小 StatefulSet 的 replica count 時, 會按照 Pod 的名字倒序移除 Pod, 一次只能移除一個 Pod, 序號最大的先被移除, ReplicaSet 則會隨機移除一個 Pod

![](https://i.imgur.com/TMw8Jdq.png)

- 由於 Pod 移轉時必須保證新建的 Pod 依然綁定原 Volume, StatefulSet 需要維持 Pod 與 PersistentVolumeClaim 之間的對應關係, 所以 PVC 不再由用戶管理



```bash=
echo 'apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: quay.io/flysangel/nginx
          volumeMounts:
            - name: html-storage
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: html-storage
      spec:
        storageClassName: local-path
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi' > statefulset-nginx.yaml
```

```bash=
$ kubectl exec -it nginx-0 -- bash -c "echo 'My nginx-0' > /usr/share/nginx/html/index.html"

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
nginx-0                  1/1     Running   0          4m53s   10.244.2.55   w2       <none>           <none>
nginx-1                  1/1     Running   0          4m27s   10.244.0.26   m1       <none>           <none>
nginx-2                  1/1     Running   0          58s     10.244.2.57   w2       <none>           <none>
nginx-6f47499ff9-2dfcp   0/1     Pending   0          37m     <none>        <none>   <none>           <none>
nginx-6f47499ff9-5q6q8   0/1     Pending   0          37m     <none>        <none>   <none>           <none>
nginx-6f47499ff9-f5lh5   0/1     Pending   0          37m     <none>        <none>   <none>           <none>

$ curl 10.244.2.55
My nginx-0

$ curl 10.244.2.57
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>

$ curl 10.244.0.26
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>
```



```bash=
$ kubectl exec -it nginx-1 -- bash -c "echo 'My nginx-1' > /usr/share/nginx/html/index.html"

$ curl 10.244.2.57
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>

$ curl 10.244.0.26
My nginx-1

$ curl 10.244.2.55
My nginx-0
```


```bash=
$ kubectl exec -it nginx-2 -- bash -c "echo 'My nginx-2' > /usr/share/nginx/html/index.html"

bigred@m1:~/0613$ curl 10.244.2.57
My nginx-2

bigred@m1:~/0613$ curl 10.244.0.26
My nginx-1

bigred@m1:~/0613$ curl 10.244.2.55
My nginx-0
```


## 認識 StatefulSet Service

![](https://i.imgur.com/5kHPENo.png)

- 對於指向 StatefulSet 的 Service, 其他服務訪問可透過該 Service 取得每個 Pod 的 DNS 名稱, 以 Pod A-0 為例, 透過 a-0.A.default.svc.cluster.local 即可訪問服務

```bash=
$ echo 'apiVersion: v1
> kind: Service
> metadata:
>   name: nginx
> spec:
>   clusterIP: None
>   selector:
>     app: nginx' > svc-nginx.yaml

$ kubectl apply -f svc-nginx.yaml
service/nginx unchanged

$ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
app1         ClusterIP   10.98.0.2     <none>         8080/TCP   4d18h
app2         ClusterIP   10.98.0.200   <none>         8080/TCP   4d18h
kubernetes   ClusterIP   10.98.0.1     <none>         443/TCP    13d
nginx        ClusterIP   None          <none>         <none>     58m
svc-fbs      ClusterIP   10.98.0.136   192.168.61.4   8080/TCP   11d

$ kg pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
nginx-0                  1/1     Running   0          20m   10.244.2.55   w2       <none>           <none>
nginx-1                  1/1     Running   0          19m   10.244.0.26   m1       <none>           <none>
nginx-2                  1/1     Running   0          16m   10.244.2.57   w2       <none>           <none>
nginx-6f47499ff9-2dfcp   0/1     Pending   0          52m   <none>        <none>   <none>           <none>
nginx-6f47499ff9-5q6q8   0/1     Pending   0          52m   <none>        <none>   <none>           <none>
nginx-6f47499ff9-f5lh5   0/1     Pending   0          52m   <none>        <none>   <none>           <none>

$ nslookup
> server 10.98.0.10
Default server: 10.98.0.10
Address: 10.98.0.10#53
> 10.244.2.55
55.2.244.10.in-addr.arpa        name = nginx-0.nginx.default.svc.k8s.org.

> 10.244.0.26
26.0.244.10.in-addr.arpa        name = nginx-1.nginx.default.svc.k8s.org.

> 10.244.2.57
57.2.244.10.in-addr.arpa        name = nginx-2.nginx.default.svc.k8s.org.

> nginx.default.svc.k8s.org
Server:         10.98.0.10
Address:        10.98.0.10#53

Name:   nginx.default.svc.k8s.org
Address: 10.244.2.55
Name:   nginx.default.svc.k8s.org
Address: 10.244.0.26
Name:   nginx.default.svc.k8s.org
Address: 10.244.2.57
```

## 刪除 StatefulSet

```bash=
$ kubectl delete -f svc-nginx.yaml
service "nginx" deleted

$ kubectl delete -f statefulset-nginx.yaml
statefulset.apps "nginx" deleted

$ kubectl delete pvc html-storage-nginx-0 html-storage-nginx-1 html-storage-nginx-2
persistentvolumeclaim "html-storage-nginx-0" deleted
persistentvolumeclaim "html-storage-nginx-1" deleted
persistentvolumeclaim "html-storage-nginx-2" deleted

```

**由 StatefulSet 生成的 pvc ，他的生命週期會跟著 Namespace 活著，直到 Namespace 被刪除**



## 練習

請問如何擴展 StatefulSet。
請觀察擴展 StatefulSet 時，pod 或 container 狀態為何。
請建立 StatefulSet replicas 為 3，並逐一新增 nginx 網頁，當 replicas 由 3 轉為 2 再恢復至 3 時，請問資料是否都還保存完整。

練習完後請刪除練習環境
Pod -> PVC


```bash=
$ kubectl apply -f svc-nginx.yaml

$ kubectl apply -f statefulset-nginx.yaml

$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
nginx-0   1/1     Running   0          86s   10.244.2.61   w2     <none>           <none>
nginx-1   1/1     Running   0          76s   10.244.0.29   m1     <none>           <none>
nginx-2   1/1     Running   0          66s   10.244.2.63   w2     <none>           <none>

$ $ kubectl scale  statefulset/nginx --replicas=6
statefulset.apps/nginx scale

$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP             NODE   NOMINATED NODE   READINESS GATES
nginx-0   1/1     Running   0          2m55s   10.244.2.61    w2     <none>           <none>
nginx-1   1/1     Running   0          2m45s   10.244.0.29    m1     <none>           <none>
nginx-2   1/1     Running   0          2m35s   10.244.2.63    w2     <none>           <none>
nginx-3   1/1     Running   0          45s     10.244.1.126   w1     <none>           <none>
nginx-4   1/1     Running   0          37s     10.244.1.128   w1     <none>           <none>
nginx-5   1/1     Running   0          28s     10.244.0.31    m1     <none>           <none>

$ kubectl scale  statefulset/nginx --replicas=3
statefulset.apps/nginx scaled

bigred@m1:~/0613$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx-0   1/1     Running   0          3m38s   10.244.2.61   w2     <none>           <none>
nginx-1   1/1     Running   0          3m28s   10.244.0.29   m1     <none>           <none>
nginx-2   1/1     Running   0          3m18s   10.244.2.63   w2     <none>           <none>

$ kubectl exec -it nginx-2 -- bash -c "echo 'My nginx-2' > /usr/share/nginx/html/index.html"

bigred@m1:~/0613$ curl 10.244.2.63
My nginx-2

$ kubectl scale  statefulset/nginx --replicas=2
statefulset.apps/nginx scaled

bigred@m1:~/0613$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx-0   1/1     Running   0          5m13s   10.244.2.61   w2     <none>           <none>
nginx-1   1/1     Running   0          5m3s    10.244.0.29   m1     <none>           <none>

$ kubectl scale  statefulset/nginx --replicas=3
statefulset.apps/nginx scaled

bigred@m1:~/0613$ kg pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx-0   1/1     Running   0          5m54s   10.244.2.61   w2     <none>           <none>
nginx-1   1/1     Running   0          5m44s   10.244.0.29   m1     <none>           <none>
nginx-2   1/1     Running   0          12s     10.244.2.64   w2     <none>           <none>

$ curl 10.244.2.64
My nginx-2
```


答：**資料有保存**



###### tags: `系統工程`
