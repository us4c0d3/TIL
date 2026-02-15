# 모던 파이썬 개발 - 1 패키지 관리

## 0. 학습 배경

https://github.com/modelcontextprotocol/python-sdk/blob/main/CLAUDE.md

MCP에 대해 알게 되었고, 이를 알아보기 위해 레포에 직접 들어갔다가 클로드 프롬프트에서 pip 대신 uv를 사용해라라는 내용이 있어서 uv에 대해 알게됨. 이를 기점으로 좀 더 파보니 내가 이전에 알던 파이썬 개발 방식과 바뀐 점이 많아 찾아보게 됨.

## 1. 배경 지식

### PyPI

코드 까보면서 알았는데 `pip`는 파이썬으로 구현되어 있었음. 당연히 속도를 생각해서 C로 구현된 줄 알았음.

그런데 패키지 매니저이기 때문에 연산 속도보다 중요한 것이 관리의 용이성 및 파이썬 개발자의 기여 편의성이라고 생각되어 파이썬으로 구현되었다고 함.

이전까지는 큰 문제가 없었으나 파이썬 생태계가 발전하면서 점점 문제가 발생함. 대표적으로 2가지가 있음

1. 의존성 문제
  pip 자체가 파이썬 패키지에 의존성을 가지고 있기 때문에 의존성 충돌, 다이아몬드 의존성 문제, 설치할 때마다 달라지는 버전 등 여러 문제를 일으켰음.
    이러한 문제를 해결하기 위해 나온 것이 venv를 이용한 의존성 격리, `requirements.txt`등을 이용한 버전관리임.
    또한 `pip 20.3` 버전부터는 Resolver라는 것이 생겨, 모든 의존성을 계산해보고 문제가 생긴다면 설치를 아예 시작하지 않는 기능이 생김. 그러나 이러한 연산을 할 때 속도가 느리다는 문제가 생김

2. 속도
  어떻게 보면 1번에서 파생된 문제.
    최근에 MCP도 나오고 LLM을 이용한 개발이 대중화되었음. 그러면서 개발 규모도 커지고 사용하는 패키지 수도 엄청나게 많아짐.
    그로 인해 패키지 매니저가 관리해야 될 의존성도 많아졌고, 상기했듯 리졸버는 의존성 충돌이 생기는지 연산을 해야되는데 패키지가 많아 그 속도가 굉장히 느려짐

이러한 문제를 해결하기 위해 uv라는 것이 출시됨.

