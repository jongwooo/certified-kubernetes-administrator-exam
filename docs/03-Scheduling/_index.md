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
- 스케줄러는 모든 파드를 보며 nodeName 필드가 설정되지 않은 파드를 탐색한다.
- 이후 스케줄링 알고리즘을 실행하여 파드에 적합한 노드를 식별한다.
- 식별되면 nodeName을 해당 노드의 이름으로 설정하여 노드에서 파드를 스케줄에 예약한다.

### 스케줄러가 없다면?

- 파드는 계속 Pending 상태가 된다.
- 스케줄러 없이 수동으로 직접 파드를 노드에 할당할 수 있다.
  - 가장 쉬운 방법은 해당 파드의 매니페스트 파일에 nodeName 필드로 노드를 직접 지정해주는 것이다.
- 파드 생성 시에만 nodeName을 직접 지정할 수 있다.
  - 파드가 이미 생성된 경우, nodeName을 수정하는 것은 허용되지 않는다.
  - 이 경우에는 바인딩 오브젝트를 정의하고, 바인딩 API에 POST 요청을 요청을 보내면 된다.
    - yaml 파일은 json 형식으로 변환하여 POST 요청의 body에 담아야 한다.
    - 이는 스케줄러가 하는 일을 매뉴얼하게 따라하는 것과 비슷하다.
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

## Labels and Selectors

- 레이블과 셀렉터는 그룹화하는 표준 방법이다.

### 어떻게 사용될까?

- 쿠버네티스에서 파드, 서비스, 디플로이먼트 등 다양한 다른 타입의 리소스를 생성할 수 있다.
- 다양한 오브젝트 사이에서 효율적으로 원하는 오브젝트를 선택하기 위해서는 여러 카테고리에 따라 그룹화 해야 한다.
- 레이블과 셀렉터를 사용하여 그룹화되어 있는 오브젝트를 필요에 따라 구분지어 찾을 수 있다.

### 어떻게 사용할까?

- 매니페스트 파일에서 키-값 쌍으로 지정하면 된다.
- 아래 명령어는 app이 App1인 파드를 찾는 예시이다.
  ```bash
  kubectl get pods --selector app=App1
  ```

### 어떻게 작동할까?

- 레이블과 셀렉터는 내부적으로 다른 오브젝트를 함께 연결한다.

**주의**

- metadata 필드의 label과 template 하위 필드의 label을 혼동하면 안된다.
  - metadata.labels: 매니페스트 구성, kind 요소 자체의 레이블
  - template.metadata.labels: 파드의 레이블
- 예를 들어 레플리카셋을 생성할 때, 파드의 레이블과 레플리카셋의 셀렉터가 일치해야 정상적으로 오브젝트가 생성된다.

### Annotations

- 레이블과 셀렉터는 그룹화에 사용되지만, 어노테이션은 그 외 세부사항을 기록하는데 사용된다.

## Taints and Tolerations

![Taints and Tolerations](./taints-and-tolerations.png)

- 테인트와 톨러레이션은 노드에서 스케줄링 될 수 있는 파드를 제한할 때 사용한다.
- 노드와 파드가 제한이 없는 경우, 밸런싱되어 파드는 노드에 스케줄링되어 할당된다.
- 특정 노드에 테인트를 추가하면, 일반적인 파드는 해당 노드에 할당될 수 없다.
- 파드에 노드의 테인트에 해당하는 톨러레이션을 추가하면, 해당 노드에 할당될 수 있다.
- 테인트 조건에 해당하지 않는 파드에 대한 옵션인 taint-effect는 세 가지가 있다:
  - NoSchedule: 테인트가 허용되지 않는 파드는 스케줄링하지 않는다. 이미 실행 중인 파드는 관여하지 않는다.
  - PreferNoSchedule: 테인트가 허용되지 않는 파드는 스케줄링하지 않으려고 한다. 하지만 클러스터 리소스가 부족한 상황 등에 대해서는 톨러레이션을 만족하지 않아도 노드에 스케줄링된다.
  - NoExecute: 테인트가 허용되지 않는 파드는 스케줄링하지 않는다. 이미 실행 중인 파드도 축출한다.

### How to use

- 아래 명령어를 통해 노드에 테인트를 적용할 수 있다.
  ```bash
  kubectl taint nodes node01 app=blue:taint-effect
  ```
- 아래 명령어를 통해 테인트를 확인할 수 있다.
  ```bash
  kubectl describe node node01 | grep Taint
  ```
- 톨러레이션은 아래와 같이 구성한다.
  ```yaml
  spec:
    containers:
      - name: nginx-container
        image: nginx
    tolerations:
      - key: "app"
        operator: "Equal"
        value: "bule"
        effect: "NoSchedule"
  ```
  - 톨러레이션과 관련된 모든 값은 큰따옴표로 묶어주어야 한다.

## Node Selectors

- 노드셀렉터는 파드의 nodeSelector 필드에 명시된 레이블을 가지고 있는 노드 중에 하나를 선택하여 스케줄링할 수 있도록 하는 기능이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

