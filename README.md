# 스포렉스 합주실 PA 안내 사이트

스포렉스 합주실의 PA·모니터 세팅 문서를 MkDocs Material로 발행하는 프로젝트.

## 문서 구조

- `docs/index.md` — 랜딩(갈림길)
- `docs/guide.md` — 일반 이용자용 사용 가이드 (짧고 쉽게)
- `docs/spec.md` — 관리자/작성자용 상세 스펙
- `source/pa-setup-original.md` — 원본 PA 세팅 자료 (사이트 빌드 제외)
- `docs/superpowers/` — 설계·구현 문서 (사이트 빌드 제외)

## 로컬 미리보기

```bash
pip install -r requirements.txt
mkdocs serve
# http://127.0.0.1:8000 접속
```

## 배포

`main` 브랜치에 푸시하면 GitHub Actions가 자동으로 빌드·배포한다.

### 레포 최초 1회 설정

1. GitHub에 public 레포 `bandroom-guide` 생성
2. 코드 푸시
3. 레포 **Settings → Pages → Build and deployment → Source = "GitHub Actions"** 선택

배포 URL: `https://SingeonKim.github.io/bandroom-guide/`
