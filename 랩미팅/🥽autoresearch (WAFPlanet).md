26-1 w14, 26 6월 1주차 스터디
[[<WAF (Web application Firewall)>]]
보고서 : [[WAF Autoresearch (보고서)]]

AI-driven autonomous(자발적인) optimization(최적화) of OWASP ModSecurity CRS(Core RuleSet)

> configuration(구성)
> paranoia level [[Paranoia level (PL)]]
> legitimate(정당한)
> anomaly(변칙)
> evaluate(평가하다)
> criterion(표준)

config/REQUEST-900-BEF
- main CRS config. you modify.
`Override paranoia level (1-4). Higher = more rules = better detection + more FPs.
   `setvar:tx.paranoia_level=1,\`

config/REQUEST-900-AFT
- rule-level overrides you modify.
- loaded after all CRS rules. 
- use for rule removals, custom rules, 

---
