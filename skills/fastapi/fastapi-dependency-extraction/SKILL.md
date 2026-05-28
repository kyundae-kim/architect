---
name: fastapi-dependency-extraction
description: "FastAPI 프로젝트에서 라우터 간 중복 헬퍼 함수/로직을 dependencies/ 모듈로 추출하는 리팩토링 패턴"
---

# FastAPI Dependency Extraction

## 언제 사용하나 (Triggers)
- 동일한 함수가 두 개 이상의 라우터 파일에 복사되어 있을 때
- 이슈에서 "중복 코드", "동작 불일치 위험", "한 파일에서만 수정될 경우" 같은 표현이 등장할 때
- 인증/권한/사용자 정보 추출 로직이 여러 라우터에 흩어져 있을 때

## 목표
중복 로직을 `dependencies/` 하위 적절한 모듈로 이동하여 단일 진실 공급원(Single Source of Truth)을 만든다.

## 실행 절차

1. 중복 함수 위치 확인
   - 관련 라우터 파일들을 읽어 동일/유사 함수 식별
   - `search_files`로 프로젝트 전체에서 해당 함수명 검색하여 추가 중복 여부 파악

2. 이동 대상 모듈 결정
   - 인증/사용자 관련 → `dependencies/security.py`
   - DB 세션/스토리지 관련 → `dependencies/storage.py`
   - 도메인 특화 로직 → `dependencies/<domain>.py`
   - 모듈이 없으면 신규 생성

3. 공통 함수 작성
   - 기존 private 함수(`_name`)를 public 함수로 이름 변경 (예: `_current_username` → `get_username_from_user`)
   - docstring으로 우선순위/의도 명시
   - 타입 힌트 완전히 작성

4. 라우터 파일 정리
   - 중복 함수 정의 블록 제거
   - import 줄에 새 함수명 추가
   - 함수 호출부를 새 이름으로 교체 (replace_all=true)

5. 검증
   - lint 오류 없음 확인 (patch 도구가 자동 실행)
   - import 구문 문법 확인

## 코드 예시 (JWT 사용자명 추출 패턴)

```python
# docmesh_doc/dependencies/security.py
def get_username_from_user(current_user: UserInfo) -> str:
    """JWT 클레임 우선순위(preferred_username -> username -> sub)에 따라 사용자명 반환."""
    username = getattr(current_user, "preferred_username", None) or getattr(
        current_user, "username", None
    )
    return username or current_user.sub

# 라우터에서 사용
from docmesh_doc.dependencies.security import User, get_current_user, get_username_from_user

username = get_username_from_user(current_user)
```

## Pitfalls
- 라우터 파일을 offset/limit으로 부분 읽은 경우, patch 전에 전체를 다시 읽어야 unique match 보장됨
- `_current_username` 같은 private 이름은 외부 import 시 컨벤션상 어색하므로 `get_xxx` 형태로 개명
- `replace_all=true` 없이 동일 패턴이 여러 핸들러에 있으면 patch 실패 — 반드시 replace_all 사용
- 이동 후 기존 import 줄의 다른 심볼이 깨지지 않도록 확인 (User, get_current_user 등 기존 것 유지)

## 산출물 형태
1. 무엇을 왜 바꿨는지 요약
2. 변경 파일 목록
3. lint/syntax 검증 결과
4. 이후 동일 패턴이 추가될 경우 안내
