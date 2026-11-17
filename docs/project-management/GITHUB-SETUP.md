# 차곡ERP GitHub Organization 생성·권한·협업 설정 가이드

> 작성일: 2026-11-17 · **실제 계정/조직에는 아직 적용하지 않은 설정 가이드**. GitHub UI 메뉴는 업데이트에 따라 이름·위치가 일부 달라질 수 있다.  
> 출처: GitHub 공식 Docs 및 제공된 `../../CONTRIBUTING.md`, `../../.github/pull_request_template.md`. 사용자 요청에 맞춘 **제안 정책**은 별도 표시.

## 0. 먼저 선택할 것

**추천 조직 이름:** `chagok-erp` (가용 여부는 GitHub 생성 폼에서 확인). **저장소 이름:** `chagok-erp` → URL 예시 `https://github.com/chagok-erp/chagok-erp` (실제 URL 아님).

**협업 전략:** 2명 + 단일 monorepo + **기능 브랜치 → `develop` → `main`** + PR 리뷰 후 병합 + 최소 권한. 백엔드·프론트엔드는 최신 `develop`에서 영역별 기능 브랜치를 만들고, 완료한 기능을 `develop`에 통합한다. 연동 검증을 마친 결과를 `develop` → `main` PR로 최종 반영한다. 상세 규칙은 `../../CONTRIBUTING.md`를 따른다.

**주의 (2026-11-17 기준 공식 Docs):** GitHub Free **organization의 private repository**에서는 branch protection / rulesets **강제 적용 불가**. 이 조합에서는 직접 push/강제 push를 GitHub가 막지 못하므로 PR 관례만으로 통제된다. 보호 규칙 강제가 꼭 필요하다면 ① public repository(GitHub Free 조직) 또는 ② private repository+GitHub Team(또는 상위 지원 플랜)을 선택해야 한다. **Pro 개인 구독이 조직 private 저장소를 자동 보호하지 않는다.**

| 선택 | 장점 | 제약/추천 |
|---|---|---|
| **A. GitHub Free 조직 + Private 저장소** | 비공개로 초기 개발, 비용 부담 낮음 | 보호 규칙 강제 불가. **초기 추천**, 팀 간 PR 약속과 주기적 백업 필요 |
| B. GitHub Free 조직 + Public 저장소 | PR 승인/CI/브랜치 보호 강제 가능 | 코드·이력이 공개됨. 민감 데이터·키 완전 제거 및 공개 합의 필요 |
| C. GitHub Team 조직 + Private 저장소 | 비공개 + 강제 규칙 적용 | 유료, 비용·결제 합의 필요 |

## 1. GitHub 조직 만들기 (현재 미실행)

1. 각자의 GitHub 개인 계정 준비. 계정은 서로 공유하지 않는다.
2. GitHub 로그인 → 오른쪽 상단 프로필 사진 → **Settings** → 왼쪽 **Organizations** → **New organization**. 바로가기: https://github.com/settings/organizations . 공식 절차: https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch
3. 플랜 선택 시 먼저 **Free** 선택(비공개 강제 규칙이 필요하면 Team 요금 확인 후 별도 선택).
4. Organization account name 입력(예: `chagok-erp`), 연락 이메일 입력. 이름 충돌이면 `chagok-erp-team` 등 대안을 사용.
5. 약관·생성·이메일 확인 등 화면 지시에 따라 완료.
6. 생성 완료 후 조직 URL을 두 사람이 공유하고, 팀원의 GitHub **정확한 사용자명**을 확인.

**현재 내가 확인하지 못한 것:** 사용자 GitHub 계정 권한·연결 여부, 조직 이름 가용성, 실제 생성 결과. 조직이 만들어졌다고 가정하지 말 것.

## 2. 소유권과 팀원 초대 — 충돌 예방의 핵심

### 2.1 Owner 선택

- **추천(서로 동등한 공동 프로젝트라는 전제): 두 사람 모두 Organization Owner**. 한 명 계정 분실/탈퇴 시에도 조직 운영과 결제·접근 복구 가능.
- **대가:** Owner는 모든 조직 리포지토리에 Admin 권한을 갖고, 조직·설정을 바꿀 수 있어. 기술적으로 서로를 절대적으로 제한할 수 없으므로 **서로의 신뢰와 서면 운영 규칙**이 필요. Owner를 1명만 두면 권한 위험은 줄지만 한 명에게 통제권이 집중됨.
- 두 사람 모두 **개인 2FA + 복구 코드 백업**을 먼저 완료. GitHub 계정을 공유하거나 동일한 PAT를 같이 사용하지 않는다.
- **두 사람의 사전 서면 합의(권장):** 조직 삭제·저장소 이전/삭제·공개 전환·유료 플랜 변경·API/DB 공통 계약 변경·배포 비밀값 변경은 두 사람의 명시적 동의 후 수행. GitHub가 이 약속을 자동 강제하는 것은 아니다.

