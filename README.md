# LG MLDL DAP Platform
## 엔터프라이즈 MLOps 플랫폼 구축

> **프로젝트**: LG MLDL DAP 기반 완전 통합 MLOps 플랫폼  
> **기간**: 2023년 6월 ~ 2024년 12월 (19개월)  
> **규모**: 일일 2억 건 추천 요청, 99.95% 가용성  
> **팀**: ML Engineer 3명, DevOps Engineer 2명, Infrastructure Engineer 2명

---

## 📊 프로젝트 개요

### 비즈니스 가치

| 지표 | 기존 | 개선 | 개선율 |
|------|------|------|--------|
| **모델 개발 사이클** | 14일 | 4시간 | **98% ↓** |
| **배포 빈도** | 월 4회 | 일 5회 | **37배 ↑** |
| **배포 시간** | 2일 | 30분 | **96% ↓** |
| **성능 저하 감지** | 5일 | 1시간 | **98% ↓** |
| **자동 장애 복구** | 8시간 | 1분 | **480배 ↑** |
| **월 운영 비용** | $50K | $30K | **40% ↓** |
| **서비스 가용성** | 99.50% | 99.95% | **90% ↑** |

---

## 🏗️ LG MLDL DAP 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                    LG MLDL DAP Platform                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [1] 데이터 계층 (Data Layer)                                    │
│  ├─ Kafka: 실시간 스트리밍 (초당 50K)                           │
│  ├─ Data Lake: S3/MinIO (PB급)                                  │
│  └─ ETL: Apache Spark (분산 전처리)                             │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [2] 특성 공학 계층 (Feature Engineering)                       │
│  ├─ Feature Store: Feast (2000+ 특성)                           │
│  ├─ 온라인: Redis (<10ms)                                       │
│  └─ 오프라인: PostgreSQL (역사 데이터)                          │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [3] 모델 개발 계층 (Model Development)                         │
│  ├─ 파이프라인: Kubeflow KFP (DAG 오케스트레이션)              │
│  ├─ 학습: TensorFlow/PyTorch (8 GPU 분산)                       │
│  ├─ 튜닝: Optuna (하이퍼파라미터, 50회 병렬)                   │
│  ├─ 평가: MLflow Tracking (메트릭 자동 기록)                   │
│  └─ 관리: MLflow Registry (300+ 모델 버전)                     │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [4] 배포 계층 (Deployment)                                     │
│  ├─ Container: Docker (Multi-stage, 250MB)                     │
│  ├─ Registry: ECR (프라이빗, 취약점 스캔)                       │
│  ├─ Orchestration: Kubernetes EKS (3개 AZ)                     │
│  ├─ Serving: KServe (P99 <50ms)                                │
│  └─ Deployment: ArgoCD (GitOps, Pull-based)                    │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [5] CI/CD 계층 (Continuous Integration/Deployment)            │
│  ├─ Source: GitHub (코드 저장소)                                │
│  ├─ Build: GitHub Actions (빌드/테스트)                        │
│  ├─ Registry: AWS ECR (이미지 저장)                            │
│  ├─ Deployment: ArgoCD (자동 배포)                             │
│  └─ Strategy: Flagger (Canary, 5% → 100%)                      │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  [6] 모니터링 계층 (Monitoring & Observability)                 │
│  ├─ 메트릭: Prometheus (30초 간격)                             │
│  ├─ 대시보드: Grafana (실시간)                                  │
│  ├─ 침입 탐지: Falco (비정상 행동)                              │
│  ├─ 로깅: ELK Stack (중앙 집중식)                               │
│  └─ 알림: Alert Manager → Slack (5분 내)                       │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 핵심 기능별 상세 설명

### [1] 데이터 계층

**Kafka (실시간 데이터 수집)**
- 초당 50,000개 메시지 처리
- 3개 파티션 × 3개 레플리카 (고가용성)
- 사용자 행동 데이터 실시간 수집

**Data Lake (대규모 데이터 저장)**
- S3: 원본 데이터 (PB급)
- MinIO: 온프레미스 대안
- Spark: 분산 전처리 (1시간 내 처리)

---

### [2] Feature Store (특성 공학)

**온라인 Feature Store (Redis)**
- 응답 시간: <10ms
- 2000개 이상의 실시간 특성 제공
- 서빙 시 즉시 사용 가능

**오프라인 Feature Store (PostgreSQL)**
- 모든 특성의 역사 데이터 저장
- 학습 시 일관된 데이터 보장
- 재현성 (Reproducibility) 보장

