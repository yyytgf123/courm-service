# courm-service

이 레포는 **서비스 배포 단위(Helm Chart + per-service values)** 를 담습니다.

- 공통 차트: `charts/courm-app` (Rollout/Service/HTTPRoute 템플릿)
- 서비스별 값: `services/<svc>/values.yaml` + `services/<svc>/value-<env>.yaml`

Argo CD ApplicationSet은 `services/*` 디렉토리를 스캔해서 서비스 개수만큼 Application을 자동 생성합니다.
