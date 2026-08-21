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

## 설치 (팀원용)

**Claude Code**

```
/plugin marketplace add tecace-soft/tecace-skills
/plugin install tecace-design@tecace
```

**claude.ai 웹 / Desktop / Cowork**

Customize → Plugins 탭 → Personal plugins의 + 버튼 → Add marketplace → Add from a repository → tecace-soft/tecace-skills

## 업데이트

관리자 (스킬 수정 후):

```
git add . && git commit -m "수정 내용" && git push
```

또는 GitHub 웹에서 파일을 열고 연필 아이콘으로 바로 수정해도 됩니다.

팀원:

```
/plugin marketplace update tecace
/plugin update tecace-design@tecace
```

두 줄 모두 필요합니다. 첫 줄만 하면 카탈로그만 갱신되고 설치본은 그대로입니다. 적용하려면 Claude Code 재시작이 필요합니다. 웹과 Cowork은 플러그인 화면에서 갱신합니다.

자동 반영은 아닙니다. 팀원이 한 번은 눌러야 합니다.

## version을 두지 않는 이유

plugin.json에 version이 없으면 커밋 SHA가 버전 역할을 합니다. 그래서 push할 때마다 팀원이 새 버전을 받을 수 있습니다. 반대로 version을 1.0.0 같이 박아두고 올리지 않으면, 새 커밋을 push해도 기존 사용자는 캐시본을 계속 씁니다.

## 스킬 수정 규칙

수정은 반드시 이 레포에서 합니다. 로컬에 설치된 스킬은 커밋 SHA 이름의 캐시 폴더 안에 들어가기 때문에, 거기를 고쳐도 다른 사람에게 전파되지 않고 업데이트 시 사라집니다.

## 스킬 추가하기

1. plugins/tecace-design/skills/ 아래에 스킬 폴더를 통째로 넣습니다.
2. 별개 주제라면 plugins/ 아래에 새 플러그인 폴더를 만들고 marketplace.json의 plugins 배열에 추가합니다.
3. claude plugin validate . 로 확인 후 push합니다.
