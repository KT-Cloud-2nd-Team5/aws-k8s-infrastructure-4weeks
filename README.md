# AWS 기반 쿠버네티스 클러스터 + CI/CD 자동화 파이프라인 구축 
- Terraform과 Ansible을 활용한 인프라 자동화 및 GitHub Actions 기바의 안전한 CD 환경 구현 

# 프로젝트 개요 
- AWS 클라우드 환경에서 IaC(Infrastructure as Code) 도구를 활용하여 3-Tier 아키텍처(Web-WAS-DB)를 설계하고, Kubernetes(K3s) 기반의 컨테이너 오케스트레이션 환경을 구축하는 것을 목표로 합니다. 또한 단순 배포를 넘어,네트워크 보안 격리(VPC), 형상 관리 자동화(Ansible), 그리고 CI/CD 파이프라인(GitHub Actions)까지를 목표로 삼았습니다.

# 프로젝트 특징
- Security First: 모든 워크로드(K3s, DB)는 Private Subnet에 배치하여 외부 접근을 차단하고, Bastion Host와 ALB를 통해서만 접근을 허용합니다.
- Role Separation: Web(App) 노드와 Data(DB) 노드를 물리적으로 분리하여 단일 장애 지점(SPOF) 위험을 줄이고 리소스 경합을 방지합니다.

- # 핵심 기술

| **기술** | **역할** |
| --- | --- |
| **Terraform** | AWS 인프라 자동 생성 (VPC, EC2, 보안 그룹) |
| **Ansible** | K3s 자동 설치 및 구성 |
| **K3s** | 쿠버네티스 클러스터 |
| **GitHub Actions** | CI/CD 자동화 |
| **Prometheus/Grafana** | 모니터링 대시보드 |
| **AWS ALB** | 외부 트래픽 분산 |

 # 기술 스택 및 선정 이유
 | **기술 스택** | **사용 목적** | **선정 이유 (Why)** |
| --- | --- | --- |
| **AWS (EC2, VPC)** | Cloud Provider |  **VPC/Subnet/NAT Gateway** 등 실제 엔터프라이즈 네트워크 환경을 가장 유사하게 시뮬레이션하기 위함. |
| **Terraform** | IaC (Provisioning) | 콘솔 클릭(Click-Ops) 방식의 비효율과 실수 방지. 인프라를 코드로 정의하여 **생성-수정-삭제의 라이프사이클을 명확히 관리**하고 재현성을 확보하기 위함. |
| **Ansible** | Config Mgmt | Terraform이 만든 Docker, K3s 등을 설치하는 과정을 자동화.  관리가 편하고, 멱등성(Idempotency)을 보장하기 위함. |
| **Kubernetes (K3s)** | Orchestration | 표준 컨테이너 관리 도구. 학습 및 비용 효율을 위해 CNCF 인증을 받은 경량화 버전(K3s)을 채택하여, 적은 리소스로도 완벽한 K8s 기능을 구현. |
| **GitHub Actions** | CI/CD | 코드 저장소와 통합된 파이프라인. 특히 **Self-hosted Runner**를 구축하여, 외부에서 접근 불가능한 Private Subnet 내부의 K8s 클러스터에 안전하게 배포하기 위함. |
| **Prometheus & Grafana** | Monitoring | 클라우드 네이티브 환경의 모니터링 표준. **Pull 방식**의 구조가 동적으로 변하는 컨테이너 환경에 적합하며, 시각화 도구(Grafana)와의 강력한 연동성 때문. |

# 단계벌 구현 계획
# 1단계: Terraform으로 AWS 인프라 구축
- VPC, Subnet(Public/Private), IGW, NAT Gateway 등 네트워크 레이어 설계
- Bastion Host 및 K3s Master/Worker용 EC2 인스턴스 프로비저닝

보안 그룹(SG) 설정 및 SSH Agent Forwarding을 통한 접속 환경 구성

# 2단계: Ansible로 K3s 클러스터 설치
- ProxyCommand를 활용하여 Bastion 경유 Private 노드 제어 설정
- OS 기본 설정(Swap 비활성화 등) 및 K3s Master/Worker 설치 자동화
- kubectl get nodes를 통한 클러스터 상태 및 노드 조인 확인

# 3단계: GitHub Actions CI/CD 구축
- Docker Hub 레지스트리 연동 및 GitHub Secrets 보안 정보 저장
- Bastion Host 내 Self-hosted Runner 설치 및 배포 권한 설정
- 코드 Push 시 이미지 빌드부터 클러스터 반영까지의 자동화 파이프라인 완성

# 4단계: 모니터링 및 마무리
- Helm을 이용한 kube-prometheus-stack 배포 및 메트릭 수집
- AWS ALB 생성 및 K8s NodePort 서비스 연동을 통한 외부 접속 테스트
- 실습 종료 후 terraform destroy를 통한 리소스 삭제 및 비용 정리
