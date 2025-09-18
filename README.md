---
layout: post
title: "Kubernetes 기본 학습 정리 (2일차)"
date: 2025-09-18 22:00:00 +0900
categories: [Study, Kubernetes]
tags: [docker, kubernetes, k8s, container, orchestration, study-note]
---

## 🚀 Kubernetes란?
컨테이너 오케스트레이션 도구로, **여러 서버(노드)에 분산된 컨테이너를 자동으로 생성·배치·관리**한다.  
시스템 전체를 통합적으로 관리하며, 사용자가 정의한 상태(Desired State)를 항상 유지하려고 한다.

- 컨테이너 오케스트레이션 = "컨테이너 관리 자동화"
- 원하는 상태를 **매니페스트(YAML)** 파일로 정의
- 쿠버네티스는 이를 `etcd`에 저장하고, 항상 실제 상태를 바람직한 상태로 맞춘다

---

## 🔑 K8s
- `kubernetes`의 축약형  
- `k`와 `s` 사이에 8글자가 있어 **k8s**라고 부른다  

---

## 🖥️ 클러스터 구조

### 🧠 Control Plane (마스터 노드)

쿠버네티스 클러스터의 **두뇌 역할**을 담당하는 부분으로,  
클러스터 전체 상태를 관리하고 워커 노드를 제어한다.  
컨테이너를 직접 실행하지 않고, 오직 **관리 기능**에 집중한다.

👉 **마스터 노드 내부에는 컨트롤 플레인 컴포넌트들이 존재하며,  
이 컴포넌트들이 클러스터를 제어한다.**  
즉, **마스터 노드 = 컨트롤 플레인을 실행하는 노드**라고 이해하면 된다.

#### 컨트롤 플레인 주요 구성 요소
- **kube-apiserver**: 클러스터와 통신하는 유일한 관문. `kubectl` CLI 요청을 받아 내부 컴포넌트에 전달.  
- **kube-controller-manager**: 리소스 상태를 감시하고 원하는 상태로 유지. (예: 파드 수 복구)  
- **kube-scheduler**: 새 파드를 어느 노드에 배치할지 결정. (리소스 사용량, 레이블 등 고려)  
- **etcd**: 클러스터 상태 및 모든 메타데이터 저장소. “Single Source of Truth”  
- **cloud-controller-manager**: 클라우드 연동(AWS, GCP, Azure 등) 관련 기능 담당  

---

### 🖥️ Worker Node (워커 노드)
컨테이너가 실제로 실행되는 곳. 파드(Pod)와 볼륨이 동작한다.

- **kubelet**: 마스터와 통신. 파드 생성/삭제 및 상태 관리.  
- **kube-proxy**: 네트워크 프록시. 파드/서비스 간 통신 처리.  
- **Container Runtime**: Docker, containerd, CRI-O 등 실제 컨테이너 실행 엔진.  
- **CNI 플러그인**: Pod ↔ Pod, Node ↔ Node 네트워킹 담당 (Flannel, Calico 등).  

---

## 🌐 쿠버네티스 네트워킹 (CNI)

쿠버네티스 자체에는 네트워크 기능이 내장되어 있지 않다.  
**CNI(Container Network Interface)** 표준을 제공하고, 실제 네트워크는 플러그인이 담당한다.

- **Flannel**: 가장 간단한 오버레이 네트워크  
- **Calico**: 네트워크 + 보안 정책 지원  
- **Weave Net**: 자동 네트워크 메쉬  
- **Cilium**: eBPF 기반 고성능 네트워킹/보안  

---

## ⚙️ 쿠버네티스의 핵심 원리
- **Desired State 유지**: 매니페스트(YAML)에 정의된 상태를 항상 지키려고 함.  
  - 예: 파드 5개 설정 → 1개 삭제하면 자동으로 다시 1개 생성.  
- **Self-healing (자가치유)**: 항상 올바른 상태로 자동 복구.  

---

## 🔑 쿠버네티스 핵심 리소스

- **Pod**: 컨테이너를 실행하는 최소 단위. (컨테이너+볼륨+네트워크 환경)  
- **ReplicaSet**: 동일한 Pod의 개수를 관리. 스케일 인/아웃 담당.  
- **Deployment**: ReplicaSet을 관리하는 상위 개념. 버전 배포/롤링 업데이트/롤백 지원.  
- **Service**: Pod 집합에 대한 네트워크 접근 제공. 로드밸런싱 기능 포함.  
- **Ingress**: 외부 요청을 내부 서비스로 라우팅. (도메인 기반, SSL 종료 지원)  
- **etcd**: 쿠버네티스의 모든 객체와 상태 저장. 백업/복원 핵심 포인트.  

---

## ✅ 오늘의 정리
- 쿠버네티스는 선언적 상태 관리 기반의 오케스트레이터  
- 클러스터 = 마스터 노드(Control Plane) + 워커 노드  
- 리소스 관계: Pod → ReplicaSet → Deployment → Service → Ingress  
- 모든 상태는 `etcd`에 저장, 컨트롤러가 자동으로 Desired State 유지  

---

