# TecAce Skills Marketplace

TecAce 사내 Claude 스킬 마켓플레이스입니다. 원본은 이 레포 하나이며, 팀원은 이 레포를 등록해두고 업데이트만 받으면 됩니다.

레포 = 마켓플레이스, 그 안에 플러그인, 플러그인 안에 스킬. 3단 구조입니다.

## 구조

```
.claude-plugin/marketplace.json
plugins/tecace-design/.claude-plugin/plugin.json
plugins/tecace-design/skills/tecace-dashboard-ui/SKILL.md
plugins/tecace-design/skills/tecace-dashboard-ui/assets/
plugins/tecace-design/skills/tecace-dashboard-ui/references/
plugins/tecace-design/skills/tecace-dashboard-ui/templates/
```

## 사전 준비 (private 레포이므로 필수)

이 레포는 private입니다. claude.ai 웹 / Desktop / Cowork에서 붙이려면 **Claude GitHub App이 tecace-soft 조직과 이 레포에 접근 권한을 가지고 있어야** 합니다. 권한이 없으면 Sync 시 아래 메시지가 나옵니다.

Repository not accessible. If it's private, the Claude GitHub App needs access to this repository.

그 화면의 Install the Claude GitHub App 버튼을 눌러 조직과 레포를 선택하면 됩니다. 조직 owner 승인이 필요할 수 있습니다.

Claude Code(CLI)는 사용자의 기존 git 자격증명을 쓰기 때문에 이 절차가 필요 없습니다.

## 설치 (팀원용)

**Claude Code**

```
/plugin marketplace add tecace-soft/tecace-skills
/plugin install tecace-design@tecace
```

**claude.ai 웹 / Desktop / Cowork**

Customize → Plugins 탭 → 오른쪽 Add 버튼 → Add marketplace → Add from a repository → tecace-soft/tecace-skills 입력 → Sync

Sync automatically 토글은 켜둔 채로 두십시오.

## 업데이트

관리자 (스킬 수정 후):

```
git add . && git commit -m "수정 내용" && git push
```

또는 GitHub 웹에서 파일을 열고 연필 아이콘으로 바로 수정해도 됩니다.

**팀원 — claude.ai 웹 / Desktop / Cowork**

따로 할 것이 없습니다. 마켓플레이스를 추가할 때 Sync automatically를 켜두면 레포가 바뀌면 자동으로 따라옵니다. 다만 반영까지 얼마나 걸리는지는 명시되어 있지 않으니, 급한 변경은 플러그인 화면에서 수동으로 한 번 갱신해 주십시오.

**팀원 — Claude Code(CLI)**

```
/plugin marketplace update tecace
/plugin update tecace-design@tecace
```

두 줄 모두 필요합니다. 첫 줄만 하면 카탈로그만 갱신되고 설치본은 그대로입니다. 적용하려면 Claude Code 재시작이 필요합니다. CLI는 자동 반영이 아닙니다.

## version을 두지 않는 이유

plugin.json에 version이 없으면 커밋 SHA가 버전 역할을 합니다. 그래서 push할 때마다 팀원이 새 버전을 받을 수 있습니다. 반대로 version을 1.0.0 같이 박아두고 올리지 않으면, 새 커밋을 push해도 기존 사용자는 캐시본을 계속 씁니다.

## 스킬 수정 규칙

수정은 반드시 이 레포에서 합니다. 로컬에 설치된 스킬은 커밋 SHA 이름의 캐시 폴더 안에 들어가기 때문에, 거기를 고쳐도 다른 사람에게 전파되지 않고 업데이트 시 사라집니다.

## 스킬 추가하기

(1) plugins/tecace-design/skills/ 아래에 스킬 폴더를 통째로 넣습니다.
(2) 별개 주제라면 plugins/ 아래에 새 플러그인 폴더를 만들고 marketplace.json의 plugins 배열에 추가합니다.
(3) claude plugin validate . 로 확인 후 push합니다.
