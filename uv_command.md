# uv 명령어 레퍼런스

> 기준 버전: **uv 0.13.0** (`uv --version`)
> 각 명령의 전체 옵션은 `uv <command> --help` 또는 `uv help <command>`로 확인할 수 있습니다.

---

## 목차

1. [프로젝트 관리](#1-프로젝트-관리)
2. [스크립트 실행](#2-스크립트-실행)
3. [코드 품질 (format / check / audit)](#3-코드-품질-format--check--audit)
4. [도구(Tool) 관리](#4-도구tool-관리)
5. [Python 버전 관리](#5-python-버전-관리)
6. [pip 호환 인터페이스](#6-pip-호환-인터페이스)
7. [가상환경](#7-가상환경)
8. [빌드 & 배포](#8-빌드--배포)
9. [워크스페이스](#9-워크스페이스)
10. [인증](#10-인증)
11. [캐시](#11-캐시)
12. [uv 자체 관리](#12-uv-자체-관리)
13. [공통 옵션](#13-공통-옵션)
14. [주요 환경 변수](#14-주요-환경-변수)
15. [자주 쓰는 워크플로](#15-자주-쓰는-워크플로)

---

## 1. 프로젝트 관리

| 명령 | 설명 |
|---|---|
| `uv init` | 새 프로젝트 생성 (`pyproject.toml`, `.python-version`, `README.md` 등) |
| `uv add` | 의존성 추가 (pyproject.toml + uv.lock 갱신 + 환경 동기화) |
| `uv remove` | 의존성 제거 |
| `uv sync` | lock 파일 기준으로 프로젝트 환경(`.venv`) 동기화 |
| `uv lock` | `uv.lock` 파일 생성/갱신 |
| `uv export` | lock 파일을 다른 형식(requirements.txt, pylock.toml 등)으로 내보내기 |
| `uv tree` | 프로젝트 의존성 트리 표시 |
| `uv version` | 프로젝트 버전 조회/변경 |

### uv init

```bash
uv init                      # 현재 디렉터리에 프로젝트 생성
uv init myproj               # myproj 디렉터리에 생성
uv init --app                # 애플리케이션 프로젝트 (기본값)
uv init --lib                # 라이브러리 프로젝트 (src 레이아웃, 빌드 시스템 포함)
uv init --package            # 패키지형 애플리케이션 (src 레이아웃)
uv init --script hello.py    # PEP 723 인라인 메타데이터 스크립트 생성
uv init --python 3.12        # Python 버전 지정
uv init --bare               # pyproject.toml만 생성
```

### uv add / uv remove

```bash
uv add requests                       # 일반 의존성
uv add "httpx>=0.27"                  # 버전 제약
uv add pytest --dev                   # dev 그룹에 추가
uv add ruff --group lint              # 특정 의존성 그룹에 추가
uv add pandas --optional analysis     # optional extra에 추가
uv add git+https://github.com/org/repo  # Git 저장소
uv add ./libs/mylib --editable        # 로컬 경로 (editable)
uv add -r requirements.txt            # requirements 파일에서 일괄 추가
uv add requests --frozen              # lock/sync 없이 pyproject만 수정

uv remove requests
uv remove pytest --dev
uv remove ruff --group lint
```

### uv sync

```bash
uv sync                      # lock 기준 동기화 (필요 시 lock 갱신)
uv sync --locked             # uv.lock 변경이 필요하면 실패 (CI용)
uv sync --frozen             # uv.lock을 갱신하지 않고 그대로 사용
uv sync --no-dev             # dev 그룹 제외
uv sync --all-extras         # 모든 extra 포함
uv sync --extra analysis     # 특정 extra 포함
uv sync --group lint         # 특정 그룹 포함
uv sync --all-groups         # 모든 그룹 포함
uv sync --upgrade            # 모든 패키지 업그레이드 후 동기화
uv sync --reinstall          # 전부 재설치
uv sync --inexact            # lock에 없는 패키지도 삭제하지 않음
```

### uv lock

```bash
uv lock                              # lock 파일 생성/갱신
uv lock --check                      # lock이 최신인지 확인만
uv lock --upgrade                    # 모든 패키지 최신 호환 버전으로 업그레이드
uv lock --upgrade-package requests   # 특정 패키지만 업그레이드
```

### uv export

```bash
uv export --format requirements.txt -o requirements.txt
uv export --format pylock.toml -o pylock.toml
uv export --no-dev --no-hashes -o requirements.txt
```

### uv tree

```bash
uv tree                      # 의존성 트리
uv tree --depth 1            # 깊이 제한
uv tree --invert --package urllib3   # 역방향 (누가 이 패키지를 쓰는지)
uv tree --outdated           # 최신 버전 정보 함께 표시
```

### uv version

```bash
uv version                   # 현재 프로젝트 버전 출력
uv version 1.2.0             # 버전 직접 설정
uv version --bump patch      # major | minor | patch | alpha | beta | rc | post | dev
uv version --short           # 버전 숫자만 출력
```

---

## 2. 스크립트 실행

### uv run

프로젝트 환경을 자동으로 동기화한 뒤 명령을 실행합니다.

```bash
uv run python main.py
uv run main.py                         # .py 파일은 python 생략 가능
uv run -m pytest                       # 모듈 실행
uv run pytest -x                       # 환경 내 실행 파일 실행
uv run --with rich main.py             # 임시로 추가 패키지 포함
uv run --python 3.11 main.py           # 특정 Python으로 실행
uv run --no-project script.py          # 프로젝트 무시
uv run --isolated main.py              # 격리된 임시 환경에서 실행
uv run --env-file .env main.py         # .env 파일 로드
uv run --frozen / --locked / --no-sync main.py
```

### PEP 723 인라인 스크립트

```bash
uv init --script tool.py --python 3.12   # 메타데이터 블록 생성
uv add --script tool.py requests         # 스크립트에 의존성 추가
uv remove --script tool.py requests
uv lock --script tool.py                 # 스크립트용 lock 파일 생성
uv run tool.py                           # 의존성 자동 설치 후 실행
```

스크립트 상단 메타데이터 예시:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["requests", "rich"]
# ///
```

shebang으로 직접 실행하기:

```python
#!/usr/bin/env -S uv run --script
```

---

## 3. 코드 품질 (format / check / audit)

### uv format — Ruff 기반 코드 포매팅

```bash
uv format                    # 프로젝트 전체 포매팅
uv format --check            # 변경 없이 검사만 (CI용)
uv format --diff             # 변경 사항 diff만 출력
uv format --version 0.8.0    # 사용할 Ruff 버전 지정
uv format -- --line-length 100   # Ruff에 추가 인자 전달
```

### uv check — 프로젝트 검사 (타입 체크 등)

```bash
uv check                     # 프로젝트 검사
uv check --fix               # 안전한 수정 자동 적용
uv check --all-packages      # 워크스페이스 전체 검사
uv check --package mypkg     # 특정 패키지만
uv check --script tool.py    # PEP 723 스크립트 검사
```

### uv audit — 의존성 보안 취약점 감사

```bash
uv audit                              # 의존성 취약점 검사
uv audit --no-dev                     # dev 그룹 제외
uv audit --output-format json         # text | json | sarif
uv audit --ignore GHSA-xxxx-xxxx-xxxx # 특정 취약점 ID 무시
```

---

## 4. 도구(Tool) 관리

Python 패키지가 제공하는 CLI 도구를 격리 환경에 설치/실행합니다.

| 명령 | 설명 |
|---|---|
| `uv tool run` / `uvx` | 설치 없이 임시 환경에서 도구 실행 |
| `uv tool install` | 도구를 전역 설치 |
| `uv tool upgrade` | 설치된 도구 업그레이드 |
| `uv tool list` | 설치된 도구 목록 |
| `uv tool audit` | 설치된 도구의 취약점 감사 |
| `uv tool uninstall` | 도구 제거 |
| `uv tool update-shell` | 도구 실행 디렉터리를 `PATH`에 추가 |
| `uv tool dir` | 도구 설치 디렉터리 출력 |

```bash
uvx ruff check .                      # = uv tool run ruff check .
uvx ruff@0.8.0 check .                # 특정 버전
uvx ruff@latest check .               # 최신 버전
uvx --from httpie http GET example.com   # 패키지명 ≠ 명령명일 때
uvx --with mkdocs-material mkdocs serve  # 추가 패키지 포함
uvx --python 3.12 black .

uv tool install ruff
uv tool install "ruff==0.8.0"
uv tool install --with mkdocs-material mkdocs
uv tool install -e ./my-cli           # 로컬 editable 설치
uv tool upgrade ruff
uv tool upgrade --all
uv tool list --show-paths
uv tool uninstall ruff
```

---

## 5. Python 버전 관리

| 명령 | 설명 |
|---|---|
| `uv python list` | 사용 가능/설치된 Python 목록 |
| `uv python install` | Python 다운로드 및 설치 |
| `uv python upgrade` | 설치된 Python 최신 패치 버전으로 업그레이드 |
| `uv python find` | Python 설치 경로 검색 |
| `uv python pin` | `.python-version` 파일로 버전 고정 |
| `uv python dir` | uv 관리 Python 설치 디렉터리 |
| `uv python uninstall` | Python 제거 |
| `uv python update-shell` | Python 실행 디렉터리를 `PATH`에 추가 |

```bash
uv python list
uv python list --only-installed
uv python install                     # 최신 버전 설치
uv python install 3.12 3.13           # 여러 버전 설치
uv python install pypy@3.10           # PyPy
uv python install 3.13t               # free-threaded 빌드
uv python install 3.12 --default      # python, python3 실행 파일로도 설치
uv python upgrade 3.12
uv python find 3.12
uv python pin 3.12                    # 현재 디렉터리 .python-version
uv python pin --global 3.12           # 전역 기본 버전
uv python uninstall 3.11
```

---

## 6. pip 호환 인터페이스

기존 `pip` / `pip-tools` 워크플로를 그대로 대체합니다.

| 명령 | 설명 |
|---|---|
| `uv pip install` | 패키지 설치 |
| `uv pip uninstall` | 패키지 제거 |
| `uv pip compile` | `requirements.in` → `requirements.txt` / `pylock.toml` |
| `uv pip sync` | requirements 파일과 환경을 정확히 일치 |
| `uv pip freeze` | requirements 형식으로 설치 목록 출력 |
| `uv pip list` | 표 형식으로 설치 목록 출력 |
| `uv pip show` | 패키지 상세 정보 |
| `uv pip tree` | 환경의 의존성 트리 |
| `uv pip check` | 설치된 패키지 간 호환성 검사 |

```bash
uv pip install requests
uv pip install -r requirements.txt
uv pip install -e .                   # editable 설치
uv pip install ".[dev]"               # extra 포함
uv pip install --system requests      # 시스템 Python에 설치
uv pip install --python 3.12 requests

uv pip compile requirements.in -o requirements.txt
uv pip compile pyproject.toml -o requirements.txt
uv pip compile requirements.in --upgrade
uv pip compile requirements.in --universal   # 플랫폼 독립 해석
uv pip sync requirements.txt

uv pip freeze > requirements.txt
uv pip list --outdated
uv pip show requests
uv pip tree
uv pip check
uv pip uninstall requests
```

---

## 7. 가상환경

```bash
uv venv                              # .venv 생성
uv venv myenv                        # 이름 지정
uv venv --python 3.12                # Python 버전 지정
uv venv --seed                       # pip/setuptools 포함
uv venv --system-site-packages       # 시스템 site-packages 접근 허용
uv venv --prompt myproj              # 프롬프트 이름 지정
uv venv --clear                      # 기존 환경 삭제 후 재생성
```

활성화:

```bash
# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (cmd)
.venv\Scripts\activate.bat
```

> 대부분의 경우 활성화 없이 `uv run`을 쓰는 것이 더 간편합니다.

---

## 8. 빌드 & 배포

```bash
uv build                             # sdist + wheel 빌드 (dist/)
uv build --sdist                     # sdist만
uv build --wheel                     # wheel만
uv build --package mypkg             # 워크스페이스 내 특정 패키지
uv build --all-packages              # 워크스페이스 전체

uv publish                           # dist/* 를 PyPI에 업로드
uv publish --token $PYPI_TOKEN
uv publish --index testpypi          # pyproject.toml에 정의된 인덱스
uv publish --publish-url https://test.pypi.org/legacy/
uv publish --trusted-publishing always   # GitHub Actions 등
```

---

## 9. 워크스페이스

| 명령 | 설명 |
|---|---|
| `uv workspace list` | 워크스페이스 멤버 목록 |
| `uv workspace dir` | 워크스페이스 멤버 경로 출력 |
| `uv workspace metadata` | 워크스페이스 메타데이터 조회 |

```bash
uv workspace list
uv workspace dir --package mypkg
uv run --package mypkg python -m mypkg
uv sync --all-packages
```

`pyproject.toml` 설정 예시:

```toml
[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/legacy"]
```

---

## 10. 인증

| 명령 | 설명 |
|---|---|
| `uv auth login` | 서비스(인덱스)에 로그인 |
| `uv auth logout` | 로그아웃 |
| `uv auth token` | 저장된 인증 토큰 출력 |
| `uv auth dir` | uv 자격 증명 디렉터리 경로 |

```bash
uv auth login example.com --username user --password pass
uv auth login example.com --token $TOKEN
uv auth token example.com
uv auth logout example.com
```

---

## 11. 캐시

```bash
uv cache dir                 # 캐시 디렉터리 경로
uv cache size                # 캐시 크기
uv cache clean               # 전체 캐시 삭제
uv cache clean requests      # 특정 패키지 캐시만 삭제
uv cache prune               # 사용하지 않는 항목 정리
uv cache prune --ci          # CI용 정리 (pre-built wheel 제거, 소스 빌드 결과는 유지)
```

---

## 12. uv 자체 관리

```bash
uv self version              # uv 버전 (= uv --version)
uv self update               # 최신 버전으로 업데이트 (standalone 설치 시)
uv self update 0.13.0        # 특정 버전으로

uv generate-shell-completion powershell   # bash | zsh | fish | elvish | powershell
uv help <command>            # 명령 도움말
```

셸 자동완성 설정:

```bash
# bash
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
```

```powershell
# PowerShell
Add-Content -Path $PROFILE -Value '(& uv generate-shell-completion powershell) | Out-String | Invoke-Expression'
```

---

## 13. 공통 옵션

| 옵션 | 설명 |
|---|---|
| `-q`, `--quiet` | 출력 최소화 |
| `-v`, `--verbose` | 상세 출력 |
| `--color <auto\|always\|never>` | 색상 출력 제어 |
| `--offline` | 네트워크 사용 안 함 |
| `--directory <DIR>` | 지정 디렉터리에서 실행 |
| `--project <DIR>` | 프로젝트 디렉터리 지정 |
| `--config-file <FILE>` | `uv.toml` 설정 파일 지정 |
| `--no-config` | 설정 파일 무시 |
| `-n`, `--no-cache` | 캐시 사용 안 함 |
| `--cache-dir <DIR>` | 캐시 디렉터리 지정 |
| `-p`, `--python <VER>` | Python 버전/경로 지정 |
| `--managed-python` | uv 관리 Python만 사용 |
| `--no-managed-python` | uv 관리 Python 사용 안 함 |
| `--no-python-downloads` | Python 자동 다운로드 비활성화 |
| `--index <URL>` | 추가 패키지 인덱스 |
| `--default-index <URL>` | 기본 인덱스(PyPI) 대체 |
| `--locked` | lock 파일이 변경되면 실패 |
| `--frozen` | lock 파일 갱신 없이 사용 |
| `--no-sync` | 환경 동기화 생략 |
| `--exclude-newer <DATE>` | 지정 날짜 이후 릴리스 제외 (재현성) |
| `--resolution <highest\|lowest\|lowest-direct>` | 버전 해석 전략 |

---

## 14. 주요 환경 변수

| 변수 | 설명 |
|---|---|
| `UV_CACHE_DIR` | 캐시 디렉터리 |
| `UV_NO_CACHE` | 캐시 비활성화 |
| `UV_PYTHON` | 기본 Python 버전/경로 |
| `UV_PYTHON_INSTALL_DIR` | uv 관리 Python 설치 경로 |
| `UV_PYTHON_DOWNLOADS` | `never`로 설정 시 자동 다운로드 비활성화 |
| `UV_MANAGED_PYTHON` / `UV_NO_MANAGED_PYTHON` | 관리 Python 사용 여부 |
| `UV_PROJECT_ENVIRONMENT` | 프로젝트 가상환경 경로 (기본 `.venv`) |
| `UV_INDEX` / `UV_DEFAULT_INDEX` | 패키지 인덱스 URL |
| `UV_INDEX_<NAME>_USERNAME` / `_PASSWORD` | 인덱스별 인증 정보 |
| `UV_LOCKED` / `UV_FROZEN` / `UV_NO_SYNC` | lock/sync 동작 제어 |
| `UV_NO_DEV` | dev 그룹 제외 |
| `UV_EXCLUDE_NEWER` | 지정 날짜 이후 릴리스 제외 |
| `UV_TOOL_DIR` / `UV_TOOL_BIN_DIR` | 도구 설치/실행 파일 디렉터리 |
| `UV_LINK_MODE` | `copy` \| `hardlink` \| `symlink` \| `clone` |
| `UV_COMPILE_BYTECODE` | 설치 시 바이트코드 컴파일 |
| `UV_PUBLISH_TOKEN` | `uv publish` 토큰 |
| `UV_ENV_FILE` | `uv run` 시 로드할 .env 파일 |

---

## 15. 자주 쓰는 워크플로

### 새 프로젝트 시작

```bash
uv init myapp --python 3.12
cd myapp
uv add fastapi uvicorn
uv add --dev pytest ruff
uv run uvicorn main:app --reload
```

### 기존 프로젝트 클론 후 환경 구성

```bash
git clone <repo> && cd <repo>
uv sync
uv run pytest
```

### requirements.txt 프로젝트를 uv 프로젝트로 전환

```bash
uv init --bare
uv add -r requirements.txt
uv add --dev -r requirements-dev.txt
```

### CI 파이프라인

```bash
uv sync --locked --all-extras --dev
uv format --check
uv check
uv audit
uv run pytest
uv build
```

### 패키지 업그레이드

```bash
uv lock --upgrade-package requests   # 특정 패키지
uv lock --upgrade                    # 전체
uv sync
```

### pip → uv 명령 대응표

| pip / 기존 도구 | uv |
|---|---|
| `python -m venv .venv` | `uv venv` |
| `pip install X` | `uv add X` (프로젝트) / `uv pip install X` |
| `pip install -r requirements.txt` | `uv pip install -r requirements.txt` / `uv add -r ...` |
| `pip uninstall X` | `uv remove X` / `uv pip uninstall X` |
| `pip freeze` | `uv pip freeze` |
| `pip-compile` | `uv pip compile` / `uv lock` |
| `pip-sync` | `uv pip sync` / `uv sync` |
| `pipx run X` | `uvx X` |
| `pipx install X` | `uv tool install X` |
| `pyenv install 3.12` | `uv python install 3.12` |
| `pyenv local 3.12` | `uv python pin 3.12` |
| `python -m build` | `uv build` |
| `twine upload dist/*` | `uv publish` |
| `poetry lock` | `uv lock` |
| `poetry run X` | `uv run X` |
