# NATS 도입 문서 반영 체크리스트 (session reference)

이 참조 파일은 fastapi-core 같은 SDK/플랫폼 레이어에서 NATS 도입 이슈를 문서 중심으로 반영할 때 누락을 줄이기 위한 압축 체크리스트다.

## 필수 반영 포인트
- 의존성
  - pyproject.toml: `nats-py` 추가

- 환경설정
  - `NATS__SERVERS` (콤마 구분 서버 목록)
  - `NATS__NAME` (연결 이름)
  - `NATS__CONNECT_TIMEOUT`
  - `NATS__MAX_RECONNECT_ATTEMPTS`
  - `NATS__RECONNECT_TIME_WAIT_MS`
  - `NATS__QUEUE_GROUP`

- README
  - 기능 개요에 "NATS 메시징" 추가
  - state 싱글톤 목록에 `app.state.nats_client` 추가
  - 테스트/문서 섹션에 NATS 관련 안내 + 새 문서 링크 추가

- 상세 문서
  - docs/config.md: 환경변수 표 + .env 예시
  - docs/api.md: messaging 섹션(구현 전이면 `추가 예정` 명시)
  - docs/prd.md: 요구사항/패키지구조/기술스택 반영
  - docs/test.md: 단위/통합 테스트 전략 반영
  - docs/messaging.md: 설치/설정/pub-sub 샘플/도메인 적용/운영 고려사항

## Subject 네이밍 권장
- `<domain>.<entity>.created`
- `<domain>.<entity>.updated`
- `<domain>.<entity>.deleted`

예시:
- `orders.created`
- `billing.invoice.updated`
- `documents.file.deleted`

## 구현 상태 표기 규칙
- 실제 구현 전 API는 반드시 `*(추가 예정)*`로 표기.
- 샘플 코드에서 아직 없는 모듈 import는 주석 처리로 "미구현/예정" 의도를 명확히 할 것.

## 검증 루틴
1. git diff로 문서 간 정합성 확인
2. 테스트 실행(`uv run pytest -q`)으로 회귀 확인
3. 최종 보고에 변경 파일 + 테스트 결과 수치 포함
