---
date: '2025-05-03T23:50:44+09:00'
draft: false
title: "GitHub 라벨 자동 업데이트 스크립트"
---

GitHub 프로젝트에서 라벨(label)을 일괄적으로 관리하고 싶을 때 사용할 수 있는 Bash 스크립트입니다. 기존 라벨을 삭제하고, 지정한 JSON 파일을 기반으로 새로운 라벨을 생성합니다.

## 🔧 사전 준비

1. **gh (GitHub CLI) 설치**
   - [GitHub CLI 다운로드 페이지](https://cli.github.com/)

2. **jq 설치** (JSON 파싱을 위한 도구)
   - macOS: `brew install jq`
   - Ubuntu: `sudo apt install jq`

3. **GitHub 인증** (1회만 수행)
   ```bash
   gh auth login
   ```

## 📁 라벨 정의 파일

`.github/labels.json` 파일에 라벨을 정의합니다:

```json
[
  {
    "name": "type: bug",
    "color": "#d73a4a",
    "description": "예상과 다른 동작 또는 오류"
  },
  {
    "name": "type: feature",
    "color": "#a2eeef",
    "description": "새로운 기능 제안"
  }
]
```

## 📜 스크립트 내용

`create-labels.sh` 파일을 생성하고 다음 내용을 작성합니다:

```bash
#!/bin/bash

REPO="vardoong/speed-all-in-one"
LABELS_FILE="./.github/labels.json"

if ! command -v jq &> /dev/null; then
  echo "jq 필요. 설치 후 다시 시도하세요."
  exit 1
fi

echo "🔄 라벨 강제 업데이트 시작..."

jq -c '.[]' "$LABELS_FILE" | while read -r label; do
  NAME=$(echo "$label" | jq -r '.name')
  COLOR=$(echo "$label" | jq -r '.color' | sed 's/#//')
  DESCRIPTION=$(echo "$label" | jq -r '.description')

  echo "🛠️ $NAME 업데이트 중..."

  # 기존 라벨 있으면 삭제
  gh label delete "$NAME" --repo "$REPO" --yes 2>/dev/null

  # 새 라벨 생성
  gh label create "$NAME" \
    --color "$COLOR" \
    --description "$DESCRIPTION" \
    --repo "$REPO"
done

echo "✅ 라벨 재설정 완료!"
```

## ✅ 실행 방법

1. 스크립트에 실행 권한 부여:
   ```bash
   chmod +x create-labels.sh
   ```

2. 스크립트 실행:
   ```bash
   ./create-labels.sh
   ```

## ⚠️ 주의 사항

- 기존 라벨은 무조건 삭제되고 다시 생성됩니다.
- 라벨 이름이 같더라도 색상이나 설명이 다르면 무조건 덮어씌워집니다.