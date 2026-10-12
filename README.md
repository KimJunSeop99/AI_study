# AI_study

AI 공부용 Python 프로젝트입니다. 패키지와 가상환경은 [uv](https://docs.astral.sh/uv/)로 관리합니다.

- Python 3.12 이상
- uv 0.13.0

## 프로젝트 구조

```
AI_study/
├── src/temp/__init__.py   # 패키지 코드 (진입점: main)
├── pyproject.toml         # 프로젝트 설정 및 의존성
├── uv.lock                # 고정된 의존성 버전
├── .python-version        # 사용할 Python 버전
├── uv_command.md          # uv 명령어 정리
└── git_command.md         # git 명령어 정리
```

## 시작하기

```powershell
# 의존성 설치 및 .venv 생성/동기화
uv sync

# 프로그램 실행
uv run temp
```

## 가상환경

가상환경은 `.venv` 폴더에 있고, 활성화하면 프롬프트 앞에 `(temp)`가 표시됩니다.

```powershell
# 활성화 (PowerShell)
.venv\Scripts\Activate.ps1

# 비활성화
deactivate
```

PowerShell에서 스크립트 실행이 막혀 있다면 아래 명령을 한 번 실행하세요.

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

`uv run`을 쓰면 가상환경을 켜지 않아도 `.venv`가 자동으로 사용됩니다.

## 패키지 관리

```powershell
uv add <패키지>      # 패키지 추가
uv remove <패키지>   # 패키지 제거
uv tree              # 의존성 트리 보기
```

## 참고 문서

- [uv_command.md](uv_command.md): uv 명령어 정리
- [git_command.md](git_command.md): git 명령어 정리
