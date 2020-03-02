# Stateful

在实际的场景中，尤其是分布式应用，它的多个实例之间，往往有依赖关系，比如:主从关系、主备关系。还有就是数据存储类应用，它的多个实例，往往都会在本地磁盘上保存一份数据。而这些实例一旦被杀掉，即便重建出来，实例与数据之间的对应关系也已经丢失，从而导致应用失败。

容器技术诞生后，大家很快发现，它用来封装“无状态应用”(Stateless Application)，尤其是 Web 服务，非常好用。但是，一旦你想要用容器运行“有状态应用”，其困难程度就会直线上升。

Kubernetes 项目很早就在 Deployment 的基础上，扩展出了对“有状态应用”的初步支持。这个编排功能，就是：StatefulSet。

StatefulSet 的设计其实非常容易理解。它把真实世界里的应用状态，抽象为了两种情况: 

1. 拓扑状态。这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必 须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到 这个新 Pod。 

2. 存储状态。这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实 例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一 份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的 多个存储实例。 

所以，StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时， 能够为新 Pod 恢复这些状态。

## 拓扑状态

Service 是 Kubernetes 项目中用来将一组 Pod 暴露给外界访问的一种机制。比如，一个 Deployment 有 3 个 Pod，那么我就可以定义一个 Service。然后，用户只要能访问到这个 Service，它就能访问到某个具体的 Pod。

第一种方式，是以 Service 的 VIP(Virtual IP，即:虚拟 IP)方式。比如:当我访问 10.0.23.1 这个 Service 的 IP 地址时，10.0.23.1 其实就是一个 VIP，它会把请求转发到该 Service 所代理 的某一个 Pod 上。这里的具体原理，我会在后续的 Service 章节中进行详细介绍。 

第二种方式，就是以 Service 的 DNS 方式。比如:这时候，只要我访问“my-svc.my- namespace.svc.cluster.local”这条 DNS 记录，就可以访问到名叫 my-svc 的 Service 所代理 的某一个 Pod。 

而在第二种 Service DNS 的方式下，具体还可以分为两种处理方法: 

第一种处理方法，是 Normal Service。这种情况下，你访问“my-svc.my- namespace.svc.cluster.local”解析到的，正是 my-svc 这个 Service 的 VIP，后面的流程就跟 VIP 方式一致了。 

而第二种处理方法，正是 Headless Service。这种情况下，你访问“my-svc.my- namespace.svc.cluster.local”解析到的，直接就是 my-svc 代理的某一个 Pod 的 IP 地址。可 以看到，这里的区别在于，**Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录 的方式解析出被代理 Pod 的 IP 地址。** 

Headless Service 对应的 YAML 文件:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
app: nginx
```

可以看到，所谓的 Headless Service，其实仍是一个标准 Service 的 YAML 文件。只不过，它的 clusterIP 字段的值是:None，即:这个 Service，没有一个 VIP 作为“头”。这也就是Headless 的含义。所以，这个 Service 被创建后并不会被分配一个 VIP，而是会以 DNS 记录的方式暴露出它所代理的 Pod。

当你按照这样的方式创建了一个 Headless Service 之后，它所代理的所有 Pod 的 IP 地址，都会被绑定一个这样格式的 DNS 记录，如下所示:

```
<pod-name>.<svc-name>.<namespace>.svc.cluster.local
```

有了这个“可解析身份”，只要你知道了一个 Pod 的名字，以及它对应的 Service 的名字，你就可以非常确定地通过这条 DNS 记录访问到 Pod 的 IP 地址。

StatefulSet 又是如何使用这个 DNS 记录来维持 Pod 的拓扑状态的呢?

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
					name: web
```

这个 YAML 文件，和我们在前面文章中用到的 nginx-deployment 的唯一区别，就是多了一个 serviceName=nginx 字段。 

这个字段的作用，就是告诉 StatefulSet 控制器，在执行控制循环(Control Loop)的时候， 请使用 nginx 这个 Headless Service 来保证 Pod 的“可解析身份”。 

所以，当你通过 kubectl create 创建了上面这个 Service 和 StatefulSet 之后，就会看到如下两个对象:

```
$ kubectl create -f svc.yaml
$ kubectl get service nginx
NAME      TYPE         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP    None         <none>        80/TCP    10s
$ kubectl create -f statefulset.yaml
$ kubectl get statefulset web
NAME      DESIRED   CURRENT   AGE
web       2         1         19s
```

这时候，如果你手比较快的话，还可以通过 kubectl 的 -w 参数，即:Watch 功能，实时查看 StatefulSet 创建两个有状态实例的过程:

```
$ kubectl get pods -w -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     0/1       Pending   0          0s
web-0     0/1
web-0     0/1
web-0     1/1
web-1     0/1
web-1     0/1
web-1     0/1
web-1     1/1
```

通过上面这个 Pod 的创建过程，我们不难看到，StatefulSet 给它所管理的所有 Pod 的名字， 进行了编号，编号规则是:-。 

而且这些编号都是从 0 开始累加，与 StatefulSet 的每个 Pod 实例一一对应，绝不重复。 

更重要的是，这些 Pod 的创建，也是严格按照编号顺序进行的。比如，在 web-0 进入到 Running 状态、并且细分状态(Conditions)成为 Ready 之前，web-1 会一直处于 Pending 状态。 

当这两个 Pod 都进入了 Running 状态之后，你就可以查看到它们各自唯一的“网络身 份”了。 

我们使用 kubectl exec 命令进入到容器中查看它们的 hostname: 

```
$ kubectl exec web-0 -- sh -c 'hostname'
web-0
$ kubectl exec web-1 -- sh -c 'hostname'
web-1
```

