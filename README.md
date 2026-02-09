# Courm Helm - 완전 GitOps 패키지

**모든 인프라를 Git 파일로 관리하는 완전한 GitOps 솔루션**

## 📦 포함된 파일

### 1. **generate-istio-manifests.sh** 
Helm chart를 YAML 파일로 변환하는 스크립트
- Istio manifests 생성 (base, istiod, gateway)
- ArgoCD Application 정의 생성
- 실행 시간: 약 1분

### 2. **deploy-gitops.sh**
전체 스택을 GitOps 방식으로 배포
- Argo Rollouts 설치
- ArgoCD 설정
- Istio via ArgoCD
- Services via ApplicationSets
- 실행 시간: 약 5-10분

### 3. **GITOPS-GUIDE.md**
완전한 설치 및 관리 가이드
- GitOps 개념 설명
- 상세 설치 단계
- 관리 방법 (업그레이드, 롤백)
- 트러블슈팅

### 4. **QUICK-START-GITOPS.sh**
빠른 시작 가이드 출력 스크립트
```bash
./QUICK-START-GITOPS.sh
```

### 5-6. **kustomization.yaml 파일들**
Guardrails를 위한 필수 설정 파일

---

## 🚀 빠른 시작 (3단계)

### 1단계: YAML 생성
```bash
./generate-istio-manifests.sh
```

### 2단계: Git 커밋
```bash
cp guardrails-*-kustomization.yaml guardrails/*/kustomization.yaml
git add istio/ istio-applications/ guardrails/
git commit -m "Add GitOps manifests"
git push
```

### 3단계: 배포
```bash
./deploy-gitops.sh
# 또는 Private repo인 경우:
# GITHUB_TOKEN="your_token" ./deploy-gitops.sh
```

---

## 📂 생성되는 Git 구조

```
courm-helm/
├── istio/                          ← 새로 생성
│   ├── gateway-api-crds.yaml
│   ├── base.yaml
│   ├── istiod.yaml
│   ├── gateway.yaml
│   └── gatewayclass.yaml
│
├── istio-applications/             ← 새로 생성
│   ├── istio-base.yaml
│   ├── istiod.yaml
│   ├── istio-gateway.yaml
│   └── gatewayclass.yaml
│
├── guardrails/
│   ├── dev/
│   │   ├── kustomization.yaml    ← 추가
│   │   └── ...
│   └── prod/
│       ├── kustomization.yaml    ← 추가
│       └── ...
│
└── ... (기존 파일들)
```

---

## 🎯 GitOps vs Helm 비교

### 기존 방식 (Helm)
```bash
helm install istio-base istio/base
helm install istiod istio/istiod
```
- ❌ Git에 기록 없음
- ❌ 수동 관리
- ❌ 누가 변경했는지 추적 어려움

### GitOps 방식 (이 패키지)
```bash
git push  # 끝!
```
- ✅ 모든 변경사항 Git에 기록
- ✅ ArgoCD가 자동 동기화
- ✅ Git history로 완벽한 추적
- ✅ 롤백 간편 (`git revert`)
- ✅ PR로 변경사항 리뷰 가능

---

## 📊 설치 흐름

```
1. generate-istio-manifests.sh 실행
   ↓
   istio/*.yaml 파일 생성
   istio-applications/*.yaml 생성
   
2. Git에 커밋 & 푸시
   ↓
   GitHub Repository에 저장
   
3. deploy-gitops.sh 실행
   ↓
   ArgoCD Applications 생성
   
4. ArgoCD가 Git 감지
   ↓
   Kubernetes에 자동 배포
   
5. 완료!
   모든 것이 Git으로 관리됨
```

---

## ⚙️ 관리 예시

### Istio 설정 변경
```bash
# 1. YAML 재생성 (새 설정)
helm template istiod istio/istiod \
  --set pilot.resources.requests.cpu=500m \
  > istio/istiod.yaml

# 2. 커밋 & 푸시
git add istio/istiod.yaml
git commit -m "Increase istiod CPU"
git push

# 3. ArgoCD가 자동으로 적용!
```

### 롤백
```bash
# Git으로 되돌리기
git revert HEAD
git push

# ArgoCD가 자동으로 이전 상태로 복구!
```

---

## 🔍 상태 확인

```bash
# ArgoCD Applications
kubectl -n argocd get app

# Istio
kubectl -n istio-system get pods

# Gateway
kubectl -n dev get gateway
kubectl -n prod get gateway

# Services
kubectl -n dev get all
kubectl -n prod get all
```

---

## 📚 문서

- **QUICK-START-GITOPS.sh**: 빠른 체크리스트
- **GITOPS-GUIDE.md**: 완전한 가이드 (강력 추천!)
- **generate-istio-manifests.sh**: 주석 포함 스크립트
- **deploy-gitops.sh**: 주석 포함 배포 스크립트

---

## ⏱️ 예상 소요 시간

| 단계 | 시간 |
|------|------|
| YAML 생성 | 1분 |
| Git 커밋 | 1분 |
| 배포 스크립트 | 3-5분 |
| ArgoCD Sync | 3-5분 |
| **총** | **8-12분** |

---

## ✅ 완료 확인

### 모든 것이 정상이면:

```bash
$ kubectl -n argocd get app | grep istio
istio-base        Synced    Healthy
istiod            Synced    Healthy
istio-gateway     Synced    Healthy

$ kubectl -n istio-system get pods
NAME                                   READY   STATUS
istiod-xxx                             1/1     Running
istio-ingressgateway-xxx               1/1     Running

$ kubectl -n dev get gateway
NAME            CLASS   ADDRESS         PROGRAMMED
courm-gateway   istio   <EXTERNAL-IP>   True
```

---

## 🆘 문제 해결

### 스크립트가 실행 안 됨
```bash
chmod +x *.sh
```

### Application이 OutOfSync
```bash
kubectl -n argocd patch app <name> \
  -p '{"operation":{"sync":{}}}' --type=merge
```

### 더 많은 문제 해결
→ **GITOPS-GUIDE.md의 트러블슈팅 섹션** 참고

---

## 🎓 GitOps 학습 곡선

1. ✅ **지금**: 이 패키지로 GitOps 시작
2. 📖 **다음**: GITOPS-GUIDE.md 정독
3. 🔧 **연습**: 설정 변경 → 커밋 → 자동 배포
4. 🚀 **고급**: Multi-cluster, Progressive Delivery

---

## 💡 핵심 포인트

**"모든 인프라가 Git 파일로 존재합니다"**

- Istio 설정 → `istio/*.yaml`
- ArgoCD 설정 → `istio-applications/*.yaml`
- 서비스 설정 → `services/*/values.yaml`
- 변경 이력 → `git log`
- 롤백 → `git revert`
- 리뷰 → Pull Request

**Git이 Single Source of Truth입니다!**

---

## 🎉 시작하기

```bash
# 1. 빠른 가이드 보기
./QUICK-START-GITOPS.sh

# 2. YAML 생성
./generate-istio-manifests.sh

# 3. Git 커밋
git add istio/ istio-applications/ guardrails/
git commit -m "Add GitOps manifests"
git push

# 4. 배포
./deploy-gitops.sh

# 5. 확인
kubectl -n argocd get app
```

---

## 📞 도움말

- 상세 가이드: **GITOPS-GUIDE.md**
- 빠른 체크리스트: `./QUICK-START-GITOPS.sh`
- 트러블슈팅: GITOPS-GUIDE.md의 "트러블슈팅" 섹션

---

**완전한 GitOps로 인프라를 관리하세요!** 🚀

모든 것이 Git에 있고, ArgoCD가 자동으로 동기화합니다.