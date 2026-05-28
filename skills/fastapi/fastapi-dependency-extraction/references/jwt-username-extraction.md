# JWT 사용자명 추출 패턴 (docmesh-doc 사례)

## 문제
`_current_username` 함수가 `routes/documents.py`와 `routes/metadata.py` 두 곳에 복사되어 있었음.
`preferred_username -> username -> sub` 우선순위 로직이 한 파일에서만 수정될 경우 두 라우터 간 동작 불일치 발생 가능.

## 해결
`dependencies/security.py`에 `get_username_from_user(current_user: UserInfo) -> str` 함수로 통합.

## 변경 파일
- `docmesh_doc/dependencies/security.py` — 함수 추가
- `docmesh_doc/routes/documents.py` — 중복 함수 제거 + import 교체
- `docmesh_doc/routes/metadata.py` — 중복 함수 제거 + import 교체

## 우선순위 로직
```python
def get_username_from_user(current_user: UserInfo) -> str:
    """JWT 클레임 우선순위(preferred_username -> username -> sub)에 따라 사용자명 반환."""
    username = getattr(current_user, "preferred_username", None) or getattr(
        current_user, "username", None
    )
    return username or current_user.sub
```

## 심각도
Medium — 즉각적 버그는 아니나 향후 유지보수 시 동작 불일치 위험
