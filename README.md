# Pipeline - Kubernetes Manifest Repository

Kubernetes 매니페스트 저장소 (GitOps)

## 📦 Repository 구조

```
wonseok-mainfes-project/
└─ k8s/
   ├─ deployment.yaml
   └─ service.yaml
```

## 🔄 GitOps 워크플로우

1. [wonseok-cicd-project](https://github.com/SAMJOYAP/wonseok-cicd-project)에 코드 푸시
2. GitHub Actions가 Docker 이미지 빌드 후 Docker Hub에 올림
3. GitHub Actions가 여기 `deployment.yaml` 이미지 태그 자동 업데이트
4. ArgoCD가 감지해서 클러스터에 배포

## 🐳 Docker 이미지

- Repository: `ws5670/pipeline`
- Tag: `main-{7자리 SHA}`

## 🎯 ArgoCD Application 생성

```bash
argocd app create pipeline-app \
  --repo https://github.com/SAMJOYAP/wonseok-mainfes-project.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```

UI에서 만들려면:
- Application Name: `pipeline-app`
- Repository: 위 URL
- Path: `k8s`
- Cluster: `https://kubernetes.default.svc`
- Namespace: `default`
- Sync Policy: Automatic 체크

## 📊 배포 리소스

**Deployment**
- Name: cicd-demo-app
- Replicas: 3
- Port: 8080

**Service**
- Name: cicd-demo-app-service
- Type: NodePort
- Port: 80 → 8080
- NodePort: 30090

## 🔍 확인

```bash
kubectl get pods -l app=cicd-demo-app
kubectl get svc cicd-demo-app-service

# 접속: http://<node-ip>:30090
curl http://<node-ip>:30090/health
```

## ⚙️ ArgoCD 설치

```bash
# 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 외부 노출
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# 초기 비밀번호
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## 🔗 관련 Repository

- 소스코드: [wonseok-cicd-project](https://github.com/SAMJOYAP/wonseok-cicd-project)

## ⚠️ 참고

deployment.yaml의 이미지 태그는 GitHub Actions가 관리함. 직접 수정 X
