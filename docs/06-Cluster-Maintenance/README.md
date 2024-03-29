# Cluster Maintenance

[메인으로 돌아가기](../../README.md)

## OS Upgrades

### Pod Eviction Timeout

- 파드가 다운되었을 때, Kubelet은 파드를 다시 활성 상태로 만든다.
- 하지만 파드가 5분 이상 다운 상태가 되면 쿠버네티스는 그 파드가 죽은 것으로 간주하여 해당 파드는 노드로부터 종료된다.
- 만약 파드가 레플리카셋의 일부인 경우에는 다른 노드에서 재생성된다.
- 파드가 다시 온라인 상태가 될 때까지 기다리는 시간을 파드 제거 시간 초과라고 하며, 아래 명령어를 통해 설정할 수 있다.
  ```bash
  kube-controller-manager --pod-eviction-timeout=5m0s
  ```
- 기본 설정은 5분으로, 마스터 노드는 파드가 죽은 것으로 간주하기 전에 최대 5분동안 기다리는 것이다.
- 파드 제거 시간 초과 이후 노드가 다시 활성화되면 예약된 파드 없이 빈 노드 상태로 활성화된다.

### Drain

- 안전하게 노드를 작업하기 위해, 노드를 비워(drain) 모든 워크로드를 다른 노드로 옮길 수 있다. 이때 데몬셋을 제외하고 적용하려면 `--ignore-daemonsets` 옵션을 사용한다.
  ```bash
  kubectl drain node-1
  ```
- 기존 노드에서 파드가 종료되고, 다른 노드에서 재생성된다.
- 노드가 통제(cordon)되거나 스케줄 불가능 상태가 된다. 이는 특별한 제한이 제거될 때까지 해당 노드에 스케줄된 파드가 없음을 의미한다.
- 파드가 다른 노드에서 정상적으로 구동되면, 노드를 재시동할 수 있다. 해당 노드가 다시 활성화되어도 여전히 스케줄 불가능 상태이다.
- 노드의 통제를 해제해야만 다시 파드가 스케줄된다.
  ```bash
  kubectl uncordon node-1
  ```
- 노드를 스케줄 불가능하게 하지만 노드를 비우는 것(drain)과 다르게 파드를 종료/이동시키지 않고 싶다면 통제(cordon)를 적용한다. 이는 새로운 파드가 더 이상 스케줄링되지 않도록 처리한다.
  ```bash
  kubectl cordon node-1
  ```

## Cluster Upgrade Process

- 쿠버네티스의 구성 요소들의 버전이 모두 같을 필요는 없다.
- 하지만 kube-apiserver는 컨트롤 플레인의 핵심 구성 요소이기 때문에 다른 구성 요소들이 kube-apiserver보다 버전이 높으면 안된다.
  - kube-controller-manager, kube-scheduler는 한 버전 더 낮을 수 있다.
  - kubelet, kube-proxy는 두 버전까지 낮을 수 있다.
  - kubectl은 1 버전 높거나, 같거나, 1 버전 낮을 수 있다.
- 쿠버네티스는 오직 최근 세 개의 버전까지만 지원한다.
- 추천되는 버전 업그레이드에 대한 접근은 마이너 버전을 하나씩 올리는 것이다.
- 버전 업그레이드는 클러스터를 어떤 방식으로 구성했는지에 따라 다르다.
  - 클라우드 제공자: 제공되는 업그레이드 기능을 사용한다.
  - kubeadm: kubeadm을 통해 업그레이드한다.
  - scratch: 수동으로 업그레이드한다.

### 어떻게 업그레이드할까?

**마스터 노드 업그레이드**

- 마스터 노드가 업그레이드할 동안 kube-apiserver, kube-controller-manager, kube-scheduler 등의 컨트롤 플레인 구성 요소들이 다운된다.
- 마스터 노드가 다운된다고 해서 워커 노드가 영향을 받는 것은 아니다.
- kube-apiserver가 다운되어 있기 때문에, 이와 관련된 리소스에는 접근할 수 없으며, 새로운 애플리케이션을 배포하거나 삭제하거나 수정할 수 없다.
- 만약 파드가 죽으면, 새로운 파드가 자동으로 재생성되지 않는다.
- 하지만 노드와 파드가 구동 중일 때는 영향을 받지 않는다.

**워커 노드 업그레이드**

1. 워커 노드를 한 번에 업그레이드한다: 업그레이드 동안 애플리케이션에 접근이 불가능하다.
2. 워커 노드 하나씩 업그레이드한다: 워커 노드마다 하나씩 업그레이드할 노드의 파드를 다른 실행 중인 노드에 옮기고 업그레이드를 한다. 완료 후 다시 옮기는 과정을 반복한다.
3. 버전 업그레이드한 새로운 노드를 클러스터에 추가하고, 기존 노드를 축출한다: 클라우드 제공자 환경에서 제공되는 기능이다.

### Kubeadm을 통한 버전 업그레이드

- 아래 명령어를 통해 업그레이드할 버전에 대한 정보를 알 수 있다.
  ```bash
  kubeadm upgrade plan
  ```