참고로 파이썬 3.12가 출시될 즈음에 [PEP 668](https://peps.python.org/pep-0668/)이 대부분의 OS에 적용되어 가상환경 사용이 강제되었고, 매 번 패키지를 새로 설치해야되는 번거로움이 생김.

## 2. uv

[repo](https://github.com/astral-sh/uv)

![uv_chart](./images/uv_chart.svg)

> A single tool to replace pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv, and more.
10-100x faster than pip.
Provides comprehensive project management, with a universal lockfile.
Runs scripts, with support for inline dependency metadata.
Installs and manages Python versions.
Runs and installs tools published as Python packages.
Includes a pip-compatible interface for a performance boost with a familiar CLI.
Supports Cargo-style workspaces for scalable projects.
Disk-space efficient, with a global cache for dependency deduplication.
Installable without Rust or Python via curl or pip.
Supports macOS, Linux, and Windows.

> pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv 등을 대체하는 단일 도구입니다.
> pip보다 10~100배 빠릅니다.
> 범용 잠금 파일을 포함한 포괄적인 프로젝트 관리 기능을 제공합니다.
> 인라인 종속성 메타데이터를 지원하며 스크립트를 실행합니다.
> Python 버전을 설치하고 관리합니다.
> Python 패키지로 게시된 도구를 실행하고 설치합니다.
> 익숙한 CLI 환경에서 성능을 향상시키는 pip 호환 인터페이스를 제공합니다.
> 확장 가능한 프로젝트를 위한 Cargo 스타일 워크스페이스를 지원합니다.
> 종속성 중복 제거를 위한 전역 캐시를 통해 디스크 공간을 효율적으로 사용합니다.
> curl 또는 pip를 통해 Rust나 Python 없이도 설치할 수 있습니다.
> macOS, Linux, Windows를 지원합니다.

uv 레포의 하이라이트 부분을 긁어왔다. 요약하자면 pip보다 훨씬 빠르고 pip와 호환이 되며, 가상환경을 대체하는 툴이다.

### 속도가 빠른 이유

Rust로 개발되어 빠르다고 할 수 있지만 더 정확히 말하면 파이썬 의존성 관리 기법에 대한 최적화라고 할 수 있다. 레거시 파이썬에서는 `.egg`, `pip.conf` 및 `setup.py`를 이용해 의존성을 관리했는데, uv에서는 레거시 호환 기능을 제거했다. 

또한, uv에는 병렬 다운로드, 하드링크 기반 글로벌 캐시, HTTP range request(HTTP/2), PubGrub 의존성 해석 알고리즘 등이 적용되어 속도를 빠르게 했다. 이는 굳이 Rust가 아니어도 적용시킬 수 있다.

### pip vs uv

| 작업 | pip + venv | uv |
| :---: | :-----------| :------ |
| 프로젝트 초기화 | `mkdir my-project`<br/>`cd my-project`<br/>`python -m venv .venv` | `uv init my-project`<br/>`cd my-project` |
| 가상환경 생성 | `python -m venv .venv` | uv init 시 자동 생성 |
| 가상환경 활성화 | `source .venv/bin/activate` | 자동 활성화 (패키지 설치/실행 시) |
| 패키지 설치 | `pip install {package}` | `uv pip install {package}`<br/>또는<br/>`uv add {package}` |
| 의존성 관리 | `pip freeze > requirements.txt`<br/>`pip install -r requirements.txt` | `uv pip freeze > requirements.txt`<br/>`pip install -r requirements.txt`<br/>또는<br/>`uv lock`<br/>`uv sync` |
| 파이썬 버전 관리 | 수동 설치 및 관리 | `uv python install 3.12`<br/>`uv run --python 3.12 script.py` |
| 의존성 해결 | 순차적 처리 | 병렬 처리 (PubGrub 알고리즘) |
| 캐시 관리 | 프로젝트별 캐시 | 전역 캐시 시스템 |
| 디스크 공간 | 각 프로젝트별 복사본 | 하드 링크 사용 |
| 네트워크 최적화 | HTTP/1.1 | HTTP/2 지원, 연결 풀링 |

#### 주요 차이점

1. 가상환경 관리 자동화
  - pip: 수동으로 가상환경 생성하고 활성화해야 함.
  - uv: 프로젝트 초기화 시 자동으로 가상환경 생성, 패키지 설치/실행 시 자동 활성화
2. 의존성 관리 방식
  - pip: 기본적인 패키지 설치/제거 기능
  - uv: 프로젝트 의존성을 체계적으로 관리 (`uv add`), 버전 잠금 기능 (`uv lock`)
3. 성능 최적화
  - pip: 순차적 처리, 프로젝트별 캐시
  - uv: 병렬 처리, 전역 캐시, 하드 링크 사용
4. python 버전 관리
  - pip: Python 버전 관리를 지원하지 않음
  - uv: 통합된 Python 버전 관리 기능 제공

사실 표를 보면 알겠지만, 기존 pip 명령어에 접두사로 uv를 붙이면 거의 완벽하게 호환이 된다. 이는 기존 pip 프로젝트와의 호환성을 위해 이렇게 설계되었다고 한다.

## 3. uv를 이용한 프로젝트 관리

기존에 pip를 이용한 개발을 할 때 가장 많이 썼던 의존성 관리 기법은 `pip freeze > requirements.txt`로 해당 프로젝트에서 사용한 **모든 패키지**를 `requirements.txt`에 저장하는 식이었다. 그러나 이 방법은 내가 직접 설치한 패키지 뿐만 아니라 그 패키지를 설치하기 위한 하위 패키지까지 모두 작성된다.

그러나 uv를 이용한 현재의 파이썬 생태계에서는 `pyproject.toml` 파일 하나로 프로젝트의 모든 설정(의존성, 파이썬 버전, 빌드 설정 등)을 관리한다.

### 1. 프로젝트 전체 구조

`uv init` 명령어를 실행하면 다음과 같은 표준 구조가 생성된다.

```
my-project/
├── .venv/              # uv가 자동으로 관리하는 가상환경
├── pyproject.toml      # 프로젝트 설정 및 의존성 정의
├── uv.lock             # 의존성 버전이 고정된 파일 (pip의 freeze 역할)
├── .python-version     # 프로젝트에서 사용할 파이썬 버전 명시
├── README.md           # 프로젝트 설명
└── main.py (또는 src/)  # 소스 코드
```

### 2. `pyproject.toml` 구조

1. [Project] (표준 메타데이터)

프로젝트 이름, 버전, 그리고 직접 설치한 패키지들이 나열된다. `pip freeze`와 같이 모든 하위 의존성까지 작성되는 것이 아니라 직접 설치한 **최상위 패키지**만 작성된다.

```toml
[project]
name = "my-app"
version = "0.1.0"
dependencies = [
    "fastapi>=0.115.0",
    "pydantic",
]
requires-python = ">=3.12"
```

2. [dependency-groups] (개발용 의존성)

실제 서비스 운영에는 필요없지만 개발할 때만 쓰는 도구를 분리해서 관리한다.

```toml
[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "ruff",
]
```

`uv add --dev {package}`를 이용해 패키지를 추가하면 이 영역에 들어간다.

3. [tool.py] (uv 전용 설정)

uv 도구 자체에 대한 설정이 있다. 소스 경로를 지정하거나 특정 인덱스(PyPI 외의 저장소)를 설정할 때 사용한다.

### 4. 명령어 추가 설명

- `uv sync`: `pyproject.toml`내용에 맞춰 가상환경과 `uv.lock`을 동기화시킴
- `uv lock`: `pyproject.toml`파일에 정의된 패키지 목록을 읽어, 실제로 설치 가능한 최적의 버전 조합을 계산하고 그 결과를 `uv.lock` 파일에 기록한다. `uv.lock`파일만 생성하거나 업데이트 한다. 일반적으로 해당 명령어는 `uv.lock`파일 최신화를 할 때 사용된다.
  - `uv lock --upgrade`: `pyproject.toml`에 적힌 범위 내에서 가장 최신 버전으로 모든 패키지를 업데이트하여 다시 잠근다. 의존성 최신화를 할 때 사용한다. (보안 패치 등)