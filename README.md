# Konvoy

kubernetes version: 1.15.5 calico version: 3.8.2 containerd: 1.2.6 Dokcer version: 19.03.5

배포 서버 필요사항

Docker 18.09.2 이상 (Nvidia 때문에 19버전 추천)

kubectl 1.15.5 이상

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version
```

노드 마다 최소 필요사항

8core , 16G 메모리

- CentOS 7.6
- 방화벽이 비활성화
- 컨테이너가 제거
- Docker-ce가 제거
- 스왑이 비활성화

스왑 비활성화

```bash
systemctl disable firewalld
systemctl stop firewalld
swapoff -a
sudo setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config 
```

```
mkdir konvoy-deploy
cd konvoy-deploy
```

배포서버에서 콘보이 고고

```bash
wget https://downloads.mesosphere.io/konvoy/konvoy_v1.4.2_linux.tar.bz2
bzip2 -d konvoy_v1.4.2_linux.tar.bz2
tar -xvf konvoy_v1.4.2_linux.tar
```

```bash
./konvoy init --provisioner=none --cluster-name kbsys-k8s
```

*git 참고해서 작성할것.

*cluster.yml 의 `controlPlaneEndpointOverride` 항목, `addresses` 항목은 실제 설치 대상 서버와 같은 ip 대역을 사용하되, 중복되어 사용하지 않는 다른 ip를 작성해 주어야 함. `networking` 항목의 대역들은 겹치지 않는 대역으로 작성할 것.

배포노드에서 ssh 설정 준비

```bash
cd ~/.ssh
ssh-keygen
```

```bash
ssh-copy-id root@hostname
```

**pv생성**

controlplane 노드를 포함한 모든 cluster의 노드들에 pv로 사용할 디렉토리를 생성해준다.

```bash
#예시

mkdir -p /pv/konvoy/pv-10-1
mkdir -p /pv/konvoy/pv-10-2
mkdir -p /pv/konvoy/pv-10-3
mkdir -p /pv/konvoy/pv-10-4
mkdir -p /pv/konvoy/pv-10-5
mkdir -p /pv/konvoy/pv-50-1
mkdir -p /pv/konvoy/pv-50-2
mkdir -p /pv/konvoy/pv-50-3
mkdir -p /pv/konvoy/pv-50-4
```

`pv.yaml` 파일의 내용을 생성할 pv의 용량과 경로에 맞게 수정하여 controlplane노드에 위치시킨다. 아래 예시 파일은 10GB pv 3개, 50GB pv 1개를 생성하는 예시이다.

```yaml
#파일예시

apiVersion: v1
kind: PersistentVolume
metadata:
  name: konvoy-pv-volume-10-1
  labels:
    type: local
spec:
  storageClassName: localvolumeprovisioner
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/pv/konvoy/pv-10-1"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: konvoy-pv-volume-10-2
  labels:
    type: local
spec:
  storageClassName: localvolumeprovisioner
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/pv/konvoy/pv-10-2"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: konvoy-pv-volume-10-3
  labels:
    type: local
spec:
  storageClassName: localvolumeprovisioner
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/pv/konvoy/pv-10-3"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: konvoy-pv-volume-50-1
  labels:
    type: local
spec:
  storageClassName: localvolumeprovisioner
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/pv/konvoy/pv-50-1"
---
```

위의 내용들까지 준비가 되었다면, konvoy 설치 준비가 다 되었다.

```yaml
./konvoy up
```

콘보이 설치 중, add on deploy 단계로 진행이 넘어가면, controlplane노드에 접속하여 미리 준비한 pv.yaml 파일을 이용하여 pv생성을 진행한다.

```yaml
KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f pv.yaml
```

기다린다.

끝 ㅇㅈ?
