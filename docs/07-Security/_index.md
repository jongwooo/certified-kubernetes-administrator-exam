# Security

## Security Primitives

- 호스트에 대한 모든 액세스는 보안되어야 한다.
- 루트 액세스는 비활성화되어야 하며, 암호 기반 인증도 비활성화되어야 한다.
- 오직 SSH Key를 기반으로 한 인증만 사용할 수 있다.
- kube-apiserver는 쿠버네티스의 모든 동작의 중심으로, api에 직접 접근하거나 kubectl을 통해 상호작용하게 된다.
- 첫 번째 방어선인 kube-apiserver에 대한 접근을 어떻게 제어할 수 있을까? 아래의 두 가지로 결정해야 한다:
  - 누가 클러스터에 접근할 수 있는가?
  - 무엇을 할 수 있을까?

### Who can access?

- 인증 메커니즘에 의해 결정된다. 여러 가지 방법이 존재한다.
  - Files - Username and Passwords
  - Files - Username and Tokens
  - Certificates
  - External Authentication providers - LDAP
  - Service Accounts

### What can they do?

- RBAC Authorization - Role Based Access Control
- ABAC Authorization - Attribute Based Access Control
- Node Authorization
- Webhook Mode
- etcd 클러스터, kube-controller-manager, kube-scheduler, kube-apiserver 와 같은 다양한 구성 요소 사이의 모든 통신은 TLS Encryption 에 의해 보호된다.
- 클러스터 내의 애플리케이션 간의 통신은 네트워크 폴리시에 의해 제한된다.