- 아래 명령어를 통해 업그레이드를 적용할 수 있다.
  ```bash
  kubeadm upgrade apply <version>
  ```
- kubeadm은 kubelet을 업그레이드하지 않는다. 수동으로 업그레이드해야 한다.
- kubeadm은 쿠버네티스 버전에 맞게 업그레이드되기 때문에 v1.11에서 바로 v1.13으로 업그레이드된다. 하지만 한 번에 한 버전씩 업그레이드하는 방법이 추천되기 때문에 아래와 같은 버전으로 버전을 한
  단계씩 올리도록 하자.
  ```bash
  apt-get upgrade -y kubeadm=1.12.0-00
  kubeadm upgrade apply v1.12.0
  ```

**마스터 노드 업그레이드**

- `kubectl get nodes` 명령어를 통해 각 노드의 버전을 확인해도 버전이 변하지 않은 것을 확인할 수 있다.
- 이는 kube-apiserver의 버전이 아닌, kube-apiserver에 등록된 kubelet의 버전이기 때문이다.
- 마스터 노드의 kubelet도 수동으로 업그레이드해야 한다.
- 아래 명령어를 통해 kubelet을 업그레이드할 수 있다.
  ```bash
  apt-get upgrade -y kubelet-1.12.0-00
  systemctl restart kubelet
  ```

**워커 노드 업그레이드**

- node-1, node-2, node-3의 세 개의 노드가 실행되고 있다고 가정한다.
- 아래 명령어를 통해 먼저 node-1을 비운다.
  ```bash
  kubectl drain node-1
  ```
- 이후 ssh를 통해 노드에 접속하여 버전 업그레이드를 진행한다.
  ```bash
  apt-get upgrade -y kubeadm=1.12.0-00
  apt-get upgrade -y kubelet=1.12.0-00
  kubeadm upgrade node config --kubelet-version v1.12.0
  systemctl restart kubelet
  ```
- 업그레이드 이후, 통제(cordon)를 해제하여 스케줄링이 가능하도록 변경한다. 기존의 파드가 다시 해당 노드에 스케줄 되진 않지만 새로운 파드는 스케줄 될 것이다.
  ```bash
  kubectl uncordon node-1
  ```
- 위 과정을 각 워커 노드마다 반복한다.
- 아래 명령어를 통해 업그레이드가 적용되었는지 확인할 수 있다.
  ```bash
  kubectl get nodes
  ```

## Backup and Restore Methods

- 백업은 중요하다. 깃허브와 같은 코드 레포지토리에 저장을 하면 재사용에도, 누군가와 공유하기도 편하기 때문에 큰 문제가 되지 않는다.
- 하지만 누군가가 명령형 방식으로 리소스를 생성하였고, 이에 대한 문서도 전혀 남아 있지 않다면 어떻게 백업을 해야 할까?
  - 모든 리소스를 파일로 저장하는 방법이 있다. 이는 일부 리소스 그룹에만 해당된다.
    ```bash
    kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
    ```

### ETCD Backup

- etcd 클러스터는 클러스터의 상태에 대해 모든 정보를 저장한다.
- 모든 리소스를 백업하는 대신, etcd 서버 자체를 백업할 수 있다.
- etcd 클러스터는 마스터 노드에 호스팅되며, `--data-dir` 옵션을 통해 모든 데이터가 저장되는 위치를 확인할 수 있다.

### ETCD Snapshot

- etcd에는 스냅샷 솔루션이 내장되어 있다.
- 아래 명령어를 통해 snapshot.db라는 스냅샷을 생성할 수 있다.
  ```bash
  ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    snapshot save snapshot.db
  ```
- 아래 명령어를 통해 저장된 스냅샷 파일을 확인할 수 있다.
  ```bash
  ETCDCTL_API=3 etcdctl snapshot status snapshot.db
  ```

### ETCD Restore

- 아래 명령어를 통해 kube-apiserver를 중지시킨다.
  ```bash
  service kube-apiserver stop
  ```
- 아래 명령어를 통해 etcd 클러스터를 복구한다. `--data-dir` 옵션에 명시한 경로에 새로운 데이터 디렉토리가 생성된다.
  ```bash
  ETCDCTL_API=3 etcdctl \
    snapshot restore snapshot.db \
    --data-dir /var/lib/etcd-from-backup
  ```
- etcd.service의 `--data-dir` 옵션을 수정하여 새로운 데이터 디렉토리를 사용하도록 한다.
- 이후 etcd를 재시동하고 kube-apiserver를 실행한다.
  ```bash
  systemctl daemon-reload
  service etcd restart
  service kube-apiserver start
  ```

### ETCD가 스태틱 파드인 경우 복구하기

- 아래 명령어를 통해 etcd 클러스터를 복구한다. `--data-dir` 옵션에 명시한 경로에 새로운 데이터 디렉토리가 생성된다.

  ```bash
  ETCDCTL_API=3 etcdctl \
    snapshot restore snapshot.db \
    --data-dir /var/lib/etcd-from-backup
  ```

- /etc/kubernetes/manifests/etcd.yaml에서 hostPath.path 필드를 수정하여 새로운 데이터 디렉토리를 사용하도록 한다.
  ```yaml
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
  ```
