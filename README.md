# Model
## azure-aks-microservices-kafka-lab-20250617
https://labs.msaez.io/#/courses/cna-full/2c7ffd60-3a9c-11f0-833f-b38345d437ae/deploy-my-app-2024

제목: Azure AKS 기반 마이크로서비스 (FE/BE) 배포 및 Kafka 연동 실습

요약:  
- Azure AKS 클러스터에 Spring Boot 마이크로서비스(BE) 및 웹 프론트엔드(FE)를 배포했습니다.
- Docker 이미지 빌드부터 Helm을 통한 Kafka 설치, 그리고 Kubernetes YAML 파일을 이용한 서비스 배포 및 상호 연동을 실습했습니다.
- 이를 통해 클라우드 네이티브 환경에서의 애플리케이션 배포 및 운영 전반을 경험했습니다.

## 사전 준비
Azure 계정 및 구독, Gitpod 워크스페이스, Spring Boot 애플리케이션 코드

![스크린샷 2025-06-17 135726](https://github.com/user-attachments/assets/919602b1-5f5a-4f96-9a6d-4b973b358b62)
![스크린샷 2025-06-17 142125](https://github.com/user-attachments/assets/a4a2ea0e-2112-4021-acd4-dd0bc709c30e)
![스크린샷 2025-06-17 142627](https://github.com/user-attachments/assets/835aa5a7-a447-44c3-a67d-e6489f01acf6)
![스크린샷 2025-06-17 142636](https://github.com/user-attachments/assets/ab0888ec-0a25-4af7-90cf-36f3c61bbf49)
![스크린샷 2025-06-17 145012](https://github.com/user-attachments/assets/60964665-c4e8-4eb2-a772-8b6f3f976491)
![스크린샷 2025-06-17 145452](https://github.com/user-attachments/assets/1781b5ac-cac2-477b-ab3f-ffa9e6d28a3b)
![스크린샷 2025-06-17 154345](https://github.com/user-attachments/assets/85ff472a-3f5a-4b2a-84ad-4e8fd5a413b9)
![스크린샷 2025-06-17 154727](https://github.com/user-attachments/assets/2918aa7d-6929-4ff7-8590-2b154fab5ce4)
![스크린샷 2025-06-17 161500](https://github.com/user-attachments/assets/a0b84166-0284-48bd-80b0-33a19466e60c)

![스크린샷 2025-06-17 163216](https://github.com/user-attachments/assets/7373de26-fe52-4ce0-a9a3-7ab9dc327b88)
![스크린샷 2025-06-17 163314](https://github.com/user-attachments/assets/4abaca2d-a5e9-48be-a852-0ac7998a0308)
![스크린샷 2025-06-17 163905](https://github.com/user-attachments/assets/3817206a-44dd-4f4f-afa8-7ce7f7487cfc)
![스크린샷 2025-06-17 165255](https://github.com/user-attachments/assets/96fd98b4-99a3-4ed6-88d7-749abcc33988)
![스크린샷 2025-06-17 165946](https://github.com/user-attachments/assets/c8b23c52-5ce0-4691-ad9d-6579de9b370b)
![스크린샷 2025-06-17 165850](https://github.com/user-attachments/assets/57f2d6fc-0f04-4926-a45c-c2e60e55f782)
![스크린샷 2025-06-17 165856](https://github.com/user-attachments/assets/8b0ffaa7-fc98-4965-8270-52fa071db04e)

---

이 문서는 Azure Kubernetes Service (AKS) 클러스터 환경에 Spring Boot 기반의 마이크로서비스 백엔드와 웹 프론트엔드를 배포하고 Kafka를 연동하는 과정을 상세히 기록합니다.
Gitpod을 개발 환경으로 활용하여 Docker 이미지 빌드, Helm을 통한 미들웨어 설치, 그리고 Kubernetes YAML 파일을 이용한 서비스 배포 및 관리를 실습합니다.

실습 단계별 터미널 명령어
1. 애플리케이션 도커라이징 및 Docker Hub 푸시
각 Spring Boot 마이크로서비스(order, delivery, product, gateway)의 JAR 파일을 빌드하고 Docker 이미지로 생성하여 Docker Hub에 푸시합니다. 프론트엔드 서비스도 유사하게 도커라이징 합니다.

# (필요시) Java SDK 설치/업그레이드
sdk install java

# Order 서비스 도커라이징
cd order
mvn package -B -Dmaven.test.skip=true
# (선택 사항: 로컬 실행 확인) java -jar target/order-0.0.1-SNAPSHOT.jar
docker build -t sukuai/order:250617 .
docker images
docker push sukuai/order:250617
cd .. # 상위 디렉토리로 이동

# Delivery 서비스 도커라이징
cd delivery
mvn package -B -Dmaven.test.skip=true
# (선택 사항: 로컬 실행 확인) java -jar target/delivery-0.0.1-SNAPSHOT.jar
docker build -t sukuai/delivery:250617 .
docker images
docker push sukuai/delivery:250617
cd .. # 상위 디렉토리로 이동

# Product 서비스 도커라이징
cd product
mvn package -B -Dmaven.test.skip=true
# (선택 사항: 로컬 실행 확인) java -jar target/product-0.0.1-SNAPSHOT.jar
docker build -t sukuai/product:250617 .
docker images
docker push sukuai/product:250617
cd .. # 상위 디렉토리로 이동

# Gateway 서비스 도커라이징
cd gateway
mvn package -B -Dmaven.test.skip=true
# (선택 사항: 로컬 실행 확인) java -jar target/boot-camp-gateway-0.0.1-SNAPSHOT.jar
docker build -t sukuai/gateway:250617 .
docker images
docker push sukuai/gateway:250617
cd .. # 상위 디렉토리로 이동

2. Azure CLI 로그인 및 AKS 클러스터 연동
Azure 계정에 로그인하고 AKS 클러스터 자격 증명을 가져와 Kubectl과 연동합니다.

# Azure CLI 설치 확인 (Gitpod에 보통 사전 설치됨)
az --version

# Azure 계정 로그인 (브라우저를 통한 디바이스 코드 인증)
az login --use-device-code

# AKS 클러스터 자격 증명 가져오기 및 Kubectl 컨텍스트 설정
az aks get-credentials --resource-group a071098-rsrcgrp --name a071098-aks

# 클러스터 리소스 및 노드 상태 확인
kubectl get all
kubectl get node

# (선택 사항) 이전 배포된 리소스가 있다면 삭제하여 초기화
# kubectl delete deploy --all
# kubectl delete svc --all

3. Helm 설치 및 Kafka 배포
Kubernetes 패키지 관리자인 Helm을 설치하고, 이를 사용하여 Kafka 서버를 클러스터에 배포합니다.

# Helm 3.x 설치 (Linux 환경 기준)
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

# Bitnami Helm 저장소 추가 및 업데이트
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Kafka 설치 (버전 23.0.5 지정)
helm install my-kafka bitnami/kafka --version 23.0.5

# Kafka 설치 확인
kubectl get all

4. 마이크로서비스 배포 (YAML 파일 사용)
각 마이크로서비스(order, delivery, product, gateway)의 kubernetes/deployment.yaml 파일 내 image: 경로를 본인이 푸시한 Docker Hub 이미지명(sukuai/[서비스명]:250617)으로 수정한 후, deployment.yaml과 service.yaml을 적용합니다.

# Order 서비스 배포
cd order
# (필수) kubernetes/deployment.yaml 파일 내 image: 'sukuai/order:250617'로 수정
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl get all
cd ..

# Delivery 서비스 배포
cd delivery
# (필수) kubernetes/deployment.yaml 파일 내 image: 'sukuai/delivery:250617'로 수정
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl get all
cd ..

# Product 서비스 배포
cd product
# (필수) kubernetes/deployment.yaml 파일 내 image: 'sukuai/product:250617'로 수정
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl get all
cd ..

# Gateway 서비스 배포
cd gateway
# (필수) kubernetes/deployment.yaml 파일 내 image: 'sukuai/gateway:250617'로 수정
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl get all
cd ..

5. 서비스 동작 확인
배포된 마이크로서비스들의 상태를 확인하고, Gateway의 외부 IP를 통해 API 호출을 테스트합니다.

# 모든 서비스 및 파드 상태 확인
kubectl get svc
kubectl get po

# Gateway 서비스의 External IP 확인 (예시 IP: 20.249.163.136)
# service/gateway        LoadBalancer   10.0.28.176    20.249.163.136   8080:31664/TCP

# 재고 생성 API 호출 (httpie 사용)
http 20.249.163.136:8080/inventories id=1 stock=100

# 주문 생성 API 호출
http 20.249.163.136:8080/orders productId=1 productName="TV" qty=3

# 재고 및 주문 목록 확인 (데이터 변화 확인)
http 20.249.163.136:8080/inventories
http 20.249.163.136:8080/orders

6. Kafka 이벤트 확인 (Kafka Client 사용)
임시 Kafka 클라이언트 파드를 생성하여 Kafka 토픽의 메시지를 소비합니다.

# 임시 Kafka 클라이언트 파드 생성
kubectl run my-kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.5.0-debian-11-r21 --namespace default --command -- sleep infinity
kubectl get all # 클라이언트 파드 생성 확인

# Kafka 클라이언트 파드 쉘 접속
kubectl exec --tty -i my-kafka-client --namespace default -- bash

# Kafka 컨슈머 실행 (my-kafka.default.svc.cluster.local:9092는 Kafka 서비스 DNS)
kafka-console-consumer.sh --bootstrap-server my-kafka.default.svc.cluster.local:9092 --topic modelforops --from-beginning
# (이후 다른 터미널에서 주문 생성 등 이벤트를 발생시키면 메시지 확인 가능)
# (컨슈머 종료: Ctrl+C 후 exit)

7. 고급 테스트 및 트러블슈팅
Kubernetes의 자체 복구 기능, 스케일링, 로그 확인 및 포트 포워딩 등을 실습합니다.

# 파드 자체 복구 테스트 (터미널 1: watch kubectl get po, 터미널 2: kubectl delete po --all)
kubectl delete po --all
kubectl get po # 파드가 다시 생성되었는지 확인

# Deployment 스케일링 (Order 서비스 파드 3개로 확장)
kubectl scale deploy order --replicas=3
kubectl get po # order 파드가 3개로 늘어났는지 확인

# 특정 파드의 상세 정보 확인 (Pod 이름은 kubectl get po로 확인)
kubectl describe po <pod-name>

# 특정 파드의 실시간 로그 확인
kubectl logs -f <pod-name>

# ReplicaSet 상태 확인 (예: 비활성화된 ReplicaSet)
kubectl describe replicaset order-fdc45f5f6
# (선택 사항) 비활성화된 ReplicaSet 삭제
# kubectl delete replicaset order-fdc45f5f6

# External IP 접속 불가 시 포트 포워딩 (임시 로컬 접속)
# 새 터미널 1:
# kubectl port-forward svc/order 8080:8080
# 새 터미널 2:
# curl localhost:8080

8. 프론트엔드 배포
프론트엔드 프로젝트를 빌드하고 Docker 이미지로 만들어 Kubernetes에 배포합니다.

# Frontend 프로젝트 빌드
cd frontend/
npm install   # 의존성 설치
npm run build # 프로덕션 빌드 (dist/ 폴더 생성)

# Frontend Docker 이미지 빌드
docker build -t sukuai/frontend:250617 .
docker push sukuai/frontend:250617
cd .. # 상위 디렉토리로 이동

# Frontend Deployment 및 Service YAML 파일 수정 및 적용
# frontend/kubernetes/deployment.yaml 파일 내 image: 'sukuai/frontend:250617'로 수정 (Nginx와 같은 정적 웹서버 기반)
kubectl apply -f frontend/kubernetes/deployment.yml
kubectl apply -f frontend/kubernetes/service.yaml

# 프론트엔드 배포 확인
kubectl get all

# 웹 UI 테스트 (Gateway IP 주소로 접속, 예: http://20.249.163.136:8080/#/)
