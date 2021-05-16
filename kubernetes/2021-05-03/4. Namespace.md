# Namespace
- 논리적으로 서비스를 나눔


#### namespace 이 있는지 확인 (모두 마스터 노드에서 실습)


쿠버네티스가 기본으로 갖고 있는 namespace 들이다.

```bash
$ kubectl get namespaces

NAME              STATUS   AGE
default           Active   2d16h
kube-node-lease   Active   2d16h
kube-public       Active   2d16h
kube-system       Active   2d16h
```

## yaml 파일을 이용해서 pod 만들기

#### nginx.yaml 만들기

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.14
    name: nginx
    ports:
    - containerPort: 80
    - containerPort: 443
```

#### mypod 만들기


```bash
$ kubectl create -f nginx.yaml

Default namespace 에서 만들어지는게 포인트.

$ kubectl get pods --namespace default

NAME        READY   STATUS    RESTARTS   AGE
mypod       1/1     Running   0          87s
```

## CLI 를 이용해서 namespace 만들기

```bash
$ kubectl create namespace blue

namespace/blue created
```

#### yaml 파일을 이용해서 namespace 만들기

> *'orange'라는 namespace 만들건데 지금 만들진 말고, 만들 수 있는지 확인하고, 그 결과를 yaml 로 만들어줘.*

```bash
$ kubectl create namespace orange --dry-run=client -o yaml > orange-namespace.yaml
```

orange-namespace.yaml 필요한 내용만 남기고 수정.
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: orange
```

이제 namespace 만들어보자
```bash
$ kubectl create -f orange-namespace.yaml

namespace/orange created
```

#### pod 를 특정(blue) namespace 에 실행

```bash
$ kubectl create -f nginx.yaml -n blue

pod/mypod created
```

아니면 pod yaml 에 namespace 를 명시할 수도 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: orange
spec:
  containers:
  - image: nginx:1.14
    name: nginx
    ports:
    - containerPort: 80
    - containerPort: 443
```

```bash
$ kubectl create -f nginx.yaml

pod/mypod created
```

orange namespace 에 pod 가 실행되고 있는걸 확인할 수 있다.
```bash
$ kubectl get pods -n orange

NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          77s
```

## kubectl config

```bash
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.0.139:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

#### Set context

blue namespace 세팅
```bash
$ kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace=blue

Context "blue@kubernetes" created.
```

현재 context 확인
```bash
$ kubectl config current-context

kubernetes-admin@kubernetes
```

context 변경

이제 기본 namespace 는 default namespace 가 아니라 blue context에 있는 namespace (= blue) 가 된다.
```bash
$ kubectl config use-context blue@kubernetes

Switched to context "blue@kubernetes".
```

다시 현재 context 확인
```bash
$ kubectl config current-context

blue@kubernetes
```


default namespace에 있는 mypod 삭제

> 만약 -n default 를 안 붙였으면 현재 blue namespace 에 있는 mypod가 지워질거다.
```bash
$ kubectl delete pods mypod -n default

pod "mypod" deleted
```

다시 기본 context 로 바꿔두자.
```bash
$ kubectl config use-context kubernetes-admin@kubernetes

Switched to context "kubernetes-admin@kubernetes".
```

#### Namespace 삭제

namespace 를 삭제하면 안에 있는 API 들 모두가 삭제된다. **주의!**

```bash
$ kubectl delete namespaces blue

namespace "blue" deleted
```


##### [Go Overview.](https://github.com/es5es5/TIL/tree/main/kubernetes/2021-05-03)