### 2.2 멤버 초대

조직 → **People** → **Invite member** → 친구 GitHub ID 검색 → 조직 초대. 역할은 처음 **Member**로 넣고, 공동 Owner로 정했다면 초대 수락 후 조직의 People/Role 관리에서 **Owner**로 올린다. GitHub 초대는 상대가 수락해야 완료된다.

출처: https://docs.github.com/en/organizations/managing-membership-in-your-organization

### 2.3 개발 팀

조직 → **Teams** → **New team** → `developers` 생성 → 두 사람 추가. 조직 저장소를 만들면 `developers` 팀에 **Write** 접근을 준다. `Write`면 브랜치 작성/PR 개발이 가능하지만 저장소의 위험한 관리 작업은 별도 Admin 권한이 필요.

**Owner와 Team Write는 별개**: Owner는 이미 조직 전역 Admin이므로 `developers` 팀의 Write만으로 Owner 권한이 축소되지는 않는다. 팀 설정은 나중에 일반 기여자를 초대할 때 권한 범위를 명확히 하는 효과가 큼.

출처: https://docs.github.com/en/organizations/organizing-members-into-teams/creating-a-team

## 3. 조직 기본 권한·보안 설정

조직 → **Settings** 기준. 설정 후 아래 체크리스트에 실제 적용 여부 기록.

| 메뉴 | 권장 값 | 이유/주의 |
|---|---|---|
| `Member privileges` → `Base permissions` | **No permission** 또는 가장 낮은 허용 수준 | 모든 신규 멤버에게 모든 저장소 접근권한이 자동 부여되지 않음. 공개 저장소는 공개 특성상 누구나 읽을 수 있음 |
| `Member privileges` → `Repository creation` | 멤버의 새 저장소 생성 권한 **최소화/해제** | 임의 저장소·공개 누출 위험 감소. Owner는 생성 가능 |
| `Member privileges` → `Repository deletion and transfer` | 멤버의 삭제·이전 권한 **해제** | 원치 않는 이동·삭제 방지. Owner 자체는 여전히 강한 권한 |
| `Settings` → `Authentication security` | **Require 2FA** | 두 사람 모두 2FA 활성화 확인 후 적용. 그렇지 않으면 접근이 차단될 수 있음 |
| `Settings` → `Actions` → `General` | 신뢰 가능한 Actions만 허용, `GITHUB_TOKEN` 기본 **Read** | 서드파티 액션 및 과도한 토큰 권한 최소화 |
| `Member privileges`/관련 보안 설정 | Private 저장소 Fork 허용 여부 검토 후 보수적으로 | 개인 저장소 복제/원본 통제와 관련. 팀 개발에 꼭 필요하지 않음 |

공식 문서:  
- https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization  
- https://docs.github.com/en/organizations/managing-organization-settings/restricting-repository-creation-in-your-organization  
- https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-two-factor-authentication-for-your-organization/requiring-two-factor-authentication-in-your-organization  
- https://docs.github.com/en/organizations/managing-organization-settings/disabling-or-limiting-github-actions-for-your-organization

## 4. 저장소 생성과 기본 레이아웃

1. 조직 페이지 → **Repositories → New**.
2. Owner: **조직 선택**, Repository name: `chagok-erp`, Description: `제조기업 구매·재고 ERP 및 통합 AI Assistant`.
3. Visibility: **Private (초기 추천)**. `main` 기본 브랜치를 쓰도록 **Add README** 선택.
4. 생성 후 **Settings → Collaborators and teams / Manage access**에서 `developers` 팀에 Write 권한.
5. **Settings → General → Pull Requests**: Squash merging과 Merge commits를 모두 허용한다. 기능 PR은 Squash merge, `develop` → `main` PR은 Merge commit을 사용한다. 병합 후 기능 브랜치는 정리하고 `develop`은 삭제하지 않는다. 자동 head branch 삭제는 끄고 기능 브랜치를 수동 정리한다.
6. 최상위 `../README.md`, `CLAUDE.md`, `../../.editorconfig`, `../../CONTRIBUTING.md`, `../../.github/pull_request_template.md` 업로드. PR 템플릿은 제공된 파일을 정확한 경로로 옮긴다.
7. 초기 Monorepo 구조 **제안**(서버·AI 언어 PoC 결정 후 세부 폴더 변경 가능):

