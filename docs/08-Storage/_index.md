# Storage

[메인으로 돌아가기](../../README.md)

## Volumes

- 도커에서 컨테이너는 본질적으로 일시적이기 때문에 해당 컨테이너가 유지되는 동안만 사용할 수 있다.
- 컨테이너를 삭제할 경우 컨테이너에서 사용한 데이터가 유실되는 단점이 있는데, 이러한 단점을 보완하기 위한 기술이 볼륨이다.
- 볼륨을 사용하면, 컨테이너에 의해 처리된 데이터는 볼륨에 배치되기 때문에 데이터가 영구적으로 유지된다.
- 쿠버네티스에서도 마찬가지로 생성된 파드는 본질적으로 일시적이다.
- 파드를 생성하여 데이터를 처리한 후 삭제하면 파드에서 처리한 데이터도 삭제된다.

### Volumes & Mounts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: [ "/bin/sh", "-c" ]
      args: [ "shuf -i 0-100 -n 1 >> /opt/number.out;" ]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

- spec.volumeMounts 필드를 통해 데이터 볼륨을 컨테이너 내의 /opt 디렉토리에 마운트한다.
- volumes.hostPath 필드를 사용하여 디렉토리를 구성했기 때문에 호스트에는 볼륨에 대한 스토리지 공간이 있다. 이는 단일 노드에서는 정상적으로 작동하지만 다중 노드 클러스터에서의 사용은 적합하지 않다.
- 다중 노드 클러스터 환경에서는 파드가 모든 노드에서 /data 디렉토리를 사용하고, 모든 노드가 동일한 데이터를 가질 것이기 때문이다.
- volumes.hostPath 필드를 사용하여 다중 노드 클러스터 환경을 구성해야 한다면 AWS EBS와 같은 솔루션을 사용하자.

## Persistent Volumes

- 퍼시스턴트 볼륨은 애플리케이션을 배포하는 사용자가 사용하도록 구성한 클러스터 전체 스토리지 볼륨 풀이다.
- 관리자가 큰 풀의 스토리지를 생성하고, 사용자들이 필요한 만큼 조각으로 사용하는 방식이다.
- 퍼시스턴트 볼륨을 사용하기 위해서는 퍼시스턴트 볼륨 클레임이 필요하다. 퍼시스턴트 볼륨 클레임을 사용하여 스토리지 볼륨 풀에서 퍼시스턴트 볼륨을 선택할 수 있다.

### Create Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

- 아래 명령어를 통해 퍼시스턴트 볼륨을 생성할 수 있다.
  ```bash
  kubectl create -f pv-definition.yaml
  ```
- 아래 명령어를 통해 퍼시스턴트 볼륨 목록을 조회할 수 있다.
  ```bash
  kubectl get persistentvolume
  ```

## Persistent Volume Claims

- 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임은 쿠버네티스 네임스페이스에서 다른 오브젝트이다.
- 관리자는 퍼시스턴트 볼륨을 생성하고, 사용자는 스토리지에 사용할 퍼시스턴트 볼륨 클레임을 생성한다.
- 퍼시스턴트 볼륨 클레임이 생성되면 쿠버네티스는 퍼시스턴트 볼륨을 생성할 때의 구성에 따라 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임을 바인딩한다. 이때, 모든 퍼시스턴트 볼륨 클레임은 하나의 퍼시스턴트 볼륨과
  마운트된다.
- 바인딩 동안 쿠버네티스는 클레임에 의해 요청되는 대로 충분한 용량을 가진 퍼시스턴트 볼륨이나 다른 기타 요청 속성(Access Mode, Volume Modes, Storage Class, Selector,
  etc)을 찾으려고 한다.
- 여러 개의 퍼시스턴트 볼륨 구성에서 매칭되는 클레임이 하나라면 레이블과 셀렉터를 통해 매칭시킬 수 있다.
- 퍼시스턴트 볼륨의 용량은 큰데, 퍼시스턴트 볼륨 클레임의 용량이 작다면 바인딩 될 수 있다. 하지만 이 둘은 일대일 관계이므로 다른 클레임이 퍼시스턴트 볼륨의 남은 용량을 사용할 수는 없다.
- 사용 가능한 볼륨이 없는 경우, 클러스터에서 새 볼륨을 사용할 수 있게 될 때까지 Pending 상태가 된다.

### Create Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

- 아래 명령어를 통해 퍼시스턴트 볼륨 클레임을 생성할 수 있다.
  ```bash
  kubectl create -f pvc-definition.yaml
  ```
- 아래 명령어를 통해 퍼시스턴트 볼륨 클레임 목록을 조회할 수 있다.
  ```bash
  kubectl get persistentvolumeclaim
  ```

### Delete Persistent Volume Claim

- persistentVolumeReclaimPolicy 필드를 통해 퍼시스턴트 볼륨 클레임 삭제 이후 퍼시스턴트 볼륨을 관리할 수 있다.
  - Retain: 기본 설정으로, 관리자가 삭제하기 전까지 퍼시스턴트 볼륨은 남아 있다.
  - Delete: 퍼시스턴트 볼륨 클레임이 삭제되면 곧 퍼시스턴트 볼륨도 삭제된다.
  - Recycle: 다른 클레임이 사용할 수 있도록 퍼시스턴트 볼륨 내 데이터만 삭제한다.

### Using PVCs in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

- 퍼시스턴트 볼륨 클레임이 생성되면, 파드 매니페스트 파일의 volumes.persistentVolumeClaim 필드를 명시하여 퍼시스턴트 볼륨 클레임을 지정할 수 있다. 이는 레플리카셋이나 디플로이먼트에서도
  동일하다.

## Storage Class

- Google Cloud와 같은 서비스를 사용한다면, 퍼시스턴트 볼륨 생성 전에 Google Cloud Disk를 생성해야 한다.
- 수동으로 Disk를 프로비저닝한 후 퍼시스턴트 볼륨을 정의하는데, 이때 생성된 Disk와 이름을 매칭시켜 생성해야 한다. 이를 정적 프로비저닝이라고 한다.
- 이는 수동으로 작업해야 하기 때문에 불편하다. 이 작업을 도와주는 것이 스토리지 클래스이다.
- 스토리지 클래스를 사용하면 Google Cloud와 같은 서비스에서 스토리지를 자동으로 프로비저닝하고 클레임이 발생할 때 파드에 연결할 수 있는 퍼시스턴트 볼륨을 자동 생성할 수 있다. 이를 볼륨의 동적
  프로비저닝이라고 한다.

### Create Storage Class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

- 스토리지 클래스가 생성되면 스토리지가 자동 생성된다.
- spec.storageClassName 필드를 지정하여 퍼시스턴트 볼륨 클레임에서 스토리지 클래스를 사용할 수 있다.
- 스토리지 클래스는 GCP에 필요한 크기의 새 디스크를 프로비저닝한 다음 퍼시스턴트 볼륨을 생성한 뒤, 퍼시스턴트 볼륨 클레임을 해당 볼륨에 바인딩한다.
- 스토리지 클래스를 사용한다고 해서 퍼시스턴트 볼륨이 생성되지 않는 것은 아니다. 직접 생성할 필요가 없는 것이다.
