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
