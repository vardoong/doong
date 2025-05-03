---
date: '2025-05-03T20:52:46+09:00'
draft: false
title: "Shell 설정 파일 정리"
---


# 🔍 Shell 설정 파일 정리: `.bashrc` vs `.zshrc`

터미널 설정을 자주 바꾸다 보면 `~/.bashrc`와 `~/.zshrc`가 헷갈릴 수 있다. 두 파일의 공통점과 차이점을 정리해본다.

---

## ✅ 공통점

- 모두 **셸 실행 시 로드되는 설정 파일**이다.
- 보통 다음과 같은 용도로 사용된다:
  - 환경 변수 설정 (`export`)
  - alias 등록 (`alias gs='git status'`)
  - PATH 추가
  - 프롬프트 설정
  - 셸 시작 시 자동 실행 명령어

---

## ✅ 차이점 요약

| 항목 | `~/.bashrc` | `~/.zshrc` |
|------|-------------|-------------|
| 대상 셸 | Bash (`bash`) | Zsh (`zsh`) |
| 사용 위치 | Ubuntu 등 대부분의 리눅스 | macOS (Catalina 이상), 또는 zsh 사용 시 |
| 실행 시점 | Bash의 **인터랙티브 셸** 시작 시 | Zsh의 **인터랙티브 셸** 시작 시 |
| 기타 | 종종 `~/.bash_profile`과 함께 사용됨 | 설정 파일이 단일화됨 (`~/.zshrc`만 씀) |

---

## ✅ 내 셸은 뭘까?

터미널에서 아래 명령어를 입력해보자:

```bash
echo $SHELL
```
- 결과가 /bin/zsh → zsh 사용 중
- 결과가 /bin/bash → bash 사용 중


## ✅ 참고 정보
- bash
- ~/.bash_profile: 로그인 셸에서 실행
- ~/.bashrc: 비로그인 인터랙티브 셸에서 실행
- zsh
- 대부분의 설정은 ~/.zshrc 하나로 통합


## ✅ 요약
- macOS 유저는 보통 ~/.zshrc만 수정하면 됨
- Linux 유저는 ~/.bashrc가 메인 설정 파일일 가능성이 높음
- 자신의 셸이 무엇인지 먼저 확인하는 게 중요!

