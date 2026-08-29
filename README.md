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
| `~/.zshrc` | OS/아키텍처별 Homebrew 경로 분기, 미설치 도구는 자동 skip |
| `~/.gitconfig` | 이메일 템플릿 + 개인/기본 분기 |
| `~/.config/git/personal` | 개인 이메일 override |

## 요구사항 / 현재 한계

- git 2.36 이상 (`hasconfig` includeIf 사용)
- `.zshrc` 가 참조하는 도구(oh-my-zsh, powerlevel10k, lsd, bat, zsh-autosuggestions, zsh-syntax-highlighting, jenv, nvm)는 **아직 자동 설치되지 않습니다.** 설치돼 있지 않으면 에러 없이 건너뜁니다.
