---
date: '2025-04-26T18:27:38+09:00'
draft: false
title: "AWS EC2 + Nginx + Spring + SSL 배포 가이드"
---


Spring 프로젝트를 AWS EC2에 배포하고  
**Nginx 리버스 프록시 + SSL(HTTPS)** 까지 적용한 기록.

---

## ☁️ 사용 환경

| 항목 | 내용 |
|:---|:---|
| 도메인 | `www.my-example.com` |
| EC2 OS | Amazon Linux 2023 |
| EC2 IP | `123.1.2.3` |
| EC2 DNS | `ec2-123-1-2-3.region.compute.amazonaws.com` |
| 키 파일 | `prj-mojo.pem` |
| 웹서버 | Nginx (80 → 8080 프록시) |
| 애플리케이션 | Spring Boot (port 8080) |
| SSL 인증서 | Let's Encrypt + Certbot |


## 1️⃣ AWS EC2 생성

- Amazon Linux 2023
- 프리티어 / default 설정으로 생성
- 퍼블릭 IP 확인 (예: `123.1.2.3`)



## 2️⃣ EC2 접속

```bash
ssh -i prj-mojo.pem ec2-user@ec2-123-1-2-3.region.compute.amazonaws.com
```

## 3️⃣ Java 17 설치
```bash
sudo yum install java-17-amazon-corretto -y
```

## 4️⃣ Nginx 설치
```bash
sudo yum install nginx -y
```

## 5️⃣ EC2 보안 그룹 설정
인바운드 규칙 추가:
- TCP 80 (HTTP)
- TCP 443 (HTTPS)

## 6️⃣ Spring 프로젝트 준비
- Spring Boot로 프로젝트 생성: start.spring.io
- 포트: 8080
- 간단한 엔드포인트:
```java
@GetMapping("/")
public String hello(@RequestParam String name) {
    return "hello " + name;
}
```

## 7️⃣ Nginx 설정
```bash
sudo vi /etc/nginx/nginx.conf
```

http { ... } 블럭 안에 다음 server 블럭 추가:

```nginx
server {
    listen 80;
    server_name www.my-example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

저장 후 적용:
```bash
sudo systemctl restart nginx
```

## 8️⃣ SSL 인증서 (Let’s Encrypt + Certbot)
1. Certbot 설치:
```bash
sudo dnf install -y python3 python3-pip nginx
sudo pip3 install certbot certbot-nginx
```

2. 인증서 발급:
```bash
sudo certbot --nginx -d www.my-example.com
```

3. 성공 시 HTTPS 자동 적용됨


## 9️⃣ 도메인 연결

도메인 관리 사이트 (ex. Route53, Cafe24 등) 에서
```
www.my-example.com → 123.1.2.3
```
A 레코드 추가해주면 끝.


## 🎉 완료!
이제 아래 주소로 접속 시 Spring 앱이 HTTPS로 응답한다:

🔗 https://www.my-example.com/?name=doyouknow

📦 응답: hello doyouknow