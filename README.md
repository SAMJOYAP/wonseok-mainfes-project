# 📦 Manifest Repository (wonseok-mainfes-project)

## 1. Repository 개요

이 레포지토리는 **Kubernetes 배포를 위한 Manifest 파일만 관리**합니다.  
애플리케이션 소스 코드나 빌드 로직은 포함하지 않으며,  
GitHub Actions와 ArgoCD를 통해 **GitOps 기반 CD(Continuous Deployment)** 를 수행합니다.

---

## 2. 사용 기술

- Kubernetes
- ArgoCD
- GitHub

---

## 3. 개념 설명

### GitOps

GitOps는 **Git 저장소에 정의된 선언적 설정을 기준으로 시스템을 운영**하는 방식입니다.

- Kubernetes 클러스터의 원하는 상태는 Git에 정의됩니다.
- 배포 변경은 `kubectl` 명령이 아닌 **Git 변경을 통해서만 수행**됩니다.
- Git에 반영된 변경 사항을 기준으로 실제 클러스터 상태를 동기화합니다.

---

### ArgoCD 역할

ArgoCD는 Manifest Repository를 감시하며 다음 역할을 수행합니다.

- Git 저장소의 변경 사항 지속 감시
- Kubernetes 클러스터 상태와 Git 상태 비교
- 변경 사항 발생 시 자동 동기화(Sync)
- 배포 이력 및 상태 시각화

---

## 4. 디렉터리 구조

```text
wonseok-mainfes-project/
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

### deployment.yaml
- 애플리케이션 Pod 구성
- Docker 이미지 정보 및 실행 설정 정의
- Replicas: 3개
- 이미지: ws5670/pipeline:main-{SHA}

### service.yaml
- 외부에서 애플리케이션에 접근할 수 있도록 서비스 노출
- Type: NodePort
- Port: 30090

---

## 5. CI/CD 흐름

```
[1] 소스코드 푸시
    ↓
[2] GitHub Actions 실행 (wonseok-app-repo)
    - Docker 이미지 빌드
    - Docker Hub에 푸시
    ↓
[3] Manifest 자동 업데이트 (이 저장소)
    - deployment.yaml 이미지 태그 변경
    ↓
[4] ArgoCD 감지
    ↓
[5] Kubernetes 클러스터 배포
```

---

## 6. ArgoCD 설정

### Application 생성

```bash
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: pipeline-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/SAMJOYAP/wonseok-mainfes-project.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