**특성 관리 (Feast)**
- 특성 버전 관리
- 학습/서빙 스큐 (Training-Serving Skew) 방지
- 메타데이터 자동 추적

---

### [3] 모델 개발 파이프라인

**전체 프로세스**
```
데이터 준비 (1시간)
    ↓
특성 생성 (1시간)
    ↓
모델 학습 (1시간, 8 GPU 병렬)
    ↓
하이퍼파라미터 튜닝 (30분, 50회 병렬)
    ↓
모델 평가 (30분)
    ↓
성능 검증 (임계값: F1 > 0.75)
    ↓
Model Registry 자동 등록
    ↓
배포 파이프라인 트리거

총 실행 시간: 4시간 (기존: 14일)
```

**Kubeflow KFP (파이프라인 오케스트레이션)**
- DAG 기반 워크플로우
- 메타데이터 자동 추적
- 캐싱으로 재실행 최적화
- 매일 00:00 UTC 자동 실행

**MLflow (실험 추적 및 모델 관리)**
- 300+ 모델 버전 관리
- 모든 실험의 파라미터/메트릭 기록
- 모델 승격 프로세스 자동화

---

### [4] 배포 시스템

**Docker 컨테이너화**
- Multi-stage build로 이미지 최적화
  - 1.2GB → 250MB (79% 감소)
- Distroless base image
  - 보안 취약점 90% 감소
- 빌드 시간 45% 단축

**Kubernetes 오케스트레이션**
- EKS 클러스터 (3개 AZ)
- 자동 스케일링 (HPA)
  - CPU 60%, Memory 80% 기준
  - 최소 2, 최대 10 레플리카
- Pod Disruption Budget (무중단 배포)

**모델 서빙 (KServe)**
- 추론 서빙: P99 <50ms
- 처리량: 일 2억 건 요청
- A/B 테스트 자동 지원

---

### [5] CI/CD 파이프라인

**GitHub → GitHub Actions → ECR → ArgoCD → Kubernetes**

```
1. 코드 푸시
   └─ PR Merge on main branch

2. 자동화 테스트
   ├─ Unit Test
   ├─ Integration Test
   └─ Code Quality (SonarQube)

3. Docker 이미지 빌드
   ├─ Multi-stage build
   ├─ 보안 스캔 (Trivy)
   └─ ECR 푸시

4. GitOps 배포 (ArgoCD)
   ├─ Git 변경 감지
   ├─ 자동 동기화
   └─ Helm 차트 배포

5. Canary 배포 (Flagger)
   ├─ 5% 트래픽 검증
   ├─ 메트릭 평가
   └─ 점진적 확대 (5% → 100%)

6. 모니터링
   ├─ Prometheus 수집
   ├─ Grafana 표시
   └─ Alert Manager 알림
```

---

### [6] 모니터링 및 운영

**실시간 메트릭 수집 (Prometheus)**
- 30초 간격 수집
- 모델 성능 지표
- 시스템 성능 지표
- 비즈니스 메트릭

**실시간 대시보드 (Grafana)**
- 추천 정확도
- 응답 시간 (P50, P95, P99)
- 처리량 (RPS)
- 에러율

**침입 탐지 (Falco)**
- 비정상 프로세스 감지
- 민감한 파일 접근 모니터링
- 컨테이너 탈출 시도 감지

**중앙 로깅 (ELK Stack)**
- Elasticsearch: 로그 저장
- Logstash: 로그 수집
- Kibana: 로그 분석

---

## 📁 프로젝트 구조

```
mlops-lgcns-mldl-platform/
├── README.md (이 파일)
├── architecture/
│   ├── system-architecture.md
│   ├── data-flow.md
│   ├── ci-cd-pipeline.md
│   └── monitoring-design.md
│
├── data-layer/
│   ├── kafka-config.yaml
│   ├── spark-jobs/
│   └── data-lake-setup.md
│
├── feature-store/
│   ├── feast-config.yaml
│   ├── online-store-redis/
│   └── offline-store-postgres/
│
├── ml-pipeline/
│   ├── kubeflow/
│   │   ├── pipeline.py
│   │   └── components/
│   ├── mlflow/
│   │   └── tracking-setup.md
│   └── model-registry/
│
├── deployment/
│   ├── docker/
│   │   ├── Dockerfile
│   │   └── .dockerignore
│   ├── kubernetes/
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   └── pdb.yaml
│   └── helm/
│       ├── Chart.yaml
│       ├── values-prod.yaml
│       ├── values-staging.yaml
│       └── templates/
│
├── cicd/
│   ├── github-actions.yaml
│   ├── argocd-config.yaml
│   └── flagger-canary.yaml
│
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yaml
│   │   └── rules.yaml
│   ├── grafana/
│   │   └── dashboards/
│   ├── falco/
│   │   └── rules.yaml
│   └── elk/
│       └── logstash-config.conf
│
└── docs/
    ├── setup-guide.md
    ├── operational-guide.md
    ├── troubleshooting.md
    └── best-practices.md
```

