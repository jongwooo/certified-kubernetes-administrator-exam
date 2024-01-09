# Scheduling

## Manual Scheduling

### How scheduling works

![How scheduling works](how-scheduling-works.png)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  nodeName: node02
```

- 일반적으로는 쿠버네티스가 nodeName 필드를 자동으로 추가한다.
- 스케쥴러는 모든 파드를 보며 nodeName 필드가 설정되지 않은 파드를 탐색한다.
- 이후 스케쥴링 알고리즘을 실행하여 파드에 적합한 노드를 식별한다.
- 식별되면 nodeName을 해당 노드의 이름으로 설정하여 노드에서 파드를 스케쥴에 예약한다.

### 스케쥴러가 없다면?

- 파드는 계속 Pending 상태가 된다.
- 스케쥴러 없이 수동으로 직접 파드를 노드에 할당할 수 있다.
  - 가장 쉬운 방법은 해당 파드의 매니페스트 파일에 nodeName 필드로 노드를 직접 지정해주는 것이다.
- 파드 생성 시에만 nodeName을 직접 지정할 수 있다.
  - 파드가 이미 생성된 경우, nodeName을 수정하는 것은 허용되지 않는다.
  - 이 경우에는 바인딩 오브젝트를 정의하고, 바인딩 API에 POST 요청을 요청을 보내면 된다.
    - yaml 파일은 json 형식으로 변환하여 POST 요청의 body에 담아야 한다.
    - 이는 스케쥴러가 하는 일을 매뉴얼하게 따라하는 것과 비슷하다.
    ```yaml
    apiVersion: v1
    kind: Binding
    metadata:
      name: nginx
    target:
      apiVersion: v1
      kind: Node
      name: node02
    ```
