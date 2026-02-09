# courm-service (3-repo split 기준)

이 레포는 "서비스 배포에 필요한 것"만 담습니다.

- 공통 Helm Chart: `charts/courm-app`
- 서비스별 values: `services/<service-name>/{values.yaml,value-dev.yaml,value-prod.yaml}`

Argo CD ApplicationSet이 `services/*` 디렉토리를 스캔해서 서비스 Application들을 생성합니다.
