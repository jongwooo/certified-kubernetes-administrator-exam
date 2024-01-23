# Storage

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
