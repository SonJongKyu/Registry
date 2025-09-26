**2024.11 (1人 개인 프로젝트)**

## **📌 Summary**

본 프로젝트는 **Docker Registry, Harbor Registry, Nexus Registry**를 직접 구축하고 비교·분석하여,  
**프라이빗 컨테이너 레지스트리 환경**을 설계·실습한 프로젝트입니다.  

단순히 이미지를 저장하는 수준을 넘어,  
- Self-signed 인증서 기반 TLS 적용  
- 인증 및 접근 제어 (htpasswd, Harbor/Nexus 사용자 계정 관리)  
- Nginx Reverse Proxy / SSL 적용  
- 취약점 스캐너(Trivy) 연동  

등을 통해 **보안·운영 자동화**까지 고려한 실전형 구축 경험을 확보했습니다.  

> **주요 기능**
> - Docker Registry: 기본 프라이빗 레지스트리 + Nginx Reverse Proxy + Basic Auth
> - Harbor Registry: Trivy 기반 취약점 스캐닝, 프로젝트 단위 이미지 관리
> - Nexus Registry: Maven/NPM/Docker 등 멀티 레지스트리 통합 관리
> - OpenSSL 기반 TLS, Docker insecure-registry 및 hosts 설정, Push/Pull 테스트
> - 도커 이미지 빌드 및 Registry Push/Pull 시나리오 검증

---

## **🤔 Background**

최근 컨테이너 기반 서비스 운영에서 **이미지 관리와 보안**은 필수적인 요소가 되고 있습니다.  
DockerHub 같은 퍼블릭 레지스트리 의존도를 줄이고,  
**조직 내부에서 안전하게 이미지를 관리할 수 있는 Private Registry 구축**의 필요성이 커지고 있습니다.  

이 프로젝트는 Docker Registry, Harbor, Nexus 세 가지 솔루션을 직접 설치·구성하고,  
각각의 장단점을 실습을 통해 체감하여,  
**엔터프라이즈 환경에서 적합한 레지스트리 아키텍처 설계 역량**을 확보하는 것을 목표로 진행했습니다.  

---

## **🔍 Meaning**

- **기술 학습 측면**
  - Docker Registry → 가장 기본적인 프라이빗 레지스트리 동작 이해
  - Harbor → 보안(Trivy), RBAC, Web UI를 갖춘 엔터프라이즈 레벨 경험
  - Nexus → Docker 포함 멀티 레지스트리 관리 및 Proxy Registry 학습

- **운영/보안 측면**
  - Self-signed 인증서 발급 및 Docker TLS 인증 절차 실습
  - htpasswd, 사용자 계정, insecure-registry 설정 등 실무 환경과 유사한 경험
  - Harbor Trivy, Nexus Proxy 등을 통한 보안·운영 확장성 확인

단순 설치에 그치지 않고 **보안·운영 환경까지 고려한 레지스트리 구축**이라는 점에서 의미가 큽니다.  

---

## **🔨 Technology Stack(s)**

- **Languages & Tools**
  - Docker, Docker Compose
  - OpenSSL
  - Trivy (취약점 스캐너)

- **Registry Solutions**
  - Docker Registry
  - Harbor Registry
  - Nexus Repository Manager 3

- **Infra & DevOps**
  - Nginx (Reverse Proxy, TLS Offloading)
  - Linux (Ubuntu 20.04/22.04)

- **Deployment Layout**
  - Docker Registry: `docker-compose.yml`, `nginx.conf`, `htpasswd`
  - Harbor Registry: `harbor.yml`, `install.sh`, `Trivy`
  - Nexus Registry: `docker-compose.yml`, `systemd service`, Nginx SSL proxy

---

## **⚙️ Install**

```
# ========================================
# ▶ Option 1: Docker Registry
# ========================================

# 인증서 생성
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 -nodes \
  -keyout server.key -out server.crt \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=acs7th/OU=IT/CN=192.168.3.14" \
  -addext "subjectAltName = IP:192.168.3.14"

# Docker Compose 실행
docker compose up -d

# 로그인/푸시/풀 테스트
docker login 192.168.3.14
docker tag nginx:1.27.2 192.168.3.14/nginx:1.27.2
docker push 192.168.3.14/nginx:1.27.2
docker pull 192.168.3.14/nginx:1.27.2
```
# ========================================
# ▶ Option 2: Harbor Registry
# ========================================
```
# Harbor 설치 파일 다운로드 및 압축 해제
wget https://github.com/goharbor/harbor/releases/download/v2.11.1/harbor-offline-installer-v2.11.1.tgz
tar -zxvf harbor-offline-installer-v2.11.1.tgz && cd harbor

# harbor.yml 설정 (hostname, https 인증서 경로 등)
cp harbor.yml.tmpl harbor.yml
vi harbor.yml
# hostname: 192.168.2.21
# https:
#   port: 443
#   certificate: /home/kevin/cert/ca.crt
#   private_key: /home/kevin/cert/ca.key

# 설치 및 실행
sudo ./install.sh --with-trivy
sudo docker-compose up -d

# Harbor 로그인 및 이미지 푸시/풀
docker login -u admin 192.168.2.21:443
docker tag phpserver:1.0 192.168.2.21:443/project/phpserver:1.0
docker push 192.168.2.21:443/project/phpserver:1.0
docker pull 192.168.2.21:443/project/phpserver:1.0
```
# ========================================
# ▶ Option 3: Nexus Registry
# ========================================
```
# Nexus 설치 (Docker Compose 기반)
cat > docker-compose.yml <<EOF
version: '3'
services:
  nexus:
    image: sonatype/nexus3
    container_name: nexus
    volumes:
      - ./nexus-data:/nexus-data
    ports:
      - "8081:8081"
    restart: always
EOF

docker compose up -d

# 초기 비밀번호 확인
docker exec -it nexus cat /nexus-data/admin.password

# Docker Registry 연동 테스트
docker login 192.168.1.248:5000
docker tag mysql:8.0.32 192.168.1.248:5000/docker-hosted/mysql:8.0.32
docker push 192.168.1.248:5000/docker-hosted/mysql:8.0.32
docker pull 192.168.1.248:5000/docker-hosted/mysql:8.0.32
```
