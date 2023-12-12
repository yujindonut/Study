### Config 맵이 필요한 이유

<img width="835" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8ebd6378-7361-44a5-84de-e8aafd736d44">

- Dev
  보안접근이 false가 되어도 가능
  User도 dev
- Prod환경
  보안 접근이 true가 되어야함.

환경마다 다른 Setting 값을 container 자원에 저장해 놓는 것은 비효율적이다.
따라서 이를 ConfigMap, Secret 에 저장해두고 관리한다.

<img width="791" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/a5413513-f10d-45f6-bf0d-e48ef9cc3e23">

Env값을 변경하면서 따로 효율적으로 관리가 가능하다.

- Env (Literal)
  <img width="282" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/ba97c112-fda1-4530-9dc9-b6b1d3b584a0">

- Env (File)
  <img width="275" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8bef1d02-e370-4bbc-b95d-76c2b9476c70">
  Pod가 재생성되어야지만 file의 변경 내용이 바뀐것이 적용이 된다.
  
- Volume Mount (File)
  <img width="272" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/46b319dc-42a6-425b-80b0-72d1e4ab0d47">
  파일이 Mount 되는 것이므로 file의 변경사항들이 Pod에도 적용이 그대로 된다. 

### 실습

- Env (Literal)
  환경변수로 저장한 값은 변경될 시에 적용이 바로 되지 않는다. 
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: 'false'
  User: dev

apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA== // base64 인코딩한 값만 들어감

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```
- Env (File)

```
// config Map
echo "Content" >> file-c.txt
kubectl create configmap cm-file --from-file=./file-c.txt

// Secret
echo "Content" >> file-s.txt
kubectl create secret generic sec-file --from-file=./file-s.txt

// Pod
apiVersion: v1
kind: Pod
metadata:
  name: pod-file
spec:
  containers:
  - name: container
    image: kubetm/init
    env:
    - name: file-c
      valueFrom:
        configMapKeyRef:
          name: cm-file
          key: file-c.txt
    - name: file-s
      valueFrom:
        secretKeyRef:
          name: sec-file
          key: file-s.txt
```

- Volume Mount (File)
```
// Pod
apiVersion: v1
kind: Pod
metadata:
  name: pod-mount
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount
  volumes:
  - name: file-volume
    configMap:
      name: cm-file

```
