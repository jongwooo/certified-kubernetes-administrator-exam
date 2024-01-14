# Application Lifecycle Management

## Rolling Updates and Rollbacks

- 디플로이먼트가 처음 생성되면, 롤아웃이 트리거된다.
- 새로운 롤아웃은 새로운 디플로이먼트의 리비전을 생성한다.
- 리비전은 디플로이먼트가 변경사항을 추적하고자 할 때와 이전 버전으로 롤백하고자 할 때 도움을 준다.
- 두 개의 리비전이 유지되고 있기 때문에 새로운 버전으로 롤아웃/이전 버전으로 롤백하는 것이 가능하다

### 배포 전략

![Deployment Strategy](./deployment-strategy.png)

**Recreate**

- 기존 배포된 애플리케이션을 모두 삭제하고, 새 버전의 애플리케이션을 다시 생성한다.
- 삭제 후 새 버전을 생성하는 사이에 다운타임이 발생할 수 있다.
- 쿠버네티스의 기본 배포 전략이 아니다.

**Rolling Update**

- 하나씩 이전 버전을 종료하고 새로운 버전을 배포하는 방식이다.
- 어플리케이션 다운타임이 발생하지 않고 원활하게 새 버전을 배포할 수 있다.
- 쿠버네티스의 기본 배포 전략에 해당한다.

### Updates

- 매니페스트 파일의 변경 이후, `kubectl apply` 명령어를 통해 업데이트를 적용할 수 있다. 이를 통해 디플로이먼트의 새로운 리비전이 생성된다.
- `kubectl set image` 명령어를 통해 애플리케이션 이미지 업데이트를 진행할 수 있다. 이 방식은 yaml 파일 내 애플리케이션 이미지 버전을 변경하지는 않는다.
  ```bash
  kubectl set image deployment frontend simple-webapp=kodecloud/webapp-color:v2
  ```

### Commands

- 아래 명령어를 통해 롤아웃 상태를 확인할 수 있다.
  ```bash
  kubectl rollout status deployment/myapp-deploy
  ```
  - 아래 명령어를 통해 롤아웃의 기록을 확인할 수 있다.
  ```bash
  kubectl rollout history deployment/myapp-deploy
  ```
- 아래 명령어를 통해 이전 버전으로 롤백할 수 있다.
  ```bash
  kubectl rollout undo deployment/myapp-deploy
  ```

## Commands and Arguments

- Dockerfile의 ENTRYPOINT는 쿠버네티스의 command 필드와 대응되며, CMD는 쿠버네티스의 args 필드와 대응된다.

```dockerfile
FROM Ubuntu

ENTRYPOINT [ "sleep" ]

CMD [ "5" ]
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: [ "sleep2.0" ]
      args: [ "10" ]
```

## Configure Environment Variables in Applications

- 쿠버네티스에서 환경변수를 설정하는 방법은 다음과 같다.
  ```yaml
  env:
    - name: APP_COLOR
      value: pink
  ```
- 컨피그맵을 통해 환경변수를 설정하는 방법은 다음과 같다.
  ```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
  ```
- 시크릿을 통해 환경변수를 설정하는 방법은 다음과 같다.
  ```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
  ```

## Configuring ConfigMaps in Applications

- 환경변수를 파드 매니페스트 파일에 정의할 수 있지만, 환경변수를 각 파드별로 관리하는 것보다 컨피그맵을 통해 하나의 파일에서 모든 환경변수를 관리하는 것이 효율적이다.

### Create ConfigMaps

- `--from-literal` 옵션을 사용하여 명령형 방식으로 컨피그맵을 생성할 수 있다.
  ```bash
  kubectl create configmap \
    app-config --from-literal=APP_COLOR=blue \
               --from-literal=APP_MODE=prod
  ```
- 또는 `--from-file` 옵션을 통해 해당 파일을 읽어 컨피그맵을 생성할 수 있다.
  ```bash
  kubectl create configmap \
    app-config --from-file=app_config.properties
  ```
- 아래와 같이 선언형으로 컨피그맵을 생성할 수 있다.
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: "blue"
    APP_MODE: "prod"
  ```

### View ConfigMaps

- 아래 명령어를 통해 생성된 컨피그맵을 조회할 수 있다.
  ```bash
  kubectl get configmaps
  ```
- 아래 명령어를 통해 생성된 컨피그맵의 정보를 확인할 수 있다.
  ```bash
  kubectl describe configmaps
  ```

### ConfigMap in Pods

- 아래와 같이 envFrom.configMapRef 필드에 생성된 컨피그맵 이름을 지정할 수 있다.
  ```yaml
  envFrom:
    - configMapRef:
      name: app-config
  ```
- 단일 환경변수로 주입하려면 env.valueFrom.configMapKeyRef 필드에 생성된 컨피그맵 이름을 지정하고, 환경변수 이름을 key로 지정한다.
  ```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
  ```
- 또는 볼륨에 컨피그맵을 마운트할 수 있다.
  ```yaml
  volumes:
    - name: app-config-volume
      configMap:
        name: app-config
  ```
