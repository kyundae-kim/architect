# Alembic 마이그레이션 도입 문서 반영 패턴

## 이슈 유형

`Base.metadata.create_all()` 프로덕션 사용 + 로컬 Base 클래스 선언 문제.
심각도 High — 스키마 변경 추적 불가, 롤백 불가.

## 문서 반영 위치 및 내용

### pyproject.toml
```toml
dependencies = [
    "alembic>=1.13.0",
    ...
]
```

### PRD (prd.md)
- 비기능 요구사항에 마이그레이션 방침 추가
- 별도 섹션 "DB 스키마 관리 방침" 신설:
  - 개발: create_all() 허용
  - 프로덕션: Alembic만 허용
  - 공통 Base 위치: `docmesh_doc/models/base.py`
  - 서비스 내부 로컬 Base 금지

### API 문서 (api.md)
- "DB 스키마 마이그레이션" 섹션 추가:
  - 프로젝트 구조 트리
  - alembic 명령어 (init / revision --autogenerate / upgrade head / downgrade -1)
  - env.py 핵심 설정 예시
  - 환경변수 목록

### 테스트 문서 (test.md)
- "Alembic 마이그레이션 테스트" 섹션 추가:
  - 마이그레이션 적용/롤백 검증 명령
  - 주의사항: create_all() 호출 여부 확인, autogenerate 수동 검토

### README.md
- 핵심 동작 목록에 "DB 스키마 관리: Alembic 마이그레이션" 추가
- "DB 마이그레이션" 섹션 신설 (한 줄 설명 + upgrade head 명령)

## alembic/env.py 핵심 패턴

```python
from docmesh_doc.models.base import Base
from docmesh_doc.models import metadata  # noqa: F401

target_metadata = Base.metadata
```

## 다음 단계 (코드 구현, 문서에 명시)

1. `docmesh_doc/models/base.py` 생성 (공통 Base)
2. 서비스 파일에서 로컬 Base 제거 → 공통 Base import
3. `MetadataService.__init__`에서 create_all() 제거 또는 환경 분기
4. `alembic init` + env.py 설정 + 첫 마이그레이션 생성
