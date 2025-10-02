---
title: "🧪 쿠버네티스 기초 실습 (Pod, Deployment, Service)"
date: 2025-10-02 11:00:00 +09:00
categories: ["Study", "Kubernetes"]
tags: ["k8s", "kubernetes", "deployment", "service"]
---

# 🧪 쿠버네티스 기초 실습 (Pod, Deployment, Service)

> 이 문서는 **Pod, Deployment, Service(NodePort)** 실습을 통해  
> 쿠버네티스 기본 리소스 개념을 검증하는 과정을 정리했습니다.

---

## 1. 실습 목표
- Pod → Deployment → Service 흐름 이해
- 라벨/셀렉터 일치의 중요성 확인
- NodePort로 외부 접근 테스트
- `get / describe / endpoints` 명령어로 연결 확인
- **Service 포트 동작 방식(port / targetPort / nodePort) 차이 이해**

---

## 2. 실습 환경
- Minikube (macOS)
- Docker Desktop (Windows/macOS도 가능)
- 쿠버네티스 v1.28 (kubectl 포함)

---

## 3. 매니페스트 정답 버전


### (1) Deployment (3개 파드 복제 관리)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apa000dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apa000kube
  template:
    metadata:
      labels:
        app: apa000kube
    spec:
      containers:
        - name: apa000ex91
          image: httpd:2.4
          ports:
            - containerPort: 80
```

### (2) Service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: apa000ser
spec:
  type: NodePort
  selector:
    app: apa000kube
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30080
```

---

## 4. Service 포트 종류 이해

Service 정의 시 나오는 3가지 포트는 **역할이 다름**.

| 키워드            | 의미                                                        | 동작 위치              | 예시      |
| -------------- | --------------------------------------------------------- | ------------------ | ------- |
| **port**       | Service가 클러스터 내부에 노출하는 포트                                 | ClusterIP(Service) | `80`    |
| **targetPort** | 트래픽을 전달할 실제 Pod 컨테이너 포트                                   | Pod(Container)     | `80`    |
| **nodePort**   | (NodePort 타입일 때) 클러스터 외부 접근을 위한 노드 고정 포트 (30000~32767 범위) | Node(호스트)          | `30080` |

👉 트래픽 흐름은 다음과 같다:

```
[ Client Browser ]
        │
        ▼
[ NodeIP:nodePort ]   ← 예: 192.168.99.100:30080
        │
        ▼
[ Service:port ]      ← 예: ClusterIP:80
        │
        ▼
[ Pod:targetPort ]    ← 예: PodIP:80
```

---

## 5. 리소스 생성
# -f: 지정한 YAML/JSON 파일을 적용(리소스 생성/업데이트)

```bash
kubectl apply -f manifests/apa000dep.yml
kubectl apply -f manifests/apa000ser.yml
```

---

## 6. 상태 확인

### (1) 전체 리소스 확인
# -o wide: 기본 출력보다 상세(노드, IP 등) 표시

```bash
kubectl get deploy,rs,pod,svc -o wide
```

### (2) Service 상세 확인
# describe: 리소스 상세 정보(셀렉터, 포트, 이벤트) 출력

```bash
kubectl describe svc apa000ser
```

확인 포인트:

* **Selector**: `app=apa000kube`
* **Ports**: `Port:80 → TargetPort:80, NodePort:30080`

### (3) Endpoints 확인
# endpoints 리소스 확인 + wide 옵션으로 IP까지 상세히 표시

```bash
kubectl get endpoints apa000ser -o wide
```

정상 예시:

```
NAME         ENDPOINTS                         AGE
apa000ser    10.244.1.12:80,10.244.2.7:80...  2m
```

👉 Pod IP + 80 포트가 보여야 정상 연결.
`<none>`이면 라벨/셀렉터 mismatch 또는 Pod NotReady 문제.

---

## 7. 접근 테스트

### (A) NodePort

```bash
curl -I http://<NodeIP>:30080/
```

### (B) 포트포워딩
 # 로컬 포트(8080) → 서비스 포트(80)로 트래픽 전달

```bash
kubectl port-forward svc/apa000ser 8080:80
curl -I http://localhost:8080/
```

---

| 옵션 / 명령어            | 의미                                      |
| ------------------- | --------------------------------------- |
| `-f <file>`         | 적용할 매니페스트 파일 지정 (YAML/JSON). 여러 개 가능.   |
| `-o wide`           | 출력 시 추가 정보 표시 (예: Pod의 IP, 노드 이름).      |
| `-o json / yaml`    | 리소스를 JSON 또는 YAML 형식으로 출력.              |
| `--selector` / `-l` | 특정 라벨로 리소스 필터링. 예: `-l app=apa000kube`. |
| `describe`          | 리소스 상세 정보 (이벤트 로그, 포트 매핑, 라벨 등).        |
| `port-forward`      | 로컬 포트 → 클러스터 리소스 포트로 포워딩. 개발/테스트에 유용.   |

---

## 8. 흔한 에러 & 해결

| 문제              | 원인                                     | 해결 방법                                                         |
| --------------- | -------------------------------------- | ------------------------------------------------------------- |
| Endpoints 비어 있음 | 라벨 불일치 / Pod NotReady                  | `kubectl get po -l app=apa000kube`, `kubectl describe po`로 확인 |
| NodePort 접근 불가  | 방화벽/보안그룹 미허용                           | `30080/TCP` 오픈                                                |
| Typo 발생         | `image**s`, `containersPort`, 라벨 불일치 등 | 올바른 YAML로 수정                                                  |

---

## 9. Service 타입 한눈에 정리

| 타입                 | 설명                                                           |
| ------------------ | ------------------------------------------------------------ |
| **ClusterIP** (기본) | 클러스터 내부 IP 부여, 외부 접근 불가. 내부 통신용.                             |
| **NodePort**       | 각 노드의 특정 포트(30000–32767) 오픈, 외부 접근 가능. 내부적으로 ClusterIP도 가짐.  |
| **LoadBalancer**   | 클라우드 제공 LB와 연계, 퍼블릭 IP 자동 할당. 외부 트래픽 → LB → ClusterIP → Pods |
| **ExternalName**   | 셀렉터/포트 없이 DNS CNAME을 반환. 클러스터 내부에서 외부 호스트 이름을 깔끔하게 가리킬 때 사용. |

👉 이번 실습에서는 **NodePort**를 다뤘지만, 실제 운영 환경에서는
주로 **LoadBalancer**(클라우드) 또는 **Ingress Controller**와 조합해서 사용함.

---

## 10. 정리

* Deployment → ReplicaSet → Pod 관계 확인 완료
* Service(NodePort)로 외부에서 접근 가능
* `get / describe / endpoints` 명령어로 **Service-Pod 연결 확인** 가능
* **port / targetPort / nodePort 차이** 확실히 이해
* Service 타입별 개념 정리까지 학습

---