```text
chagok-erp/
├── backend/             # Spring Boot 모듈형 모놀리스 (A+B)
├── frontend/            # React·TypeScript (A 공통 디자인, 각 화면 분담)
├── ai/                  # Rust 여부 미확정, PoC 이후 확정
├── docs/                # PRD, DB-SPEC, SCREEN-SPEC, OpenAPI, ADR
├── infra/               # docker compose, 배포·모니터링 (A+B)
├── .github/
│   ├── pull_request_template.md
│   └── workflows/        # 실제 빌드가 준비되면 CI 추가
├── .editorconfig
├── CONTRIBUTING.md
├── CLAUDE.md
└── README.md
```

**GitHub Actions 시크릿/환경변수:** 사용 서비스가 결정된 뒤 저장소 **Settings → Secrets and variables → Actions**에서 암호화 Secret 추가. `.env.example`에는 빈 값만 작성. GitHub Actions 권한은 job 단위 최소화(예: `contents: read`), PR 이벤트가 포크 코드를 실행할 경우 시크릿/쓰기 권한 노출 주의.

## 5. `develop`·`main` 브랜치 보호 — 플랜 분기

### 5.1 Public GitHub Free 조직 또는 Private GitHub Team일 때

저장소 → **Settings → Rules → Rulesets → New ruleset → New branch ruleset** 또는 **Branches → Add branch protection rule**(UI에 따라 메뉴 다름).

- 대상 브랜치: `develop`과 `main`. 두 브랜치 모두 보호한다. `develop`은 기능 통합, `main`은 최종 검증 결과 반영에 사용한다.
- `develop` → `main`의 Merge commit을 허용하도록 `main`에 선형 이력 강제를 설정하지 않는다.
- **Require a pull request before merging** = ON.
- **Required approvals** = 1 (2인 개발 시 PR 작성자의 상대 팀원이 승인).
- **Dismiss stale pull request approvals when new commits are pushed** = ON 권장 (리뷰 뒤 코드 바꾸면 재검토).
- **Require status checks to pass** = CI가 실제 존재하고 통과한 뒤 ON. 존재하지 않는 검사명을 강제하면 PR이 영구 차단될 수 있음.
- **Block force pushes** = ON, **Restrict deletions** = ON.
- **Do not allow bypassing / Include administrators** (표시되는 경우) = 가능한 범위에서 ON. Owner가 조직/저장소 규칙을 수정할 권한까지 봉쇄하는 기능은 아님.
- 선택적으로 **Require conversation resolution before merging** = ON.
- Enforcement = Active, 실제 PR 시뮬레이션으로 적용 여부 검증.

출처: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets

### 5.2 Private GitHub Free 조직을 선택했다면

- 보호 설정을 시도해도 **강제 적용 가능한 규칙이 아니다.** GitHub Free로 비공개 조직 저장소의 PR 승인 1건을 기술적으로 필수화했다고 표시하지 않는다.
- **운영 규칙:** 영역별 기능 브랜치 → `develop` PR → 상대 검토·CI 확인 → 연동 검증 → `develop` → `main` PR. `develop`·`main` 직접 push·force push·삭제 금지 **자율 합의**.
- 코드는 Pull Request 활동과 리뷰 기록을 근거로 관리. 저장소 백업/오프사이트 보관을 정기 실행.
- 강제 설정이 중요한 경우 팀이 공개 전환 또는 GitHub Team으로 전환 여부를 판단.

## 6. PR 및 Git 운영 규칙: 현재 제공된 파일을 그대로 기준으로

`../../CONTRIBUTING.md`에 따른 흐름:

```bash
git switch develop
git pull --ff-only origin develop
git switch -c feat/backend/pr-draft
# 작업 후
git add .
git commit -m "feat: 구매요청 초안 저장 기능 추가"
git push -u origin feat/backend/pr-draft
```

GitHub에서 `Compare & pull request` → Base=`develop` → 템플릿 `../../.github/pull_request_template.md` 작성 → **상대 팀원 Review 요청** → CI/수동 검증 결과 확인 → 승인 후 Squash merge. 프론트엔드 기능 브랜치는 `feat/frontend/pr-draft`처럼 생성한다.

최초 저장소 준비 시 `main`에서 `develop`을 만들고 원격에 push한다. 이후에는 `develop`을 계속 유지한다.

```bash
git switch main
git pull --ff-only origin main
git switch -c develop
git push -u origin develop
```

최종 반영은 Base=`main`, Compare=`develop`으로 PR을 생성한다. 백엔드·프론트엔드 연동 및 주요 시나리오 검증 결과를 확인한 후 **Merge commit**으로 병합한다.

