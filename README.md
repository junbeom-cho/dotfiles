# dotfiles

[chezmoi](https://chezmoi.io) 로 관리하는 개발환경 설정. macOS / Linux / Windows 간 마이그레이션용.

## 새 머신 세팅

### 1. chezmoi 설치

**macOS / Linux (Homebrew)**
```shell
brew install chezmoi
```

**Linux (Homebrew 없이)**
```shell
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin
```

**Windows**
```powershell
winget install twpayne.chezmoi
```

### 2. 적용

```shell
chezmoi init --apply git@github.com:junbeom-cho/dotfiles.git
```

clone → 프롬프트 응답 → 홈 디렉터리 적용까지 한 번에 실행됩니다. 프롬프트는 4개입니다.

| 항목 | 설명 |
|---|---|
| `git user.name` | 커밋 author 이름 |
| `personal GitHub username` | 개인 GitHub 계정. 이메일 자동 분기에 사용 |
| `personal git email` | 개인 프로젝트용 이메일 |
| `이 머신의 기본 git email` | 이 머신의 기본값. **개인용 머신이면 그냥 Enter** |

응답은 `~/.config/chezmoi/chezmoi.toml` 에 저장되며 이 저장소에는 올라가지 않습니다.
적용 전에 확인하고 싶으면 `--apply` 를 빼고 `chezmoi diff` 로 먼저 보세요.

## 일상 사용

| 명령 | 설명 |
|---|---|
| `chezmoi edit ~/.zshrc` | 소스 편집 (템플릿이면 템플릿을 엶) |
| `chezmoi add ~/.foo` | 새 파일을 관리 대상에 추가 |
| `chezmoi diff` | 적용하면 뭐가 바뀌는지 미리보기 |
| `chezmoi apply` | 홈에 반영 |
| `chezmoi update` | git pull + apply |
| `chezmoi cd` | 소스 저장소로 이동 |

## git 이메일 자동 분기

`~/.gitconfig` 의 기본 이메일은 머신마다 다르고(위 4번째 프롬프트), 아래 **두 조건 중 하나만 맞아도** 개인 이메일로 전환됩니다.

| 조건 | 판단 기준 |
|---|---|
| `gitdir:~/projects/personal/` | 레포가 그 폴더(및 하위) 안에 있으면 |
| `hasconfig:remote.*.url` | 레포의 remote 가 `github.com/<개인계정>/...` 이면 |

```
~/projects/personal/ 안        →  개인 이메일 (remote 없어도 됨)
개인 GitHub remote             →  개인 이메일 (폴더 위치 무관)
그 외 (회사 GitLab, 타인 레포)  →  머신 기본 이메일
```

두 규칙이 서로를 보완합니다. remote 규칙은 폴더를 어디에 두든 따라오고, 폴더 규칙은 **remote 를 붙이기 전에 한 커밋**까지 잡아줍니다. 그래서 새 개인 프로젝트는 `~/projects/personal/` 에서 시작하는 것을 규칙으로 삼습니다 — 이 경로는 모든 머신에서 동일하게 유지합니다.

이미 개인 GitHub remote 가 붙은 기존 레포는 옮기지 않아도 됩니다.

> ⚠️ 회사 레포를 `~/projects/personal/` 안에 clone 하면 개인 이메일로 커밋됩니다. 두 조건이 같은 파일을 가리켜서 순서로는 막을 수 없으니, 폴더를 섞지 마세요.

확인: `git config --show-origin user.email`

## 관리 대상

| 파일 | 비고 |
|---|---|
| `~/.zprofile` | Homebrew `shellenv`, PATH. OS/아키텍처별 분기 |
| `~/.zshrc` | OS/아키텍처별 Homebrew 경로 분기, 미설치 도구는 자동 skip |
| `~/.gitconfig` | 이메일 템플릿 + 개인/기본 분기 |
| `~/.config/git/personal` | 개인 이메일 override |
| `~/.homebrew/Brewfile` | 설치할 패키지 목록 (formula / cask / VS Code 확장). `dump` 가 덮어씀 |
| `~/.homebrew/Brewfile.optional` | 상황에 따라 켜는 패키지. 손으로 관리, 주석이 유지됨 |

## 패키지 관리

`~/.homebrew/Brewfile` 에 formula / cask / VS Code 확장 목록이 들어 있고, 이 파일이 바뀌면
`chezmoi apply` 시 `run_onchange_before_10-install-packages.sh` 가 `brew bundle` 을 돌립니다.
Homebrew 가 없는 환경(Linux / Windows)에서는 안내만 출력하고 건너뜁니다.

**목록 갱신** — 새 패키지를 설치했으면 현재 상태를 그대로 덤프해서 반영합니다.

```shell
brew bundle dump --global --force   # 설치 상태 → ~/.homebrew/Brewfile
chezmoi add ~/.homebrew/Brewfile    # 소스에 반영
```

이 파일을 템플릿(`.tmpl`)으로 만들지 않은 이유가 여기 있습니다. `dump` 가 파일을 통째로
덮어쓰기 때문에 템플릿이면 조건문이 매번 날아갑니다. OS 분기는 파일이 아니라 설치 스크립트에
두었습니다.

**상황에 따라 켜는 패키지** — `~/.homebrew/Brewfile.optional` 에 주석 상태로 보관합니다.

```ruby
# --- 주변기기 / 하드웨어 의존 ---
# cask "displaylink"       # DisplayLink 도킹스테이션 드라이버
# cask "logi-options+"     # 로지텍 마우스·키보드 설정 앱
```

필요한 줄의 주석을 풀고 `chezmoi apply` 하면 설치됩니다. 전부 주석 상태면 설치 스크립트가
이 파일을 아예 건너뜁니다.

**메인 Brewfile 에는 주석을 달지 마세요.** `brew bundle dump` 가 파일을 통째로 덮어쓰기
때문에 다음 덤프에서 사라집니다. `dump` 가 건드리지 않는 파일이 필요해서 분리한 것입니다.

> 이 파일도 저장소에 커밋되므로 주석을 풀면 모든 머신에 전파됩니다. Mac 이 한 대인 동안은
> 문제가 없고, 두 대가 되면 그때 머신별로 나눕니다.

**다른 머신에 반영** — `chezmoi update` 한 번이면 pull → apply → 패키지 설치까지 이어집니다.

> 덤프에 특정 기기에서만 의미 있는 cask 가 딸려 들어왔으면 메인 Brewfile 에서 빼고
> `Brewfile.optional` 로 옮겨두세요. 회사/개인 Brewfile 분리는 두 번째 Mac 이 생기면 그때 합니다.

## 요구사항 / 현재 한계

- git 2.36 이상 (`hasconfig` includeIf 사용)
- Homebrew 자체는 자동 설치하지 않습니다. 없으면 패키지 설치를 건너뛰므로 [brew.sh](https://brew.sh) 를 보고 먼저 설치하세요.
- oh-my-zsh 는 아직 Brewfile 로 관리되지 않습니다. 없으면 `.zshrc` 가 에러 없이 건너뜁니다.
- Linux / Windows 용 패키지 설치는 아직 없습니다. macOS 만 자동화돼 있습니다.