当我们把这两个 Pod 删除之后，Kubernetes 会按照原先编号的顺序，创建出了两个新的 Pod。并且，Kubernetes 依然为它们分配了与原来相同的“网络身份”:web-0.nginx和 web-1.nginx。

如果 web-0 是一个需要先启动的主节点，web-1 是一个后启动的从节点，那么只要这个StatefulSet 不被删除，你访问 web-0.nginx 时始终都会落在主节点上，访问 web-1.nginx 时，则始终都会落在从节点上，这个关系绝对不会发生任何变化。

通过这种方法，Kubernetes 就成功地将 Pod 的拓扑状态(比如:哪个节点先启动，哪个节点后启动)，按照 Pod 的“名字 + 编号”的方式固定了下来。此外，Kubernetes 还为每一个 Pod 提供了一个固定并且唯一的访问入口，即:这个 Pod 对应的 DNS 记录。

## 存储状态

作为一个应用开发者，我可能对持久化存储项目(比如 Ceph、GlusterFS 等)一窍不通，也不知道公司的 Kubernetes 集群里到底是怎么搭建出来的，我也自然不会编写它们对应的 Volume 定义文件。

直接创建存储 Pod 的问题是什么？

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rbd
spec:
  containers:
    - image: kubernetes/pause
      name: rbd-rw
      volumeMounts:
      - name: rbdpd
        mountPath: /mnt/rbd
  volumes:
    - name: rbdpd
      rbd:
        monitors:
        - '10.16.154.78:6789'
        - '10.16.154.82:6789'
        - '10.16.154.83:6789'
        pool: kube
        image: foo
        fsType: ext4
        readOnly: true
        user: admin
        keyring: /etc/ceph/keyring
        imageformat: "2"
        imagefeatures: "layering"
```

其一，如果不懂得 Ceph RBD 的使用方法，那么这个 Pod 里 Volumes 字段，你十有八九也完全看不懂。其二，这个 Ceph RBD 对应的存储服务器的地址、用户名、授权文件的位置，也都被轻易地暴露给了全公司的所有开发人员，这是一个典型的信息被“过度暴露”的例子。

在后来的演化中，Kubernetes 项目引入了一组叫作 Persistent VolumeClaim(PVC)和 Persistent Volume(PV)的 API 对象，大大降低了用户声明和使用持久化 Volume 的门槛。

- 定义一个 PVC，声明想要的 Volume 的属性:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

可以看到，在这个 PVC 对象里，不需要任何关于 Volume 细节的字段，只有描述性的属性和定义。比如，storage: 1Gi，表示我想要的 Volume 大小至少是 1 GB;accessModes: ReadWriteOnce，表示这个 Volume 的挂载方式是可读写，并且只能被挂载在一个节点上而非被多个节点共享。

[AccessModes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

- 在应用的 Pod 中，声明使用这个 PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
```

- 绑定 PV

这时候，只要我们创建这个 PVC 对象，Kubernetes 就会自动为它绑定一个符合条件的 Volume。可是，这些符合条件的 Volume 又是从哪里来的呢? 

答案是，它们来自于由运维人员维护的 PV(Persistent Volume)对象。接下来，我们一起看 一个常见的 PV 对象的 YAML 文件: 

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  rbd:
    monitors:
    - '10.16.154.78:6789'
    - '10.16.154.82:6789'
    - '10.16.154.83:6789'
    pool: kube
    image: foo
    fsType: ext4
    readOnly: true
    user: admin
    keyring: /etc/ceph/keyring
    imageformat: "2"
    imagefeatures: "layering"
```

这个 PV 对象的 spec.rbd 字段，正是我们前面介绍过的 Ceph RBD Volume 的详细定义。

以 StatefulSet yaml

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
				app: nginx
  spec:
    containers:
    - name: nginx
      image: nginx:1.9.1
      ports:
      - containerPort: 80
        name: web
      volumeMounts:
      - name: www
        mountPath: /usr/share/nginx/html
volumeClaimTemplates:
- metadata:
    name: www
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi		
```

运行

```shell
$ kubectl create -f statefulset.yaml
$ kubectl get pvc -l app=nginx
```

可以看到，这些 PVC，都以“<PVC 名字 >-<StatefulSet 名字 >-< 编号 >”的方式命名，并且处于 Bound 状态。

StatefulSet 控制器恢复这个 Pod 的过程

首先，当你把一个 Pod，比如 web-0，删除之后，这个 Pod 对应的 PVC 和 PV，并不会被删 除，而这个 Volume 里已经写入的数据，也依然会保存在远程存储服务里(比如，我们在这个 例子里用到的 Ceph 服务器)。 

此时，StatefulSet 控制器发现，一个名叫 web-0 的 Pod 消失了。所以，控制器就会重新创建 一个新的、名字还是叫作 web-0 的 Pod 来，“纠正”这个不一致的情况。 

需要注意的是，在这个新的 Pod 对象的定义里，它声明使用的 PVC 的名字，还是叫作:www- web-0。这个 PVC 的定义，还是来自于 PVC 模板(volumeClaimTemplates)，这是 StatefulSet 创建 Pod 的标准流程。 

所以，在这个新的 web-0 Pod 被创建出来之后，Kubernetes 为它查找名叫 www-web-0 的 PVC 时，就会直接找到旧 Pod 遗留下来的同名的 PVC，进而找到跟这个 PVC 绑定在一起的 PV。 

这样，新的 Pod 就可以挂载到旧 Pod 对应的那个 Volume，并且获取到保存在 Volume 里的数据。 

通过这种方式，Kubernetes 的 StatefulSet 就实现了对应用存储状态的管理。 