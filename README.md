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