- 기능 하나당 PR 하나. 최소 변경, 무관한 포맷 수정 금지.
- **공통 API/DB 변경:** 두 사람이 사전 협의하고 계약 파일(`../api/openapi.yaml`, `../database/DB-SPEC.md`)과 영향 구현을 PR에서 함께 리뷰.
- **Migration 파일:** 번호/버전 충돌 방지 위해 변경을 먼저 공유. 각각 임의로 최상단 번호를 쓰지 않음.
- **브랜치 규칙:** `<type>/<backend|frontend|shared>/<작업-설명>`. `../../CONTRIBUTING.md` 타입을 준수한다.
- **충돌 시:** 최신 `develop` 변경을 기능 브랜치에 반영해 해결하고 검증한다. 다른 팀원 변경을 강제로 덮는 `git reset --hard`/`push --force` 금지.
- **이슈/PR:** 가능하면 작업 ID/담당자/검증/이슈 링크를 남기고 리뷰 요청 후 병합.

## 7. 팀원 간 문제가 생기지 않도록 한 번만 합의할 것

1. **조직/코드 소유권**: 개인 GitHub가 아니라 조직 소유, 두 사람의 포트폴리오 기여를 인정한다는 합의. 법률상 저작권 지분은 GitHub 소유자 역할만으로 결정되지 않으니 필요 시 별도 약정.
2. **Owner 역할**: 두 사람 모두 Owner로 할지, 1 Owner+1 Admin으로 할지. 2 Owner는 복구성·대등성, 반면 둘 다 파괴적 권한 보유.
3. **공통 파일 변경**: `../api/openapi.yaml`, DB migration, docker compose, 공통 인증·컴포넌트는 PR에 상대 필수 리뷰(기술적 강제 가능 여부는 플랜에 따라 다름).
4. **비밀정보**: 각자 개인 토큰/키 사용, 유료 API 키·클라우드 결제 공유 범위와 비용 상한 설정, 소유자가 이탈할 때의 키 회전 계획.
5. **배포와 백업**: 운영 DB 백업 주기, 배포 승인자, 긴급 복구 절차, 공용 서비스 비용 분담.
6. **리포지토리 공개 시점**: 취업 포트폴리오 용도 공개 여부는 두 사람 합의 후 변경.
7. **이탈/분쟁/프로젝트 종료**: 저장소 삭제·이전·아카이브, 개인 포트폴리오 활용, 접근권한 정리, 산출물 보관에 대한 최소한의 합의문 작성.

**중요:** Owner 두 명에게 동시에 Admin 권한이 있으므로 플랫폼 설정만으로 서로의 일방적 삭제/이전 위험을 완전히 제거할 수 없다. 신뢰·기록·외부 백업·사전 합의를 병행해야 한다.

## 8. 실제 생성·설정 확인 체크리스트

- [ ] 조직 이름 가용성 확인 및 GitHub Organization 생성 URL 기록: __________________
- [ ] A/B 개인 계정 2FA 및 복구 코드 확보
- [ ] 팀원 초대 수락 확인·Owner 배분 서면 합의
- [ ] `developers` Team 생성, 저장소 Write 부여
- [ ] Organization base permission 최소화, 멤버 저장소 생성 권한 제한
- [ ] 삭제/이전 권한 제한(Owner 권한 한계 확인)
- [ ] 개인 계정 및 Actions 토큰 최소 권한 설정
- [ ] `chagok-erp` 저장소 생성/Private 또는 Public 선택 기록
- [ ] `main`·`develop` 생성, 기능 PR의 base=`develop` 및 최종 반영 PR의 base=`main` 확인
- [ ] PR 템플릿·CONTRIBUTING·CLAUDE·editorconfig 반영
- [ ] 브랜치 보호가 **실제로 강제되는 플랜인지** 확인; 가능한 경우 리뷰 1명 + CI + 강제 push/삭제 차단 적용
- [ ] 테스트 PR 생성: 작성자 직접 병합 차단 여부/승인/CI를 실제 확인
- [ ] 실 API 키·비밀번호·DB 자격증명 Git 이력에 미포함 확인
- [ ] 조직·배포·코드 소유권과 비용·공개 규칙 A/B 합의

## 9. GitHub 공식 참고 URL

- Organization 만들기: https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch
- Team 만들기: https://docs.github.com/en/organizations/organizing-members-into-teams/creating-a-team
- 조직 기본 권한: https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization
- Organization 2FA: https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-two-factor-authentication-for-your-organization/requiring-two-factor-authentication-in-your-organization
- Branch protection 지원 플랜: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
- Rulesets 지원 플랜: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets
- Workflow token 권한: https://docs.github.com/en/organizations/managing-organization-settings/disabling-or-limiting-github-actions-for-your-organization
