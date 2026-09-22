[[🥽autoresearch (WAFPlanet)]][[<WAF (Web application Firewall)>]]
- Docker Image 사용 
	- 설치 완료된 컴퓨터 스냅샷
	- ex. ubuntu+ nginx+ modsecurity 이미 만들어놓은 상태.
- Docker Container
	- image 를 실제 실행한 상태
- `docker compose up -d`
	- 필요한 image 다운로드 -> container 생성 -> 실행 (`-d` 통한 background 실행.)

프로그램 실행을 위한 구체적인 Ubuntu, Nginx, ModSecurity, ... 들을 내 컴퓨터에 직접 설치 -> 파일들 섞임, 삭제도 어렵고 버전 충돌 생김
=> 프로그램을 작은 가상 컴퓨터 안에 넣어서 실행. ()

---
실행
`docker compose up -d`

상태확인
`docker ps`

로그 보기
`docker compose logs`
`docker compose logs -f`  실시간 로그 추적

종료
`docker compose down`

재시작
`docker compose restart`