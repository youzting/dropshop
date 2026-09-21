# 개인 EC2 실습 배포

`docker-compose.prod.yml`은 앱, Kafka 1개, MySQL, Redis를 같은 서버에서 실행합니다.
메모리 제한 합계는 1696MiB이며 2GiB 서버에서 부하에 따라 부족할 수 있습니다.
실제 기동/부하 검증 전에는 안정적인 운영을 보장하지 않습니다.

## 환경 변수

`.env.prod.example`을 기준으로 서로 다른 무작위 비밀번호와 JWT 키를 생성하고,
내용 전체를 GitHub Actions Secret `PROD_ENV_FILE`에 저장합니다.
예시의 REPLACE 값은 반드시 교체하세요. 실제 값은 Git에 커밋하지 마세요.
`APP_PORT=80`은 HTTP 실습용입니다. 로그인/결제 등 실제 사용 전 HTTPS가 필요합니다.

## DB 초기화

Compose는 `prod,personal` 프로필을 사용합니다. 기존 Flyway 파일에는 완전한 초기
스키마가 없으므로 개인 실습 프로필에서 Flyway를 끄고 Hibernate `update`로 테이블을
생성합니다. ShedLock 테이블은 별도 멱등 SQL로 생성합니다.
실제 고객 데이터 운영 전에는 초기 마이그레이션을 정비하고 `validate`로 전환해야 합니다.
기존 팀 DB에 이 프로필을 적용하지 마세요. 테스트 데이터 초기화는 local/default에서만 실행됩니다.

MySQL/Redis/Kafka 데이터는 named volume에 보존됩니다. `docker compose down -v`는
데이터를 삭제하므로 사용하지 마세요. MySQL 볼륨 초기화 이후 비밀번호를 바꾸려면
DB 내부 계정 비밀번호와 Secret을 함께 변경해야 합니다.

## 배포 순서

1. 서버/보안 그룹/AWS/GHCR Secrets와 `PROD_ENV_FILE` 등록.
2. `EC2_DEPLOY_DIR=/home/ubuntu/dropshop`, `CD_ENABLED=false` 유지.
3. 변경을 검토하고 main에 반영한 뒤 `CD_ENABLED=true`로 설정.
4. Actions에서 CD-EC2-Docker를 수동 실행하고 컨테이너 상태와 health 확인.

PortOne, S3 이미지 업로드 및 AI 추천용 외부 서비스 인증은 별도 설정입니다.
현재 기본 더미 값으로 실제 결제/이미지 업로드를 사용할 수 없습니다.