- 노드에도 해당 키 값에 해당하는 레이블을 지정해주어야 한다. 아래 명령어를 통해 레이블을 지정할 수 있다.
  ```bash
  kubectl label nodes node01 size=Large
  ```

### 한계

- 노드셀렉터는 다음과 같은 한계를 가지고 있다. 레이블 small, medium, large을 가진 노드가 있다고 가정하자.
  - medium, large 값을 가진 노드만 선택하고 싶다면 어떡할까?
  - 같은 의미로, small이 아닌 노드만 선택하고 싶다면 어떡할까?
- 이러한 한계를 보완하기 위해 나온 것이 노드어피니티 기능이다.

## Node Affinity

- 노드어피니티는 특정 노드에 파드를 확실하게 할당하기 위한 기능이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

### Node Affinity Types

**Available**

- requiredDuringSchedulingIgnoredDuringExecution: 규칙이 만족되지 않으면 스케줄러가 파드를 스케줄링할 수 없다.
- preferredDuringSchedulingIgnoredDuringExecution: 스케줄러는 조건을 만족하는 노드를 찾으려고 노력한다. 해당되는 노드가 없더라도, 스케줄러는 여전히 파드를 스케줄링한다.

**Planned**

- requiredDuringSchedulingRequiredDuringExecution: 규칙이 만족되지 않으면 스케줄러가 파드를 스케줄링할 수 없다. 또한 실행 중에 레이블이 변경되어 규칙에 만족되지 않으면
  파드는 축출된다.

## Node Affinity vs Taints and Tolerations

![Taints/Tolerations and Node Affinity](./taints-tolerations-and-node-affinity.png)

- 테인트와 톨러레이션으로 물리적으로 완전히 노드를 분리할 수는 없다. 예를 들어 blue, red, green으로 테인트한 노드가 있고, 테인트가 없는 노드가 두 개 있다. 파드 또한 각 blue, red,
  green의 톨러레이션을 추가한 파드가 있다.
- 이때 blue로 톨러레이션된 파드가 blue로 테인트한 노드에만 스케줄링된다고 보장할 수 있는가? 그렇지 않다. 테인트가 존재하지 않는 노드에도 스케줄링 될 수 있다.
- 두 번째 상황으로, blue, red, green의 레이블을 추가한 노드가 있으며 각 파드에도 노드어피니티를 사용하여 노드가 스케줄링되도록 적용한 상황이다. 이 상황에서의 문제는 어떠한 노드어피니티도 추가하지
  않은 파드가 레이블이 적용된 노드에 스케줄링 될 수 있다는 문제가 있다.
- 따라서 테인트/톨러레이션과 노드어피니티를 조합해서 사용해야 완전한 물리적 노드 분리가 가능해진다.

## Resource Requirements and Limits

- 각 노드마다 할당된 자원이 있다(e.g. CPU, 메모리, 디스크).
- 쿠버네티스 스케줄러에 의해 각 노드의 자원 상황에 맞게 파드가 할당된다.
- 그런데 어떠한 노드도 추가적인 파드를 감당할 자원이 부족한 상황이 발생하면 어떻게 될까?
- 쿠버네티스는 해당 파드를 스케줄링하는 것을 보류한다. 그리고 해당 파드는 Pending 상태가 된다.

### Resource

- 쿠버네티스는 기본값으로 파드에 0.5CPU, 256Mi 메모리를 할당한다.
- 이는 노드에 파드를 할당할 때 컨테이너에 의해 요청된 최소의 CPU/메모리의 양이다. 이 숫자로 파드를 할당할 충분한 리소스 여유가 있는지 판단한다.
- resources.requests.cpu와 resources.requests.memory 필드로 설정 가능하다.
  ```yaml
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
  ```

### Limit

- 한정된 자원을 설정할 수 있다.
  ```yaml
  resources:
    limits:
      memory: "2Gi"
      cpu: 2
  ```

### 초과하려고 하면?

- 만약 제한된 자원을 실행 중에 초과하려고 하면 어떻게 할까?
- CPU의 경우 쿠버네티스가 지정된 제한을 초과하지 않도록 조절한다.
  - 컨테이너는 제한된 CPU보다 많은 자원을 사용할 수 없기 때문이다.
- 메모리의 경우 다르다. 지정된 제한을 초과할 수 있다.
  - 컨테이너가 지정된 메모리보다 더 많은 메모리 자원을 사용할 수 있기 때문이다.
  - 하지만 파드가 지속적으로 제한된 자원보다 많은 메모리를 사용하려고 하면 파드는 Out Of Memory 에러가 발생하며 종료된다.

### LimitRange

- 기본값으로 정해진 자원을 선택하기 위해서는 해당 네임스페이스에서 리밋레인지라는 리소스를 사전에 정의해야 한다.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
    - default:
        memory: 1Gi
      defaultRequest:
        memory: 1Gi
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: Container
```
