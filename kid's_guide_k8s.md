# The Illustrated Children's Guide to Kubernetes
[you tube link](https://www.youtube.com/watch?v=3I9PkvZ80BQ&t=9s)

## 태초의 웹 앱 (Web app)
- single app itself
- on Web server or Various OS

## 독립된 공간 필요 (Container)
- app on Container
- Provide an isolated context
- Need to be managed 
    - especially, Networking is hard
    - must be scheduled, distributed, load balanced
    - data는 어떤 다른 공간에 저장되어야 함

## 그래서 관리 해줄게! (Kubernetes, k8s)
### 관리를 위해 컨테이너를 담아놓는 공간 pods
- pod은 overlay network를 통해 다른 environment와 연결(통신)
- *what's overlay network?*
### 각 pod의 이름표 labels (name tag)
- query based on these labels
### 좀더 안정적으로 replicaSets
- have a pod template -> creating pod copies
- scaling the pod up of down
- can be used for rolling deploys
- *what's rolling deploys?*
### 으어어 다른 pod들이 보고싶어어어 Services
- Persistent
- Provide discovery
- Provide load balancing
- Provide stable service address
- Find pods by label selector
### 저장공간이 필요하다 Volumes
- Provider expose both 'persistent' and 'ephemeral' storage (EBS, Ceph, Gluster)
- Pods can mount volumes like filesystems
### Namespaces
- Group + segment pods, ReplicaSets, volumes, & secrets from each other

