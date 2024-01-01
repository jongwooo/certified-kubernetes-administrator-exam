# Core Concepts
## Cluster Architecture
### 쿠버네티스의 목적
- 애플리케이션을 컨테이너 형식으로, 자동화된 방식으로 호스팅하는 것.
	- 요구에 따라 애플리케이션의 많은 인스턴스를 쉽게 배포할 수 있다.
	- 애플리케이션 내 다양한 서비스 간의 통신이 쉽게 가능하다.

### Kubernetes Cluster Architecture
- 쿠버네티스가 하는 역할은 크게 두 가지로 나뉜다. 배로 비유하자면:
	- Cargo Ships that Carries Containers.
	- Monitoring / Managing the Cargo Ships.

![Kubernetes Cluster Architecture](./kubernetes-cluster-architecture.png)

**노드**
> 쿠버네티스 클러스터는 위의 역할을 담당하기 위한 **노드**의 집합이다.

- **워커 노드**: 컨테이너를 Loading하기 위한 용도의 Cargo Ship.

- **마스터 노드**: 컨테이너를 올릴 Cargo Ship(Worker Node)을 관리하는 Control Ship.
	- 어떤 종류의 컨테이너를 올릴 것인지 정의한다.
	- 관리하고 있는 Ship의 정보를 저장한다.
	- 컨테이너의 상태를 모니터링하고 추적한다 (Manage the Whole Loading Process 등).

**컨트롤 플레인 컴포넌트**
> 마스터 노드는 위에 정의한 작업들을 수행하기 위해 **컨트롤 플레인 컴포넌트**라는 것을 사용한다.

- **etcd**: 언제, 어떤 컨테이너가 어떤 노드에 올라갔는 지를 저장하기 위한 키-값 저장소.
- **kube-scheduler**: 노드가 배정되지 않은 새로 생성된 파드를 감지하고, 실행할 노드를 선택한다.
	- resource requirement
	- 워커 노드의 capacity
	- 기타 policy / constraint 고려 (e.g. node-affinity rule)
- **kube-controller-manager**
	- **노드 컨트롤러**: 클러스터에 있는 노드를 관리한다.
		- 새 노드가 추가되거나, 사용 불가능하거나 파괴되는 상황을 처리한다.
	- **레플리케이션 컨트롤러**: 파드가 특정 개수만큼 복제되고 동작하는 것을 보장한다.
- **kube-apiserver**: 클러스터 내에서 모든 작업을 오케스트레이션한다.
	- 쿠버네티스 API를 노출하는 컨트롤 플레인 컴포넌트이다.

**노드 컴포넌트**
> **노드 컴포넌트**는 동작 중인 파드를 유지시키고 쿠버네티스 런타임 환경을 제공하며, 모든 노드 상에서 동작한다.

- **컨테이너 런타임**: 컨테이너 실행을 담당하는 소프트웨어이다 (e.g. Docker, containerd)
- **kubelet**: 클러스터의 각 노드에서 실행되는 에이전트이다.
	- kube-apiserver에서 오는 요청을 받아서 노드에 적용한다.
	- 주기적으로 kube-apiserver에서 노드와 컨테이너 상태를 모니터링하기 위해 상태 보고서를 가져온다.
- **kube-proxy**: 실행 중인 컨테이너 애플리케이션 간 통신을 담당한다.
	- 각 워커 노드에서 개별적으로 실행되는 서비스이다.