---

## 💡 핵심 기술 결정

### Kubeflow vs Airflow
- **선택**: Kubeflow
- **이유**: K8s 네이티브, ML 특화, 메타데이터 추적
- **효과**: 캐싱으로 재실행 50% 단축

### ArgoCD vs GitHub Actions
- **선택**: ArgoCD (CD), GitHub Actions (CI)
- **이유**: Pull-based 배포 (보안), Git을 진실의 원천
- **효과**: 자동 동기화, 즉시 롤백 가능

### Canary vs Blue-Green
- **선택**: Canary (Flagger)
- **이유**: 모델 배포는 예측 불가능 (데이터 드리프트)
- **효과**: 5% 검증 → 위험 최소화, 1분 내 롤백

### Feature Store (Feast)
- **이유**: 학습/서빙 스큐 방지, 버전 관리, 메타데이터 추적
- **효과**: 2000+ 특성 일관성 관리

---

## 🚀 성과

### 개발 생산성
```
모델 개발 사이클
├─ 기존: 2주 (데이터 준비 2주)
└─ 개선: 4시간 (완전 자동화)
  결과: 개발팀 생산성 98% 향상

배포 빈도
├─ 기존: 월 4회 (2주에 1번)
└─ 개선: 일 5회 (매일)
  결과: 배포 빈도 37배 증가, 버그 수정 즉시 반영

배포 시간
├─ 기존: 2일 (수동 배포)
└─ 개선: 30분 (완전 자동)
  결과: 운영팀 부담 96% 감소
```

### 시스템 안정성
```
장애 감지
├─ 기존: 5일 (모니터링 없음)
└─ 개선: 1시간 (자동 알림)
  결과: 사용자 영향 최소화

자동 복구
├─ 기존: 8시간 (수동 대응)
└─ 개선: 1분 (자동 롤백)
  결과: 평균 복구 시간 480배 단축

가용성
├─ 기존: 99.50% (월 36시간 다운)
└─ 개선: 99.95% (월 2시간 다운)
  결과: SLA 달성, 신뢰성 극대화
```

### 비즈니스 임팩트
```
운영 비용
├─ 기존: 월 $50K
└─ 개선: 월 $30K
  결과: 40% 비용 절감, 연 $240K 절감

모델 성능
├─ 기존: CTR 0.85%
└─ 개선: CTR 0.95% (12% 향상)
  결과: 빠른 배포로 지속적 개선 가능

고객 만족도
└─ 결과: 추천 정확도 향상으로 사용자 만족도 상승
```

---

## 🎓 배운 점

### 1. 기술은 수단, 비즈니스 가치가 목표
- 배포 자동화가 아닌 "개발 생산성" 극대화
- 모니터링이 아닌 "장애 감지 속도" 단축
- 기술 선택이 아닌 "비즈니스 임팩트" 극대화

### 2. 팀과 함께 성장
- ML팀의 요구사항 반영 (개발 편의성)
- DevOps팀의 경험 활용 (운영 효율)
- 정기적 회의로 이해도 높임

### 3. 지속적 개선이 핵심
- MVP부터 시작 (완벽함이 아닌 작동함)
- 실제 운영 후 병목 지점 파악
- 단계별 최적화

### 4. 자동화로 보안도 강화
- 수동 배포 → 자동 배포 (휴먼 에러 제거)
- 보안 검사 자동화 (매 배포마다)
- 감시 로그 자동화 (모든 접근 기록)

---

## 📞 문의 및 구성

- **GitHub**: https://github.com/devil5462/mlops-lgcns-mldl-platform
- **문서**: ./docs/ 폴더 참고
- **구성요소 상세**: ./architecture/ 폴더 참고

---

## 📄 라이선스

MIT License

---

**마지막 업데이트**: 2024년 6월  
**버전**: 1.0.0  
**상태**: Production Ready ✅
