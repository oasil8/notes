# Git & GitHub 완전정복 노트

> **목표:** 팀 협업 가능한 중급 수준  
> **용도:** 개인 프로젝트 + 공모전/동아리 팀 협업  
> **환경:** Windows + MINGW64 (Git Bash)  
> **작성:** JeongWoo Song · 2026.06

---

## 목차

1. [핵심 멘탈 모델](#1-핵심-멘탈-모델)
2. [Stage 1 — Git 기초](#2-stage-1--git-기초)
3. [Stage 2 — GitHub 연동](#3-stage-2--github-연동)
4. [Stage 3 — 브랜치·협업](#4-stage-3--브랜치협업)
5. [Stage 4 — 실전 습관](#5-stage-4--실전-습관)
6. [명령어 치트시트](#6-명령어-치트시트)
7. [일일 루틴](#7-일일-루틴)

---

## 전체 로드맵

| Stage | 내용 | 기간 |
|-------|------|------|
| 1 | Git 핵심 개념 (3-tree / init / add / commit / log) | 1주 |
| 2 | GitHub 연동 (remote / push / pull / PAT / SSH / README) | 1주 |
| **3** | **브랜치 전략 / PR / Merge Conflict ← 핵심** | **2주** |
| 4 | 커밋 컨벤션 / rebase / reset / Issues & Projects | 지속 |

---

## 1. 핵심 멘탈 모델

> Git의 본질: **"파일 변경 이력을 스냅샷으로 저장하는 시스템"**  
> 명령어를 외우지 말고 "지금 스냅샷에 무슨 일이 일어나는가?"를 물어봐.

### 3-tree 구조

세 영역이 존재하는 이유: *"저장하고 싶은 변경사항을 **선택적으로 골라서** 의미 있는 단위로 묶고 싶다."*

```
┌─────────────────────┐   git add    ┌──────────────────┐   git commit  ┌─────────────────┐
│  Working Directory  │ ──────────→  │  Staging Area    │ ────────────→ │  Repository     │
│  (작업 책상)         │              │  (포장 테이블)    │               │  (.git/ 창고)   │
│                     │ ←──────────  │                  │               │                 │
│  실제 파일 수정 공간  │ git restore  │  커밋 예약 목록   │               │  영구 스냅샷     │
└─────────────────────┘  --staged    └──────────────────┘               └─────────────────┘
```

| 영역 | 내부 저장 위치 | 상태 키워드 |
|------|--------------|------------|
| Working Directory | 실제 파일시스템 | `modified`, `untracked` |
| Staging Area | `.git/index` | `staged`, `changes to be committed` |
| Repository | `.git/objects/` | `committed` |

> **왜 Staging이 필요한가?**  
> 오늘 파일 5개를 수정해도 그 중 2개만 골라서 "버그 수정" 커밋, 나머지 3개는 "기능 추가" 커밋으로 분리할 수 있기 때문.

---

## 2. Stage 1 — Git 기초

### 2-1. 환경 설정 (최초 1회)

```bash
git config --global user.name "JeongWoo Song"
git config --global user.email "sjg406302@gmail.com"
git config --list   # 확인
```

> 설정은 `~/.gitconfig`에 저장됨 → **머신(PC) 단위**. 노트북에서도 동일하게 설정해야 함.

---

### 2-2. git init

```bash
mkdir my-project && cd my-project
git init
# → .git/ 폴더 생성. 이 폴더가 모든 버전 정보의 창고.
```

> ⚠ 홈 디렉토리(`~`)에서 `git init` 절대 금지. 반드시 프로젝트 폴더 안에서 실행.

---

### 2-3. git add / commit

```bash
# 파일 만들기
echo "# My Project" > README.md
echo "print('hello')" > main.py

# Staging
git add README.md          # 특정 파일만
git add .                  # 현재 폴더 전체

# 커밋
git commit -m "Initial commit: add README and main.py"
# → [main a3f8c21] Initial commit ...
# → a3f8c21 = 커밋 해시 (이 버전의 "주민번호")
```

**자주 하는 실수**

| 실수 | 원인 | 해결 |
|------|------|------|
| `git commit` 했는데 아무것도 안 들어감 | `git add`를 안 함 | add → commit 순서 지키기 |
| 수정했는데 커밋에 반영 안 됨 | add 이후에 또 수정함 | 수정 후 다시 `git add` |
| 관계없는 파일이 같이 올라감 | `git add .` 남발 | 파일명 지정 or `.gitignore` 활용 |

---

### 2-4. git status / diff / log

> 세 명령어 모두 **읽기 전용**. 아무것도 변경하지 않아 — 마음껏 쳐봐도 됨.

#### git status

```bash
git status

# 출력 해석
# Changes not staged:  → 빨강 = WD에만 있음, add 안 함
# Changes to be committed: → 초록 = Staging에 있음, 커밋 준비 완료
# nothing to commit, working tree clean → 모든 것이 커밋된 깨끗한 상태
```

#### git diff

```bash
git diff              # WD ↔ Staging 비교 (add 안 한 것만)
git diff --staged     # Staging ↔ 마지막 커밋 비교 ← 커밋 전 필수 확인
git diff HEAD         # WD ↔ 마지막 커밋 (staged든 아니든 전체)
git diff abc123 def456 # 두 커밋 해시 간 비교
```

> `+` (초록) = 추가된 줄, `-` (빨강) = 삭제된 줄

#### git log

```bash
git log                          # 기본 (상세)
git log --oneline                # 한 줄 요약 ← 가장 자주 씀
git log --oneline --graph --all  # 브랜치 분기 시각화
git log --stat                   # 파일별 변경 줄 수 요약
git log -p -1                    # 최근 1개 커밋의 diff 포함
git log -n 5                     # 최근 5개만
git log --author="JeongWoo"      # 특정 작성자만
```

**실전 루틴**

```bash
git status           # 1. 현재 상태 파악
git diff             # 2. 줄 단위 변경사항 확인
git add main.py      # 3. 원하는 파일만 staging
git diff --staged    # 4. 커밋 전 최종 확인
git commit -m "feat: refactor greet()"
git log --oneline    # 5. 커밋 히스토리 확인
```

---

## 3. Stage 2 — GitHub 연동

### 3-1. 원격 저장소 연결 & Push

> **개념:** 로컬 Repository = 내 PC 창고. GitHub = 인터넷 창고. `remote` = 주소록. `push/pull` = 택배.

```bash
# 최초 연결 (딱 한 번)
git remote add origin https://github.com/JeongWooSong/my-project.git
git branch -M main          # 브랜치 이름 main으로 통일
git remote -v               # 연결 확인
git push -u origin main     # 첫 push (-u는 딱 한 번만)

# 이후 매일 반복 루틴
git add .
git commit -m "메시지"
git push                    # -u 이후엔 이것만

# GitHub → 로컬 최신 내용 가져오기
git pull
```

**자주 막히는 지점**

| 에러 | 원인 | 해결 |
|------|------|------|
| `Support for password authentication was removed` | 비밀번호로 push 시도 | Password 칸에 PAT 토큰 입력 |
| `failed to push some refs` | README 체크로 히스토리 불일치 | `git pull origin main --allow-unrelated-histories` 후 push |
| `main` vs `master` 불일치 | 구버전 Git 기본 브랜치명 | `git branch -M main`으로 통일 |

---

### 3-2. 인증 — PAT vs SSH

| | PAT | SSH |
|--|-----|-----|
| 설정 시간 | 5분 | 15분 |
| 만료 | 있음 (주기적 재발급) | 없음 |
| 편의성 | push마다 토큰 입력 (또는 저장) | 한 번 설정 후 자동 |
| 보안 | 보통 | 강함 (비대칭 암호화) |
| 추천 용도 | 임시 환경, 빠른 시작 | 본인 PC 장기 사용 ← **추천** |

#### PAT 발급

```
GitHub → Settings → Developer settings
→ Personal access tokens → Tokens (classic)
→ Generate new token (classic)
→ repo 권한 체크 → Generate
→ 토큰 즉시 복사 (다시 못 봄!)
```

Windows Credential Manager에 저장:
```bash
git config --global credential.helper manager
# 첫 push 시 Username = GitHub 아이디, Password = PAT 토큰 입력
# 이후 자동 저장
```

#### SSH 설정

```bash
# 1. 키 생성
ssh-keygen -t ed25519 -C "sjg406302@gmail.com"
# 세 번 다 Enter (기본 경로 + 비밀번호 없음)

# 2. ssh-agent에 등록
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. 공개키 복사
cat ~/.ssh/id_ed25519.pub | clip
# → github.com/settings/ssh/new 에 붙여넣기 → Add SSH key

# 4. 연결 테스트
ssh -T git@github.com
# → "Hi JeongWooSong! You've successfully authenticated." 뜨면 성공

# 5. remote URL SSH로 변경
git remote set-url origin git@github.com:JeongWooSong/my-project.git
git push   # 비밀번호 없이 바로 됨
```

> HTTPS URL: `https://github.com/유저명/레포.git`  
> SSH URL: `git@github.com:유저명/레포.git` ← 콜론(:) 위치 주의

> **노트북도 쓴다면:** 노트북에서 1~4단계 동일하게 진행 후 GitHub에 두 번째 키로 등록. PC당 키 하나가 원칙.

---

### 3-3. README 작성법

> **목적:** 공모전 심사위원·채용담당자가 30초 안에 "이 사람 실력 있다"는 판단을 내리게 하는 것.

**5가지 원칙**

1. **30초 룰** — 스크롤 없이 보이는 첫 화면에 "뭔지", "왜 만들었는지", "얼마나 잘 만들었는지" 전부
2. **재현 가능성** — 복붙하면 바로 돌아가는 설치 명령어 (Python 버전, OS 조건 포함)
3. **수치로 말해** — "성능 개선" X → "FEA 대비 설계 시간 78% 단축, 무게 23% 절감" O
4. **뱃지** — shields.io에서 기술스택·라이선스·빌드 상태 뱃지. 신뢰의 시그널
5. **독자를 특정** — 공모전용: 문제정의+결과수치+라이선스 / 팀 협업용: 설치법+기여방법

**에어론 프로젝트 README 템플릿**

```markdown
# Aeron Drone Frame
> GNN + RL 기반 항공우주 구조 최적화 드론 프레임 설계 자동화

![Python](https://img.shields.io/badge/Python-3.10-3572A5?logo=python)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0-EE4C2C?logo=pytorch)
![License](https://img.shields.io/badge/License-MIT-green)

## 개요
기존 FEA 대비 **설계 시간 78% 단축**, **구조 무게 23% 절감**을 달성한
드론 프레임 위상 최적화 도구입니다.

## 주요 기능
- GNN 기반 실시간 응력 예측 (FEA 대비 94% 정확도)
- RL 에이전트 자동 위상 최적화
- Isaac Sim Sim-to-Real 검증 파이프라인

## 설치
git clone https://github.com/JeongWooSong/aeron-drone-frame
cd aeron-drone-frame
pip install -r requirements.txt
python main.py --config configs/default.yaml

## 기술 스택
Python 3.10 · PyTorch · Stable-Baselines3 · ROS2 · Isaac Sim

## 라이선스
MIT © 2026 JeongWoo Song (에어론)
```

**섹션별 중요도**

| 섹션 | 중요도 | 비고 |
|------|--------|------|
| 프로젝트 제목 & 뱃지 | 필수 | 첫인상 결정 |
| 데모 이미지/GIF | 필수 | 30초 룰의 핵심 |
| 한 줄 소개 (What & Why) | 필수 | 존재 이유 |
| 설치 & 실행 방법 | 필수 | 재현 가능성 |
| 주요 기능 목록 | 권장 | 핵심 3~5개 |
| 기술 스택 | 권장 | 기술 어필 |
| 라이선스 | 권장 | 공모전 제출 시 필수 |
| 프로젝트 구조 | 선택 | 대형 프로젝트에서 유용 |
| 기여 방법 | 선택 | 팀/오픈소스 필수 |

---

## 4. Stage 3 — 브랜치·협업

> **핵심:** 브랜치는 "내 작업 공간 격리", PR은 "검토 후 합치자", 충돌 해결은 "두 버전을 사람이 중재".

### 4-1. 브랜치 전략 — GitHub Flow

소규모 팀(공모전/동아리)에 최적. 규칙은 하나 — **main은 항상 배포 가능한 상태**.

```
main       ●──────────────────────────────● merge
            ╲                            ╱
feat/login   ●──●──●──●──●  PR → merge
```

**브랜치 이름 컨벤션**

```
feat/login-page       # 새 기능
fix/null-crash        # 버그 수정
docs/update-readme    # 문서
refactor/auth-module  # 리팩토링
hotfix/payment-bug    # 긴급 수정
```

**브랜치 작업 전체 흐름**

```bash
# 항상 최신 main에서 시작
git checkout main && git pull origin main

# 새 브랜치 생성 + 이동 (한 번에)
git checkout -b feat/login-page

# 작업 후 push
git add . && git commit -m "feat: add login page"
git push origin feat/login-page
# → GitHub에서 PR 생성

# Merge 후 정리
git checkout main && git pull origin main
git branch -d feat/login-page                    # 로컬 삭제
git push origin --delete feat/login-page         # 원격 삭제
```

**브랜치 관리 명령어**

```bash
git branch            # 로컬 브랜치 목록
git branch -a         # 원격 포함 전체 목록
git checkout 브랜치명  # 브랜치 이동
git branch -d 브랜치명 # 브랜치 삭제 (merge 후)
```

> 💡 브랜치 수명은 최대 2~3일이 이상적. 오래 살수록 main과 벌어지고 충돌이 눈덩이처럼 불어남.

---

### 4-2. Pull Request

> "내 브랜치를 main에 합쳐도 될까요?" 라는 공식 요청. 코드 리뷰와 토론이 여기서 일어남.

**좋은 PR의 조건**

- PR 하나 = 목적 하나. 기능+버그수정+리팩토링 섞으면 리뷰 불가
- 파일 변경 10개 이하가 리뷰어에게 친절한 선
- 본문 구조: 변경사항(무엇을 왜) + 테스트 체크리스트 + 스크린샷/로그

**PR 본문 템플릿**

```markdown
## 변경 사항
- JWT 기반 로그인 엔드포인트 추가
- 토큰 만료 처리 로직 구현

## 테스트
- [ ] 로그인 성공 시나리오
- [ ] 토큰 만료 후 재발급

## 관련 이슈
Closes #12
```

**Merge 방식 비교**

| 방식 | 특징 | 추천 상황 |
|------|------|----------|
| Merge commit | 브랜치 히스토리 전체 보존 | 히스토리 추적 중요한 경우 |
| **Squash and merge** | 여러 커밋을 1개로 압축 | **소규모 팀 공모전 ← 추천** |
| Rebase and merge | 선형 히스토리 유지 | 깔끔한 히스토리 원하는 경우 |

> Squash merge: 브랜치의 잡다한 커밋들이 하나로 정리돼서 main 히스토리가 읽기 쉬워짐.

---

### 4-3. Merge Conflict 해결

> 두 브랜치가 **같은 파일의 같은 줄**을 다르게 수정했을 때 발생. 당황하지 마 — 정상적인 상황.

**충돌 마커 구조**

```
<<<<<<< HEAD  ← 현재 브랜치 (main)
def greet(name):
    return f"Hello, {name}!"
=======  ← 구분선
def greet(name, formal=False):
    prefix = "Dear" if formal else "Hello"
    return f"{prefix}, {name}!"
>>>>>>> feat/greeting  ← 들어오는 브랜치
```

**해결 3단계**

```bash
# 1. 충돌 파일 확인
git status   # "both modified" 로 표시된 파일들

# 2. 에디터에서 마커(<<<, ===, >>>)와 필요없는 코드 제거 후 원하는 최종 형태만 남기기

# 3. 해결 후 커밋
git add main.py
git commit -m "merge: resolve conflict in greet()"
```

**충돌 예방 3원칙**

```bash
# 1. 작업 시작 전 항상 pull
git pull origin main

# 2. 작업 중 main이 업데이트되면 내 브랜치에 반영
git checkout feat/my-feature && git merge main

# 3. 브랜치 수명 짧게 유지 (2~3일)
```

> ✅ 충돌의 90%는 "오래된 브랜치에서 작업"이 원인. 매일 main을 pull하면 대부분 예방.

---

## 5. Stage 4 — 실전 습관

### 5-1. 커밋 메시지 컨벤션 — Conventional Commits

**형식:** `타입(스코프): 변경 내용 요약`

```
feat(auth): add JWT login endpoint
fix(greet): handle null name input
docs: update README installation guide
refactor(auth): extract token validation logic
test(auth): add unit tests for login flow
chore: update dependencies
```

**타입 종류**

| 타입 | 의미 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 | `feat: add login page` |
| `fix` | 버그 수정 | `fix: handle null pointer in greet()` |
| `docs` | 문서 수정 | `docs: update README` |
| `refactor` | 기능 변경 없는 코드 구조 개선 | `refactor: extract auth module` |
| `test` | 테스트 추가/수정 | `test: add login unit tests` |
| `chore` | 빌드 시스템, 설정 파일 변경 | `chore: update dependencies` |
| `style` | 포맷팅, 세미콜론 등 (기능 변경 없음) | `style: fix indentation` |

**좋은 커밋 메시지 5원칙**

1. 동사 원형으로 시작 — "Add", "Fix", "Remove" (과거형 X)
2. 50자 이내 요약 — 이메일 제목처럼 간결하게
3. "무엇을" + "왜" 포함 — "fix bug" X → "fix null crash when name is empty" O
4. 한 커밋 = 한 목적 — 기능 추가와 버그 수정을 섞지 않기
5. 현재 시제 — "Added" X → "Add" O

---

### 5-2. rebase / reset / stash

#### git rebase

커밋들을 다른 베이스 위로 재배치. 선형 히스토리 유지.

```bash
# feature 브랜치를 최신 main 위로 올리기
git checkout feat/login
git rebase main

# 대화형 rebase — 커밋 순서 변경/합치기/삭제
git rebase -i HEAD~3   # 최근 3개 커밋 편집
```

> ⚠ **이미 push한 커밋에는 rebase 금지.** 팀원의 히스토리와 충돌해. 로컬 + push 전에만.

#### git reset

HEAD 포인터를 과거 커밋으로 이동.

```bash
git reset --soft HEAD~1    # 커밋만 취소, 변경사항은 Staging 유지
git reset --mixed HEAD~1   # 커밋 취소 + Staging 취소, WD에 변경사항 유지 (기본값)
git reset --hard HEAD~1    # 완전 롤백 — 변경사항 모두 삭제 ⚠ 위험
```

| 옵션 | 커밋 | Staging | WD |
|------|------|---------|-----|
| `--soft` | 취소 | 유지 | 유지 |
| `--mixed` | 취소 | 취소 | 유지 |
| `--hard` | 취소 | 취소 | **삭제** |

#### git stash

작업 중 급하게 브랜치를 전환해야 할 때 임시 저장.

```bash
git stash              # 현재 변경사항 임시 저장
git stash pop          # 최근 stash 복원 + 삭제
git stash list         # stash 목록 확인
git stash apply        # 복원 (stash 유지)
git stash drop         # stash 삭제
```

---

### 5-3. GitHub Issues & Projects

#### Issues — 할 일 관리

- 버그 리포트, 기능 요청, 토론 모두 Issue로 관리
- Label로 분류: `bug`, `enhancement`, `documentation`, `help wanted`
- PR 본문에 `Closes #12` 작성하면 merge 시 Issue 자동 닫힘
- Assignee로 담당자 지정, Milestone으로 버전/마감일 관리

#### Projects — 칸반 보드

- Todo / In Progress / Done 3열 칸반 기본 구성
- Issue와 PR을 카드로 추가해서 진행 상황 시각화

**공모전 팀 협업 플로우**

```
기능별 Issue 생성
    → 브랜치 만들어서 작업 (feat/이름)
    → PR 생성 (Closes #이슈번호 포함)
    → 코드 리뷰 & 승인
    → Squash and merge
    → Issue 자동 닫힘
    → 브랜치 삭제
```

**팀 레포 시작 전 반드시 정해야 할 4가지**

1. 브랜치 이름 규칙 (`feat/`, `fix/`, `docs/` 등)
2. 커밋 메시지 컨벤션 (Conventional Commits)
3. PR 최소 승인자 수 (1명 이상 approve 후 merge)
4. main 직접 push 금지 (Branch protection rule 설정)

---

## 6. 명령어 치트시트

### 기초

| 명령어 | 설명 |
|--------|------|
| `git init` | 현재 폴더를 Git 저장소로 초기화 |
| `git add <file>` | 파일을 Staging Area로 이동 |
| `git add .` | 모든 변경사항을 Staging Area로 |
| `git commit -m "msg"` | Staging → Repository 스냅샷 저장 |
| `git status` | 현재 WD/Staging 상태 확인 |
| `git log --oneline` | 커밋 히스토리 한 줄 요약 |
| `git diff` | WD ↔ Staging 변경사항 비교 |
| `git diff --staged` | Staging ↔ 마지막 커밋 비교 |
| `git restore --staged <file>` | Staging 취소 |
| `git commit --amend` | 마지막 커밋 메시지 수정 |

### 원격 저장소

| 명령어 | 설명 |
|--------|------|
| `git remote add origin <url>` | 원격 저장소 주소 등록 |
| `git remote -v` | 연결된 원격 저장소 확인 |
| `git push -u origin main` | 첫 push + upstream 설정 |
| `git push` | 이후 push (-u 설정 후) |
| `git pull` | 원격 → 로컬 최신 내용 동기화 |
| `git clone <url>` | 원격 레포를 로컬에 복제 |
| `git remote set-url origin <url>` | remote URL 변경 |

### 브랜치 & 협업

| 명령어 | 설명 |
|--------|------|
| `git checkout -b feat/이름` | 새 브랜치 생성 + 이동 |
| `git checkout 브랜치명` | 브랜치 이동 |
| `git branch` | 로컬 브랜치 목록 |
| `git branch -a` | 원격 포함 전체 브랜치 목록 |
| `git branch -d 브랜치명` | 브랜치 삭제 (merge 후) |
| `git push origin --delete 브랜치명` | 원격 브랜치 삭제 |
| `git merge 브랜치명` | 현재 브랜치에 대상 브랜치 합치기 |
| `git rebase main` | 현재 브랜치를 main 위로 재배치 |

### 히스토리 수정

| 명령어 | 설명 |
|--------|------|
| `git reset --soft HEAD~1` | 마지막 커밋 취소, staging 유지 |
| `git reset --mixed HEAD~1` | 마지막 커밋 취소, WD로 되돌림 |
| `git reset --hard HEAD~1` | 완전 롤백 ⚠ 복구 어려움 |
| `git stash` | 작업 중 임시 저장 |
| `git stash pop` | 임시 저장 복원 |
| `git rebase -i HEAD~3` | 최근 3개 커밋 대화형 편집 |

### 자주 쓰는 log 옵션

| 명령어 | 설명 |
|--------|------|
| `git log --oneline` | 한 줄 요약 (가장 자주 씀) |
| `git log --oneline --graph --all` | 브랜치 분기 시각화 |
| `git log --stat` | 파일별 변경 줄 수 요약 |
| `git log -p -1` | 최근 1개 커밋의 diff 포함 |
| `git log -n 5` | 최근 5개만 |
| `git log --author="이름"` | 특정 작성자 커밋만 |

---

## 7. 일일 루틴

### 혼자 작업할 때

```bash
# 코드 수정 후
git status                           # 1. 현재 상태 파악
git diff                             # 2. 변경사항 확인
git add .
git diff --staged                    # 3. 커밋 전 최종 확인
git commit -m "feat: add new feature"
git push
```

### 팀 작업할 때

```bash
# 작업 시작 전 — 항상 최신 main에서
git checkout main && git pull origin main
git checkout -b feat/내기능

# 작업 중
git add . && git commit -m "feat: ..."

# 완료 후
git push origin feat/내기능
# → GitHub에서 PR 생성 → 리뷰 → Squash merge

# merge 후 정리
git checkout main && git pull origin main
git branch -d feat/내기능
```

### 작업 중 긴급 브랜치 전환

```bash
git stash                    # 현재 작업 임시 저장
git checkout other-branch    # 다른 브랜치로 이동해서 작업
git checkout feat/내기능      # 돌아오기
git stash pop                # 임시 저장 복원
```

---

## 참고 링크

- [HackMD Git 명령어 총정리 (코딩알려주는 누나)](https://hackmd.io/@oW_dDxdsRoSpl0M64Tfg2g/ByfwpNJ-K)
- [Conventional Commits 공식 스펙](https://www.conventionalcommits.org/ko/v1.0.0/)
- [shields.io 뱃지 생성기](https://shields.io/)
- [GitHub Flow 공식 가이드](https://docs.github.com/en/get-started/quickstart/github-flow)

---

*Last updated: 2026.06.15 · JeongWoo Song*
