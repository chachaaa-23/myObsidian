[[<WAF (Web application Firewall)>]] [[🥽autoresearch (WAFPlanet)]]
- protect web applications from a wide range of attacks with a minimum of false alert
	- WAF가 web application 을 보호할 때 사용하는 표준 방어 규칙
	: set of generic attack detection rules for use with ModSecurity compatible WAF (such as OWASP Coraze)

- OWASP Top10을 포함한 수많은 Malicious web 공격 패턴을 막음
- Anomaly scoring (이상 징후 점수제): 공격 심각성에따라 점수 매김, 누적 총점이 기준치를 넘을 시 차단함. FP 줄이기 가능
- Paranoia Level : 사이트의 보안 민감도 설정가능


[OWASP CRS 문서](https://devguide.owasp.org/en/09-operations/04-crs/)
