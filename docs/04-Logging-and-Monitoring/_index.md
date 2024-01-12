# Logging and Monitoring

## Monitor Cluster Components

- 노드, 파드 등의 상태를 모니터링해야 한다.
- 하지만 쿠버네티스에서는 이러한 모든 데이터를 모니터링할 수 있는 내장 모니터링 솔루션을 완벽하게 제공하지 않는다.
- Metrics-Server, Prometheus, Elastic Stack 등의 오픈소스 솔루션이나 독점 소유권을 가진 Datadog 혹은 Dynatrace 같은 솔루션을 통해 모니터링한다.

### Metrics-Server

- 쿠버네티스 클러스터 당 하나의 메트릭 서버를 가질 수 있다.
- 메트릭 서버는 각 노드와 파드로부터 모니터링 데이터를 검색한 뒤, 이를 종합하여 메모리에 저장한다.
- 메트릭 서버는 인메모리 모니터링 솔루션이기 때문에 측정 항목을 디스크에 저장하지 않는다.

### Getting Started

- minikube 사용 시, 아래 명령어를 통해 메트릭 서버를 추가할 수 있다.
  ```bash
  minikube addons enable metrics-server
  ```
- 아래 명령어를 통해 실행 중인 파드나 노드의 CPU와 메모리 사용량을 쉽게 확인할 수 있다.
  ```bash
  kubectl top pod pod01
  ```
