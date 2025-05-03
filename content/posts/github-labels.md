---
title: "GitHub Labels 자동 생성 스크립트 만들기"
date: 2025-05-02T15:21:04+09:00
draft: false
---

이 글은 GitHub 이슈/PR 라벨을 일관성 있게 관리하기 위한 `labels.sh` 자동화 스크립트를 만드는 방법과 실행 절차를 소개합니다.

---

## 1. `labels.sh` 파일을 만드는 이유와 작성 방식

### ✅ 왜 필요한가?
- 협업 중 라벨 이름이나 색상이 제각각이면 이슈 분류나 필터링이 어려움  
- 레포지토리마다 라벨을 매번 수동으로 만들기 귀찮음  
- `.sh` 파일과 `labels.json` 파일로 라벨을 템플릿처럼 관리 가능  

### ✅ 어떤 구조로 작성하는가?

`labels.sh`는 GitHub CLI(`gh`)와 JSON 파서를 위한 `jq`를 이용합니다.

#### 예시: labels.sh

```bash
#!/bin/bash

REPO="your-username/your-repo"
LABELS_FILE="labels.json"

if ! command -v jq &> /dev/null; then
  echo "❌ jq 명령어가 필요합니다. 설치 후 다시 시도하세요."
  exit 1
fi

echo "🔄 GitHub 라벨을 업데이트합니다..."

jq -c '.[]' "$LABELS_FILE" | while read -r label; do
  NAME=$(echo "$label" | jq -r '.name')
  COLOR=$(echo "$label" | jq -r '.color' | sed 's/#//')
  DESCRIPTION=$(echo "$label" | jq -r '.description')

  echo "🛠️ $NAME 처리 중..."

  gh label delete "$NAME" --repo "$REPO" --yes 2>/dev/null

  gh label create "$NAME" \
    --color "$COLOR" \
    --description "$DESCRIPTION" \
    --repo "$REPO"
done

echo "✅ 완료!"
```

#### 예시: labels.json

```json
[
  { "name": "type: bug", "color": "#FF2E63", "description": "버그 또는 오류" },
  { "name": "type: feature", "color": "#08D9D6", "description": "새로운 기능" },
  { "name": "priority: high", "color": "#FF0000", "description": "긴급 처리" }
]
```

### ✅ 컬러나 종류는 어떻게 정하나?
- 색상은 팀의 컨셉 컬러를 기반으로 선정 (`#FF2E63`, `#252A34`, `#08D9D6`, `#EAEAEA` 등)  
- `type`, `priority`, `status` 같이 prefix로 그룹을 나누면 관리하기 쉬움  
- 색상 조합 도구: https://coolors.co/  

---

## 2. 파일 만들고 실행하는 방법

### 🧱 준비물
- GitHub CLI 설치: https://cli.github.com/  
- `jq` 설치  
  - macOS: `brew install jq`  
  - Ubuntu: `sudo apt install jq`  

### 📁 실행 절차

1. 위 `labels.sh`와 `labels.json` 파일을 원하는 폴더에 저장  
2. `.sh` 파일 실행 권한 부여  
   ```bash
   chmod +x labels.sh
   ```
3. GitHub CLI 로그인  
   ```bash
   gh auth login
   ```
4. 실행  
   ```bash
   ./labels.sh
   ```
5. 완료 후 GitHub 레포지토리 > Issues 탭에서 라벨이 반영됐는지 확인

---

## 📌 참고

- GitHub 공식 `gh label` 문서: https://cli.github.com/manual/gh_label  
- `jq` 튜토리얼: https://stedolan.github.io/jq/tutorial/  

---

이렇게 만들어두면 새로운 프로젝트 세팅할 때나 다른 레포로 옮길 때 아주 유용하게 쓸 수 있습니다. 😉
