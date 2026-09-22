[[<WAF (Web application Firewall)>]] [[🐢Akamai WAAP 보호 (26-1 w13)]]

-악성 코드 앞에 대량의 무의미한 데이터 붙임, 방화벽 검사 버퍼 한계(보통 8KB~128KB) 넘기려는 수법
- 대용량 요청 시, 검사없이 통과시키는 ‘fail open’ 약점 노림
    - ex. React2Shell