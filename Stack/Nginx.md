# Nginx

> 웹사이트로 들어오는 요청을 가장 먼저 받아서 적절한 곳으로 보내주는 프로그램
> (**웹 서버 / 리버스 프록시**)

사용자가 `https://test.com` 같은 주소로 접속하면 백엔드 서버에 바로 닿는 게 아니라 그 앞에 있는 **Nginx를 먼저 거친다.** Nginx가 요청을 분석해서 어디로 보낼지 정하고, 결과를 다시 사용자에게 전달한다.

```text
[사용자] → [Nginx] → [백엔드 서버]
[사용자] ← [Nginx] ← [응답]
```

---

## 📌 Nginx가 하는 일

### 1. 정적 파일 전달
HTML, CSS, JS, 이미지처럼 **이미 만들어져 있는 파일**은 백엔드까지 갈 필요 없이 Nginx가 직접 응답한다.
→ 빠르고 효율적

### 2. 리버스 프록시 (Reverse Proxy)
동적인 요청(예: API 호출)은 뒤에 있는 백엔드 서버로 전달한다.
사용자 입장에서는 Nginx 하나랑만 대화하는 것처럼 보이지만 실제론 그 뒤에 여러 서버가 숨어 있을 수 있다.

> **리버스 프록시란?**
> 클라이언트와 서버 사이에서 요청을 대신 받아 백엔드 서버로 전달하고, 응답을 다시 클라이언트에게 돌려주는 서버.
> 서버 앞단에 위치해 트래픽을 중계하면서 **보안, 부하 분산, 성능 최적화**를 담당하는 중개 서버다.

```text
[클라이언트 요청] → [Nginx] → [백엔드 서버]
```

### 3. 로드 밸런싱 (Load Balancing)
백엔드 서버가 `여러 대`일 때 트래픽(데이터 흐름)을 골고루 분배해준다.
→ 한 서버에 요청이 몰리지 않음

### 4. SSL / HTTPS 처리
HTTPS 통신에 필요한 암호화·복호화를 Nginx가 대신 처리한다.
→ 백엔드 서버는 보안에 신경 쓰지 않고 본업에만 집중 가능

### 5. API Gateway 역할
마이크로서비스 구조에서는 어떤 요청을 어느 서비스로 보낼지 결정하는 **입구**가 필요하다.
→ Nginx가 이 역할을 수행한다.

```text
                    ┌─→ 유저 서비스
[사용자] → [Nginx] ─┼─→ 결제 서비스
                    ├─→ 알림 서비스
                    └─→ 상품 서비스
```

---

## ⚙️ 기본 설정법

### 1. 설치

```bash
# macOS
brew install nginx

# Ubuntu / Debian
sudo apt update
sudo apt install nginx
```

### 2. 주요 명령어

```bash
sudo nginx              # 시작
sudo nginx -s stop      # 즉시 중지
sudo nginx -s quit      # 안전하게 종료
sudo nginx -s reload    # 설정 변경 후 재적용
sudo nginx -t           # 설정 파일 문법 검사
```

> 💡 설정을 바꾼 뒤에는 항상 `nginx -t`로 문법을 검사하고 `nginx -s reload`로 적용한다.

### 3. 설정 파일 구조

설정 파일은 보통 `/etc/nginx/nginx.conf`에 있으며, `server` 블록 단위로 사이트를 정의한다.

```nginx
server {
    listen 80;                  # 80번 포트(HTTP)로 들어오는 요청을 받음
    server_name test.com;       # 이 도메인으로 들어온 요청을 처리

    # 정적 파일 서빙
    location / {
        root /var/www/html;     # 파일이 위치한 디렉터리
        index index.html;       # 기본으로 보여줄 파일
    }
}
```

### 4. 리버스 프록시 설정

들어온 요청을 백엔드 서버로 넘겨주는 가장 기본적인 패턴이다.

```nginx
server {
    listen 80;
    server_name test.com;

    location /api/ {
        proxy_pass http://localhost:3000;   # 백엔드 서버 주소로 전달
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 5. 로드 밸런싱 설정

`upstream` 블록에 서버들을 묶어두고 그 그룹으로 요청을 분배한다.


```nginx
upstream backend {
    # upstream 블록을 사용해 여러 서버에 트래픽을 분산
    server localhost:3000;
    server localhost:3001;
    server localhost:3002;
}

server {
    listen 80;
    server_name test.com;

    location / {
        proxy_pass http://backend;   # upstream 그룹으로 분배
    }
}
```
